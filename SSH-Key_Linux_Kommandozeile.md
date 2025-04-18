[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)

# HowTo: SSH-Key auf der Linux Kommandozeile erstellen und einrichten

## Worum geht es?
Die folgende Anleitung beschreibt die Einrichtung einer **SSH-Public-Key-Authentifizierung** auf der **Kommandozeile** von einem lokalen Linux-Client-Betriebssystem zu einem Linux-basierten Remote-Server.

## Einleitung
Secure Shell, abgekürzt SSH, ist ein Netzwerkprotokoll zum Aufbau verschlüsselter Verbindungen zwischen Geräten im lokalen Netzwerk oder über das Internet. Bei der SSH-Public-Key-Authentifizierung wird mit Hilfe eines Schlüsselpaares, bestehend aus einem privaten und einem öffentlichen Schlüssel, eine passwortfreie Anmeldung zu einem Remote-Server aufgebaut, die bei Bedarf durch die Eingabe einer zusätzlichen Passphrase weiter abgesichert werden kann. Die Verwendung eines solchen Schlüsselpaares ist daher wesentlich schwieriger zu kompromittieren als die Eingabe eines Passwortes.

#### _Hinweis: Texte in Großbuchstaben innerhalb eckiger Klammern dienen als Platzhalter und müssen durch eigene Angaben ersetzt werden, können aber an einigen Stellen auch nur der Information dienen. Es ist zu beachten, dass die eckigen Klammern Teil des Platzhalters sind und beim Ersetzen durch eigene Angaben ebenfalls entfernt werden müssen._

## Terminal öffnen
Zuerst benötigst du ein Terminalprogramm deiner Wahl, um auf die Kommandozeile deines Client-Betriebssystems zugreifen zu können.

1. Nach dem Start des Terminalprogramms solltest du dich in deinem eigenen Benutzer-Home-Verzeichnis befinden. Dies kannst du überprüfen, indem du den Befehl **pwd** _(steht für "print working directory")_ eingibst und die Eingabetaste drückst.

    ```
    [BENUTZERNAME]@[CLIENT-PC]:~$ pwd
    /home/[BENUTZERNAME]

    ```

    Diesem Beispiel folgend befindest du dich also im Home-Verzeichnis des Benutzers [BENUTZERNAME] (korrekterweise müsste hier natürlich dein Benutzername stehen) und genau dort solltest du dich auch befinden. Je nachdem, mit welchen Linux Betriebssystem du arbeitest, kann hier ein abweichender Pfad ausgegeben werden. Bei Verwendung eines Synology NAS wäre der Pfad zum Home-Verzeichnis des aktuell angemeldeten Benutzers beispielsweise `/var/services/homes/[BENUTZERNAME]`.

    **Hinweis:** Zur besseren Lesbarkeit werden alle folgenden Eingaben ohne die Verwendung des Pompts `[BENUTZERNAME]@[CLIENT-PC]:~$` dargestellt.

## SSH Verzeichnisstruktur erstellen

1. Zuerst wird ein neues verstecktes Verzeichnis mit dem Namen **.ssh** im Home-Verzeichnis des aktuell angemeldeten Benutzers angelegt.

       mkdir .ssh

## SSH Schlüsselpaar erstellen
1. Du kannst nun beginnen, ein neues **SSH-Schlüsselpaar** zu erzeugen, das aus einem **öffentlichen** und einem **privaten Schlüssel** besteht. Dies geschieht mit dem Befehl **ssh-keygen**. Durch die einfache Eingabe des Befehls ssh-keygen können je nach Konfiguration verschiedene Verschlüsselungsalgorithmen zur Anwendung kommen. Daher sollte immer angegeben werden, welcher Algorithmus verwendet werden soll. Im Folgenden wird der **RSA**-Algorithmus mit einer Verschlüsselung von **4096** Bit verwendet.

       ssh-keygen -t rsa -b 4096

    _(**-t** steht für type, also den Verschlüsselungstyp, hier rsa für RSA Protokoll 2)_

    _(**-b** steht für Bit, also die Bitlänge des Verschlüsselungsschlüssels, hier 4096 Bit)_

