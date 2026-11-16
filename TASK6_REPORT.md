<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15; approved Task 6; reviewed by: pending -->
# Task 6 – geprüfte Termine in Datenbank, API und Kalender

**Status: lokal abgeschlossen.** Die vorhandene App und API verwenden jetzt den
vorbereiteten Kursbestand mit **967 belegten und 85 vorläufigen Terminen** vom
16.11. bis 13.12.2026. Task 7 wurde nicht begonnen; kein Push, Merge oder Deployment.

## Akzeptanz und tatsächliches Ergebnis

| Prüfung | Ergebnis |
| --- | --- |
| Gemeinsame SQL-/API-Datenform mit Herkunft | umgesetzt, einschließlich älterer Demo-Datensätze |
| App-Bestand | 263 Kursmuster / 1.052 Termine; ursprüngliche IDs erhalten |
| Website-Nachweis | 967, mit tatsächlicher Quelle und Prüfzeit |
| Vorläufig | 85, mit Beobachtungsbasis/Evidenzwochen |
| Historische Quelle | alle 9.585 Zeilen und sämtliche SQL-Werte aus Task 5 unverändert |
| Private Daten im App-Bestand | keine Archivzeilen, Dateiherkünfte, Teilnehmerstatistiken oder kopierten Profile |
| Wiederholte Vorbereitung | null neue Termine, gleiche Bestandskennung |
| Verfügbarkeit | alle `unknown`, unabhängig vom Website-Nachweis |
| Bestehender Demo-Ablauf | weiterhin funktionsfähig, einschließlich trainiertem Empfehlungsmodell |

Die lokale Standarddatei `data/weekwise.sqlite` wurde ausdrücklich vorbereitet; sie
existierte zuvor nicht. App und `python api_server.py` nutzen damit denselben Bestand.
Zusätzlich liegt eine getrennte Vorschau unter `data/private/training_app.sqlite`.
Beide Dateien sind ignoriert und wurden nicht zu Git hinzugefügt. Die private Quelle
bleibt `data/private/training_history.sqlite`.

## Sichtbares Verhalten

Kurskarten und Details zeigen **Website checked** oder **Provisional**. Bei belegten
Terminen sind Prüfzeit und tatsächliche Nachweisquelle sichtbar; der normale Link führt
zum ausgewählten Tag im offiziellen Sportkalender. Prognosen erklären Beobachtungsdatum,
Evidenzwochen und gegebenenfalls geschätztes Ende. Ältere Termine ohne Herkunft erhalten
keinen erfundenen Website-Nachweis.

Alle Termine bleiben Vorschläge, solange Verfügbarkeit unbekannt ist. **Confirm** ist
deaktiviert; Einladungen können nicht gesendet oder angenommen werden. Der bestehende
Datenbank-Zustandsautomat erzwingt dieselbe Regel auch über die API. Favorisieren,
Filtern, Wochen-/Tagesnavigation und Details bleiben nutzbar. Unbestätigte Kursideen
belegen keine private Kalenderzeit.

Die App kopiert ausschließlich freigegebene öffentliche Kurslabels aus der Task-5-Liste.
**15 nicht freigegebene Muster mit 60 vorläufigen Terminen** erhalten eine neutrale
Beschriftung statt ungeprüfter Originaltitel. Die übrigen 25 vorläufigen Termine gehören
zu bereits freigegebenen Kursmustern. Orte, Preise und Teilnahmeberechtigungen werden
nicht erfunden; öffentlich erkennbare Einschränkungen bleiben im geprüften Kurslabel.

## Datenfluss und Schutz des Bestands

`training_catalogue.py` liest die Quelle für den CLI-Lauf über eine schreibgeschützte
Verbindung. Ein neuer App-Bestand erhält nur ausgewählte Kurse, Termine und deren
Herkunft, mit Zuordnung über die eingefrorenen Kategorie-Schlüssel. Keine komplette
Kopie der Arbeitsdatenbank. Die Vorlage enthält leere Archivtabellen als Teil des
vorhandenen Schemas, aber keine privaten Archivdaten.

