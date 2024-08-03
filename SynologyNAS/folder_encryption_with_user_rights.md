[English](folder_encryption_with_user_rights_en.md) | Deutsch

## Einhängen und trennen verschlüsselter Ordner als DSM-Benutzer
Mit dem hier vorgestellten Überwachungsscript lassen sich verschlüsselte Ordner auf auf deinem Synology-NAS auch von Benutzern einhängen und trennen, die nicht der Gruppe der Administratoren angehören.

## Arbeitsweise des Überwachungsscripts
Nach der **einmaligen Ausführung** des **Überwachungsscripts** über den **DSM-Aufgabenplaner** als Benutzer "**root**" wird zunächst **im Wurzelverzeichnis** eines zuvor bestimmten **Benutzer-Home-Ordners** geprüft, ob bereits eine sogenannte "**Ereignisdatei**" existiert. Wurde keine "Ereignisdatei" gefunden, wird diese durch das Script mit einen frei definierbaren Dateinamen erstellt. Im Anschluss daran wird die "Ereignisdatei" durch das Überwachungsscript auf **abgespeicherte Inhaltsänderungen** hin **überwacht**.

Innerhalb der "Ereignisdatei" werden dabei frei definierbare "**Auslösewerte**" für das einhängen und trennen verschlüsselter Ordner sowie für die Beendigung der Überwachung **hineingeschrieben**. Wird nach dem abspeichern ein gültiger Auslösewert von dem Überwachungsscript erkannt, erfolgt eine entsprechende Aktion.

**Wichtiger Hinweis:** Das Editieren der "Ereignisdatei" ist nur über den DSM Text-Editor möglich.

## Insallationshinweise
- ### Benutzer-Home-Dienst aktivieren   
    Aktiviere im Vorfeld bitte den **Benutzer-Home-Dienst**. Zum aktivieren des Benutzer-Home-Dienstes gehe über **DSM-Hauptmenü** > **Systemsteuerung** > **Benutzer und Gruppe** und wechsel dort in den Reiter > **Erweitert**. Aktivere unter dem Menüpunkt **Benutzerbasis** die Checkbox **Benutzer-Home-Dienst aktiveren**.

- ### DSM Text-Editor installieren
    Installiere dir über das **DSM-Paketzentrum** den **Text-Editor** um im späteren Verlauf die "Ereignisdatei" innerhalb des DSM editieren zu können. Achte darauf, das der gewünschte DSM-Benutzer den Text-Editor auch verwenden darf. Dies kannst du über **Hauptmenü > Systemsteuerung > Anwendungsberechtigungen** entsprechend einstellen.

