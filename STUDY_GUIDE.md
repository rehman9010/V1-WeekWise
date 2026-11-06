<!-- AI CITATION: OpenAI Codex (GPT-6), updated 2026-11-06, Navy & Oak design, SPEC.md v1.0; review pending -->
# WeekWise study guide – through M8

Start with `models.py`, then `gaps.py`, `database.py`, `rankings.py`, `recommender.py`
and calendar export. Read the importers when tracing source data into the database.
Read `app.py`, `ui_components.py` and `views/week.py` last to see how those pieces are displayed.
Both learned recommendations and the heuristic fallback are implemented, alongside evaluation.

`# Course:` comments identify concepts using only the reference codes in CLAUDE.md §8.
They are reading aids, not claims that third-party calendar internals were taught in the course.

| File | Functions/classes to read | Concept | Course reference |
| --- | --- | --- | --- |
| `config.py` | constants, `local_now` | shared values, default arguments, local time boundary | U03 s2, A06 |
| `models.py` | `Event`, `UnisportCourse`, `ClubEvent`, `CityEvent` | inheritance, overriding, polymorphism | L05, U05 s2 |
| `models.py` | `_Interval`, validated properties, `price_to_rappen` | encapsulation and exact Decimal money | U05 s2, L02 |
| `models.py` | `Gap.fits`, `UserProfile.favorite_kinds` | comparisons and set membership | L03, L04 |
| `gaps.py` | `merge_intervals` | sorted copy, tuple unpacking, one-pass merging | U03 s3–s5 |
| `gaps.py` | `find_gaps_for_day` | clipping, initialization/processing/termination, sentinel | U03 s1 |
| `gaps.py` | `assign_events_to_gaps`, `placement_problem` | branching, sets, early continue, break | L03, L04 |
| `gaps.py` | `week_start`, `navigation_bounds` | dates and timedelta arithmetic | A06 |
| `database.py` | `init_db`, `get_connection` | schema, primary/foreign keys, constraints | A07 |
| `database.py` | `apply_source_records`, upserts | parameter tuples, transactions, stable identity | A07 |
| `database.py` | `get_events_df`, `get_calendar_confirmations` | joins, date filters, DataFrames | A07, A08 |
| `categories.py` | `normalize_text`, `keyword_matches`, `assign_category` | loops, strings, deterministic scoring | U03 s1–s3, L04 |
| `importers.py` | `load_unisport_excel`, `expand_weekly_course` | pandas file input, recurring dates | A08, A06 |
| `importers.py` | `normalize_record`, `save_records`, `import_source` | boundary validation, fallback, atomic updates | A06, A07 |
| `calendar_import.py` | `fetch_ics_text`, `load_calendar` | HTTP status, timeouts, exception handling | A06, L06 |
| `calendar_import.py` | `parse_calendar`, `_add_component` | reducing input into domain objects, sets | L05, L04 |
| `calendar_import.py` | recurrence expansion, `to_local_naive` | accepted calendar-library extras and time zones | A06 for datetime context; extras per CLAUDE §4 |
| `calendar_export.py` | `escape_ics_text`, `fold_ics_line` | ordered replacements, loops, UTF-8 byte limits | U02 s3, U03 s1 |
| `calendar_export.py` | `build_ics`, `make_uid`, `format_utc` | f-strings, date conversion, stable identifiers | U02 s3, A06 |
| `calendar_export.py` | Google/Outlook compose helpers | URL-encoded query parameters | A06, L06 |
| `synthetic_data.py` | `write_placeholder_sources`, `write_demo_calendar` | seeded generation, loops, text/files | U02 s5, A08 |
| `app.py` | `initialize`, `main` | orchestration and Streamlit page registration | A04, L04 |
| `ui_components.py` | caches, calendar controls | session state, input widgets, cache invalidation | L04; framework behavior per CLAUDE §9 |
| `views/week.py` | `_navigation`, `_draw_day`, `render_week` | columns, containers, separation of UI and logic | L04, A04 |
| `tests/support.py` | fixtures, `DatabaseCase` | reusable isolated test setup | U05 s2 |
| `tests/test_foundation.py` | models, database, taxonomy, import tests | assertions and invalid-input cases | U05 s2, A07, A08 |
| `tests/test_gaps.py` | worked example and boundaries | test the business definition, not just an implementation | U03 s1–s5 |
| `tests/test_calendar_import.py` | recurrence, DST, privacy, fallback | representative fixtures and failure cases | A06 |
| `tests/test_calendar_export.py` | round trip, folding, UTC and links | interoperability and boundary checks | U02 s3, A06 |
| `tests/test_project_rules.py` | citation/import checks | AST inspection used only as test tooling | project requirement, not a claimed course topic |

