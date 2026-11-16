<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-16; approved Task 7; reviewed by: pending -->
# Task 7 – Veröffentlichung des bereinigten Startbestands

Freigabe: **OK Start Task 7** am 16.11.2026. Dieser Bericht beschreibt den geprüften
Veröffentlichungsstand. Der jeweilige GitHub-Prüfstatus ist unter
[WeekWise checks](https://github.com/rehman9010/V1-WeekWise/actions/workflows/check.yml)
einsehbar; native Ergebnisse werden nicht vor dem tatsächlichen Lauf behauptet.

## Ergebnis und Datenabgrenzung

- Öffentliche SQLite-Vorlage: `data/public/unisport_starter.sqlite`, **581.632 Bytes**.
- **263 Kursmuster / 1.052 Termine**, Zeitraum 16.11.–13.12.2026.
- **967 Website-Nachweise / 85 Prognosen**; Quelle geprüft am 15.11.2026.
- 15 ungeprüfte Kursbezeichnungen sind neutral maskiert; sie betreffen 60 Prognosen.
- Keine Profile, privaten Kalendereinträge, Dateinamen, Originaldateien, Rohhistorie,
  Teilnehmerstatistiken, Bewertungen oder persönlichen UniClubs-Pläne enthalten.
- Alle Verfügbarkeiten bleiben `unknown`; Orte/Preise werden nicht erfunden.
- SHA-256: `c7880899488155214ff78b02630458fcbe6f93a77df3b89bb0e3d6b127d33708`.

Das öffentliche Manifest enthält Prüfsumme, Zeitraum, Bestandskennung und Zähler.
Eine unabhängige Nur-Lese-Prüfung bestätigt die erlaubten Felder, leere Privat-/Nutzertabellen,
SQLite-Integrität, Fremdschlüssel und null freie Seiten mit potenziellen Altwerten.
Die Vorlage wurde nochmals über den bereits geprüften Feldexport erzeugt. Sie ist
bytegleich mit der unabhängig geprüften Task-6-Vorschau. Die private Historie mit
9.585 Zeilen bleibt lokal und unverändert.

## Integration mit dem aktuellen GitHub-Stand

Seit dem ursprünglichen gemeinsamen Stand `ecf0e92` wurden auf `main` die eigenständigen
UniClubs-Funktionen ergänzt. Vor Veröffentlichung wurden diese Änderungen bis `0ea49f9`
eingelesen und mit den lokalen Unisport-Tasks 3–6 zusammengeführt. Feed, Aktualisierungs-
Workflow, Quellen-Namensräume, private Planung, Kalenderexport und source-aware ML bleiben
erhalten. Die öffentliche UniClubs-JSON-Datei wird nicht durch den Sportexport ersetzt.

Die gemeinsame Kalenderansicht liest beide Quellen. Sport-IDs bleiben Ganzzahlen,
UniClubs verwendet seinen bestehenden Namensraum. Beim Profil-Reset werden eigene
UniClubs-Pläne und deren Besitzerkennung entfernt; eine wiederverwendete numerische
Nutzer-ID übernimmt keine alten privaten Pläne.

## Start auf frischen und bestehenden Installationen

`starter_catalogue.bootstrap_starter` öffnet die Vorlage schreibgeschützt und prüft
Schema, zulässige Felder, Herkunft und Integrität. Kopiert wird ausschließlich in ein
leeres dateibasiertes Ziel. Bestehende Profile, Kurse, Archive oder Einstellungen
verhindern jede Ersetzung. Veränderte Tabellen mit versteckten Zusatzspalten und
unerwartete Views/Trigger werden zurückgewiesen. In-Memory-Testfixtures bleiben leer.

Der veröffentlichte Standard verwendet **`data/weekwise-reviewed.sqlite`**. Dadurch
bleibt eine eventuell vorhandene alte `data/weekwise.sqlite` erhalten. Der echte
Bestand erhält sein eigenes Onboarding. Weder Demo-Bestätigungen noch personenbezogene
Pläne werden auf zufällig gleiche Sport-IDs übertragen. `WEEKWISE_DB_FILE` kann einen
ausdrücklichen anderen Arbeitsdateipfad wählen.

Die App und ein API-first-Start verwenden denselben Bootstrap. Bei einer beschädigten
vorhandenen Vorlage erfolgt kein stiller Ersatz durch erfundene Kurse. Ein expliziter
Demo-Modus (`WEEKWISE_DATA_MODE=demo`) bleibt möglich; ein Demo-Reset behält diese Wahl.
Der veröffentlichte Seed wird nie als normale beschreibbare Arbeitsdatei verwendet.

## Dokumentation und Kursbezug

Die [deutsche Betriebsanleitung](PUBLIC_STARTER_GUIDE.md) erklärt Installation, Dateien,
Herkunft, Reload/Reset, die echte lokale HTTP-API und den getrennten ML-Demo-Modus.
README, Study Guide und KI-Protokoll wurden um den finalen Datenweg ergänzt.

Die Umsetzung bleibt bei direktem SQLite, `?`-Parametern, Transaktionen, einfachen
Python-Funktionen und der vorhandenen Streamlit-/Flask-/requests-Struktur. Keine neue
Bibliothek, Tabelle, App-Seite, Hintergrundaufgabe oder Laufzeit-KI. `sqlite3.backup`
ist der ausdrücklich dokumentierte technische Kopierhelfer für die vollständige
öffentliche Vorlage; er wird nicht als zusätzliche Vorlesungsübung ausgegeben.
Die normalen Datenzugriffe bleiben das in A07 verwendete SQL. Prüfsummen verwenden
den bereits dokumentierten `hashlib`-Helfer.

## Verifikation

Lokal unter Linux/Python 3.11:

- **492 Unit-/Regeltests** einschließlich 16 neuer Bootstrap-/Starttests.
- Bestehender kompletter M8-Demo-/API-Ablauf bestanden.
- Eigenständiger UniClubs-Kalender-/Planungsablauf bestanden.
- UniClubs-Cold-Start und Wiederherstellung über neue Prozesse bestanden.
- Neuer gemeinsamer App-Ablauf mit dem tatsächlichen Sport-Seed und einem unabhängigen
  synthetischen UniClubs-Termin bestanden: 1.053 eindeutige gemischte IDs, Herkunft,
  private UniClubs-Bestätigung/Favorit, neue Sitzung, Reload, Profil-Reset und neuer
  Besitzer ohne geerbte Pläne. Alle 1.052 Sporttermine und Nachweise bleiben erhalten.
- Manifest-Prüfsumme/Größe/Zeitraum/Zähler und unveränderte Seed-Datei im App-Test geprüft.
- Frischer API-Dateistart vor Streamlit im Test geprüft; Quelle und API/DB-Vertrag erhalten.
- Alle installierten Abhängigkeiten kompatibel; Pins unverändert.

Der Wochen-Cache ist zusätzlich nach Bestandskennung getrennt. Die Verbindung liest
den konfigurierten Arbeitsdateipfad ausdrücklich; die geänderte Cache-Funktion wird
bei der automatischen Code-Aktualisierung neu aufgebaut. So kann eine alte gecachte
Demo-Woche nicht als Ergebnis des neuen Kursbestands weiterverwendet werden.

Die GitHub-Prüfmatrix führt diese Tests und alle vier App-Abläufe auf Linux, Windows und
macOS aus. Für den zusätzlichen gemeinsamen Ablauf ist die Job-Grenze auf 20 Minuten
gesetzt; individuelle App-/API-Zeitgrenzen bleiben bestehen. Nach dem Upload werden
die tatsächlichen Ergebnisse geprüft und im Abschluss verlinkt.

Der reale Kursbestand erfindet weiterhin keine Einzelratings. Die bestehende Heuristik
und die vorhandene echte UniClubs-Bewertungsanbindung bleiben nutzbar; das trainierte
Modell ist im getrennten Demo-Modus demonstrierbar. Teamprüfung und eigenes Video mit
nicht generierter Stimme bleiben Aufgaben des Projektteams.
