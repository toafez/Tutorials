# HowTo: SSH-Key mit PuTTYgen erstellen und in PuTTY einrichten

## Worum geht es?
Die folgende Anleitung beschreibt die Einrichtung einer **SSH-Public-Key-Authentifizierung** mit Hilfe der Programme **PuTTYgen** und **PuTTY** von einem lokalen Linux-Client-Betriebssystem zu einem Linux-basierten Remote-Server.

## Einleitung
Secure Shell, abgekürzt SSH, ist ein Netzwerkprotokoll zum Aufbau verschlüsselter Verbindungen zwischen Geräten im lokalen Netzwerk oder über das Internet. Bei der SSH-Public-Key-Authentifizierung wird mit Hilfe eines Schlüsselpaares, bestehend aus einem privaten und einem öffentlichen Schlüssel, eine passwortfreie Anmeldung zu einem Remote-Server aufgebaut, die bei Bedarf durch die Eingabe einer zusätzlichen Passphrase weiter abgesichert werden kann. Die Verwendung eines solchen Schlüsselpaares ist daher wesentlich schwieriger zu kompromittieren als die Eingabe eines Passwortes.

## PuTTY und PuTTYgen installieren
Um unter Windows einen RSA-Key zu erzeugen und sich anschließend über SSH mit einem Remote-Server verbinden zu können, benötigst du den PuTTY-Client und das darin enthaltene Programm PuTTYgen. Beide Programme können als Windows-Installer von der [PuTTY Download-Seite](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) heruntergeladen werden.

## Öffentlichen und privaten Schlüssel mit PuTTYgen erzeugen
1. Starte **PuTTYgen** aus der Liste der Windows-Programme.
2. Wähle im Abschnitt **Parameters / Typ of key to generated** die Option **RSA** aus.
3. Im Abschnitt **Parameters / Number of bits in a generated key** wird die Anzahl der Bits auf mindestens **2048**, besser auf **4096** eingestellt.
4. Klicke anschließend im Abschnitt **Actions / Generate a public/private key pair** auf die Schaltfläche **Generate**.

    ![SSH-Key_PuTTYgen_PuTTY_010](/images/SSH-Key_PuTTYgen_PuTTY_010.png)

5. Du wirst nun aufgefordert, den Mauszeiger als Zufallsgenerator über das Fenster des PuTTY Schlüsselgenerators zu bewegen, um den öffentlichen und den privaten Schlüssel zu erzeugen.

    ![SSH-Key_PuTTYgen_PuTTY_020](/images/SSH-Key_PuTTYgen_PuTTY_020.png)

6. Sobald der Prozess abgeschlossen ist und die Schlüsselinformationen angezeigt werden, kann im Abschnitt **Key / Key comment:** ein eindeutiger Name für den soeben erstellten Schlüssel angegeben werden. In diesem Beispiel nennen wir den Schlüssel **puttygen-rsa-key**.
7.  Die Verwendung einer Passphrase im Abschnitt **Key / Key passphrase:** wird einerseits dringend empfohlen, da sie verhindert, dass der private Schlüssel einfach kopiert und verwendet werden kann, selbst wenn dein Client-Betriebssystem kompromittiert wurde. Der Nachteil einer Passphrase ist, dass du sie jedes Mal eingeben musst, wenn du eine Verbindung über SSH herstellen möchtest. Dies kann kontraproduktiv sein, wenn man später unbeaufsichtigte und damit automatisierte SSH-Verbindungen aufbauen möchte, um z.B. automatisierte Backups durchzuführen. In diesem Fall ist von der Eingabe einer Passphrase dringend abzuraten. Es bleibt also jedem selbst überlassen, ob er eine Passphrase verwenden möchte oder nicht. In diesem Beispiel wird keine Passphrase verwendet, sondern die beiden folgenden Abfragen werden einfach durch Drücken der Eingabetaste übersprungen.
8. Markiere anschließend den öffentlichen Schlüssel im Abschnitt **Key / Public key for pasting into OpenSSH authorized_keys file**. Klicke mit der rechten Maustaste auf den markierten Text, kopiere den öffentlichen Schlüssel in die Zwischenablage, öffne einen Texteditor deiner Wahl, füge den öffentlichen Schlüssel in die Datei ein und speichere die Textdatei unter einem Namen deiner Wahl an einem sicheren Ort.
9. Speichere anschließend den privaten Schlüssel an demselben sicheren Ort, indem du im Bereich **Actions / Save the generated key** auf die Schaltfläche **Safe private key** klickst. Auch hier ist der Dateiname frei wählbar, erhält aber die Endung **.ppk**. In diesem Beispiel lautet der Dateiname puttygen-rsa-key.ppk.

    ![SSH-Key_PuTTYgen_PuTTY_030](/images/SSH-Key_PuTTYgen_PuTTY_030.png)

