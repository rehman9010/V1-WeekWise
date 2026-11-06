<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-06, Navy & Oak design, SPEC.md v1.0; review pending -->
# WeekWise

WeekWise helps HSG students fit sport, leisure and career activities around classes and private
appointments. Offers are spread across different sources; manually comparing every activity with
a calendar takes time. WeekWise finds free gaps, ranks suitable activities, exports confirmed
plans and learns from attendance feedback.

This checkout implements **M1–M8: data, calendar/gaps, planning, ML/feedback, the local API,
rankings, model evaluation, friends/invitations and the reproducible acceptance check**.
M8 has open external checks: native Windows/macOS runs, real-source review when inputs arrive,
and manual Google/Outlook/browser verification. See [MILESTONE_M8.md](MILESTONE_M8.md).
The interface uses English and Europe/Zurich local time.

## Run

Use Anaconda/Miniconda and Python 3.11. From the repository directory, on Windows (Anaconda
Prompt), macOS or Linux:

```bash
conda create -n weekwise python=3.11
conda activate weekwise
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

Open the local URL printed by Streamlit, normally `http://localhost:8501`. Stop with `Ctrl+C`.
Installation needs package downloads or a prepared package cache; the installed demo needs no
external network. An alternative on macOS/Linux is:

```bash
python3.11 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/streamlit run app.py
```

The prepared cloud environment already has `.venv`. Its interpreter was installed with `uv`;
to refresh dependencies there, use `uv pip install --python .venv/bin/python -r requirements.txt`.
For Windows venv users: `py -3.11 -m venv .venv`, then
`.venv\Scripts\python -m pip install -r requirements.txt` and
`.venv\Scripts\python -m streamlit run app.py`. The commands below use `python` from an activated
environment; replace it with the venv interpreter if you have not activated one.

The first page run creates SQLite, 480 Unisport occurrences, 60 UniClubs events, 30 city events,
the demo calendar, and 60 synthetic profiles with preferences and feedback. No account, URL or
network is needed. Restarting preserves occurrence IDs and your local profile.
The demo is anchored to the Monday of the first-start week and contains four past and eight
current/future weeks. After generating histories, unreferenced past occurrences are cleaned up;
events with interactions or invitations remain available.

## Optional local API

Start the API in a second terminal, using the same environment and checkout:

```bash
python api_server.py
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

To connect Google Calendar, open its web settings, select your calendar, then **Integrate
calendar → Secret address in iCal format**. Paste that address into WeekWise's **My calendar
link**, select **Test link**, then **Use calendar** (or **Finish** during onboarding). This does
not require making the calendar public. If your account hides the secret address, use
**Import & export → Export**, unzip the download and upload the desired `.ics` file instead.

For Outlook on the web, look under **Settings → Calendar → Shared calendars → Publish a
calendar**. If your account permits publishing, use its **ICS** link, not the HTML viewing link.
Publishing makes the chosen details accessible to anyone with that URL; choose the permitted
detail level and keep the link private. If institutional policy disables publishing, export/save
an `.ics` file from your calendar application and use WeekWise's upload option. Provider menu
names can differ by account and version. Uploads are snapshots and need replacing after changes.

To check export compatibility, confirm an activity and download its `.ics`. Import it into a
temporary Google or Outlook calendar, checking title, date and Europe/Zurich time against Details.
Also check **Add to Google Calendar** / **Add to Outlook** before saving the composed entry.
WeekWise supplies links and files; saving in the provider remains your action.

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
python -m unittest discover -s tests -v
python -m views.check_demo
python importers.py
```

Unit tests use fixed clocks, in-memory SQLite and temporary fixtures. HTTP is mocked there.
`python -m views.check_demo` runs the M8 acceptance scenario using the actual Streamlit scripts,
a temporary fresh database and the fixed Zurich clock **2026-11-02 10:00**. It blocks external
requests and starts/stops its own Flask process on a free loopback port. It verifies onboarding,
feedback, suggestions/conflicts, filters, navigation, rankings, friends, model evaluation,
real HTTP GET/POST, API fallback, source refresh and a restored session. It captures the actual
ICS export and checks the Google compose URL. Existing local data and private files are untouched.

