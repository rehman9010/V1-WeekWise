<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# WeekWise study guide – through M3

Start with `models.py`, then `gaps.py`, `database.py`, the importers and calendar export.
Read `app.py`, `ui_components.py` and `views/week.py` last to see how those pieces are displayed.
The baseline scorer is implemented; later ranking and machine-learning training are still pending.

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

Attendance/rating transitions are covered at the database layer, but their UI and model training
remain M4 work. The M3 scorer deliberately rejects a supplied trained model rather than silently
ignoring it.

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

Evaluation and held-out train/validation/test splitting belong to M6. M4's production forest fits
all eligible rows; its tests check behavior without presenting synthetic-data accuracy as evidence
of real-world recommendation quality.
