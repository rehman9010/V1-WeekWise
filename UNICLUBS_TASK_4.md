<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-15, historical Task 4 report; review pending -->
# UniClubs – Task 4: Wiederholbare Abgleiche

Dieser Bericht dokumentiert den manuellen Abgleich aus Task 4. [Task 5](UNICLUBS_TASK_5.md)
ergänzt den öffentlichen Export und die Konfiguration für zeitgesteuerte GitHub-Läufe.

Der neue Katalog kann jetzt wiederholt mit der öffentlichen UniClubs-Quelle abgeglichen werden.
Dabei werden neue Events hinzugefügt und bestehende Events aktualisiert, ohne ihre lokale
Identität zu ersetzen. Die Datei bleibt `data/uniclubs.sqlite`; die bisherige App-Datenbank
wird dafür nicht geöffnet.

## Ausführen

Im Repository-Verzeichnis, mit der vorhandenen Python-3.11-Umgebung:

```bash
.venv/bin/python -m uniclubs_importer
```

Unter Windows lautet der Befehl `.venv\Scripts\python -m uniclubs_importer`.
Dieser Befehl funktioniert für einen leeren und für einen bereits gefüllten Katalog.
Er startet genau einen Abgleich und beendet sich danach. Der frühere Erstimport-Helfer
`run_import` bleibt für die Task-3-Tests erhalten; die Kommandozeile verwendet nun `run_sync`.

## Verhalten anhand einfacher Fälle

| Situation | Ergebnis |
| --- | --- |
| Neue UniClubs-ID mit künftigem Beginn | Neuer Event wird eingefügt |
| Gleiche ID, geänderter Titel, Ort oder Termin | Bestehender Event wird aktualisiert |
| Gleiche ID, gleiche Daten | Keine zweite Veranstaltung; Beobachtungszeit wird erneuert |
| Bekannter Event wird in Vergangenheit verschoben | Auch seine neue Zeit wird übernommen |
| Neuer, schon vergangener oder laufender Event | Wird nicht erstmals in den Katalog aufgenommen |
| Geänderter Clubname oder Clubwechsel | Clubdaten bzw. Clubzuordnung werden aktualisiert |
| Event fehlt im verifizierten Quellenbestand | Daten bleiben erhalten, `source_missing=1` |
| Event erscheint später mit derselben Quellen-ID wieder | Fehlmarker wird zurückgesetzt |
| Quelle nennt ausdrücklich einen Absagestatus | Dieser Quellstatus wird gespeichert |
| Nur die Anmeldung ist geschlossen | Keine Absage daraus ableiten |

Ein gleichbleibender Titel ist keine Eventkennung. Zwei verschiedene Quellen-IDs dürfen den
gleichen Titel haben, etwa bei mehreren Trainings. Umgekehrt bleibt bei einer Terminverschiebung
die lokale Ganzzahl-ID erhalten. `first_seen_utc` bleibt ebenfalls erhalten.

## Vollständigkeit vor dem Schreiben prüfen

1. Die öffentliche, ungefilterte HSG-Eventübersicht herunterladen.
2. Mindestens eine Sekunde warten und die öffentliche Sitemap herunterladen.
3. Die vollständige eingebettete Eventliste und ihre Clubzuordnungen validieren.
4. Sichtbare Links gegen diese Daten prüfen.
5. Alle HSG-Eventlinks der Sitemap mit allen gelieferten Eventlinks vergleichen, einschließlich
   historischer Events. Erst wenn die Mengen übereinstimmen, wird geschrieben.

Damit wird auch eine verkürzte Übersicht erkannt, deren sichtbare Karten und eingebettete Daten
gemeinsam verkürzt wurden, solange die Sitemap den vollständigen Bestand enthält. Das ist keine
Garantie gegen eine gleichzeitige unbemerkte Verkürzung aller Quelldokumente.

Eine echte Websiteänderung zwischen beiden Abrufen kann ebenfalls einen Unterschied verursachen.
In diesem Fall bricht der Lauf vorsichtshalber ab. Ein späterer manueller oder geplanter Lauf kann
es erneut versuchen; es gibt keine automatische schnelle Wiederholungsschleife.

Leere, beschädigte, zu große, doppelte oder widersprüchliche Quellen werden weiterhin abgewiesen.
Beide Downloads behalten TLS-Prüfung, Timeout und Größenbegrenzung bei. Es werden keine privaten
Bereiche, API-Endpunkte oder Benutzerkonten benötigt. Beschreibungen, Bilder und Zugangscodes
werden nicht in den Katalog übernommen.

## Fehlende Quelle ist keine Absage

Die neue Spalte `events.source_missing` ist ein separater 0/1-Wert. Sie zeigt ausschließlich,
dass die Event-ID im letzten verifizierten Quellenbestand nicht vorkam. Titel, Ort, Zeiten,
Veröffentlichungsstatus und letzte bestätigte Beobachtung bleiben dabei erhalten.

Der Marker führt zu keiner Löschung oder automatisch erfundenen Absage. Die spätere Oberfläche
muss einen solchen Hinweis verständlich anzeigen; diese Streamlit-Anbindung ist noch nicht Teil
von Task 4. Bei Wiederauftauchen wird derselbe Datensatz aktualisiert und der Marker gelöscht.

