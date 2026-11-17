<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-17; requested spring-semester extension; reviewed by: pending -->
# Frühlingssemester 2027 – Erweiterung des Sportbestands

Status: Umsetzung, Datenaufbereitung und lokale Abnahme abgeschlossen.
Freigabe: Nutzerwunsch „Bis Ende des
Frühlingssemesters 2027“ und anschließendes „weiter“.

## Ergebnis und Abnahmekriterien

1. Planungshorizont **16.11.2026–31.07.2027** mit **9.731 Terminen aus 263 Kursmustern**.
2. **1.551 Website-Nachweise / 8.180 Prognosen**. 15 ungeprüfte Kursbezeichnungen sind
   weiterhin neutral maskiert; sie betreffen 555 Prognosen. Alle Verfügbarkeiten bleiben
   `unknown`, Orte und Preise ungeprüft.
3. **8.679 zusätzliche Termine** ergänzen den ursprünglichen Bestand. **584** davon
   erhielten durch exakten Abgleich einen Website-Nachweis. Alle ursprünglichen
   263 Kursidentitäten und 1.052 Ereignisse einschließlich Herkunft bleiben erhalten.
4. Bestehende geprüfte Arbeitsbestände erhalten die Erweiterung durch eine atomare
   Ergänzung. Profile, Favoriten, Planungen, manuelle Felder, vorhandene Nachweise und
   private UniClubs-Besitzerkennungen werden erhalten. Identitätskonflikte verhindern
   die gesamte Ergänzung. Fremde Bestände und Demo-Dateien werden nicht ersetzt.
5. Kein automatischer Sportabruf, keine neue Bibliothek, Tabelle, ORM oder Laufzeit-KI.
   Der eigenständige UniClubs-Feed behält seine vorhandene Aktualisierung.

Öffentliche Datei: `data/public/unisport_starter.sqlite`, **1.941.504 Bytes**.
SHA-256: `f8868c123f6ef4439a46111e10a013f7de9aa852970f55f2e51843c517da4d42`.
Das [Manifest](data/public/unisport_manifest.json) dokumentiert Zeitraum, Zähler,
Bestandskennung, Vorgänger und die Prognoseregel. Historische Rohdateien, Profile,
Teilnehmerstatistiken und private Kalenderdaten sind nicht Teil der Veröffentlichung.

## Was Semesterende und Prognose bedeuten

