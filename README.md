<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# WeekWise

WeekWise is an HSG course project for finding activities that fit around a student's calendar.
This checkout implements **M1–M4: data, calendar/gaps, planning, ML and feedback**. The interface uses
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
the demo calendar, and 60 synthetic profiles with preferences and feedback. No account, URL or
network is needed. Restarting preserves occurrence IDs and your local profile.
The demo is anchored to the Monday of the first-start week and contains four past and eight
current/future weeks. After generating histories, unreferenced past occurrences are cleaned up;
events with interactions or invitations remain available.

## First use and planning

Complete the three-step welcome flow: read the introduction, enter a display name and select
at least one focus direction, then use the demo calendar, test your HTTPS link, or upload an ICS
file. A changed link must be tested again before Finish. Invalid input shows a friendly fallback
message and you can explicitly choose Demo. **Load demo history** is optional and checked by
default: it adds sample attendance/ratings, up to two pending feedback events and three demo friends.

The Week page ranks fitting activities with a seeded Random Forest trained on attended ratings.
It shows three suggestions per gap and a **More options** dropdown for the rest. Match scores use
the probability of a positive rating plus capped favorite boosts. With fewer than 50 ratings or
only one label class, a visible baseline uses focus and favorites. Open **Details** to favorite
a course/series, club or category.

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
and privacy switches. The profile, favorites, confirmations and ratings survive a new session.

After an activity ends, answer **Attended? → ✓ / ✗** in the week or the **How was it?** popup.
If you attended, choose 1–5 stars. A saved rating updates suggestions once and shows a toast.
No-shows are excluded from training; your genuine ratings have weight five, demo ratings weight one.
Change a rating in **Details**. The popup shows at most five oldest unanswered activities;
**Later** or closing it hides it until the next session. Settings → Profile offers **Clear my
demo history**, enabled by a confirmation checkbox; genuine feedback is retained.

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

The own API, Rankings/model comparison and full Friends/invitation UI remain later milestones.
M4 includes private-calendar-independent demo friend data and sharing-aware friend counts.

## Tests and demo tools

```bash
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python synthetic_data.py
.venv/bin/python importers.py
```

The tests use fixed clocks, in-memory SQLite and temporary fixtures. HTTP is mocked.
**Stop the app before running `synthetic_data.py`: this command resets SQLite, including your
local profile and history, and regenerates the full demo.** Normal app startup upgrades M3 data
without resetting it. Remembered files in `data/private/` are retained by the reset.
Source importers default to local placeholder files. See
`sources.example.json` for the optional private configuration shape; real feeds are task R.

The current environment's additional AppTest check is stored outside the checkout at
`/workspace/weekwise-tools/check_m4_app.py` and `check_m4_regression.py`. They exercise the actual
Streamlit script with temporary fresh databases and reject unexpected network calls. They cover
onboarding with/without demo history, feedback and its five-event limit, exact retrain counts,
rating corrections, deletion, confirm/export/cancel, favorites, filters, navigation and privacy.
AppTest requests whole scripts, so the harness requeues an open event dialog before testing its
widget actions. Native browser fragment behavior and visual layout still merit team review.

## Review

Read [MILESTONE_M4.md](MILESTONE_M4.md) for the acceptance results and remaining Google/Outlook
team checks, [STUDY_GUIDE.md](STUDY_GUIDE.md) for the course mapping, and
[AI_USAGE.md](AI_USAGE.md) for the generation record.

After the cloud environment was replaced during the interrupted M2 turn, the checkout contained
no application files or earlier commits. M1 was reconstructed from the unchanged specification
before M2 was completed. The current tests verify this reconstruction; it is not a byte-for-byte
restoration of the earlier checkout. A source backup is also kept outside the checkout under
`/workspace/weekwise-tools/`; it should be downloaded or otherwise preserved before replacing
this environment again. Cross-environment restoration has not been verified.