For M1, trace one JSON record through normalization, category assignment and the two upserts.
Compare a second import: its event ID remains the same. Inspect the SQL constraints and why a
failed batch rolls back. The reconstructed M1 tests are consolidated in `test_foundation.py`.

For M2, reproduce SPEC §10.3 on paper, then step through `find_gaps_for_day`. Five minutes is an
allowed overlap for placing an event; it does not shrink the calendar's busy blocks or turn a
59-minute gap into a qualifying gap. After adding a confirmation, recompute the gaps.

Next read one exported ICS file: it represents one occurrence, uses a stable UID, escapes text
before folding, and converts Zurich wall time to UTC. The import round trip recognizes that UID.
Understand why unknown IDs must remain busy after a database reset. Calendar parsing and download
errors do not expose a private link; fallback is visible in the UI.

## M3: follow one planning decision

Start with the new profile queries in `database.py`. `complete_onboarding()` validates before
writing and creates the user, focus and completion settings in one transaction. Favorite targets
are checked with fixed SQL because one favorite table refers to three possible target tables.

Next trace an activity through `heuristic_score()`, `score_events()` and `filter_suggestions()`.
Focus changes rank; it does not exclude events from other directions. A favorite matches at
series, club or bundle level independently, with a capped combined bonus. Filters use OR, so a
favorite remains visible alongside activities in a separately chosen bundle.

Finally follow Details → Confirm → full rerun → recomputed gaps. `save_interaction()` checks
both the previous state and the clock before writing. The same occurrence can be cancelled and
confirmed again without creating another interaction row. Its export UID remains stable.
Known exports stay recognized after cancellation, so an old external feed entry cannot undo the
local cancellation. Unknown IDs still become ordinary busy blocks.

| File | Functions to study | Concept | Course reference |
| --- | --- | --- | --- |
| `database.py` | `complete_onboarding`, `update_user_profile`, `get_user_profile` | atomic writes, joins, set-valued preferences | A07, L04 |
| `database.py` | `add_favorite`, `remove_favorite` | reference validation, idempotent operations | A07 |
| `database.py` | `save_interaction`, `_validate_transition` | state machines, time bounds, rejecting invalid actions | L03, A06, A07 |
| `database.py` | `get_known_event_ids` | set membership for export deduplication | L04 |
| `friends.py` | `generate_friend_code` | seeded random choices, sets and bounded retries | U02 s5, U03 s1 |
| `recommender.py` | `heuristic_score`, `favorite_boost` | explicit formula, capped contributions | L02, L03 |
| `recommender.py` | `Recommender.score_events`, `filter_suggestions` | objects, stable sorting, OR semantics | L05, A05, L04 |
| `views/onboarding.py` | focus/calendar validation, `_finish` | staged input, state, validation before persistence | L04, A04 |
| `views/settings.py` | `_profile_settings`, `render_settings` | reusable controls and saved preferences | L04 |
| `ui_components.py` | `render_event_card`, `render_gap_suggestions` | slicing top three, remaining dropdown options | U03 s5, L04 |
| `ui_components.py` | `show_event_dialog`, `render_queued_dialog` | one queued dialog, reruns after writes | L04; Streamlit behavior per CLAUDE §9 |
| `views/week.py` | `_week_contents`, `_render_grid`, `_render_beyond` | composition of pure logic and UI | L04, L05 |
| `tests/test_profiles.py` | profile, favorite, transition and cancelled-export tests | isolated fixtures and boundary assertions | A07, U05 s2 |
| `tests/test_recommender.py` | formula, boost caps, ordering and OR filters | deterministic unit tests | L02, A05 |

