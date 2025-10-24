## Einleitung
Vor kurzem stand ich vor der Herausforderung, im Zusammenspiel mit Paperless-ngx **ein Major-Update der** der damit verbundenen **PostgreSQL-Datenbank von Version 17 auf Version 18 durchzuführen**. Da dieser Vorgang nicht ganz trivial war und ich dabei die eine oder andere Hürde überwinden musste, möchte ich nun versuchen, das Ganze in der nachfolgenden Anleitung zu beschreiben.

## Hinweise!
***Texte in Großbuchstaben innerhalb eckiger Klammern dienen als Platzhalter und müssen durch eigene Angaben ersetzt werden, können aber an einigen Stellen auch nur der Information dienen. Es ist zu beachten, dass die eckigen Klammern Teil des Platzhalters sind und beim Ersetzen durch eigene Angaben ebenfalls entfernt werden müssen.***

- Im Folgenden wird davon ausgegangen, dass Paperless-ngx das Verzeichnis `/volume1/docker/paperless-ngx` zur Aufnahme der persistenten Daten verwendet.
- **Erstelle zur Sicherheit ein Backup der persitenten Daten.**
- Die Inhalte der PostgreSQL-Datenbank liegen im Unterverzeichnis `/pgdata`. 
- Als Datenbankname `[POSTGRES_DB_NAME]`, Benutzername `[POSTGRES_USERNAME]` und Passwort `[POSTGRES_PASSWORD]` wird der Name **"paperless"** verwendet.
- In der Docker-Compose-Datei ist der PostgreSQL-Datenbank-Container mit dem Service-Namen `[DOCKER_COMPOSE_DB_SERVICE_NAME]` **db** und dem Containernamen `[POSTGRES_CONTAINER_NAME]` **Paperless-ngx-PostgreSQL** benannt. 

**Solltet ihr abweichenden Verzeichnisse oder Namen verwenden, passt die nachfolgenen Befehle entsprechend an.**

## SSH Verbindung herstellen
Starte eine SSH-Terminalverbindung zu deinem UGREEN-NAS, indem du dich als Administrator anmeldest. Im Folgenden werden alle Befehle über den am System angemeldeten Administrator ausgeführt und ggf. mit einem vorangestellten `sudo` ergänzt, um diese mit erweiterten bzw. Root-Rechte auszuführen. Wer die Befehle direkt als Root ausführt, benötigt kein vorangestelltes `sudo`.


## Istzustand ermitteln
Wechsle in das Verzeichnis, in dem die persistenten Daten deiner Paperless-ngx Installation gespeichert sind. Im folgenden Beispiel befindet sich dieses Verzeichnis unter `/volume1/docker/paperless-ngx`. Passe den Pfad ggf. an.
```
cd /volume1/docker/paperless-ngx
```

Zur Sicherheit solltest du dir noch einmal alle Services anzeigen lassen, die zusammen mit deiner Paperless-ngx-Docker-Compose-Datei geladen wurden.
```
sudo docker compose watch
```

Das Ergebnis könnte so aussehen:
```
[+] Running 3/0
 ✔ Container Paperless-ngx-PostgreSQL  Running 
 ✔ Container Paperless-ngx-Redis       Running
 ✔ Container Paperless-ngx             Running    
```

Überprüfe die installierte PostgreSQL Version mit...

**Syntax:**
```
sudo docker exec -it [POSTGRES_CONTAINER_NAME] psql -U [POSTGRES_USERNAME] -d [POSTGRES_DB_NAME]
```

**Beispiel:**
```
sudo docker exec -it Paperless-ngx-PostgreSQL psql -U paperless -d paperless
```

**Beispielergebnis:**
```
psql (17.6 (Debian 17.6-2.pgdg13+1))
Type "help" for help.

paperless=#
```
Beenden mit `exit`

## PostgreSQL-17-Datenbank sichern
Du solltest dich immer noch im Verzeichnis unter `/volume1/docker/paperless-ngx` befinden. Falls nicht, wechsle zunächst in dieses bzw. in das von dir verwendete Verzeichnis.

Beende Paperless-ngx zusammen mit allen geladenen Containern bzw. mit der im Paperless-ngx Docker-Compose Datei eingebundenen Services.
```
sudo docker compose down
```

Starte den mit Paperless-ngx verbundenen PostgreSQL-17-Datenbank-Container. Verwende dazu den Service-Namen, den du in der Docker-Compose-Datei für die PostgreSQL-17-Datenbank angegeben hast.

