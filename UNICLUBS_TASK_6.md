<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-15, user-approved UniClubs Task 6; review pending -->
# UniClubs – Task 6: Streamlit-Anbindung

Status: abgeschlossen. Der Stand ist auf GitHub in `main` übernommen; eine öffentliche
Streamlit-Instanz wurde in diesem Schritt nicht als Deployment geprüft.

Der öffentliche GitHub-Feed ist mit dem bestehenden Streamlit-Programm verbunden. Die Events
bleiben in der eigenen Datei `data/uniclubs.sqlite`; es werden keine UniClubs-Katalogzeilen in
die bisherigen Tabellen `events`, `series` oder `clubs` kopiert.

## Was man in der App sieht

1. App starten und das vorhandene Onboarding abschließen.
2. In der Wochenansicht eine Woche mit UniClubs-Events auswählen. Beim echten Prüfkatalog liegen
   die ersten Termine am 16. November 2026, also gegebenenfalls **Next week** wählen.
3. Geeignete Events erscheinen in den normalen Aktivitätsvorschlägen. Zusätzlich listet
   **UniClubs this week** alle zugehörigen Angebote, einschließlich geschlossener Anmeldungen und
   nicht in freie Lücken einplanbarer Veranstaltungen.
4. **Details** öffnet die vertrauten Favoriten-, Planungs- und Exportfunktionen.
5. In der Sidebar stehen **Source updated**, **Last checked** und **Refresh UniClubs**.

Ein bestätigter, zeitlich bestimmter Event wird im Kalender eingetragen und bei freien Lücken
berücksichtigt. Stornierung öffnet das Zeitfenster wieder. Eine Terminänderung derselben
Quellen-ID verschiebt den bestehenden App-Plan, statt eine zweite Aktivität anzulegen.
Eine Planung in WeekWise ist keine Anmeldung beim Club; dafür dient der Original-Link.

## Datenfluss und Trennung

```text
GitHub-Ref auf den aktuellen main-Commit lesen
                 ↓
Öffentlichen JSON-Feed dieses Commits herunterladen und validieren
                 ↓
Nur den separaten öffentlichen UniClubs-Katalog aktualisieren
                 ↓
UTC-Zeitwerte in Zurich-Kalenderobjekte umwandeln
                 ↓
In Python mit den bisherigen Quellen und privaten Plänen zusammenführen
                 ↓
Streamlit-Woche, Vorschläge, Details und Kalenderexport
```

Die Quellenkennungen heißen im App-Modell `uniclubs:<Quellen-ID>`. Serien- und Clubfavoriten
haben ebenfalls eigene Namensräume. Sie können deshalb nicht mit den bisherigen Ganzzahl-IDs
kollidieren. Die vorhandene lokale API bleibt für die bisherigen Quellen zuständig; UniClubs
benutzt den öffentlichen Feed und seine eigene SQLite-Datei.

Persönliche UniClubs-Pläne, Favoriten und Bewertungen liegen in
`data/private/uniclubs_user.sqlite`. Ein zufälliger Eigentümer-Token verbindet sie mit dem
aktuellen lokalen Profil. In der bisherigen Datenbank wird dafür nur eine Profileinstellung
ergänzt. Eine Namensänderung erhält den Token; ein Profilreset kann keine alten Pläne unter
einer wiederverwendeten Benutzer-ID übernehmen. Der ausdrückliche App-Reset löscht die
persönlichen UniClubs-Daten des aktuellen Profils.

Die private Datei wird von Git ignoriert und gehört nicht zum öffentlichen Export. Der
GitHub-Abgleich benötigt sie nicht. Freundeinladungen und geteilte Pläne der bisherigen
Friends-Funktion sind für diese neue Quelle nicht implementiert; die UI verspricht sie nicht.

## Automatische Prüfung und Ausfälle

- GitHub Actions aktualisiert den öffentlichen Katalog weiterhin im Zeitplan aus Task 5.
- Ein nativer Streamlit-Fragmenttimer prüft auch eine untätige offene Sitzung alle 60 Sekunden.
- Die eigentlichen Netzabrufe sind gemeinsam fünf Minuten gecacht. Ein neuer Quellenstand
  löscht die Ereignis-/Modellcaches und löst eine vollständige Kalender-Neuberechnung aus.
