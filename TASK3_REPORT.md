<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15; approved Task 3; reviewed by: pending -->
# Task 3 – private Trainingsdatenbank und CSV-Import

**Status: lokal abgeschlossen.** Task 4 wurde nicht gestartet. Es gab keinen Push,
keinen Merge und keine Änderung der veröffentlichten Streamlit-App.

## Ergebnis

Die Datei `data/private/training_history.sqlite` ist eine tatsächliche SQLite-Datenbank.
Sie enthält die vorhandene App-Struktur und die zwei freigegebenen Archivtabellen:

- `training_imports`: ein Datensatz für Dateiname, Dateiprüfkennung und Importzeit.
- `training_history`: 9.585 historische Trainings mit Originalzeilennummer, fachlichen
  Quellwerten, Original-Datumstext und ausdrücklich gelesenem ISO-Datum.

`series` und `events` wurden in diesem neuen Archiv nicht befüllt. Es wurden keine
Zukunftstermine erzeugt und keine aggregierten Teilnehmerzahlen als individuelle
Nutzer, Bewertungen oder Anwesenheitsmeldungen übernommen. `event_provenance` und
die Erweiterung der Terminverfügbarkeit um `unknown` bleiben spätere Tasks.

Die 71 zusätzlichen identischen Zeilen und die 683 Zeilen ohne beide Sportangaben
sind erhalten. Auch die zehn historischen Zeilen aus 2025 bleiben bestehen.
Leere Angaben werden als NULL gespeichert; echte Nullen bleiben Ganzzahlen. Die
Bedeutung von `Teilnehmer teacher` und Werten wie 9999 wird nicht neu interpretiert.

## Unabhängige Prüfung der echten Daten

| Prüfung | Ergebnis |
| --- | --- |
| Historische Datenzeilen | 9.585 |
| Originalzeilenreferenzen | 2 bis 9.586, vollständig |
| Fachlicher Vergleich mit Excel | 76.680 Werte geprüft, keine Abweichungen nach der dokumentierten Leerwert-Normalisierung |
| Original-Datumstexte aus CSV | alle 9.585 zusätzlich geprüft |
| Zweiter und dritter Import derselben Datei | jeweils null zusätzliche Datensätze; dieselbe Import-ID |
| SQLite-Integritätsprüfung | `ok` |
| Fremdschlüsselprüfung | keine Fehler |
| Neue Kurse und Kalendertermine | jeweils null |

Die Originaldateien sind unverändert. CSV und Excel einschließlich der Task-1-Prüfung
liegen privat vor. Die CSV kann die 13 ursprünglich explizit leeren Excel-Textzellen
nicht von echten Leerzellen unterscheiden; beide sind im Archiv als fehlend normalisiert.
Die Original-Excel und das Prüfprotokoll bleiben dafür die Referenz.

## Bedienung

Im Projektverzeichnis mit aktiviertem Python 3.11:

```bash
python training_history.py "data/private/training_inputs/CSV_2026 Auswertung Trainings .csv"
```

Standardziel ist `data/private/training_history.sqlite`. Ein anderes privates Ziel
kann mit `--database path/to/private.sqlite` gewählt werden. Der Import wird nicht
automatisch bei einem Streamlit-Neustart oder Kalender-Refresh ausgeführt.

Gleiche Dateibytes werden am SHA-256-Fingerabdruck erkannt, auch nach Umbenennen.
Der Wiederimport vergleicht zusätzlich alle archivierten Zeilen mit der Quelle.
Eine beschädigte Historie wird gemeldet und nicht stillschweigend repariert.
Veränderte Datei-Bytes ergeben einen neuen, zusätzlich erhaltenen Importbestand.
Das gilt auch für reine BOM-/Zeilenumbruchänderungen; Task 4 muss verschiedene
Trainingstermine zählen und darf wiederholte Zeilen/Exporte nicht als zusätzliche
Wochenbelege behandeln.

Die gesamten Eingabedaten werden vor dem Schreiben geprüft. Metadaten und Zeilen
werden in einer Transaktion gespeichert; bei Fehlern gibt es keinen Teilimport.
Fehlermeldungen geben Zeile/Feld an, aber keine privaten Rohwerte.

## Tests und Regression

- Python 3.11.18 mit den unveränderten Projektabhängigkeiten; Kompatibilitätsprüfung bestanden.
- `python -m unittest discover -s tests -v`: **198 Tests bestanden**, davon 19 neue Importtests.
- `python -m views.check_demo`: **bestanden**, einschließlich echtem lokalen API-Start,
  Interaktionen und Datenbank-Fallback im isolierten Demo-Ablauf.
- Separater Streamlit-Start: `/` liefert 200, `/_stcore/health` liefert 200 / `ok`.
  Der eigene Testserver wurde anschließend beendet.
- Alle Prüfungen dieser Arbeitsstufe wurden lokal unter Linux ausgeführt. Es gibt
  noch keinen GitHub-Actions-Lauf für diesen lokalen Branch.

Die neuen Tests prüfen unter anderem Unicode, CSV-Quoting, fehlend gegenüber null,
Duplikate, fehlerhafte Daten/Spalten/Zahlen, Archivgrenzen, stabile Wiederimporte,
SQL-ähnliche Quelltexte, beschädigte Archive und einen erzwungenen Transaktionsabbruch.
Der bestehende Schema-Test prüft jetzt genau die 15 erwarteten Tabellen statt nur
die frühere Anzahl 13. Kalenderbereinigung und Demo-Reset erhalten die Archivtabellen.

## Einfache Umsetzung und Abgrenzung

- Zwei zusätzliche Tabellen mit Primär-/Fremdschlüsseln und gebundenen SQL-Parametern.
- `pd.read_csv` mit Semikolon, UTF-8-BOM-Unterstützung und ausdrücklich behandelten Leerwerten.
- Deutsche Monats-/Wochentagsnamen werden über kleine Wörterbücher validiert;
  eine deutsche System-Locale ist nicht nötig.
- Eigene begrenzte Archivkapazität: 25.000 Zeilen / 10 MB. Das bisherige
  5.000er-Limit für aktive Quellenimporte bleibt unverändert.
- Keine neue Drittanbieterabhängigkeit, kein ORM, kein neuer Webserver und kein Lernmodell.
- `hashlib` für die in Task 2 freigegebene Dateiprüfkennung sowie `argparse` für die
  Kommandozeile sind technische Helfer; sie werden nicht als belegter Week07-Stoff ausgegeben.

## Dateien und nächste Grenze

Neu: `training_history.py`, `tests/test_training_history.py`, dieser Bericht.
Geändert: Archivschema in `database.py`, Konstanten in `config.py`, der bestehende
Schema-Test sowie README, Study Guide und AI-Nutzungsprotokoll.

Rohdaten und Datenbank bleiben unter dem bereits ignorierten `data/private/`.
Der lokale Arbeitsbranch heißt `data/unisport-history`. Die Veröffentlichung der
geprüften, freigegebenen Startdatenbank und des Codes erfolgt erst in Task 7.

Task 4 beginnt ausschließlich nach dem nächsten ausdrücklichen Startsignal.