2. Unmittelbar nach dem Ausführen des Befehls wirst du gefragt, unter welchem Dateinamen die SSH-Schlüssel gespeichert werden sollen. Wenn du den vorgeschlagenen Pfad und Dateinamen beibehalten möchtest, drücke zur Bestätigung einfach die Eingabetaste. Andernfalls kannst du unter Verwendung des vollständigen Pfads einen alternativen Dateinamen ohne Dateiendung eingeben. In diesem Beispiel wird der vorgeschlagene Dateiname verwendet.

	```
	Generating public/private rsa key pair.
    Enter file in which to save the key (/home/[BENUTZERNAME]/.ssh/id_rsa):
    Created directory '/home/[BENUTZERNAME]/.ssh'.
	```

3. Anschließend wirst du aufgefordert, eine Passphrase, also ein Passwort, einzugeben. Die Verwendung einer Passphrase wird einerseits dringend empfohlen, da sie verhindert, dass der private Schlüssel einfach kopiert und verwendet werden kann, selbst wenn dein Client-Betriebssystem kompromittiert wurde. Der Nachteil einer Passphrase ist, dass du sie jedes Mal eingeben musst, wenn du eine Verbindung über SSH herstellen möchtest. Dies kann kontraproduktiv sein, wenn man später unbeaufsichtigte und damit automatisierte SSH-Verbindungen aufbauen möchte, um z.B. automatisierte Backups durchzuführen. In diesem Fall ist von der Eingabe einer Passphrase dringend abzuraten. Es bleibt also jedem selbst überlassen, ob er eine Passphrase verwenden möchte oder nicht. In diesem Beispiel wird keine Passphrase verwendet, sondern die beiden folgenden Abfragen werden einfach durch Drücken der Eingabetaste übersprungen.

	```
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	```

	In der Folge erscheint eine ähnliche Ausgabe wie diese...

	```
	Your identification has been saved in /home/[BENUTZERNAME]/.ssh/id_rsa
    Your public key has been saved in /home/[BENUTZERNAME]/.ssh/id_rsa.pub
    The key fingerprint is:
	SHA256:ScteBU5B3/m7/cod5OqlH8JSDpiNC8oGGYx0sEhoHFM [BENUTZERNAME]@[CLIENT-PC]
	The key's randomart image is:
	+---[RSA 4096]----+
	|o*+E     .=.     |
	|=o*      o o . . |
	|oo o    . . o o  |
	|    o  o o=.   . |
	|   o   .S+.o . ..|
	|    o ..... = o .|
	|     +  .. . + * |
	|    .       ..=.=|
	|            .+++=|
	+----[SHA256]-----+
	```

4. Nach der Ausführung befindet sich nun im Verzeichnis `/home/[BENUTZERNAME]/.ssh/` eine Datei mit dem Namen **id_rsa**, die den **privaten** Schlüssel enthält und eine Datei mit dem Namen **id_rsa.pub**, die den **öffentlichen** Schlüssel enthält.

5. Wechsel mit **cd** _(steht für "change Directory")_ in dieses Verzeichnis

       cd .ssh

6. Der Prompt sollte an dieser Stelle das Verzeichnis .ssh als Bestätigung enthalten und etwa so aussehen

       [BENUTZERNAME]@[CLIENT-PC]:~/ssh$