- Der manuelle Refresh verwirft den Downloadcache und startet einen neuen Prüfversuch.
- Der GitHub-Ref wird anonym aufgelöst; danach wird die commit-spezifische Raw-Adresse gelesen.
  Das umgeht die zuvor beobachtete veraltete Branch-URL im CDN. Falls nötig gibt es einen
  konservativen normalen Raw-Rückfall, der niemals einen neueren gespeicherten Stand ersetzt.
- Ist das Netz nicht erreichbar, bleiben die letzten gültigen lokalen Events erhalten. Beim
  erstmaligen Start kann auch die mitgelieferte öffentliche JSON-Datei als Ausgangsbasis dienen.

Quellenzeit und Prüfzeit werden getrennt angezeigt. Ein erfolgreicher Download beweist nicht,
dass der Herausgeber gerade neue Quelldaten gesammelt hat. Nach mehr als 90 Minuten Quellenalter
zeigt die Sidebar einen Hinweis. Zeitplan, Cache und GitHub-Verzögerungen bedeuten weiterhin
keine Echtzeitsynchronisation.

Neue SQLite-Verbindungen werden pro Operation geschlossen. Es werden keine SQLite-Verbindungen
mit Standard-Threadbindung über Streamlit-Caches zwischen Sitzungen geteilt.

## Zeit- und Statusregeln

UTC bleibt unverändert im öffentlichen Katalog. Erst der Adapter verwendet die vorhandene
Kalender-Zeitkonvertierung nach Europe/Zurich, einschließlich Sommer-/Winterzeit.

- `sales_closed` heißt **Registration closed**, keine erfundene Absage.
- Eine ausdrückliche Absage durch den Veranstalter wird gesondert angezeigt. Ein solcher Plan
  blockiert keine freie Zeit mehr und bietet keinen normalen Bestätigungs-ICS-Export an.
- Fehlende Quellen bleiben sichtbar und werden nicht als sicher abgesagt interpretiert.
  Bereits bestätigte, zeitlich bekannte Pläne bleiben dabei berücksichtigt.
- Ein unbekanntes Ende bleibt in SQLite `NULL`. Nur zur Anzeige wird die vorhandene geschätzte
  Standarddauer verwendet, klar markiert. Das erlaubt weder eine neue Bestätigung noch Feedback.
- Mehrtägige bzw. mehrwöchige Programme werden nicht als erfundene wöchentliche Einzeltermine
  oder ungeprüft als durchgehende Kalenderblöcke bestätigt.
- Wird ein gespeicherter Plan nachträglich zeitlich unklar, werden für die betroffene Woche
  keine verlässlichen freien Lücken zugesagt. Die App fordert zur Prüfung dieses Plans auf.

Kalender-UIDs der neuen Quelle bleiben bei Verschiebungen und Stornierungen gleich. Alte
UIDs und vorhandene positive Event-IDs funktionieren weiterhin. Google-/Outlook-Links und ICS
werden angeboten. Ein externer Kalender wird erst durch die jeweilige Nutzeraktion ergänzt;
die App schreibt nicht selbstständig über eine Kalender-API in Google oder Outlook.

## Feedback und Modell

Anwesenheit und Bewertungen werden über die gleichen sichtbaren Bedienelemente erfasst,
aber privat bei der neuen Quelle gespeichert. Die Modellrevision berücksichtigt geänderte
Source-Ratings. Gültige echte Bewertungen erhalten das bisherige Trainingsgewicht fünf.

Die Quellenbeispiele werden in dieselben chronologischen Trainingsbatches eingereiht. Die
eigene Bewertung und gleichzeitige Peer-Beispiele verändern nicht ihre historischen Eingaben.
Zukünftige Rückmeldungen, No-Shows, abgesagte oder zeitlich unklare Events und unbekannte
Antwortzeitpunkte sind keine verfügbaren Trainingsbeispiele. Persönliche Rohdaten verlassen
die App dabei nicht.

Wie im bisherigen Modell speichert `answered_at` den Anwesenheitszeitpunkt, keine vollständige
Historie späterer Bewertungsänderungen. Das Training verwendet den aktuellen Bewertungsstand;
eine historische Rekonstruktion jeder früheren Sternebewertung ist damit nicht möglich.

## Prüfnachweise

