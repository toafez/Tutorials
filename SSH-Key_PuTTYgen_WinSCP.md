[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)

# HowTo: SSH-Key mit PuTTYgen erstellen und in WinSCP einrichten

## Worum geht es?
Die folgende Anleitung baut auf meiner Anleitung [SSH-Key mit PuTTYgen erstellen und in PuTTY einrichten](https://github.com/toafez/Tutorials/blob/main/SSH-Key_PuTTYgen_PuTTY.md) auf, in der bereits eine passwortfreie SSH-Public-Key-Authentifizierung von einem lokalen Linux-Client-Betriebsystem zu einem Linux-basierten Remote-Server beschrieben wurde. **Durch die Einbindung des** bereits mit PuTTYgen erzeugten **privaten Schlüssels in das Windows-Programm WinSCP kann** auch hier **ein passwortfreier Zugriff auf einen entfernten Linux-Server erfolgen**.

## WinSCP installieren
Das Programm WinSCP kann als Windows-Installer von der [WinSCP Download-Seite](https://winscp.net/eng/download.php) heruntergeladen werden.

## WinSCP öffnen
1. Starte **WinSCP** aus der Liste der Windows-Programme.
2. Wähle im linken Abschnitt **Sitzung /Übertragungsprotokoll** den Punkt **SCP** aus dem Aufklappmenü aus.
3. Gib in den Abschnitten **Session / Server Address** und **Port Number** die IP-Adresse des Remote-Servers ein, mit dem du dich verbinden möchtest und passe ggf. den zu verwendenden Port an. In diesem Beispiel wird die IP-Adresse 172.16.1.12 und der Port 22 verwendet.
4.  Gib im Abschnitt **Session / Benutzername** den Benutzernamen ein, mit dem du dich auf dem Remote-Server anmelden möchtest. In diesem Beispiel wird der Superuser **root** verwendet.
5. Klicke auf die Schaltfläche **Erweitert**

    ![SSH-Key_PuTTYgen_WinSCP_010](/images/SSH-Key_PuTTYgen_WinSCP_010.png)

6. Wähle im rechten Abschnitt **SSH / Authentifizierung** und klicke dann im rechten Bereich **Authentifizierungsparameter / Datei mit privatem Schlüssel:** auf die gelb markierte Schaltfläche, um im Dateisystem nach dem privaten Schlüssel zu suchen, den du bereits nach meiner Anleitung [SSH-Key mit PuTTYgen erstellen und in PuTTY einrichten](https://github.com/toafez/Tutorials/blob/main/SSH-Key_PuTTYgen_PuTTY.md) erstellt und an einem sicheren Ort gespeichert hast.

    ![SSH-Key_PuTTYgen_WinSCP_020](/images/SSH-Key_PuTTYgen_WinSCP_020.png)

7. Wähle die passende Datei aus und klicke auf **Öffnen**

    ![SSH-Key_PuTTYgen_WinSCP_030](/images/SSH-Key_PuTTYgen_WinSCP_030.png)

8. Der Pfad inklusive Dateiname sollte nun im gelb markierten Textfeld angezeigt werden. Klicke auf die Schaltfläche **OK** um zur Anmeldeseite von WinSCP zurückzukehren.

    ![SSH-Key_PuTTYgen_WinSCP_040](/images/SSH-Key_PuTTYgen_WinSCP_040.png)

9. Speichere deine Eingaben, indem du auf die Schaltfläche **Speichern* klickst.

    ![SSH-Key_PuTTYgen_WinSCP_050](/images/SSH-Key_PuTTYgen_WinSCP_050.png)

10. Gib unter **Name des Verbindungsziels:** einen beliebigen Namen ein oder verwende den vorgegebenen Namen. Klicke auf **OK**, um die Sitzung zu speichern.

    ![SSH-Key_PuTTYgen_WinSCP_060](/images/SSH-Key_PuTTYgen_WinSCP_060.png)

11. Dein gespeichertes Verbindungsziel kannst du ab sofort auf der linken Seite auswählen, um eine Verbindung zu deinem Remote Server herzustellen. Klicke dazu auf die Schaltfläche **Verbinden**.

    ![SSH-Key_PuTTYgen_WinSCP_070](/images/SSH-Key_PuTTYgen_WinSCP_070.png)

[Zurück zum Inhaltsverzeichnis](https://github.com/toafez/Tutorials)