The runner lives in `views/` to respect the Streamlit import boundary; neither test helper is a
registered page. Run it through `views.check_demo`, which sets temporary paths before importing
modules with default path arguments. AppTest reruns whole scripts, so dialogs are requeued;
tracked tabs use their session-state labels. This checks UI handlers and chart output, while
browser layout and external calendar-provider behavior still need manual review.

`.github/workflows/check.yml` defines fresh Python 3.11 installations, dependency checks, unit
tests and this scenario on Linux, Windows and macOS. It runs on push, pull request or manual
dispatch. A committed workflow is not evidence of a successful native run; see the milestone
report for the platforms actually tested.

**Stop the app before running `python synthetic_data.py`: this command resets SQLite, including your
local profile and history, and regenerates the full demo.** Normal app startup upgrades M3 data
without resetting it. Remembered files in `data/private/` are retained by the reset.
Source importers default to local placeholder files. The original environment-specific M2–M7
AppTest harnesses remain historical verification evidence; the committed M8 runner works without
those external files. Unit tests also cover failure cases, privacy, time boundaries, model
evaluation and forced transaction rollback.

On a cloud host with a read-only home directory, prefix commands with
`MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache` for writable plotting caches.

## Real activity sources and taxonomy

No real Excel file or API credentials were supplied for M8. To integrate an input (task R):

1. Put the original Unisport workbook at `data/sources/unisport.xlsx`. Inspect its headers and
   map them to SPEC §7.2's canonical columns with `UNISPORT_COLUMN_MAPPING` in `config.py`.
   Check weekday names, dates, times and the validity range; do not infer them from demo data.
2. Copy `sources.example.json` to `data/private/sources.json` (create the directory if needed).
   Enter the actual UniClubs/City HTTPS URLs and tokens there. Adapt `UNICLUBS_FIELD_MAPPING` /
   `CITY_FIELD_MAPPING` after inspecting the actual payload; never put credentials in config.py.
3. Run `python importers.py` or **Settings → Data sources → Refresh event data**. Review rows
   read, skipped reasons, counts and origin. Fix mapping problems and retry. Successful API
   inputs produce ignored local snapshots; failed sources fall back to stored inputs. Empty or
   oversized live imports are rejected to preserve existing data.
4. Review categories on the imported real series, especially **Other** (target below about 15%).
   Add evidence-based keywords if needed. Taxonomy v1 keys are frozen: never rename/delete them;
   additional bundles must be appended. Record any mapping/keyword changes and validation in
   `AI_USAGE.md` and a task R report. No real-data accuracy claim follows from demo statistics.

The sidebar counts actual placeholder series and synthetic users, so real imports can coexist
with clearly labelled demo data. A taxonomy version mismatch stops normal startup and offers an
explicit reset after review; it does not silently remap saved history.

## Files and project structure

| Path | Purpose |
| --- | --- |
| `app.py`, `views/*.py`, `ui_components.py` | App startup, four pages, onboarding and shared widgets/caches |
| `models.py`, `gaps.py`, `categories.py` | Domain objects, interval logic and taxonomy assignment |
| `database.py`, `importers.py`, `synthetic_data.py` | SQLite, source adapters and deterministic demo generation |
| `calendar_import.py`, `calendar_export.py` | Private calendar parsing, ICS and provider links |
| `recommender.py`, `rankings.py`, `friends.py` | Learning/evaluation, anonymous statistics and local social actions |
| `api_server.py`, `api_client.py` | Optional local HTTP API and SQLite fallback |
| `config.py`, `requirements.txt` | Shared settings and exact dependency pins |
| `data/taxonomy.json` | Committed category keys and keyword rules |
| `data/weekwise.sqlite` | Ignored local profile, activities, plans, ratings, friends and import logs |
| `data/sources/`, `data/demo/` | Activity source files and generated demo calendar; generated files ignored |
| `data/private/`, `data/snapshots/` | Ignored remembered calendar/source credentials and API snapshots |
| `tests/`, `views/check_demo.py`, `views/demo_scenario.py` | Unit/rule tests and portable M8 acceptance runner |
| `.github/workflows/check.yml` | Fresh-install CI matrix for Linux, Windows and macOS |

