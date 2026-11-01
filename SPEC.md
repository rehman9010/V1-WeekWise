
# WeekWise – End Goal Specification

**Version 1.0 · 1 November 2026 · Status: ready for implementation**

Group project, *Fundamentals and Methods of Computer Science* (FCS-BWL), University of St.Gallen (HSG), autumn semester 2026.

Upload deadline for code, video and declaration of authorship: **Thursday 10.12.2026, 23:59 (Canvas)**.

## 0 Document control

| Item | Value |
|---|---|
| Purpose | Single source of truth for building WeekWise end to end. A coding agent must be able to implement the whole app from this file plus `CLAUDE.md`, without opening the course files. |
| Primary source | `WeekWise_Planning_Summary_v2.pdf` (team decision log + digest of all 26 course files). Course conventions it cites were spot-checked against Assignments 06, 07 and 11. |
| Companion file | `CLAUDE.md` – binding coding rules (toolbox, conventions, AI-citation header, working protocol). |
| Audience | (1) The coding agent that writes all code. (2) The team, who reviews milestones and studies the code for the exam. |
| Out of scope for this document | The 4-minute video and the contribution matrix (grading requirements 7 and 8). |

### 0.1 How to read this specification

**Tags.** Every rule carries the status it had in the planning summary, so the team can see what was decided by whom.

| Tag | Meaning |
|---|---|
| [DECIDED] | Explicitly decided or confirmed by the team. Binding. |
| [COURSE RULE] | Taken from the course material (slides, notebooks). Binding. |
| [DEFAULT D-nn] | A previously open item, or a detail the planning left unspecified, resolved by this spec with a reasoned default. Binding for the agent until the team overrides it. All defaults are listed in the decision register (§22). |
| [STRETCH] | Optional. Build only after all milestone acceptance criteria are met. |

Items the planning marked *[ACCEPTED DEFAULT]* are binding and appear here as [DECIDED]. Planning *[SUGGESTION]* and *[ASSUMPTION]* items that this spec adopts are marked as adopted or turned into a [DEFAULT D-nn].

**Precedence.** For *what the app does*, `SPEC.md` wins. For *how the code is written*, `CLAUDE.md` wins. If the two contradict each other, the agent stops and asks the team. A newer explicit team instruction overrides both.

**Wording.** "must / must not" = binding; "should" = strong default, deviate only with a reason recorded in the milestone report; "may" = optional.

**Names.** Function, class, table and constant names in this spec are the names to use. If a better name is clearly needed, the agent may rename and must say so in the milestone report.

---

## 1 Product

### 1.1 Problem statement (grading requirement 1)

[DEFAULT D-37] Final wording, shown on onboarding step 1 and in *Settings & About → About WeekWise*:

> HSG students juggle fragmented weekly timetables, while extracurricular options are scattered across Unisport, more than 70 student clubs on UniClubs and the city's event listings. Finding activities that actually fit into one's free time and interests is tedious, so students miss out or give up searching. WeekWise reads your own calendar, detects your free slots and uses a machine-learning model trained on student feedback to suggest fitting activities right inside those gaps. A chosen activity lands in your calendar with one click, and every rating you give makes the next suggestions better.

### 1.2 Target users and core loop

**Users.** HSG students whose main calendar (Google, Outlook, Apple, …) already contains their classes and private appointments.

**Core loop.** [DECIDED]

1. **Onboard** – one short questionnaire: which directions to focus on first (Sports, Leisure, Career), plus a display name.
2. **Connect** – paste the secret iCal link of the main calendar, or upload an `.ics` file, or use the demo calendar (default).
3. **Discover** – the week view shows busy blocks; free gaps of at least 60 minutes are filled with ML-ranked suggestions from Unisport, UniClubs and city events, colour-coded by direction. Top 3 per gap, the rest in a dropdown.
4. **Decide** – click a suggestion → details → **Confirm** → overlapping suggestions vanish → add to own calendar (`.ics` download, *Add to Google*, *Add to Outlook*) → follow the external registration link if needed.
5. **Reflect** – after the event the slot asks *Attended?* ✓ / ✗; on ✓ five stars appear. A pop-up on app start catches pending feedback. Every new rating retrains the model.
6. **Compare** – page *Rankings*: anonymised rankings, you vs. all users, overall and per category.
7. **Socialise** – page *Friends*: friend codes, "friends going" badges, invitations (synthetic friends in the demo).

### 1.3 Pages

| Page | Purpose | Status |
|---|---|---|
| Onboarding | Problem statement, questionnaire (directions + display name), calendar connection. Only page visible until onboarding is complete. | [DECIDED] |
| Week (page 1) | Main page. Week grid with busy blocks, suggestions in gaps, filter sidebar, event details / confirm / export, inline feedback. | [DECIDED] |
| Rankings (page 2) | Four ranking metrics, you vs. all users, overall and per category, as matplotlib charts. | [DECIDED] |
| Friends (page 3) | Friend code, add friend, friends' plans, invitations received and sent. | [DECIDED] |
| Settings & About | Edit questionnaire and calendar connection, data-source status and refresh, *About the model* (evaluation), *About WeekWise*. | [DEFAULT D-11] |

### 1.4 Scope

**In scope:** everything in §§4–19.

**Out of scope** (must not be built): login, accounts or OAuth; writing to the student's calendar through an API; registering for events inside WeekWise; runtime calls to any LLM or third-party AI; public deployment (only listed as an extra idea, §21); a mobile app; e-mail or push notifications; scraping `sportangebot.unisg.ch` (a JavaScript app – the team receives an Excel file instead); the video and the contribution matrix.

### 1.5 Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-1 | [COURSE RULE] Graders must be able to run the app: it starts and is fully demonstrable on Windows and macOS with Anaconda Python 3.11 and `pip install -r requirements.txt`, **without** any secret link, API key or network access. Demo mode is the default. |
| NFR-2 | First start creates the database and all demo data automatically (target < 30 s). Later starts render the Week page in < 5 s. |
| NFR-3 | Every network call has a timeout and a fallback. No network or data error may crash the app; the user sees a short, friendly message instead. |
| NFR-4 | [DECIDED] UI language English. Swiss date style (`Mon 02.11.`, `02.11.2026`), 24-hour clock, prices as `CHF 25.00`. |
| NFR-5 | Privacy: titles and details of private calendar entries are never written to the database or to log output; the secret calendar link is never logged, displayed in full, or committed. |
| NFR-6 | Readability: all code follows `CLAUDE.md`; every source file carries the AI-citation header. |
| NFR-7 | Reproducibility: fixed seeds (`RANDOM_SEED = 13`); the same anchor date produces the same demo data. |
| NFR-8 | The Week page renders in < 3 s with up to 500 event occurrences in the visible week on a typical student laptop. |

---

## 2 Mapping to the grading requirements

[COURSE RULE] Each requirement scores 0–3; 16 of 24 points give 100 %. Target: at least "good" (2) everywhere, "outstanding" (3) where marked. The lecture stresses that clarity and a working, well-documented app matter more than technical complexity.

| Req. | How WeekWise covers it | Where | Evidence a grader sees |
|---|---|---|---|
| 1 Problem | Problem statement (§1.1) | Onboarding step 1; Settings & About | Text visible in app and README |
| 2 Data via API / DB | SQLite database in Assignment 07 style; **own small Web API** (Flask) in front of it (aim: 3); importers for the Unisport Excel file, UniClubs API and city events API via `requests`; calendar ICS feed | `database.py`, `api_server.py`, `api_client.py`, `importers.py`, `calendar_import.py` | Sidebar status "Event data: own API" / "database"; Settings → Data sources table |
| 3 Visualisation | Colour-coded week grid with suggestions; matplotlib ranking charts; category overview chart; model-comparison chart and feature-importance chart | `views/week.py`, `views/rankings.py`, `views/settings.py` | Pages Week, Rankings, About the model |
| 4 Interaction | Questionnaire, calendar connection, week navigation, filters, favorites, confirm/cancel, export, ✓/✗ + stars, friend codes, invitations, data refresh | all views | Demo walkthrough (§20, M8 scenario) |
| 5 Machine learning | Own Random Forest predicting "rating ≥ 4", trained on synthetic + real feedback, retrained after each rating; stratified 80/10/10 split and comparison of 5 classifiers (scaled/unscaled) exactly as in Assignments 10–11, shown in the app (aim: 3) | `recommender.py`, About the model | Match % on cards; comparison table and charts |
| 6 Comments | Module headers, docstrings for every module/class/function, *why* comments, "Course:" tags, README, STUDY_GUIDE | every file | Code review |
| 7 Contributions + GenAI | Out of scope for this spec (video). Supported by `AI_USAGE.md` and the AI-citation headers. | `AI_USAGE.md` | – |
| 8 Video | Out of scope. Demo data make the demo reproducible; the voice-over must not be generated. | – | – |

---

## 3 Glossary and business definitions

[COURSE RULE] "Definitions are business decisions": each term below is defined **once**, in exactly one function, constant or method, and used everywhere else through that single definition.

| Term | Definition | Single implementation |
|---|---|---|
| Direction | Top-level category: `sports`, `leisure` or `career`. Same three as the questionnaire. | `data/taxonomy.json`, table `directions` |
| Bundle | Second-level category under one direction (e.g. *Racket sports*). Each direction has an *Other* bundle. | `data/taxonomy.json`, table `bundles` |
| Category | A direction or a bundle (used in filters and rankings). | – |
| Series | One activity as offered by a source: a weekly Unisport course, or a one-off club/city event (a series with one occurrence). Favorites and rankings refer to series. | table `series` |
| Event | One dated occurrence of a series, with start and end. Suggestions, confirmations and ratings refer to events. | table `events`, class `Event` |
| Busy block | A time interval in which the student is not free: a timed entry of the student's calendar, or a confirmed WeekWise event. | class `BusyBlock` |
| Free gap | A period of at least `MIN_GAP_MINUTES` (60) inside the day window (`DAY_WINDOW_START`–`DAY_WINDOW_END`) without busy blocks. | `gaps.find_gaps_for_day()` |
| Fits a gap | An event fits a gap if it starts no earlier than `gap.start − 5 min` and ends no later than `gap.end + 5 min` (overlap tolerance, not a buffer). | `Gap.fits()` |
| Placeable | An event may be placed into gaps: location in St.Gallen, online or TBA; not a trip; not multi-day; status not hidden. Otherwise it goes to *Beyond your gaps*. | `gaps.placement_problem()` |
| Suggestion | A placeable, future, unconfirmed event that fits a gap, with a match score. | class `Suggestion` |
| Match score | Model probability that the user rates the event ≥ 4 stars, plus favorite boost, capped at 1.0; shown as "Match 82 %". | `Recommender.score_events()` |
| Confirmed | The user clicked *Confirm*; status `confirmed`. Counts as "added to calendar". | `database.save_interaction()` |
| Pending feedback | A confirmed event whose end lies in the past and that has no attendance answer yet, or an attended event without a rating. | `database.get_pending_feedback()` |
| Attended / not attended | Answer ✓ / ✗ to *Attended?*. Not attended is **not** a negative signal and is excluded from training. [DECIDED] | `database.save_interaction()` |
| Positive label | Rating ≥ `LABEL_MIN_STARS` (4). | `recommender.make_label()` |
| Most popular | Most often confirmed (status confirmed, attended or not attended – cancelled excluded). | `rankings.get_ranking()` |
| Most attended | Most ✓ answers. | `rankings.get_ranking()` |
| Best rated | Highest average stars; at least 3 ratings for *All users*, at least 1 for *You*. | `rankings.get_ranking()` |
| Most feedbacked | Most ratings. | `rankings.get_ranking()` |
| Friends going | Number of the user's friends with a confirmed (or attended) interaction for the same event. | `friends.friends_going_counts()` |
| Planning horizon | From the earliest to the latest event date stored in the database; the week view can be navigated within this range. [DECIDED] | `gaps.navigation_bounds()` |
| Now | The current local time. Every logic function that depends on time takes `now` as a parameter (default `None` → `datetime.now()`), so tests can fix it. | `CLAUDE.md` rule |

---

## 4 Architecture

### 4.1 Runtime overview

```
 Browser
    │
    ▼
 Streamlit app ── app.py (navigation, first-start setup, feedback pop-up)
    │             views/ (onboarding, week, rankings, friends, settings)
    │             ui_components.py (cards, dialogs, cached loaders)
    │  plain Python calls (no Streamlit below this line)
    ▼
 Logic modules: gaps · recommender · rankings · friends · categories
                calendar_import · calendar_export · importers · synthetic_data
    │
    ├── api_client.py ──HTTP/JSON──► api_server.py (Flask, own small Web API)
    │        │ if the API is not running: direct fallback      │
    │        ▼                                                 ▼
    └──────► database.py (sqlite3, Assignment 07 style) ◄──────┘
                 │
                 ▼
          data/weekwise.sqlite

 importers.py      ◄── Unisport Excel · UniClubs API · city events API
                       (order: live source → stored snapshot → placeholder file)
 calendar_import.py◄── secret iCal link · uploaded .ics · demo calendar .ics
 calendar_export.py ──► .ics download · "Add to Google/Outlook" links
```

Two processes at most: the Streamlit app (required) and the Flask API (optional; the app works without it). [DECIDED] Everything runs locally; there is no deployment.

### 4.2 File structure

[DEFAULT D-18] Same as planning §11.10, with one change: page files live in `views/` instead of `pages/`, because the app uses `st.navigation` to hide the main pages until onboarding is done, and Streamlit treats a folder called `pages/` specially. Flat modules, no packages.

```
weekwise/
├── app.py                 Entry point: page config, first-start setup, navigation, feedback pop-up
├── views/
│   ├── onboarding.py      Onboarding flow (3 steps)
│   ├── week.py            Page 1 – week grid, filter sidebar, "Beyond your gaps"
│   ├── rankings.py        Page 2 – rankings and category overview
│   ├── friends.py         Page 3 – friend code, friends, invitations
│   └── settings.py        Settings & About (profile, calendar, data sources, model, about)
├── ui_components.py       Reusable Streamlit pieces: event card, event dialog, feedback widget/dialog,
│                          badges, cached loader wrappers, sidebar status
├── config.py              ALL constants (paths, thresholds, colours, mappings, seeds)
├── models.py              Domain classes: Event (+3 subclasses), BusyBlock, Gap, Suggestion, UserProfile
├── database.py            Connection, schema, reads/writes for events, users, favorites, interactions, settings
├── api_server.py          Own small Web API (Flask) – thin layer over database.py, no SQL inside
├── api_client.py          Talks to the own API with requests; falls back to database.py
├── importers.py           Unisport Excel, UniClubs and city adapters → canonical records → database
├── categories.py          Frozen taxonomy: loading, seeding, keyword-based assignment
├── calendar_import.py     Read the student's calendar (link / upload / demo) → busy blocks
├── calendar_export.py     Write .ics (plain f-strings) and add-to-calendar links
├── gaps.py                Week helpers, free-gap detection, placing events into gaps
├── recommender.py         Features, label, Random Forest, scoring, model evaluation (A10–A11 style)
├── rankings.py            Ranking metrics (SQL aggregates) and their matplotlib charts
├── friends.py             Friend codes, friendships, friends going, invitations (incl. simulation)
├── synthetic_data.py      All demo data: placeholder sources, users, interactions, friends, demo calendar
├── data/
│   ├── taxonomy.json      Frozen category taxonomy (committed)
│   ├── sources/           Input files: real unisport.xlsx (when delivered) and generated placeholders
│   ├── snapshots/         Last successful live API responses (generated, git-ignored)
│   ├── demo/              demo_calendar.ics (generated)
│   ├── private/           Git-ignored: sources.json (API URLs/tokens), remembered calendar link/upload
│   └── weekwise.sqlite    The database (generated, git-ignored)
├── tests/                 unittest test modules (§18)
├── README.md              Setup and start instructions (§19)
├── STUDY_GUIDE.md         File/function → course topic map (§19)
├── AI_USAGE.md            Every use of AI in the project (§19)
├── CLAUDE.md              Binding rules for the coding agent
├── SPEC.md                This specification
├── requirements.txt       Pinned versions (§19)
└── .gitignore
```

### 4.3 Module contracts and dependency rules

Dependency rules (binding):

