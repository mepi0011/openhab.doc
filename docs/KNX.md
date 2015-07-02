Anbindung von KNX an openHAB
============================

Im folgenden Kapitel wird erklärt wie openHAB an den KNX-Bus angebunden werden kann. Dabei wird das [Beispielhaus](#Beispiel--Visualisierung-Haus) herangezogen. Zusätzlich wird für die Kommunikation über TCP/IP zum KNX-Bus ein KNX/IP Gateway benötigt (z.B.: Siemens IP-Schnittstelle N148/22).

Vorbereitungen
--------------

Nach erfolgter Installation des KNX/IP Gateways, muss dessen IP-Adresse
bekannt sein. Hier empfiehlt es sich eine feste IP-Adresse zu vergeben.
Zusätzlich muss die IP-Adresse des System auf dem openHAB läuft bekannt
sein. Dies kann der Konsole unter Windows mit ipconfig bzw. unter Linux
mit ifconfig ermittelt werden. Nachdem die IP-Adressen bekannt sind,
kann mit dem Einrichten von openHAB begonnen werden.

Als erstes kopieren wir das Binding org.openhab.binding.knx-\*.jar in
das Verzeichnis <Pfad_zu_openHAB>/runtime/addons. Das Herunterladen
der Bindings ist in Kapitel [Bindings/Addons herunterladen](#BINDING-Addons-herunterladen) beschrieben.  

Nun müssen noch folgende Zeilen in der Konfigurationsdatei openhab.cfg
angepasst werden.

-   knx:ip=\<IP Adresse des KNX Gateway\>

-   knx:type=TUNNEL (ROUTER Modus läuft nicht auf allen Geräten)

-   knx.localIp=\<IP Adresse des PC auf dem die openhab-runtime läuft\>

Bei dem Auszug der folgenden Beispiel openhab.cfg, läuft openHAB auf
einem PC mit der IP-Adresse 192.168.1.50, das KNX/IP Gateway mit der
IP-Adresse 192.168.1.100. Die Kommunikation wird im Tunnel-Modus
aufgebaut.

    ######################### KNX Binding #################################
    #
    # KNX gateway IP address
    # (optional, if serialPort or connection type 'ROUTER' is specified)
    knx:ip=192.168.1.100

    # KNX IP connection type. Could be either TUNNEL or ROUTER
    # (optional, defaults to TUNNEL)
    # Note: If you cannot get the ROUTER mode working
    # (even if it claims it is connected),
    # use TUNNEL mode instead with setting both the ip of the KNX gateway
    # and the localIp.
    knx:type=TUNNEL

    # KNX gateway port (optional, defaults to 3671)
    #knx:port=

    # Local endpoint to specify the multicast interface, no port is used
    # (optional)
    knx:localIp=192.168.1.50

    # Serial port of FT1.2 KNX interface (ignored, if ip is specified)
    # Valid values are e.g.
    # COM1 for Windows and /dev/ttyS0 or /dev/ttyUSB0 for Linux
    #knx:serialPort=

    # Pause in milliseconds between two read requests on the KNX bus during
    # initialization (optional, defaults to 50)
    #knx:pause=

    # Timeout in milliseconds to wait for a response from the KNX bus
    # (optional, defaults to 10000)
    #knx:timeout

    # Number of read retries while initialization items from the KNX bus
    # (optional, defaults to 3)
    #knx:readRetries

    # Seconds between connect retries when KNX link has been lost
    # 0 means never retry, it will only reconnect on next write
    # or read request
    # Note: without periodic retries all events will be lost up
    #       to the next read/write request
    # (optional, default is 0)
    #knx:autoReconnectPeriod=30


* * * * *
![Hinweis!](images/Warning.png "Hinweis! Konfiguration Bindings in der openhab.cfg")
Wichtig ist, dass die Zeilen durch entfernen des voranstehenden Zeichen
\# aktiviert werden!

* * * * *

Alle Voraussetzungen für einen erfolgreichen Verbindungsaufbau zwischen dem KNX-Bus und openHAB sind nun gegeben. Die Items können nun den Gruppenadressen (GA) zugeordnet werden.

### Licht schalten

Nach erfolgter Konfiguration des KNX-Binding, können nun die Items den
Gruppenadressen zugeordnet werden. Das folgende Beispiel verdeutlicht
dies an einem Licht. Hierfür wird das Beispiel-Haus aus Kapitel
[sec:BeispielHaus] verwendet. Das Licht des Kinderzimmer 1 im
Obergeschoss, soll mit dem zugehörigen Item verbunden und mit openHAB
geschaltet werden können. Das Licht im Kinderzimmer hat die
Gruppenadresse 2/2/1.\

Die Item-Datei aus den Beispiel wird wie folgt abgeändert:

    /*Licht*/
    Switch Licht_OG_Kinderzimmer1	"Licht Kinderzimmer1"	(OG_Kinderzimmer1)	{knx="2/2/1"}
