<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-15, Tasks 2/3; review pending -->
# UniClubs – Task 2: Eigene SQLite-Datenbank

Dieser Bericht dokumentiert den Stand bei Abschluss von Task 2 mit leeren Tabellen.
Inzwischen hat [Task 3](UNICLUBS_TASK_3.md) den ersten echten Eventimport ergänzt.

Die neue Datei liegt unter `data/uniclubs.sqlite`. Sie ist ein eigenständiger öffentlicher
Eventkatalog. Die vorhandene WeekWise-Datenbank wird dafür weder gelesen noch beschrieben.
Die Erstellung und alle Zugriffe stehen im neuen Modul `uniclubs_database.py`.

## Erstellen und erneut öffnen

Im Repository-Verzeichnis mit der vorbereiteten Python-3.11-Umgebung ausführen:

```bash
.venv/bin/python -m uniclubs_database
```

Unter Windows wird stattdessen `.venv\Scripts\python -m uniclubs_database` verwendet.
Das Programm erstellt nur fehlende Tabellen und Indizes. Ein erneuter Aufruf löscht keine
Einträge. Die Verbindung wird am Ende geschlossen. Für diesen Task bleiben alle Tabellen leer;
es werden keine Live-Events oder Testeinträge in die echte Datei geschrieben.

## Tabellen verstehen

| Tabelle | Inhalt | Schlüssel und Regeln |
| --- | --- | --- |
| `clubs` | UniClubs-Clubkennung, Name, Quellenlink | Lokale Ganzzahl-ID; Quellenkennung eindeutig und nicht leer |
| `events` | UniClubs-Eventkennung, Clubbezug, Titel, UTC-Zeiten, Ort, Typ, Quellenlink, Status und Beobachtungszeiten | Eindeutige Quellenkennung; Club muss existieren; bekanntes Ende nach Beginn |
| `import_log` | Zeitpunkt, Ergebnis, Anzahl und kurze öffentliche Meldung eines Importversuchs | Ergebnis `success` oder `error`; Anzahl ganzzahlig und nicht negativ |

`events.club_id` verweist auf die lokale `clubs.id`. Die externe Kennung aus der Website bleibt
jeweils in `external_id`. Gleiche Titel sind erlaubt: Zwei Trainings mit verschiedenen
Quellenkennungen sind zwei Events. Derselbe Quellevent darf dagegen nicht doppelt eingefügt werden.
Bei einem solchen Versuch meldet SQLite einen `IntegrityError`; eine Aktualisierung folgt in Task 4.

`publication_status` und `availability_state` sind getrennt. Ein Event kann veröffentlicht
sein, obwohl seine Anmeldung geschlossen ist. Unbekannte Werte heißen zunächst `unknown`.
Eine unbekannte Mitgliedschaftsvoraussetzung und ein unbekanntes Ende bleiben SQL `NULL`.

## Zeitangaben

Im separaten Katalog werden UTC-Quellzeiten als Text mit stets gleicher Präzision gespeichert:

```text
2026-11-16T11:00:00.000000Z
```

Die Schreibfunktion akzeptiert ISO-UTC-Zeiten mit `Z`, auch ohne Nachkommastellen oder mit
Millisekunden, und vereinheitlicht sie. Dadurch funktionieren Sortierung und Zeitvergleich auf
TEXT-Spalten. Nicht normalisierte Zeitzonenangaben werden abgewiesen. Zeitprüfungen und die
Normalisierung erfolgen über die Python-Schreibfunktionen; direkte SQL-Zugriffe müssen diese
Konvention ebenfalls beachten.

Dies ist eine bewusste Grenze zum bisherigen App-Modell mit naiven Zurich-Zeitwerten: Die
öffentlichen Quellzeiten bleiben hier als UTC-Text erhalten. Die spätere Streamlit-Anbindung
konvertiert sie nach Europe/Zurich. An der Uhr der bestehenden App ändert Task 2 nichts.

`first_seen_utc` und `last_seen_utc` sind unsere Beobachtungszeitpunkte, keine vom Veranstalter
bestätigten Änderungsdaten. Beim erstmaligen Einfügen sind sie gleich.

## Einfache Funktionen

| Funktion | Aufgabe |
| --- | --- |
| `get_connection()` | Neue Datenbank öffnen und Fremdschlüssel aktivieren; Verbindung anschließend schließen |
| `init_db(connection)` | Die drei Tabellen und Suchindizes erstellen, vorhandene Daten erhalten |
| `add_club(connection, external_id, name, source_url)` | Club einfügen und lokale Ganzzahl-ID zurückgeben |
| `add_event(connection, record, now=None)` | Event-Dictionary prüfen und einfügen; lokale Event-ID zurückgeben |
| `get_events(connection)` | Events mit Clubnamen chronologisch lesen |
| `record_import(connection, status, event_count=0, message='', now=None)` | Ein Importergebnis protokollieren; führt keinen Import aus |
| `get_import_log(connection)` | Protokolle in umgekehrter Einfügereihenfolge lesen |