* `streamlit` is imported **only** in `app.py`, `views/*.py` and `ui_components.py`. All other modules are plain Python and testable with `unittest`.
* `flask` only in `api_server.py`. `requests` only in `api_client.py`, `importers.py`, `calendar_import.py`. `icalendar` and `recurring_ical_events` only in `calendar_import.py`. `sklearn` only in `recommender.py`. `matplotlib` only in `rankings.py` and `recommender.py` (functions that *return* a `Figure`; the UI shows it with `st.pyplot`).
* SQL statements live in `database.py`, `rankings.py` (aggregate queries) and `friends.py` (friend/invitation queries). All three obtain connections from `database.get_connection()`.
* `config.py` imports only the standard library. No module may import a view.

Public functions per module (names binding; parameters may gain optional extras). Every function that depends on the current time takes `now=None`.

| Module | Public functions / classes |
|---|---|
| `models.py` | `Event`, `UnisportCourse`, `ClubEvent`, `CityEvent`, `BusyBlock`, `Gap`, `Suggestion`, `UserProfile`; `event_from_row(row)`; `price_from_rappen(rappen)`; `price_to_rappen(price)` |
| `database.py` | `get_connection(db_file=DB_FILE)`; `init_db(connection)`; `to_db_text(dt)`; `from_db_text(text)`; `get_setting(connection, key, default=None)`; `set_setting(connection, key, value)`; `upsert_club(...)`; `upsert_series(connection, record, bundle_id, club_id)`; `upsert_event(connection, series_id, start, end, end_is_estimated, status)`; `get_events_df(connection, start, end)`; `get_event_row(connection, event_id)`; `get_event_date_range(connection)`; `get_categories_df(connection)`; `create_user(...)`; `get_current_user_id(connection)`; `set_user_directions(connection, user_id, direction_keys)`; `get_user_profile(connection, user_id)`; `add_favorite(...)`; `remove_favorite(...)`; `save_interaction(connection, user_id, event_id, action, rating=None, now=None)`; `get_user_interactions_df(connection, user_id)`; `get_user_bundle_averages(connection, user_id, now=None)`; `get_pending_feedback(connection, user_id, now=None)`; `delete_past_unconfirmed_events(connection, now=None)`; `log_import(...)`; `get_import_log_df(connection)`; `reset_database(connection)` |
| `importers.py` | `read_private_sources()`; `load_unisport_excel(path, column_mapping)`; `expand_weekly_course(weekday, start_time, end_time, valid_from, valid_to)`; `fetch_json(url, token)`; `parse_uniclubs_payload(payload, field_mapping)`; `parse_city_payload(payload, field_mapping)`; `classify_location(location_text)`; `estimate_end(start, event_type)`; `normalize_record(raw)`; `save_records(connection, records, source, taxonomy, include_past=False, now=None)`; `import_source(connection, source, taxonomy, include_past=False, now=None)`; `import_all(connection, taxonomy, include_past=False, now=None)` |
| `categories.py` | `load_taxonomy(path=TAXONOMY_FILE)`; `validate_taxonomy(taxonomy)`; `seed_categories(connection, taxonomy)`; `normalize_text(text)`; `keyword_matches(keyword, words, text)`; `bundle_score(bundle, title, description)`; `assign_category(record, taxonomy)`; `direction_color(direction_key, kind='streamlit')` |
| `calendar_import.py` | `normalize_calendar_link(link)`; `fetch_ics_text(link)`; `parse_calendar(ics_text, range_start, range_end, all_day_busy=False)`; `to_local_naive(value)`; `extract_weekwise_event_id(uid, description)`; `load_calendar(mode, range_start, range_end, link=None, upload_bytes=None)` |
| `calendar_export.py` | `make_uid(event)`; `escape_ics_text(text)`; `fold_ics_line(line)`; `format_utc(dt)`; `build_description(event)`; `build_ics(event, cancel=False, now=None)`; `google_calendar_link(event)`; `outlook_calendar_link(event)` |
| `gaps.py` | `week_start(day)`; `week_days(monday)`; `navigation_bounds(first_date, last_date)`; `merge_intervals(intervals)`; `find_gaps_for_day(day, busy_intervals, ...)`; `find_gaps_for_week(monday, busy_intervals)`; `placement_problem(event)`; `assign_events_to_gaps(events, gaps, confirmed_ids, now=None)` |
| `recommender.py` | `make_label(rating)`; `event_feature_row(event, profile, friends_going, earlier_avg_rating)`; `build_training_data(connection, now=None)`; `to_feature_matrix(rows, feature_columns=None)`; `train_production_model(X, y, weights)`; `heuristic_score(event, profile, friends_going)`; `favorite_boost(event, profile)`; `Recommender` (class with `score_events(...)`); `evaluate_models(X, y)`; `feature_importances(model, feature_columns)`; `plot_model_comparison(results_df)`; `plot_feature_importances(importances_df)` |
| `rankings.py` | `METRICS`; `get_ranking(connection, metric, scope, user_id, direction_key=None, bundle_key=None, top_n=RANKING_TOP_N)`; `category_overview(connection, user_id)`; `plot_ranking(ranking_df, title)`; `plot_category_overview(overview_df)` |
| `friends.py` | `generate_friend_code(existing_codes, rng)`; `add_friend(connection, user_id, code)`; `get_friends_df(connection, user_id)`; `get_friend_plans_df(connection, user_id, now=None)`; `friends_going_counts(connection, user_id, event_ids)`; `send_invitations(connection, sender_id, event_id, friend_ids, now=None)`; `simulate_friend_response(connection, invitation_id, now=None)`; `get_invitations(connection, user_id)`; `respond_to_invitation(connection, invitation_id, accept, now=None)` |
| `synthetic_data.py` | `generate_all(connection, anchor_date=None)`; `write_placeholder_sources(anchor_date)`; `write_demo_calendar(anchor_date)`; `create_synthetic_users(connection, rng)`; `create_synthetic_friendships(connection, rng)`; `create_synthetic_interactions(connection, rng, now)`; `create_demo_history(connection, user_id, now=None)`; `create_showcase_friends(connection, user_id, now=None)` |
| `api_client.py` | `is_api_available()`; `get_events(start, end)` → `(events, origin)`; `get_event(event_id)`; `save_interaction(user_id, event_id, action, rating=None)` → `(result, origin)` |
| `ui_components.py` | `cached_connection()`; `render_sidebar_status()`; `direction_badge(direction_key, label)`; `render_event_card(suggestion, key_prefix, show_link=False)`; `show_event_dialog(event_id)`; `render_feedback_widget(interaction, key_prefix)`; `show_feedback_dialog(pending)`; `cached_events(start, end)`; `cached_calendar(mode, start, end, link, upload_bytes)`; `cached_recommender(n_ratings)`; `cached_evaluation(n_ratings)` |

### 4.4 `config.py` – all constants

[COURSE RULE] No magic numbers in code: every threshold, colour, path and mapping lives here, named in `UPPER_CASE`. The agent creates `config.py` with at least these constants and values (comments explain each one).

```python
# config.py  (excerpt – values binding, comments to be written out in full)
from datetime import date, time
from pathlib import Path

APP_NAME = 'WeekWise'
APP_VERSION = '1.0'
TIMEZONE_NAME = 'Europe/Zurich'
DB_TIME_FORMAT = '%Y-%m-%d %H:%M:%S'   # how datetimes are stored as text
RANDOM_SEED = 13                       # Course: A11 uses seed 13

# Paths
BASE_DIR = Path(__file__).parent
DATA_DIR = BASE_DIR / 'data'
DB_FILE = DATA_DIR / 'weekwise.sqlite'
TAXONOMY_FILE = DATA_DIR / 'taxonomy.json'
SOURCES_DIR = DATA_DIR / 'sources'
SNAPSHOT_DIR = DATA_DIR / 'snapshots'
DEMO_DIR = DATA_DIR / 'demo'
PRIVATE_DIR = DATA_DIR / 'private'
UNISPORT_REAL_FILE = SOURCES_DIR / 'unisport.xlsx'
UNISPORT_PLACEHOLDER_FILE = SOURCES_DIR / 'unisport_placeholder.xlsx'
UNICLUBS_PLACEHOLDER_FILE = SOURCES_DIR / 'uniclubs_placeholder.json'
CITY_PLACEHOLDER_FILE = SOURCES_DIR / 'city_placeholder.json'
DEMO_CALENDAR_FILE = DEMO_DIR / 'demo_calendar.ics'
PRIVATE_SOURCES_FILE = PRIVATE_DIR / 'sources.json'
PRIVATE_CALENDAR_LINK_FILE = PRIVATE_DIR / 'calendar_link.txt'
PRIVATE_CALENDAR_UPLOAD_FILE = PRIVATE_DIR / 'calendar_upload.ics'

# Gap detection  (§10)
MIN_GAP_MINUTES = 60                   # [DECIDED]
OVERLAP_TOLERANCE_MINUTES = 5          # [DECIDED] tolerance, not a buffer
DAY_WINDOW_START = time(7, 0)          # [DEFAULT D-09]
DAY_WINDOW_END = time(23, 0)           # [DEFAULT D-09]
ALL_DAY_ENTRIES_ARE_BUSY = False       # [DEFAULT D-16] user can switch it in Settings

# Suggestions  (§12, §16)
TOP_SUGGESTIONS_PER_GAP = 3            # [DECIDED]
MAX_EVENTS_PER_GAP_FILTER_VIEW = 30    # sanity cap for the category-filter view
FAVORITE_BOOST = 0.10                  # [DEFAULT D-22] per matching favorite kind
MAX_FAVORITE_BOOST = 0.25              # [DEFAULT D-22]
FILTER_LIST_MAX = 8                    # more bundles than this per direction → multiselect

# Event data  (§7)
DEFAULT_DURATION_MINUTES = 90          # [DECIDED] missing end time → 1.5 h, flagged
USE_TYPE_DURATIONS = False             # [DEFAULT D-07] documented alternative
DURATION_BY_TYPE_MINUTES = {'Talk': 90, 'Workshop': 120, 'Social': 180}
HIDDEN_STATUSES = {'full', 'closed'}   # [DECIDED]
PLACEABLE_LOCATION_TYPES = {'st_gallen', 'online', 'tba'}
NOT_PLACEABLE_EVENT_TYPES = {'Trip'}
UNISPORT_DEFAULT_VALID_TO = date(2026, 12, 20)  # [DEFAULT D-36] if the Excel has no end date
MAX_RECORDS_PER_SOURCE = 5000          # sanity check: reject absurd imports
HTTP_TIMEOUT_SECONDS = 10
MAX_CALENDAR_BYTES = 5000000           # 5 MB
UNISPORT_COLUMN_MAPPING = {...}        # §7.2
UNICLUBS_FIELD_MAPPING = {...}         # §7.3
CITY_FIELD_MAPPING = {...}             # §7.4
WEEKDAY_NAMES = {...}                  # §7.2 – English and German names/abbreviations → 0..6
LOCATION_KEYWORDS = {...}              # §7.5

# Calendar export  (§11)
WEEKWISE_UID_DOMAIN = 'weekwise'
WEEKWISE_MARKER_PREFIX = 'WeekWise ID: '
GOOGLE_CALENDAR_URL = 'https://calendar.google.com/calendar/render'
OUTLOOK_CALENDAR_URL = 'https://outlook.office.com/calendar/0/deeplink/compose'

# Machine learning  (§12)
LABEL_MIN_STARS = 4                    # [DECIDED]
TRAIN_SHARE, VAL_SHARE, TEST_SHARE = 0.8, 0.1, 0.1   # Course: A11
REAL_USER_SAMPLE_WEIGHT = 5.0          # [DEFAULT D-23]
SYNTHETIC_SAMPLE_WEIGHT = 1.0
RF_N_ESTIMATORS = 200
RF_MAX_DEPTH = 10
MIN_TRAINING_ROWS = 50                 # fewer rows → heuristic scoring
NEUTRAL_RATING = 3.0                   # stands in for "no earlier rating"

# Rankings  (§13)
MIN_RATINGS_ALL_USERS = 3              # [DECIDED]
MIN_RATINGS_YOU = 1                    # [DEFAULT D-24]
RANKING_TOP_N = 10

# Friends  (§14)
FRIEND_CODE_LENGTH = 6
FRIEND_CODE_ALPHABET = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'   # no 0/O, 1/I
DISPLAY_NAME_MIN, DISPLAY_NAME_MAX = 2, 20

# Synthetic data  (§17)
N_SYNTHETIC_USERS = 60
HISTORY_WEEKS = 4
FUTURE_WEEKS = 8

# Own API  (§15)
API_HOST = '127.0.0.1'
API_PORT = 5001                        # 5000 is taken by AirPlay on many Macs
API_BASE_URL = f'http://{API_HOST}:{API_PORT}'
API_TIMEOUT_SECONDS = 2
MAX_API_RANGE_DAYS = 120

# Colours  (§8.1) – Streamlit palette name for UI, hex for matplotlib
DIRECTION_COLORS = {
    'sports':  {'streamlit': 'blue',   'hex': '#1f77b4'},
    'leisure': {'streamlit': 'orange', 'hex': '#ff7f0e'},
    'career':  {'streamlit': 'violet', 'hex': '#9467bd'},
}
BUSY_COLOR = {'streamlit': 'gray', 'hex': '#7f7f7f'}
```

### 4.5 Conventions for time, money and identifiers

* **Time.** [DEFAULT D-19] Inside WeekWise every `datetime` is **naive and means Europe/Zurich wall-clock time**. Conversion happens only at the boundaries: `calendar_import.to_local_naive()` (reading) and `calendar_export.format_utc()` (writing), both using `zoneinfo.ZoneInfo(TIMEZONE_NAME)`. The database stores times as text `'YYYY-MM-DD HH:MM:SS'` (`DB_TIME_FORMAT`; `database.to_db_text()` / `from_db_text()`), which SQLite's `DATE()` understands (Course: A07). `models.event_from_row()` parses the same format with `datetime.strptime(text, DB_TIME_FORMAT)` itself, so `models.py` never imports `database.py` (no circular imports: `models` depends only on `config`).
* **Money.** [COURSE RULE] Prices are `Decimal` objects created from strings (`Decimal('25.00')`), never floats. The database stores integer Rappen in `price_rappen` (NULL = unknown, 0 = free). Conversion only via `models.price_from_rappen()` / `price_to_rappen()`. The ML feature `price_chf` is the only place a price becomes a float, with a comment explaining why that is acceptable there.
* **Identifiers.** [COURSE RULE] Every table has an integer primary key column called `id` (so `pd.read_sql(..., index_col=['id'])` always works, as in A07). Natural keys get `UNIQUE` constraints.

### 4.6 Start-up sequence

`app.py` runs these steps on every script run (Streamlit reruns the script after each interaction); expensive steps are cached so they effectively run once per session:

1. `st.set_page_config(page_title='WeekWise', page_icon=':material/calendar_month:', layout='wide')`.
2. Open the connection (`ui_components.cached_connection()`, which wraps `database.get_connection()` in `st.cache_resource`) and run `init_db()` (idempotent, `CREATE TABLE IF NOT EXISTS`).
3. **First start** (setting `setup_done` missing): show a spinner "Preparing demo data…", then `categories.seed_categories()`, `synthetic_data.generate_all()` (writes placeholder sources and demo calendar, imports them with `include_past=True`, creates synthetic users, friendships and interactions), set `setup_done`.
4. **Every new session** (flag in `st.session_state`): `importers.import_all()` (live → snapshot → placeholder per source, see §7.6) and `database.delete_past_unconfirmed_events()`.
5. If the local user has not finished onboarding → navigation shows only *Onboarding*. Otherwise → *Week* (default), *Rankings*, *Friends*, *Settings & About*.
6. After onboarding, once per session, before the page runs: if `get_pending_feedback()` is not empty → `show_feedback_dialog()` (§16.5).
7. `render_sidebar_status()`, then `page.run()`.

---

## 5 Domain model (`models.py`)

[COURSE RULE] This is the most exam-relevant part of the code (L05, U05 s2, A05). Explicit `__init__` methods (no dataclasses), validation through `@property` setters that `raise ValueError` with a clear message, `__str__` and `__repr__`, class constants in `UPPER_CASE`, `_underscore` names for internal attributes, inheritance with `super().__init__()` and polymorphic method overriding.