- **357 Unit- und Projektregeltests bestanden.**
- Historischer vollständiger Demo-Akzeptanzlauf weiterhin bestanden.
- Neuer vollständiger UniClubs-Streamlit-Akzeptanzlauf bestanden: Feedanzeige, geschlossene
  Anmeldung, private Event-/Clubfavoriten, Bestätigen, Kalenderblock, ICS, neue Sitzung,
  Titel-/Terminänderung, Feed-Ausfall, Stornieren, Anwesenheit, Sterne und Veranstalterabsage.
- Der neue Lauf prüft ausdrücklich, dass keine Source-Events oder Source-Interaktionen in
  den bisherigen Katalog-/Interaktionstabellen gespeichert wurden.
- Ein echter anonymer Feedabruf wurde auf den vorhandenen Katalog angewendet: 49 Events,
  ursprüngliche IDs und Erstbeobachtungen erhalten, SQLite-Integrität und Fremdschlüssel gültig.
- Der echte Kalenderadapter lieferte 21 UniClubs-Angebote für die Woche ab 16. November 2026,
  mit den erwarteten Zurich-Zeiten und durchgehend quellenbezogenen Kennungen.

```bash
.venv/bin/python -m unittest discover -s tests -v
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache MPLBACKEND=Agg NO_PROXY=127.0.0.1,localhost .venv/bin/python -m views.check_demo
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache MPLBACKEND=Agg NO_PROXY=127.0.0.1,localhost .venv/bin/python -m views.check_uniclubs
.venv/bin/python -m streamlit run app.py
```

Die beiden Abnahmeläufe verwenden isolierte temporäre Dateien und feste Uhren. Der historische
Lauf prüft weiter sein unverändertes Offline-Demoszenario; der neue Lauf schaltet die Quelle ein
und ersetzt ausschließlich den externen Feed durch kontrollierte Daten. Die CI führt beide aus.

## GitHub-Nachweis

- [Pull Request #2](https://github.com/rehman9010/V1-WeekWise/pull/2) wurde am 15. November 2026
  nach erfolgreichen Prüfungen regulär nach `main` übernommen.
- Der [Pull-Request-Prüflauf](https://github.com/rehman9010/V1-WeekWise/actions/runs/43748191502)
  und der [Branch-Prüflauf](https://github.com/rehman9010/V1-WeekWise/actions/runs/43748164149)
  bestanden jeweils vollständig auf Linux, Windows und macOS: frische Installation,
  Paketkompatibilität, 357 Tests und beide Streamlit-Abnahmeläufe.
- Merge-Commit: `9f3cd6ae6aa799b0a6b060d302f4582bc58f5fca`.
- Es wurden keine SQLite-Dateien oder privaten Nutzerdaten hochgeladen. Der vorhandene
  öffentliche JSON-Snapshot wurde durch diesen Code-Commit nicht überschrieben.
- Der lokale Streamlit-HTTP-Healthcheck antwortet nach der Übernahme mit `ok`.

Die Cloud-Umgebung wurde nicht veröffentlicht oder neu eingerichtet. Für die öffentlich
laufende Streamlit-App ist hier keine verifizierte Deployment-Adresse hinterlegt; GitHub- und
AppTest-Erfolg werden deshalb nicht als Nachweis eines dort abgeschlossenen Deployments gewertet.

## Betriebsgrenzen und nächster Schritt

Es wurden keine zusätzlichen Anwendungspakete, ORM-Bibliotheken, Browser-Roboter oder Threads
eingeführt. Streamlit-Fragmenttimer, Quellen-Namensräume und der private Owner-Token werden im
Study Guide als erklärte Erweiterungen gekennzeichnet.

Auf Streamlit Community Cloud sind lokale Dateien kein dauerhaft garantierter Nutzerdatenspeicher
über neue Deployments hinweg. Der öffentliche Katalog kann aus GitHub wiederhergestellt werden;
private Planungen und Bewertungen sind davon getrennt und werden nicht veröffentlicht.
Die Umsetzung bleibt das bestehende lokale Profilmodell, keine neue Mehrbenutzeranmeldung.

Task 7 ist die gesonderte Freigabe für die abschließenden Neustart- und Fehlerprüfungen des
Gesamtablaufs. Er startet erst nach `Start Task 7`.
