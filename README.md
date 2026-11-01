<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# WeekWise

WeekWise is an HSG course project for finding activities that fit around a student's calendar.
This checkout implements **M1 data, M2 calendar/gaps and the M3 planning MVP**. The interface uses
English and Europe/Zurich local time.

## Run

Python 3.11 is required. From the repository directory:

```bash
python3.11 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/streamlit run app.py
```

The prepared cloud environment already has `.venv`. Its interpreter was installed with `uv`;
to refresh dependencies there, use `uv pip install --python .venv/bin/python -r requirements.txt`.
Windows users can use the Conda commands in CLAUDE.md or the equivalent `.venv\Scripts` paths.

The first page run creates SQLite, 480 Unisport occurrences, 60 UniClubs events, 30 city events,
and the demo calendar. No account, URL or network is needed. Restarting preserves occurrence IDs.
The demo is anchored to the Monday of the first-start week and contains four past and eight
current/future weeks. Past demo occurrences remain stored for later M4 history generation;
automatic historical cleanup is deferred until that milestone.

## First use and planning

Complete the three-step welcome flow: read the introduction, enter a display name and select
at least one focus direction, then use the demo calendar, test your HTTPS link, or upload an ICS
file. A changed link must be tested again before Finish. Invalid input shows a friendly fallback
message and you can explicitly choose Demo. The demo-history checkbox is inactive until M4.

The Week page ranks fitting activities with the specified focus/favorite heuristic. It shows
three suggestions per gap and a **More options** dropdown for the rest. Scores are a baseline,
not trained model probabilities. Open **Details** to favorite a course/series, club or category.

Choose **Confirm** to block that occurrence and remove conflicting suggestions. The dialog then
offers **Download .ics**, **Add to Google Calendar**, **Add to Outlook**, and any source or
registration links. Only one date is exported, including for weekly courses. **Cancel this event**
reopens its slot, restores suggestions, and offers a cancellation ICS. If the exported entry
remains in your external calendar, remove it there manually as the dialog explains.

Sidebar filters combine favorites and selected bundles with OR. Filter mode shows all fitting
matches with links, up to 30 per gap; a note explains any overflow. **Clear filters** returns to
the top three. **Layout → Day** provides a wide single-day view. **Beyond your gaps** contains
off-site, trip, multi-day and outside-window events with an explicit unchecked-calendar warning.

Use **Settings & About → Profile** to edit your name and focus, or **Calendar** to change inputs
and privacy switches. The profile, favorites and confirmations survive a new session.

## Calendar and privacy

- Seven day columns show appointment times, all-day labels and free gaps of at least 60 minutes.
- Previous week / Today / Next week stay within the event catalogue's date range.
- Open **Settings & About → Calendar** to choose Demo, Your link or Uploaded file, then
  **Use calendar**. HTTPS and `webcal://` links are supported; webcal is converted to HTTPS.
- All-day entries are labels by default. **Treat all-day entries as busy** blocks 07:00–23:00.
  **Hide titles of my calendar entries** masks private titles and all-day labels.
- Calendar inputs stay in session memory unless **Remember on this computer** is selected.
  Remembered inputs are in ignored `data/private/`; **Forget remembered calendar** clears them.
- A failed link or upload falls back to a remembered upload, then the demo, with a visible message.
  If every calendar fails, no free gaps are presented as trustworthy availability.
- A confirmed event's **Details → Download .ics** supplies the calendar compatibility sample.
  Its displayed local time is the reference for manual Google/Outlook import testing.

Private calendar blocks and titles are never saved to SQLite or source import logs. Calendar
cache entries live only in memory and expire after five minutes. Only public activity records
are stored. Do not share secret calendar links in issues or commit private source files.

Model training, attendance/rating UI, Rankings and Friends pages belong to later milestones.
The interaction state machine already validates all states; M3 exposes confirm/cancel only.

## Tests and demo tools

```bash
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python synthetic_data.py
.venv/bin/python importers.py
```

The tests use fixed clocks, in-memory SQLite and temporary fixtures. HTTP is mocked.
`synthetic_data.py` regenerates placeholder sources and imports them; it does not reset a user's
database or remove their history. Source importers default to local placeholder files. See
`sources.example.json` for the optional private configuration shape; real feeds are task R.

The current environment's additional AppTest check is stored outside the checkout at
`/workspace/weekwise-tools/check_m3_app.py`. It exercises the actual Streamlit script with a
temporary fresh database and rejects unexpected network calls. It covers onboarding, dialog
actions, filters, navigation, settings, a second session and privacy. AppTest requests whole
scripts, so the harness requeues the open dialog before testing a dialog widget action.

## Review

Read [MILESTONE_M3.md](MILESTONE_M3.md) for the acceptance results and remaining Google/Outlook
team checks, [STUDY_GUIDE.md](STUDY_GUIDE.md) for the course mapping, and
[AI_USAGE.md](AI_USAGE.md) for the generation record.

After the cloud environment was replaced during the interrupted M2 turn, the checkout contained
no application files or earlier commits. M1 was reconstructed from the unchanged specification
before M2 was completed. The current tests verify this reconstruction; it is not a byte-for-byte
restoration of the earlier checkout. A source backup is also kept outside the checkout under
`/workspace/weekwise-tools/`; it should be downloaded or otherwise preserved before replacing
this environment again. Cross-environment restoration has not been verified.