`publication_status` und `availability_state` bleiben getrennt. Beispielsweise bedeutet
`sales_closed` nur geschlossene Anmeldung. Absagefälle wurden mit synthetischen Quelldaten
getestet; während der echten Prüfläufe wurde keine neue Absage beobachtet.

## Bestehende Daten erhalten

Das Programm prüft beim Start mit `PRAGMA table_info(events)`, ob die neue Spalte vorhanden ist.
Falls nötig, wird sie mit `ALTER TABLE` ergänzt. Der Standardwert ist 0. Es werden keine Tabellen
gelöscht oder neu aufgebaut. Vor der ersten echten Migration wurde eine konsistente Sicherung
außerhalb des Checkouts unter `/workspace/uniclubs-research/backups/` angelegt.

Die 49 Events aus Task 3 behalten nach Migration und Abgleichen ihre lokalen IDs und
Erstbeobachtungszeiten. Alle SQL-Schreibvorgänge eines Abgleichs einschließlich des Erfolgslogs
werden gemeinsam bestätigt oder gemeinsam zurückgerollt.

Bekannte Events werden auch dann aus der vollständigen Quelle aktualisiert, wenn ihr Start schon
vorbei ist. Für bisher unbekannte Events gilt weiterhin der Zukunftsfilter. Eine leere Zukunftsliste
ist deshalb zulässig, wenn der vollständige öffentliche Quellenbestand gültig und nicht leer ist.

## Zähler und Protokolle verstehen

- `events_added`: neue Events.
- `events_updated`: geänderte Eventdaten oder wiederhergestellte Quellenzuordnung.
- `events_unchanged`: gleiche Eventdaten; lediglich `last_seen_utc` wird erneuert.
- `events_missing`: alle gespeicherten IDs, die im gültigen Quellenbestand fehlen.
- `events_marked_missing`: davon in diesem Lauf erstmals markierte IDs.
- `events_restored`: wieder aufgetauchte IDs; diese sind zusätzlich in `events_updated` enthalten.
- Für Clubs gibt es entsprechende Zähler für neue, geänderte und unveränderte Einträge.

Ein unveränderter Abgleich erneuert Beobachtungszeiten und schreibt ein neues Erfolgsprotokoll.
Er verändert deshalb die Datei, aber weder die Anzahl der Events noch deren Identitäten.

Bei einem Abruf-, Prüf- oder Schreibfehler bleiben die Katalogdaten auf dem letzten gültigen Stand.
Nach einem Rollback wird ein kurzer, bereinigter Fehlerbericht separat gespeichert. Laufende
fremde Transaktionen werden vorab abgewiesen, ohne ausstehende Änderungen zu verwerfen.
Eine ältere Beobachtungszeit darf keinen neueren erfolgreichen Stand überschreiben.

## Kursbezug

Die Aktualisierung verwendet die im Kurs behandelten Befehle `SELECT`, `INSERT` und `UPDATE`
mit SQL-Platzhaltern sowie Listen, Dictionaries, Sets und kleine Funktionen. Es gibt kein ORM
und keine neuen Pakete. XML-Auswertung, die zusätzliche Spalte per `ALTER TABLE` und der kurze
Abstand zwischen zwei Abrufen sind erklärte Ergänzungen zur bestehenden Umsetzung.

## Prüfergebnisse am 15. November 2026

- **21 neue Synchronisierungstests und 4 zusätzliche Downloadtests bestanden.**
- **246 Unit- und Projektregeltests insgesamt bestanden.**
- **Ein vollständiger isolierter Demo-Akzeptanztest bestanden**, einschließlich der bisherigen
  Streamlit-Abläufe und echter lokaler API-Anfragen.
- Zwei echte Abgleiche meldeten jeweils **0 neue, 0 geänderte und 49 unveränderte Events**.
- Die ursprünglichen 49 lokalen Event-IDs und Erstbeobachtungszeiten stimmen weiterhin mit der
  Sicherung vor Task 4 überein. Der Katalog enthält 23 Clubs und keine fehlend markierten Events.
- SQLite-Integrität und Fremdschlüssel sind fehlerfrei.

Terminverschiebungen, neue Events, explizite Absagen, Clubwechsel, fehlende und wiederkehrende
Quellen wurden mit kontrollierten Testdaten geprüft. Ein SQLite-Trigger löste außerdem einen
Fehler nach tatsächlichen Änderungen aus; sämtliche Daten und der Erfolgslog wurden zurückgerollt.
Die echten UniClubs-Daten wurden für diese Szenarien nicht verändert.

Zum Nachprüfen:

```bash
.venv/bin/python -m unittest discover -s tests -p 'test_uniclubs_*.py' -v
.venv/bin/python -m unittest discover -s tests -v
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache MPLBACKEND=Agg NO_PROXY=127.0.0.1,localhost .venv/bin/python -m views.check_demo
```

## Nächster Freigabepunkt

Der Abgleich läuft bislang manuell. Ein GitHub-Actions-Zeitplan und der öffentliche Eventexport
gehören zu Task 5. Die Streamlit-Anbindung folgt danach. Es wurde nichts veröffentlicht.
Task 5 startet erst nach `Start Task 5`.