Das Event-Dictionary benötigt `external_id`, `club_id`, `title`, `start_utc` und `source_url`.
Optional sind `end_utc`, `location`, `event_type`, `publication_status`, `availability_state`
und `requires_membership` mit `True`, `False` oder `None`. `club_id` kommt aus `add_club`.

Alle Funktionen verwenden SQL-Platzhalter `?` und Parametertupel. Namen mit Apostrophen werden
als Daten behandelt. Schreibvorgänge stehen in `with connection:`, damit SQLite bei einem
Schreibfehler die jeweilige Transaktion zurückrollt. Nach einem Fehler bleiben bereits erfolgreich
gespeicherte Einträge erhalten. Stapelimporte und gemeinsame Transaktionen mehrerer Schritte
gehören noch nicht zu diesem Task.

## Bezug zum Unterricht

- **Unit07.section3/4, A07:** `sqlite3.connect`, `CREATE TABLE`, `INSERT`, `SELECT`, `JOIN`,
  Primär-/Fremdschlüssel, eindeutige Kennungen und Transaktionen.
- **Unit02.section5 und Unit03.section2:** Kleine Funktionen mit Parametern und Rückgabewerten.
- **Unit03.section3/4 und Unit04.section3:** Listen, Tupel und Dictionaries für Daten und SQL-Werte.
- **Ergänzung:** Einheitliche UTC-Textwerte und SQLite-Prüfbedingungen werden im Code erklärt.
  Die automatisierten Tests sind Entwicklungswerkzeug, kein zusätzlich behaupteter Kursstoff.

## Prüfen

```bash
.venv/bin/python -m unittest discover -s tests -p test_uniclubs_database.py -v
.venv/bin/python -m unittest discover -s tests -v
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache MPLBACKEND=Agg NO_PROXY=127.0.0.1,localhost .venv/bin/python -m views.check_demo
```

Die neuen Tests laufen ausschließlich mit einer eigenen SQLite-Datenbank im Arbeitsspeicher.
Die echte Katalogdatei wurde zweimal initialisiert und mit `PRAGMA integrity_check` sowie
`PRAGMA foreign_key_check` geprüft. Beide Prüfungen sind erfolgreich; alle drei Tabellen sind leer.

Ergebnisse der aktuellen Linux-Prüfung mit Python 3.11:

- **18 neue UniClubs-Datenbanktests bestanden.**
- **197 Unit- und Projektregeltests insgesamt bestanden.**
- **Ein vollständiger isolierter Demo-Akzeptanztest bestanden**, einschließlich frischer
  App-Datenbank, Streamlit-UI-Ablauf und echter lokaler API-Anfragen.
- Streamlit-Prozess gestartet; Health-Endpunkt antwortet `ok`, die Startseite liefert HTML.
- Die vorhandene `data/weekwise.sqlite` wurde durch Task 2 nicht angelegt oder geöffnet.

Ein gezielter Test prüft außerdem den erneuten Schema-Aufbau während einer noch offenen
Transaktion: Er wird mit einer klaren Meldung abgewiesen, ohne ausstehende Daten zu verwerfen.

Die neue Datenbank ist damit bereit für Task 3. Diese Ergebnisse prüfen noch keinen echten
UniClubs-Import und keine Aktualisierung über GitHub.

## Abgrenzung und nächste Freigabe

Die separate Datenbank ist ausdrücklich vom Nutzer gewünscht und ergänzt damit die bisherige
Spezifikation mit gemeinsamem Datenbestand. Es wurden keine neuen Pakete installiert,
keine Netzwerkabrufe implementiert und keine bestehenden App-Tabellen geändert.

Die SQLite-Datei und ihre Journaldateien sind Git-ignoriert. Das Schema, der Code und die
Anleitung ermöglichen eine neue Erstellung. Auf Streamlit Community Cloud ist eine lokale Datei
keine garantierte dauerhafte Speicherung; der Wiederaufbau aus einem öffentlichen Export wird
in den späteren Tasks behandelt. SQLite-Datenspeicherung allein liefert noch keine Aktualisierung.

Vor dem nächsten Schritt ist `Start Task 3` erforderlich: echte öffentliche UniClubs-Events
manuell abrufen und in diese neue Datenbank importieren.
