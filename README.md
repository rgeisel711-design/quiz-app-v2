# 🚀 Übung: Quiz App mit Docker Compose

Willkommen zur Praxisübung! In diesem Projekt haben wir eine komplette, moderne Web-Anwendung, die aus drei Bausteinen (Services) besteht:

- 🖥️ **`web`**: Ein Next.js Frontend (Das, was der Nutzer im Browser sieht).
- ⚙️ **`api`**: Ein Express-Backend (Verarbeitet die Logik und spricht mit der Datenbank).
- 🗄️ **`db`**: Eine PostgreSQL Datenbank (Speichert unsere Fragen und Antworten).

**Eure Mission:** Erstellt selbstständig die `compose.yaml`, die diese drei isolierten Container startet, miteinander vernetzt und zum Laufen bringt!

---

## 🛠️ Voraussetzungen

- Docker Desktop (oder Docker Engine + Compose Plugin) läuft auf eurem Rechner.
- Git ist installiert.
- Der Port `3000` auf eurem Rechner ist frei (Hierüber greifen wir später auf die App zu).

---

## 🧠 Vorab: Klärende Fragen (Bitte lesen & verstehen!)

Bevor ihr anfangt, die `compose.yaml` zu schreiben, macht euch Gedanken über folgende Punkte. Das hilft euch enorm beim Verständnis der Architektur:

1. **Netzwerk & Sicherheit:** Warum geben wir nur dem `web`-Service ein Port-Mapping (Port `3000`) nach außen, aber der `api` und der `db` nicht? Wie können `web` und `api` trotzdem miteinander reden?
2. **Datenpersistenz:** Was passiert mit unseren gespeicherten Quizfragen, wenn der Datenbank-Container gelöscht wird? Wie verhindern wir den Datenverlust?
3. **Start-Reihenfolge:** Das Frontend (`web`) stürzt ab, wenn die `api` noch nicht da ist. Die `api` stürzt ab, wenn die `db` noch nicht da ist. Wie teilen wir Docker mit, wer auf wen warten muss?
4. **Datenbank-Initialisierung:** Wenn wir ein komplett nacktes `postgres`-Image herunterladen, hat es keine Tabellen. Wie bekommt die Datenbank beim allerersten Start automatisch unsere Tabellen-Struktur?
   *(Tipp: Recherchiert nach "Postgres Docker Entrypoint initdb").*
5. **Konfiguration:** Woher weiß die API, mit welchem Passwort sie sich bei der Datenbank anmelden muss?

---

## 🏃‍♂️ Vorbereitung & Ablauf

Führt die folgenden Schritte im Terminal aus, um euren Arbeitsplatz vorzubereiten:

1. **Repository klonen & betreten:**
   ```bash
   git clone git@github.com:MartinCloudHelden/quiz-app-v2.git
   cd quiz-app-v2
   ```

2. **Umgebungsvariablen (Environment Variables) anlegen:**
   Kopiert die Vorlage. Diese Datei enthält wichtige Passwörter und URLs für die Container.
   ```bash
   cp .env.example .env
   ```

3. **Die eigenen Images bauen:**
   Wir nutzen für die Datenbank ein fertiges Image, aber unseren eigenen Code für `web` und `api` müssen wir erst in Images verpacken:
   ```bash
   # Baut das Frontend-Image und nennt es "quiz-web:local"
   docker build -f services/web/Dockerfile -t quiz-web:local .
   
   # Baut das Backend-Image und nennt es "quiz-api:local"
   docker build -f services/api/Dockerfile -t quiz-api:local ./services/api
   ```

---

## 🏗️ Eure Aufgabe: Die `compose.yaml` schreiben

Erstellt nun eine Datei namens `compose.yaml` im Hauptverzeichnis. Nutzt die folgenden Hinweise und Links zur offiziellen Dokumentation, um die Datei aufzubauen:

### 1. Die drei Services definieren
Eure Datei braucht drei Blöcke unter `services:` -> `web`, `api` und `db`.

