<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15, user-approved UniClubs Task 5; review pending -->
# UniClubs – Task 5: Öffentlicher Export und GitHub Actions

Der Abgleich aus Task 4 erhält einen öffentlichen JSON-Export und einen GitHub-Actions-Workflow.
Der Workflow verwendet Python 3.11 und die vorhandenen Projektpakete. Er benötigt kein UniClubs-
Konto und keinen manuell eingetragenen API-Schlüssel.

## Datenfluss

```text
Bisheriger öffentlicher JSON-Export aus GitHub
                 ↓
Eigene SQLite-Datenbank des neuen GitHub-Runners wiederherstellen
                 ↓
Öffentliche UniClubs-Übersicht und Sitemap verifiziert abgleichen
                 ↓
Neuen JSON-Export vollständig prüfen und atomar schreiben
                 ↓
Nur data/public/uniclubs_events.json auf GitHub aktualisieren
```

Die SQLite-Datei selbst wird weiterhin nicht in Git gespeichert. Der JSON-Export erhält die
Quellenkennungen, lokalen IDs, Clubverknüpfungen, Erst-/Letztbeobachtung und Fehlmarker. Dadurch
verliert ein frischer Runner weder vergangene bekannte Events noch den Hinweis auf fehlende Quellen.
Die frühere Datenbank mit Profilen, Bewertungen und privaten App-Daten ist nicht beteiligt.

## Zeitplan und manuelle Ausführung

Workflowdatei: `.github/workflows/uniclubs-sync.yml`.

- Geplanter Start: Minuten **17 und 47 jeder UTC-Stunde**, also alle 30 Minuten.
- Manueller Start: GitHub → Actions → **UniClubs public catalog** → **Run workflow**.
- Der Workflow verwendet ausschließlich den Defaultbranch, in diesem Repository `main`.
- Parallel laufende Katalogjobs werden verhindert; ein laufender Abgleich wird nicht abgebrochen.
- GitHub kann geplante Läufe verzögern. Das ist keine Echtzeitgarantie.

Der Zeitplan wird erst wirksam, wenn Workflow und benötigter Code auf dem Defaultbranch liegen.
Die Rechte des automatisch bereitgestellten `GITHUB_TOKEN` werden für diesen Job auf
`contents: write` gesetzt. Es wird kein persönlicher Token benötigt und kein Geheimnis im Code
gespeichert. Branchschutz oder Organisationsrichtlinien können den Bot-Push verhindern; sie
werden nicht automatisch abgeschaltet. Manuelle Starts von einem anderen Branch werden übersprungen.

Die offiziellen GitHub-Actions-Versionen sind mit zuvor überprüften vollständigen Commit-SHAs
festgelegt. Der Job installiert die gepinnten Projektpakete und prüft vor dem Abruf die
UniClubs-Tests. Erst nach einem erfolgreichen Abgleich wird ausschließlich die öffentliche
JSON-Datei gestaged, committed und mit normalem Push übertragen. Ein zwischenzeitlicher
Branchwechsel durch andere Änderungen führt zu einem fehlgeschlagenen Push statt Überschreiben.

## Exportformat

Datei: `data/public/uniclubs_events.json`.

- `_meta`: Formatversion 1, Quellenlink, AI-Deklaration und Zeitpunkt des letzten Erfolgs in UTC.
- `clubs`: lokale ID, Quellen-ID, Clubname und öffentlicher Club-Link.
- `events`: IDs und Clubbezug, Titel, Beginn/Ende, Ort, Typ, Quellenlink, Veröffentlichungs- und
  Anmeldestatus, Mitgliedschaftsvoraussetzung, Beobachtungszeiten und Fehlmarker.

Der Export enthält eine ausdrücklich festgelegte Feldliste. Er enthält keine Benutzerprofile,
privaten Kalender, vollständigen Beschreibungen, Bilder, Zugangscodes oder Importprotokoll-Historie.
Unbekanntes Ende und unbekannte Mitgliedschaft bleiben `null`; Kennzeichnungen sind JSON-Booleans.
Bekannte historische und fehlend markierte Events bleiben als Katalogzustand erhalten. Die spätere
Kalenderanzeige filtert nach Zeitraum; sie wird in Task 6 ergänzt.

