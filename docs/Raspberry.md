OpenHAB auf dem dem Raspberry PI
================================

Die folgenden Kapitel wird die Installation und Verwendung von Raspbian
auf dem Raspberry PI sowie die Nutzung von openHAB auf diesem System
erklärt. Prinzipiell können für die Nutzung von openHAB auf dem
Raspberry PI auch andere Betriebssysteme verwendet werden. Im Folgenden
beschränke ich mich auf Raspbian. Hierbei handelt es sich wie bei den
meisten Linux Systemen um ein offenes Betriebssystem. Raspbian wurde
speziell für den Raspberry optimiert und basiert auf Debian. Dies
ermöglicht die Nutzung vieler Pakete (Programme) die für Debian
bereitgestellt werden. Der Name Raspbian setzt sich aus Raspberry und
Debian zusammen.

Raspbian installieren
---------------------

Das Raspbian Image kann entweder direkt von www.raspberry.org oder
www.raspbian.org heruntergeladen werden. Alternativ kann für die
Installation von Raspbian oder andere Systeme für den Raspberry NOBOS
verwendet werden. Hierzu gibt es auf der Homepage www.raspberry.org
ausführliche Installationsanleitungen und ein Video. Außerdem gibt es im
Handel bereits SD Karte auf denen Raspbian oder NOBOS bereits
vorinstalliert ist. Dies erleichtert es für Anfänger die sich mit der
Installation des Betriebssystems nicht beschäftigen wollen. Im Folgenden
wird die direkte Installation des Raspbian-Images beschrieben wie sie
von raspberry.org heruntergeladen werden kann.

-   Die aktuelle Raspbian Version von www.raspberrypi.org herunterladen und entpacken.

-   Raspbian-Image von einem Windows Betriebssystem auf eine SD-Karte
    (mind. 4 GB) kopieren.  
    Hierzu kann das Programm Win32DiskImager verwendet werden. Dieses
    kann hier http://sourceforge.net/projects/win32diskimager/files/
    heruntergeladen werden.  
    ![Eingabefenster von Win32 Disk Imager](images/win32-imagewriter.png "Eingabefenster von Win32 Disk Imager")

    Nach Installation und Programmstart die SD-Karte und das Image auswählen, anschließend den Vorgang
    starten.

-   Raspbian-Image von einem linux System auf die SD-Karte kopieren.  
    Hierzu kann das Programm dd im Terminal bzw. Konsole verwendet werden. Das Programm dd ist in den meisten Linux Distributionen bereits in der Grundinstallation enthalten.  

    Der Gerätepfad (hier /dev/sdd) für die SD Karte muss allerdings
    zuerst ausfindig gemacht werden! In manchen Fällen ist das
    Aufspielen den Images mit dem Parameter bs=4M fehlerhaft, in diesem
    Fall das Ganze nochmals mit bs=1M testen.

-   Nach dem Aufspielen des Image auf die SD-Karte, diese in den
    Raspberry einstecken und diesen an einen TV oder Beamer über HDMI
    oder Composite anschließen und mit Spannung versorgen. Beim ersten
    Start wird ein Konfigurationstool gestartet (siehe Bild) sollte dies
    nicht der Fall sein, dann das Konfigurationstool wie folgt
    aufgerufen werden: sudo raspi-config Im Konfigurationstool sollten
    folgende Punkte nacheinander durchgeführt werden: Enable ssh (bei
    der aktuellen Version nicht mehr Notwendig), change_pass,
    configure_keyboard and expand_rootfs

-   Nach dem Einrichten der wichtigsten Einstellungen die Konfiguration
    mit <Finish> abschließen. Der Raspberry boote neu, sollte dies
    nicht der Fall sein, dann dies mit folgendem Befehl durchgeführt
    werden:  
    `sudo reboot -now`

-   Nach einem Neustart kann man sich mit seinem eigenen Passwort
    anmelden oder wenn dies nicht geändert wurde mit USER: pi,
    Passwort:raspberry Achtung: Zu beginn ist die Tastatur nicht auf
    deutsch eingestellt, wodurch y und z vertauscht sind!

Installation des Midnight Commander (mc)
----------------------------------------

