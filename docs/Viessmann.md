Steuerung von Viessmann Heizungen
=================================

Installation der Software vcontrold und vclien
----------------------------------------------

Mit Hilfe des Projektes openv und dem Daemon Vcontrold ist es möglich
Viessmann Heizungen auszulesen und steuern. Die Folgende Dokumentation
ist zum Teil von der Homepage des openv Projektes entnommen (siehe
http://openv.wikispaces.com)

1.  Vor Beginn der eigentlichen Installation des Vcontrold Daemon müssen
    auf dem Raspbian-System die Pakete subversion, build-essential,
    libxml2-dev, automake und autoconf installiert werden. Die
    Installation geschieht mit folgendem Befehl:  
        sudo apt-get install subversion build-essential libxml2-dev automake autoconf

2.  Bevor der Quellcode von Vcontrold heruntergeladen werden kann, muss
    ein Ordner angelegt werden. Hier im Beispiel ist es der Unterordner “vcontrold” im aktuellen Verzeichniss  
        mkdir vcontrold

3.  Herunterladen der Sourcen erfolgt schließlich mit  
        svn checkout svn://svn.code.sf.net/p/vcontrold/code/ vcontrold

4.  In das Verzeichnis mit den Sourchen wechseln  
        cd vcontrold/vcontrold

5.  Die Datei auto-build.sh ausführbar machen  
        chmod +x auto-build.sh

6.  Nacheinander folgendes ausführen  
        ./auto-build.sh
        ./configure

7.  Wenn beides ohne Fehler gelaufen ist, noch folgenden Befehl
    ausführen um das ausführbaren Programme zu erstellen  
        make

8.  Die Installation wird abgeschlossen durch folgende Befehle  
        sudo make install

vcontrold einrichten und testen
-------------------------------

Nach erfolgreicher Installation von vcontrold muss dies noch
eingerichtet werden, da die Software nach den XML Dateien vcontrold.xml
und vito.xml sucht. In diesen Dateien sind die Heizungsspezifischen
Kommandos gespeichert. Eine Anleitung wie sich diese Dateien
zusammensetzen findet sich auf der Projektseite von openc. Im Internet
findet man auch Beispieldateien.

1.  Anlegen eines des Ordner vcontrold im Verzeichnis /etc  
    `sudo mkdir /etc/vcontrold`

2.  Die Dateien vcontrold.xml und vito.xml in das Verzeichnis /etc/vcontrold kopieren

3.  In der Datei vcontrold.xml muss noch der Port der RS232 und die ID
    der Heizungs eingestellt werden. Eine Übersicht der Heizungstypen Ids und Aufbau der Datei findet man unter
    http://openv.wikispaces.com  
    Der Verwendete Port wird im Feld <tty> eingerichtet und die ID der Heizungsanlage im Feld <device ID=“xxxx“> Auszug einer vcontrold.xml Datei für eine Vitocal300G:

4.  Nun kann der Dämon vcnontrold das erste mal gestartet werden  
    `sudo vcontrold`

5.  Nach erfolgreichen Start von vcontrold kann ein erste Verbindung/Abfrage zur Heizung erfolgen.  
    Mit dem folgenden Befehl wird die Außentemperatur abgefragt.  
    `vclient -h 127.0.0.1:3002 -c getTempA`

Autostart von vcontrold einrichten
----------------------------------

Damit vcontrold und vclient zusammen mit openHAB arbeiten können, muss
der Dämon beim Booten gestartet werden. Hierzu wird wieder ein
Startscript in /etc/init.d erstellt.

1.  Erstellen des vcontrold Start-Script mit folgendem Inhalt und dieses unter /etc/init.d speichern.

2.  In das Verzeichnis wechseln in dem das Script openhab liegt  
    `cd /etc/init.d`

3.  Die Datei ausführbar machen  
    `sudo chmod a+x vcontrold`

4.  Gruppe und Eigentümer der Datei ändern  
    `sudo chgrp root vcontrold`  
    `sudo chown root vcontrold`

5.  Überprüfen der Änderungen
    `ls -l`
    Die Ausgabe müsste wie folgt aussehen:
    `-rwxr-xr-x 1 root root 1757 Apr 13 23:27 /etc/init.d/vcontrold`

6.  Nachdem das Script ausführbar ist, diesen noch in die Runnnlevel eintragen  
    `sudo update-rc.d vcontrold defaults`

7.  Nun kann openHAB mit dem Befehl  
    `sudo /etc/init.d/vcontrold start`  
    gestartet, bez mit  
    `sudo /etc/init.d/vcontrold stop`  
    gestoppt werden.


PS: Der Autostart kann auch mit `sudo update-rc.d -n vcontrold remove` rückgängig gemacht werden.