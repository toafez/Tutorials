[English](ssh_from_os_to_nas_en.md) | Deutsch

### Dieses Tutorial enthält die ergänzende Videobeschreibung meines YouTube Videos... ###
# **[SSH Public Key Verbindung zu einem Synology NAS einrichten](https://youtu.be/VjoWjX_8E3Q)** #

## **Auf deinem Windows-, Linux-, Mac- oder Whatever Betriebssystem**
Wir benötigen zunächst einmal ein Terminal Programm um uns auf die Konsole unseres Windows-, Linux-, Mac- oder whatever Betriebssystem zu begeben. Auf der Konsole angekommen wollen wir uns kurz orientieren...
- **Wer bin ich**

    ```whoami```

- **Wo befinde ich mich überhaupt**

    `pwd`

    _(Print Working Directory)_

- **Ordnerinhalt anzeigen**

    `ls -la`

    _(Option **l** = Datei-Informationen werden in Langform ausgegeben)_

    _(Option **a** = Zeigt auch versteckte Dateien und Ordner an, die mit einem Punkt beginnen)_

- **Einen neuen, versteckten Ordner mit dem Namen .ssh anlegen und Ordnerrechte vergeben**

    `mkdir -p ~/.ssh`
  
    `chmod 0700 ~/.ssh`

- **Die Bedeutung von Tilde (~)**

    Das **Tilde**-Symbol (**~**) im Linux-Dateisystem stellt das Home-Verzeichnis des aktuellen Benutzers dar. Es handelt sich um eine Kurzschreibweise, die es Benutzern ermöglicht, auf ihr eigenes Home-Verzeichnis zu verweisen, ohne den vollständigen Pfad eingeben zu müssen.

- **RSA-Schlüsselpaar erstellen**

    Durch die einfache Eingabe des Befehls **ssh-keygen**, können je nach Konfiguration, unterschiedlichen Verschlüsselungsalgorithmen herangezogen werden. Daher immer angeben, welchen Algorithmus man verwenden möchte.

    `ssh-keygen -b 4096 -t rsa -f ~/.ssh/id_rsa`

    _(**-b** steht für Bit, also die Bitlänge des Verschlüsselungsschlüssels, hier 4096 Bit)_

    _(**-t** steht für type, also den Verschlüsselungstyp, hier rsa für RSA Protokoll 2)_

    _(**-f** steht für den zu verwendenden Ordnerpfad und Dateinamen. Beispielhaft wird hier der Standard Dateiname id_rsa verwendet)_

- **Den Inhalt des öffentlichen Schlüssels ausgeben**

    `cat ~/.ssh/id_rsa.pub`

- **Eine Datei mit dem Namen authorized_keys erstellen und den eigenen öffentlichen Schlüssel hinzufügen**

    `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

    _(**cat** = concatenate = verketten, verknüpfen)_

    _(Befehl **>** Datei = Standardausgabe des Befehls in Zieldatei (über-)schreiben.)_

    _(Befehl **>>** Datei = Standardausgabe des Befehl an Zieldatei anhängen.)_

- **Ordner und Dateirechte setzten**

    Mit **chmod** verändert man die Zugriffsrechte von Ordner und Dateien.

    `chmod 0700 ~/.ssh`

    `chmod 0600 ~/.ssh/id_rsa*`

    `chmod 0600 ~/.ssh/authorized_keys`

- **SSH-Verbindung zu deinem Synology NAS aufbauen**

    **Hinweis:** Aktiviere auf deinem Synology NAS im Vorfeld bitte den SSH Terminal-Dienst sowie den Benutzer-Home-Dienst.

    **Syntax:** ssh -p [PORT] [BENUTZER]@[NAS-IP]

    **Beispiel:** `ssh -p 22 tommes@172.16.1.11`

    _(Verbindung durch einen Handshake akzeptieren und Benutzerpasswort eingeben)_

## Auf deinem Synology NAS

- **Wo befinde ich mich überhaupt**

    `pwd`

    _(Print Working Directory)_

    _(Du befindest dich im Benutzer-Home-Ordner deines Administrators unter /var/services/homes/[BENUTZER] bzw. /volume1/homes/[BENUTZER])_

- **Einen neuen, versteckten Ordner mit dem Namen .ssh anlegen und Ordnerrechte vergeben**

    `mkdir -p ~/.ssh`

    `chmod 0700 ~/.ssh`

    `exit`

## Zurück auf deinem Windows-, Linux-, Mac- oder Whatever Rechner

- **Den Inhalt der Datei id\_rsa.pub über eine SSH-Verbindung zu eurem Synology NAS an die Datei authorized\_keys anhängen**

    `cat ~/.ssh/id_rsa.pub | ssh -p [PORT] [BENUTZER]@[NAS-IP] "cat >> ~/.ssh/authorized_keys"`

- **Erneute Verbindung zu unserem Synolog NAS, diesmal jedoch ohne die Eingabe eines Passwortes**

    `ssh -p [PORT] [BENUTZER]@[NAS-IP]`

    _(Anmeldung ohne Passwort)_

## Wieder auf deinem Synology NAS

- **Dateirechte setzten**

    `chmod 0600 ~/.ssh/authorized_keys`

- **Wechsel zum Root-Konto**

    `sudo -i`

    _(Passwort des Administrators wiederholt eingeben und Handshake ausführen)_

- **Wo befinde ich mich überhaupt**

    `pwd`

    _(Print Working Directory)_

    _(Du befindest dich im Home-Ordner von root unter /root)_

- **Einen neuen, versteckten Ordner mit dem Namen .ssh anlegen und Ordnerrechte vergeben**

    `mkdir -p ~/.ssh`
  
    `chmod 0700 ~/.ssh`

- **Den Inhalt der Datei id_rsa.pub aus dem Benutzer-Home-Ordner eures Administrators in den root Ordner kopieren**

    `cp /var/services/homes/[BENUTZER]/.ssh/id_rsa.pub /root/.ssh/authorized_keys`

    ... oder alternativ ...

    `cp /volume1/homes/[BENUTZER]/.ssh/id_rsa.pub /root/.ssh/authorized_keys`

- **Dateirechte setzten**

    `chmod 0600 ~/.ssh/authorized_keys`