### 5.1 `Event` and its subclasses

```
Event                      base class: one dated occurrence of a series
├── UnisportCourse         SOURCE = 'unisport'
├── ClubEvent              SOURCE = 'uniclubs'
└── CityEvent              SOURCE = 'city'
```

**`Event.__init__`** parameters (all keyword-friendly): `event_id`, `series_id`, `title`, `start`, `end`, `location`, `location_type`, `direction_key`, `bundle_key`, `bundle_name`, `description=''`, `organizer=''`, `event_type=''`, `source_url=''`, `registration_url=''`, `price=None`, `members_only=False`, `end_is_estimated=False`, `status='open'`, `is_recurring=False`, `club_id=None`.

| Member | Kind | Rule |
|---|---|---|
| `SOURCE` | class constant | `'generic'` in the base class, overridden by subclasses |
| `start`, `end` | properties | Must be `datetime`; `end` must be after `start`, otherwise `ValueError('End time must be after start time.')`. Set `start` before `end` in `__init__`. |
| `price` | property | `None` or a `Decimal` ≥ 0; a `str` is converted with `Decimal(text)`; a `float` raises `ValueError` (money is never float). |
| `location_type` | property | One of `'st_gallen'`, `'online'`, `'elsewhere'`, `'tba'`, else `ValueError`. |
| `status` | property | One of `'open'`, `'apply'`, `'members_only'`, `'ongoing'`, `'full'`, `'closed'`. |
| `duration_minutes()` | method | Whole minutes between start and end. |
| `is_multi_day()` | method | `True` if end date ≠ start date (an event ending exactly at midnight counts as single-day). |
| `overlap_minutes(start, end)` | method | Minutes of overlap with the interval `[start, end)`, 0 if none. |
| `source_label()` | method | Overridden: `'Unisport HSG'`, the club name, `'St.Gallen city events'`. |
| `price_label()` | method | `'Free'`, `'CHF 25.00'` or `''` if unknown. |
| `display_tags()` | method | List of short strings shown as tags: `'Estimated end'`, `'Members only'`, price label, `'Apply'`, `'Location TBA'`. Subclasses extend the list via `super().display_tags()` (e.g. `UnisportCourse` adds its level, `ClubEvent` its event type). |
| `__str__` / `__repr__` | methods | `'Volleyball Intermediate · Thu 05.11. 18:30–20:00'` / `Event(event_id=42, title='…')` |

Subclass extras: `UnisportCourse(sport, level, …)`; `ClubEvent(club_name, club_category, …)`; `CityEvent(venue, …)`. `event_from_row(row)` picks the class from a dictionary `{'unisport': UnisportCourse, 'uniclubs': ClubEvent, 'city': CityEvent}` by `row['source']` (polymorphism instead of `if` chains) and builds the object from a database/API row.

### 5.2 Other classes

| Class | Attributes | Methods / rules |
|---|---|---|
| `BusyBlock` | `start`, `end`, `title` (display only, may be `''`), `is_weekwise=False`, `weekwise_event_id=None` | Same start/end validation as `Event`; `__str__`. Never persisted. |
| `Gap` | `day`, `start`, `end`, `gap_id` (string `'YYYYMMDD-HHMM'`) | `duration_minutes()`; `fits(event, tolerance_minutes=OVERLAP_TOLERANCE_MINUTES)` → `event.start >= start − tol` **and** `event.end <= end + tol`; `label()` → `'Free 12:00–14:00 (2 h)'`. |
| `Suggestion` | `event`, `score` (0.0–1.0), `reasons` (list of str, e.g. `'Favorite club'`, `'2 friends going'`, `'Matches your focus: Sports'`), `friends_going=0`, `invited_by=None` | `match_label()` → `'Match 82 %'`; `is_invited()`. |
| `UserProfile` | `user_id`, `display_name`, `friend_code`, `directions` (set of keys), `favorite_series_ids`, `favorite_club_ids`, `favorite_bundle_keys` (sets) | `wants_direction(event)`; `favorite_kinds(event)` → list of matching kinds (`'series'`, `'club'`, `'bundle'`); `display_name` property validates length `DISPLAY_NAME_MIN`–`DISPLAY_NAME_MAX` and non-blank. Membership tests use the sets (Course: L04 hashing reality check). |

---

## 6 Database (`database.py`, SQLite)

[COURSE RULE] Exactly Assignment 07 style: `pathlib.Path` for the file, `sqlite3.connect(db_file, timeout=10, check_same_thread=False)`, `PRAGMA foreign_keys = 1` right after connecting, schema with `CREATE TABLE IF NOT EXISTS` and explicit `PRIMARY KEY` / `FOREIGN KEY` / `NOT NULL`, reads with `pd.read_sql(..., index_col=['id'], params=(...))` or `fetchone()`/`fetchall()`, every write inside `with DB:`, **only `?` placeholders**, never f-strings in SQL. Booleans are stored as `INTEGER` 0/1.

`check_same_thread=False` is required and must be explained in a comment: Streamlit may run successive reruns of a session in different threads, and the cached connection is reused across them. WeekWise is a single-user local app whose writes all happen inside `with connection:`, so sharing one connection is safe here. The Flask API opens its own connection per request.

### 6.1 Schema (binding DDL)

```sql
CREATE TABLE IF NOT EXISTS directions (
    id          INTEGER PRIMARY KEY,
    key         TEXT NOT NULL UNIQUE,          -- 'sports' | 'leisure' | 'career'
    name        TEXT NOT NULL,
    sort_order  INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS bundles (
    id            INTEGER PRIMARY KEY,
    key           TEXT NOT NULL UNIQUE,        -- e.g. 'sports_racket'
    direction_id  INTEGER NOT NULL REFERENCES directions(id),
    name          TEXT NOT NULL,
    is_other      INTEGER NOT NULL DEFAULT 0,
    sort_order    INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS clubs (
    id             INTEGER PRIMARY KEY,
    name           TEXT NOT NULL UNIQUE,
    club_category  TEXT,                       -- UniClubs category, e.g. 'Business'
    url            TEXT
);

CREATE TABLE IF NOT EXISTS series (
    id                INTEGER PRIMARY KEY,
    source            TEXT NOT NULL CHECK (source IN ('unisport', 'uniclubs', 'city')),
    external_id       TEXT NOT NULL,
    title             TEXT NOT NULL,
    description       TEXT NOT NULL DEFAULT '',
    organizer         TEXT NOT NULL DEFAULT '',
    club_id           INTEGER REFERENCES clubs(id),
    bundle_id         INTEGER NOT NULL REFERENCES bundles(id),
    event_type        TEXT NOT NULL DEFAULT '',
    sport             TEXT NOT NULL DEFAULT '',       -- Unisport only
    level             TEXT NOT NULL DEFAULT '',       -- Unisport only
    location          TEXT NOT NULL DEFAULT '',
    location_type     TEXT NOT NULL CHECK (location_type IN ('st_gallen', 'online', 'elsewhere', 'tba')),
    source_url        TEXT NOT NULL DEFAULT '',
    registration_url  TEXT NOT NULL DEFAULT '',
    price_rappen      INTEGER,                        -- NULL = unknown, 0 = free
    members_only      INTEGER NOT NULL DEFAULT 0,
    is_recurring      INTEGER NOT NULL DEFAULT 0,
    is_placeholder    INTEGER NOT NULL DEFAULT 0,     -- 1 = synthetic placeholder data
    UNIQUE (source, external_id)
);

CREATE TABLE IF NOT EXISTS events (
    id                INTEGER PRIMARY KEY,
    series_id         INTEGER NOT NULL REFERENCES series(id),
    start_time        TEXT NOT NULL,                  -- 'YYYY-MM-DD HH:MM:SS', Europe/Zurich
    end_time          TEXT NOT NULL,
    end_is_estimated  INTEGER NOT NULL DEFAULT 0,
    status            TEXT NOT NULL DEFAULT 'open'
                      CHECK (status IN ('open', 'apply', 'members_only', 'ongoing', 'full', 'closed')),
    UNIQUE (series_id, start_time)
);

CREATE TABLE IF NOT EXISTS users (
    id              INTEGER PRIMARY KEY,
    display_name    TEXT NOT NULL,
    friend_code     TEXT NOT NULL UNIQUE,
    is_synthetic    INTEGER NOT NULL DEFAULT 1,
    sharing_opt_in  INTEGER NOT NULL DEFAULT 0,
    created_at      TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS user_directions (
    id            INTEGER PRIMARY KEY,
    user_id       INTEGER NOT NULL REFERENCES users(id),
    direction_id  INTEGER NOT NULL REFERENCES directions(id),
    UNIQUE (user_id, direction_id)
);

-- One table for three kinds of favorites. target_id points to series.id, clubs.id or
-- bundles.id depending on kind, so SQLite cannot enforce it as a foreign key:
-- add_favorite() checks that the target exists before inserting.  [DEFAULT D-21]
CREATE TABLE IF NOT EXISTS favorites (
    id          INTEGER PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id),
    kind        TEXT NOT NULL CHECK (kind IN ('series', 'club', 'bundle')),
    target_id   INTEGER NOT NULL,
    created_at  TEXT NOT NULL,
    UNIQUE (user_id, kind, target_id)
);

CREATE TABLE IF NOT EXISTS interactions (
    id            INTEGER PRIMARY KEY,
    user_id       INTEGER NOT NULL REFERENCES users(id),
    event_id      INTEGER NOT NULL REFERENCES events(id),
    status        TEXT NOT NULL CHECK (status IN ('confirmed', 'cancelled', 'attended', 'not_attended')),
    rating        INTEGER CHECK (rating BETWEEN 1 AND 5),
    confirmed_at  TEXT NOT NULL,
    answered_at   TEXT,
    is_demo       INTEGER NOT NULL DEFAULT 0,   -- 1 = demo history of the local user  [DEFAULT D-25]
    UNIQUE (user_id, event_id),
    CHECK (rating IS NULL OR status = 'attended')
);

CREATE TABLE IF NOT EXISTS friendships (
    id          INTEGER PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id),
    friend_id   INTEGER NOT NULL REFERENCES users(id),
    created_at  TEXT NOT NULL,
    UNIQUE (user_id, friend_id),
    CHECK (user_id <> friend_id)
);

CREATE TABLE IF NOT EXISTS invitations (
    id            INTEGER PRIMARY KEY,
    sender_id     INTEGER NOT NULL REFERENCES users(id),
    recipient_id  INTEGER NOT NULL REFERENCES users(id),
    event_id      INTEGER NOT NULL REFERENCES events(id),
    status        TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'declined')),
    is_simulated  INTEGER NOT NULL DEFAULT 0,
    created_at    TEXT NOT NULL,
    answered_at   TEXT,
    UNIQUE (sender_id, recipient_id, event_id)
);

CREATE TABLE IF NOT EXISTS app_settings (
    id     INTEGER PRIMARY KEY,
    key    TEXT NOT NULL UNIQUE,
    value  TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS import_log (
    id           INTEGER PRIMARY KEY,
    source       TEXT NOT NULL,
    started_at   TEXT NOT NULL,
    origin       TEXT NOT NULL CHECK (origin IN ('live', 'snapshot', 'placeholder', 'excel')),
    n_series     INTEGER NOT NULL DEFAULT 0,
    n_events     INTEGER NOT NULL DEFAULT 0,
    outcome      TEXT NOT NULL CHECK (outcome IN ('ok', 'failed', 'rejected')),
    message      TEXT NOT NULL DEFAULT ''
);
```

[DEFAULT D-20] The split into `series` (the activity) and `events` (its dated occurrences) lets a weekly Unisport course be favorited and ranked as one activity, while confirmations and ratings stay attached to a single occurrence. One-off events are series with exactly one event.

**Friendships** are stored in both directions (two rows) so "my friends" is one simple `WHERE user_id = ?`.

**`app_settings` keys:** `setup_done`, `current_user_id`, `onboarding_done`, `calendar_mode` (`'demo' | 'link' | 'upload'`), `all_day_busy` (`'0' | '1'`), `hide_busy_titles` (`'0' | '1'`), `taxonomy_version`, `demo_anchor_date`. Never the calendar link itself.

### 6.2 Interaction state machine

`save_interaction(connection, user_id, event_id, action, rating=None, now=None)` is the only function that writes `interactions`. Allowed transitions (anything else raises `ValueError`):

| Action | From | To | Extra rules |
|---|---|---|---|
| `confirm` | no row, or `cancelled` | `confirmed` | The event must not have started yet (`event.start > now`); this also applies when an invitation is accepted. Sets `confirmed_at`. |
| `cancel` | `confirmed` | `cancelled` | Only before the event starts. |
| `attended` | `confirmed` | `attended` (rating NULL) | Only after the event ended. Sets `answered_at`. |
| `not_attended` | `confirmed` | `not_attended` | Only after the event ended. Sets `answered_at`. |
| `rate` | `attended` | `attended` with rating 1–5 | Rating must be an `int` 1–5. A rating may be changed later (same action). |

Each successful `rate` changes the total number of ratings, which is the cache key that triggers exactly one retrain (§12.6).

### 6.3 Key queries (examples of the expected style)

```python
def get_events_df(connection, start, end):
    """Return all events that start in [start, end) with their series and category data.

    :param connection: open sqlite3 connection
    :param start: datetime, inclusive lower bound (Europe/Zurich)
    :param end: datetime, exclusive upper bound
    :return: pandas DataFrame indexed by event id
    """
    query = """
        SELECT e.id, e.series_id, e.start_time, e.end_time, e.end_is_estimated, e.status,
               s.source, s.title, s.description, s.organizer, s.event_type, s.sport, s.level,
               s.location, s.location_type, s.source_url, s.registration_url, s.price_rappen,
               s.members_only, s.is_recurring, s.club_id, c.name AS club_name,
               c.club_category, b.key AS bundle_key, b.name AS bundle_name, d.key AS direction_key
        FROM events AS e
        JOIN series AS s ON s.id = e.series_id
        JOIN bundles AS b ON b.id = s.bundle_id
        JOIN directions AS d ON d.id = b.direction_id
        LEFT JOIN clubs AS c ON c.id = s.club_id
        WHERE e.start_time >= ? AND e.start_time < ?
        ORDER BY e.start_time
    """
    return pd.read_sql(query, connection, index_col=['id'],
                       params=(to_db_text(start), to_db_text(end)))
```

`delete_past_unconfirmed_events(connection, now)` deletes events whose `end_time < now` that have **no** row in `interactions` and none in `invitations` ([DECIDED] past potential events disappear). Events with interactions stay, because rankings and training need them.

---

## 7 Event data sources and importers (`importers.py`)

[DECIDED] Three sources: Unisport HSG (Excel file → own database → own API), UniClubs (API access promised), City of St.Gallen events (API to be added later). None of the real inputs is available yet, so every source gets an **adapter with a configurable field mapping** plus **placeholder data** modelled on the observed real structure. When the real input arrives, only the mapping in `config.py` (and possibly the parse function) changes.

### 7.1 Canonical record

Every adapter returns a list of dictionaries in this canonical form. `normalize_record(raw)` validates one record and raises `ValueError` with the field name if it is invalid; invalid records are skipped and counted, never crash the import.

| Field | Type | Required | Notes |
|---|---|---|---|
| `source` | str | yes | `'unisport'`, `'uniclubs'`, `'city'` |
| `external_id` | str | yes | Stable ID of the series in the source; if missing, built from title + weekday + start time + location |
| `title` | str | yes | |
| `description` | str | no | |
| `organizer` | str | no | Club name, `'Unisport HSG'`, venue organiser |
| `club_name`, `club_category` | str | no | UniClubs only |
| `event_type` | str | no | UniClubs: Talk, Workshop, Social, Career, Trip, Competition, … |
| `sport`, `level` | str | no | Unisport only |
| `start` | datetime | yes | Naive Europe/Zurich |
| `end` | datetime or None | no | None → estimated (§7.6) |
| `location` | str | no | Raw text as delivered |
| `source_url`, `registration_url` | str | no | |
| `price` | `Decimal` or None | no | From strings like `'25'`, `'CHF 25.00'`, `'free'` |
| `members_only` | bool | no | |
| `status` | str | no | Lower-case; default `'open'` |
| `is_recurring` | bool | no | True for weekly Unisport courses |