Aktivitätszeilen und Bestandsmarker werden gemeinsam in einer Transaktion geschrieben.
Schema/Kategoriedefinitionen können bereits vor dieser Transaktion angelegt sein;
bei Fehlern bleiben keine teilweise importierten Aktivitäten oder Bestandsmarker zurück.
Ein belegtes Ziel mit anderen Daten wird abgelehnt. Wiederholung derselben Vorlage
ersetzt keine Profile oder manuellen Änderungen; ein veränderter Quellbestand verlangt
ein ausdrücklich neu gewähltes Ziel.

Der App-Start im vorbereiteten Modus überspringt Demo-Kurse, künstliche Nutzer,
synthetische Bewertungen und Quellenimport. Ein Demo-Privatkalender kann weiterhin
beispielhafte belegte Zeiten zeigen. Die App-Quelle und der Privatkalender sind getrennt.
Das direkte Starten von App oder API mit einem gefüllten Trainingsarchiv wird abgelehnt.

**Reload saved catalogue** liest gespeicherte Daten und behauptet keine neue Website-
Prüfung. **Reset personal data** löscht Nutzerdaten und Einstellungen, erhält aber
Kurse, Termine, Herkunft, Kategorieversion und Bestandskennung. Generische Importe
überspringen den vorbereiteten Bestand und das Privatarchiv. Normale Upserts und
Quellenbereinigung schützen zusätzlich bereits belegte Herkunftszeilen. Eine explizite
normale Vergangenheitsbereinigung entfernt Herkunft passend zum gelöschten Termin;
das historische Archiv bleibt dabei erhalten.

## API-Vertrag

Die bestehenden Routen bleiben erhalten. Die SQL-Abfragen für Zeitraum, Einzeltermin,
Bestätigungen und ML-Trainingsdaten verwenden `LEFT JOIN event_provenance`.
Zusätzliche nullable Felder: `provenance_kind`, `forecast_anchor_date`, `evidence_weeks`,
`verification_url`, `verified_at`. Die Antwort enthält weiterhin nur ausgewählte
öffentliche Kursfelder und keine Roharchivdaten. Vollständige alte Antworten werden
als Herkunft unbekannt akzeptiert; teilweise oder widersprüchliche Herkunft wird abgelehnt.

`X-WeekWise-Catalogue` kennzeichnet den vorbereiteten Bestand. Der Client sendet die
lokale Kennung und verlangt die gleiche Antwortkennung. Bei falschem/fehlendem Nachweis
oder ausgefallener API nutzt er dieselben Daten direkt aus SQLite. Die Kennung ist
kein Geheimnis und keine Benutzeranmeldung. Die beiden Prozesse sollen weiterhin dieselbe
Datenbankdatei verwenden. Der ursprüngliche Demo-Modus verwendet die Kennung `demo`.

Vor Schreibanfragen im vorbereiteten Modus wird die API-Kennung frisch geprüft, damit
ein veralteter Gesundheitscache keinen alten Server ohne diesen Vertrag beschreiben
lässt. Bei fehlgeschlagener Prüfung gibt es keinen POST. Eine erfolgreiche Prüfung
kann von einem POST gefolgt werden; jeder Aufruf hat den vorhandenen Timeout. Der
Server lehnt falsche Kennungen mit 409 und unbekannte Verfügbarkeit mit 400 ab.
Es wurde kein neuer Server-Typ oder öffentlich erreichbarer Buchungsdienst eingeführt.

## So prüfen – ungefähr fünf Minuten

Mit Python 3.11 und vorhandener privater Task-5-Quelle:

```bash
python training_catalogue.py --output data/weekwise.sqlite
streamlit run app.py
```

Ein Ziel mit abweichenden vorhandenen Daten wird geschützt. Als Alternative die
separate Vorschau vorbereiten und `WEEKWISE_DB_FILE` in beiden Prozessen auf denselben
Pfad setzen; Bash-/PowerShell-Beispiele stehen in README. In einem zweiten Terminal:

```bash
python api_server.py
```