## Verbindung zum Remote Server mit PuTTY herstellen
1. Starte **PuTTY** aus der Liste der Windows-Programme.
2. Trage im Abschnitt **Specify the destination you want to connect to / Host Name (or IP address)** und **Port** die IP-Adresse des Remote-Servers ein, mit dem du dich verbinden möchtest und passe ggf. den zu verwendenden Port an. Dieses Beispiel verwendet die IP-Adresse 172.16.1.12 und den Port 22.
3. Wähle im Abschnitt **Specify the destination you want to connect to / Connection type** die Option **SSH** aus.
4. Klicke anschließend auf die Schaltfläche **Open**, um eine Verbindung zu deinem Remote-Server aufzubauen.

    ![SSH-Key_PuTTYgen_PuTTY_040](/images/SSH-Key_PuTTYgen_PuTTY_040.png)

5. Beim ersten Verbindungsaufbau erhält man einen **„Security Alert“** und wird aufgefordert, die Verbindung zu akzeptieren, um einen ersten Handshake durchzuführen, bei dem ein sogenannter Fingerprint für den zukünftigen Abgleich hinterlegt wird. Klicke daher auf die Schaltfläche **Accept**.

    ![SSH-Key_PuTTYgen_PuTTY_050](/images/SSH-Key_PuTTYgen_PuTTY_050.png)

6. Nun sollte sich ein Terminalfenster öffnen, in das du deinen Benutzernamen und dein Passwort eingeben musst, um dich an der Konsole deines Remote-Servers anzumelden. In diesem Beispiel wird der Benutzername **tommes** verwendet, um sich an einem fiktiven **Ubuntu-Server** anzumelden.

    ```
    login as: tommes
    tommes@172.16.1.12's password:
    ```

7. Nach erfolgreicher Anmeldung sollte nach einer kurzen Begrüßung und ggf. weiteren Informationen die Eingabeaufforderung bzw. der Prompt erscheinen.

   ```
   tommes@Ubuntu-Server:~$
   ```

8. Normalerweise solltest du dich nun in deinem eigenen Benutzer-Home-Verzeichnis befinden. Du kannst dies überprüfen, indem du den Befehl **pwd** (steht für "print working directory") eingibst und die Eingabetaste drückst.

    ```
    tommes@Ubuntu-Server:~$ pwd
    /home/tommes
    ```

   Diesem Beispiel folgend befinden wir uns also im Home-Verzeichnis des Benutzers tommes (korrekterweise müsste hier natürlich dein Benutzername stehen) und genau dort wollen wir auch sein. Je nachdem, zu welchem Remote-Server-System eine Verbindung aufgebaut wird, kann hier ein abweichender Pfad ausgegeben werden. Bei Verwendung eines Synology NAS wäre der Pfad zum Home-Verzeichnis des Benutzers beispielsweise /var/services/homes/tommes.

    **Anmerkung:** Wenn das verwendete System eine direkte Authentifizierung mit Schlüssel zum Superuser **root** zulässt, kann an dieser Stelle zum root-Konto gewechselt werden, um den öffentlichen Schlüssel zu kopieren. Dies geschieht durch Eingabe des Befehls `su -`, `sudo -s` bzw. `sudo -i` und Eingabe eines Passworts.


## Kopieren des öffentlichen Schlüssels auf den Remote-Server
Zur besseren Lesbarkeit werden alle folgenden Eingaben ohne die Verwendung des Pompts `tommes@Ubuntu-Server:~$` dargestellt.

1. Zuerst wird ein neues verstecktes Verzeichnis mit dem Namen .ssh im Home-Verzeichnis des aktuell angemeldeten Benutzers angelegt.

    `mkdir .ssh`

2. Anschließend wird das soeben erstellte Verzeichnis mit den entsprechenden Rechten versehen.

    `chmod 700 .ssh`

3. Wechsel nun in dieses Verzeichnis

    `cd .ssh`

4. Der Prompt sollte an dieser Stelle das Verzeichnis als Bestätigung enthalten, was in etwa so aussehen sollte

    `tommes@Ubuntu-Server:~/ssh$`

