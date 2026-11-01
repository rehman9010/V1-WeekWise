<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
## Milestone M2 report – Calendar and gaps

Status: **done with open points** — implementation is complete; acceptance item 3 needs the team.

Acceptance criteria:

1. ✅ The actual Streamlit page renders seven day columns, the specified recurring classes,
   Thursday's “Mum's birthday” label and free-gap labels. The transparent reminder is ignored.
   Next/previous/Today navigation, all-day busy, title hiding and source fallback were exercised.
2. ✅ SPEC §10.3, five-versus-six-minute tolerance, recalculation after confirmation, and the
   ICS export/import round trip pass. Further tests cover recurrence exceptions/cancellations,
   Zurich DST, all-day exclusive end dates, HTTPS redirects, download limits, opt-in remembering,
   UTF-8 folding, escaped text and Google/Outlook query parameters.
3. ⚠️ Actual import into Google Calendar and Outlook, and reading a real secret Google feed,
   remain manual team checks. No account or private feed was provided; no success is claimed.

Files created/changed:

- M2 logic: `calendar_import.py`, `gaps.py`, `calendar_export.py`.
- Integration: `synthetic_data.py`, `config.py`, `database.py`, `app.py`, `ui_components.py`,
  `views/week.py`; demo calendar generated on first start.
- Tests: `tests/test_calendar_import.py`, `tests/test_calendar_export.py`, `tests/test_gaps.py`,
  `tests/test_foundation.py`, `tests/support.py`, `tests/test_project_rules.py`.
- Reconstructed M1 files after checkout loss: `models.py`, `categories.py`, `importers.py`,
  the foundation portions of the shared modules, `requirements.txt`, `.gitignore`,
  `sources.example.json`, `data/taxonomy.json`.
- Documentation: `README.md`, `STUDY_GUIDE.md`, `AI_USAGE.md`, `MILESTONE_M1.md`, this report.
  `SPEC.md` and `CLAUDE.md` are unchanged copies of the uploads.

How to check (≈5 min):

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/streamlit run app.py
```

Open the Week page. Check the seven columns, Thursday's all-day label, weekday appointment
times and free gaps. Use Next week, Previous week and Today. In **Your calendar**, switch on
all-day busy: Thursday has no free gaps. Switch on hidden titles: private titles become Busy.
For the manual compatibility check, open **Data sources and calendar export check**, download
the test ICS, import it into Google and Outlook, and compare its start/end with the displayed
Zurich time. Then enter the real secret feed only in the app, select **Use calendar**, and
compare a known private appointment. Leave Remember unchecked unless local persistence is wanted.

Validation performed in the current environment:

- Python 3.11.17 with the exact SPEC dependency pins; installed from the retained package cache.
- Unit-test command above: **84 tests passed**, using fixed clocks, mocked HTTP and in-memory
  databases. No skipped or expected-failure tests.
- `.venv/bin/python /workspace/weekwise-tools/check_m2_app.py`: fresh temporary database,
  actual Streamlit script execution, seven-column layout, navigation, privacy settings, ICS
  download availability, failed-link fallback, forgetting inputs, stable 570 events and no
  private calendar titles or secret link in the database. Unexpected HTTP calls fail this check.
- `.venv/bin/streamlit run app.py --server.headless=true --server.address=127.0.0.1
  --server.port=8501 --browser.gatherUsageStats=false`: server started successfully.
  Local HTTP requests verified health `200 / ok` and the HTML/asset entry point. The server was
  stopped afterward. AppTest supplies functional verification; no visual browser review is claimed.

Decisions I made (not specified in SPEC):

- Sidebar controls expose calendar input and privacy settings during M2. Dedicated onboarding
  and the core confirmation/suggestion flow remain M3 work. A test ICS download supports M2 review.
- Cache private parsed data only in memory, for five minutes and at most eight entries. Range,
  input, privacy switches, confirmed IDs and file modification times invalidate it.
- Limit redirects to five and reject HTTPS downgrades. Reject over 10,000 expanded occurrences
  per query. If all fallbacks fail, suppress free gaps and explain the unavailable calendar.
- Pass IDs of current-user confirmations explicitly to the pure parser. Only entries drawn
  separately from SQLite are suppressed; otherwise a recognized marker remains safely busy.
- Normalize a cancelled recurrence notice that omits DTSTART using its RECURRENCE-ID before
  handing it to the recurrence library. This prevents resurrecting the original appointment.
- If today's week falls outside the stored catalogue, show the closest available week.
- Use an isolated `venv` in the cloud environment; the application still targets Python 3.11.

Deviations from SPEC/CLAUDE.md:

- Reconstructed the lost M1 foundation during M2 because no earlier source checkout/commit was
  available. Current foundation tests are consolidated in `test_foundation.py`; they replace the
  lost module-by-module files. No byte-identical recovery is claimed.
- Keep past demo occurrences at startup until M4 supplies the historical interactions. Cleanup
  is implemented and tested separately. This preserves the M1 demo counts and M4 history input.
- AI headers use the actual OpenAI Codex (GPT-6) identity, as required by the honesty rule,
  rather than the Anthropic name in the example template. `tempfile`, `unittest.mock`, `copy`
  and `ast` support isolated tests and source-rule checks; no application dependency was added.

Course topics to study in this milestone:

| File | Function | Course code |
| --- | --- | --- |
| `gaps.py` | `merge_intervals`, `find_gaps_for_day` | U03 s1, s3–s5 |
| `calendar_import.py` | `fetch_ics_text`, `load_calendar` | A06, L06 |
| `calendar_export.py` | `build_ics`, `escape_ics_text`, `fold_ics_line` | U02 s3, U03 s1, A06 |
| `synthetic_data.py` | `write_demo_calendar` | U03 s1–s2, A06 |
| `database.py` | `get_calendar_confirmations` | A07, A08 |
| `views/week.py`, `ui_components.py` | grid and calendar controls | L04, A04 |

Open questions for the team:

- Confirm acceptance item 3 using your Google/Outlook accounts and an actual secret feed.
- Review the reconstructed foundation and M2 source; citations remain marked pending.

M3 has not been started. A source-only backup is retained outside the checkout; preservation
in a newly replaced cloud environment is not yet verified.
