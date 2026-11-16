<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15; approved Task 4; review pending -->
# Task 4 – vorläufige Wochen aus der Trainingshistorie

**Status: lokal abgeschlossen.** Die Berechnung wurde auf dem privaten Arbeitsbranch
`data/unisport-history` umgesetzt. Es gab keinen Push, keinen Merge, keinen Website-
Abgleich und keine Änderung der veröffentlichten Streamlit-App. Task 5 wurde nicht gestartet.

## Regel und Ergebnis

Ein Muster besteht aus exakt gleichem Sport, gleicher Variante, gleichem Wochentag
und gleicher Startzeit. Es muss einen Sportnamen haben, an mindestens drei verschiedenen
Terminen im letzten beobachteten Vier-Wochen-Fenster vorkommen und auch in dessen
jüngster Woche noch vorhanden sein. Mehrere identische Zeilen oder erneut importierte
Exporte desselben Trainings erhöhen die Evidenz nicht.

| Prüfung | Ergebnis |
| --- | --- |
| Unveränderte historische Trainings | 9.585 |
| Letztes Quelldatum | 06.11.2026 |
| Beobachtungsfenster, einschließlich Grenzen | 10.10.–06.11.2026 |
| Jüngste beobachtete Woche | 31.10.–06.11.2026 |
| Qualifizierte Wochenmuster | 263 |
| Davon mit drei unterschiedlichen Wochen | 84 |
| Davon mit vier unterschiedlichen Wochen | 179 |
| Ausgewähltes Planungsfenster | 16.11.–13.12.2026, vier vollständige Wochen |
| Neue vorläufige Termine | 1.052 |
| Wiederholter Lauf desselben Horizonts | null zusätzliche Termine, stabile IDs |

Die 263 Signaturen sind historische Wochenmuster, keine nachgewiesenen offiziellen
Unisport-Kurs-IDs. Fehlende Orts-/Identitätsinformationen können durch die Historie
nicht rekonstruiert werden. Die letzte tatsächlich erzeugte Durchführung liegt am
Samstag, 12.12.; für den letzten Sonntag qualifiziert sich kein Muster. Das ausgewählte
Vier-Wochen-Fenster schließt diesen Sonntag trotzdem ein.

## Kennzeichnung und Bedeutung

Jeder neue Termin hat:

- `event_provenance.kind = projected`, mit Quelldatum und drei/vier Evidenzwochen;
- `events.status = unknown`: keine nachgewiesene Buchbarkeit;
- `end_is_estimated = 1`: die vorhandene 90-Minuten-Appschätzung ist ausdrücklich vorläufig;
- Ort TBA und unbekannten Preis; keine erfundenen Orte, Endzeiten oder freien Plätze;
- keine Website-Prüfquelle und keinen angeblichen Website-Prüfzeitpunkt.

Normale Bestätigungen und Einladungen sind bei unbekannter Buchbarkeit gesperrt. Es
wurden keine persönlichen Vormerkungen, Einladungen, Anwesenheitsdaten oder Ratings
angelegt. Eine spätere bewusste Darstellung vorläufiger Ideen gehört zur UI-Integration.
Die Wiederholungsregel ist kein neues ML-Modell; das vorhandene Empfehlungsmodell
bleibt davon getrennt.

## Datenhaltung und wiederholte Läufe

Die private Datei `data/private/training_history.sqlite` enthält jetzt auch die
263 lokalen Muster-Kursdatensätze in `series`, 1.052 Einträge in `events` und die
zugehörigen 1.052 Herkunftsdatensätze. Eine weitere dauerhafte Mustertabelle ist
nicht nötig. Die lokale Kennung `history-pattern:<kleinste historische Datensatz-ID>`
bleibt stabil und enthält keine persönlichen Freitexte.

Bereits vorhandene Termine behalten ihre Werte und IDs. Manuell korrigierte Dauern,
Website-Nachweise und bestehende Termine ohne Herkunftsmetadaten werden nicht
überschrieben. Ein späterer überlappender Horizont ergänzt nur fehlende Termine.
Es gibt keine pauschale Quellenbereinigung und kein automatisches Löschen alter
Prognosen. Die gespeicherte Beobachtungsbasis bleibt bei bestehenden Prognosen erhalten.
Die normale Kalenderbereinigung ist davon getrennt; sie verändert keine Archivzeilen.