Vorgesehene öffentliche Feed-Adresse des Repositories:

```text
https://raw.githubusercontent.com/rehman9010/V1-WeekWise/main/data/public/uniclubs_events.json
```

Die tatsächliche Erreichbarkeit wird separat von der lokalen Dateierstellung überprüft.

## Lokal ausführen

Aus dem Repository-Verzeichnis:

```bash
.venv/bin/python -m uniclubs_refresh
```

Unter Windows wird `.venv\Scripts\python -m uniclubs_refresh` verwendet.

Wenn die öffentliche Datei fehlt, kann ein erster gültiger Quellenabruf einen Katalog erzeugen.
Eine vorhandene beschädigte oder inkompatible Datei führt dagegen zum Abbruch. Bei einer frischen
leeren SQLite-Datenbank wird die vorhandene öffentliche Datei zuerst wiederhergestellt.
Eine schon vorhandene lokale Datenbank darf nicht älter als dieser gespeicherte Export sein;
sonst wird eine bewusst frische Katalogdatei benötigt. Vorhandene Daten werden nicht gelöscht.

Nach einem erfolgreichen Datenbankabgleich kann ein Dateischreibfehler dazu führen, dass die
lokale Datenbank neuer als die öffentliche Datei ist. Die vorherige öffentliche Datei bleibt
intakt; ein späterer erfolgreicher Lauf kann sie aktualisieren. Ein fehlgeschlagener Workflow
committed und pusht keinen neuen Export.

## Häufigkeit und Grenzen

Erfolgreiche Läufe erneuern die Beobachtungszeiten. Daher kann auch ohne geänderte Eventdetails
alle 30 Minuten ein Datencommit entstehen. Diese kleinen Commits tragen `[skip ci]`, damit sie
nicht zusätzlich die gesamte plattformübergreifende App-CI starten. Sie lösen keinen erneuten
Katalogjob aus; der Workflow reagiert nur auf Zeitplan oder manuellen Start.

Bei Konflikten wird weder automatisch rebased noch mit Force gepusht. Der nächste Lauf beginnt
vom aktuellen Defaultbranch. Die Größen- und Datensatzgrenzen verhindern übergroße oder
widersprüchliche Exporte; ein überschrittener Grenzwert muss bewusst untersucht werden.

Die Streamlit-App liest diesen Feed noch nicht. Das wird erst nach Freigabe von Task 6 umgesetzt.
Die Cloud-Umgebung und ihre Konfiguration wurden für Task 5 nicht veröffentlicht oder verändert.

## Lokale Prüfergebnisse

- **19 Export-/Wiederherstellungstests und 9 Ablauf-/Fehlertests bestanden.**
- **95 UniClubs-Tests und 274 Unit-/Projektregeltests insgesamt bestanden.**
- **Ein vollständiger isolierter Demo-Akzeptanztest bestanden**, einschließlich der bisherigen
  Streamlit-Abläufe und echter lokaler API-Anfragen.
- Der echte Export enthält 49 Events und 23 Clubs. Eine frische SQLite-Datenbank wurde aus einer
  Kopie wiederhergestellt, gegen die echte UniClubs-Quelle abgeglichen und erneut exportiert.
  Alle Quellen-/lokalen IDs und Erstbeobachtungszeiten blieben erhalten.
- `actionlint` prüfte die Workflowdatei erfolgreich. Das offizielle Prüfprogramm wurde nach
  Verifikation seines veröffentlichten SHA-256-Checksums außerhalb des Projekts verwendet.
- Der tatsächliche Commit-/Push-Shellschritt wurde gegen ein temporäres lokales Git-Repository
  ausgeführt. Nur die öffentliche Datei wurde committed; eine andere vorbereitete Änderung
  wurde nicht mit veröffentlicht. Es wurde kein Forcepush verwendet.

Diese lokalen Ergebnisse sind kein Nachweis eines GitHub-Laufs. Aktivierung setzt den Merge
auf `main` voraus. Danach sind die Workflowdatei auf dem Defaultbranch, ein erfolgreicher
GitHub-Lauf und der anonyme Abruf der oben genannten Raw-Adresse gesondert zu prüfen.