At the M3 checkpoint, attendance/rating transitions existed only at the database layer.
M4 subsequently added their UI and model training, and extended the baseline-only scorer.

## M4 – training and feedback

| File | Functions to study | Concept | Course reference |
| --- | --- | --- | --- |
| `recommender.py` | `event_feature_row`, `to_feature_matrix` | explicit features, one-hot encoding and column alignment | L02, A08, A10 |
| `recommender.py` | `build_training_data`, `train_production_model` | chronological history, leakage prevention, weights and Random Forest | L02, U03 s1, A10 |
| `database.py` | `get_pending_feedback`, `get_training_interactions_df` | joins, state and time filters | A07 |
| `database.py`, `ui_components.py` | `get_rating_cache_key`, `cached_recommender` | change detection, bounded resource caching | L04 |
| `friends.py` | `friends_going_counts` | sharing-aware aggregate query | A07 |
| `synthetic_data.py` | `create_synthetic_users`, `_weighted_events`, `_synthetic_rating` | private seeded RNG, bounded sampling, hidden tastes and noisy labels | U02 s5, U03 s1, L02 |
| `synthetic_data.py` | `create_demo_history`, `create_showcase_friends` | compose calendar gaps with generated history; idempotent setup | L03, L05 |
| `ui_components.py` | `render_feedback_widget`, `prepare_feedback_popup`, `show_feedback_dialog` | attendance state machine, 0-based stars, one dialog per run, session state | L03, L04 |
| `database.py` | `clear_demo_history`, `reset_database`, `remove_incomplete_synthetic_seed` | targeted deletion, foreign-key order, preserving genuine data during upgrades | A07 |
| `tests/test_training.py`, `tests/test_synthetic_data.py` | leakage, weighting, repeatability and same-gap top-three tests | isolated fixtures and meaningful model assertions | A07, A08, A10 |

M4 extends the M3 scorer to accept a fitted model. The heuristic remains the fallback for fewer
than 50 training rows or a single label class.

Follow a new rating end to end: Streamlit returns 0–4 → add one → `save_interaction` validates
attendance and stars → rating count/revision changes → full rerun → cached model trains once →
cards score current events. A corrected rating changes the revision even when the count stays
constant. A no-show changes attendance but contributes no training row.

Evaluation and held-out train/validation/test splitting are described in M6 below. Production fits
all eligible rows; its tests check behavior without presenting synthetic-data accuracy as evidence
of real-world recommendation quality.

## M5 – local Web API and database fallback

| File → function | Topic | Course |
| --- | --- | --- |
| `api_server.py` → `create_app`, route functions | HTTP verbs, paths, query parameters, JSON and status codes | L06, A06 |
| `api_server.py` → `_open_connection`, `_close_connection`, `prepare_database` | Request resource lifetime and preserving startup initialization | A07 |
| `api_server.py` → `_date_range`, `_interaction_body` | Input validation before a state change | L03, A06 |
| `api_client.py` → `get_events`, `get_event`, `save_interaction` | Requests, bounded timeout, identical fallback behavior | L06, A06 |
| `api_client.py` → `_fallback_interaction` | Recovering a committed write after a lost response | L06, A07 |
| `database.py` → `get_event_records`, `get_events_df` | SQLite integers/nulls, exact Rappen and pandas dtype conversion | L02, A07, A08 |
| `database.py` → `validate_interaction_input`, `save_interaction` | Shared validation instead of two different sets of rules | L03, A06 |
| `ui_components.py` → `cached_api_available`, `load_events`, `save_user_interaction` | Cache keys, failure cooldown and visible origin | L04, L06 |
| `tests/test_api.py` → route/client/connection tests | In-memory fixtures, Flask test client and mocked transports | A06, A07 |

