<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-02, M8, SPEC.md v1.0; review pending -->
# Milestone M8 report – hardening, documentation and freeze checkpoint

**Status: done with open points** — local implementation and verification completed on
2026-11-02. Windows/macOS fresh-install results, manual browser/calendar-provider checks and
the real-input taxonomy review remain open. No real Unisport workbook, UniClubs/City endpoint
or credentials were supplied. Task R therefore has no arrived input to integrate.

## What changed

- Added `python -m views.check_demo`: a portable end-to-end acceptance test using the actual
  Streamlit scripts, an isolated fresh database, fixed clock and a real disposable Flask process.
  External requests are rejected; the process and database handles are closed after success or
  failure. Existing app data are untouched.
- Fixed the first-week demo: the previous three trips all occurred outside the first week, so
  **Beyond your gaps** was empty during the specified demo. One existing trip now starts on
  Saturday of the anchor week. Source counts and reproducibility are preserved. A unit regression
  and the UI scenario verify it. Existing saved catalogues are preserved until an explicit reset.
- Made **Demo data** provenance accurate: the sidebar reports actual placeholder-series and
  synthetic-user counts, including mixed or real-only activity sources. Updated the stale
  recommendation explanation to include ratings and social features.
- Completed setup, calendar, real-input, storage, reset, structure and troubleshooting docs;
  reviewed comments/docstrings and course tags; recorded the taxonomy freeze checkpoint.
- Added a GitHub Actions matrix for fresh Python 3.11 installations on Linux, Windows and macOS.
  The workflow is committed configuration; it has not been run on GitHub in this session.

## Acceptance criteria

The automated scenario uses Zurich local time **2026-11-02 10:00**, seed 13, no external services,
and its own temporary source files/database. Its localhost API traffic is real HTTP.

| SPEC M8 step | Result and evidence |
| --- | --- |
| 1 Fresh database → Welcome | ✅ Actual app initializes the fresh schema/demo and displays Welcome. AppTest checks the page; transient spinner appearance/layout still belongs to browser review. |
| 2 Alex, Sports + Career, demo history/calendar | ✅ Actual onboarding widgets; persisted profile and demo counts checked. |
| 3 Two prompts; attend + four stars, Later | ✅ Stored rating is 4, model retrains once for that change, other event remains pending. |
| 4 Current week, top three, match %, friends, Lena pin | ✅ Actual cards are capped at three per gap; scores and at least two social badges are present. |
| 5 Confirm, ICS, Google link, conflicts disappear | ✅ Confirmation removes visible overlapping suggestions. Actual UI export text round-trips with correct times/ID; generated Google URL/title checked. ⚠️ Opening/saving in Google or Outlook has not been manually tested. |
| 6 Racket sports and clear | ✅ Matching visible course IDs and public links checked; filter cleared through its widget. |
| 7 Next/back/Today and Beyond | ✅ Navigation dates checked; first-week Beyond displays the newly positioned demo trip. |
| 8 Career rankings for You and All users | ✅ Both tables contain data and all three page charts render. |
| 9 Add Luca, invite Lena, accept Noah | ✅ Actual handlers; labelled simulated response, stored confirmation and export verified. |
| 10 About the model | ✅ Ten comparison variants, held-out metrics/report and both charts render. |
| 11 API start and shutdown fallback | ✅ Separate production Flask factory on a free loopback port; sidebar/API GET/POST confirmed, then immediate SQLite fallback after stopping it. Health cache cleared only to avoid waiting for startup discovery. |
| 12 Refresh sources | ✅ Three import-log entries added and rating revision/history preserved. |
| 13 New session returns Later event | ✅ Exactly the deferred event reappears; answered event stays answered and profile persists. Private demo-calendar titles are absent from SQLite. |
| 14 All tests and project rules | ✅ 174 unit/rule tests pass, plus the complete acceptance scenario. |

AppTest operates on real widget handlers but is not a browser layout or accessibility audit.
Dialog actions requeue their dialog because AppTest executes whole scripts; tracked tabs are
selected through their supported session-state labels. Provider links are inspected offline.
The scenario is separate from the network-free `tests/` suite because it needs localhost sockets.

## Verification performed

| Check | Actual result |
| --- | --- |
| Clean Linux environment | New `/tmp/weekwise-m8-env`, CPython 3.11.17; all pinned requirements installed offline from the existing package cache (56 packages including transitive dependencies). |
| Dependency consistency | `uv pip check --python /tmp/weekwise-m8-env/bin/python`: all installed packages compatible. |
| Unit/rule suite in fresh environment | `python -m unittest discover -s tests -v`: **174 passed**, about 23 seconds. |
| Complete M8 scenario in fresh environment | `python -m views.check_demo`: **passed**, about 40 seconds; local API child stopped by cleanup. |
| Actual Streamlit server | Started with `python -m streamlit run app.py --server.address 127.0.0.1 --server.headless true` on a free test port. `/` returned **200**, `/_stcore/health` returned **200 / ok**. Owned process then stopped. App behavior itself was exercised by AppTest above. |
| Comment/rule review | 38 Python modules, 629 function/class docstrings checked; all production modules and new scenario helpers carry course tags. Import boundaries, citations, line lengths and SQL-format checks pass. |
| Original input documents | `SPEC.md` and `CLAUDE.md` unchanged. No dependency added or changed. |
| Native Windows/macOS | **Not run**: this environment is Linux. Matrix workflow and platform instructions are supplied; successful native results remain required before claiming full M8 acceptance. |

