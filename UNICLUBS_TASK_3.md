<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-15, historical Task 3 report; review pending -->
# UniClubs – Task 3: Erster manueller Import

Dieser Bericht beschreibt den damaligen Erstimport. Seit [Task 4](UNICLUBS_TASK_4.md) führt
derselbe Kommandozeilenbefehl wiederholbare Abgleiche auch für einen bereits gefüllten Katalog aus.
Die unten dokumentierte Ablehnung eines zweiten Starts gilt nur für den historischen Task-3-Helfer.

Der erste echte Import wurde am 15. November 2026 um ca. 18:22 Uhr Europe/Zurich ausgeführt.
Die separate Datei `data/uniclubs.sqlite` enthält jetzt 49 künftig beginnende Events aus
23 Clubs sowie einen erfolgreichen Eintrag in `import_log`.

## Was ausgeführt wurde

Im Repository-Verzeichnis:

```bash
.venv/bin/python -m uniclubs_importer
```

Unter Windows lautet der Befehl `.venv\Scripts\python -m uniclubs_importer`.
Ohne virtuelle Umgebung kann das bereits aktivierte Python 3.11 verwendet werden.

Der Befehl erstellt gegebenenfalls fehlende Tabellen. Er führt den Erstimport nur in einem
Katalog ohne Clubs und Events aus. Nach dem erfolgreichen Import verweigert ein weiterer Start
den erneuten Import mit Exitcode 1 und einer erklärenden Meldung. Die gespeicherten Daten bleiben
erhalten. Das ist eine bewusst begrenzte Erstimport-Funktion; Änderungen und wiederholte Abgleiche
folgen nach Freigabe von Task 4. Die Datenbank muss dafür nicht gelöscht oder zurückgesetzt werden.

## Ablauf im Programm

1. Prüfen, dass der neue Katalog leer ist und keine offene Transaktion enthält.
2. Einmal die öffentliche Seite `https://uniclubs.ch/hsg/events` abrufen.
3. Im HTML die öffentlichen Eventlinks und eingebetteten Datenfragmente sammeln.
4. Die Fragmente zusammensetzen und als JSON lesen. JavaScript wird nicht ausgeführt.
5. Die gesamte gelieferte Liste auf IDs, Clubzuordnung, öffentliche Sichtbarkeit, HSG-Zuordnung,
   Pflichtangaben und gültige UTC-Zeitintervalle prüfen.
6. Nur Events auswählen, deren Beginn am oder nach dem festen Abrufzeitpunkt liegt.
7. Prüfen, dass alle ausgewählten Events einen sichtbaren Detail-Link haben und alle gefundenen
   Detail-Links zum gelieferten Quellenbestand gehören.
8. Die zugehörigen Clubs, Events und den Erfolgseintrag in einer Transaktion speichern.

Die neue Klasse `UniClubsPage` sammelt mit Pythons `HTMLParser` lediglich vorhandene Seitendaten.
`json.loads` und `JSONDecoder.raw_decode` lesen deren Struktur. Die vollständige Website wird
weder gespiegelt noch als Programm ausgeführt. Ein anderer Website-Aufbau führt zu einem
kontrollierten Fehler, statt unvollständige Daten stillschweigend zu übernehmen.

## Echte Ergebnisse

| Prüfung | Ergebnis |
| --- | --- |
| Gelieferte öffentliche HSG-Eventobjekte | 440 |
| Sichtbare aktuelle Eventdetail-Links | 50 |
| Künftig beginnende Events gespeichert | 49 |
| Bereits laufende Angebote erkannt und ausgeschlossen | 1 |
| Bereits beendete Events ausgeschlossen | 390 |
| Zugehörige Clubs gespeichert | 23 |
| Erfolgsprotokolle in der echten Datenbank | 1 |
| SQLite-Integrität | `ok` |
| Fehlerhafte Fremdschlüssel | keine |

Die Zahlen sind eine Momentaufnahme des öffentlichen Bestands, keine fest programmierten Werte.
Private und unveröffentlichte Events sind nicht Bestandteil dieses Imports. Die vollständige
Quelle wird geprüft; nur künftige Starts werden gespeichert. Lang laufende Angebote erhalten
keine erfundenen wöchentlichen Einzeltermine.

Beispiele, umgerechnet nach Europe/Zurich:

