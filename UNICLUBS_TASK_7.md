<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15, user-approved UniClubs Task 7; review pending -->
# UniClubs – Task 7: Gesamtabnahme und Betriebsanleitung

Status: Die abschließenden lokalen und GitHub-Prüfungen werden hier mit ihrem tatsächlichen
Ergebnis dokumentiert. Eine öffentliche Streamlit-Adresse ist bislang nicht hinterlegt.

## Geprüfter Datenfluss

```text
Öffentliche UniClubs-Eventübersicht + Sitemap
                    ↓ GitHub Actions, alle 30 Minuten geplant
Eigene SQLite-Datenbank auf dem Runner → öffentlicher JSON-Export auf GitHub
                    ↓ anonymer Download, fünf Minuten Cache
Separate SQLite-Datei der Streamlit-App: data/uniclubs.sqlite
                    ↓ Kalenderadapter und Zusammenführung in Python
UniClubs-Wochenliste, Vorschläge, gespeicherte Pläne und Kalenderexport
```

Für diesen Weg braucht ihr weder ein UniClubs-Konto noch einen UniClubs-API-Schlüssel oder
Zugang zur fremden Datenbank. Der Import liest öffentlich ausgelieferte Daten. Die Umsetzung
verwendet grundlegende Python-Funktionen, JSON, Schleifen, SQL mit `sqlite3` und die schon
vorhandenen Streamlit-/HTTP-Pakete. Die Datenbank aus der früheren App bleibt getrennt.

## Echter Quellen- und GitHub-Abgleich

