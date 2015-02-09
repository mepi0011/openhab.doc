Erste Schritte mit openHAB
==========================

Die grundlegende Funktionsweise von openHAB, wird im folgenden ein kleines Beispiel Schritt für Schritt erklärt.
Der [Code](https://github.com/mepi0011/openhab.doc/raw/master/examples/house.zip "Souce Code für Beispielhaus ohne KNX Erweiterung") für dieses Beispiel kann heruntergeladen werden.
Für das Beispiel wird eine Gebäudestruktur eines Einfamilienhauses verwendet, das aus
zwei Stockwerken und 12 Räumen besteht [siehe Abbildung [Grundriss]].
Mit Hilfe der Gebäudestruktur wird im folgenden der Aufbau von Items,
Gruppen und der Sitemap veranschaulicht.\
Im Beispielhaus befindet sich in jedem Raum mindestens 1 KNX-Schalter,
die Leuchten werden entweder über KNX-Schaltaktoren oder KNX-Dimmer
angesteuert. Erst Kapitel Anbindung an den KNX-Bus wird auf die KNX Eweiterung eingegangen und ist für das folgende Beispiel nicht relevant.

![Grundriss EG Beispielhaus](images/grundriss_erdgeschoss.png "Grundriss vom EG des Beispielhaus")

![Grundriss OG Beispielhaus](images/grundriss_obergeschoss.png "Grundriss vom OG des Beispielhaus")

In openHAB werden die Objekte zum Schalten oder Dimmen Items genannt und
in der Datei *.items definiert. Zudem können Items in Gruppen
zusammengefasst werden.

Beispiel: Visualisierung Haus
-----------------------------

Vor Beginn muss die Item-Datei, Sitemap-Datei und openHAB
Konfigurationsdatei openhab.cfg angelegt werden. Beim ersten Start des
openHAB Designer muss dieser ebenfalls einrichten werden.

Als erstes wird mit Hilfe eines gewöhnlichen Texteditors die Item-Datei
im Verzeichnis <Pfad_zu_openHAB>/configurations/items angelegt. In
unserem Beispiel nennen wir diese Haus.items. In ihr werden zukünftig
alle benötigen Items und Gruppen definiert.

* * * * *
![Hinweis!](images/Warning.png "Hinweis: Auf Dateierweiterung achten!")
Beim Anlegen der Datei Haus.items darauf achten, dass die
Dateierweiterung nicht .txt ist! Bei inkorrekter Dateierweiterung kann
es zu Fehlern bei openHab bzw. beim openHab-Designer kommen.

* * * * *

Im Verzeichnis <Pfad_zu_openHAB>/configurations muss eine kopie der
Datei openhab_default.cfg angelegt und in openhab.cfg umbenennen
werden. Ohne diese Datei gibt openHAB eine Fehlermeldung aus! Die
Einstellungen dieser Datei sind für das aktuell Beispiel noch nicht relevant. Der Designer kann
nun gestartet werden. Je nach Betriebssystem und Rechenleistung kann
dies einige Zeit dauern. Nach erfolgreichem Start, muss dem Designer der
Pfad zum Verzeichnis „configurations“ von openHAB mitgeteilt werden. Hierzu das Ordner Symbol drücken
und anschließend im angezeigten Dialog das Verzeichnis auswählen in dem
sich die openhab.cfg Datei befindet und mit OK bestätigen. In unserem
Fall ist dies <Pfad_zu_openHAB>/configrurations.

![Konfiguration des openHAB Designer](images/openHAB_Designer_Configuration.png "Konfiguration des openHAB Designer")

Danach sollte wie in folgender Abbildung der Reiter „configurations“ dargestellt werden. Die Vorbereitungen sind nun
abgeschlossen und es kann mit dem Beispiel begonnen werden.

![Auswahl des Item-File](images/openHAB_Designer_Item_File.png "Auswahl des Item-File")

Im Designer öffnen wir wie in der Abbildung oben zu sehen ist mit einem doppelklick der linken Maustaste die Datei Haus.items.
Die aktuell noch leere Datei erscheint nun in der Mitte des Designer. Nun können wir mit dem Anlegen der Gruppen und Items beginnen. Eine Gruppen Item Definition ist in openHAB wie folgt aufgebaut:

![Item Befehlsreferenz](images/openHAB_Befehlsreferenz.png)


In unserer Visualisierung sollen die Räume in das Erd- bzw. Obergeschoss
gegliedert werden. Hierzu legen wir die Gruppe gEG und gOG an und tragen
folgendes in die Item.datei ein:   

    // Gruppen
    Group gEG 	// Gruppe Erdgeschoss
    Group gOG 	// Gruppe Obergeschoss

Die beiden Gruppen bestehen aus dem Schlüsselwort Group und ihrem Namen. Mehr wird hierfür aktuell nicht benötigt.

Wie in Programmiersprachen auch, kennt openHAB Kommentare. Diese können
durch // für eine Zeile bzw. für mehrere Zeilen durch \\* am Anfang und
*/ am Ende eingeleitet werden.

Nun müssen noch die Räume definieren werden. Zuvor wird für jeden Raum eine Gruppe definieren, in dem die eigentlichen Items (Schalter, Sensoren und Aktoren) zugeordnet werden.   
Die Item-Datei erweitern wir mit folgendem Code:

    Group OG_Schlafzimmer	"Schlafzimmer"	<bedroom>	(gOG)
    Group OG_Kinderzimmer1	"Kinderzimmer1"	<boy1>		(gOG)
    Group OG_Kinderzimmer2	"Kinderzimmer2"	<child2>	(gOG)
    Group OG_Bad		"Bad"		<bath>		(gOG)
    Group OG_Flur		"Flur OG"			(gOG)

    Group EG_Kueche		"Kueche" 	<kitchen>	(gEG)
    Group EG_Esszimmer		"Esszimmer" 	<child1> 	(gEG)
    Group EG_WC			"WC"		<bath>		(gEG)
    Group EG_Wohnzimmer		"Wohnzimmer"	<sofa>		(gEG)
    Group EG_Abstellraum	"Abstellraum"	<wardrobe2>	(gEG)
    Group EG_Flur		"Flur EG"			(gEG)

Mit Hilfe des Codes und der Abbildung "Befehlsreferenz" sollte der Aufbau verständlich werden.
Items bestehen immer aus den Komponenten Funktion, Name, Text und Icon.

Zum Schluss werden noch die eingentlichen Funktionen der einzelnen
Räumen über Items definiert. Zu Beginn beschränken wir uns auf Schalter
und Dimmer und fügen folgendes in die Items-Datei ein:

    /*Licht*/
    Switch Licht_OG_Kinderzimmer1	"Licht Kinderzimmer1"	(OG_Kinderzimmer1)
    Switch Licht_OG_Kinderzimmer2	"Licht Kinderzimmer2"	(OG_Kinderzimmer2)
    Switch Licht_OG_Bad_Spiegel		"Licht Spiegelschrank"	(OG_Bad)
    Switch Licht_OG_Flur		"Licht Flur/Treppe OG"	(OG_Flur)
    Switch Licht_EG_Kueche		"Licht Kueche"		(EG_Kueche)
    Switch Licht_EG_Esszimmer		"Licht Esszimmer"	(EG_Esszimmer)
    Switch Licht_EG_WC			"Licht WC"		(EG_WC)
    Switch Licht_EG_Abstellraum		"Licht Abstellraum"	(EG_Abstellraum)
    Switch Licht_EG_Flur		"Licht Flur/Treppe EG"	(EG_Flur)

    /*Dimmer*/
    Dimmer Licht_OG_Schlafzimmer	"Licht Schlafzimmer"	(OG_Schlafzimmer)
    Dimmer Licht_OG_Bad			"Licht Bad"		(OG_Bad)
    Dimmer Licht_EG_Wohnzimmer		"Licht Wohnzimmer"	(EG_Wohnzimmer)

Auch hier ist der Aufbau der Items gleich, sie bestehen bestehen aus den
Komponenten Funktion, Name, Text und Gruppe. In diesem Beispiel wurden
kein Icons angegeben, da bei bestimmten Items wie zum Beispiel Switch,
Dimmer und Rollershutter ein Standard-Icon verwendet wird. Möchte man
dieses Standard-Icon nicht, so kann auch ein eigenes Icon angegeben
werden.

Die komplette Item-Datei sollte nun wie folgt aussehen:

    // Gruppen
    Group gEG 	// Gruppe Erdgeschoss
    Group gOG 	// Gruppe Obergeschoss

    Group OG_Schlafzimmer	"Schlafzimmer"	<bedroom>	(gOG)
    Group OG_Kinderzimmer1	"Kinderzimmer1"	<boy1>		(gOG)
    Group OG_Kinderzimmer2	"Kinderzimmer2"	<child2>	(gOG)
    Group OG_Bad		"Bad"		<bath>		(gOG)
    Group OG_Flur		"Flur OG"			(gOG)

    Group EG_Kueche		"Kueche" 	<kitchen>	(gEG)
    Group EG_Esszimmer		"Esszimmer" 	<child1> 	(gEG)
    Group EG_WC			"WC"		<bath>		(gEG)
    Group EG_Wohnzimmer		"Wohnzimmer"	<sofa>		(gEG)
    Group EG_Abstellraum	"Abstellraum"	<wardrobe2>	(gEG)
    Group EG_Flur		"Flur EG"			(gEG)

    /*Licht*/
    Switch Licht_OG_Kinderzimmer1	"Licht Kinderzimmer1"	(OG_Kinderzimmer1)
    Switch Licht_OG_Kinderzimmer2	"Licht Kinderzimmer2"	(OG_Kinderzimmer2)
    Switch Licht_OG_Bad_Spiegel		"Licht Spiegelschrank"	(OG_Bad)
    Switch Licht_OG_Flur		"Licht Flur/Treppe OG"	(OG_Flur)
    Switch Licht_EG_Kueche		"Licht Kueche"		(EG_Kueche)
    Switch Licht_EG_Esszimmer		"Licht Esszimmer"	(EG_Esszimmer)
    Switch Licht_EG_WC			"Licht WC"		(EG_WC)
    Switch Licht_EG_Abstellraum		"Licht Abstellraum"	(EG_Abstellraum)
    Switch Licht_EG_Flur		"Licht Flur/Treppe EG"	(EG_Flur)

    /*Dimmer*/
    Dimmer Licht_OG_Schlafzimmer	"Licht Schlafzimmer"	(OG_Schlafzimmer)
    Dimmer Licht_OG_Bad			"Licht Bad"		(OG_Bad)
    Dimmer Licht_EG_Wohnzimmer		"Licht Wohnzimmer"	(EG_Wohnzimmer)

Damit das Beispiel noch lauffähig wird, benötigt openHAB noch die
Sitemap-Datei. In ihr wird der Aufbau des Weboberfläche definiert. Im
Ordner sitemaps wird eine Datei Haus.Sitemap angelegt und folgendes
eingetragen.

    sitemap Haus label="Hauptmenue"
    {
     Frame {
            Group item=gOG	label="Obergeschoss"	icon="firstfloor"
            Group item=gEG	label="Erdgeschoss"	icon="groundfloor"
           }
    }

Nachdem nun die drei wichtigsten Dateien angelegt wurden, kann das
Beispiel gestartet werden. Hierzu öffnen wir die Konsole (Linux) bzw.
die Dos-Box (Windows) und wechseln mit dem Befehl   
    cd <Pfad_zu_openHAB>/runtime (Linux)
oder
    dir <Pfad_zu_openHAB>\runtime (Windows)
in das Verzeichnis von openHAB.
Anschließend wird openHAB mit einem der folgenden Befehl gestartet.
    ./start.sh (Linux)
oder
    start.bat (Windows)

Nach erfolgreichem Start öffnen wir einen Browser und geben folgendes
ein:   
http://localhost:8080/openhab.app?sitemap=Haus\#   
Bei manchen Browsern (z.B. Firefox) gibt es Probleme mit der Darstelung. Als Browser kann Chronium bzw. Chrome empfohlen werden.   
Das Ergebniss des Ersten Beispiel sieht wie folgt aus:

![openHAB Ausgabe: Hauptmenü des Beispielhaus](images/Haus_Hauptmenue_klein.png "Hauptmenü des Beispielhaus")

![openHAB Ausgabe: OG des Beispielhaus](images/Haus_OG.png "OG des Beispielhaus")
![openHAB Ausgabe: EG Beispiel Haus](images/Haus_EG.png "UG des Beispielhaus")

OpenHAB kann durch die Tastenkombination Strg+C beendet werden.


Erweiterung des Beispielhaus
============================

Im vorherigen [Beispiel](#Beispiel--Visualisierung-Haus) wurden die Items Switch und Dimmer sowie der Grundlegende Aufbau von Gruppen erläutert. Nun soll das Beispiel um weitere Items und Gruppen ergänzt werden.

Contact Item
------------
Mit hilfe des Contact-Item können die Zustände offen und geschlossen angezeigt werden, vorzugsweise für Fenster- oder Türkontakte.

Im Beispiel ergänzen wir nun in der Items-Datei Haus.items um folgene Zeilen um z.B. den Status des Küchenfenster sowie der Haustür abzufragen. 

Code und Bild müssen noch eingefügt werden!