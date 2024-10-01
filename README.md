
# Architecture Ref.Card 02 - React Application (serverless)

Link zur Übersicht<br/>
https://gitlab.com/bbwrl/m346-ref-card-overview

## Installation der benötigten Werkzeuge

Für das Bauen der App wird Node bzw. npm benötigt. Die Tools sind unter 
der folgenden URL zu finden. Für die meisten Benutzer:innen empfiehlt sich 
die LTS Version.<br/>
https://nodejs.org/en/download/

Node Version Manager<br/>
Für erfahren Benutzer:innen empfiehlt sich die Installation des 
Node Version Manager nvm. Dieses Tool erlaubt das Installiert und das 
Wechseln der Node Version über die Kommandozeile.<br/>
**Achtung: Node darf noch nicht auf dem Computer installiert sein.**<br/>
https://learn2torials.com/a/how-to-install-nvm


## Inbetriebnahme auf eigenem Computer

Projekt herunterladen<br/>
```git clone git@gitlab.com:bbwrl/m346-ref-card-02.git```
<br/>
```cd architecture-refcard-02```

### Projekt bauen und starten
Die Ausführung der Befehle erfolgt im Projektordner

Builden mit Node/npm<br/>
```$ npm install```

Das Projekt wird gebaut und die entsprechenden Dateien unter dem Ordner node_modules gespeichert.

Die App kann nun mit folgendem Befehl gestartet werden<br/>
```$ npm start```

Die App kann nun im Browser unter der URL http://localhost:3000 betrachtet werden.

### Inbetriebnahme mit Docker Container
1. ``Dockerfile`` im Root vom Projekt mit folgender Konfiguration erstellen:
   ```.dockerfile
   FROM node:16-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   RUN npm update
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   ```
2. Docker Image builden und pushen
   ```.shell
   docker build -t  gentian2002/m346-ref-card-gentian-beqiraj .
   docker push  gentian2002/m346-ref-card-gentian-beqiraj
   ```

### GitHub Actions konfigurieren
1. Erstelle ein ```.github/workflows/ci.yml``` file
2. Auf www.docker.com ein Token generieren:
   ![img_1.png](img_1.png)
3. Docker Hub Credentials als Secrets hinzufügen
   ```.yaml
   name: CI

   on:
     push:
       branches: [ main ]

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
         - uses: actions/checkout@v3
         - name: Build the Docker image
           run: docker build -t gentian2002/m346-ref-card-gentian-beqiraj .
         - name: Log in to Docker Hub
           run: docker login -u $DOCKER_USERNAME -p  ${{ secrets.DOCKER_PASSWORD }}
         - name: Push the Docker image
           run: docker push gentian2002/m346-ref-card-gentian-beqiraj
   ```
4. Secrets in meinem GitHub Repository Settings erstellen.
   ![img_3.png](img_3.png)
5. Änderungen pushen. (Anschliessend wird die Github Actions ausgeführt und ein Container wird auf 
   DockerHub erstellt.)

### Schlussfolgerung
Diese Pipeline automatisiert den Prozess des Builden, Testens und Deployen meiner React-App. 
Sie kann weiter angepasst werden, um zusätzliche Schritte wie die Ausführung von Tests, das 
Deployment auf einem Server oder die Verwendung von Caching einzubeziehen.
![img_4.png](img_4.png)
![img_5.png](img_5.png)

## Docker auf AWS builden und deployen
Dieses Repository enthält einen GitHub Actions-Workflow zur Automatisierung des Prozesses der 
Erstellung eines Docker-Images und dessen Bereitstellung in einer Amazon Web Services (AWS) Elastic 
Container Registry (ECR) und EC2-Instanz.

### Workflow Übersicht

#### 1. Variablen und Secrets setzen
- **Repository Secrets**
   Um den AWS CLI zu nutzen, müssen die AWS tokens (`AWS_ACCESS_KEY_ID`, `AWS_REGION`, 
   `AWS_SECRET_ACCESS_KEY` und `AWS_SESSION_TOKEN`) als Secrets auf GitHub gespeichert sein.
   ![img.png](img.png)
   ![img_2.png](img_2.png)
- **Repository Variablen**
  Zur Verwaltung der Versionierung verwenden wir die Variable `VERSION`. Diese Variable wird 
  verwendet, um das Docker-Image und die GitHub-Version zu kennzeichnen. Wir müssen diese Variable 
  jedes Mal inkrementieren, wenn das Production Deployment erfolgreich durchgeführt wurde. 

#### 2. ECR-Repository erstellen.
1. Suche in der AWS Konsole nach "ECR".
2. Klicke auf "Repository Erstellen".
3. Benenne das Repository und übernehme die Default-Einstellungen. Klicke erneut auf "Erstellen".
   ![img_7.png](img_7.png)
4. Repository wird nun erstellt.
   ![img_8.png](img_8.png)
