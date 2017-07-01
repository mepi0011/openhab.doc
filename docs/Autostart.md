Autostart von openHAB einrichten
================================

Das Linux-System auf dem Raspberry PI kann so eingerichtet werden, dass openHAB bei einem Neustart automatisch gestartet wird.
Voraussetzung für die korrekte Funktionsweise der folgenden Anleitung, ist eine Installation von openHAB nach den schritten der vorherigen Anleitung. Wurde openHAB nicht nach dieser Anleitung installiert, müssen die Pfade dementsprechend angepasst werden.   Anhängig vom verwendeten Init-System welches das installierte Linux verwendet, muss einer der beiden Lösungen verwendet werden. Mit dem Befehl `cat /proc/1/comm` kann herausgefunden werden, welches Init-System die installierte Linux Version verwendet. 

Autostart mit systemd
----------------------

**(bei aktuellen Linux Versionen z. B. Raspbian Jessy)**  

Damit openHAB beim Booten automatisch startet, werden die beiden Script-Dateien openhab und openhab.service benötigt.  
Die [Dateien](https://github.com/mepi0011/openhab.doc/raw/master/examples/autostart_systemd.zip "Script-Files für openHAB Autostart mit systemd") herunterladen, entpacken und anschließend per WINSCP oder USB Stick auf den Paspberry PI kopieren.   

Die Script-Datei openhab:  

    #! /bin/sh
    # Author: Pierre Metzner

    # set path to openhab
    OPENHAB_PATH=/opt/openhab

    # set ports for HTTP(S) server
    HTTP_PORT=8080
    HTTPS_PORT=8443

    # change directory to openHAB runtime
    cd $OPENHAB_PATH

    # get path to equinox jar inside $OPENHAB_PATH folder
    cp=$(find ./server -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);

    # starting openHAB
    echo "Launching the openHAB runtime.."
    java -Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTP_PORT -Djetty.port.ssl=$HTTPS_PORT -Djetty.home=. -Dlogback.configurationFile=configurations/logback.xml -Dfelix.fileinstall.dir=addons -Djava.library.path=lib -Djava.security.auth.login.config=./etc/login.conf -Dorg.quartz.properties=./etc/quartz.properties -Dequinox.ds.block_timeout=240000 -Dequinox.scr.waitTimeOnBlock=60000 -Djava.awt.headless=true -jar $cp -console &> /dev/null

    exit 0


Das Autostart-Script openhab.service:  

    [Unit]
    Description=A vendor and technology agnostic open source automation software for your smart home.
    After=network.target
    
    [Service]
    Type=forking
    ExecStart=/usr/bin/openhab
    
    [Install]
    WantedBy=multi-user.target


Nachdem die Dateien heruntergeladen und entpackt wurden, diese auf den Raspberry PI in folgende Verzeichnisse kopieren.
 
* Die Script-Datei openhab in das Verzeichnis /usr/bin kopieren
* Das Autostart-Script openhab.service in /etc/systemd/system kopieren

Um den Autostart einzurichten, müssen auf dem Raspberry noch folgende Schritte durchgeführt werden:
 
1. In das Verzeichnis wechseln in dem das Script openhab liegt / kopiert wird  
   `cd /usr/bin`

2. Die Dateieigenschaften ändern, damit diese ausgeführt werden kann  
   `sudo chmod a+x openhab`

3. Die Gruppe und den Besitzer der Datei ändern  
   `sudo chgrp root openhab`  
   `sudo chown root openhab`

4. Überprüfen der Änderungen  
   `ls -l`  
   Die Ausgabe müsste wie folgt aussehen:  
   `-rwxr-xr-x 1 root root 1757 Apr 16 23:27 openhab`

5. Sind die Schritte 1-4 erfolgt, kann mit folgenden Befehlen openHAB gestartet und beendet werden:  
   `sudo service openhab start` gestartet  
   `sudo service openhab stop` gestoppt und mit  
   `sudo service openhab status` der Status abgefragt werden
 
6. Abschließend muss noch das System angewiesen werden, openhab beim booten zu starten  
   `sudo systemctl enable openhab.service`

Weiterführenden Links:
-  https://wiki.ubuntuusers.de/Dienste/
-  https://wiki.ubuntuusers.de/systemd/



Autostart mit SysVinit
----------------------

**(bei älteren Linux Versionen z. B. Raspbian Wheezy oder Debia Wheety)**  

Damit openHAB beim Booten automatisch startet, werden die beiden Script-Dateien openhab und openhab.cfg benötigt.  
Hierzu einfach die [Dateien](https://github.com/mepi0011/openhab.doc/raw/master/examples/autostart.zip "Script-Files für openHAB Autostart") herunterladen, entpacken und anschließend per WINSCP oder per USB Stick auf den Paspberry PI kopieren.   

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
