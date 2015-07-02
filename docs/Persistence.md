Persistence
===========

Mit den verschiedenen Arten von Persistence die openHAB bietet, besteht
die Möglichkeit den Status der Items in Datenbanken oder Log-Dateien zu
speichern. Die gespeicherten Werte lassen sich in openHab oder externen
Tools weiterverarbeiten. Es besteht die Möglichkeit, z. B. in Rules,
auf die von der Persistence gespeicherten Daten zuzugreifen, um z. B.
historische Daten die eine Woche zurückliegen anzuzeigen. Zusätzlich
kann auf einfache Weise beim Neustart von openHab der letzte Wert des Items aus
der Datenbank ausgelesen und dem jeweiligen Items zugewiesen werden.

Die verschiedenen Persistences können parallel betrieben werden, so ist
es möglich, das gleiche Item in mehreren Persistences zu speichern.

Aktuell bietet openHAB folgende Datenbanken an:

-   **db4o** - eine leichtgewichtige 100% Java Datenbank

-   **rrd4j** - eine Java Version der Round-Robin-Datenbank Lösung (RRDtool).

-   **mysql** - MySQL Datenbank

-   **mongodb** - MongoDB

-   **Open.Sen.Se** - Eine Internet-of-Things Platform

-   **logging** - Schreibt Zustände der Items in Log-Dateien

-   **InfluxDB** - Eine open-source Datenbank

Allgemeines zu Persistence
--------------------------

Für jeden Persistence-Service der Verwendet werden soll, muss eine
entsprechende Konfigurationsdatei <persistenceservice>.persist (z. B.
db4o.persist) im Verzeichnis <Pfad_zu_openHAB>/configurations/persistence angelegt werden.
Um Fehler zu vermeiden sollten die Konfigurationsdateien mit dem openHAB
Designer angelegt und bearbeitet werden, da dieser Autovervollständigung
und Syntax-Checks bietet.

Das Konzept der Konfigurationsdatei ist, openHAB möglichst einfach zu
sagen welche Items gespeichert wann gespeichert werden sollen. Die
Konfigurationsdatei untergliedert sich in die zwei Sektionen Strategies
und Items.

Die Strategies Sektion ermöglicht das definieren der Zeitpunkte an denen
die Items für dieses Persistence gespeichert werden. Der Syntax ist wie
folgt:

    Strategies {
        <strategyName1> : "<cronexpression1>"
        <strategyName2> : "<cronexpression2>"
        ...

        default = <strategyNameX>, <strategyNameY>
    }

Die folgenden Strategien sind bereits vordefiniert und müssen nicht
definiert werden:

-   **everyChange** - speichert den Status bei jeder Änderung.

-   **everyUpdate** - speichert den Status bei jedem Update auch wenn
    dieser sich nicht geändert hat.

-   **restoreOnStartup** - Wenn der Status des Item bei einem Neustart
    von openHAB undefiniert ist, wird das Item mit dem zuletzt
    gespeichertem Wert Inizialisiert. Dies ist für virtuelle" Items die
    denen kein Binding oder Hardware zugeordnet ist.

Die Items Sektion definiert die Items die mit den zuvor definierten
Strategien gespeichert werden. Der Syntax ist wie folgt:  

    Items {
        <itemlist1> [-> "<alias1>"] : [strategy = <strategy1>, <strategy2>, ...]
        <itemlist2> [-> "<alias2>"] : [strategy = <strategyX>, <strategyY>, ...]
        ...
    }

Wobei es sich bei <itemlist> um ein einzelnes Item einer Gruppe oder
um eine durch Kommma getrennte Liste von Items handeln kann. Zusätzlich
ist noch das Jokerzeichen \* möglich. Hierzu siehe folgende Erklärung:  

-   \* - Alle Item werden gespeichert.

-   \<itemName\> - Ein einzelnes Item das durch seinen Namen
    Identifiziert wird. Das Item kann auch ein gruppen-item sein,
    allerdings wird nur der Gruppen-Wert gespeichert nicht die einzelnen
    Werte der Items die Mitglied der Gruppe sind.

-   \<groupName\>\* - Alle Werte der Mitglieder der Gruppe werden
    gespeichert allerdings die nicht der Gruppen-Wert selbst.

Den Items kann optional noch ein Alias hinzugefügt werden, sofern für
den Persistence-Service ein spezifischer Namen erforderlich ist. (z. B.
eine Feed-ID für einen IoT-service)\

Eine gültige Persistence konfigurationsdatei könnte wie folgt aussehen:

    // Persistence Strategien haben einen Namen und eine Definition
    Strategies {
        everyHour : "0 0 * * * ?"
        everyDay  : "0 0 0 * * ?"

        // Wenn bei einem unten definierten Item keine Strategie angebeben ist,
        // wird die Default-Liste angewendet
        default = everyChange
    }

    /*
     * Each line in this section defines for which item(s) which strategy(ies)
     * should be applied. You can list single items, use "*" for all items or
     * "groupitem*" for all members of a group item (excl. the group item itself).
     */
    Items {
        // Speichere die Daten aller Items jeden Tag (veryDay), bei jeder Änderung (everyChange) und
        // ordne den Items bei einem Neustartand den letzten Wert zu (restoreOnStartup)
        * : strategy = everyChange, everyDay, restoreOnStartup

        // Zusätzlich speichere alle Temperaturen und Wetterdaten stündlich (everyHour)
        Temperature*, Weather* : strategy = everyHour
    }

Zusätzlich zur Konfigurationsdatei muss noch das entsprechende Binding
in das Verzeichnis <Pfad_zu_openHAB>/runtime/addons kopiert und die
openHAB Konfigurationsdatei openhab.cfg angepasst werden.  

Zusammenfassend sind folgende Schritte für die Verwendung eines
Persistence durchzuführen:

-   Das entsprechende Binding in das Verzeichnis <Pfad_zu_openHAB\>/runtime/addons kopieren.

-   Die Konfigurationsdatei openhab.cfg anpassen

-   Im Verzeichnis <Pfad\_zu\_openHAB>/runtime/configurations/persistence/ die Datei <persistence>.persist erstellen.

Im folgenden wird gezielter auf ein Teil der möglichen Datenbanken
eingegangen.

Persistence.db4o
----------------

Persistence.rrd4j
-----------------

Persistence.sql
---------------