Der [offizielle HSG-Kalender](https://www.unisg.ch/de/studium/studieren-an-der-hsg/semesterdaten/)
nennt **22.02.–28.05.2027** als Vorlesungszeit, **05.–16.04.2027** als Semester-Break
und **21.06.–23.07.2027** als zentrale Prüfungen. Das volle Frühlingssemester läuft
gemäß Artikel 145 der [akademischen Ausführungsbestimmungen](https://erlasse.unisg.ch/lexoverview-home/lex-II_A_09/lex-II_A_09.pdf?change-id=357&effective-from=20250201)
vom 1. Februar bis **31. Juli**. Der Nutzerwunsch wird auf dieses volle Semester bezogen.
Studienfreie Tage beweisen weder Ausfall noch Durchführung eines Sportkurses.

Die einfache Regel bleibt unverändert: exakte Kombination aus Sport, Variante,
Wochentag und Startzeit; mindestens drei unterschiedliche Wochenbeobachtungen innerhalb
der letzten 28 beobachteten Tage und ein Vorkommen in deren letzten sieben Tagen.
Anker ist der **06.11.2026**, nicht das aktuelle Rechnerdatum. Es entsteht kein
trainiertes Saisonmodell. Obwohl die Historie auch frühere Monate enthält, werden
Semesterwechsel, Ferien und Sommerprogramme mit dieser Regel nicht vorhergesagt.
Die längere Reichweite vergrößert daher die Unsicherheit und bleibt sichtbar vorläufig.

Am **17.11.2026** wurden 33 öffentliche Wochenabfragen für 14.12.2026–31.07.2027
gespeichert. Die Antworten enthalten insgesamt 971 Quellzeilen, darunter auch leere
Wochenantworten. Nur vollständige nichtleere Antworten wurden mit den bereits
explizit freigegebenen 248 Musterzuordnungen abgeglichen. Produktidentität, öffentlicher
Titel, Terminbeschreibung sowie Zürcher Datum und Startzeit müssen exakt passen.
584 neue Termine erfüllten diese Kriterien. Leere Antworten oder fehlende Termine
sind kein Absagenachweis. Der späteste exakt belegte Termin liegt am **20.02.2027**.
Die ursprünglichen Website-Nachweise vom 15.11.2026 bleiben unverändert.
Rohantworten und Zuordnungsprüfung bleiben privat; die öffentliche Vorlage enthält
ausschließlich die schon freigegebenen Felder.

## Umsetzung und Schutz bestehender Daten

`training_forecast.py` akzeptiert optional `--until YYYY-MM-DD` als einschließliches
Enddatum. Die interne Grenze ist der Folgetag; die letzte unvollständige Woche enthält
nur Termine innerhalb des Horizonts. Ohne Option bleiben vier vollständige Wochen
der Standard. Ein Lauf ist auf 366 Tage, der angesammelte Sportbestand auf 20.000
Termine begrenzt. Der bestehende Grenzwert von 5.000 Zeilen für andere Importe wird
nicht pauschal erhöht. Vor den Schreibzugriffen werden Grenze und Kapazität geprüft.

Die vorhandene private Datenbank wurde ab **14.12.2026** erweitert. Ein Neubau ab
November würde bei längerer Reichweite wegen der Einfügereihenfolge alte IDs
verschieben. Stattdessen bleiben frühere Termine und Nachweise vollständig erhalten;
die öffentliche Datei wird über den vorhandenen Feldexport in einer neuen Datei
aufgebaut und erst nach der Prüfung ausgetauscht. Die private Historie bleibt privat.

`starter_catalogue.bootstrap_starter` kopiert weiterhin nur in ein leeres Ziel.
Für den bekannten ursprünglichen veröffentlichten Bestand prüft der zusätzliche
Erweiterungspfad die vollständige neue Inhaltskennung und den Hash des ursprünglichen
Kurs-/Termin-/Nachweis-Präfixes. Die Arbeitskopie muss dieselben Kursidentitäten und
alten Terminidentitäten samt vorhandenem Herkunftseintrag besitzen. Lokale
Feldkorrekturen und persönliche Daten werden dabei nicht durch die Vorlage ersetzt.
Neue Primärschlüssel- oder eindeutige Terminplatz-Konflikte führen zum Abbruch.

Ein `BEGIN IMMEDIATE` serialisiert gleichzeitige Starts von App und API. Die Kennung
wird nach Erhalt der Schreibsperre erneut geprüft und erst nach erfolgreichen
Ergänzungen aktualisiert. Fehler rollen Termine, Herkunft und Kennungswechsel
gemeinsam zurück. Wiederholte Starts sind wirkungslos. Die bestehende Cache- und
API-Kennung `X-WeekWise-Catalogue` folgt der aktualisierten Bestandskennung.
Die Rückwärtskompatibilität gilt gezielt für diesen bekannten Vorgänger; ein beliebiger
anderer Katalog wird nicht als kompatibel angenommen.

## Geänderte Dateien

- `training_forecast.py`, `config.py`, `training_catalogue.py`: einschließliches Enddatum
  und getrennte Grenzen für den erweiterten Bestand.
- `starter_catalogue.py` und zugehörige Tests: sichere Ergänzung belegter Arbeitsdateien.
- Forecast- und Starter-Tests sowie `views/check_starter.py`, `views/starter_scenario.py`:
  Grenzen, Identitätsschutz, Rollback und der tatsächliche längere Kalenderbestand.
- `data/public/unisport_starter.sqlite`, `data/public/unisport_manifest.json`: neuer
  bereinigter öffentlicher Datenstand.
- README, `PUBLIC_STARTER_GUIDE.md`, `STUDY_GUIDE.md`, `AI_USAGE.md`, dieser Bericht:
  Übergabe, Methode, Einschränkungen und Kursbezug.

## In ungefähr fünf Minuten prüfen

```bash
python -m unittest discover -s tests -v
python -m views.check_starter
python -m streamlit run app.py
```

In der App ein Profil anlegen bzw. weiterverwenden, Ende Juli 2027 wählen und die
vorläufigen Sporttermine öffnen. Herkunft und unbekannte Verfügbarkeit bleiben sichtbar.
Die API kann mit `python api_server.py` gestartet werden, etwa mit
`/events?from=2027-07-26&to=2027-08-01`. Sie erlaubt höchstens 120 Tage pro Anfrage;
der Gesamtbestand wird deshalb wochen- oder monatsweise abgefragt.

Für die Datenübergabe die öffentliche SQLite-Datei und ihr Manifest senden oder auf
[den GitHub-Ordner](https://github.com/rehman9010/V1-WeekWise/tree/main/data/public)
verlinken. Eine SQLite-Datei ist eine Datenbank; die vorhandene Flask-Anwendung
stellt bei Bedarf die HTTP-/JSON-API darüber bereit. Es ist kein externer
Datenbankdienst nötig. Einzelheiten stehen in der [deutschen Anleitung](PUBLIC_STARTER_GUIDE.md).

## Verifikation vor Veröffentlichung

- **511 Unit- und Regeltests bestanden**, darunter Grenzen des längeren Horizonts,
  Identitätskonflikte, Wiederholung und vollständiger Rollback der Ergänzung.
- **Vier App-Szenarien bestanden**: Demo mit laufender lokaler API und SQL-Fallback,
  UniClubs-Kalender, Wiederherstellung in getrennten Prozessen und veröffentlichter
  Sportbestand einschließlich der letzten Juliwoche in Streamlit und der Flask-API.
- Die tatsächliche alte Arbeitskopie wurde von 1.052 auf 9.731 Termine ergänzt;
  ein zweiter Start änderte nichts.
- Unabhängiger Datenabgleich: sämtliche alten 263 Kurszeilen, 1.052 Ereignisse und
  1.052 Nachweise sowie alle 9.585 Archivzeilen und die Importmetadaten unverändert.
  Integrität, Fremdschlüssel, freie Seiten, zusätzliche Schemaobjekte, private Tabellen
  und Manifest-Prüfsumme geprüft. Alle installierten Pakete sind kompatibel.
- Die Veröffentlichung führt zusätzlich die bestehende GitHub-Actions-Matrix auf
  Linux, Windows und macOS aus. Deren Ergebnis steht am veröffentlichten Commit unter
  [GitHub Actions](https://github.com/rehman9010/V1-WeekWise/actions).

## Entscheidungen und Kursbezug

- Ende 31.07.2027 statt bloßem Vorlesungsende, entsprechend dem gewählten vollen Semester.
- Historische Regel bewusst unverändert; größere Reichweite bleibt klar als Prognose markiert.
- Gezielte Ergänzung für die bekannte Vorgängerversion statt Austausch privater Arbeitsdateien.
- `training_forecast` → Datumsarithmetik/Schleifen → A06, L03.
- `starter_catalogue` → Primär-/Fremdschlüssel, parametrisierte Inserts und Transaktionen → A07.
- Inhaltskennungen und SQLite-Sperre → dokumentierte technische Hilfen; keine als
  Vorlesungsstoff ausgegebenen neuen Methoden.

Abweichungen von früherem Umfang: Der ausdrücklich gewünschte längere Horizont ersetzt
die ursprüngliche Beschränkung auf vier Zukunftswochen; der Standard ohne `--until`
bleibt erhalten. SPEC.md und CLAUDE.md bleiben unverändert. Offene Produktfragen: keine.
Teamprüfung der KI-generierten Änderungen bleibt wie bisher ausstehend.