Nach erfolgreicher Installation von Linux sollten das Hilfreiche
Programm mc nicht fehlen.Damit der Midnight Commander installiert wird,
nach dem hochfahren und anmelden folgenden Befehl ausführen: sudo
apt-get install mc Danach wird man aufgefordert, das root-Passwort
einzugeben, dies ist das gleiche wie für den user pi. Nach Installation
kann das Programm mc wie folgt aufgerufen werden: mc (ohne root Rechte)
oder sudo mc (mit root Rechte)  
![Midnight Commander](images/mc_home.png "Midnight Commander")

Statische IP-Adressen inrichten
-------------------------------

Standardmäßig läuft die Netzwerkkarten mit DHCP. Möchte man einem der
Netzwerk Schnittstellen eine eine feste IP Adresse zuweisen, muss unter
Linux die Datei /etc/network/interface angepasst werden. Die Datei
selbst kann allerdings nur von root angepasst werden. Um die Datei
anzupassen, wie folgt vorgehen:

1.  Wechseln in das entsprechende Verzeichnis cd /etc/network

2.  Starten des Midnight Commander und öffnen der Datei interface  
    `sudo mc`

3.  Standardmäßig sollte folgendes drin stehen:  
    ![Inhalt der Datei interface](images/mc_network.png "Inhalt der Datei interface")

4.  Um z.B. der Netzwerkschnittstelle eine feste IP einzurichten, muss
    der Block für eth0 wie folgt geändert werden: Man erkennt sehr
    schnell dass das Wort ’static’ wohl eine feste IP-Adresse andeutet.
    address - Die IP-Adresse für das Interface broadcast - Die
    Broadcast-Adresse (Endet meist auf .255) netmask - Die Subnetzmaske
    gateway - Das Gateway für Zieladressen die nicht im Lokalen Netz
    liegen

Quelle: http://juliusbeckmann.de/blog/statische-ip-adressen-in-debian-und-konsorten-einrichten.html

Java Installieren
-----------------

Seit kurzen hat sich die Java-Installation deutlich vereinfacht. Die
passenden hard-float oder soft-float Java Variante muss nicht von Oracle
heruntergeladen und über mehrere Schritte installiert werden. Raspbian
bringt nun die passende Java Version die auch hard-float unterstützt mit
und kann einfach mit folgenden Befehl installiert werden.  
    sudo apt-get update
    sudo apt-get install oracle-java7-jdk

openHAB auf dem Raspberry Einrichten
------------------------------------

Die „Installation“ von openHAB auf dem Raspberry ist gleich wie auf
einen PC. Damit später der Autostart funktioniert, wird empfohlen die
openHAB Dateien in das Verzeichnis /opt/openhab zu kopieren. Die Daten
mit Hilfe von WinSPC oder über einen gemounteten USB Stick und mc an die
gewünschte Stelle kopieren.

Autostart von openHAB einrichten
--------------------------------

Damit der Autostart funktioniert, werden die Dateien openhab und
openhab.cfg benötigt. Die beiden Dateien werden am einfachsten am PC
erstellt und anschließend auf den Raspberry kopiert. Die Script-Datei
openhab:

Die Konfiguration-Datei openhab.cfg

-   Die Script-Datei openhab in das Verzeichnis /etc/init.d/ kopieren

-   Die Konfigurationsdatei openhab.cfg in /etc/defaults kopieren

-   In das Verzeichnis wechseln in dem das Script openhab liegt cd
    /etc/init.d

-   Die Datei ausführbar machen sudo chmod a+x openhab

-   Gruppe und Eigentümer der Datei ändern\
    sudo chgrp root openhab\
    sudo chown root openhab\

-   Überprüfen der Änderungen\
    ls -l\
    Die Ausgabe müsste wie folgt aussehen:\
    -rwxr-xr-x 1 root root 1757 Apr 13 23:27 /etc/init.d/openhab\

-   Nachdem das Script ausführbar ist, diesen noch in die Runnnlevel
    eintragen\
    update-rc.d opehab defaults

-   Nun kann openHAB mit dem Befehl\
    sudo /etc/init.d/openhab stop\
    gestartet, bez mit\
    sudo /etc/init.d/openhab stop\
    gestoppt werden.


