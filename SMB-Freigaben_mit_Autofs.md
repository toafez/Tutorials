[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)

# HowTo: SMB-Netzwerkfreigaben mit Autofs auf einer Debian-basierten Linux-Distribution

## Worum geht es?
Die folgende Anleitung beschreibt die **Installation und Konfiguration** von **Autofs** zur dynamischen Einbindung von SMB-Netzwerkfreigaben auf einer **Debian-basierten Linux-Distribution**, genauer gesagt unter **Linux Mint**.

## Einleitung
Um SMB-Netzwerkfreigaben bereits beim Starten des Betriebssystems dauerhaft einzubinden, werden häufig Einträge in der Systemdatei `/etc/fstab` vorgenommen. Enthält ein solcher Eintrag inhaltliche oder syntaktische Fehler oder kann die beschriebene Netzwerkfreigabe aus irgendwelchen Gründen während des Bootvorgangs nicht eingebunden werden, kann dies den Start des Betriebssystems verzögern oder sogar verhindern. Autofs hingegen bindet SMB-Netzwerkfreigaben erst nach dem Start des Betriebssystems und nur auf Anforderung des Benutzers ein. Bis zu diesem Zeitpunkt befinden sich die in Autofs konfigurierten Freigaben im ungemounteten Zustand. Erst wenn der Benutzer auf die Freigabe zugreift, wird diese automatisch eingebunden und ggf. nach einer voreingestellten Zeit automatisch wieder getrennt.

#### _Hinweis: Texte in Großbuchstaben innerhalb eckiger Klammern dienen als Platzhalter und müssen durch eigene Angaben ersetzt werden, können aber an einigen Stellen auch nur der Information dienen. Es ist zu beachten, dass die eckigen Klammern Teil des Platzhalters sind und beim Ersetzen durch eigene Angaben ebenfalls entfernt werden müssen._

## Installation des Tool-Pakets _cifs-utils_
Damit SMB-Netzwerkfreigaben überhaupt in das lokale Linux-Dateisystem eingebunden werden können, wird noch das Programm `mount.cifs` benötigt, das Bestandteil des Tool-Pakets `cifs-utils` ist. Der Befehl zur Installation des Tool-Pakets lautet

    sudo apt install cifs-utils

## Authentifikationsdatei _.smbcredentials_ anlegen
Eine SMB-Netzwerkfreigabe erfordert in der Regel die Eingabe eines Benutzernamens und eines Passworts. Um später eine automatische Anmeldung an einer SMB-Netzwerkfreigabe mittels Autofs zu ermöglichen, sollten diese Zugangsdaten möglichst in einer **versteckten Datei** mit dem Namen `.smbcredentials` im Home-Verzeichnis des aktuell angemeldeten Benutzers gespeichert werden. 

Wenn auf mehrere SMB-Netzwerkfreigaben unterschiedlicher Server mit abweichenden Zugangsdaten zugegriffen werden soll, empfiehlt es sich, die Datei `.smbcredentials` um den Namen des jeweiligen Servers zu erweitern z. B. `.smbcredentials_[SERVERNAME]` oder einen komplett anderen, aussagekräftigeren Dateinamen, **der mit einem Punkt beginnt**, zu wählen.

- Eine neue, leere Datei mit dem Namen `.smbcredentials` oder einem abweichenden Dateinamen im Home-Verzeichnis des aktuell angemeldeten Benutzers erstellen.

      touch ~/.smbcredentials

- Die soeben erstellte Datei editieren. In diesem Beispiel wird der Texteditor `nano` verwendet.

      nano ~/.smbcredentials

- Bitte ersetze die folgenden [PLATZHALTER] durch deine eigenen Zugangsdaten, wobei die dritte Zeile nur benötigt wird, wenn der Server, auf dem sich die SMB-Freigabe befindet, zu einer Windows-Domäne gehört. Ist dies nicht der Fall, muss bzw. sollte die dritte Zeile entfernt werden.

    ```
    username=[BENUTZERNAME]
    password=[PASSWORT]
    domain=[DOMAIN]
    ```

- Da in der Datei `.smbcredentials` die Zugangsdaten im Klartext gespeichert werden, müssen die Dateirechte noch so angepasst werden, dass sie nur vom Besitzer selbst eingesehen werden können.

      sudo chmod 600 ~/.smbcredentials

## Installation von _Autofs_
Da jetzt alle Vorbereitungen abgeschlossen sind, kann nun damit begonnen werden Autofs zu installieren. Der Befehl dazu lautet

    sudo apt install autofs

## Mountpoint festlegen, wohin die SMB-Freigaben gemountet werden sollen.
Für jeden Server, der SMB-Netzwerkfreigaben zur Verfügung stellt, die in das lokale Linux-Dateisystem eingebunden werden sollen, muss ein eigener Mountpoint definiert werden. Prädestiniert dafür ist das Systemverzeichnis `/media`. In Mehrbenutzersystemen sollte der Mountpoint zusätzlich den Benutzernamen enthalten, der die Freigabe anfordert, gefolgt vom eigentlichen Servernamen. Ersetze bitte die folgenden Platzhalter durch deine eigenen Angaben. Führe den Befehl in diesem Fall mit Root-Berechtigung aus, da es sich um einen Systemordner handelt.

    sudo mkdir -p /media/[BENUTZERNAME]/[SERVERNAME]

Alternativ kann auch ein frei wählbares Verzeichnis im eigenen Benutzer-Home-Ordner verwendet werden, wie z.B. 

    mkdir -p /home/[BENUTZERNAME]/Netzlaufwerke/[SERVERNAME]
    
