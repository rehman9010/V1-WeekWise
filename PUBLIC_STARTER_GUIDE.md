<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-17; spring-semester extension; reviewed by: pending -->
# WeekWise mit der veröffentlichten Sportdatenbank starten

Die App enthält einen bereinigten SQLite-Startbestand mit dem Planungshorizont
**16.11.2026–31.07.2027**: **9.731 Termine aus 263 Kursmustern**. Davon haben **1.551**
einen konkreten Website-Nachweis; **8.180** bleiben vorläufig. Geprüft wurde am
15. und 17.11.2026; der späteste exakt nachgewiesene Termin liegt am 20.02.2027.
Zähler und Prüfsumme stehen im [Manifest](data/public/unisport_manifest.json).
Website-Nachweise gelten nur für den konkreten Termin zum gespeicherten Prüfzeitpunkt. Alle übrigen Termine bleiben Prognosen.
Freie Plätze, Preise und Kursorte bleiben ungeprüft.

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
Bestehende Arbeitsdateien werden nicht ersetzt. Eine Arbeitskopie des ursprünglich
veröffentlichten Sportbestands erhält die zusätzlichen Termine beim nächsten Start
durch eine geprüfte Ergänzung. Profile, Favoriten, gespeicherte Pläne, manuelle Änderungen
und bestehende Herkunftsnachweise bleiben erhalten. Bei widersprüchlichen IDs wird die
Ergänzung abgebrochen. Der separate Demo-Bestand bleibt erhalten; dessen Pläne werden
nicht auf andere Kurs-IDs übertragen.

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
http://127.0.0.1:5001/events?from=2027-07-26&to=2027-08-01
```

`from` ist einschließlich, `to` ausschließlich. Das Beispiel liest die letzte
Planungswoche mit Kursfeldern und Herkunft. Pro Anfrage sind höchstens 120 Tage
erlaubt; einen ganzen Semesterbestand deshalb in Wochen oder Monaten lesen. Die
UniClubs-Quelle wird von der App über ihren getrennten Feed gelesen. Wenn die eigene API nicht läuft, verwendet Streamlit
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

Es gibt weiterhin 15 neutral maskierte Kursmuster mit insgesamt 555 vorläufigen Terminen. „Website checked“ bestätigt keine freien Plätze oder individuelle
Teilnahmeberechtigung. Ein leerer Website-Ausschnitt beweist keine Absage.

Die Prognosen verwenden dieselbe einfache Wochenregel: gleiche Sportart/Variante,
gleicher Wochentag und gleiche Startzeit in mindestens drei der letzten vier
beobachteten Wochen, einschließlich der jüngsten Woche. Die historische Grundlage
endet am 06.11.2026. Sommerpläne, Feiertage und Semesterwechsel werden damit nicht
vorhergesagt; bei weiter entfernten Terminen sind Abweichungen wahrscheinlicher.
Diese Regel ist kein selbst trainiertes ML-Modell.

Als Ende des vollständigen Frühlingssemesters wird der **31.07.2027** verwendet.
Die Vorlesungszeit endet schon am **28.05.2027**, die zentralen Prüfungen am **23.07.2027**.
Quellen: [HSG-Semesterdaten](https://www.unisg.ch/de/studium/studieren-an-der-hsg/semesterdaten/)
und [Ausführungsbestimmungen, Art. 145](https://erlasse.unisg.ch/lexoverview-home/lex-II_A_09/lex-II_A_09.pdf?change-id=357&effective-from=20250201).
Vorlesungsfreie Zeit bedeutet nicht automatisch, dass der Sportbetrieb geschlossen ist.

**Reload saved catalogue** lädt gespeicherte Daten und führt keinen neuen Websiteabruf
aus. Der Sportbestand ist eine offline nutzbare Momentaufnahme; es gibt keinen
automatischen Sport-Download und keine tägliche Fortschreibung. Eine spätere Erweiterung
wird bewusst erzeugt, geprüft und hochgeladen. Die eigenständige UniClubs-Quelle behält
ihre bereits vorhandene automatische Aktualisierung.

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


## 7. Die Datenbank an jemanden weitergeben

Die Datei zum Teilen liegt in **`data/public/unisport_starter.sqlite`**. Am einfachsten
diesen Link senden und die SQLite-Datei zusammen mit dem Manifest herunterladen lassen:

- [Öffentlicher Datenordner auf GitHub](https://github.com/rehman9010/V1-WeekWise/tree/main/data/public)
- [SQLite-Datei direkt herunterladen](https://raw.githubusercontent.com/rehman9010/V1-WeekWise/main/data/public/unisport_starter.sqlite)
- [Manifest herunterladen](https://raw.githubusercontent.com/rehman9010/V1-WeekWise/main/data/public/unisport_manifest.json)

Die Person braucht die ursprüngliche Excel/CSV nicht. Sie kann SQL direkt verwenden
oder für HTTP/JSON das Repository herunterladen und `python api_server.py` starten.
Ein einfaches Beispiel ohne zusätzliche Bibliothek, aus dem Repository-Ordner:

```python
import sqlite3

connection = sqlite3.connect(
    'file:data/public/unisport_starter.sqlite?mode=ro', uri=True)
rows = connection.execute(
    'SELECT id, series_id, start_time, end_time FROM events '
    'WHERE start_time >= ? AND start_time < ? ORDER BY start_time',
    ('2027-07-26 00:00:00', '2027-08-01 00:00:00')).fetchall()
connection.close()
```

`series` enthält Kursfelder; `events.series_id` verbindet den Termin mit dem Kurs.
`event_provenance.event_id` verbindet ihn mit Prognose- oder Website-Nachweisen.
Die öffentliche Vorlage enthält weder Nutzerprofile noch private Kalender- oder
Trainingsrohdateien. Eigene Arbeitsdateien unter `data/` oder `data/private/` bleiben
getrennt. Ein Link auf GitHub liefert jeweils den dort aktuellen veröffentlichten
Bestand; für eine reproduzierbare Übergabe zusätzlich die heruntergeladene Datei und
die Prüfsumme aus dem Manifest aufbewahren.
