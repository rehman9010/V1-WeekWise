<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# WeekWise study guide – through M2

Start with `models.py`, then `gaps.py`, `database.py`, the importers and calendar export.
Read `app.py`, `ui_components.py` and `views/week.py` last to see how those pieces are displayed.
Later ranking and machine-learning modules are not implemented yet.

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
