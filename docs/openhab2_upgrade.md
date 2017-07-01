Upgrade auf openHAB2
====================

Das Kapitel erklät wie schrittweise von openHAB 1.x auf openHAB 2.x aktualisiert werden kann. Die Installation von openHAB2 wird nicht explizit beschrieben, es wir davon ausgegangen dass dies auf dem Zielsystem bereits läuft.


Install Oracle Java 8 Rev > 101
-------------------------------

* Raspbian Lite hat keine Java version vorinstalliert

Mit folgenden Befehlen Java nachinstallieren:

    echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/webupd8team-java.list

    echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee -a /etc/apt/sources.list.d/webupd8team-java.list

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886

    sudo apt-get update
    
    sudo apt-get install oracle-java8-installer

    sudo apt-get install oracle-java8-set-default
    
    reboot your System: sudo reboot
    
    check Java: java -version
    
    Output:
    
    java version "1.8.0_131"
    Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
    Java HotSpot(TM) Client VM (build 25.131-b11, mixed mode)

    
Quelle: http://docs.openhab.org/installation/linux.html



Instalation openHAB2 über paketquelle
-------------------------------------

First, add the openHAB 2 Bintray repository key to your package manager and allow Apt to use the HTTPS Protocol:

    wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
    sudo apt-get install apt-transport-https

Then, you can choose between, Official (Stable), Beta or Snapshot builds:

Stable Release

The stable builds contain the latest official release with tested features.

Add the openHAB 2 Stable Repository to your systems apt sources list:

    echo 'deb https://dl.bintray.com/openhab/apt-repo2 stable main' | sudo tee /etc/apt/sources.list.d/openhab2.list

Next, resynchronize the package index:

    sudo apt-get update

Now install openHAB with the following command:

    sudo apt-get install openhab2

When you choose to install an add-on, openHAB will download it from the internet on request. If you plan on disconnecting your machine from the internet, then you will want to also install the add-ons package.

    sudo apt-get install openhab2-addons


Autostart von openHAB
---------------------

pi@openHAB:~ $ sudo systemctl start openhab2.service
pi@openHAB:~ $ sudo systemctl status openhab2.service
● openhab2.service - openHAB 2 - empowering the smart home
   Loaded: loaded (/usr/lib/systemd/system/openhab2.service; disabled)
   Active: active (running) since Di 2017-06-27 20:45:07 CEST; 8s ago
     Docs: http://docs.openhab.org
           https://community.openhab.org
 Main PID: 831 (karaf)
   CGroup: /system.slice/openhab2.service
           ├─831 /bin/bash /usr/share/openhab2/runtime/bin/karaf server
           └─982 /usr/bin/java -Dopenhab.home=/usr/share/openhab2 -Dopenhab.conf=/etc/openhab2 -Dopenhab.runtime=/usr/share/openhab2/runtime -Dopenhab.us...

Jun 27 20:45:07 openHAB systemd[1]: Started openHAB 2 - empowering the smart home.
Jun 27 20:45:07 openHAB start.sh[831]: Launching the openHAB runtime...
pi@openHAB:~ $ sudo systemctl daemon-reload
pi@openHAB:~ $ sudo systemctl enable openhab2.service
Synchronizing state for openhab2.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d openhab2 defaults
Executing /usr/sbin/update-rc.d openhab2 enable
Created symlink from /etc/systemd/system/multi-user.target.wants/openhab2.service to /usr/lib/systemd/system/openhab2.service.
pi@openHAB:~ $ 


Vor dem Konfigurieren von openHAB ist ggf. ein Backup der Specherkarte empfehlenswert!

    sudo shutdown now
    
    
    am PC:
    
    dmesg
    
    [ 5975.873361]  sdc: sdc1 sdc2
[pierre@pierre-pc Downloads]$ dd bs=4M if=/dev/sdc of=/home/pierre/Downloads/2017-06-28-raspbian-jessie-lite_openHAB_backup.img
dd: konnte '/dev/sdc' nicht öffnen: Keine Berechtigung
[pierre@pierre-pc Downloads]$ sudo dd bs=4M if=/dev/sdc of=/home/pierre/Downloads/2017-06-28-raspbian-jessie-lite_openHAB_backup.img
[sudo] Passwort für pierre: 
931+0 Datensätze ein
931+0 Datensätze aus
3904897024 Bytes (3,9 GB, 3,6 GiB) kopiert, 190,048 s, 20,5 MB/s
[pierre@pierre-pc Downloads]$ sudo mc

[pierre@pierre-pc Downloads]$ sudo dd bs=4M if=/dev/sdc of=/home/pierre/Downloads/2017-06-28-raspbian-jessie-lite_openHAB_backup.img
931+0 Datensätze ein
931+0 Datensätze aus
3904897024 Bytes (3,9 GB, 3,6 GiB) kopiert, 182,758 s, 21,4 MB/s


    
Nach erfolgreichem ausschalten und herunterfahren des Systems die Speicherkarte entfernen.


Text Basiert
------------

Im Konfigurstionsverzeichnis von openHAB 2 (/etc/openhab2 bei einer Installation via apt-get z. B. Raspberry oder /snap/openhab/current/conf in ubuntu snapy) gibt es das neue Verzeichnis *services*. In diesem Verzeichnis gibt es die Datei *runtime.cfg*, in der die globalen openHAB 2 parameter gespeichert sind. Öffnen sie diese Datei und bearbeiten sie folgende Parameter:  

    org.eclipse.smarthome.persistence:default=mapdb - Standard *persistence*, hier die bisherige Einstellung verwenden welche unter persistence:default gesetzt ist, siehe openhab.cfg
    autoapprove:enabled=false - makes it so you must manually approve automatically discovered Things in the Inbox (see below)
    org.eclipse.smarthome.links:autoLinks=false - prevents openHAB from automatically creating an Item for each automatically discovered Thing.


Karaf Console
-------------