### 7.2 Unisport HSG – Excel adapter  [DEFAULT D-01]

* Input: `data/sources/unisport.xlsx` if present, else `unisport_placeholder.xlsx` (generated, §17). Read with `pd.read_excel(path)` (openpyxl engine, never imported directly), then `df.rename(columns=UNISPORT_COLUMN_MAPPING)`, `dropna` on required columns, `astype(str)` where needed (Course: A08).
* Target columns after renaming (the placeholder file uses exactly these names, so its mapping is the identity):

| Target column | Meaning | Example |
|---|---|---|
| `course_id` | optional stable ID | `VB-INT-THU-1830` |
| `sport` | activity | `Volleyball` |
| `title` | course name | `Volleyball Intermediate` |
| `level` | optional | `Intermediate` |
| `weekday` | for weekly courses: English or German name/abbreviation | `Thursday` / `Do` |
| `date` | for one-off sessions instead of weekday | `2026-11-14` |
| `start_time`, `end_time` | `HH:MM` | `18:30`, `20:00` |
| `location` | | `HSG Sports Hall` |
| `valid_from`, `valid_to` | optional date range of a weekly course | `2026-09-14`, `2026-12-20` |
| `registration_url`, `description`, `price_chf` | optional | |

* `UNISPORT_COLUMN_MAPPING` maps the real Excel headers to these names. When the real file arrives, the team (or agent) edits only this dictionary; `python importers.py` then prints a report: rows read, rows skipped with reasons, series and events created.
* Weekly courses are expanded with `expand_weekly_course()` into one event per week between `valid_from` (default: start of `HISTORY_WEEKS` before today) and `valid_to` (default `UNISPORT_DEFAULT_VALID_TO`, [DEFAULT D-36]). [DECIDED] Weekly courses appear every week. Weekday parsing uses `WEEKDAY_NAMES` (`'monday'`, `'mon'`, `'mo'`, `'montag'` → 0, …).
* Direction is always `sports`; the bundle comes from §8.3.

### 7.3 UniClubs – API adapter  [DEFAULT D-02]

* Credentials and URL live in the git-ignored `data/private/sources.json` (template `sources.example.json` committed):
  `{"uniclubs": {"url": "", "token": ""}, "city": {"url": "", "token": ""}}`. Empty URL → no live call.
* `fetch_json(url, token)`: `requests.get(url, headers={'Authorization': f'Bearer {token}'}, timeout=HTTP_TIMEOUT_SECONDS)` (Course: A06), `response.raise_for_status()`, `response.json()`.
* Assumed payload (to be confirmed when access arrives): either a list of events or `{"events": [...]}`; each event has `id`, `title`, `club` (`{"name", "category"}`) or `club_name`, `start` (ISO string), `end` (ISO or null), `location`, `description`, `type`, `status`, `price`, `members_only`, `url`, `registration_url`. `UNICLUBS_FIELD_MAPPING` maps payload keys to canonical fields; dates are parsed with `dateutil.parser.parse` and converted with `calendar_import.to_local_naive()`.
* Observed real structure (field notes, 1 Nov 2026): 72 clubs; club categories include Academic, Arts, Business, Cultural, Social, Sports; event types Talk, Workshop, Social, Career, Trip, Competition; statuses OPEN, CLOSED, FULL, APPLY, Members Only, ONGOING; usually **no end time**; locations from HSG rooms to Zurich, Munich, Prague, "Online" and "TBA / Details closer to the event".
* Display rules [DECIDED]: only St.Gallen and online events go into gaps; trips and events elsewhere go to *Beyond your gaps*; `full` and `closed` are hidden; members-only and paid events carry a tag.
* Status mapping: `'Members Only'` → `'members_only'`, others lower-cased.

### 7.4 City of St.Gallen events – API adapter  [DEFAULT D-03]

Same pattern as UniClubs with `CITY_FIELD_MAPPING` and `parse_city_payload()`; categorised with the same taxonomy rules. Placeholder `city_placeholder.json` until the API details arrive.

### 7.5 Location classification

`classify_location(location_text)` returns one of four types, checking in this order (case-insensitive, keyword lists in `LOCATION_KEYWORDS`):

1. `online` – contains *online, zoom, teams, webinar, virtual*.
2. `tba` – empty, or contains *tba, tbd, details closer*.
3. `elsewhere` – contains a known other city: *zurich, zürich, basel, bern, geneva, genève, lucerne, luzern, lausanne, winterthur, munich, münchen, prague, vienna, wien, milan, paris, london, berlin, amsterdam, barcelona*.
4. `st_gallen` – everything else. [DEFAULT D-17] All three sources are local to St.Gallen, so an unknown venue name (a bar, a room code) is most likely in St.Gallen. TBA events are placed into gaps with the tag *Location TBA*.

### 7.6 Import pipeline

`import_source(connection, source, taxonomy, now)` for each source, in this order of origins:

1. **Live** (only if a URL is configured, or the real Excel file exists): fetch → parse. On success, save the raw payload to `data/snapshots/<source>.json` (UniClubs, city).
2. **Snapshot**: if live failed or is not configured and a snapshot exists → parse the snapshot.
3. **Placeholder**: otherwise parse the placeholder file.

Rules for every import:

* All network calls inside `try/except (requests.RequestException, ValueError)`; on failure log `outcome='failed'` with a short message and continue with the next origin.
* **Sanity checks** [COURSE RULE]: if a source returns more than `MAX_RECORDS_PER_SOURCE` records, or a live source returns zero records, the import is **rejected** (`outcome='rejected'`) and the existing data stays untouched.
* **Missing end time** [DECIDED]: `estimate_end(start, event_type)` returns `start + DEFAULT_DURATION_MINUTES` (90 min) and the event is stored with `end_is_estimated = 1`, shown as tag *Estimated end* and explained in the exported calendar description. [DEFAULT D-07] If `USE_TYPE_DURATIONS` is `True`, `DURATION_BY_TYPE_MINUTES` is used instead (documented alternative, off by default).
* Records whose event has already ended are skipped (`include_past=False`), except during the initial demo build (`include_past=True`, §17).
* Upsert: series by `(source, external_id)`, events by `(series_id, start_time)`. Clubs are upserted by name.
* Future events of a source that are no longer delivered and have no interactions or invitations are deleted (cancelled by the organiser).
* Category assignment through `categories.assign_category()` (§8.3).
* Each run writes one row to `import_log`.

---

## 8 Categories: directions and bundles (`categories.py`)

[DECIDED] Two levels: fixed directions (Sports, Leisure, Career) and bundles underneath. The taxonomy is generated **once** by Claude during development and then **frozen**, because regenerating it would break favorites, statistics and ML features. No runtime LLM calls.

### 8.1 Directions and colours

[DECIDED] One colour per direction; the bundle appears as a label. Colours never carry meaning alone – the direction name is always written next to it.

| Direction key | Name | Streamlit colour | Hex (matplotlib) |
|---|---|---|---|
| `sports` | Sports | `blue` | `#1f77b4` |
| `leisure` | Leisure | `orange` | `#ff7f0e` |
| `career` | Career | `violet` | `#9467bd` |
| (busy) | Busy | `gray` | `#7f7f7f` |

In the UI, colours are applied only with built-in Streamlit features: `st.badge(label, color=...)` and colored Markdown (`:blue[text]`, `:blue-background[text]`). No custom CSS.

### 8.2 Taxonomy v1 (to be written to `data/taxonomy.json`)

This first version was produced from the observed UniClubs structure and a typical Unisport programme. [DEFAULT D-05] *Finance & investing* is a bundle under **Career** (resolves the unclear "Sport / ? / Finance" note). Keywords are lower-case; matching rules in §8.3.

| Bundle key | Name | Keywords (initial list) |
|---|---|---|
| **Sports** | | |
| `sports_ball` | Ball & team sports | football, soccer, futsal, basketball, volleyball, beach volleyball, handball, floorball, unihockey, uhu, rugby, hockey, ultimate, frisbee, baseball |
| `sports_racket` | Racket sports | tennis, badminton, squash, table tennis, padel, ping pong |
| `sports_endurance` | Running & endurance | running, run, jogging, marathon, cycling, spinning, rowing, triathlon, swimming, swim, endurance |
| `sports_fitness` | Fitness & strength | fitness, gym, unigym, strength, functional, bootcamp, hiit, circuit, weightlifting, bodyweight, core |
| `sports_mindbody` | Yoga & mind-body | yoga, pilates, meditation, stretching, mobility, tai chi, qigong, breathwork |
| `sports_dance` | Dance & cheer | dance, dancing, salsa, bachata, zumba, hip hop, ballet, cheerleading, cheer, swing |
| `sports_combat` | Combat & self-defence | boxing, kickboxing, judo, karate, taekwondo, self-defence, self-defense, krav maga, wrestling, fencing |
| `sports_outdoor` | Outdoor & climbing | climbing, bouldering, hiking, ski, skiing, snowboard, outdoor, trail, kayak, sailing |
| `sports_other` | Other sports | (none – fallback) |
| **Leisure** | | |
| `leisure_music_arts` | Music & arts | concert, music, choir, band, orchestra, jazz, theatre, theater, art, painting, photography, film, cinema, open mic |
| `leisure_culture` | Culture & languages | culture, cultural, language, tandem, international, museum, exhibition, literature, reading |
| `leisure_social` | Socials & parties | social, party, karaoke, raclette, fondue, drinks, apero, apéro, dinner, brunch, bbq, mixer, pub, quiz night |
| `leisure_games` | Games & gaming | gaming, esports, board game, poker, chess, video game, lan |
| `leisure_trips` | Trips & excursions | trip, excursion, tour, weekend trip, journey |
| `leisure_community` | Volunteering & community | volunteer, volunteering, charity, sustainability, community, fundraising, environment |
| `leisure_other` | Other leisure | (none – fallback) |
| **Career** | | |
| `career_finance` | Finance & investing | finance, investment, investing, hedge fund, asset management, banking, private equity, trading, stocks, valuation, portfolio, wealth |
| `career_consulting` | Consulting & strategy | consulting, consultant, case, case study, strategy |
| `career_tech` | Tech & data | tech, technology, data, ai, artificial intelligence, coding, programming, software, python, analytics, blockchain |
| `career_entrepreneurship` | Entrepreneurship & startups | startup, start-up, founder, entrepreneur, entrepreneurship, venture, pitch, innovation |
| `career_networking` | Company events & networking | company presentation, networking, recruiting, career fair, career, alumni, panel, fireside chat |
| `career_skills` | Skills & workshops | workshop, cv, interview, public speaking, negotiation, leadership, debate, rhetoric, excel |
| `career_other` | Other career | (none – fallback) |

`taxonomy.json` format:

```json
{
  "_meta": {"version": 1, "frozen": true, "ai_citation": "Generated by Anthropic Claude for the WeekWise spec, 2026-11-01; reviewed by the team: pending"},
  "directions": [{"key": "sports", "name": "Sports", "sort_order": 1}, "..."],
  "bundles": [{"key": "sports_ball", "direction": "sports", "name": "Ball & team sports",
               "is_other": false, "sort_order": 1, "keywords": ["football", "..."]}, "..."],
  "club_category_to_direction": {"Sports": "sports", "Business": "career", "Academic": "career",
                                 "Arts": "leisure", "Cultural": "leisure", "Social": "leisure"},
  "event_type_to_direction": {"Career": "career", "Social": "leisure", "Trip": "leisure"},
  "event_type_to_bundle": {"Trip": "leisure_trips"}
}
```

### 8.3 Assigning events to bundles  [DEFAULT D-06]

`assign_category(record, taxonomy)` returns `(direction_key, bundle_key)` deterministically:

1. **Direction.** Unisport → `sports`. Otherwise, the first rule that applies: `event_type_to_direction[event_type]`; `club_category_to_direction[club_category]`; else the direction of the best-scoring bundle over *all* bundles (step 2); if every score is 0 → `leisure`.
2. **Bundle.** If `event_type_to_bundle` has an entry → that bundle. Otherwise compute `bundle_score()` for each non-*Other* bundle of the direction: a keyword found in the title scores **2**, found in the description scores **1**; sum over keywords. Highest score wins; ties go to the lower `sort_order`; score 0 → the direction's *Other* bundle.
3. **Keyword matching** (`keyword_matches`): text is lower-cased and split into words on every character that is not a letter or digit (`normalize_text`). A one-word keyword matches a whole word, also with a plural *s* or *es* (so `ai` does not match "raclette" and `concert` matches "concerts"). A multi-word keyword matches if it appears in the normalised text with single spaces.

Every rule above gets a unit test (§18).

### 8.4 Freezing rules

* `validate_taxonomy()` checks: three directions; every bundle points to an existing direction; exactly one *Other* bundle per direction; keys unique.
* After the freeze, bundle **keys are never renamed or deleted**; new bundles may only be appended. The `taxonomy_version` setting is compared with `_meta.version` on start; a mismatch shows a warning in Settings and offers *Reset demo data*.
* When real data (Excel, APIs) arrive, the team may have Claude review the keyword lists **once** against all real events before the final freeze (recommended in milestone M8). This is recorded in `AI_USAGE.md`.

---

## 9 Calendar import (`calendar_import.py`)

[DECIDED] WeekWise reads the student's main calendar without registration or OAuth.

### 9.1 Modes

| Mode | Input | Notes |
|---|---|---|
| `demo` (default) | `data/demo/demo_calendar.ics` (§17) | Works without any account; used for the video. |
| `link` | Secret iCal address pasted once | Google: *Settings → the calendar → Secret address in iCal format*. Outlook: *Settings → Calendar → Shared calendars → Publish a calendar → ICS link*. Some university Outlook accounts have publishing disabled – then use `upload`. |
| `upload` | Exported `.ics` file (`st.file_uploader(type=['ics'])`) | |

[DEFAULT D-28] The link is kept only in `st.session_state` unless the user ticks *Remember on this computer*; then it is written to the git-ignored `PRIVATE_CALENDAR_LINK_FILE`. Uploaded files are remembered the same way in `PRIVATE_CALENDAR_UPLOAD_FILE`. If a new session starts in `link` mode without a stored link, the app falls back to `demo` and says so.

### 9.2 Pipeline

`load_calendar(mode, range_start, range_end, link=None, upload_bytes=None)` returns `(busy_blocks, all_day_labels, weekwise_event_ids, used_mode, message)`:

1. **Get text.** `link`: `normalize_calendar_link()` turns `webcal://` into `https://` and rejects anything not starting with `https://`; `fetch_ics_text()` uses `requests.get(link, timeout=HTTP_TIMEOUT_SECONDS)` (Course: A06), checks status 200, size ≤ `MAX_CALENDAR_BYTES` and that the text contains `BEGIN:VCALENDAR`. `upload`/`demo`: decode bytes/read file as UTF-8.
2. **Parse.** `icalendar.Calendar.from_ical(text)`; expand recurring entries (RRULE, exceptions) with `recurring_ical_events.of(calendar).between(range_start, range_end)`.
3. **Per component:**
   * Skip `STATUS:CANCELLED` and `TRANSP:TRANSPARENT` (marked as free).
   * `to_local_naive()`: aware datetimes → `astimezone(ZoneInfo(TIMEZONE_NAME))` → drop tzinfo; naive (floating) datetimes are taken as local time.
   * **All-day entries** (`DTSTART` is a `date`): [DEFAULT D-16] not busy by default – they become day labels (e.g. a birthday) shown under the day header. Setting *Treat all-day entries as busy* (`all_day_busy`) turns them into busy blocks covering the whole day window.
   * Missing `DTEND`: use `DURATION` if present; otherwise skip the entry (zero length).
   * **WeekWise's own events:** `extract_weekwise_event_id(uid, description)` recognises the UID `weekwise-<event_id>-<YYYYMMDD>@weekwise` or, for entries created through the Google/Outlook links (which cannot carry a UID), the description line `WeekWise ID: weekwise-<event_id>-<YYYYMMDD>@weekwise` [DEFAULT D-33]. Recognised entries are **not** returned as busy blocks (the confirmed event is drawn from the database) and are not duplicated. If the ID is unknown to the database (e.g. after a reset), the entry is treated as a normal busy block titled *WeekWise event (not in this database)*.
