<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-02, M7, SPEC.md v1.0; review pending -->
# WeekWise

WeekWise is an HSG course project for finding activities that fit around a student's calendar.
This checkout implements **M1–M7: data, calendar/gaps, planning, ML/feedback, the local API,
rankings, model evaluation and friends/invitations**.
The interface uses English and Europe/Zurich local time.

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

## Optional local API

Start the API in a second terminal, using the same environment and checkout:

```bash
.venv/bin/python api_server.py
```

It runs on `http://127.0.0.1:5001` with debug/reloading off and shares the app's SQLite file.
It can start first: it prepares an empty schema, and the first Streamlit page run fills demo data.
The sidebar shows **Event data: own API ✓** when available. Events, event details and interaction
writes use HTTP; the app continues directly with SQLite when the API is unavailable.

Health is cached for 30 seconds. A failed operation immediately switches the status to
**Event data: database (API not running)** and suppresses retries for 30 seconds. A subsequent
UI rerun after that interval discovers a restarted API. No background polling is required.

| Method | Endpoint | Result |
| --- | --- | --- |
| GET | `/health` | Status and version |
| GET | `/categories` | Directions, bundle lists and configured colors |
| GET | `/events?from=2026-11-02&to=2026-11-09` | Events in `[from,to)`; optional `direction`, `bundle`, `source` filters |
| GET | `/events/42` | One event, or JSON 404 |
| POST | `/interactions` | JSON `user_id`, `event_id`, `action`, optional `rating`; stored row with HTTP 201 |

Ranges use strict ISO dates and cover at most 120 days. Money is integer Rappen or null in JSON.
Validation errors return JSON 400, unknown users/events JSON 404. API and database paths share
the same interaction rules. A committed write with a lost response is recovered from SQLite
without duplicating its transition or rating revision.

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

## Friends and invitations

Open **Friends** to copy your friend-code message and add a friend. Codes ignore surrounding
spaces and letter case. Try **LUCA26**; the demo-code expander also lists **SARA26** and **JONAS6**.
Unknown, own and duplicate codes show clear errors. With demo history enabled, Lena, Noah and Mia
are already connected and Lena/Noah have pending invitations. If you skipped history, start by
adding a code. Codes and invitations operate within this installation's SQLite database.

**Share my WeekWise plans with friends** is off by default. When enabled, friends can see your
upcoming confirmed WeekWise activities. The switch also controls your inclusion in their
friends-going names/counts. Private appointments never enter this shared view. Sending an
invitation explicitly shares the selected activity, independently of sharing your overall plans.

Pending invitations appear first among eligible gap suggestions, with an inviter badge. Category
and favorite filters still apply. Every received invitation remains on Friends, including ones
that overlap your calendar. **Accept** saves both the answer and confirmation together, then opens
Details with ICS/Google/Outlook export. A displayed overlap is a warning; you can choose to accept.
**Decline** changes only the invitation. Started/full/closed events cannot be accepted.

For any confirmed future event, **Details → Invite friends → Send invitations** stores one
invitation per selected friend and occurrence. Synthetic friends answer immediately: matching
focus and no overlapping other confirmed plan means accepted, otherwise declined. Answers carry
**(simulated)** labels; real recipients stay pending until they respond. Repeated sends reuse
existing invitations and answers. Accepted demo plans immediately affect friends-going counts.

Friends shows received/sent history and opted-in friends' upcoming plans with Details buttons.
Event details also name the opted-in friends going. A sharing change takes effect on the next
rerun; current friend counts feed suggestion features without forcing an extra model retrain.

## Rankings and model evaluation

**Rankings** compares You with All users (60 synthetic users plus you). Select all categories,
a direction or a bundle, then choose Most popular, Most attended, Best rated or Most feedbacked.
Rankings group activity series across dates; cancelled plans do not count. Best rated requires
one rating for You and three for All users. **Show table** provides anonymous aggregate values.
The category overview counts current confirmed plans and lists average attended ratings.

**Settings & About → About the model** runs five classifiers both unscaled and with RobustScaler.
It shows the ten-row validation comparison, a chart, Random Forest test metrics and classification
report, and its ten most important features. The split is stratified 80/10/10 with seed 13;
scaling fits training rows only. The unweighted evaluation forest fits only the training split.
The separate production forest still uses all eligible rows, weighting genuine ratings fivefold.
These synthetic-data scores illustrate the method and do not establish real-world accuracy.

Evaluation runs only when its tab opens. New or edited ratings invalidate the cached result;
ordinary reruns reuse it for up to one hour. Refresh and reset clear the model/evaluation caches.
Insufficient data produces an explanation instead of invented scores.

**Data sources** shows placeholder counts and import history. **Refresh event data** reimports
all three sources, preserving referenced history and reporting failures per source. **Reset demo
data** requires a confirmation checkbox: it removes your profile, favorites, plans and ratings,
clears session/caches, rebuilds the demo and returns to onboarding. Remembered files in
`data/private/` are retained. A changed taxonomy version offers this same explicit recovery.

**About WeekWise** explains the project, sources and privacy, and displays/downloads `AI_USAGE.md`.
The team label is currently “WeekWise project team”; individual member names were not supplied.

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
`/workspace/weekwise-tools/check_m5_feedback.py` and `check_m5_regression.py`. They exercise the
actual Streamlit script offline with temporary fresh databases. They cover
onboarding with/without demo history, feedback and its five-event limit, exact retrain counts,
rating corrections, deletion, confirm/export/cancel, favorites, filters, navigation and privacy.
AppTest requests whole scripts, so the harness requeues an open event dialog before testing its
widget actions. Native browser fragment behavior and visual layout still merit team review.

`/workspace/weekwise-tools/check_m5_live.py` additionally starts/stops its own local Flask process
against temporary data, tests real GET/POST requests, failure/recovery, sidebar status and
HTTP-driven feedback/retraining. It requires port 5001 to be free and stops its API processes.
Unit tests use Flask's test client and shared in-memory SQLite; they do not require network.

`/workspace/weekwise-tools/check_m6_app.py` covers rankings, tracked tab selection, lazy evaluation,
new/corrected rating invalidation, refresh failures, reset/re-onboarding and retained private files.
AppTest does not expose a tab-selection method, so this harness sets the supported tab label in
session state. It tests the actual tab body and widget actions; it is not a browser layout check.

`/workspace/weekwise-tools/check_m7_app.py` tests the complete Friends flow on a fresh offline
demo: pins/badges, codes, sharing, acceptance/export, simulated answers, duplicate sends, decline,
non-fitting invitations, opt-out privacy and restored sessions. The M7 unit tests additionally
force an invitation-write failure to verify that its confirmation rolls back too.

On a cloud host with a read-only home directory, prefix commands with
`MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache` for writable plotting caches.

## Review

Read [MILESTONE_M7.md](MILESTONE_M7.md) for the acceptance results and remaining Google/Outlook
team checks, [STUDY_GUIDE.md](STUDY_GUIDE.md) for the course mapping, and
[AI_USAGE.md](AI_USAGE.md) for the generation record.

After the cloud environment was replaced during the interrupted M2 turn, the checkout contained
no application files or earlier commits. M1 was reconstructed from the unchanged specification
before M2 was completed. The current tests verify this reconstruction; it is not a byte-for-byte
restoration of the earlier checkout. A source backup is also kept outside the checkout under
`/workspace/weekwise-tools/`; it should be downloaded or otherwise preserved before replacing
this environment again. Cross-environment restoration has not been verified.