### 2. Images referenzieren
- Für `web` und `api` nutzt ihr die Images, die ihr in Schritt 3 selbst gebaut habt (`quiz-web:local` und `quiz-api:local`).
- Für die `db` nutzt das offizielle Image: `postgres:16-alpine`.
- 📖 *Docs:* [Compose `image` Referenz](https://docs.docker.com/compose/compose-file/05-services/#image)

### 3. Ports & Erreichbarkeit
- **Nur** das Frontend (`web`) soll von außen über euren Browser erreichbar sein. Mappt den Port `3000` eures Laptops auf den Port `3000` des Containers.
- 📖 *Docs:* [Compose `ports` Referenz](https://docs.docker.com/compose/compose-file/05-services/#ports)

### 4. Umgebungsvariablen (`.env`)
Schaut in die `.env` Datei. Diese Werte müssen in die Container.
- Gebt der `db` die Variablen `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`.
- Die Services `web` und `api` brauchen ebenfalls ihre jeweiligen Variablen aus der Liste.
- *Wichtig:* Wenn ihr die Datenbank-Zugangsdaten in der `.env` ändert, müsst ihr zwingend auch den Connection-String in `DATABASE_URL` anpassen!
- 📖 *Docs:* [Compose `environment` Referenz](https://docs.docker.com/compose/compose-file/05-services/#environment)

### 5. Start-Reihenfolge (Dependencies)
Sorgt dafür, dass `web` erst startet, wenn `api` läuft. Und `api` erst, wenn `db` läuft.
- 📖 *Docs:* [Compose `depends_on` Referenz](https://docs.docker.com/compose/compose-file/05-services/#depends_on)

### 6. Daten speichern (Named Volumes)
Erstellt ein "Named Volume" (z.B. `db-data`), um die Datenbankdateien dauerhaft zu speichern. Mountet es in der `db` unter dem Pfad `/var/lib/postgresql/data` (Das ist der Standard-Pfad, wo Postgres intern seine Daten ablegt).
- 📖 *Docs:* [Compose `volumes` Referenz](https://docs.docker.com/compose/compose-file/07-volumes/)

### 7. Der Magic-Trick: Datenbank initialisieren (Bind Mount)
Postgres Container haben eine tolle Funktion: Alle `.sql` Skripte, die im Ordner `/docker-entrypoint-initdb.d` liegen, werden beim *allerersten* Start automatisch ausgeführt!
- Mountet unseren lokalen Ordner `./services/db/init` (dort liegt unsere Struktur) in den Container genau in diesen Pfad.
- *Tipp: Macht es `ro` (read-only), der Container muss unsere lokalen SQL-Dateien nur lesen, nicht überschreiben.*

---

## 🚀 Testen & Starten

Wenn eure `compose.yaml` fertig ist, startet die Magie:

1. **Stack hochfahren:**
   ```bash
   docker compose up
   # (Tipp: mit -d am Ende startet es im Hintergrund)
   ```

2. **Frontend prüfen:**
   Öffnet euren Browser: `http://localhost:3000`. Seht ihr die Quiz-App?

3. **Test-Daten importieren (Optional aber empfohlen):**
   Um zu prüfen, ob die App voll funktioniert, nehmt das JSON-Schema aus `examples/questions-upload.sample.json`. 
   *Tipp: Gebt das Schema einem LLM (ChatGPT/Claude) und lasst euch echte Quizfragen in genau diesem Format generieren, um sie im Frontend hochzuladen!*

4. **Aufräumen:**
   Wenn ihr fertig seid, fahrt die Umgebung sauber herunter:
   ```bash
   docker compose down
   ```
   *Wollt ihr auch die Datenbank komplett löschen (z.B. weil ihr Fehler in euren Tabellen hattet)? Dann tippt:*
   ```bash
   docker compose down -v
   ```

---

## 📚 Wo finde ich Hilfe zur App-Logik?

Falls ihr verstehen wollt, wie die App intern aufgebaut ist, findet ihr in diesem Projekt noch weitere Dokumentation:

- **Wie sieht die Datenbank aus?** `docs/data-model.md`
- **Wie kommuniziert das Frontend mit der API?** `docs/api-contract.md`
- **Wie müssen Dateien für den Upload aussehen?** `docs/upload-format.md`
- **Abnahme-Checkliste (Muss am Ende alles funktionieren):** `docs/smoke-test.md`
