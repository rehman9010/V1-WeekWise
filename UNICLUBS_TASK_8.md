<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15, user-approved UniClubs Task 8; review pending -->
# UniClubs – Schritt 8: Betriebsübergabe ohne öffentliche App-URL

Status: **abgeschlossen für den URL-unabhängigen Umfang.** Die offene Frage nach der
öffentlichen Streamlit-Adresse muss dafür nicht beantwortet werden. Dieser Schritt stellt
einen geprüften Datenstand und eine Betriebsanleitung bereit; eine unbekannte öffentliche
App wird damit nicht als geprüft oder neu veröffentlicht bezeichnet.

## Bereitgestelltes Paket

Im Chat-Arbeitsbereich liegt `artifacts/uniclubs-task8.zip`. Es enthält:

| Datei | Zweck |
| --- | --- |
| `uniclubs.sqlite` | Neu erstellte, direkt lesbare SQLite-Datei: 49 Events, 23 Clubs. |
| `uniclubs_events.json` | Validierter öffentlicher Datenstand aus GitHub. |
| `uniclubs.sql` | Lesbare CREATE-/INSERT-Anweisungen für denselben Stand. |
| `README.md` | Deutsche Anleitung zum Öffnen mit Python und zum Betrieb der App. |
| `manifest.json` | Quellen-Commit, Datenzeitpunkt, Prüfergebnisse und SHA-256-Prüfsummen. |

Die Datei wurde ausschließlich aus dem öffentlichen JSON neu aufgebaut. Die alte WeekWise-
Datenbank, vorhandene persönliche Dateien und der schon eingerichtete App-Katalog wurden
dabei weder kopiert noch verändert. Das Paket enthält nur `clubs`, `events` und einen
öffentlichen Wiederherstellungseintrag in `import_log`.

Die Dateien sind als Chat-Artefakt bereitgestellt und werden nicht zusätzlich in Git committed.
Im Repository bleibt der kleine öffentliche JSON-Feed der Austauschweg für die Automation.

## Herkunft und Prüfung

