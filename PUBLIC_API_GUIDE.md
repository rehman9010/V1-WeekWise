<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-21; public API handoff; reviewed by: pending -->
# Zwei öffentliche Datenzugänge für WeekWise

Die öffentliche Flask-Anwendung liest UniSPORT und UniCLUBS aus getrennten
SQLite-Datenbanken und gibt JSON zurück. Sie bietet lesende Datenabfragen für
Teammates; persönliche Profile, Kalender und Planungen bleiben in der Streamlit-App.
Die bisherige lokale App-API bleibt separat verfügbar.

## Einmalig auf Render starten

Die Konfiguration ist vorbereitet. Es ist noch kein öffentlicher API-Server angelegt
und es gibt noch keine bestätigte öffentliche Basisadresse.

Direkt zur Einrichtung:
[Render mit diesem Repository öffnen](https://render.com/deploy?repo=https://github.com/rehman9010/V1-WeekWise).

1. Unter <https://dashboard.render.com/> einen Account anlegen und anmelden.
2. GitHub verbinden und Zugriff auf `rehman9010/V1-WeekWise` geben.
3. **New → Blueprint** wählen, das Repository und den Branch `main` auswählen.
4. Render liest `render.yaml`. Prüfen: ein Python-Webservice namens
   `weekwise-public-api`, Tarif **Free**, keine zusätzliche Datenbank und kein Datenträger.
5. Bereitstellung starten. Nach dem erfolgreichen Start die im Dashboard angezeigte
   öffentliche Service-URL kopieren und `/health` öffnen.

Die exakte URL wird von Render vergeben. In den folgenden Beispielen ist
`DEINE_RENDER_URL` ausdrücklich ein Platzhalter; er muss durch diese URL ersetzt werden.
Ein möglicherweise anders belegter Servicename ist kein Beleg für eine vorhandene API.

Falls Render stattdessen manuell als **New → Web Service** eingerichtet wird:

| Einstellung | Wert |
| --- | --- |
| Branch | `main` |
| Runtime | Python 3 |
| Python-Version | `3.11.18` |
| Build Command | `pip install -r requirements-api.txt` |
| Start Command | `gunicorn 'api_server:create_public_app()' --bind 0.0.0.0:$PORT --workers 1 --threads 1 --timeout 60` |
| Health Check Path | `/health` |
| Instance Type | Free |

Der kostenlose Dienst kann nach Inaktivität einschlafen. Der nächste Aufruf kann
deshalb ungefähr eine Minute für den Start benötigen. Seine lokalen Dateien sind
nicht dauerhaft; die öffentliche Datenbasis liegt weiter auf GitHub. Die API stellt
ihren UniClubs-Lesebestand aus den öffentlichen Daten wieder her. Private Daten dürfen
nicht in diesen öffentlichen Dienst kopiert werden.

## Die beiden API-Adressen weitergeben

Beispiel für eine Woche, jeweils mit einschließlich `from` und ausschließlich `to`:

```text
DEINE_RENDER_URL/unisport/events?from=2027-02-01&to=2027-02-08
DEINE_RENDER_URL/uniclubs/events?from=2027-02-01&to=2027-02-08
```

Die Datumsangaben gelten für **Europe/Zurich**. Ein Zeitraum muss 1 bis 120 Tage
umfassen. Für ein ganzes Semester mehrere Wochen- oder Monatsabfragen ausführen.
Die originalen UTC-Zeitfelder von UniClubs bleiben als solche erkennbar erhalten;
Sporttermine verwenden die vorhandenen lokalen Zürcher Zeitfelder.

Ein einzelner Termin ist unter `/unisport/events/ID` beziehungsweise
`/uniclubs/events/ID` abrufbar. Die ID stammt aus der Antwort der jeweiligen Quelle;
gleiche Zahlen aus den beiden Datenbanken bedeuten nicht denselben Termin.
Ungültige Daten ergeben HTTP 400, unbekannte Termine HTTP 404.

Unter `/` steht eine kurze Liste der Zugänge, `/health` prüft die Lesbarkeit beider
Quellen und `/sources` zeigt öffentliche Bestands- und Aktualisierungsangaben.
`/uniclubs/clubs` liefert die öffentlichen Clubdaten.

Ein Teammate kann die Daten mit dem aus dem Kurs bekannten `requests` abrufen:

```python
import requests

basis = 'https://DEINE_RENDER_ADRESSE'
antwort = requests.get(
    basis + '/unisport/events',
    params={'from': '2027-02-01', 'to': '2027-02-08'},
    timeout=90,
)
antwort.raise_for_status()
termine = antwort.json()
```

Die lange Client-Wartezeit berücksichtigt einen möglichen Kaltstart beim Free-Tarif.
Innerhalb der laufenden API bleiben Netzwerkabfragen zeitlich begrenzt.

## Welche Daten werden aktualisiert?

- **UniSPORT:** Der veröffentlichte Bestand aus Excel/CSV enthält Termine bis
  31.07.2027. Die API liest diese SQLite-Datei. Sie erfindet keine neuen Nachweise:
  `projected` bleibt eine Prognose, `website` ein datierter Website-Abgleich.
- **UniCLUBS:** Der bestehende GitHub-Workflow aktualisiert öffentliche Daten, geplant
  alle 30 Minuten. GitHub kann Ausführungen verzögern. Die API holt neuere öffentliche
  Daten regelmäßig bei Anfragen und verwendet bei einem Abruffehler den letzten
  lesbaren Bestand. Ein Abruf ist keine neue Quellenprüfung und macht alte Daten nicht frisch.

Die API synchronisiert bei Bedarf; es gibt keinen zusätzlichen Hintergrund-Scheduler.
Der Render-Buildfilter verhindert eine vollständige Neuinstallation allein wegen eines
neuen UniClubs-Datenexports. Änderungen am Programm können weiterhin neu bereitgestellt werden.

## Kursbezug und technische Ergänzungen

Der Datenspeicher bleibt SQLite. Im Quellcode sind die SQL-Abfragen, Parameter und
Tabellenbeziehungen sichtbar (**A07**). Der Client nutzt HTTP, JSON und `requests`
(**A06/L06**). JSON ist nur das Antwortformat.

Flask ist bereits die technische API-Ergänzung des Projekts. **Gunicorn** kommt nur in
`requirements-api.txt` für den Linux-Server hinzu und startet die Flask-Anwendung;
es ist kein Datenbankwerkzeug und wird nicht als gelehrter Kursstoff ausgegeben.
Die Streamlit-Anforderungen in `requirements.txt` bleiben unverändert.

Offizielle Anleitungen: [Flask auf Render](https://render.com/docs/deploy-flask),
[Free-Tarif](https://render.com/docs/free),
[Render Blueprint](https://render.com/docs/blueprint-spec).
