<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-16; approved Task 7; reviewed by: pending -->
# WeekWise mit der veröffentlichten Sportdatenbank starten

Die App enthält einen bereinigten SQLite-Startbestand für **16.11.–13.12.2026**:
1.052 Termine aus 263 Kursmustern. 967 konkrete Termine wurden am 15.11.2026 im
offiziellen Kalender nachgewiesen; 85 bleiben vorläufig. Die Veröffentlichung ist
keine neue Websiteprüfung. Freie Plätze, Preise und Kursorte bleiben ungeprüft.

## 1. Einfach starten

Repository herunterladen bzw. klonen und im Projektordner mit Python 3.11 ausführen:

```bash
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

Beim ersten Start wird die öffentliche Vorlage in eine eigene Arbeitsdatei kopiert.
Danach Namen/Fokus und Privatkalender auswählen. Ein Demo-Privatkalender ist möglich;
er ändert nichts an der Herkunft der echten Sportangebote.

Original-Excel, CSV und private Trainingshistorie sind dafür nicht erforderlich.
Bestehende Arbeitsdateien werden nicht ersetzt. Der neue Standardbestand erhält ein
eigenes Onboarding; alte Demo-Pläne werden nicht auf andere Kurs-IDs übertragen.

## 2. Welche Datei enthält was?

| Datei | Verwendung |
| --- | --- |
| [unisport_starter.sqlite](data/public/unisport_starter.sqlite) | Öffentliche, unveränderte Startvorlage; keine Nutzerprofile oder Rohhistorie |
| [unisport_manifest.json](data/public/unisport_manifest.json) | Prüfsumme, Größe, Zeitraum, Bestandskennung und Zähler |
| `data/weekwise-reviewed.sqlite` | Lokale Arbeitskopie mit den eigenen Einstellungen/Plänen; von Git ausgeschlossen |
| `data/weekwise.sqlite` | Bisheriger Bestand bzw. expliziter Demo-Modus; bleibt erhalten |
| `data/private/` | Private Dateien und getrennte UniClubs-Planungen; von Git ausgeschlossen |

Die vorhandene UniClubs-Anbindung bleibt erhalten: öffentlicher Feed und Katalog
liegen getrennt von den Sporttabellen; persönliche UniClubs-Pläne bleiben privat.
Unterschiedliche ID-Namensräume verhindern Kollisionen mit Sportterminen.

## 3. Datenbank über die eigene API lesen

Optional in einem zweiten Terminal mit demselben Arbeitsordner und denselben
Umgebungsvariablen starten:

```bash
python api_server.py
```

Diese echte HTTP-/JSON-API kann auch vor Streamlit gestartet werden und befüllt eine
frische Sport-Arbeitsdatei aus der öffentlichen Vorlage. Beispiel zum Lesen:

```text
http://127.0.0.1:5001/events?from=2026-11-16&to=2026-12-14
```

`from` ist einschließlich, `to` ausschließlich. Die Antwort enthält die 1.052
Sporttermine mit Kursfeldern und Herkunft. Die UniClubs-Quelle wird von der App über
ihren getrennten Feed gelesen. Wenn die eigene API nicht läuft, verwendet Streamlit
dieselben Sportdaten direkt aus SQLite. Eine Datenbankdatei ist selbst keine HTTP-API;
hier stellt der bereits vorhandene kleine Flask-Server die Schnittstelle bereit.

Die App prüft die Bestandskennung in `X-WeekWise-Catalogue`, damit sie nicht versehentlich
einen anderen laufenden API-Datenbestand verwendet. Unbekannte Verfügbarkeit bleibt
in SQL, API und UI gleich: Sport-Confirm und Einladungen sind deaktiviert. Der Link
zum offiziellen Kalender dient zur Prüfung und gegebenenfalls Buchung beim Anbieter.

## 4. Herkunft und Reload verstehen

- **Website checked:** Dieser konkrete Termin war zum gespeicherten Prüfzeitpunkt veröffentlicht.
- **Provisional:** Aus drei oder vier beobachteten Wochen abgeleitet; der Zukunftstermin ist ungeprüft.
- **Unreviewed sport activity:** Der ursprüngliche Kurstitel ist nicht freigegeben und wird neutral angezeigt.

Es gibt 15 solche neutralen Muster mit 60 vorläufigen Terminen. Die weiteren 25
vorläufigen Termine gehören zu bereits freigegebenen Kursmustern. „Website checked“
bestätigt keine freien Plätze oder individuelle Teilnahmeberechtigung.

**Reload saved catalogue** lädt gespeicherte Daten. Das ist kein neuer Websiteabruf.
**Reset my WeekWise data** entfernt persönliche Daten, erhält Sportangebote und
Herkunft und löscht die eigenen getrennten UniClubs-Pläne. Der öffentliche Feed bleibt
erhalten; ein neues Profil übernimmt keine alten privaten UniClubs-Pläne.

## 5. Den bestehenden ML-Demo-Modus verwenden

Der echte Startbestand enthält keine erfundenen Einzelbewertungen. Bis genügend
gültige Bewertungen vorliegen, verwendet die Empfehlung die vorhandene Heuristik.
Der bestehende getrennte Demo-Modus kann das trainierte Modell zeigen. Mit einer
eigenen privaten Demo-Arbeitsdatei, beispielsweise unter Bash:

```bash
WEEKWISE_DATA_MODE=demo WEEKWISE_DB_FILE=data/private/ml_demo.sqlite python -m streamlit run app.py
```

PowerShell:

```powershell
$env:WEEKWISE_DATA_MODE='demo'
$env:WEEKWISE_DB_FILE='data/private/ml_demo.sqlite'
python -m streamlit run app.py
```

Für den normalen veröffentlichten Bestand die beiden Variablen wieder entfernen und
den Prozess neu starten. Ein bereits belegter Zielpfad wird auch im Demo-Modus nicht
überschrieben. Künstliche Nutzer/Bewertungen bleiben ausdrücklich als Demo markiert;
historische Teilnehmerzahlen werden nicht in persönliche Ratings umgedeutet.

## 6. Prüfungen selbst ausführen

```bash
python -m unittest discover -s tests -v
python -m views.check_demo
python -m views.check_uniclubs
python -m views.check_uniclubs_recovery
python -m views.check_starter
```

Die vier App-Prüfungen verwenden temporäre Dateien. Die letzte prüft die tatsächliche
Startdatei gegen das Manifest, startet eine frische App und kontrolliert Sport- und
UniClubs-Daten gemeinsam einschließlich Reload, Profil-Reset und neuer Anmeldung.
In GitHub Actions laufen diese Prüfungen unter Linux, Windows und macOS mit Python 3.11.

Es kommen keine neuen Bibliotheken, kein ORM und kein KI-Aufruf zur Laufzeit hinzu.
SQLite/SQL, Python-Listen/Wörterbücher, Datumsfunktionen und die bestehende API/UI
bleiben die Grundlage. Die Verwendung von GenAI ist in `AI_USAGE.md` dokumentiert.
