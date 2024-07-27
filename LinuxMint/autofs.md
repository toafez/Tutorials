[English](autofs_en.md) | Deutsch

# SMB-Freigaben mit Autofs
Mit Autofs lassen sich u.a. Remote-Server Freigaben bei Bedarf automatisch einhängen (mounten) und bei Bedarf nach einer zuvor festgelten Zeit auch wieder automatisch aushängen. 

## Installation von Autofs

`sudo apt-get install autofs`

## .smbcredentials

Wird für die Freigabe ein Passwort benötigt, muss eine versteckte Textdatei mit dem Namen .smbcredentials im eigenen Benutzer-Home-Ordner erstellt werden.

- Eine neue, leere Datei mit dem Namen .smbcredentials erstellen  
    `touch ~/.smbcredentials`  

- Die grade erstellte Datei .smbcredentials editieren    
    `nano ~/.smbcredentials`  

- Inhalt der Datei ~/.smbcredentials  
    ```
    username=[BENUTZERNAME]
    password=[PASSWORT]
    ```    
- Die Dateirechte der Datei ~/.smbcredentials beschränken  
    `sudo chmod 600 ~/.smbcredentials`
    
## Die Master-Map-Datei

- Mountpoint festlegen, wohin die Freigaben gemountet werden sollen  
    `sudo mkdir /media/[BENUTZERNAME]/Fileserver`  

- Am Ende der Datei /etc/auto.master folgende Zeile eintragen  
    ***Syntax:** \[mountpoint\] \[Map-Datei\] \[options\]*  
    `/media/[BENUTZERNAME]/Fileserver /etc/auto.fileserver --timeout=0 --ghost`

## Die Map-Datei

- Einen neue, leere Datei erstellen  
    `sudo touch /etc/auto.[DATEINAME]`  

    ...oder einfacher...  
    `sudo nano /etc/auto.[DATEINAME]`  

- Ausführbarkeit einer Datei /etc/auto.\[DATEINAME\] entziehen  
    `sudo chmod -x /etc/auto.[filename]`

     **Achtung:** Dateien mit dieser Struktur dürfen nicht als ausführbar markiert sein.

- Inhalt der Datei /etc/auto.\[DATEINAME\]  
    ***Syntax:** \[FREIGABE-NAME\] -fstype=cifs,\[OPTIONEN\] ://\[IP-ADRESSE\]/\[FREIGABE-PFAD\]*  
    ```
    [FREIGABE-NAME] -fstype=cifs,credentials=/home/[BENUTZERNAME]/.smbcredentials ://[IP-ADRESSE]/[FREIGABE-PFAD]
    [FREIGABE-NAME] -fstype=cifs,credentials=/home/[BENUTZERNAME]/.smbcredentials ://[IP-ADRESSE]/[FREIGABE-PFAD]
    ```

Weitere Informationen zu autofs findet ihr unter https://wiki.ubuntuusers.de/Autofs/