4. **Reduce** everything to `BusyBlock` objects before any further logic (Course: lists of tuples / small classes).
5. **Errors.** Any exception in steps 1–3 → fall back to the remembered upload, else to `demo`, and return a friendly `message` (e.g. *"Could not read your calendar link (timeout). Showing the demo calendar instead."*). Never include the link in messages.

### 9.3 Privacy  [DECIDED / SUGGESTION adopted]

* Busy blocks are never written to the database. Titles are kept in memory only to draw the student's own week.
* Setting *Hide titles of my calendar entries* (`hide_busy_titles`) shows them as *Busy*.
* The link is displayed only masked (`https://calendar.google.com/…/basic.ics`), never logged, never committed.

---

## 10 Gap detection and placement (`gaps.py`)

### 10.1 Rules

| Rule | Value | Status |
|---|---|---|
| Minimum gap | 60 minutes without busy blocks | [DECIDED] |
| Overlap tolerance | A suggested event may overlap an existing entry by at most 5 minutes at its start or end. Example: class ends 10:00 → an event starting 09:55 is still suggested, one starting 09:54 is not. | [DECIDED] |
| Weekends | Included | [DECIDED] |
| Day window | Gaps are searched between 07:00 and 23:00 | [DEFAULT D-09] |
| What is busy | Timed calendar entries + the user's own **confirmed** WeekWise events (+ all-day entries only if the setting is on) | [DECIDED] / D-16 |
| Location | Only placeable events (§3) go into gaps | [DECIDED] |
| Past | Only events that start after `now` are suggested | [DECIDED] |
| Horizon | As far into the future as event data exists | [DECIDED] |

[DEFAULT D-09] The planning proposed 07:00–22:00 (unconfirmed). This spec uses **23:00** as the end because many club socials and city events run into the late evening; with a 22:00 limit a 20:00–22:30 karaoke night could never be suggested. The team can change one constant.

### 10.2 Algorithm

`find_gaps_for_day(day, busy_intervals, window_start=DAY_WINDOW_START, window_end=DAY_WINDOW_END, min_gap_minutes=MIN_GAP_MINUTES)`:

```
initialization phase
    window = (day + window_start, day + window_end)
    clip every busy interval to the window; drop intervals that end up empty
    sort the clipped intervals by start time                       # Course: U03 s3 sort()
    merged = merge_intervals(clipped)   # overlapping or touching intervals become one
processing phase
    cursor = window start
    for (start, end) in merged:
        if start - cursor >= min_gap: add Gap(cursor, start)
        cursor = max(cursor, end)
    if window end - cursor >= min_gap: add Gap(cursor, window end)
termination phase
    return the list of gaps
```

`assign_events_to_gaps(events, gaps, now)` returns `(placed, beyond)`:

* Events with a hidden status (`full`, `closed`) and events whose ID is in `confirmed_ids` (a set; they are already busy blocks) are skipped entirely.
* `placed`: dictionary `gap_id → list of events` – an event is added to the **first** gap it fits (`Gap.fits()`), if it is placeable and starts after `now`.
* `beyond`: list of `(event, reason)` for events in the visible week that start after `now`, are not hidden by status, but cannot be placed. `placement_problem(event)` returns the reason, or `None` if the event is placeable: `'Off-site: Zurich'` (uses the location text), `'Trip'`, `'Multi-day'`, `'Outside your day window'` (starts before 07:00 or ends after 23:00 + tolerance). Events that are placeable but simply collide with a busy block are **not** listed (that would flood the list).

**"Competing suggestions disappear"** [DECIDED] follows automatically: after a confirmation the event becomes a busy block, the gaps are recomputed, and every suggestion overlapping the confirmed event vanishes; suggestions in the remaining free part of the former gap (if it is still ≥ 60 min) stay.

`navigation_bounds(first_date, last_date)` returns the Monday of the first and of the last week that contains events; the week navigation cannot leave this range. Default week: the current week.

### 10.3 Worked example (becomes a unit test)

Thursday, busy: 08:15–10:00 lecture, 10:15–12:00 lecture, 14:00–18:00 part-time job. Window 07:00–23:00.

* Gaps: 07:00–08:15 (75 min ✓), 10:00–10:15 (15 min ✗), 12:00–14:00 (120 min ✓), 18:00–23:00 (300 min ✓).
* Event 12:15–13:15 → fits gap 12:00–14:00. Event 13:00–14:05 → fits (5 min tolerance). Event 13:00–14:06 → does not fit. Event 17:55–19:25 → fits gap 18:00–23:00 (starts 5 min early). Event 17:54–19:00 → does not fit.
* Confirm 18:30–20:00 → new gaps 18:00–18:30 (✗, 30 min) and 20:00–23:00 (✓); an event 19:00–20:30 disappears, an event 20:15–21:45 stays.

---

## 11 Calendar export (`calendar_export.py`)

[DECIDED] A confirmed event is added to the student's calendar through a generated entry, not through an API. No sign-up inside WeekWise; if registration is needed, WeekWise only shows the external link. Confirming a weekly Unisport course adds **only that single occurrence**.

### 11.1 Delivery options per confirmed event

1. **Download .ics** – `st.download_button(label, data=build_ics(event), file_name=f'weekwise-{event_id}.ics', mime='text/calendar')`. Works with Google, Outlook and Apple Calendar.
2. **Add to Google Calendar** – `st.link_button` to `google_calendar_link(event)`.
3. **Add to Outlook** – `st.link_button` to `outlook_calendar_link(event)` (Microsoft 365 / HSG accounts; `outlook.live.com` for private accounts can be set in `config.py`).
4. **Register / original page** – `st.link_button` to `registration_url` (if any) and `source_url`.

The two link endpoints are public "compose event" URLs, not official APIs; the `.ics` download is the reliable path and is listed first.

### 11.2 The .ics file (plain text with f-strings)

[COURSE RULE] Written with f-strings in one small, well-commented function – no library. Lines end with `\r\n`.

```
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//WeekWise//HSG FCS project//EN
CALSCALE:GREGORIAN
METHOD:PUBLISH                       (CANCEL for a cancellation)
BEGIN:VEVENT
UID:weekwise-42-20261105@weekwise
DTSTAMP:20261101T120000Z
DTSTART:20261105T173000Z             (UTC; converted from Europe/Zurich with zoneinfo)
DTEND:20261105T190000Z
SUMMARY:Volleyball Intermediate
LOCATION:HSG Sports Hall
DESCRIPTION:<build_description(event)>
URL:https://…
SEQUENCE:0                           (1 for a cancellation)
STATUS:CONFIRMED                     (CANCELLED for a cancellation)
END:VEVENT
END:VCALENDAR
```

* **UID** [DECIDED]: `make_uid(event)` → `f'weekwise-{event.event_id}-{event.start:%Y%m%d}@{WEEKWISE_UID_DOMAIN}'` – the hidden marker by which WeekWise recognises its own entries.
* **Escaping** (`escape_ics_text`): backslash → `\\`, `;` → `\;`, `,` → `\,`, newline → `\n` (order matters: backslash first).
* **Folding** (`fold_ics_line`): lines longer than 75 bytes are split; continuation lines start with one space. Count bytes of the UTF-8 encoding character by character in a simple loop (umlauts take 2 bytes).
* **Description** (`build_description`): the event description (shortened to 500 characters), source label, registration link, original page, then – if `end_is_estimated` – *"End time estimated by WeekWise (default 1.5 h) – please check the original page."*, then the marker line `WeekWise ID: <uid>`.

### 11.3 Add-to-calendar links

Built with `urllib.parse.urlencode` (standard library, Tier B).

* Google: `GOOGLE_CALENDAR_URL + '?' + urlencode({'action': 'TEMPLATE', 'text': title, 'dates': f'{format_utc(start)}/{format_utc(end)}', 'details': description, 'location': location})`.
* Outlook: `OUTLOOK_CALENDAR_URL + '?' + urlencode({'path': '/calendar/action/compose', 'rru': 'addevent', 'subject': title, 'startdt': start_iso, 'enddt': end_iso, 'body': description, 'location': location})`, where the ISO strings carry the Europe/Zurich offset (e.g. `2026-11-05T18:30:00+01:00`).
* Both descriptions contain the marker line (§9.2).

### 11.4 Cancelling  [DEFAULT D-08]

[DECIDED] A confirmed future event can be cancelled in WeekWise. Because export is file/link-based, WeekWise cannot delete the entry from the real calendar. Handling:

1. `save_interaction(..., action='cancel')` → status `cancelled`; the slot is free again and suggestions reappear.
2. Offer **Download cancellation .ics** (`build_ics(event, cancel=True)`: same UID, `METHOD:CANCEL`, `STATUS:CANCELLED`, `SEQUENCE:1`). Some calendar apps remove the entry when it is imported.
3. Show: *"If the event is still in your calendar, delete it there manually."*

---

## 12 Recommender – own machine-learning model (`recommender.py`)

[DECIDED] A Random Forest classifier (scikit-learn) predicts whether a student will rate an activity **4 stars or more**. Its predicted probability ranks the suggestions in each gap. Training data: synthetic users' feedback + the local user's questionnaire, favorites and feedback. *Not attended* (✗) is excluded. Suggestions appear instantly; the model retrains after each new rating. [COURSE RULE] The workflow follows the lecture 02 ML intro and Assignments 10–11.

### 12.1 Label and training rows

* One training row = one interaction with `status = 'attended'` **and** a rating (any user). `make_label(rating)` → `1` if `rating >= LABEL_MIN_STARS` else `0`.
* Excluded: `confirmed` (no answer yet), `cancelled`, `not_attended` [DECIDED].
* Sample weights [DEFAULT D-23]: the local user's own ratings (`is_synthetic = 0` and `is_demo = 0`) get `REAL_USER_SAMPLE_WEIGHT` (5.0); synthetic users' ratings and the demo history get 1.0. Passed to `fit(..., sample_weight=weights)`.

### 12.2 Features

`event_feature_row(event, profile, friends_going, earlier_avg_rating)` returns one dictionary (one row):