- Grundlage ist der öffentliche Feed aus Commit
  `bccefd3be3ec1f8dd3ff1149e7df460d3625ec20`:
  [commit-spezifischer JSON-Feed](https://raw.githubusercontent.com/rehman9010/V1-WeekWise/bccefd3be3ec1f8dd3ff1149e7df460d3625ec20/data/public/uniclubs_events.json).
- Zeitpunkt des tatsächlichen Quellenabgleichs: **15.11.2026, 18:49:28 UTC**.
- Paket erstellt und geprüft: **15.11.2026, 19:18:31 UTC**; Quelldaten dabei etwa 29 Minuten alt.
  Die Paketerstellung wird nicht als neuer UniClubs-Quellenabruf ausgegeben.
- JSON → neue SQLite-Datei → JSON: vollständige Übereinstimmung, einschließlich IDs,
  Erst-/Letztbeobachtungen, Clubbezug und Fehlmarkern.
- SQLite → SQL-Dump → weitere leere Datenbank → JSON: ebenfalls vollständige Übereinstimmung.
- SQLite-Integrität `ok`, keine Fremdschlüsselfehler, nur die drei vorgesehenen Tabellen.
- Nach Schließen erneut schreibgeschützt geöffnet; 49 Eventzeilen vorhanden.
- Kalenderadapter: 21 Angebote in der Woche ab 16.11.2026.
- Dateigrößen und alle vier im Manifest erfassten SHA-256-Prüfsummen stimmen.
- ZIP-CRC und alle archivierten Dateiinhalte stimmen mit den geprüften Dateien überein.

SHA-256 des ZIP-Pakets:

```text
4202ee329321b414a49b946d208e2a5296c5cb6349b048fe8427d9c1f72f1789
```

Auch der [Task-7-Prüflauf auf main](https://github.com/rehman9010/V1-WeekWise/actions/runs/43750991770)
ist inzwischen vollständig erfolgreich: Linux, Windows und macOS, 357 Unit-/Projektregeltests
und alle drei App-Abnahmen einschließlich der vier Prozessneustartphasen.
Für dieses reine Daten-/Dokumentationspaket wurden keine Produktfunktionen oder Abhängigkeiten
geändert und diese Prüfungen nicht unnötig nochmals gestartet.

## So verwendet ihr die Datenbank

1. ZIP entpacken und `README.md` öffnen.
2. Die dortige Python-Abfrage im entpackten Ordner ausführen oder `uniclubs.sqlite` mit einer
   SQLite-Anwendung öffnen. Es werden keine UniClubs-Zugangsdaten benötigt.
3. `clubs` enthält die Clubs. `events.club_id` verweist auf `clubs.id`; ein JOIN liefert deshalb
   Eventtitel und Clubname gemeinsam. Die IDs bleiben zum öffentlichen Feed identisch.
4. `start_utc`/`end_utc` sind UTC. Ein unbekanntes Ende bleibt `NULL`; die App übernimmt die
   Umrechnung nach Europe/Zurich und die Regeln für unklare Dauer und Anmeldestatus.

Die Download-Datei ist ein fester Datenstand. Der laufende automatische Abgleich bleibt Teil
der App-/GitHub-Integration: GitHub erzeugt den öffentlichen Feed, die App lädt ihn in ihre
separate `data/uniclubs.sqlite`. Für den normalen Betrieb muss das Paket nicht manuell in
die App kopiert werden.

## Betrieb ohne Kenntnis der öffentlichen App-Adresse

1. Im Projektordner mit aktivierter Python-3.11-Umgebung `python -m streamlit run app.py`
   ausführen. Die Datenquelle ist bereits konfiguriert.
2. In der Wochenansicht **UniClubs this week** öffnen; für den Prüfkatalog eine Woche mit
   Terminen wählen, etwa ab 16.11.2026.
3. **Source updated** zeigt das Alter des Herausgeberstands, **Last checked** den App-Prüfversuch.
   **Refresh UniClubs** versucht einen sofortigen Download.
4. Den Quellenabgleich in GitHub unter **Actions → UniClubs public catalog** prüfen oder dort
   mit **Run workflow** starten. Nur dieser Job benötigt das automatisch bereitgestellte
   GitHub-Token; ihr müsst kein UniClubs-Konto oder UniClubs-API-Token beschaffen.

Der Workflow ist weiterhin **active**, mit geplantem Abgleich zu den Minuten 17 und 47 jeder
UTC-Stunde. Beim Check am 15.11.2026 um 19:21 UTC war kein durch `schedule` gestarteter Lauf
beobachtbar; die erfolgreichen manuellen Ausführungen sind in Task 5 und 7 belegt.
Der Zeitplan ist eingerichtet, eine konkrete Cron-Ausführung wird hier nicht erfunden.

Offene App-Sitzungen prüfen alle 60 Sekunden den fünf Minuten gecachten Feed. Netzfehler
lassen den letzten gültigen Katalog verfügbar. GitHub-Verzögerungen und das Cache-Intervall
bedeuten, dass keine Echtzeitzustellung zugesagt wird.

## Dateien, Lernstoff und Grenze dieses Schritts

Im Repository geändert: dieser Bericht, README, Study Guide und AI-Nutzungsdokumentation.
Außerhalb des Repositories erstellt: Datenpaket und temporäres Erstellungs-/Prüfskript.
Keine Änderungen an App-Code, Datenbankkonfiguration, Cloud-Umgebung oder privaten Dateien.

Der neue Aufbau verwendet die vorhandenen Funktionen `get_connection`, `init_db`,
`restore_export` und `build_export`. JSON, Tabellen, Fremdschlüssel und JOINs gehören zu eurem
bisherigen Lernpfad; ZIP und SHA-256 sind erklärte Übergabe-/Prüfwerkzeuge.

Die externe öffentliche Streamlit-Instanz bleibt ohne ihre URL außerhalb des Nachweises.
Diese Grenze hindert weder die Betriebsübergabe noch die Nutzung der bereitgestellten Datenbank.
Ein weiterer Schritt wird erst auf eure Anweisung begonnen.
