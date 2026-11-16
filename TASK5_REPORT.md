<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15; approved Task 5; reviewed by: pending -->
# Task 5 – Abgleich mit dem offiziellen Sportkalender

**Status: lokal abgeschlossen.** 967 konkrete zukünftige Termine sind durch die
offizielle Website belegt; 85 bleiben vorläufig. Task 6 wurde nicht begonnen.
Es gab keinen Push, Merge oder Eingriff in die veröffentlichte Streamlit-App.

## Quellen und Abruf

Geprüft am 15.11.2026: [offizieller Sportkalender](https://sportangebot.unisg.ch/public-discovery-schedule?language=de&selectedDate=2026-11-15).
Die Seite liefert eine JavaScript-Anwendung. Ihre öffentlich ausgelieferten Komponenten
verwenden ohne Anmeldung den lesenden Endpunkt
`https://backbone-web-api.production.st-gallen.delcom.nl/bookings/schedule-bookings`.
Es wurden keine Anmeldung, Buchung oder privaten Mitgliederabfragen ausgeführt.

Vier tatsächliche HTTP-200-Antworten wurden mit `startDateMin`, `startDateMax`,
`showBookingsFull=true`, `showCourses=true` abgerufen und privat gespeichert:

| Angefragte Woche | Antwortzeilen | Vollständigkeit |
| --- | ---: | --- |
| 16.–22.11.2026 | 311 | count = total = Zeilenzahl, page = pageCount = 1 |
| 23.–29.11.2026 | 322 | ebenso |
| 30.11.–06.12.2026 | 334 | ebenso |
| 07.–13.12.2026 | 335 | ebenso |

Zusammen: 1.302 Antwortzeilen, 1.290 unterschiedliche offizielle Termin-IDs.
Zwölf IDs kommen in mehreren Antworten mit gleichen Identitätswerten vor. Außerdem
liefert die Quelle einzelne Termine am folgenden Montag außerhalb des angefragten
Datumsbereichs. Nach Beachtung der angefragten lokalen Daten stehen **1.286 eindeutige
offizielle Termine** für diesen Abgleich zur Verfügung. Widersprüchliche doppelte IDs
würden den Import abbrechen; gleichzeitige unterschiedliche Termin-IDs werden nicht
zusammengezogen. Die öffentliche Ortsliste wurde ebenfalls angesehen, aber ihre Werte
wurden nicht pauschal in die gemeinsamen Kursfelder übernommen.

## Zuordnung und Feldfreigabe

Die bestehende lokale Prognose umfasst 263 Muster / 1.052 Termine vom 16.11. bis
13.12.2026. Für jedes Muster wurde die öffentliche Kursbezeichnung einschließlich
Level bzw. Terminbeschreibung geprüft. Übersetzungen wie „Boxing“ → „Boxen“ wurden
ausdrücklich geprüft; im Programm gibt es keinen unscharfen Namensabgleich.

- **248 Muster** haben eine freigegebene Zuordnung zu genau einer stabilen öffentlichen
  Produkt-ID über ihre eindeutigen Treffer. Daraus ergeben sich 967 belegte Termine;
  25 weitere Termine dieser Muster haben keinen exakten Treffer.
- **Fünf Muster** bleiben wegen Mehrdeutigkeit ausgeschlossen: Basketball, Volleyball
  sowie drei Tennis-Level-3-Muster mit mehreren gleichzeitigen Plätzen/Termin-IDs.
- **Zwei BGVer-Muster** bleiben wegen ungeklärter Namenszuordnung ausgeschlossen.
- **Acht weitere Muster** haben keinen exakten passenden Nachweis, darunter EMBA und Segeln.
- Die 15 nicht freigegebenen Muster betreffen 60 lokale Termine. Zusammen mit den
  25 fehlenden Einzeltreffern bleiben **85 Termine unverändert vorläufig**.

Jeder angewendete Treffer verlangt Produkt-ID, öffentliche Bezeichnung, genaue
Terminbeschreibung sowie dasselbe vollständige Datum und dieselbe lokale Startzeit.
Leere historische Varianten verlangen eine leere öffentliche Terminbeschreibung;
sie sind kein Platzhalter für beliebige Levels. Produktbezogene Levels werden über
die ausdrücklich geprüfte Produkt-ID unterschieden. Eine offizielle Termin-ID darf
höchstens einen lokalen Termin belegen. **967 Treffer verwenden 967 verschiedene
offizielle IDs.** Allgemeine Wochenpläne oder Kurslaufzeiten bestätigen keine weiteren Daten.

Alle 20 verschiedenen nichtleeren Variantenbezeichnungen aus der Prognose sind auch
öffentlich nachweisbar. Dazu gehören die zuvor gesondert geprüften kurzen Bodypump-
und langen Indoor-Cycling-Beschreibungen. Die akzeptierte Exportvorlage darf nur die
geprüfte öffentliche Kursbezeichnung und deren öffentliche Terminbeschreibung verwenden.
Die vollständigen historischen Freitexte und Originaldateien bleiben privat. Angebote
für Mitarbeitende, Teams oder Beratung werden nicht zu allgemeinen freien Studierenden-
Kursen umgedeutet; berechtigungsabhängige Preise bleiben unbekannt.

Die freigegebenen Zuordnungen gelten nur für dieses geprüfte Archiv und dessen lokale
`history-pattern:`-IDs. Sie sind keine Behauptung, dass historische und öffentliche
Datensätze dieselbe offizielle Kurs-ID hatten. Ein anderes Archiv benötigt eine neue Prüfung.

## Änderungen an der privaten Datenbank

Bei den **967 belegten Terminen** wurden ausschließlich folgende Felder geändert:

- `events.end_time`: das veröffentlichte Ende statt der bisherigen Schätzung.
- `events.end_is_estimated`: 0.
- `event_provenance.kind`: `website`.
- `event_provenance.verification_url`: die tatsächlich abgerufene öffentliche Quelle.
- `event_provenance.verified_at`: tatsächlicher Abrufzeitpunkt im lokalen Projektformat.

Die offizielle [Frontend-Komponente](https://sportangebot.unisg.ch/chunk-52TRL245.js)
addiert für Uhrzeit- und Daueranzeige eine Minute auf das inklusive API-Ende. Dieselbe
beobachtete Konvention wird übernommen; es wird nicht auf beliebige volle Stunden gerundet.
UTC-Daten werden über den vorhandenen Zeithelfer nach Europe/Zurich umgerechnet.
Ein mehrtägiges Schneesportangebot in den öffentlichen Antworten ist gültige Quelldaten;
es wird nicht wegen einer erfundenen Ein-Tages-Grenze abgelehnt.

Alle 1.052 Verfügbarkeitswerte bleiben **`unknown`**. Ein veröffentlichter Termin ist
kein Nachweis für freie Plätze, einen bestimmten Preis oder eine Teilnahmeberechtigung.
Es wurden keine Teilnehmerzahlen zu Nutzerbewertungen oder persönlichen Zusagen gemacht.
Die bestehende Sperre normaler Bestätigungen/Einladungen bei unbekannter Verfügbarkeit
bleibt bestehen. Fehlende Treffer gelten nicht automatisch als abgesagt.

Termin-IDs, Muster-IDs, Beobachtungsbasis, Evidenzwochen und ursprüngliches Erstellungsdatum
bleiben erhalten. Bestehende manuelle Endzeiten im Konflikt, neuere Website-Nachweise und
Termine ohne Herkunft sind geschützt. Ein Fehler führt zum Rollback der gesamten Änderung.
Keine Löschung, Quellenbereinigung oder automatische Wiederholung beim App-Start.

Gemeinsame Kursfelder wie Titel, Ort, Preis und Links wurden in dieser Stufe nicht
überschrieben: Eine datumsspezifische Ortsangabe darf nicht ungeprüft auf alle Wochen
übertragen werden. Die private Zuordnungsliste enthält die geprüften öffentlichen Labels
als Grundlage für den späteren gezielten Export.

## Wiederholung und Prüfung

```bash
python training_verification.py data/private/website_review/evidence.json
python -m unittest discover -s tests -v
python -m views.check_demo
streamlit run app.py
```

Der erste tatsächliche Lauf änderte 967 Termine. Ein zweiter Lauf änderte **null**
und erkannte alle 967 gespeicherten Nachweise. Das private Paket enthält die vollständigen
Quellantworten, die realen Abrufzeiten/-URLs und die ausdrücklich freigegebenen Zuordnungen.
Es ist kein frei austauschbarer Beweisgenerator; neue Daten brauchen einen neuen Abruf
und eine erneute Zuordnungsprüfung. Der Code selbst führt keinen Netzwerkaufruf aus.

- **245 lokale Tests bestanden**, darunter 18 neue Tests für Terminprüfung, leere Varianten,
  Mehrdeutigkeit, doppelte IDs, fehlerhafte Quellen, Idempotenz, Schutz und Transaktionsabbruch.
- Vollständiger isolierter Demo-/API-Ablauf mit echten lokalen HTTP-Anfragen bestanden.
- Separater Streamlit-Start: `/` = 200, `/_stcore/health` = 200 / `ok`; Server danach beendet.
- Vorher-Backup vorhanden; unabhängige Prüfung aller 967 Änderungen und ihrer Quellen.
- Erneute Task-4-Fortschreibung erzeugt keine zusätzlichen Termine und schützt alle
  967 Website-Nachweise; die übrigen 85 bleiben vorhandene Prognosen.
- Historische 9.585 Trainings, Importmetadaten, gemeinsame Kurse und Nutzerbeziehungen
  bleiben vollständig unverändert. SQLite-Integrität `ok`, keine Fremdschlüsselfehler.
- Alle Prüfungen liefen lokal unter Linux/Python 3.11. Für diese lokalen Änderungen
  wurde kein GitHub-Actions-Lauf gestartet.

## Dateien, Kursbezug und Grenze

Neu: `training_verification.py`, `tests/test_training_verification.py`, dieser Bericht.
Geändert: öffentliche Quellenkonstanten in `config.py`, README, Study Guide, AI-Protokoll.
Keine neue Abhängigkeit, Tabelle, UI-Seite oder Server-Komponente.

Verwendet werden Listen/Wörterbücher, Bedingungen, Schleifen, Funktionen, JSON,
`datetime` und direktes SQLite mit `?`-Parametern und Transaktion. Kursbezug: L04,
A06 und A07. Die beobachtete API-Feldbedeutung ist Quellenrecherche, kein erfundener
Vorlesungsinhalt. `argparse` ist wie in Task 3/4 nur der dokumentierte technische CLI-Helfer.

Private Daten und Belege liegen weiterhin unter ignoriertem `data/private/`. Die spätere
Veröffentlichung verwendet eine gezielte Feldauswahl und keine Kopie der Arbeitsdatenbank.
Teamprüfung bleibt offen; die KI-Nutzung ist dokumentiert.

**Nächster Schritt erst nach `OK Start Task 6`:** Herkunft und geprüfte Termine in die
bestehenden SQL-/API-Abfragen und den Kalender führen. Veröffentlichung bleibt Task 7.
