# Anbindung B-Control Manager EM 100, 210 oder EM300 an openHAB 
Das folgende Beispeil zeigt wie mit openHAB2 die Leistung und Energie von einen TQ Manager EM210 und EM300 ausgelesen werden kann. Der TQ Manager wurde früher unter der Marke b-control vertrieben, daher wird in der Beschreibung beide Bezeichnungen verwendet.  


## Introduction
Der B-Control Energie Manager (EM) wird vorzugsweise in Verbindung mit einer Photovoltaik-Anlage verwendet um die Einspeise oder Bezugsleistung und der Energie zu messen. Auf der Homepage von TQ können für die [TQ Manager](https://www.tq-automation.com/de/Service-Support/Downloads/Downloads-Energiemanagement) zusätzliche Spezifikationen heruntergeladen werden. Das folgende Beispiel beruht auf der Beschreibung der [JSON Schnittstelle](https://www.tq-automation.com/de/content/download/10996/file/TQ%20Energy%20Manager%20-%20JSON-API.0104.pdf).

## Auslesen der Daten via JSON
Diese Lösung verwendet das openHAB2 [exec Binding](https://www.openhab.org/addons/bindings/exec/) in Kombination mit einem Linux Bash Skript sowie eine openHAB Rule in Verbindung mit [JsonPath](https://www.openhab.org/addons/transformations/jsonpath/).
Um das Skript ausführen zu können, mussen das Programm curl instaliert sein.
In dem meisten Linux Distributionen ist curl bereits installiert oder kann einfach aus den Quellen nach installiert werden.

Das folgende Bash-Skript *bcontrol* liest die Daten über die JSON Schnittstelle des b-control/TQ Energie Manager und gibt diese auf der Konsole aus.  

Als erster Schritt wird in dem von openHAB vorgesehen Verzeichnis (z.b. */etc/openhab2/scripts*) die Datei *bcontrol* angelegt und den folgenden Inhalt hinein kopieren.

```
#!/bin/bash
# Script um die JSON Schnittstelle des TQ Managers auszulesen

# Eine Fehler exit Funktion
function error_exit
{
  echo "$1" 1>&2
  exit 1
}

# Pfad in dem die JSON Datei abgelegt werden soll
JSONPfad="/etc/openhab2/scripts"

# Datei in der die JSON Ausgabe gespeichert wird
JSONDatei="bcontrol.out"

# Datei in dem curl die Cookies ablegt
cookieDatei="cookie.txt"

# IP Addresse des TQManggers
bcontrolIP="192.168.1.12"

# Aufbau der benoetigten Pfade und Links
CookiePfad="${JSONPfad}/${cookieDatei}"
JSONLink="http://${bcontrolIP}/mum-webservice/data.php"
LoginLink="${bcontrolIP}/start.php"

# Daten (JSON) vom b-control Manager abfragen
Ausgabe=$(curl -s -b $CookiePfad -X GET $JSONLink)

# Überprüfe ob in der vorherigen Zeile ein Fehler aufgetreten ist
if ! [ "$?" = "0" ]; then
  error_exit "Fehler 1"
fi

if [[ $Ausgabe = *authentication* ]]
then
  #echo "Login Fehler: Login wird wiederholt!"
  curl -s --cookie-jar $CookiePfad $LoginLink > /dev/null
  curl -s -b $CookiePath --cookie-jar $CookiePath -d "login=<serial-no>&password=<password>&save_login=1" $LoginLink > /dev/null
  Ausgabe=$(curl -s -b $CookiePfad -X GET $JSONLink)
fi

echo "$Ausgabe"
exit 0

```  

* * * * *
<tr>
<td> ![Hinweis!](images/Warning.png "Hinweis: Auf Dateierweiterung achten!") </td>
<td> Bitte ersetzen sie <serial-no> mit dem Benutzernamen der im B-Control Manager gesetzt ist (Normalerweise ist dies die Seriennummer), <password> mit dem Password des Benutzers (ist kein Passwort gesetzt, bitte dies frei lassen) und noch die IP-Adresse des B-Control Manager!</td>
</tr>
</table>
* * * * *

Bitte vergessen Sie nicht, die Datei mit Hilfe von [chmod](https://wiki.ubuntuusers.de/chmod/) als ausführbar zu kennzeichnen.
```
sudo chmod ug+x ./bcontrol
```

Zusätzlich müssen die Rechte des Script mit Hilfe der Befehle [chown](https://wiki.ubuntuusers.de/chown/) und [chgrp](https://wiki.ubuntuusers.de/chgrp/) angepasst werden, damit openHAB das Script ausführen kann.
Für mein System verwende ich den OWNER openhab und die GRUPPE pi, dies unterscheidet sich je nach verwendetem System und Distribution.  

```
sudo chown openhab bcontrol
sudo chgrp pi bcontrol
```

Eine Ausgabe von [ls](https://wiki.ubuntuusers.de/ls/) sieht wie folgt aus:
```
-rwxr-xr-- 1 openhab pi      1327 Jan  9 22:35 bcontrol
```

Nun kann das Skript vorab schon mal ausgeführt (ggf. als admin) und korrekte Funktionsweise geprüft werden:  
```
/etc//openhab/scripts/bcontrol
```  

Wenn soweit alles funktioniert, kann openHAB eingerichtet werden. Installieren Sie das [exec Binding](https://www.openhab.org/addons/bindings/exec/) sowie die [JsonPath](https://www.openhab.org/addons/transformations/jsonpath/) Transformation.
Erstellen Sie folgende Items und Regel (Rule) um das Script einzubinden.

**Item**:  

```
String    TQManager_DatenString             "Ausgabe: [%s]"
Number    TQManager_ZaehlerBezug            "Zählerstand Einspeisung [%.1f Wh]"
Number    TQManager_ZaehlerEinspeisung      "Zählerstand Einspeisung [%.1f Wh]"
Number    TQManager_Bezug                   "aktueller Bezug [%.1f W]"
Number    TQManager_Einspeisung             "aktuelle Einspeisung [%.1f W]"
```

**Rule**:  

```
rule "Convert bcontrol JSON to Item Type Number"
  when
    Item TQManager_DatenString changed
 then
    var String newValue = "0"

    // Aktueller Bezug 1.4.0 auslesen und Number Item zuordnen
    newValue = transform("JSONPATH", "['1-0:1.4.0*255']", TQManager_DatenString.state.toString)
    // post the new value to the Number Item
    TQManager_Bezug.postUpdate( newValue )

    // Aktuelle Einspeisung 2.4.0 auslesen und Number Item zuordnen
    newValue = transform("JSONPATH", "['1-0:2.4.0*255']", TQManager_DatenString.state.toString)
    // post the new value to the Number Item
    TQManager_Einspeisung.postUpdate( newValue )

    // Zähler 1.8.0 Bezug auslesen und Number Item zuordnen
    newValue = transform("JSONPATH", "['1-0:1.8.0*255']", TQManager_DatenString.state.toString)
    // post the new value to the Number Item
    TQManager_ZaehlerBezug.postUpdate( newValue )

    // Zähler 2.8.0 Einspeisung auslesen und Number Item zuordnen
    newValue = transform("JSONPATH", "['1-0:2.8.0*255']", TQManager_DatenString.state.toString)
    // post the new value to the Number Item
    TQManager_ZaehlerEinspeisung.postUpdate( newValue )
 end

```