In diesem Fall kann beim Erstellen des Verzeichnisses die Root-Berechtigung (ein vorangestelltes „sudo”) ignoriert werden. Der Speicherort und somit die genaue Definition des Mountpoints bleibt jedoch jedem selbst überlassen. Im Folgenden wird beispielhaft das Systemverzeichnis „/media” verwendet. 

## Die Map-Datei
Die eigentlichen SMB-Netzwerkfreigaben werden nun zeilenweise in eine Map-Datei eingetragen, deren Speicherort und Dateiname frei gewählt werden kann. Es empfiehlt sich jedoch, die Datei unter /etc mit dem Dateipräfix `auto.`, gefolgt vom Servernamen abzulegen, also z.B. `/etc/auto.mein-server`. Führe alle nachfolgenden Befehle als `root` aus.

- Erstelle eine neue, leere Map-Datei im Verzeichnis /etc und vergib dieser Datei einen eindeutigen Namen.

      sudo touch /etc/auto.[SERVERNAME]

- Es ist zwingend erforderlich, der Map-Datei die Ausführbarkeit zu entziehen.

      sudo chmod -x /etc/auto.[SERVERNAME]

- Die soeben erstellte Datei kann nun bearbeitet werden, um die gewünschten Netzwerkfreigaben Zeile für Zeile einzutragen.

      sudo nano /etc/auto.[SERVERNAME]

- Bitte ersetze die folgenden [PLATZHALTER] durch deine eigenen Angaben. Als [NAME-DER-FREIGABE] ist ein eindeutiger Name zu verwenden, der sich vom tatsächlichen Namen der Freigabe unterscheiden kann, aber Aufschluss über die einzubindende Freigabe gibt. Bei einer SMB-Freigabe enthält der Platzhalter [WEITERE-OPTIONEN] neben der, durch Komma getrennten `uid` und ggf. `gid` des Benutzers, Informationen über den Speicherort der `.smbcredentials`. Zusätzlich muss die IP-Adresse [IP-ADRESSE], der Pfad und der Name des freigegebenen Ordners [PFAD-UND-NAME-DER-FREIGABE] des Servers angegeben werden.

    ***Syntax:***
  
      [NAME-DER-FREIGABE] -fstype=cifs,[WEITERE-OPTIONEN] ://[IP-ADRESSE]/[PFAD-UND-NAME-DER-FREIGABE]

    ***Beispiel:***
  
      Dokumente -fstype=cifs,uid=1000,credentials=/home/tommes/.smbcredentials ://172.16.1.10/Dokumente

## Die Master-Map-Datei
In der Map-Master-Datei, die bereits während der Installation von Autofs unter `/etc/auto.master` angelegt wurde, wird der oben definierte Mountpoint mit der Map-Datei verknüpft, in der bereits die Verbindungsinformationen der einzelnen Freigaben gespeichert wurden. In der Regel sollte die Map-Master-Datei noch keine Daten enthalten, andernfalls sind die folgenden Einträge am Ende der Datei `/etc/auto.master` einzufügen.

***Syntax:***

    [PFAD-UND-NAME-DES-MOUNTPOINTS] [PFAD-UND-NAME-DER-MAP-DATEI] [WEITERE-OPTIONEN]

***Beispiel:*** 

    /media/tommes/Fileserver /etc/auto.Fileserver --timeout=0 --ghost

Die Option `--timeout` gibt an, nach welcher Zeit die Freigabe wieder ausgehängt werden soll. Standardmäßig wird ein Wert von 300 Sekunden (entspricht 5 Minuten) empfohlen. Wird der Wert 0 eingegeben, wird die Freigabe nicht automatisch ausgehängt.

Die Option `--ghost` veranlasst Autofs, leere Dateiverzeichnisse im angegebenen Mountpoint anzulegen, damit die Verbindung nach einem Timeout wieder aufgebaut werden kann.

## Starten, neu laden und stoppen von Autofs
Autofs wird nach der Installation automatisch gestartet, kann mit Root-Rechten aber jederzeit gestppt, gestartet oder neu geladen werden. Der dafür nötige Befehl lautet...

    sudo service autofs [OPTION]

Als Optionen können die Werte `stop`, `start`, `reload`, `restart` und `status` eingesetzt werden. Da die Optionen selbsterklärend sind, wird an dieser Stelle nicht weiter darauf eingegangen.

## Tipps & Tricks
In der Vergangenheit wurden die eingebundenen SMB-Netzwerklaufwerke im Dateimanager `Nemo` von Linux Mint direkt nach dem Systemstart ohne Probleme unter `Geräte` angezeigt. Dies scheint sich mittlerweile geändert zu haben, vermutlich nach einem Update, dessen Zeitpunkt und Name mir nicht bekannt ist.

Mit einem kleinen Trick lässt sich das Problem jedoch beheben. Dazu lässt man über die App „Startprogramme” ein kleines Skript unmittelbar nach dem Systemstart ausführen, das jeden Mountpoint einmal über den `ls` Befehl aufruft. Alternativ kann man das Skript auch direkt als Cron Job konfigurieren.

Nachfolgend ein Beispiel, wie so ein Skript aussehen könnte (natürlich )

```
#!/bin/bash
ls /home/[BENUTZERNAME]/Netzlaufwerke/[SERVERNAME]/Dokumente >/dev/null
ls /home/[BENUTZERNAME]/Netzlaufwerke/[SERVERNAME]/Downloads >/dev/null
...
..
.. usw. ..

```


[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)