Onboarding mit eigenem Namen und Fokus durchlaufen; Demo-Historie ist hier deaktiviert.
Die Woche ab 16.11. wählen, einen belegten und einen vorläufigen Termin öffnen,
Herkunft/Quelle und deaktiviertes Confirm prüfen. In Settings → Data sources die
967/85-Zähler und den gespeicherten Reload ansehen. Optional die API lesen:

```text
http://127.0.0.1:5001/events?from=2026-11-16&to=2026-12-14
```

## Ausgeführte Prüfungen

- **297 Unit-/Regeltests bestanden**, einschließlich 52 zusätzlicher Tests für Herkunft,
  öffentliche Vorlage, API-Identität, Lebenszyklus und UI-Regeln.
- Der ursprüngliche vollständige isolierte Demo-/API-Ablauf ist bestanden; der trainierte
  Empfehlungsweg, Feedback, Freunde und Datenbank-Fallback funktionieren weiterhin.
- Native Streamlit-App-Prüfung auf einer temporären Kopie aller echten 1.052 Termine:
  Onboarding, Wochenwechsel, beide Herkunftsdetails, Quelllinks, deaktiviertes Confirm,
  neutrale Titel, neue Sitzung, Reload und Profil-Reset erfolgreich. Alle 1.052 Termin-
  und Herkunfts-IDs bleiben dabei erhalten; keine künstlichen Nutzer entstehen.
- Echter lokaler HTTP-Server auf einer separaten Kopie: alle 1.052 API-Datensätze stimmen
  vollständig mit SQLite-Objekten überein; 967/85 stimmen. Einzeltermine und Fallback
  nach Server-Ende geprüft. Kein Archiv-Endpunkt vorhanden; private Quelldatei abgelehnt.
- Falsche Bestandskennung: 409; Bestätigungen beider Herkunftsarten: 400, keine Interaktion.
  Tests mit kollidierenden IDs in zwei Datenbanken belegen den Schutz vor falschen Writes.
- SQL-Integrität `ok`, keine Fremdschlüsselfehler. Vergleich der gesamten privaten
  Task-5-Datenbank auf Ebene der SQL-Werte: unverändert. QA-Kopien verändern weder
  ursprüngliches Archiv noch Vorschau.
- Separater Start der Standard-App: `/` = 200 und `/_stcore/health` = 200 / `ok`.
  Die eigenen Testserver wurden anschließend beendet.
- Linux/Python 3.11; kein neuer GitHub-Actions- oder nativer Windows/macOS-Lauf behauptet.

## Dateien, Entscheidungen und Kursbezug

Neu: `training_catalogue.py`, vier synthetische Testmodule und dieser Bericht.
Geändert: `config.py`, `database.py`, `models.py`, `api_server.py`, `api_client.py`,
`app.py`, `importers.py`, `ui_components.py`, Onboarding/Settings, bestehende API-Tests
sowie README, Study Guide und KI-Protokoll. Keine neue Abhängigkeit, Tabelle oder App-Seite.

Verwendet: SQLite/SQL mit Parametern und Transaktionen (A07), einfache Objekte/Validierung
(L05/U05 s2), HTTP/JSON/Headers mit dem bestehenden Flask und requests (L06/A06),
Listen/Wörterbücher und native Streamlit-Widgets (L04). `argparse`/`hashlib` bleiben die
bereits dokumentierten technischen Helfer. SPEC/CLAUDE und die Kategorie-Schlüssel
wurden nicht geändert. Die freigegebene Task-Planung ergänzt den älteren API-Vertrag
gezielt um Herkunft und den Schutz echter Daten.

Der echte Kursbestand enthält keine individuellen Bewertungen. Er verwendet bis zu
gültigen Einzelbewertungen die vorhandene Heuristik. Aggregierte Teilnehmerzahlen
werden nicht zu Präferenzen oder Trainingslabels umgedeutet. Der bisherige getrennte
Demo-Modus kann das bestehende trainierte ML-Modell weiterhin demonstrieren.

Teamprüfung bleibt offen. Mit **`OK Start Task 7`** folgt die gezielte Veröffentlichung
von Code, geprüfter Startdatenbank und Anleitung auf GitHub; die Originaldateien und
die private Arbeitsdatenbank werden nicht veröffentlicht.