**Syntax:**
```
sudo docker compose up [DOCKER_COMPOSE_DB_SERVICE_NAME] -d
```

**Beispiel:**
```
sudo docker compose up db -d
```

Erstelle nun ein Backup deiner PostgreSQL-17-Datenbank (umgangssprachlich „dump” genannt).

**Syntax:**
```
docker exec [POSTGRES_CONTAINER_NAME] pg_dumpall -U [POSTGRES_USERNAME] > backup.sql
```

**Beispiel:**
```
sudo docker exec Paperless-ngx-PostgreSQL pg_dumpall -U paperless > backup.sql
```

Nachdem der Datenbank-Dump erfolgreich erstellt wurde, wird der PostgreSQL-17-Datenbank-Container bzw. Service wieder beendet.
```
sudo docker compose down
```

### Update auf PostgreSQL 18 vorbereiten
Als Nächstes muss der aktuelle Speicherort deiner PostgreSQL-17-Datenbank umbenannt werden. In diesem Beispiel befindet er sich unter `/pgdata` und wird in `/pgdata17` umbenannt.
```
mv ./pgdata ./pgdata17
```

Erstelle nun ein neues, leeres Verzeichnis, das wiederum den Namen `/pgdata` trägt.
```
mkdir ./pgdata
```

Bearbeite im Folgenden deine Docker-Compose-Datei mit einem Editor deiner Wahl. Verwende dafür jedoch nicht die in UGOS Pro installierte Docker-App, Portainer oder ähnliches, da nach dem erneuten Bereitstellen bzw. "Deployen" Paperless-ngx inklusive aller verbundenen Services komplett gestartet wird.

Ändere folgende Zeile in deiner Docker-Compose-Datei von...
```
image: docker.io/library/postgres:17
```

... nach ...

```
image: docker.io/library/postgres:18
```

Da sich mit PostgreSQL 18 der interne Pfad zum "flüchtigen" Datenbankspeicherort geändert hat, muss dieser ebenfalls angepasst werden. Ändere dazu den Pfad hinter dem Doppelpunkt von ...
```
- - /volume1/docker/paperless-ngx/pgdata:/var/lib/postgresql/data:rw
```

... nach ...

```
- /volume1/docker/paperless-ngx/pgdata:/var/lib/postgresql/18/docker:rw
```

Passe dabei den Pfad zu deinem persistenten Datenbankordner (also den Teil vor dem Doppelpunkt) entsprechend deiner Konfiguration an.

Speichere und schließe anschließend die geänderte Docker-Compose-Datei und kehre ggf. auf das noch geöffnete Terminalfenster zurück. 

## Auf PostgreSQL 18 updaten
Starte nun den neuen, mit Paperless-ngx verbundenen PostgreSQL-18-Datenbank-Container. Verwende dazu erneut den Service-Namen, den du in der Docker-Compose-Datei für die PostgreSQL-18-Datenbank angegeben hast.
```
sudo docker compose up db -d
```

Spiele nun das zuvor angelegte PostgreSQL 17 Datenbank-Backup (Dump) zurück.

**Syntax:**
```
sudo docker exec -i [POSTGRES_CONTAINER_NAME] psql -U [POSTGRES_USERNAME] -d [POSTGRES_PASSWORD] < backup.sql
```

**Beispiel:**
```
sudo docker exec -i Paperless-ngx-PostgreSQL psql -U paperless -d paperless < backup.sql
```

Beende den PostgreSQL-18-Datenbank-Container bzw. Service erneut.
```
sudo docker compose down
```

### Paperless-ngx starten und den Update-Erfolg überprüfen
Starte Paperless-ngx über die UGOS-Pro Docker App.

Überprüfe abermals die installierte PostgreSQL Version mit...

**Syntax:**
```
sudo docker exec -it [POSTGRES_CONTAINER_NAME] psql -U [POSTGRES_USERNAME] -d [POSTGRES_DB_NAME]
```

**Beispiel:**
```
sudo docker exec -it Paperless-ngx-PostgreSQL psql -U paperless -d paperless
```

**Beispielausgabe:**
```
psql (18.0 (Debian 18.0-1.pgdg13+3))
Type "help" for help.

paperless=#
```
Beenden mit `exit`

## Nicht mehr benötigte Verzeichnisse und Dateien löschen
Lösche nach Bedraf das alte Datenbankverzeichnis

```
sudo rm -rf ./pgdata17
```

... und ggf. die Datenbank-Backup Datei `backup.sql`.

```
sudo rm backup.sql
```

Fertig!