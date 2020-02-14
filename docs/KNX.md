KNX-Binding
===========

Im folgenden Kapitel wird die Anbindung an den KNX-Bus per KNX/IP Gateway erklärt.
Für die Konfiguration wird das [Beispielhaus](#Beispiel--Visualisierung-Haus) herangezogen sowie die Einrichtung des KNX-Binding per textbasierte Konfiguration erklärt.
Um auf den KNX-Bus zuzugreifen, wird ein an den KNX-Bus angeschlossenes KNX-Gateway benötigt (z. B.: Siemens IP-Schnittstelle N148/22).
Das KNX Verbindung kann dabei sowohl mit einem Ethernet (Router- oder Tunneltyp) als auch als serielles Gateway realisiert werden.
Auf die Konfiguration als serielles Gateway wird nicht eingegangen.

Vorbereitungen
--------------

Nach erfolgter Installation des KNX/IP Gateways, muss dessen IP-Adresse
bekannt sein. Hier empfiehlt es sich dem KNX/IP Gateway eine feste IP-Adresse zu vergeben.
Zusätzlich muss die IP-Adresse des Systems auf dem openHAB läuft bekannt
sein. Diese kann unter Windows mit ipconfig bzw. unter Linux
mit ifconfig ermittelt werden. Nachdem die IP-Adressen bekannt sind,
kann mit der Installation und dem Einrichten des KNX-Binding begonnen werden.  

Die Bindings werden am einfachstens über da *Paper UI* installiert.
Zu Installation des KNX-Binding wird das *Paper UI* geöffnet und links in der Liste Add-ons ausgewählt.
Danach kann in der Rubrik *Bindings* das KNX-Binding ausgewählt und installiert werden.

Im Kopf der Datei *knx.things* sind zur Konfiguration des KNX/IP Gateway folgende Zeilen anzupassen oder einzufügen:

  * **type** = <TUNNEL oder ROUTER \>  
    (Parameter ist erforderlich! / Der ROUTER Modus läuft nicht auf allen Geräten)

  * **ipAddress** = <IP Adresse des KNX Gateway \>

  * **localIp** = <IP Adresse des PC auf dem die openhab-runtime läuft \>

Bei dem Auszug der folgenden Beispiel knx.things, läuft openHAB auf
einem PC mit der IP-Adresse 192.168.1.50, das KNX/IP Gateway mit der
IP-Adresse 192.168.1.100. Die Kommunikation wird im Tunnel-Modus
aufgebaut.

    Bridge knx:ip:bridge [  
        type="TUNNEL", 
        ipAddress="192.168.1.100", 
        portNumber=3671, 
        localIp="192.168.1.50",
        readingPause=50, 
        responseTimeout=10, 
        readRetriesLimit=3, 
        autoReconnectPeriod=60,
        localSourceAddr="0.0.0"
    ]
	
Zusätzlich zur Gateway Konfiguration ist noch das *device Thing* zu definieren. Dies geschieht im Abschnitt *Thing* der Datei *knx.things*.
Bei den *Things* handelt es sich um einen Container für beliebige Gruppenadressen des KNX-Buses.
Die einzige Aufgabe des *Thing* ist es, die definierten Gruppenadressen auf dem KNX-Bus abzufragen um sicherzustellen, dass KNX-Aktor erreichbar ist.

Im Abschnitt Thing können die optionalen Parameter *address*, *fetch*, *pingInterval* und *readInterval* definiert werden.  

  * **address** = < individuelle Geräteadresse \> in 0.0.0-Notation

  * **fetch** = < Geräteparameter und Adress- und Kommunikationsobjekttabellen auslesen \>  
    (erfordert den Parameter *address* / Standardwert ist "false")

  * **pingInterval** = < Intervall in Sekunden zur Geräteabfrage und setzen des *Thing*-Status abhängig vom Ergebnis der Rückmeldung \>   (erfordert den Parameter address / Standardwert ist 600)

  * **pingInterval** = < Intervall in Sekunden zur aktiven Leseaufforderung von Werten auf dem Bus \>  
    (0, wenn sie nur einmal beim Start gelesen werden sollen / erfordert den Parameter address / Standardwert ist 0)

Der Abschnitt *device Thing* sieht beispielsweise wie folgt aus.
 
    Thing device generic [
        address="1.2.3",
        fetch=true,
        pingInterval=300,
        readInterval=0
    ]
	
Abschließend werden die KNX Gruppenadressen noch entsprechenden Kanälen zugeordnet.
Anhand eines Lichts und Dimmer aus dem [Beispielhaus](#Beispiel--Visualisierung-Haus) wird die Zuordnung der Gruppenadressen veranschaulicht.
Weitere mögliche Kanäle und eine detailliertere Beschreibung ist der englischen Beschreibung des KNX-Binding zu entnehmen (https://www.openhab.org/addons/bindings/knx/).  

* * * * *
![Hinweis!](images/Warning.png "Hinweis! Konfiguration Bindings in der openhab.cfg")
Nach Änderungen der DPT von bereits bestehenden Kanälen muss openHAB neu  
gestartet werden, damit die Änderungen wirksam werden.  
* * * * *




### Licht schalten

Als Beispiel wird das Licht des Spiegelschrank aus dem Bad des [Beispielhaus](#Beispiel--Visualisierung-Haus)verwendet.
Das Licht des Spiegelschrankes im Obergeschoss verwendet zum Schalten die Gruppenadresse 2/2/1 und 2/2/2 für den Status. Für Schalter wird der Kanal "switch" verwendet. 

Die Definition sieht dann wie folgt aus:
    
	Type switch : Licht_OG_Badspiegel	          "Licht Badspiegel"        [ ga="1.001:2/2/1" ]

### Licht dimmen

Als Beispiel wird das Licht im Bad aus dem aus dem OG des [Beispielhaus](#Beispiel--Visualisierung-Haus)verwendet. Das Licht im Bad lässt sich dimmen und verwendet hierfür folgende Gruppenadressen 2/1/40 (schalten), 2/1/41 (status bit), 2/1/44, 2/1/42 und 2/1/43.
Zum Dimmen wird der Kanal "dimmer" verwendet. 

Die Definition sieht dann wie folgt aus:
    
	Type dimmer      : Dimmer_OG_Bad		  "Licht Bad"	            [ switch="2/1/40+2/1/41", position="2/1/44+2/1/42", increaseDecrease="2/1/43" ]

Die komplette *knx.things* Datei sieht dann wie folgt aus:

    Bridge knx:ip:bridge [  
        type="TUNNEL", 
        ipAddress="192.168.1.100", 
        portNumber=3671, 
        localIp="192.168.1.50",
        readingPause=50, 
        responseTimeout=10, 
        readRetriesLimit=3, 
        autoReconnectPeriod=60,
        localSourceAddr="0.0.0"
    ] {
	    Thing device generic [
            address="1.2.3",
            fetch=true,
            pingInterval=300,
            readInterval=0
        ] {
            Type switch      : Licht_OG_Bad	          "Licht Badspiegel"        [ ga="1.001:2/2/1" ]
            Type dimmer      : Dimmer_OG_Bad		  "Licht Bad"	            [ switch="2/1/40+2/1/41", position="2/1/44+2/1/42", increaseDecrease="2/1/43" ]
	    }
	}

Alle Voraussetzungen für einen erfolgreichen Verbindungsaufbau zwischen dem KNX-Bus und openHAB sind nun gegeben.  

Nach erfolgter Konfiguration des KNX-Binding, müssen die Kanäle nun noch den Items zugeordnet werden. Am Beispiel der Item-Datei aus dem [Beispielhaus](#Beispiel--Visualisierung-Haus) ist diese wie folgt zu ändern:  

        Switch Licht_OG_Bad_Spiegel   "Licht Spiegelschrank"	(OG_Bad) { channel="knx:device:bridge:generic:Licht_OG_Bad" }
		Dimmer Licht_OG_Bad			  "Licht Bad"				(OG_Bad) { channel="knx:device:bridge:generic:Dimmer_OG_Bad" }