Local detailed logs: `/tmp/weekwise-m8-tests.log`, `/tmp/weekwise-m8-demo.log`,
`/tmp/weekwise-m8-streamlit.log`. These are temporary verification evidence, not repository files.
All processes started for verification have been stopped.

## Taxonomy and real-data checkpoint

Taxonomy **v1** remains marked `frozen: true`. No keys, keyword lists or mappings were changed.
The source file SHA-256 is
`1c74823187f3ec952725abfb0f3e01d9a3a8bf2ac05e1102cf20784ff7bb5e46`.

| Available placeholder source | Series | Other | Share |
| --- | ---: | ---: | ---: |
| Unisport | 40 | 1 | 2.5% |
| UniClubs | 60 | 0 | 0% |
| City | 30 | 0 | 0% |
| Total | 130 | 1 | 0.8% |

These are distinct demo series, not occurrence-weighted or real-world quality statistics. The
taxonomy validator verifies unique keys, three directions and exactly one Other bundle per
direction. No evidence warrants changing the frozen taxonomy based on these invented examples.
When actual inputs arrive, follow README's task R procedure: inspect headers/payloads, adapt only
the required mappings, review skipped rows and real Other assignments, then record justified
keyword changes without renaming/deleting keys. The real-data review is explicitly still pending.

## Files created/changed

Created:
`views/check_demo.py`, `views/demo_scenario.py`, `tests/test_data_status.py`,
`.github/workflows/check.yml`, `MILESTONE_M8.md`.

Changed:
`app.py`, `config.py`, `database.py`, `models.py`, `synthetic_data.py`, `ui_components.py`,
`tests/test_synthetic_data.py`, `README.md`, `STUDY_GUIDE.md`, `AI_USAGE.md`.

## How to check (approximately five minutes after dependency installation)

From the checkout in the activated Python 3.11 environment:

```bash
python -m unittest discover -s tests -v
python -m views.check_demo
python -m streamlit run app.py
```

On the prepared cloud host, use `.venv/bin/python` and prefix with
`MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache` if required by the writable-home
restriction. The portable scenario creates and removes its own data; it never resets your app.

For a manual demo, use a fresh disposable checkout, or back up first and explicitly reset through
Settings. Onboard Alex with Sports + Career and demo history; answer one popup with four stars and
choose Later for the other. Check Lena's pin, confirm a suggestion and inspect its exports; filter
Racket sports and navigate Next/back/Today. Open Beyond, Career rankings, Friends (add LUCA26,
invite Lena, accept Noah), and Settings → About the model. In a second terminal run
`python api_server.py`; after up to 30 seconds rerun/open a page to see the API status. Stop that
process with `Ctrl+C`, open event Details and verify database fallback. Refresh Data sources,
then open a new browser session to check the pending popup. Stop Streamlit with `Ctrl+C`.
The automated fixed-clock scenario remains reproducible after the calendar date changes.

## Decisions I made (not specified in SPEC)

- Keep the acceptance runner under `views/` to obey the Streamlit import restriction, without
  registering it as a page. A small launcher applies temporary paths before imports capture
  default arguments. A direct unisolated scenario invocation is rejected.
- Use a temporary free API port and a bounded child-process lifecycle instead of requiring the
  user's usual API process. The normal application has no new background processes.
- Move the existing middle trip to fixture index 24 (first Saturday); retain three trips and the
  original total of 60 club events. Do not replace existing saved catalogues automatically.
- Show actual demo counts in the sidebar instead of a fixed placeholder description.
- Preserve the current frozen taxonomy with a documented checksum, since no real input exists.

## Deviations from SPEC/CLAUDE.md

- Full native Windows/macOS and manual provider/browser acceptance cannot be established here;
  it remains open, with automation and instructions prepared. No stretch items were started.
- The test launcher uses delayed imports to isolate definition-time default paths. Standard
  library `subprocess`, `socket`, temporary-directory and mocking facilities are test-only
  infrastructure extras. No new third-party dependency, custom production decorator, threading,
  background scheduler, SQL architecture exception or product behavior override was introduced.
- The API acceptance child uses the production `create_app` factory with a disposable database
  and fixed clock; ordinary `python api_server.py` remains the documented manual entry point.

## Course topics to study in this milestone

| File → function | Topic |
| --- | --- |
| `synthetic_data.py` → `_club_rows` | U02 s5, L03: reproducible data and an explicit branch |
| `database.py` → `get_demo_counts` | A07: aggregate queries without join-induced overcounting |
| `ui_components.py` → `render_sidebar_status` | L04: derive visible status from saved state |
| `tests/test_data_status.py` → mixed-source tests | A07, U05 s2: independent fixtures and assertions |
| `views/check_demo.py` → `isolate_paths` | U03 s2: module scope and default-argument binding |
| `views/demo_scenario.py` → `DemoScenario` | U05 s2, L04: inheritance and sequential state verification |
| `views/demo_scenario.py` → `check_api_and_fallback`, `check_export` | A06, L06: real HTTP boundaries, failure handling and calendar conversion |

## Open questions for the team

- Run/inspect the Windows and macOS matrix jobs and record native installation results.
- Supply real source inputs when available for task R and the real-event taxonomy review.
- Manually check browser layout (including narrow-screen Day mode) and Google/Outlook import,
  compose-link times and cancellation behavior using disposable calendar entries.
- Confirm individual team names and AI-citation format, then complete the pending code review.

This is a code freeze checkpoint with these explicit external checks outstanding. There was no
deployment, remote push or public sharing of data.