Trace a confirmation: button → API client → POST JSON → Flask input checks → shared SQLite state
machine → HTTP 201 → UI rerun. If the API is down, the client applies that same state machine
directly. If the API committed but its response was lost, a changed matching row supplies the
result without another write. An unchanged duplicate confirmation remains invalid.

Compare the public event row and its domain object: JSON timestamps remain Zurich local text;
prices remain exact integer Rappen until `models.event_from_row` converts them to Decimal.
The object-typed DataFrame prevents a nullable integer money column from being coerced to float.

## M6 – rankings and model evaluation

| File → function | Topic | Course |
| --- | --- | --- |
| `rankings.py` → `get_ranking`, `_append_filters` | Fixed SQL fragments, bound parameters, JOIN/GROUP BY/HAVING and ordering | A07 |
| `rankings.py` → `category_overview` | LEFT JOIN, conditional COUNT/AVG and missing values | A07, A08 |
| `rankings.py` → `plot_ranking`, `plot_category_overview` | Horizontal/grouped bars, numeric labels and returning Figures | A09 |
| `recommender.py` → `_split_for_evaluation`, `evaluate_models` | Stratified 80/10/10 split, fixed seed and training-only RobustScaler | A11 |
| `recommender.py` → `_comparison_models`, `_classification_metrics` | Five classifiers, macro averages, zero division and held-out reports | A10, A11 |
| `recommender.py` → `feature_importances`, plot helpers | Sorting, pivoting, top-ten slices and chart interpretation | A08, A09 |
| `database.py` → `get_training_summary`, `get_source_counts_df` | Reuse eligibility rules and count distinct series across occurrences | A07, A08 |
| `ui_components.py` → `cached_evaluation`, `clear_data_caches`, `render_reset_controls` | Bounded caches, explicit invalidation and confirmation before destructive actions | L04 |
| `app.py` → `initialize` | Detect frozen-taxonomy mismatch before modifying category data | L03, A07 |
| `views/rankings.py` → `_category_choices`, `_ranking_pair` | Label-to-key mapping, tabs and anonymous tables | L04 |
| `views/settings.py` → `_data_sources`, `_about_model`, `_about_weekwise` | Shared logic behind widgets, on-demand execution and readable evidence | L04 |
| `tests/test_rankings.py`, `tests/test_evaluation.py` | Hand-calculated aggregates and tracing train/validation/test boundaries | A07, A10, A11 |

For rankings, calculate the small fixture by hand. Alpha has two occurrences, six non-cancelled
plans, four attendances but only three ratings. Its average is four stars. Bravo's two ratings
average five but fail the All users minimum; a single local rating qualifies for You. Understand
why grouping by series differs from grouping by occurrence, and why LEFT JOIN preserves empty
directions. The overview uses the glossary's current `confirmed` status; Most popular additionally
includes attended/no-show records. Missing averages remain null instead of becoming zero stars.

For evaluation, follow one row through both stratified splits. Training, validation and test are
disjoint. Only training rows fit the scaler or classifiers. Validation compares choices; the
unscaled forest's test results use a separate untouched subset. The displayed evaluation forest
is not the production model fitted on all data with sample weights. Macro metrics give equal
weight to both labels, unlike accuracy's per-row averaging.

`evaluate_models()` returns the required comparison DataFrame. Its `.attrs` contains plain
serializable report values, split sizes and feature importances, so the UI does not need to fit
another model or put a fitted object into a table. Small/one-class/rare-class inputs return an
empty comparison with a reason. Feature importance describes association in this fitted forest;
synthetic evaluation does not demonstrate accuracy on real students.

Finally trace Settings → Data sources → Refresh: reimport, cleanup only unreferenced ended
events, clear caches, rerun, show reports. Reset additionally requires a checkbox, drops tables in
dependency order, closes the old cached connection, clears session state and regenerates the demo.
Private calendar/source files are outside SQLite and survive. Tracked tabs use `.open` so model
evaluation is never an accidental cost of editing the profile or calendar.