5. Weiter geht es mit dem vi-Editor der beim Aufruf die Datei authorized_keys erstellen bzw. bearbeiten soll.

    `vi authorized_keys`

    **Anmerkung:** An dieser Stelle wird es etwas knifflig, denn der vi-Editor ist zwar einer der beliebtesten Texteditoren unter Linux, aber in der Bedienung sehr gewöhnungsbedürftig. Daher sollten die folgenden Anweisungen genau befolgt werden. Wenn man sich irgendwo vertippt hat, sollte man am besten durch Drücken der **Escape-** bzw, **Esc**-Taste in den Befehlsmodus zurückkehren und das Dokument schließen, ohne zu speichern. Dies geschieht durch Eingabe von `:q!` Danach einfach wieder von vorne anfangen.

 6. Zunächst benötigst du noch den öffentlichen Schlüssel, den du am Anfang dieser Anleitung aus PuTTYgen kopiert und in einer Textdatei gespeichert hast. Öffne die Datei, markiere den öffentlichen Schlüssel, klicke mit der rechten Maustaste auf den markierten Text und kopiere den öffentlichen Schlüssel erneut in die Zwischenablage. Wechsle dann zurück zum Terminalfenster, um im vi-Editor vom Befehlsmodus in den Eingabemodus zu wechseln. Gib dazu einfach ein...

    `i`

7. ... ein, bewege den Mauszeiger in das Terminalfenster und drücke die rechte Maustaste. Dadurch wird der öffentliche Schlüssel aus der Zwischenablage in die geöffnete Datei authorized_keys des vi-Editors eingefügt. Das Ganze sollte ungefähr so aussehen...

    ![SSH-Key_PuTTYgen_PuTTY_110](/images/SSH-Key_PuTTYgen_PuTTY_110.png)

8. Wenn alles gut aussieht, drücke die **Escape-** bzw, **Esc**-Taste um zurück in den Befehlsmodus wechseln und beende den vi-Editor durch die Eingabe von...

    `:qw!`

9. Du solltest dich wieder auf der Konsole wiederfinden. Beende die Terminalsitzung indem du...

    `exit`

  ... eingibs. Das Terminalfenster sollte sich automtisch schließen.

## Einrichten von PuTTY für die Authentifizierung mit Schlüsseln
1. Starte **PuTTY** erneut aus der Liste der Windows-Programme.
2. Trage im Abschnitt **Specify the destination you want to connect to / Host Name (or IP address)** und **Port** die IP-Adresse des Remote-Servers ein, mit dem du dich verbinden möchtest und passe ggf. den zu verwendenden Port an. Dieses Beispiel verwendet die IP-Adresse 172.16.1.12 und den Port 22.
3. Wähle im Abschnitt **Specify the destination you want to connect to / Connection type** die Option **SSH** aus.

    ![SSH-Key_PuTTYgen_PuTTY_060](/images/SSH-Key_PuTTYgen_PuTTY_060.png)

4. Gehe im linken Abschnitt **Category:** zum Punkt **Connection / Data** und trag im rechten Abschnitt unter **Login details / auto-login username** den Benuternamen ein, mit dem du dich auf deinen Remote-Server verbinden möchtest. Wenn das verwendete System eine direkte Authentifizierung mit Schlüssel zum Superuser **root** zulässt, kann an dieser Stelle auch **root** eingetragen werden.
5. Wähle im gleichen Abschnitt unter **When username is not specified** die Option **Prompt** aus.

    ![SSH-Key_PuTTYgen_PuTTY_070](/images/SSH-Key_PuTTYgen_PuTTY_070.png)

6. Gehe im linken Abschnitt **Category:** zum Punkt **Connection / SSH / Auth / Credentials** und trage im rechten Abschnitt **Public-key authenification / Private key file for authentification:** den Pfad zu deinem **privaten** Schlüssel ein, den du am Anfang mit PuTTYgen erstellt und gespeichert hast. Benutze dazu am besten die Schaltfläche **Browse**, um die entsprechende Datei mit der Endung .ppk auszuwählen. In diesem Beispiel lautet der Dateiname puttygen-rsa-key.ppk.

    ![SSH-Key_PuTTYgen_PuTTY_080](/images/SSH-Key_PuTTYgen_PuTTY_080.png)

7. Gehe im linken Abschnitt **Category:** zurück zum Punkt **Session** und trag im rechten Abschnitt unter **Load, save or delete a stored session / Saved Sessions** einen Namen für die Verbinung ein und klicke anschnießend auf **Save**

    ![SSH-Key_PuTTYgen_PuTTY_090](/images/SSH-Key_PuTTYgen_PuTTY_090.png)

8. Damit hast du in Zukunft die Möglichkeit, die soeben erstellte Konfiguration innerhalb von PuTTY auszuwählen, um dich mit deinem Remote-Server per Schlüsselaustausch zu authentifizieren.

    ![SSH-Key_PuTTYgen_PuTTY_100](/images/SSH-Key_PuTTYgen_PuTTY_100.png)