| Group | Feature | Type |
|---|---|---|
| Event | `direction`, `bundle`, `source`, `location_type`, `time_of_day` (`morning` < 12:00 ≤ `afternoon` < 17:00 ≤ `evening`) | categorical → one-hot |
| Event | `weekday` (0–6), `is_weekend`, `start_hour`, `duration_minutes`, `is_free`, `price_chf`, `members_only`, `is_recurring`, `end_is_estimated` | numeric / 0-1 |
| User × event | `wants_direction` (event direction in the user's questionnaire), `fav_series`, `fav_club`, `fav_bundle` (0/1 each), `earlier_avg_rating_in_bundle` | numeric |
| Social | `friends_going` | numeric |

* `earlier_avg_rating_in_bundle` [DEFAULT D-31]: the user's average rating of events in the same bundle that **started before this event**, or `NEUTRAL_RATING` (3.0) if there are none. For training rows it is computed chronologically in a plain loop over the user's ratings sorted by start time – using the row's own rating would leak the answer into the features. For scoring it uses all ratings before `now`.
* `to_feature_matrix(rows, feature_columns=None)`: `pd.DataFrame(rows)` → `pd.get_dummies(..., columns=[categorical columns])`. When scoring, the frame is aligned to the training columns with `reindex(columns=feature_columns, fill_value=0)` so unseen categories cannot break prediction.

### 12.3 Production model

`train_production_model(X, y, weights)` → `RandomForestClassifier(n_estimators=RF_N_ESTIMATORS, max_depth=RF_MAX_DEPTH, random_state=RANDOM_SEED)` fitted on **all** training rows with sample weights. No model files are written: training takes about a second and is cached with `st.cache_resource` (§12.6).

If fewer than `MIN_TRAINING_ROWS` rows exist or only one class is present, scoring falls back to `heuristic_score()` and *About the model* says so.

### 12.4 Scoring suggestions

`Recommender(model, feature_columns)` with `score_events(events, profile, friends_going_map, avg_rating_map)` (the maps come from `friends.friends_going_counts()` and `database.get_user_bundle_averages()`) → list of `Suggestion`, sorted by score (highest first):

1. `p = model.predict_proba(X)[:, 1]` (probability of a rating ≥ 4).
2. `favorite_boost(event, profile)` [DEFAULT D-22]: `FAVORITE_BOOST` (0.10) for each matching favorite kind (series, club, bundle), capped at `MAX_FAVORITE_BOOST` (0.25). [DECIDED] Favorites are prioritised *and* are model features.
3. `score = min(1.0, p + boost)`.
4. `reasons`: up to three short texts, e.g. *Favorite club*, *Matches your focus: Sports*, *2 friends going*, *You liked Racket sports before*.
5. **Invitations** are pinned: an event to which the user was invited (pending invitation) is placed first in its gap, marked *"Lena invited you"*, regardless of its score.

`heuristic_score(event, profile, friends_going)` (fallback and milestone M3 baseline): `0.40 + 0.30 × wants_direction + favorite_boost + min(0.10, 0.05 × friends_going)`, capped at 1.0.

Ranking per gap: `sorted(suggestions, key=lambda s: s.score, reverse=True)`; top 3 = `[:TOP_SUGGESTIONS_PER_GAP]`, dropdown = `[TOP_SUGGESTIONS_PER_GAP:]` (Course: A05, U03 s5).

### 12.5 Cold start

The local user starts with no own ratings. Because synthetic users' ratings depend strongly on `wants_direction` and favorites (§17.3), the model already ranks the user's chosen directions and favorites higher on day one. Every rating then adds a row with weight 5.

### 12.6 Retraining  [DECIDED, COURSE RULE sanity check]

* `ui_components.cached_recommender(n_ratings)` is decorated with `@st.cache_resource(max_entries=2)`; `n_ratings` is the total number of ratings in the database. A new rating changes the key → exactly **one** retrain; any other rerun reuses the cached model. This caps retraining to once per rating.
* After saving a rating the UI shows a toast: *"Thanks! Your suggestions were updated."*

### 12.7 Evaluation in Assignment 10–11 style

`evaluate_models(X, y)` (run when *About the model* is opened; cached per `n_ratings` with `st.cache_data` – [DEFAULT D-29], instead of at start-up, to keep the start fast):

1. Show the label distribution (share of ratings ≥ 4).
2. Stratified split with a fixed seed: `train_test_split(X, y, test_size=0.2, stratify=y, random_state=RANDOM_SEED)`, then the 20 % part split 50/50 again with `stratify` → **80 / 10 / 10** (Course: A11, seed 13).
3. `RobustScaler` fitted on the training data **only**, then applied to validation and test (Course: A11).
4. Five classifiers, each unscaled and scaled (10 rows): `RandomForestClassifier`, `LogisticRegression(max_iter=1000)`, `KNeighborsClassifier(n_neighbors=5)`, `DecisionTreeClassifier`, `GaussianNB`, all with `random_state=RANDOM_SEED` where available. Evaluation fits are unweighted, as in A11.
5. Metrics on train and validation: `accuracy_score` and `precision_recall_fscore_support(..., average='macro', zero_division=0)` → one pandas DataFrame with columns `model, scaled, train_accuracy, val_accuracy, val_precision, val_recall, val_f1`.
6. The **Random Forest** (production model, [DECIDED]) is reported on the **test** set with all four metrics and `classification_report`. If another model wins on validation F1, the page says so and explains that the team kept the Random Forest (handles mixed feature types without scaling, robust, gives feature importances); the comparison is the justification.
7. Charts: `plot_model_comparison()` – grouped bar chart of validation F1 per model, unscaled vs scaled; `plot_feature_importances()` – horizontal bar chart of the 10 most important features (`model.feature_importances_`).

---

## 13 Rankings (`rankings.py`)

[DECIDED] Anonymised statistics, no login. *All users* = synthetic users + the local user. Four metrics, each for *You* and for *All users*, overall and per category.

### 13.1 Metrics (grouped by series)

| Key | Label | Definition (SQL on `interactions` JOIN `events` JOIN `series`) | Minimum |
|---|---|---|---|
| `popular` | Most popular | `COUNT(*)` with `status IN ('confirmed', 'attended', 'not_attended')` | – |
| `attended` | Most attended | `COUNT(*)` with `status = 'attended'` | – |
| `best_rated` | Best rated | `AVG(rating)`, `HAVING COUNT(rating) >= ?` | 3 for All users, 1 for You [DEFAULT D-24] |
| `feedbacked` | Most feedbacked | `COUNT(rating)` | – |

* `get_ranking(connection, metric, scope, user_id, direction_key=None, bundle_key=None, top_n=RANKING_TOP_N)` builds the query from fixed SQL pieces chosen by dictionaries (never by inserting user text with f-strings): scope `'you'` adds `AND i.user_id = ?`; a direction or bundle filter adds `AND d.key = ?` / `AND b.key = ?`. `GROUP BY s.id ORDER BY value DESC, n DESC, s.title LIMIT ?` (Course: A07).
* Returns a DataFrame with `title, organizer, direction_key, bundle_name, value, n`.
* *All users* views show aggregates only – never user names.

### 13.2 Charts

* `plot_ranking(ranking_df, title)` → matplotlib `Figure`: horizontal bar chart (`barh`), bars coloured by direction hex colour, value labels, readable titles (Course: A09). Shown with `st.pyplot(fig)`.
* `category_overview(connection, user_id)` → per direction: number confirmed and average rating, for You and for All users. `plot_category_overview()` → grouped bar chart (two bars per direction).

---

## 14 Friends (`friends.py`)

[DECIDED] Page 3 is part of the core app, demonstrated with synthetic friends. All accepted elements:

| Element | Rule |
|---|---|
| Friend code | Each user has a 6-character code from `FRIEND_CODE_ALPHABET` (no 0/O, 1/I), generated with `random` and checked for uniqueness against a **set** of existing codes. |
| Share | [DEFAULT D-26] Because WeekWise runs locally, the "share link" is a copyable text shown with `st.code`: *"Add me on WeekWise – my friend code is LENA26."* |
| Add friend | Paste a code → `add_friend()`; no search, no profiles. Errors (raise `ValueError`, shown as `st.error`): unknown code, own code, already friends. Stores both directions. |
| Display name | Chosen by the user (onboarding, editable in Settings). |
| Friends going | Badge on suggestions, e.g. *2 friends going*; also an ML feature. Only friends with `sharing_opt_in = 1` are counted. |
| Invite | On a confirmed future event: choose friends (`st.multiselect`) → `send_invitations()`. For synthetic friends the answer is simulated (below). |
| Invited | Pending invitations to the local user appear as pinned suggestions *"Lena invited you"* in the week grid (if the event fits a gap) and always on the Friends page with **Accept** (= confirm, then export options) / **Decline**. |
| Privacy | [DECIDED] Sharing is opt-in (`sharing_opt_in`, default off for the local user, toggle on the Friends page). Only WeekWise events are ever shared, never private calendar entries. |

**Simulated responses** [DEFAULT D-27]: `simulate_friend_response()` answers immediately and deterministically: a synthetic friend **accepts** if the event's direction is among the friend's directions and the friend has no other confirmed event overlapping it; otherwise declines. Accepting creates a `confirmed` interaction for that friend, so *friends going* updates at once. Simulated answers are labelled *(simulated)* in the UI.

---

## 15 Own small Web API (`api_server.py`, `api_client.py`)

[DEFAULT D-04] The team asked for "our own small database, kinda going beyond the expected for extra credit". This spec implements it as a small local **Flask** API over the SQLite database (lecture 06: *use and create Web APIs*), with a direct-database fallback so graders can always start the app.

### 15.1 Server

* `python api_server.py` starts Flask on `API_HOST:API_PORT` (127.0.0.1:5001, debug off). One decorated function per route; each route calls `database.py` functions – **no SQL in `api_server.py`**. Each request opens its own connection.
* JSON in, JSON out (`flask.jsonify`); errors as `{"error": "<message>"}` with the right status code.

| Method | Route | Purpose | Status codes |
|---|---|---|---|
| GET | `/health` | `{"status": "ok", "version": "1.0"}` | 200 |
| GET | `/categories` | Directions and bundles with colours | 200 |
| GET | `/events?from=YYYY-MM-DD&to=YYYY-MM-DD` | Events in the range (inclusive from, exclusive to) with series and category fields; optional filters `direction`, `bundle`, `source` | 200; 400 for missing/invalid dates, `to <= from`, or a range > `MAX_API_RANGE_DAYS` |
| GET | `/events/<int:event_id>` | One event | 200; 404 |
| POST | `/interactions` | Body `{"user_id": 1, "event_id": 42, "action": "confirm", "rating": null}` → `database.save_interaction()` | 201 with the stored interaction; 400 invalid body or transition (`ValueError` message); 404 unknown event/user |

Event JSON (one element of `/events`): exactly the columns of `database.get_events_df()` (§6.3) plus `id` – `id, series_id, start_time, end_time, end_is_estimated, status, source, title, description, organizer, event_type, sport, level, location, location_type, source_url, registration_url, price_rappen, members_only, is_recurring, club_id, club_name, club_category, bundle_key, bundle_name, direction_key`. Times are text `'YYYY-MM-DD HH:MM:SS'` (Europe/Zurich, §4.5), prices integer Rappen (no floats in JSON). Because the API and the database deliver the **same row format**, `models.event_from_row(row)` has to understand only one format, and the fallback behaves identically.

### 15.2 Client

* `is_api_available()`: `GET /health` with `timeout=API_TIMEOUT_SECONDS`; the UI caches the result for 30 s.
* `get_events(start, end)`: if available → `requests.get(f'{API_BASE_URL}/events', params={...}, timeout=...)` → list of `Event` via `event_from_row()`, origin `'api'`. On any `requests.RequestException` or non-200 → `database.get_events_df()` → the same rows → `event_from_row()`, origin `'database'`.
* `save_interaction(...)`: same pattern with `POST /interactions`; on fallback calls `database.save_interaction()` directly. Both paths use the same database function, so behaviour is identical.
* The sidebar shows *Event data: own API ✓* or *Event data: database (API not running)*.
* Two processes can share the SQLite file; `timeout=10` on connect avoids "database is locked" under light load.

---

## 16 User interface (Streamlit)

### 16.1 Global shell

* `layout='wide'`, title *WeekWise*, icon `:material/calendar_month:`. Navigation with `st.navigation([...], position='sidebar')` and `st.Page('views/week.py', title='Week', icon=':material/calendar_view_week:', default=True)`, *Rankings* (`:material/leaderboard:`), *Friends* (`:material/group:`), *Settings & About* (`:material/settings:`). Icons are Streamlit's built-in Material icons, not emoji, so they look the same on every system. Before onboarding: only `st.Page('views/onboarding.py', title='Welcome')`.
* **Sidebar status** (`render_sidebar_status()`, every page): current user's display name; *Calendar: Demo / Your link / Uploaded file*; *Event data: own API ✓ / database*; when placeholder sources or synthetic users are in use, a gray badge **Demo data** with the help text *"Placeholder events and 60 synthetic users illustrate the app until real data arrive."* [COURSE RULE] Demo data must be visibly marked.
* **Session-state keys** (all prefixed `ww_`): `ww_week_start`, `ww_open_event_id`, `ww_popup_done`, `ww_session_init_done`, `ww_calendar_link`, `ww_calendar_upload`, `ww_onboarding_step`, `ww_filters`, `ww_layout`.
* **Dialog pattern.** Buttons and dropdowns never open a dialog directly inside a loop. They set `st.session_state['ww_open_event_id']`; at the end of the page script, if the key is set, `show_event_dialog(event_id)` is called once and the key is cleared. Only one dialog can be open per script run, so the feedback pop-up (§16.5) runs only on the first run of a session.
* **Widget keys** are unique and readable: `f'details_{event_id}_{gap_id}'`, `f'attended_yes_{interaction_id}'`, `f'stars_{interaction_id}'`.
* After every database write: `st.rerun()` (or a toast followed by rerun) so the grid always reflects the database.

### 16.2 Onboarding (`views/onboarding.py`)

Three steps in one page, tracked by `ww_onboarding_step`, with *Back* / *Next* buttons and a progress line *Step 1 of 3*.

| Step | Content | Validation |
|---|---|---|
| 1 Welcome | App name, problem statement (§1.1), three-line "How it works" (find gaps → get suggestions → rate). | – |
| 2 Your focus | [DECIDED] Only: *Which directions do you want to focus on first?* – Sports, Leisure, Career (three checkboxes or `st.pills(selection_mode='multi')`, each with its colour and a one-line example). Display name (`st.text_input`). Checkbox *Load demo history* (default **on**, [DEFAULT D-25]): adds a few past confirmed and rated events for you so the feedback features can be shown immediately; removable in Settings. | At least one direction; name 2–20 visible characters. |
| 3 Your calendar | `st.radio`: *Demo calendar (recommended to try WeekWise)* / *My calendar link (Google, Outlook)* / *Upload an .ics file*. Link: `st.text_input(type='password')`, instructions for Google and Outlook in `st.expander`, button *Test link* → shows *"Found 37 entries in the next 14 days"* or the error. Upload: `st.file_uploader(type=['ics'])`. Checkbox *Remember on this computer* (link/upload only). | Link must pass the test before *Finish*; upload must parse. |

*Finish*: `create_user()` (friend code generated, `is_synthetic=0`, `sharing_opt_in=0`), `set_user_directions()`, settings `current_user_id`, `calendar_mode`, `onboarding_done`; if *Load demo history*: `synthetic_data.create_demo_history()` and `create_showcase_friends()`; then `st.rerun()` → Week page.

### 16.3 Week page (`views/week.py`)

**Header row:** `◀ Previous week` · `Today` · `Next week ▶` buttons (disabled outside `navigation_bounds()`), title *Week 45 · 2 – 8 November 2026*, legend with badges *Sports*, *Leisure*, *Career*, *Busy*. One status line if the calendar fell back to demo (message from `load_calendar`).

**Sidebar filters** [DECIDED]:

* `:material/star: Favorites` checkbox.
* For each direction an `st.expander` with its coloured name; inside, one checkbox per bundle listed vertically (when a direction has more than `FILTER_LIST_MAX` bundles, an `st.multiselect` replaces the checkboxes – "depending on how many categories there are").
* *Clear filters* button. Layout toggle `st.segmented_control('Layout', ['Week', 'Day'])` [DEFAULT D-32]: *Week* = 7 columns; *Day* = a day selector plus one wide column (for narrow screens – Streamlit cannot detect screen width, so the user chooses).
* Semantics: **no filter selected** → smart mode: per gap the ML top 3 plus a dropdown with the rest. **Any filter selected** → filter mode: per gap **all** fitting events that match *any* selected filter (favorites OR the chosen bundles), sorted by score, each with a link to its original page; capped at `MAX_EVENTS_PER_GAP_FILTER_VIEW` with a note *"+ 12 more – narrow your filter"*.

**Grid** [SUGGESTION adopted, planning §11.1]: built only from `st.columns(7)` and `st.container(border=True)`. Each day column shows the header (`**Thu** 05.11.`, *Today* badge for today), all-day labels as captions, then all items in chronological order. Every item shows its start–end time, so exact proportional heights are not needed ([DEFAULT D-30]; proportional heights for busy blocks are [STRETCH]).

| Item | Appearance | Actions |
|---|---|---|
| Busy block | Gray container: `:gray[10:15–12:00]` + title (or *Busy*) | – |
| Gap (future) | Container with caption `Gap.label()` | Contains suggestion cards |
| Suggestion card | Direction badge + bundle name, **title** (max 45 characters, then …), time, organizer/source, tags (`display_tags()`), *Match 82 %*, reasons as caption, *2 friends going* badge, *Lena invited you* badge for pinned invitations | `Details` button |
| More options | `st.selectbox(f'More options ({n})', ...)` under the top 3, format `'18:30 · Salsa · 64 %'` | Selecting an item opens its details |
| Confirmed (future) | Container with direction badge and **✓ Confirmed** | `Details` (export, cancel, invite) |
| Confirmed (past, no answer) | Text *Attended?* and two buttons **✓** (green) / **✗** (red) | ✓ → `attended` → stars appear; ✗ → `not_attended` |
| Attended, no rating | `st.feedback('stars', key=...)` – returns 0–4, **stored rating = value + 1** | Saving a rating triggers one retrain (§12.6) |
| Rated | `★★★★☆ You rated 4/5` (can be changed in Details) | `Details` |
| Not attended | Caption *Not attended* | – |

Past gaps and past unconfirmed suggestions are not shown [DECIDED].

**Beyond your gaps** [DECIDED list; placement adopted from the planning]: an `st.expander('Beyond your gaps (n)')` below the grid lists the visible week's off-site events, trips, multi-day events and events outside the day window, each with reason tag, date, and `Details`. They can be confirmed and exported like any event, with the warning *"WeekWise did not check this against your calendar."*

**Empty states:** no events in this week → *"No activities found for this week. Try the next week or refresh the event data in Settings."* No gaps → *"Your week is full – no free gaps of 60 minutes or more."*

### 16.4 Event details dialog (`show_event_dialog`, `@st.dialog('Event details', width='large')`)

Shows: title, direction badge + bundle, source label, date and time (with *Estimated end* explanation if flagged), location (+ type), price, tags, description, friends going (names of friends who opted in), match score and reasons.

Actions by state:

| State | Actions |
|---|---|
| Suggestion (future) | **Confirm** (primary). After confirming the dialog shows the export block: *Download .ics*, *Add to Google Calendar*, *Add to Outlook*, *Register* (if `registration_url`), *Original page*, and the hint *"Only this date is added – weekly courses are not added as a series."* |
| Confirmed (future) | Export block; **Invite friends** (`st.multiselect` of friends + *Send*); **Cancel this event** → cancellation .ics + manual-delete hint (§11.4). |
| Past | Feedback widget (same as inline); for rated events, change the rating. |
| Any | Favorite toggles: *☆ Favorite this course/series*, *☆ Favorite this club* (club events), *☆ Favorite category "Racket sports"* [DECIDED: three kinds]. |

### 16.5 Feedback pop-up (`show_feedback_dialog`)

[DECIDED] On app start (first script run of a session, after onboarding) WeekWise checks `get_pending_feedback()`. If there is pending feedback, a dialog *"How was it?"* lists up to 5 events (oldest first) with the same ✓/✗ → stars flow. Buttons: *Done* and *Later*. [DECIDED] The pop-up can be dismissed (*Later*, or closing it) but reappears at the next session start until everything is answered. `ww_popup_done` prevents it from reappearing within the same session.

### 16.6 Rankings page (`views/rankings.py`)

* Intro line: *"Anonymised statistics. All users = 60 synthetic demo users + you."*
* Category selector (`st.selectbox`): *All categories*, then each direction followed by its bundles (indented names).
* `st.tabs(['Most popular', 'Most attended', 'Best rated', 'Most feedbacked'])`; in each tab two columns *You* | *All users*, each with the chart from `plot_ranking()` and an `st.expander('Show table')` with `st.dataframe`. Empty case: *"No data yet – confirm and rate some activities."*
* Below the tabs: *Category overview* chart (`plot_category_overview()`).

### 16.7 Friends page (`views/friends.py`)

Two columns.

* **Left – You:** display name, friend code (`st.code`) with the share text, sharing toggle *Share my WeekWise plans with friends* (opt-in), *Add a friend* (text input + button; success/error message), `st.expander('Demo friend codes')` listing the three addable showcase codes (§17.5).
* **Right – Friends:** list of friends with their upcoming WeekWise plans (only friends who opted in; title, date, direction badge, *Details*); *Invitations received* (event, sender, date, *Fits your calendar* / *Overlaps with your calendar*, **Accept** / **Decline**); *Invitations sent* (event, friend, status, *(simulated)* label).

### 16.8 Settings & About page (`views/settings.py`)

`st.tabs(['Profile', 'Calendar', 'Data sources', 'About the model', 'About WeekWise'])`

| Tab | Content |
|---|---|
| Profile | Edit focus directions [DEFAULT D-11] and display name; *Clear my demo history* (deletes interactions with `is_demo = 1` after a confirmation checkbox). |
| Calendar | Change mode / link / upload (same widgets as onboarding step 3), *Remember on this computer*, *Forget my calendar link* (deletes the private file), *Treat all-day entries as busy*, *Hide titles of my calendar entries*. |
| Data sources | `import_log` as a table (source, origin live/snapshot/placeholder/excel, counts, outcome, time); per source a short explanation of what is placeholder; button *Refresh event data* → `import_all()`; *Reset demo data* (confirmation checkbox) → `database.reset_database()` drops all tables in reverse dependency order (safer than deleting the file, which Windows may lock), clears `st.session_state` and the caches, and reruns the first-start setup; the local user goes through onboarding again. Remembered calendar files in `data/private/` are kept unless forgotten in the Calendar tab. |
| About the model | Plain-language explanation (what is predicted, which features, how feedback retrains it); number of training rows (synthetic / demo / yours); label distribution; comparison DataFrame and chart; Random Forest test metrics and `classification_report` (`st.code`); feature-importance chart; note on the heuristic fallback if active. |
| About WeekWise | Problem statement, data sources, privacy summary, course and team, AI usage summary (link to `AI_USAGE.md` content), version. |

### 16.9 Messages

All user-facing messages are short, friendly English sentences without stack traces. Unexpected exceptions in a page are caught at page level, shown as *"Something went wrong while loading this page. Details: <one line>"*, and the rest of the app keeps working.

---

## 17 Synthetic and demo data (`synthetic_data.py`)

[DECIDED] High-quality, illustrative synthetic data must show every feature. [COURSE RULE] Fixed seed; demo data visibly marked in the UI.

### 17.1 Principles

* One random generator `rng = random.Random(RANDOM_SEED)` (an object of the `random` module's `Random` class, with the same methods taught in U02 s5: `randrange`, `choice`, `shuffle`, `random`, `gauss`) is passed to every generator function, so the result does not depend on other code calling `random`.
* **Anchor date** = Monday of the current week at generation time (stored as setting `demo_anchor_date`). All placeholder dates are relative to it, so the demo always looks current. Same anchor + same seed → identical data (tested).
* All synthetic records are flagged: `users.is_synthetic = 1`, `series.is_placeholder = 1`, `interactions.is_demo = 1` for the local user's demo history.
* `python synthetic_data.py` regenerates everything (same as *Reset demo data*).
* Generation order inside `generate_all()`: placeholder source files and demo calendar → `import_all(..., include_past=True)` → synthetic users (incl. the six showcase users) → friendships → interactions → `delete_past_unconfirmed_events()` → sanity checks.

### 17.2 Placeholder event sources (written as real input files, so the importers are exercised exactly as with real data)

**Unisport – `unisport_placeholder.xlsx`** (written with `DataFrame.to_excel`): about 40 weekly courses covering all Sports bundles, valid from `anchor − HISTORY_WEEKS` weeks to `anchor + FUTURE_WEEKS` weeks (≈ 480 occurrences). Typical slots 07:00–08:00, 12:15–13:15, 17:15–18:30, 18:30–20:00, 20:00–21:30 (one late course ending 22:30 to exercise the day-window rule); fewer courses on weekends. Locations: *HSG Sports Hall*, *UniGym*, *HSG Sports Field*, *Athletik Zentrum St.Gallen*, *Hallenbad Blumenwies*, *Kletterhalle St.Gallen*. Mostly free (`0`), a few paid (e.g. climbing `20.00`). Registration and source link: `https://sportangebot.unisg.ch/`. Examples: Volleyball Beginner/Intermediate, Basketball, Football, Unihockey, Badminton, Tennis, Table Tennis, Squash, Running Group, Spinning, Swimming Technique, Rowing Ergometer, Functional Training, Bootcamp, Strength Basics, Hatha Yoga, Vinyasa Yoga, Pilates, Mobility & Stretching, Salsa, Zumba, Hip Hop, Boxing, Judo, Self-Defence, Bouldering.

**UniClubs – `uniclubs_placeholder.json`** (same shape as the assumed API payload, §7.3): about 60 events from about 20 clubs. Use the observed club names (Helvetian Investment Club, Consulting Club, Hedge Fund Club, Tech Club, Culture Society, Gaming Club, Handball Club, University Fitness Club, UHU St. Gallen, Cheerleading Club) plus generic invented ones (Entrepreneurs Club, Data Science Club, Debating Society, Photography Club, International Students Club, Sustainability Club, University Choir, Wine & Dine Club, Outdoor Club, Women in Business). Mix: types Talk / Workshop / Social / Career / Trip / Competition; about 70 % without end time; statuses mostly OPEN, some APPLY and Members Only, 3 FULL, 2 CLOSED; locations about 60 % HSG rooms (*Room 01-U102*, *SQUARE*, *HSG Sports Hall*, *UniGym*), 10 % St.Gallen venues, 10 % Online, 10 % Zurich/Basel company offices, 5 % trips (Munich, Prague, multi-day), 5 % TBA; about half free, others CHF 5–40. Spread over the history and future window, mostly evenings.

**City – `city_placeholder.json`**: about 30 events with end times: concerts (*Palace St.Gallen*, *Kugl*, *Tonhalle St.Gallen*), theatre (*Theater St.Gallen*, *Lokremise*), museums (*Kunstmuseum St.Gallen*, *Textilmuseum*), Christmas market, a public talk, a sports match; free and paid.

### 17.3 Synthetic users and their behaviour (so the model learns something meaningful)

* `N_SYNTHETIC_USERS = 60` with display names from a fixed list of first names; friend codes random (showcase users fixed, §17.5); `sharing_opt_in = 1` for 80 %.
* Directions: 1 (40 %), 2 (45 %) or 3 (15 %) of the three.
* Hidden taste per user and bundle: `affinity` drawn with `rng.uniform(0.0, 1.0)` for bundles in the user's directions and `rng.uniform(-1.0, 0.2)` otherwise. Never stored; it only drives behaviour.
* Favorites: 1–3 bundles with the highest affinity, 0–2 clubs, 0–2 series.
* Friendships: each user 3–6 friends (stored both ways).
* **Confirmations:** each user confirms 8–20 past placeable events. Events are drawn one at a time with `rng.choices(candidates, weights=...)` where `weight = 1 + 3 × max(0, affinity) + 2 × favorite_match`, skipping events already chosen, with a maximum number of attempts (every loop needs a way out); plus 2–5 events in the next two weeks (for *friends going* and popularity). About 5 % of confirmations become `cancelled`.
* **Attendance:** 85 % attended, 15 % not attended.
* **Rating** of an attended event:
  `mu = 2.7 + 1.1 × affinity + 0.5 × wants_direction + 0.4 × any_favorite + 0.25 × min(friends_going, 2) + pattern`,
  where `pattern` = +0.4 for career events on weekdays between 12:00 and 19:00, −0.4 for career events at weekends, −0.4 for sports events starting at 21:00 or later (non-linear patterns a Random Forest can pick up). `rating = round(mu + rng.gauss(0, 0.7))`, clamped to 1–5.
* **Sanity checks** [COURSE RULE]: the share of ratings ≥ 4 must lie between 0.35 and 0.65, there must be at least 500 rated interactions, and every direction must appear; otherwise `generate_all()` raises `ValueError` with a clear message.
* `confirmed_at` = 1–7 days before the event; `answered_at` = 1–48 hours after its end (never after `now`).

### 17.4 Demo calendar (`data/demo/demo_calendar.ics`)

A realistic HSG student week, written with the same f-string approach as the export, weekly entries with `RRULE:FREQ=WEEKLY;COUNT=12` starting at `anchor − 4 weeks` (HSG slots 08:15–10:00, 10:15–12:00, 12:15–14:00, 14:15–16:00, 16:15–18:00):

| Day | Entries |
|---|---|
| Mon | 08:15–10:00 Microeconomics · 10:15–12:00 Financial Accounting · 16:15–18:00 Business Law |
| Tue | 10:15–12:00 Statistics · 14:15–16:00 Marketing |
| Wed | 08:15–10:00 Fundamentals and Methods of Computer Science – Lecture · 12:15–14:00 Contextual Studies · 16:15–18:00 Management |
| Thu | 08:15–10:00 Statistics – Exercise · 14:00–18:00 Part-time job (private) |
| Fri | 10:15–12:00 FCS – Exercise group · 12:15–13:15 Lunch with friends (private) |
| Sat | 10:00–11:00 Groceries (private) |
| Sun | 17:00–19:00 Family dinner (private) |
| One-offs | Next week Wed 13:00–13:45 Dentist; this week Sat 19:00–23:00 Birthday party; this week Thu all-day *Mum's birthday* (all-day label); one `TRANSP:TRANSPARENT` reminder (must be ignored). |

### 17.5 Showcase friends and the local user's demo history

* Six synthetic users with fixed names and codes, created by `generate_all()`, all opted in: **Lena `LENA26`, Noah `NOAH26`, Mia `MIAK26`** (pre-added as friends) and **Luca `LUCA26`, Sara `SARA26`, Jonas `JONAS6`** (addable live in the demo; listed in *Demo friend codes*).
* `create_showcase_friends(connection, user_id, now)` (at onboarding, if demo history is chosen): friendships with Lena, Noah, Mia; each of them confirms 2–3 placeable events in the current and next week, preferring events that fit the demo calendar's gaps, so *friends going* badges are visible; Lena and Noah each send one pending invitation for such an event.
* `create_demo_history(connection, user_id, now)`: from events of the last two weeks that fit the demo calendar and match the user's directions – 3 attended with ratings 5, 4 and 2, 1 not attended, and 2 confirmed events that ended within the last 3 days **without** an answer (they trigger the pop-up and the inline ✓/✗). All with `is_demo = 1`. If too few suitable events exist, create as many as possible without failing.

---

## 18 Testing (`tests/`)

[SUGGESTION adopted] `unittest.TestCase` tests in the style of the assignment test cells, for all pure logic. No test may need network access or the real database: database tests use `get_connection(':memory:')` + `init_db()` (`get_connection(db_file=DB_FILE)` takes the path as a parameter for this reason); time-dependent tests pass a fixed `now`. Run with `python -m unittest discover -s tests -v`; all tests must pass at the end of every milestone.

| Test module | Must cover at least |
|---|---|
| `test_models.py` | End before start → `ValueError`; float price → `ValueError`; `'25.00'` → `Decimal('25.00')`; `display_tags()` incl. *Estimated end*; `source_label()` per subclass (polymorphism); `event_from_row()` picks the right class; `UserProfile.favorite_kinds()`; display-name validation. |
| `test_gaps.py` | The worked example §10.3 completely; touching/overlapping intervals merge; busy blocks outside the window; empty day → one gap 07:00–23:00; tolerance 5 min fits, 6 min does not; `placement_problem()` for each reason; an event goes to only one gap; events before `now` excluded; weekends; `navigation_bounds()`. |
| `test_calendar_import.py` | Demo calendar parses; RRULE expansion count; `TRANSPARENT` and `CANCELLED` skipped; all-day label vs busy setting; UTC → Europe/Zurich around the DST change (25.10.2026); WeekWise UID and description marker recognised; `webcal://` normalisation; non-https link rejected. |
| `test_calendar_export.py` | UID format; escaping order; folded lines ≤ 75 bytes incl. umlauts; CRLF endings; UTC times in winter and summer; cancellation variant; Google/Outlook links contain the right parameters; **round trip**: `build_ics()` → `parse_calendar()` recognises the WeekWise ID. |
| `test_categories.py` | Taxonomy validates; `ai` does not match "raclette"; plural match; multi-word keyword; Trip → `leisure_trips`; club category Business → Career; finance words → `career_finance`; score 0 → *Other*; Unisport → Sports; ties → lower `sort_order`. |
| `test_importers.py` | `classify_location()` for each type; missing end → +90 min, flagged; weekly expansion count and weekday names (English/German); invalid record skipped; list and `{"events": [...]}` payloads; rejection above `MAX_RECORDS_PER_SOURCE`; price parsing (`'CHF 25.00'`, `'free'`, `None`); re-import does not duplicate. |
| `test_database.py` | `init_db()` idempotent; foreign keys enforced; every allowed and a selection of forbidden interaction transitions; rating `CHECK`; `delete_past_unconfirmed_events()` keeps events with interactions or invitations. |
| `test_recommender.py` | `make_label()`; not-attended rows excluded; feature alignment with an unseen category; chronological `earlier_avg_rating_in_bundle` (no leakage); favorite boost cap; scores within 0–1; `evaluate_models()` returns 10 rows with the expected columns and uses a stratified 80/10/10 split. |
| `test_rankings.py` | Small fixture database → expected order for every metric; minimum-ratings rule for You and All users; category filters. |
| `test_friends.py` | Code length/alphabet/uniqueness; `add_friend()` errors (unknown, self, duplicate); symmetric storage; opt-in respected in *friends going*; simulated response rule. |
| `test_synthetic_data.py` | Reproducibility with a fixed anchor; label share within 0.35–0.65; required counts; demo calendar parses. |
| `test_api.py` | Flask test client (`app.test_client()`): `/health` 200; `/events` 400 for bad dates; 404 for unknown event; `POST /interactions` 201 and 400. |
| `test_project_rules.py` | Automated rule check over all `.py` files: first line `# <file name>`, module docstring, `AI CITATION` block present; `streamlit` imported only in allowed files; no `import *`; no `f"` / `f'` string passed to `execute(` or `read_sql(`. |

---

## 19 Documentation and project files

| File | Required content |
|---|---|
| `README.md` | [COURSE RULE] What WeekWise is (problem statement); feature list; requirements (Anaconda, Python 3.11); installation: `conda create -n weekwise python=3.11`, `conda activate weekwise`, `pip install -r requirements.txt`; start: `streamlit run app.py`; optional second terminal: `python api_server.py` (app works without it); demo mode explained; connecting your own calendar (Google/Outlook steps, upload fallback); adding real data (put `unisport.xlsx` in `data/sources/`, edit `UNISPORT_COLUMN_MAPPING`, copy `sources.example.json` to `data/private/sources.json`); where data lives; reset; running tests; project structure; troubleshooting (port in use, locked database, wrong Python version); pointer to `AI_USAGE.md`; team. |
| `STUDY_GUIDE.md` | Suggested reading order for the exam (`models.py` → `gaps.py` → `database.py` → `rankings.py` → `recommender.py` → `calendar_export.py` → views); a table *file → key functions → course concept → where taught* (based on planning §10, extended to the final code); explanation of the `# Course:` tags; per milestone "what to look at". |
| `AI_USAGE.md` | Table *date · tool/model · purpose · output · how the team checked it*: planning chat (1 Nov 2026), specification agent (this spec and `CLAUDE.md`, 1 Nov 2026), taxonomy v1 (part of this spec), coding agent (one row per milestone with model and files), any later use. Reminders: AI goes on the list of aids in the video; the voice-over must not be generated. |
| `requirements.txt` | The pinned set below. |
| `.gitignore` | `data/weekwise.sqlite`, `data/private/`, `data/snapshots/`, `__pycache__/`, `*.pyc`, `.DS_Store`, `.venv/`, `.streamlit/secrets.toml`. |
| `data/private/sources.example.json` | Template with empty URLs/tokens (committed; the real `sources.json` is ignored). |

**`requirements.txt`** [DEFAULT D-15] – verified on 1 November 2026 to install together on Python 3.11 and to provide every API used in this spec (`st.dialog`, `st.feedback`, `st.navigation`, `st.badge`, `st.segmented_control`, `recurring_ical_events.of`). Every version was at least two weeks old at that date. pandas is pinned to 2.3.3, the last 2.x release, rather than 3.0, because the course notebooks were written for pandas 2 behaviour (copy semantics, dtypes); numpy is pinned to 2.4.x because numpy 2.5 no longer supports Python 3.11. `tzdata` makes `zoneinfo` work on Windows.

```
streamlit==1.66.0
pandas==2.3.3
numpy==2.4.6
scikit-learn==1.9.1
matplotlib==3.11.2
seaborn==0.13.2
requests==2.34.2
python-dateutil==2.9.0.post0
flask==3.1.3
icalendar==7.3.0
recurring-ical-events==3.8.2
openpyxl==3.1.5
tzdata==2026.5
```

(The course Anaconda installation may contain older versions; installing into the fresh `weekwise` environment above avoids conflicts.)

---

## 20 Build plan and milestones

### 20.1 Rules for every milestone

* The agent builds **one milestone at a time**, in order, and stops at its end with a milestone report (template in `CLAUDE.md` §2). The next milestone starts only after the team says so.
* At the end of every milestone: the app starts with `streamlit run app.py` on a fresh database without network; all tests pass; every new file has the AI-citation header; `STUDY_GUIDE.md` and `AI_USAGE.md` are updated for the new files.
* Target dates leave room for team review (about 30 minutes per milestone: run the app, read the report, skim the new code with the study guide). They align with the course schedule: databases 11.11, data science 18.11/25.11, machine learning 02.12/09.12; optional MVP presentation GE07 on 13.11 or GE08 on 20.11; upload 10.12.

### 20.2 Overview

| # | Milestone | Target | Main files | Course topics practised |
|---|---|---|---|---|
| M0 | Project skeleton | Tue 03.11. | `config.py`, `app.py`, `requirements.txt`, README stub, test runner | A04 setup |
| M1 | Data foundation | Fri 06.11. | `models.py`, `database.py`, `categories.py`, `importers.py`, placeholder sources | L05, U05 s2, A07, A08 |
| M2 | Calendar and gaps | Mon 09.11. | `calendar_import.py`, `gaps.py`, `calendar_export.py`, demo calendar | U02–U03, A06 |
| M3 | **MVP**: onboarding + week page core loop | Thu 12.11. (GE07 13.11.) | `views/onboarding.py`, `views/week.py`, `ui_components.py` | L04, A04 Streamlit |
| M4 | ML and feedback loop | Wed 18.11. (GE08 20.11.) | `recommender.py`, `synthetic_data.py` (users/interactions), feedback UI | L02, A10 |
| M5 | Own Web API | Mon 23.11. | `api_server.py`, `api_client.py` | L06, A06 |
| M6 | Rankings and *About the model* | Fri 27.11. | `rankings.py`, `views/rankings.py`, `views/settings.py`, `evaluate_models()` | A07–A09, A11 |
| M7 | Friends | Wed 02.12. | `friends.py`, `views/friends.py`, showcase data | L04 sets/dicts, A07 |
| M8 | Real data, hardening, documentation, freeze | Mon 07.12. (buffer to 10.12.) | all; README, STUDY_GUIDE, AI_USAGE | all |
| R | Real-data integration (whenever an input arrives, after M1) | – | `config.py` mappings, adapters | A06, A08 |

### 20.3 Milestones with acceptance criteria

**M0 – Project skeleton**

* Scope: folder structure (§4.2); complete `config.py` (§4.4); `requirements.txt`; `.gitignore`; `sources.example.json`; `README.md` with install/start commands; `STUDY_GUIDE.md` and `AI_USAGE.md` stubs; `app.py` with page config and `st.navigation` showing one placeholder page; `tests/test_project_rules.py`.
* Acceptance:
  1. In a fresh environment (`conda create -n weekwise python=3.11`, `pip install -r requirements.txt`) the install succeeds and `streamlit run app.py` shows the placeholder page.
  2. `python -m unittest discover -s tests -v` passes, including the project-rules test.
  3. Every `.py` file starts with the header from `CLAUDE.md` §7.

**M1 – Data foundation**

* Scope: `models.py` (all classes, §5); `database.py` schema and the functions needed for events, categories, settings, import log (§6); `data/taxonomy.json` and `categories.py` (§8); `importers.py` with all three adapters, canonical records, location classification, end estimation, sanity checks, upserts (§7); `synthetic_data.write_placeholder_sources()`; first-start setup in `app.py` (§4.6 steps 1–4); temporary Week page that lists this week's events with `st.dataframe` grouped by direction.
* Acceptance:
  1. First start creates `data/weekwise.sqlite` with ≈ 480 Unisport, ≈ 60 UniClubs and ≈ 30 city occurrences; every series has a bundle; FULL/CLOSED events are stored with their status.
  2. Starting again does not duplicate anything; `import_log` shows one row per source with origin *placeholder* (or *excel*).
  3. Tests for models, database (schema, foreign keys), categories and importers pass.

**M2 – Calendar and gaps**

* Scope: `calendar_import.py` (three modes, §9), `synthetic_data.write_demo_calendar()`, `gaps.py` (§10), `calendar_export.py` (§11); Week page shows a real 7-column grid with busy blocks and labelled gaps (no suggestions yet) and week navigation.
* Acceptance:
  1. The demo week renders: classes on the right days and times, Thursday's all-day label, the transparent reminder ignored.
  2. The worked example (§10.3) and the export round trip pass as unit tests.
  3. Team check: a downloaded test `.ics` imports correctly into Google Calendar and Outlook at the right local time; a real secret Google link is read (team's own calendar, never committed).

**M3 – MVP: onboarding and the week-page core loop** (ready for the optional MVP presentation on 13.11.)

* Scope: onboarding (§16.2, demo history checkbox may be inactive until M4); suggestions in gaps with `heuristic_score()`; top 3 + dropdown; sidebar filters incl. favorites; event dialog with confirm, export block, cancel, favorite toggles (§16.4); *Beyond your gaps*; sidebar status; Settings tabs *Profile* and *Calendar*.
* Acceptance:
  1. A new user completes onboarding in under a minute and lands on the current week with suggestions in the gaps, colour-coded with direction labels.
  2. Confirming an event turns it into a confirmed block, overlapping suggestions disappear, the export buttons work; cancelling brings the suggestions back and offers the cancellation file.
  3. Selecting a bundle filter shows all fitting events of that bundle with links; clearing returns to top 3.
  4. An invalid calendar link shows a friendly message and the demo calendar; nothing crashes.

**M4 – Machine learning and feedback loop** (ready for the optional MVP presentation on 20.11.)

* Scope: synthetic users, friendships and interactions (§17.3); `recommender.py` (§12.1–12.6); scores and reasons on cards; inline feedback states (§16.3) and pop-up (§16.5); `create_demo_history()`; retrain once per rating; toast.
* Acceptance:
  1. The synthetic-data sanity checks pass; the model trains in under 3 seconds.
  2. Two test profiles with different directions get visibly different top 3 in the same gap (unit test on `score_events`).
  3. With demo history on, the pop-up appears on start with two pending events; *Later* hides it for the session; it reappears after a browser reload (new session) until answered.
  4. ✓ → stars → rating saved → toast; exactly one retrain (cache key = number of ratings); ✗ rows never reach the training data (unit test).

**M5 – Own Web API**

* Scope: `api_server.py` (§15.1), `api_client.py` (§15.2), sidebar status, `test_api.py`.
* Acceptance:
  1. With `python api_server.py` running, the sidebar shows *Event data: own API ✓* and the API terminal logs the `GET /events` and `POST /interactions` requests.
  2. Stopping the API: the next interaction works through the database within 2 seconds, and the sidebar says so.
  3. Every status code in the route table is covered by a test.

**M6 – Rankings and *About the model***

* Scope: `rankings.py` (§13), Rankings page (§16.6), `evaluate_models()` and charts (§12.7), Settings tabs *Data sources*, *About the model*, *About WeekWise*.
* Acceptance:
  1. All four metrics work for You and All users, overall, per direction and per bundle; the best-rated minimums are applied.
  2. *About the model* shows the 10-row comparison table, the comparison chart, the Random Forest's test metrics with `classification_report`, and the feature-importance chart; a unit test confirms the stratified 80/10/10 split with seed 13 and a scaler fitted on training data only.
  3. *Refresh event data* and the import-log table work.

**M7 – Friends**

* Scope: `friends.py` (§14), Friends page (§16.7), `create_showcase_friends()`, *friends going* badges and invitation pins in the grid, invite flow in the event dialog, `friends_going` wired into the features.
* Acceptance:
  1. Adding `LUCA26` works; unknown code, own code and duplicates give clear errors.
  2. The current demo week shows at least two *friends going* badges and Lena's pinned invitation.
  3. Inviting a friend produces a labelled simulated answer; accepting Noah's invitation confirms the event and offers the export.
  4. With sharing opt-in off for a friend, that friend is not counted.

**M8 – Real data, hardening, documentation, code freeze**

* Scope: run task R for every input that has arrived; final taxonomy review with real events and freeze (§8.4); complete comment pass (every function has a docstring, *why* comments, `# Course:` tags); `README.md`, `STUDY_GUIDE.md`, `AI_USAGE.md` complete; fresh-install test on Windows **and** macOS; stretch items only if everything else is done.
* Acceptance – **end-to-end scenario**, on a fresh machine without network, must run without errors:
  1. Delete `data/weekwise.sqlite`, start the app → spinner → Welcome page with the problem statement.
  2. Onboard: Sports + Career, name *Alex*, demo history on, demo calendar.
  3. Pop-up with two events: answer one ✓ with 4 stars, press *Later* for the other.
  4. Week page: current week, gaps with top-3 suggestions, match %, *friends going* badges, *Lena invited you*.
  5. Open a suggestion, confirm, download the `.ics`, open the Google link; overlapping suggestions are gone.
  6. Filter *Racket sports* → all fitting courses with links; clear filters.
  7. Next week, back, *Today*; *Beyond your gaps* lists an off-site event or trip.
  8. Rankings: category *Career*; charts for You and All users.
  9. Friends: add `LUCA26`; invite Lena to the confirmed event (simulated answer); accept Noah's invitation.
  10. Settings → *About the model*: table, test metrics, charts.
  11. Start `api_server.py` → sidebar shows the own API; stop it → database fallback.
  12. Settings → *Data sources*: refresh; import log updated.
  13. Reload the browser (new session) → the pop-up reappears for the event answered with *Later*.
  14. All tests pass; `test_project_rules.py` finds no violations.

**R – Real-data integration (any time after M1, when an input arrives)**

1. Unisport: put the file at `data/sources/unisport.xlsx`; inspect its headers; edit `UNISPORT_COLUMN_MAPPING` (and `WEEKDAY_NAMES` if needed); run `python importers.py`; fix every skipped-row reason that indicates a mapping problem.
2. UniClubs / city: put URL and token into `data/private/sources.json`; fetch once; compare the payload with §7.3 and adapt `UNICLUBS_FIELD_MAPPING` / `CITY_FIELD_MAPPING` (and the parse function only if the structure differs); a snapshot is stored automatically.
3. Check category assignment on the real events (*Other* share should stay below about 15 %); add keywords if needed – never rename keys.
4. Document the change in the milestone report and `AI_USAGE.md`.

---

## 21 Extra ideas (not in scope unless the team asks)

* **"Students similar to you"** [DECIDED as extra idea]: a meetup feature based on collaborative filtering – the course's own KNN with Euclidean distance (A10) over users' rating vectors could suggest people with similar taste.
* **Public deployment** on Streamlit Community Cloud (the course's optional extension) – would need a hosted database and a privacy review.
* **Subscribable WeekWise feed**: an API route serving the user's confirmed events as one ICS feed; calendar apps that can reach the computer could subscribe once, which would also make cancellations automatic.
* UniClubs *Opportunities* and *Courses* sections as further sources.
* Per-type default durations (already prepared: `USE_TYPE_DURATIONS`).
* Proportional block heights in the week grid.

---

## 22 Decision register (flagged defaults)

Every default below is binding for the coding agent until the team overrides it. To override, the team changes the entry here (and, where named, the constant in `config.py`) and tells the agent.

| ID | Topic | Default chosen by this spec | Why | Origin in the planning summary |
|---|---|---|---|---|
| D-01 | Unisport Excel format | Target schema + configurable `UNISPORT_COLUMN_MAPPING`; placeholder Excel until delivered | Real file not yet available | Planning open item 1 |
| D-02 | UniClubs API | Adapter for an assumed JSON shape + `UNICLUBS_FIELD_MAPPING`; placeholder modelled on field notes | API not yet available | Planning open item 2 |
| D-03 | City events API | Same adapter pattern, placeholder | Details missing | Planning open item 3 |
| D-04 | Meaning of "own small API" | Local Flask API (5 routes) over SQLite on 127.0.0.1:5001; app falls back to direct DB access | Extra credit for req. 2; graders can always run the app | Planning open item 4, planning §11.5 |
| D-05 | Finance placement | Bundle *Finance & investing* under Career | Fits club landscape (investment, hedge fund clubs) | Planning open item 5 |
| D-06 | New events → frozen bundles | Keyword scoring (title ×2, description ×1) with event-type and club-category rules; *Other* per direction | Deterministic, testable, no runtime LLM | Planning open item 6 |
| D-07 | Missing end times | 90 min, flagged *Estimated end*; per-type durations as config switch (off) | Team stated 1.5 h explicitly | Planning open item 7 |
| D-08 | Cancelling an exported event | Remove in WeekWise + cancellation `.ics` (same UID) + manual-delete hint | No API write access | Planning open item 8 |
| D-09 | Daily window for gaps | **07:00–23:00** (proposal was 22:00) | Evening club and city events would otherwise never fit | Planning open item 9 |
| D-10 | Calendar publishing blocked | `.ics` upload and demo calendar as fallbacks | University Outlook restrictions | Planning open item 10 |
| D-11 | Editing questionnaire later | *Settings & About → Profile* | Small settings area as recommended | Planning open item 11 |
| D-12 | Course files missing from the folder | Rely on Assignments 04–11 as digested; nothing to build | – | Planning open item 12 |
| D-13 | AI-citation format | Header template in `CLAUDE.md` §7; team checks it against the HSG guideline | Course rule: cite and quote AI code | Planning open item 13 |
| D-14 | `# Course:` tags in comments | **Yes**, at the first use of a concept per file | Doubles as exam preparation | Planning open item 14 |
| D-15 | Library versions | Pinned set in §19, verified on Python 3.11; pandas 2.3.3, numpy 2.4.6 | Needs `st.dialog`/`st.feedback`; course-near pandas | Planning open item 15 |
| D-16 | All-day calendar entries | Not busy by default (shown as day labels); setting to treat them as busy | Birthdays/holidays would otherwise block whole days | new |
| D-17 | Unknown / TBA locations | Unknown venue → St.Gallen; TBA placed with tag *Location TBA* | All sources are local | new |
| D-18 | Page folder | `views/` + `st.navigation` instead of `pages/` | Hide main pages until onboarding is done | planning §11.10 |
| D-19 | Time handling | Naive Europe/Zurich datetimes inside; conversion only at calendar import/export | Simplest correct model for a one-city app | new |
| D-20 | Event storage | `series` (activity) + `events` (occurrences) | Favorite/rank weekly courses as one activity | new |
| D-21 | Favorites table | One table with `kind` + `target_id`, existence checked in code | Three favorite kinds, one simple table | new |
| D-22 | Favorite boost | +0.10 per matching kind, max +0.25; invitations pinned first | "Prioritised" needed a mechanism | planning §4.9 assumption |
| D-23 | Sample weights | Local user's own ratings ×5, synthetic and demo ×1 | Planning suggestion, value chosen | planning §4.11 |
| D-24 | Best rated for *You* | Minimum 1 rating (All users: 3) | A single user rarely rates a series 3 times | new |
| D-25 | Demo history for the local user | Onboarding checkbox, default on; removable | Shows feedback features immediately in the demo | new |
| D-26 | Share link | Copyable text with the friend code | Local app has no public URL | new |
| D-27 | Answers of synthetic friends | Immediate, rule-based, labelled *(simulated)* | Makes invitations demonstrable | planning §4.13 assumption |
| D-28 | Secret link storage | Session only; optional "remember" in git-ignored file | Course security rule | planning §9.3 |
| D-29 | When to evaluate models | Lazily on *About the model*, cached per number of ratings | Keeps app start fast | planning §11.7 |
| D-30 | Block heights | Chronological list with times; proportional heights as stretch | Simpler grid, all information visible | planning §11.1 |
| D-31 | Bundle-average feature | Computed only from earlier events | Avoids leaking the label into features | new |
| D-32 | Narrow screens | User toggle *Week / Day* layout | Streamlit cannot detect screen width | planning §11.1 |
| D-33 | Recognising link-created entries | Marker line in the description in addition to the UID | Google/Outlook links cannot set a UID | new |
| D-34 | Rankings computation | SQL aggregates (A07) + matplotlib (A09); pandas for model evaluation | Uses both taught tools where each fits | planning §4.12 |
| D-35 | Past unconfirmed events | Deleted at session start and import; importers skip ended records except during the demo build | "Past potential events are deleted" | planning §4.5 |
| D-36 | Unisport course validity | `valid_to` defaults to 20.12.2026 if the Excel has no end date | Horizon "as far as data exists" | new |
| D-37 | Problem statement wording | Text in §1.1 | Draft finalised | planning §3 |
