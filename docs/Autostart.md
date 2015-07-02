Autostart von openHAB einrichten
================================

Das Linux-System auf dem Raspberry PI kann so eingerichtet werden, dass openHAB bei einem Neustart automatisch gestartet wird.
Voraussetzung für die korrekte Funktionsweise der folgenden Anleitung, ist eine Installation von openHAB nach den schritten der vorherigen Anleitung. Wurde openHAB nicht nach dieser Anleitung installiert, müssen die Pfade dementsprechend angepasst werden.   
Damit openHAB beim Booten automatisch startet, werden die beiden Script-Dateien openhab und openhab.cfg benötigt.  
Hierzu einfach die [Dateien](https://github.com/mepi0011/openhab.doc/raw/master/examples/autostart.zip "Script-Files für openHAB Autostart") herunterladen, entpacken und anschliesend per WINSCP oder per USB Stick auf den Paspberry PI kopieren.   

Die Script-Datei openhab:  

    #! /bin/sh
    ### BEGIN INIT INFO
    # Provides:          starts openhab from home
    # Required-Start:    $local_fs $network $named $portmap $remote_fs $syslog $time
    # Required-Stop:     $local_fs $network $named $portmap $remote_fs $syslog $time
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Kurze Beschreibung
    # Description:       Längere Bechreibung
    ### END INIT INFO
    # Author:
    
    # set ports for HTTP(S) server
    HTTP_PORT=8080
    HTTPS_PORT=8443
    
    if test -f /etc/default/openhab.conf; then
        . /etc/default/openhab.conf
    else
        echo "Please set OPENHABPATH in /etc/default/openhab.conf"
        exit 1
    fi
    
    # Aktionen
    case "$1" in
        start)
            if [ -f /var/run/openhab.pid ]; then
                    echo "openhab seems to run allready. If not, please delete /var/run/openhab.pid"
            else
    
                    cd $OPENHABPATH
                    # get path to equinox jar inside $OPENHABPATH folder
                    cp=$(find ./server -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);
    
                    echo Launching the openHAB runtime..
                    java -Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTP_PORT -Djetty.port.ssl=$HTTP$
    
                    echo $! > /var/run/openhab.pid
            fi
            ;;
        stop)
            echo "stopping openhab"
            kill `cat /var/run/openhab.pid`
            rm /var/run/openhab.pid
            ;;
        restart)
            echo "does not work"
            ;;
    esac
    
    exit 0 


Die Konfiguration-Datei openhab.cfg  

    # PATH TO OPENHAB
    OPENHABPATH=/opt/openhab
    
    # set ports for HTTP(S) server
    HTTP_PORT=8080
    HTTPS_PORT=8443

Nachdem die Dateien heruntergeladen und entpackt wurden, diese auf den Raspberry PI in folgende Verzeichnisse kopieren.
 
* Die Script-Datei openhab in das Verzeichnis /etc/init.d/ kopieren
* Die Konfigurationsdatei openhab.cfg in /etc/default kopieren

Um den Autostart einzurichten, müssen auf dem Raspberry nun folgende Schritte durchgeführt werden:
 
1. In das Verzeichnis wechseln in dem das Script openhab liegt / kopiert wird  
   `cd /etc/init.d`

2. Die Dateieigenschaften ändern, damit diese ausgeführt werden kann  
   `sudo chmod a+x openhab`

3. Die Gruppe und den Besitzer der Datei ändern  
   `sudo chgrp root openhab`  
   `sudo chown root openhab`

4. Überprüfen der Änderungen  
   `ls -l`  
   Die Ausgabe müsste wie folgt aussehen:  
   `-rwxr-xr-x 1 root root 1757 Apr 13 23:27 /etc/init.d/openhab`
   
5.  In das Verzeichnis wechseln in das die Konfigurationsdatei openhab.cfg kopiert wird  
    `ch /etc/default`

5.  Nachdem das Script ausführbar ist, muss dieses noch in die Runnnlevel eintragen werden.  
    Hierzu folgenden Befehl ausführen:  
    `sudo update-rc.d openhab defaults`

6.  Nun kann openHAB mit dem Befehl  
    `sudo /etc/init.d/openhab start` gestartet, bez mit `sudo /etc/init.d/openhab stop` gestoppt werden.