| Event | Datum | Zeit |
| --- | --- | --- |
| Organizational Design and How to Hire Exceptional Talent | 16.11.2026 | 12:00–13:00 |
| IWC Case Study On Campus | 16.11.2026 | 17:30–21:00 |
| Harris Williams Recruitment & Networking Event | 17.11.2026 | 12:00–14:30 |

Die Datenbank speichert dafür die ursprünglichen UTC-Zeitpunkte als einheitlichen Text.
`publication_status` und `availability_state` bleiben getrennt. Geschlossene Anmeldung bedeutet
keine Absage. Fehlende Endzeiten würden als unbekannt gespeichert, nicht geschätzt.

## Fehlerverhalten und Grenzen

- TLS-Prüfung bleibt aktiv; der Download hat ein Zeitlimit und eine Grenze von 5 MB.
- Es wird nur die ungefilterte öffentliche HTML-Übersicht angefragt. Keine API-Endpunkte,
  privaten Bereiche oder Anmeldeseiten werden abgefragt; Weiterleitungen werden abgewiesen.
- Pro manuellem Erstimport gibt es eine einzige HTTP-Anfrage. Kein automatischer Wiederholungs-
  oder Hintergrundprozess wird gestartet. Beim manuellen Prüfen sind die in robots.txt genannten
  Abstände von mindestens einer Sekunde einzuhalten.
- Leere, beschädigte, zu große, doppelte oder widersprüchliche Daten werden abgewiesen.
- Alle Katalogeinträge und der Erfolgseintrag werden gemeinsam bestätigt. Ein Fehler beim späteren
  Datensatz rollt auch die zuvor eingefügten Clubs und Events dieses Versuchs zurück.
- Download-/Parserfehler hinterlassen nur eine kurze Fehlermeldung im Importprotokoll, ohne rohe
  Transportdetails, Zugangscodes oder private Kalenderdaten.
- Ein erneuter Aufruf am gefüllten Katalog wurde geprüft: erwarteter Exitcode 1, unveränderter
  SHA-256-Dateiinhalt und weiterhin 49 Events. Die bereits vorhandenen Daten werden nicht ersetzt.

Der Export für GitHub und die Streamlit-Anbindung sind noch nicht vorhanden. Die vorhandene
WeekWise-Datenbank wird nicht geöffnet; der Downloadhelfer in `importers.py` wird nur als bereits
zugelassene HTTP-Schicht verwendet. Die alten Importer und ihr Verhalten bleiben erhalten.

## Kursbezug und Dateien

- `uniclubs_importer.py`: Funktionen, Bedingungen, Schleifen, Listen, Dictionaries und Sets.
- `uniclubs_database.py`: `INSERT`, `JOIN`, SQL-Platzhalter und Transaktionen aus Unit07.section3/4.
- `importers.py`: ein zusätzlicher HTTP-Helfer mit dem bereits installierten `requests`.
- `HTMLParser`, das React-Datenformat und UTC-Normalisierung sind ausdrücklich erklärte Ergänzungen.
- Kein zusätzliches Paket und kein ORM sind erforderlich.

Tests stehen in `tests/test_uniclubs_importer.py` und `tests/test_uniclubs_download.py`.
Sie verwenden synthetische Seiten, HTTP-Mocks und SQLite im Arbeitsspeicher, keine Live-Website.

## Prüfergebnisse

- **24 neue Tests bestanden:** 17 für Parsing/Erstimport und 7 für den HTTP-Download.
- **221 Unit- und Projektregeltests insgesamt bestanden**, einschließlich der Datenbanktests
  aus Task 2 und aller bisherigen App-Tests.
- **Ein vollständiger isolierter Demo-Akzeptanztest bestanden**, einschließlich echter lokaler
  API-Anfragen und bestehender Streamlit-UI-Abläufe.
- Der echte Erstimport, die gespeicherten Mengen, SQLite-Integrität und die unveränderte Datei
  nach erneuter Ausführung wurden zusätzlich direkt geprüft.

Zum Nachprüfen aus dem Repository-Verzeichnis:

```bash
.venv/bin/python -m unittest discover -s tests -p 'test_uniclubs_*.py' -v
.venv/bin/python -m unittest discover -s tests -v
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache MPLBACKEND=Agg NO_PROXY=127.0.0.1,localhost .venv/bin/python -m views.check_demo
```

## Nächster Schritt

Task 4 ergänzt wiederholbare Abgleiche: neue Events hinzufügen, bestehende über Quellen-IDs
aktualisieren und Änderungen ohne Duplikate übernehmen. Er startet erst nach `Start Task 4`.