7. Der Inhalt des Verzeichnisses kann mit dem Befehl **ls** _(steht für "list")_, ggf. gefolgt von weiteren Optionen, angezeigt werden.

       ls -la

    _(Option **l** = Datei-Informationen werden in Langform ausgegeben)_

    _(Option **a** = Zeigt auch versteckte Dateien und Ordner an, die mit einem Punkt beginnen)_

    **Beispiel für die Ausgabe**

    ```
    [BENUTZERNAME]@[CLIENT-PC]:~/ssh$ ls -la
    total 16
    drwxrwxr-x 2 [BENUTZERNAME] [BENUTZERNAME] 4096 Dez 31 07:00 .
    drwxr-x--- 8 [BENUTZERNAME] [BENUTZERNAME] 4096 Dez 31 07:00 ..
    -rw------- 1 [BENUTZERNAME] [BENUTZERNAME] 3389 Dez 31 07:00 id_rsa
    -rw-r--r-- 1 [BENUTZERNAME] [BENUTZERNAME]  746 Dez 31 07:00 id_rsa.pub
    ```

    **Hinweis:** Du kannst dir jederzeit den Inhalt des Verzeichnisses anzeigen lassen, in dem du dich gerade befindest. In Verbindung mit weiteren Optionen können mehr oder weniger Informationen angezeigt oder die Lesbarkeit der Ausgabe verbessert werden.


8. Bei Bedarf kannst du dir den Inhalt des öffentlichen Schlüssels mit dem Befehl **cat** _(steht für "concatenate", also Dateiinhalte verknüpfen)_ auf der Kommandozeile anzeigen lassen. Wenn du den öffentlichen Schlüssel an andere Personen weitergeben möchtest, kannst du den Schlüssel auch kopieren und in einer separaten Textdatei speichern.

       cat id_rsa.pub

    **Beispiel für die Ausgabe**

    ```
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCbTJMv9N+pU8n7SaDa9RLazZwB+cR0ke2ggq/Xbria3fsuLGG/wrd5mOt1KYSg7FWiiHUSec0s1sX0OAbWk8qX47UaJKe7SP6so39kVjkRGINACsojvYI773LBpDuKmGzBqsTpBSlYLGLtA7rs669YSEV+8M6Z7NJj6YySDRBL/TSx/cjiJyh5h6yHwW/+nqzFBteqzVoBQ241nGdVzcb5S16DnJLhn2MB7es70yOzq02y5XpFSPswme6sH5/jIkNhp1OU2++jPxLNOkcN73WB3pc3jz7bZxkBSh7zJxy1o39kVjkRGISbznxxbIGDn20iAFoLF6IHl0mWhiuYVwSRjZ8M1Kf71qYv7X3RHMrKQQM5qYtpk/R0NdGrvemDP/5gAI0wOG1D/RQD8nBHzHwuNvLGLtA7rs66tcfj1g+jwwMPzEgX5MsisbIKfhwLJZE34zyfWDj/Vl5n44yG7/YDkYmxtPRsYbfLUm5D+q5dhm5+g3H8t0/eQXHel7chM5I8j4pElDhFOCF8jeaCeawxAGrRn6Dab93BHF1wwowOp4kVWKw0lnvTQfbcIc5BSrxR2uYfQk9dwiiDeE6npEN11mTezRN/tsZ6JE4OrPL8oGha7Fa73AIMvyo+/kKsl2wMpBHYRZq70dp0MTUfIAQPAvNM31yNY4B8oIKV+wBxXw== [BENUTZERNAME]@[CLIENT-PC]
    ```

## Die Datei authorized_keys erstellen und den eigenen öffentlichen Schlüssel hinzufügen
1. In der Datei **authorized_keys** werden später neben dem eigenen Schlüssel auch alle öffentlichen Schlüssel bekannter Remote-Server eingetragen, die auf dein Client-Betriebssystem zugreifen wollen, um eine passwortlose SSH-Verbindung zu ermöglichen. Der folgende Befehl trägt den eigenen öffentlichen Schlüssel in authorized_keys ein. Wenn die Datei noch nicht existiert, wird sie automatisch erstellt.

       cat id_rsa.pub >> ~/.ssh/authorized_keys

    _(Befehl **>** Datei = Standardausgabe des Befehls in Zieldatei (über-)schreiben.)_

    _(Befehl **>>** Datei = Standardausgabe des Befehl an Zieldatei anhängen.)_

    **Hinweis:** Das **Tilde**-Zeichen _(**~**)_ im Linux-Dateisystem steht für das Home-Verzeichnis des aktuellen Benutzers. Dies ist eine Abkürzung, die es Benutzern ermöglicht, auf ihr eigenes Home-Verzeichnis zuzugreifen, ohne den vollständigen Pfad eingeben zu müssen. Der Pfad `~/.ssh/authorized_keys` ist also identisch mit dem Pfad `/home/[BENUTZERNAME]/.ssh/authorized_keys`.