- [GitHub-Lauf vom 15.11.2026](https://github.com/rehman9010/V1-WeekWise/actions/runs/43748856708):
  162 UniClubs-Tests, Wiederherstellung, echte HTTPS-Quellenprüfung, Export und Bot-Push erfolgreich.
- Ergebnis: **0 neue, 0 geänderte, 49 unveränderte Events von 23 Clubs**. Der echte Quellenstand
  enthielt während der Prüfung keine neu angelegte Veranstaltung. Neue Events und Änderungen
  wurden deshalb zusätzlich mit ausdrücklich kontrollierten Testdaten geprüft.
- [Bot-Commit](https://github.com/rehman9010/V1-WeekWise/commit/15757887fa02fdb19ea2e861cb140097e5b98c6c):
  ausschließlich `data/public/uniclubs_events.json` geändert, keine SQLite-Datei oder Nutzerdaten.
- Der neue Feed wurde anonym über die öffentliche Commit-Adresse geladen und auf eine separate
  temporäre SQLite-Datenbank angewendet. Der Quellenzeitpunkt stieg auf
  `2026-11-15T18:49:28.550231Z`; alle 49 IDs und Erstbeobachtungen blieben erhalten.
- Nach Schließen und erneutem Öffnen: SQLite-Integrität `ok`, keine Fremdschlüsselfehler.
  Der Kalenderadapter lieferte 21 Angebote für die Woche ab 16.11.2026 mit Zurich-Zeiten.

Der Workflow ist auf GitHub **active**. Dieser Ausführungsnachweis stammt von einem manuellen
Start desselben Workflows; ein tatsächlicher Start durch das `schedule`-Ereignis wird damit
nicht behauptet. GitHub kann Cron-Läufe verzögern.

## Offline-Erststart und echte Prozessneustarts

Neue wiederholbare Abnahme:

```bash
python -m views.check_uniclubs_recovery
```

Der Runner startet vier verschiedene Python-Interpreter mit denselben ausschließlich temporären
Dateien. Das ist stärker als ein weiterer Browser-Tab im selben Prozess: Es sind keine alten
Python-Objekte oder Streamlit-Caches verfügbar.

| Phase | Erwartung und Prüfergebnis |
| --- | --- |
| Leere Dateien, Netz ausgefallen | Bundled JSON erzeugt den separaten Katalog; Onboarding, Favoriten und Bestätigung über echte App-Handler erfolgreich. |
| Neuer Prozess, Netz weiterhin ausgefallen | Profil, Owner-Token, Plan, Favoriten und Kalenderanzeige aus gespeicherten Dateien wiederhergestellt. |
| Neuer Prozess, fehlerhafter Feed | Öffentlicher Katalog und vollständige private Interaktion bleiben unverändert; Fehlhinweis sichtbar. |
| Neuer Prozess, gültiger älterer Feed | Neuere gespeicherte Daten bleiben erhalten; ältere Titel-/Zeitwerte werden nicht übernommen. |

In jeder Phase werden SQLite- und Fremdschlüsselintegrität, stabile ICS-UID, öffentliche
Quellenidentität und das Fehlen von Quellenkopien in den alten Katalogtabellen geprüft.
Der lokale Abnahmelauf bestand in 17,533 Sekunden. Der neue Befehl ist außerdem im CI-Workflow
für Linux, Windows und macOS ergänzt.

## Browser und Regressionen

- 357 Unit-/Projektregeltests erneut erfolgreich.
- Bisheriger vollständiger Demo-/API-Abnahmelauf erneut erfolgreich.
- UniClubs-App-Abnahmelauf erneut erfolgreich: Planen, Favoriten, Exporte, Titel-/Zeitänderung,
  Ausfall, Stornierung, Feedback und Veranstalterabsage.
- Task-6-Mergeprüfung auf `main` ebenfalls auf Linux, Windows und macOS erfolgreich:
  [Prüflauf](https://github.com/rehman9010/V1-WeekWise/actions/runs/43748620306).
- Zusätzliche Browser-Timerprüfung läuft mit kontrolliertem Feed und unveränderten
  Produktionseinstellungen; das konkrete Ergebnis wird nach Abschluss eingetragen.

Prüfbefehle aus dem Projektverzeichnis mit aktiviertem Python-3.11-Environment:

```bash
python -m unittest discover -s tests -v
python -m views.check_demo
python -m views.check_uniclubs
python -m views.check_uniclubs_recovery
python -m streamlit run app.py
```

## Fünf-Minuten-Demo für euer Team

1. App öffnen, Onboarding abschließen und eine Woche mit Events wählen, im Prüfkatalog die
   Woche ab **16. November 2026**. Der Kalender zeigt immer die ausgewählte Woche.
2. **UniClubs this week** öffnen. Auch ausgebuchte, geschlossene oder zeitlich unklare Angebote
   bleiben dort auffindbar. Der Quellenlink führt zur eigentlichen Clubanmeldung.
3. Bei einem verfügbaren Event **Details → Confirm** wählen. Der Plan erscheint im Kalender;
   Favoriten und ICS-Download stehen im selben Dialog. Die Bestätigung ist keine Clubanmeldung.
4. In der Sidebar **Source updated** mit **Last checked** vergleichen. Ersteres ist das Alter
   der Quelldaten, Letzteres der Prüfversuch der App. **Refresh UniClubs** versucht sofort neu.
5. Auf GitHub unter **Actions → UniClubs public catalog → Run workflow** einen Abgleich starten.
   Nach Erfolg kann die App den neuen Stand laden, ohne neue Event-IDs manuell einzutragen.

Normalerweise werden neue Quelleninhalte innerhalb des nächsten 30-Minuten-Abgleichs plus
App-Cache verfügbar. Bei offener App prüft ein Fragment alle 60 Sekunden den gemeinsamen Cache.
Das ist regelmäßige Aktualisierung, keine garantierte Echtzeitzustellung.

## Dateien, Lernstoff und verbleibende Grenzen

Neu sind `views/check_uniclubs_recovery.py`, `views/uniclubs_recovery_scenario.py` und dieser
Bericht. Geändert sind die Prüfworkflowdatei, README, Study Guide und AI-Nutzungsdokumentation.
Keine zusätzlichen Produktpakete, Änderungen an `SPEC.md`/`CLAUDE.md` oder Cloud-Umgebung.
Details zu Funktionen, Vererbung und SQL stehen im Study Guide. Prozesssteuerung und Browser-
automatisierung sind erklärte Testwerkzeuge, kein zusätzlich behaupteter Unterrichtsstoff.

- Die öffentliche Streamlit-Instanz kann erst mit ihrer tatsächlichen URL separat abgenommen
  werden. Weder README noch öffentliche GitHub-Metadaten enthalten diese Adresse.
- Lokale persönliche SQLite-Dateien überleben einen normalen Neustart mit erhaltenen Dateien,
  sind aber auf Community Cloud über ersetzte Deployments hinweg nicht garantiert dauerhaft.
  Der öffentliche Feed stellt nur den Katalog wieder her, keine privaten Planungen.
- Persönliche UniClubs-Pläne liegen in `data/private/uniclubs_user.sqlite`, bleiben im lokalen
  Profilmodell und werden nicht durch die bisherige Friends-Funktion geteilt.
- Quelländerungen am HTML-/Datenformat können spätere Anpassungen am Importer erfordern.
  Fehler werden sichtbar; der letzte gültige Katalog bleibt verfügbar.

Die Schritte 1–7 sind damit der vereinbarte Umfang. Eine weitere nummerierte Aufgabe wird
nicht automatisch begonnen.
