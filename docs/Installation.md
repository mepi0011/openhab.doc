Installation auf PC
===================

OpenHAB ist ein in Java geschriebenes Programm, das auf jedem
Betriebssystem läuft, auf welchem Java installiert ist.

OpenHAB Runtime
---------------

Die aktuelle Version kann von der Projektseite [openhab.org](http://www.openhab.org)
heruntergeladen und in einem beliebigen Ordner entpackt werden.  

z.B.:  
Linux: /openhab/runtime
Windows: E:\openhab\runtime\

Um die Schreibweise zu vereinheitlichen, wird in den folgenden Kapitel
das Verzeichnis mit <Pfad\_zu\_openHAB\> ersetzt. Zum Ausführen von
openHAB, wechseln sie in das Verzeichnis <Pfad\_zu\_openHAB\>. Abhängig
vom verwendeten Betriebssystem führen sie eines der beiden Scripte aus.
Linux: start.sh
Windows: start.bat

![Downloadseite von openHAB](images/openhab_Download.png "Downloadseite von openHAB")


BINDING/Addons herunterladen
----------------------------

Für die Kommunikation mit anderen Geräten oder Funktionen sind
sogenannte Bindings notwendig. So wird zum Beispiel für die
Kommunikation zu einem KNX-Gateway das KNX Bindings benötigt. Wie die
openHAB-Runtime, können auch die Bindings (Addons) von openhab.org
heruntergeladen und in einem beliebigen Ordner entpackt werden. Abhängig
davon mit welchen Geräten man kommunizieren möchte wird ein
entsprechendes Binding benötigt. Dieses wird in den Ordner
<Pfad\_zu\_openHAB\>/addons kopiert. Nach einer evtl. Konfiguration des
Bindings in der openHAB-Konfigurationsdatei openhab.cfg und einem
Neustart, steht dies zur Verfügung.

* * * * *
<table>
<tr>
<td> ![Hinweis!](images/Warning.png "Hinweis! Konfiguration Bindings in der openhab.cfg") </td>
<td> Einige Bindings erfordern eine Konfiguration in der openHAB-Konfigurationsdatei openhab.cfg. </td>
</tr>
</table>
* * * * *

openHAB Designer
----------------

Es ist von Vorteil für die Arbeit mit openHAB den openHAB Designer zu
verwenden, da dieser Syntaxfehler erkennt und eine Autovervollständigung
implementiert hat. Der openHAB Designer kann auch von openHAB.org
heruntergeladen werden, dabei ist auf die Betriebssystemversion zu
achten. Nach dem Download den Designer in einem beliebigen Ordner
entpacken (z.B.: Linux: /openHAB/designer).