Reset using **Settings → Data sources → I want to reset all WeekWise data → Reset demo data**.
This deletes local app state and regenerates the demo; it retains remembered private files.
Use **Calendar → Forget remembered calendar** to remove remembered calendar inputs separately.
For a backup, stop the app/API first and copy your local data securely; a Git source archive
contains neither your database nor private inputs. The M8 showcase trip appears in newly generated
demo data; existing stored catalogues are preserved until an explicit demo reset.

## Troubleshooting

| Symptom | Check/action |
| --- | --- |
| Wrong Python or missing module | Run `python --version` (3.11), activate `weekwise`, reinstall `requirements.txt` with that interpreter, then `python -m pip check`. |
| Streamlit port already used | Stop your earlier app process or run `python -m streamlit run app.py --server.port 8502`. |
| API port 5001 already used | Stop your previous API process. The app works through SQLite while the API is off; wait 30 seconds and rerun after restarting it. |
| Database locked | Stop extra app/API/importer processes and close database viewers, then retry. Keep the checkout in a writable local folder. Avoid deleting a live database, especially on Windows. |
| Wrong calendar time | Compare the event with Europe/Zurich time; inspect the feed's timezone. Use a new `.ics` upload to rule out a stale provider feed. |
| No activities in a week | Clear filters, select Today, and inspect **Data sources**. For an old demo window, back up first and explicitly reset the demo. |
| Calendar link rejected | Use the ICS URL, Test link again after edits, or upload an `.ics`. See the visible fallback message; private URLs are deliberately omitted from errors. |
| Test cannot bind a local port | Permit loopback sockets in your test environment; `views.check_demo` needs a disposable local API. Unit tests work without sockets. |

## Team and review

Read [MILESTONE_M8.md](MILESTONE_M8.md) for the final acceptance results and remaining platform/calendar
team checks, [STUDY_GUIDE.md](STUDY_GUIDE.md) for the course mapping, and
[AI_USAGE.md](AI_USAGE.md) for the generation record. Team: **WeekWise project team**, HSG
Fundamentals and Methods of Computer Science, autumn 2026. Individual member names were not
provided. Code citations remain pending until the team reviews them; AI must be declared in the
project video, whose voice-over must not be generated.

After the cloud environment was replaced during the interrupted M2 turn, the checkout contained
no application files or earlier commits. M1 was reconstructed from the unchanged specification
before M2 was completed. The current tests verify this reconstruction; it is not a byte-for-byte
restoration of the earlier checkout. A source backup is also kept outside the checkout under
`/workspace/weekwise-tools/`; it should be downloaded or otherwise preserved before replacing
this environment again. Cross-environment restoration has not been verified.


## Appearance: Navy & Oak

The app uses a midnight-blue theme, blue controls, serif headings, a subtle oak-grain
sidebar and softly shaded planning cards. The calendar legend stays on one row; days
stack on small screens and the weekly grid scrolls within its own area when space is
limited. Charts use matching dark backgrounds. Branding and textures are local, so the
design works offline.

The native theme is in `.streamlit/config.toml`; decorative styles and SVG branding are
in `assets/`. Restart the Streamlit process after changing theme configuration. On
Streamlit Community Cloud, reboot the app if a saved theme or running process still
shows the previous appearance. Browser verification covers desktop and mobile layouts;
functional verification uses the same unit suite and isolated demo as before.


### Calendar-first wood layout (2026-11-06)

Week view is now a proportional 07:00–23:00 timetable: every day shares the same hour
lines, overlapping appointments use separate lanes, and confirmed activities occupy
their actual times. Free intervals are lightly shaded; click one to select its day in
the activity workspace below. Ideas remain separate from confirmed calendar entries.
The Week/Day switch is above the calendar, including on phones. The complete week
scrolls within its wooden frame on narrow screens; Day retains the accessible agenda.
All-day labels appear in the day headers. Fully off-window appointments remain
available in the Outside 07:00–23:00 section. Hover the time's help icon for a complete
private appointment title, unless Hide busy titles is enabled.

Oak grain now appears on navigation, buttons, day headers and the calendar frame.
The timetable uses light paper surfaces with dark text. Texture is a local SVG, not
a downloaded image. `calendar_layout.py` owns clipping and collision lanes; pixel
geometry comes from `config.py`. No events, exports, ratings or source data are changed.