## M7 – friends, sharing and invitations

| File → function | Topic | Course |
| --- | --- | --- |
| `friends.py` → `add_friend`, `get_friends_df` | Normalizing codes, symmetric relationship rows and parameterized queries | U03 s1, L04, A07 |
| `friends.py` → `set_sharing_opt_in`, `get_friend_plans_df`, `get_friends_going_df` | Explicit consent and filtering before data leaves storage | A07 |
| `friends.py` → `send_invitations`, `_invitation_recipients` | Set membership, validating a whole batch and unique-key idempotence | L04, A07 |
| `friends.py` → `simulate_friend_response` | Deterministic branching and strict interval overlap | L03, A07 |
| `friends.py` → `respond_to_invitation`, `_answer_invitation` | Recipient checks, state transitions and one transaction for two writes | L03, A07 |
| `database.py` → `save_interaction`, `apply_interaction_in_transaction` | Shared validation and caller-owned transactions | A07 |
| `recommender.py` → `Recommender.score_events` | Stable tuple-key sorting: invited first, then descending score | A05, U03 s4 |
| `ui_components.py` → `calendar_fit_message`, `render_invitation_response`, `_invite_controls` | Composition of calendar logic, saved state, reruns and export | L04, A06 |
| `views/friends.py` → page helpers | Two-column UI, explicit privacy control and received/sent lists | L04 |
| `views/week.py` → `render_week` | Load invitation metadata once and reuse it in each eligible gap | L04 |
| `tests/test_friends.py` | Fixtures, failed-write rollback, time boundaries and privacy assertions | A07, U05 s2 |

Begin with `add_friend`: normalize the submitted code, reject unknown/self/duplicate cases, then
insert both directions in one transaction. The list of friends can show names even when sharing
is off; their plans, presence names and friends-going counts are filtered by `sharing_opt_in`.
Sharing is independent of an explicit invitation to one activity. No query accesses private
calendar entries, which are never stored in SQLite.

Trace a sent invitation: require the sender's confirmed future activity, validate all recipients,
insert one row per recipient using a unique constraint, then evaluate synthetic responses. The
simulation uses focus and other confirmed activity intervals. End equal to start is not overlap;
the same already-confirmed event is excluded from the conflict query. It uses neither a private
calendar nor a random draw. Human recipients keep their pending state.

Trace Accept: check that the local user is the recipient, validate that confirmation is still
allowed, open one transaction, confirm through the shared interaction state machine, then save
the invitation answer. `apply_interaction_in_transaction` deliberately does not open/commit a
nested context. SQLite connection contexts do not create nested savepoints: an inner commit
could otherwise leave a plan behind when saving its answer fails. The test trigger proves both
writes roll back. Ordinary `save_interaction` still owns and commits its own transaction.

Invitations do not change numerical match scores. Sorting by `(is_invited(), score)` places
pending invites first, retaining normal score order within each group. Only events already
assigned to a fitting gap can appear there. Filters retain their OR semantics. The Friends page
keeps non-fitting invitations visible and assesses the user's current calendar plus existing
WeekWise plans without displaying private titles. Acceptance reuses the normal export dialog.

The M4 showcase generator already supplies the three friends, fitting plans and two invitations;
M7 connects them to real UI actions. A new or corrected rating still controls model retraining.
Adding friends, changing sharing or answering invitations refreshes prediction-time social counts
without claiming that a new model was trained.

## M8 – trace and verify the complete app

