HTTP Binding
============

Das HTTP Binding ist wie z.B. das KNX Binding als extra Modul im
Zip-File Addons enthalten. Um mit openHAB Anfragen an eine URL zu senden
wenn bestimmte Ereignuisse erfolgen oder um Zyklisch Daten einer URL
abzufragen, muss das HTTP Binding in das Verzeichnis
<Pfad_zu_openHAB>/addons kopierte und die openHAB Konfiguration
angepasst werden. Im folgenden wird dies detailliert erklärt.

Item Binding Konfiguration
--------------------------

Um ein Item mit einer HTTP Anfrage zu koppeln, muss dies konfiguriert
werden. Am einfachsten geht dies in dem man die HTTP Binding
Informationen in die Item Datei (im Verzeichnis configuration/items)
einträgt. Die Schreibweise für die Konfiguration wird unterteilt ob
Daten an die URL gesendet oder empfangen werden:

Eingabe: `http:"<[<url>:<refreshintervalinmilliseconds>:<transformationrule>]"`   
Ausgabe: `http:">[<command>:<httpmethod>:<url>]"`  

Für die Ausgabe sind zwei spezielle Komandos verfügbar:  

’*’ - Bedeutet, die URL wird aufgerufen unabhängig vom Kommando  
’CHANGED’ - Bedeutet, die URL wird aufgerufen bei Änderung des
Item-Status

Beispiele für zulässige Binding Konfigurationen:

    http="<[http://www.domain.org/weather/openhabcity/daily:60000:REGEX(.*?<title>(.*?)</title>.*)]"
    http=">[ON:POST:http://www.domain.org/home/lights/23871/?status=on&type=\"text\"] >[OFF:POST:http://www.domain.org/home/lights/23871/?status=off]"
    http=">[ON:POST:http://www.domain.org/home/lights/23871/?status=on&type=\"text\"] >[OFF:POST:http://www.domain.org/home/lights/23871/?status=off]"
    http=">[*:POST:http://www.domain.org/home/lights/23871/?status=\%2\$s&type=\"text\"] <[http://www.domain.org/weather/openhabcity/daily:60000:REGEX(.*?<title>(.*?)</title>.*)]"
    http=">[CHANGED:POST:http://www.domain.org/home/lights/23871/?status=\%2$s&date=\%1$tY-\%1$tm-\%1$td]"

Als Resultat sollten Items in ihrem Items-File so aussehen:

    Number Weather_Temperature "Temperature [\%.1f ¬8C]"  <temperature>  (Weather) { http="<[http://weather.yahooapis.com/forecastrss?w=638242&u=c:60000:XSLT(demo_yahoo_weather.xsl)]" }

Transformation rules
--------------------

OpenHAB unterstützt verschiedene Typen der Transformation der Ergebnisse, die von der
URL zurückgegebenen werden.

JSON Handhabung
---------------

Die JavaScript Object Notation, kurz JSON, ist ein kompaktes Datenformat
in einer einfach lesbaren Textform zum Zweck des Datenaustauschs
zwischen Anwendungen oder Geräten. Eine Einführung zum Aufbau der
Struktur kann auf der JSON Homepage (http://json.org/json-de.html)
nachgelesen werden  

Voraussetzung für die Handhabung von JSON in openHAB ist, dass die
Aufgerufene Seite ein JSON Format ausgibt. Im Folgenden Beispiel gibt
der Aufruf von “http://10.90.30.100:8080/tv/getTuned” folgende JSON
Struktur zurück:  

    { "callsign": "KSBW",
       "date": "20131126",
       "duration": 3600,
       "isOffAir": false,
       "isPclocked": 3,
       "isPpv": false,
       "isRecording": false,
       "isVod": false,
       "major": 8,
       "minor": 65535,
       "offset": 3157,
       "programId": "11269591",
       "rating": "No Rating",
       "startTime": 1385485200,
       "stationId": 3910459,
       "status": {     "code": 200,     "commandResult": 0,     "msg": "OK.",
       "query": "/tv/getTuned"   },
       "title": "Today"
     }

Es soll der titel ausgelesen werden, hierzu wird das Item wie folgt
definiert werden:

    String DirecTV1_Channel "Aktueller Kanal" { http="<[http://10.90.30.100:8080/tv/getTuned:30000:JS(getValue.js)]" }

Wichtig für die fehlerfreie Funktion ist, dass noch eine Datei **getValue.js** im Verzeichnis <Pfad_zu_openHAB>/configuration/transform/ angelegt wird. Der Dateinamen kann dabei frei gewählt werden, wichtig dabei ist, dass dieser gleich ist wie im zugehörigen Item.

In der Erstellten Datei **getValue.js** wird nun definiert welches Name/Wert Paar ausgelesen wird. Der Inhalt dieser Datei besteht aus der Zeile **JSON.parse(input).\<NameWert Paar\>;**   
In unserem Beispiel ist dies title: Today. Folglich wird folgender
Inhalt in der Datei **getValue.js** stehen:

    JSON.parse(input).title;

Seiten zwischenspeichern (Caching)
----------------------------------

Seit openHAB 1.3 unterstützt das HTTP binding die möglich bis zu zwei verschiedene Seiten zwischen zu speichern. Damit können mehrere Items die enpfangenen Daten auswerten, ohne die Seite nochmals abzurufen.

Hierzu muss lediglich in der openHAB Konfigurationsdatei openhab.cfg der Abschnitt HTTP Binding angepasst werden.

    % ############################### HTTP Binding ##########################################
    % # URL of the first cache item
    % # http:<cacheItemName1>.url=
    % # Update interval for first cache item
    %# http:<cacheItemName1>.updateInterval=
    
    % # URL of the second cache item
    % # http:<cacheItemName2>.url=
    % # Update interval for second cache item
    % # http:<cacheItemName2>.updateInterval=