## Ordner und Dateirechte anpassen
1. Es ist wichtig, dass sowohl das Verzeichnis .ssh als auch die darin enthaltenen Dateien bestimmte Berechtigungen haben, um den Zugriff durch Unbefugte einzuschränken und die Sicherheit zu erhöhen. Der Befehl **chmod** _(steht für "change mode")_ verändert die Zugriffsrechte von Ordnern und Dateien, die wie folgt lauten.

       chmod 700 ~/.ssh

       chmod 600 ~/.ssh/id_rsa*

       chmod 600 ~/.ssh/authorized_keys

## Übertragen des öffentlichen Schlüssels auf den Remote-Server
Um in Zukunft eine passwortfreie Anmeldung an einem Remote-Server zu ermöglichen, ist es notwendig, den öffentlichen Schlüssel auf den entfernten Server zu kopieren. Der öffentliche Schlüssel kann auch auf weitere Remote-Server kopiert werden, mit denen man sich verbinden möchte, während der private Schlüssel immer auf dem lokalen Linux Betriebssystem verbleibt und niemals weitergegeben wird. Das Kopieren des öffentlichen Schlüssels kann auf verschiedene Arten erfolgen, von denen hier nur zwei näher erläutert werden, nämlich die unter Verwendung des Befehls **ssh-copy-id** und die unter Verwendung des Befehls **cat**. Eine weitere Möglichkeit ist die Verwendung des **vi-Editors** auf der Kommandozeile des Remote-Servers, die ich bereits im meinem [HowTo: SSH-Key mit PuTTY und PuTTYgen](https://github.com/toafez/HowTo_Windows.SSH.PuTTY.PuTTYgen) im Abschnitt [Kopieren des öffentlichen Schlüssels auf den Remote-Server](https://github.com/toafez/HowTo_Windows.SSH.PuTTY.PuTTYgen#kopieren-des-%C3%B6ffentlichen-schl%C3%BCssels-auf-den-remote-server) beschrieben habe.

### 1. Möglichkeit: Die Verwendung des Befehls `ssh-copy-id`
  Der Befehl **ssh-copy-id** ist Bestandteil des Programmpakets OpenSSH und sollte daher auf den meisten unixoiden Betriebssystemen verfügbar sein. Eine Ausnahme bildet hier z.B. die Firma Synology, die auf ihren NAS-Systemen das Betriebssystem DiskStation Manager (DSM) einsetzt. Hier sucht man den Befehl ssh-copy-id vergeblich (Stand DSM 7.2.2). In diesem Fall ist es ratsam, mit der [2. Möglichkeit](https://github.com/toafez/HowTo_Linux.SSH.Terminal?tab=readme-ov-file#2-m%C3%B6glichkeit-die-verwendung-des-befehls-cat) fortzufahren, die in den nachfolgenden Abschnitt beschrieben wird.

  ssh-copy-id verbindet sich nach dem Aufruf mit dem Remote-Server und prüft, ob im Home-Verzeichnis des angemeldeten Benutzers bereits ein Verzeichnis mit dem Namen **.ssh** existiert. Wenn nicht, wird es angelegt. Im nächsten Schritt prüft ssh-copy-id, ob im Verzeichnis .ssh eine Datei mit dem Namen **authorized_keys** existiert. Falls nicht, wird diese Datei ebenfalls angelegt. Danach wird der öffentliche Schlüssel in die Datei authorized_keys kopiert. Zum Schluss werden noch die entsprechenden Verzeichnis- und Dateirechte gesetzt, bevor die Verbindung wieder beendet wird.

  1. Um dich mit deinem Remote Server verbinden zu können, benötigst du neben der IP-Adresse und dem Port auch den Benutzernamen sowie das zugehörige Passwort. Ersetze daher im folgenden Befehl die Platzhalter für [PORT], [BENUTZERNAME] und [IP-ADRESSE] durch deine eigenen Daten. Außerdem benötigst du noch den Speicherort und den Dateinamen des öffentlichen Schlüssels, der sich in diesem Beispiel im Verzeichnis `/home/[BENUTZERNAME]/.ssh/` unter dem Namen `id_rsa.pub` bzw. unter `~/.ssh/id_rsa.pub` auf deinen lokalen Linux Betriebssystem befindet. Führe anschließend folgenden Befehl aus.

         ssh-copy-id -i ~/.ssh/id_rsa.pub -p [PORT] [BENUTZERNAME]@[IP-ADRESSE]

  2. Nachdem du den Befehl mit der Eingabetaste bestätigt hast, solltest du eine Meldung sehen, die dich am Ende des Textes auffordert, den Verbindungsaufbau mit **yes** zu bestätigen. `Are you sure you want to continue connecting (yes/no/[fingerprint])?` Diese Meldung erscheint nur, wenn du dich zum ersten Mal per SSH mit dem Remote-Server verbinden willst. Dabei wird ein sogenannter **Fingerprint** in der Datei `~/.ssh/known_hosts` auf deinem Client-Betriebssystem hinterlegt, um zukünftige Verbindungen zu erlauben.

      ```
      The authenticity of host '[IP-ADRESSE] ([IP-ADRESSE])' can't be established.
      ED25519 key fingerprint is SHA256:DosguIv2tIYP+9n3BIrSM2df1841CdXbIc4pAZ01ehA.
      This key is not known by any other names.
      Are you sure you want to continue connecting (yes/no/[fingerprint])?
      ```

  3. Nun wirst du aufgefordert dein Passwort für die Anmeldung an deinem Remote-Server einzugeben.

         [BENUTZERNAME]@[IP-ADRESSE]'s password:

  4. Nach erfolgreicher Anmeldung beginnt der Befehl ssh-copy-id mit der Ausführung der oben beschriebenen Aufgaben. Nach Abschluss der Arbeiten wird die Verbindung automatisch getrennt und zur Kommandozeile des Client-Betriebssystems zurückgekehrt.

  5. Überspringe die zweite Möglichkeit und fahre mit Abschnitt [Erneute SSH-Verbindung zu deinem Remote-Server aufbauen](https://github.com/toafez/HowTo_Linux.SSH.Terminal?tab=readme-ov-file#erneute-ssh-verbindung-zu-deinem-remote-server-aufbauen) fort. 

### 2. Möglichkeit: Die Verwendung des Befehls `cat`
  Sollte der Befehl ssh-copy-id nicht zur Verfügung stehen, wäre der Befehl cat das Mittel der Wahl. Da dieser Befehl allein jedoch keine Verzeichnisse und/oder Dateien auf dem Remote-Server anlegt und auch keine Verzeichnis- und Dateirechte vergibt, müssen diese Schritte manuell mit verschiedenen Befehlen durchgeführt werden. Der Befehl cat kopiert lediglich den öffentlichen Schlüssel des Client-Betriebssystems in die authorized_keys des Remote-Servers. Im Folgenden werden alle Schritte nacheinander abgearbeitet.

  1. Um dich mit deinem Remote Server verbinden zu können, benötigst du neben der IP-Adresse und dem Port auch den Benutzernamen sowie das zugehörige Passwort. Ersetze daher im folgenden Befehl die Platzhalter für [PORT], [BENUTZERNAME] und [IP-ADRESSE] durch deine eigenen Daten. Führe anschließend folgenden Befehl aus

         ssh -p [PORT] [BENUTZERNAME]@[IP-ADRESSE]

  2. Nachdem du den Befehl mit der Eingabetaste bestätigt hast, solltest du eine Meldung sehen, die dich am Ende des Textes auffordert, den Verbindungsaufbau mit **yes** zu bestätigen. `Are you sure you want to continue connecting (yes/no/[fingerprint])?` Diese Meldung erscheint nur, wenn du dich zum ersten Mal per SSH mit dem Remote-Server verbinden willst. Dabei wird ein sogenannter **Fingerprint** in der Datei `~/.ssh/known_hosts` auf deinem Client-Betriebssystem hinterlegt, um zukünftige Verbindungen zu erlauben.

      ```
      The authenticity of host '[IP-ADRESSE] ([IP-ADRESSE])' can't be established.
      ED25519 key fingerprint is SHA256:DosguIv2tIYP+9n3BIrSM2df1841CdXbIc4pAZ01ehA.
      This key is not known by any other names.
      Are you sure you want to continue connecting (yes/no/[fingerprint])?
      ```

  3. Nun wirst du aufgefordert dein Passwort für die Anmeldung an deinem Remote-Server einzugeben.

         [BENUTZERNAME]@[IP-ADRESSE]'s password:

  4. Nach erfolgreicher Anmeldung solltest du dich auf der Kommandozeile deines Remote-Servers befinden. Überprüfe auch hier, ob du dich im Home-Verzeichnis des angemeldeten Benutzers befindest.

       ```
      [BENUTZERNAME]@[REMOTE-SERVER]:~$ pwd
      /home/[BENUTZERNAME]
      ```

  5. Erstelle auch hier ein neues verstecktes Verzeichnis mit dem Namen .ssh und weise diesem Verzeichnis die notwendigen Berechtigungen zu.

         mkdir .ssh

         chmod 700 .ssh

  6. Melde dich wieder vom Remote-Server ab, um zur Kommandozeile deines Client-Betriebssystems zurückzukehren.

         exit

  7. Nun kann der Inhalt der Datei id_rsa.pub und damit der öffentliche Schlüssel über eine SSH-Verbindung auf den Remote-Server in die Datei authorized_keys übertragen werden, die ggf. angelegt wird, wenn sie noch nicht existiert. Ersetze dafür im folgenden Befehl die Platzhalter für [PORT], [BENUTZERNAME] und [IP-ADRESSE] durch deine eigenen Daten, ohne die eckigen Klammern zu verwenden. Führe anschließend folgenden Befehl aus

         cat ~/.ssh/id_rsa.pub | ssh -p [PORT] [BENUTZERNAME]@[IP-ADRESSE] "cat >> ~/.ssh/authorized_keys"

  8. Nach erfolgreicher Anmeldung durch die Eingabe des Passwortes wurde der öffentliche Schlüssel in der Datei ~/.ssh/authorized_keys deines Remote-Servers gespeichert und die Verbindung wieder getrennt. Du befindest dich weiterhin auf der Kommandozeile deines lokalen Linux Betriebssystems.


## Erneute SSH-Verbindung zu deinem Remote-Server aufbauen
1. Ab jetzt solltest du dich am Remote Server anmelden können, ohne ein Passwort eingeben zu müssen. Um zu überprüfen, ob dies funktioniert, melde dich erneut am Remote-Server an.

       ssh -p [PORT] [BENUTZERNAME]@[IP-ADRESSE]

2. Wenn alles geklappt hat, solltest du dich nun auf der Kommandozeile deines Remote-Servers befinden, ohne ein Passwort eingegeben zu haben (es sei denn, du verwendest eine der oben genannten Passphrase). Und wenn du schon dabei bist, kannst du, solltrst du den Weg über die 2 Möglichkeit gegangen sein, auch die Berechtigungen für die Datei authorized_keys korrigieren, indem du folgenden Befehl eingibst

       chmod 600 ~/.ssh/authorized_keys

3. Du hast nun erfolgreich eine passwortlose SSH-Schlüssel-Authentifizierung eingerichtet.

[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)