5. Die vollständige ECR URI ebenfalls als secret auf GitHub abspeichern.

#### 3. Job erstellen
- **Läuft auf**: `ubuntu-latest`
- **Schritte**:
   1. **Code auschecken**: Ruft den Code aus dem Repository ab.
   2. **AWS-Credentials einrichten**: Konfiguriert AWS CLI mit den in GitHub Secrets gespeicherten Credentials.
   3. **Anmelden bei Amazon ECR**: Authentifiziert den Docker-Client gegenüber dem Amazon ECR-Service.
   4. **Docker-Image erstellen**: Erzeugt das Docker-Image aus der Dockerdatei im Repository.
   5. **Tag Docker Image**: Markiert das erstellte Docker-Image für ECR.
   6. **Docker-Image zu Amazon ECR pushen**: Pusht das getaggte Docker-Image zu ECR.
- Danach wird es an ECR gesendet und sollte wie folgt aussehen:
  ![img_17.png](img_17.png)
  Siehe Konfiguration deploy.yml
  

#### 3. ECS-Cluster erstellen
1. Suche in der AWS Konsole nach "ESC".
2. Klicke auf "Cluster erstellen".
3. Cluster benennen und Default Einstellung übernehmen. Wieder auf "Erstellen" klicken.
   ![img_9.png](img_9.png)
4. Cluster wurde erstellt.
   ![img_10.png](img_10.png)

#### 4. ECS-Aufgabendefinition erstellen
1. Gehe zu Aufgabendefinitionen und klicke auf "Neue Aufgabendefinition erstellen".
2. Namen geben und Aufgaben- und Aufgabenausführungsrolle auf "LabRole" setzen.
   ![img_11.png](img_11.png)
3. Repository-Namen und -URI definieren. Port auf 8080 setzen.
   ![img_12.png](img_12.png)
4. Auf "Erstellen klicken".
   ![img_13.png](img_13.png)

#### 5. ECS-Service erstellen
1. Klicke auf "Bereitstellen" und dann auf "Service erstellen".
   ![img_14.png](img_14.png)
2. Cluster selektieren.
   ![img_15.png](img_15.png)
3. Service-Name geben.
   ![img_16.png](img_16.png)
4. Auf erstellen klicken und überprüfen, ob die Applikation richtig läuft.
5. Überprüfe, ob alles funktioniert, sobald der Service läuft, indem du folgende Schritte unternimmst:
   1. Gehe zu Cluster und wähle dein Cluster aus.
   2. Klicke auf Aufgaben und wähle dann deine erstellte Aufgabe aus.
   3. Gehe zu Netzwerke und klicke auf die öffentliche IP Adresse
   ![img_18.png](img_18.png)

#### 6. Job deployen
`needs: build:`
Gibt an, dass dieser Job von einem früheren Job namens `build` abhängt und erst nach dessen erfolgreichem Abschluss ausgeführt wird.

`runs-on: ubuntu-latest:`
Gibt an, dass der Job in der neuesten Version einer Ubuntu-Linux-Umgebung ausgeführt wird, die von GitHub Actions bereitgestellt wird.

`steps::`
Definiert die einzelnen Schritte, die in diesem Job ausgeführt werden.

##### Schritt 1: AWS credentials für Session bereitstellen:
- **`name:`** Einen für Menschen lesbarer Name für diesen Schritt, um dessen Zweck zu zeigen
- **`env:`** Umgebungsvariablen setzen, welche im `run` Command verfügbar sind:
    - **`AWS_ACCESS_KEY_ID:`** Der Access Key für deinen AWS account.
    - **`AWS_SECRET_ACCESS_KEY:`** Der Secret Key für dein AWS account.
    - **`AWS_SESSION_TOKEN:`** Ein Session Token für temporäre AWS credentials.
    - **`AWS_DEFAULT_REGION:`** Die default region für AWS services.
- **`run:`** Beinhaltet shell commands die den AWS CLI konfigurieren:
    ```bash
    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
    aws configure set aws_session_token $AWS_SESSION_TOKEN
    aws configure set region $AWS_DEFAULT_REGION
    ```
  Dadurch wird die AWS CLI-Umgebung so eingerichtet, dass die angegebenen Anmeldeinformationen und 
  die Region für nachfolgende Befehle verwendet werden.

##### Schritt 2: ECS Services updaten
- **`name:`** Ein von Menschen lesbarer Name.
- **`run:`** Beinhaltet den Command um den ECS Service upzudaten:
    ```bash
    aws ecs update-service --cluster RefCard02 --service RefCard02-Service --task-definition refcard02
    ```
    Mit diesem Befehl soll der angegebene ECS-Service innerhalb des Clusters „RefCard02“ aktualisiert werden. 

##### Schritt 3: Deployen
1. Änderungen auf den Main-Branch pushen 
2. Der Workflow wird automatisch ausgelöst, wobei das Docker-Image erstellt und in der EC2-Instanz bereitgestellt wird.