Rohdaten und ungeprüfte Kursvarianten, einschließlich möglicher Personennamen, bleiben
privat. Task 5 prüft Angebote/Freitexte vor einem späteren öffentlichen Export.

## SQLite-Erweiterung

`events.status` erlaubt jetzt zusätzlich `unknown`. Eine eng begrenzte Migration der
bekannten bisherigen Tabelle erhält Termin-IDs, Nutzerbeziehungen, Einladungen und
bestehende Indizes, Trigger und Views. Sie prüft Fremdschlüssel und Integrität vor
Commit, setzt PRAGMA-Einstellungen zurück und nimmt bei Fehlern alle DDL-Schritte
zurück. Unbekannte Tabellenformen oder eine laufende Fremdtransaktion werden abgelehnt.

Neu ist `event_provenance` mit einer eindeutigen Terminreferenz. Löschen eines Termins
entfernt nur dessen Herkunftsmetadaten. Der Demo-Reset behält weiterhin die privaten
Archivtabellen. Bestehende Termine ohne Metadaten gelten nicht automatisch als geprüft.
Der App-/API-Feldname `status` bleibt erhalten; die vollständige Herkunftsanzeige
folgt erst in Task 6.

## Bedienung

Im Projektverzeichnis mit aktiviertem Python 3.11:

```bash
python training_forecast.py
```

Standardziel ist das private Task-3-Archiv, Start ist der nächste Montag strikt nach
heute. Ein bestimmter noch zukünftiger Montag lässt sich ausdrücklich wählen:

```bash
python training_forecast.py --start-date YYYY-MM-DD
```

Am 15.11.2026 wurde mit `--start-date 2026-11-16` geprüft. Jeder Lauf erzeugt höchstens
vier Wochen und beachtet das unveränderte aktive Quellenlimit. Die ursprünglichen
historischen Daten werden nicht auf zukünftige Tage verschoben oder überschrieben.

## Tatsächlich geprüfte Ergebnisse

- **227 lokale Unit-/Regeltests bestanden**, einschließlich 19 Prognose- und zehn
  Migrations-/Herkunftstests sowie aller bisherigen Tests.
- Vollständiger isolierter Demo-/API-Ablauf bestanden: echte lokale HTTP-Interaktionen
  und Fallback funktionieren weiterhin.
- Alle 1.052 Slots unabhängig über SQL-Gruppierung der Rohhistorie gegengeprüft:
  Muster, Wochentag, Uhrzeit, geschätztes Ende und Evidenz stimmen überein.
- Sämtliche historischen Zeilen und Dateiherkunfts-Metadaten vollständig auf Ebene der
  gelesenen SQL-Werte mit dem Vorher-Snapshot verglichen: unverändert.
- SQLite-Integrität `ok`, keine Fremdschlüsselfehler.
- Separater Streamlit-Start: `/` liefert 200; `/_stcore/health` liefert 200 / `ok`.
  Der eigene Testserver wurde anschließend beendet.
- Vor der Änderung wurde ein privater Datenbank-Snapshot zur Rückprüfung gesichert.
- Regressionen prüfen unveränderte nichtfortlaufende IDs und Referenzen, den Abbruch
  nach bereits ausgeführtem DROP, eigene Indizes/Trigger/Views, Wiederholungsnachweise,
  Transaktionsabbruch, Terminschutz und abgelehnte normale Bestätigungen/Einladungen.
- Alle Prüfungen dieser Stufe sind lokal unter Linux/Python 3.11 durchgeführt worden.
  Für den lokalen Arbeitsbranch wurde kein GitHub-Actions-Lauf gestartet.

## Grenzen und nächste Freigabe

Es wurden keine Ferien, Ausfälle, tatsächlichen Kursdauern oder zukünftigen freien
Plätze aus der Vergangenheit erfunden. Website-Abgleich ist Task 5, Kalender-/API-
Integration Task 6, Veröffentlichung Task 7. Nur ein neues ausdrückliches Startsignal
beginnt den nächsten Task.