| File | Functions or section to study | Concept | Course reference |
| --- | --- | --- | --- |
| `synthetic_data.py`, `config.py` | `_club_rows`, `DEMO_TRIP_INDICES` | Deterministic fixtures that demonstrate a required branch | U02 s5, L03 |
| `tests/test_synthetic_data.py` | `test_first_week_contains_a_future_trip` | A regression assertion about user-visible behavior | U05 s2 |
| `database.py` | `get_demo_counts` | Independent aggregate queries avoid multiplying series by occurrence joins | A07 |
| `ui_components.py` | `render_sidebar_status`, `render_filters` | Show data provenance and explain actual recommendation inputs | L04 |
| `tests/test_data_status.py` | `DataStatusTests` | Empty, real-only and mixed-origin database fixtures | A07, U05 s2 |
| `views/check_demo.py` | `isolate_paths`, `main` | Module loading and definition-time default arguments | U03 s2; test tooling explained below |
| `views/demo_scenario.py` | `DemoScenario`, `check_week_and_confirm`, `refresh_and_restore` | TestCase inheritance, state changes, invariants and cleanup | U05 s2, L04, A07 |
| `views/demo_scenario.py` | `check_export`, `check_api_and_fallback` | Calendar round trip, HTTP GET/POST and local fallback | A06, L06 |
| `app.py`, `models.py` | `main`, `UserProfile` | Reviewed session-state and domain documentation | L04, L05 |
| `.github/workflows/check.yml` | `fresh-install` matrix | Repeat installation and checks on three operating systems | Development tooling, not claimed course content |
| `README.md`, `MILESTONE_M8.md`, `AI_USAGE.md` | Setup, acceptance evidence and limitations | Reproducibility, documentation and honest attribution | L02 project rules |

Run `python -m views.check_demo`, then trace one confirmation from its button through the API
client, database state machine, recomputed gaps and calendar export. The same interaction rules
apply when the optional API process disappears. A new browser session restores profile/history
from SQLite, but its popup state starts anew; therefore the event deferred with Later returns.

The final scenario exposed a missing first-week Beyond example: the earlier trips all fell in
other weeks. One of the existing three trips now starts on the first Saturday. The fix preserves
the event count, uses the existing generator, and leaves saved user data alone. The unit test
checks the resulting open, off-site occurrence; the scenario checks that the actual page shows it.

Read `get_demo_counts` alongside the mixed-source test. A weekly course with several occurrences
is still one placeholder series. Synthetic users are a separate reason to show Demo data even
when all activity sources are real. Neither count depends on a user's private calendar.

The portable runner is deliberately separate from network-free unit-test discovery. It imports
Streamlit only under `views/`, but is never registered as a page. Temporary-directory paths must
be installed **before** importing modules whose default parameters capture config paths. The
runner rejects direct use of its scenario against the application's normal data directory.
`unittest.mock`, AppTest, temporary directories, process/socket management and the CI workflow
are test infrastructure extras, not newly claimed lecture topics. The subprocess exists only
to test the optional API; the product gains no background scheduler. Startup polling is bounded
and cleanup reaps the owned process and closes SQLite connections, including on assertion failures.

For final review, explain why synthetic validation scores cannot establish real-world quality;
why a passing Linux run does not prove Windows/macOS installation; and why inspecting a generated
Google URL is different from checking the provider's browser UI. The M8 report records those
boundaries and the still-needed real-source taxonomy review.


## Navy & Oak visual update (2026-11-06)

The requested dark design uses Streamlit's native `.streamlit/config.toml` theme for
colours, serif headings and sans-serif body text. `assets/theme.css` adds only the
requested decorative texture, card shadows and responsive spacing. This is an explicit
user-requested exception to the original no-custom-CSS convention; widgets, validation,
calendar calculations and event handlers remain native. The SVG icon and wordmark are
local and require no external fonts, texture downloads or extra dependencies.

- `ui_components.apply_visual_design`: load static local presentation assets (U03 s2, L04).
- `ui_components.render_chart`: style detached figures without changing plot data or global
  Matplotlib defaults (A09).
- `views/week._draw_day`: stable container keys connect decorative styling to day cards;
  the index keeps keys unique even when private appointments share the same times (U03 s5).
- `views/week.render_week`: a horizontal legend and scoped grid keep the week readable;
  narrow screens stack days, while intermediate screens can scroll the calendar locally.

Verify with `python -m unittest discover -s tests -v`, `python -m views.check_demo`,
and `python -m streamlit run app.py`. Restart Streamlit after theme configuration changes.
