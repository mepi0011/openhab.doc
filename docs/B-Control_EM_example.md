# Anbindung B-Control Manager EM 100, 210 oder EM300 an openHAB 
Example to read the wattage and electric energy from a B control Energy Manager EM100, EM210, EM300  

## Introduction
Der B-Control Energie Manager (EM) wird vorzugsweise in Verbindung mit einer Photovoltaik-Anlage verwendet um die Einspeise oder Bezugsleistung und der Energie zu messen. Auf der Homepage von B-Control wurde die Spezifikation des
[Modbus Protokolls](http://www.b-control.com/fileadmin/Webdata/b-control/Uploads/Energiemanagement_PDF/B-control_Energy_Manager_Modbus_Master.0100.pdf) (funktioniert nicht für EM 210) sowie die der
[JSON Schnittstelle](http://www.b-control.com/fileadmin/Webdata/b-control/Uploads/Energiemanagement_PDF/B-control_Energy_Manager_-_JSON-API_0100.pdf)
veröffentlicht. Mit Hilfe dieser Informationen ist es möglich dievers Informationen wie zum Beispiel die aktuelle Energie oder die Zählerstände des Energie manager auszulesen und in openHAB zu verarbeiten.

## Auslesen der Daten via JSON
Diese Lösung verwendet das exec Binding in Kombination mit einem Linux Bash Skript.
Um das Skript ausführen zu können, müssen die Programme curl and jq instaliert sein.
In dem meisten Linux Distributionen ist curl and JQ enthalten. Solte jq mit in den Paketquellen der verwendeten Distribution (z. B. raspbian) nicht enthalten sein, so kann es von der [Projekt Homepage](http://stedolan.github.io/jq/) heruntergeladen und [installiert](https://josef-friedrich.de/install-jq-on-raspberry-pi/) werden.  

### jq auf dem Raspberry installieren

1.  jq in das aktuelle Verzeichnis herunterladen  
    `curl -O http://stedolan.github.io/jq/download/source/jq-1.4.tar.gz`

2.  die heruntergeladene Datei entpacken  
    `tar xfvz jq-1.4.tar.gz`

3.  in das Verzeichnis wechseln in dem die entpackten Dateien liegen  
    `cd jq-1.4`

4.  Zur Installation folgende drei Befehle ausführen  
    ```
    ./configure
    make
    sudo make install
    ```

Das folgende Bash-Skript *bcontrol* liest die Daten über die B-Control Energie Manager JSON Schnittstelle und speichert die Ausgabe in die Datei *bcontrol.out* im Verzeichnis */opt/bcontrol*.  

Als erster Schritt wird der Ordner *bcontrol* im Verzeichnis */opt* angelegt!  

Anschließend die Datei *bcontrol* anlegen und den folgenden Inhalt hinein kopieren. Bitte nicht vergessen die Datei mit Hilfe von chmod in eine  ausführbare Datei zu setzen.

```
#!/bin/bash
# Script to get data from B-Control Manager over the JSON interface

# path to write JSON output
JsonPath="/opt/bcontrol"

# file name for JSON output
JsonFile="bcontrol.out"

# file Name cookie-file (output of curl)
CookieFile="cookie.txt"

# IP addresse of B-Control manager
BcontrolIP="192.168.1.12"

# build needed path and links
JsonOutput="${JsonPath}/${JsonFile}"
CookiePath="${JsonPath}/${CookieFile}"
JSONLink="http://${BcontrolIP}/mum-webservice/data.php"
LoginLink="${BcontrolIP}/start.php"

# get data (JSON) from B-Control Manager
curl -s -b $CookiePath -X GET $JSONLink > $JsonOutput

# check if login is pass
AuthError=$(/usr/local/bin/jq 'has("authentication")' $JsonOutput)

if [ $AuthError  = "true" ]
then
  echo "Login Error: Login will be repeat!"
  curl -s --cookie-jar $CookiePath $LoginLink > /dev/null
  curl -s -b $CookiePath --cookie-jar $CookiePath -d "login=<serial-no>&password=<password>&save_login=1" $LoginLink > /dev/null
  curl -s -b $CookiePath -X GET $JSONLink > $JsonOutput
fi

# Rueckgabe des gewuenschten Wertes
case "$1" in
	CurrentConsumption)	echo $(/usr/local/bin/jq '.["1-0:1.4.0*255"]' $JsonOutput);;
	CurrentFeeding)		echo $(/usr/local/bin/jq '.["1-0:2.4.0*255"]' $JsonOutput);;
	TotalConsumption)	echo $(/usr/local/bin/jq '.["1-0:1.8.0*255"]' $JsonOutput);;
	TotalFeeding)		echo $(/usr/local/bin/jq '.["1-0:2.8.0*255"]' $JsonOutput);;
	*)			echo "unknown parameter!"
				exit 1;;
esac

exit 0
```  

* * * * *
<tr>
<td> ![Hinweis!](images/Warning.png "Hinweis: Auf Dateierweiterung achten!") </td>
<td> Bitte ersetzen sie <serial-no> mit dem Benutzernamen der im B-Control Manager gesetzt ist (Normalerweise ist dies die Seriennummer), <password> mit dem Password des Benutzers (ist kein Passwort gesetzt, bitte dies frei lassen) und noch die IP-Adresse des B-Control Manager!</td>
</tr>
</table>
* * * * *

Nun kann das Skript vorab schon mal ausgeführt und korrekte Funktionsweise geprüft werden:  
```
/opt/TQManager/bcontrol TotalConsumption
```  

Wenn alles richtig ist, sollten sie einen gültigen Wert ausgegeben bekommen und das Skript kann nun in openHAB eingebunden werden.  

Um das Skript in openHAB zu verwenden, muss lediglich das exec Binding in das Verzeichnis addons kopiert werden und die folgenden Number-Items in ihre Items-Datei kopiert werden.  

```
Number BCM_consumption_wattage		"current wattage consumption [%.1f W]"	{exec="<[/opt/bcontrol/bcontrol@@CurrentConsumption:5000:REGEX((.*?))]"}
Number BCM_feeding_wattage		    "current feeding wattage [%.1f W]"	    {exec="<[/opt/bcontrol/bcontrol@@CurrentFeeding:5000:REGEX((.*?))]"}
Number BCM_energy_value_consumption	"energy value consumption [%.1f Wh]"	{exec="<[/opt/bcontrol/bcontrol@@TotalConsumption:10000:REGEX((.*?))]"}
Number BCM_energy_value_feeding		"energy value feeding [%.1f Wh]"	    {exec="<[/opt/bcontrol/bcontrol@@TotalFeeding:10000:REGEX((.*?))]"}
```