- ### inotify-tools installieren
    Installiere dir über das **DSM-Paketzentrum** des Weiteren das Paket [inotify-tools](https://synocommunity.com/package/inotify-tools) der [SynoCommunity](https://synocommunity.com/)

- ### Das Überwachungsscript zum Aufgabenplaner hinzufügen
    - Melde dich als **Adminstrator** am DSM deines Synology-NAS an.
    - Öffne im DSM unter **Hauptmenü > Systemsteuerung** den Aufgabenplaner.
    - Wähle im Aufgabenplaner über die Schaltfläche **Erstellen > Geplante Aufgabe > Benutzerdefiniertes Script** aus.
    - In dem sich nun öffnenden Pop-up-Fenster gibst du im Reiter **Allgemein > Allgemeine Einstellungen** der Aufgabe einen individuellen Namen und wählst als Benutzer **root** aus. Anschließend entfernst du noch den Haken bei **Aktiviert**.
    - Füge anschließend im Reiter **Aufgabeneinstellungen > Befehl ausführen > Benutzerdefiniertes Script** das **Überwachungsscript** in das Textfeld ein...
    - Bestätige deine Eingaben mit die Schaltfläche **OK** und akzeptiere die anschließende Warnmeldung ebenfalls mit **OK**.
    - Damit die Aufgabe dem Aufgabenplaner hinzugefügt wird, musst du abschließend das Passwort deines aktuell am DSM angemeldeten Benutzers eingeben und den Vorgang über die Schaltfläche Senden bestätigen.
    - In der Übersicht des Aufgabenplaners kannst du nun die grade erstellte Aufgabe mit der Maus **markieren** (die Zeile sollte nach dem markieren blau hinterlegt sein) und die Aufgabe durch betätigen der Schaltfläche **Ausführen** einmalig manuell auslösen.

## Das Überwachungsscript
Bevor du das Überwachungsscript dem Aufgabenplaner hinzugügst, ersetzte die in Großbuchstaben geschriebenen Platzhalter...

- **BENUTZERNAME** ... mit dem Namen des gewünschen DSM-Benutzers z.B. **tommes**
- **ORDNERNAME**... mit dem Ordnernamen des verschlüsselten Ordners z.B. **Geheim**
- **PASSWORT**... mit dem Passwort, das zum einhängen des verschlüsselten Ordners benötigt wird
- **VERBINDEN**... mit einem Auslösewert zum einhängen des verschlüsselten Ordners z.B. **1**, **mount** oder **verbinden**
- **TRENNEN**... mit einem Auslösewert zum trennen des verschlüsselten Ordners z.B. **0**, **unmount** oder **trennen**
- **BEENDEN**... mit einem Auslösewert, um die Überwachung als auch das Überwachungsscript selbst zu beenden z.B. **100**, **exit** oder **beenden**
- **EVENTFILE**... mit einem frei definierbaren Dateinamen.

    **Hinweis:** Bitte keine Umlaute, Sonder- sowie Leerzeichen verwenden!!!

```
#!/bin/bash

# DSM Benutzername (Vertreten im Benutzer-Home-Ordner)
user="BENUTZERNAME"

# Name des verschlüsselten, freigegebenen Ordners
share="ORDNERNAME"

# Verschüsselungsschlüssel
password="PASSWORT"

# Auslösewert um einen verschlüsselten Ordner einzuhängen
decrypt_trigger="VERBINDEN"

# Auslösewert um einen verschlüsselten Ordner zu trennen
encrypt_trigger="TRENNEN"

# Auslösewert um die Überwachung zu beenden
exit_trigger="BEENDEN"

# Pfad und Dateiname der Ereignisdatei
eventpath=$(echo /volume[[:digit:]]/homes/${user})
eventfile=$(echo ${eventpath}/EVENTFILE.txt)

if [ -d "${eventpath}" ]; then
	if [ ! -f "${eventfile}" ]; then
	    touch "${eventfile}"
            chown "${user}":"users" "${eventfile}"
	    echo "value" > "${eventfile}"
	fi
else
	exit 1
fi

if [ -f "${eventfile}" ]; then
    echo "Überwachung läuft..."
    inotifywait -mq --format %w%f ${eventfile} -e close_write | while read event
    do
        if [ $(cat "${event}") == "${decrypt_trigger}" ]; then
            if [ -n "/volume[[:digit:]]/@${share}@" ] && [ -n "${password}" ]; then
                /usr/syno/sbin/synoshare --enc_mount ${share} ${password}
                echo "Der verschlüsselte Ordner [ ${share} ] wurde eingehängt."
            fi
        elif [ $(cat "${event}") == "${encrypt_trigger}" ]; then
            if [ -n "/volume[[:digit:]]/${share}" ]; then
                /usr/syno/sbin/synoshare --enc_unmount ${share}
                echo "Der verschlüsselte Ordner [ ${share} ] wurde getrennt."
            fi
        elif [ $(cat "${event}") == "${exit_trigger}" ]; then
            pid=$(ps aux | grep -v "grep" | grep -E "inotifywait.*--format.*${eventfile}.*close_write" | awk -F' ' '{print $2}')
            kill ${pid}
            exit 0     
        fi
    done
else
	exit 2
fi
```
