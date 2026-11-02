<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-02, SPEC.md v1.0; reviewed by: pending -->
# Milestone M6 report – Rankings and model evaluation

Status: done

## Acceptance criteria

1. ✅ Rankings shows Most popular, Most attended, Best rated and Most feedbacked for You and
   All users, overall or filtered by direction/bundle. Series aggregate across occurrences;
   cancelled rows are excluded. Best rated requires one local or three global ratings. Charts,
   aggregate tables, empty states and a direction overview are available without exposing users.
2. ✅ About the model shows ten comparison rows: Random Forest, Logistic Regression, KNN,
   Decision Tree and Gaussian Naive Bayes, each unscaled/scaled. It includes label/origin counts,
   validation-F1 bars, Random Forest test metrics/classification report and top-ten importances.
   Tests verify disjoint stratified 80/10/10 splits with seed 13, training-only scaler fitting,
   all three transforms, unweighted comparison and an evaluation forest fitted only on training.
3. ✅ Data sources displays placeholder counts and import history. Refresh updates all sources
   independently, retains referenced history, reports failures and clears relevant caches. Reset
   requires confirmation, clears profile/session/caches, regenerates the demo and reopens
   onboarding while preserving remembered private files. About WeekWise includes sources,
   privacy, course/team/version and the viewable/downloadable AI usage record.

**157 unit tests pass** (13 new M6 tests). Existing offline planning/calendar and feedback
AppTest regressions also pass. The M6 AppTest exercises all four rankings and category choices,
lazy model evaluation, ordinary-rerun reuse, new ratings and same-count edits, source refresh and
failure, reset/re-onboarding, private-file preservation and taxonomy-version recovery.

The isolated fixed-clock demo contained 674 eligible ratings: 539 train, 67 validation and 68
test rows. Its positive share was 50.74%; the evaluation forest scored test accuracy 0.809 and
macro F1 0.808. These are reproducible demo observations, not claims of real-user performance.
The generated ranking, comparison, importance and overview images were inspected for readability.

The actual Streamlit command started successfully on port 8501; page and health endpoints both
returned HTTP 200. The server was then stopped. HTTP checks verify server startup; AppTest
verifies execution and widget behavior against fresh temporary databases without external network.

## Files created/changed

- New `rankings.py` and `views/rankings.py`: fixed-fragment aggregate SQL, scope/category filters,
  metric tabs, tables and ranking/overview charts.
- `recommender.py`: comparison classifiers, stratified split, scaler, independent held-out report,
  feature importance table and two charts. Existing production recommendations remain available.
- `views/settings.py`: Data sources, About the model and About WeekWise tabs.
- `ui_components.py`: evaluation cache, explicit cache invalidation and confirmed reset controls.
- `database.py`: distinct placeholder-series counts and shared training-eligibility summary.
- `app.py`: Rankings navigation and explicit reset recovery after a taxonomy-version mismatch.
- `config.py`: evaluation/cache limits and shared chart settings.
- New `tests/test_rankings.py`, `tests/test_evaluation.py`; README, study guide and AI usage updated.
- Environment-only `/workspace/weekwise-tools/check_m6_app.py`, with generated inspection images
  in `/workspace/weekwise-tools/m6-charts/` and run logs under `/tmp/weekwise-m6-*.log`.

## How to check (about five minutes)

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python -m streamlit run app.py --server.headless true --server.port 8501
```

On this cloud host, use `MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache`
before these commands if the home directory is read-only. No new dependency is required.

1. Finish onboarding with demo history, dismiss pending feedback if shown, and open Rankings.
   Compare all four tabs, select Career and then a bundle, and expand the aggregate tables.
   Best rated shows its minimum. The overview always covers all three directions.
2. Open Settings & About → About the model. Check the ten rows, two scaling variants, train/val/test
   row counts, four test metrics, classification report and feature importance chart.
3. Open Data sources and choose Refresh event data. Three new import reports should appear;
   your ratings remain. About WeekWise displays the development record inside its expander.
4. If willing to discard local profile/history, tick the reset confirmation and click Reset demo
   data. The app returns to Welcome; remembered private calendar/source files are preserved.
5. Stop Streamlit with Ctrl+C. The local API is optional throughout.

For a non-destructive automated UI check using temporary data:

```bash
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache \
  .venv/bin/python /workspace/weekwise-tools/check_m6_app.py
```

## Decisions I made

- All-users scope is named `'all'`. Both category filters combine with AND, and unknown keys
  return an empty result. Sorting adds series ID after the specified value/count/title keys to
  keep otherwise-identical series deterministic. Unrated series are absent from feedback ranks.
- Category overview follows the glossary: its bars count current `confirmed` plans. Its table
  contains rating averages; the chart does not mix counts and stars on one axis. Missing ratings
  stay null. Most popular separately includes attended/no-show interactions as specified.
- `evaluate_models()` returns the specified ten-row DataFrame. Plain serializable `.attrs`
  carry the held-out report, split sizes and importances without exposing a model object to the
  UI table. Inadequate or unstratifiable labels produce an empty result and an explanation.
- Evaluation uses a separate unscaled Random Forest fitted on 80% of rows. Reporting test
  metrics from the production forest fitted on all rows would leak test examples. All ten
  comparison variants are unweighted; production retains the specified fivefold genuine weight.
- Evaluation caching uses the existing rating count plus revision, so edits with unchanged
  counts invalidate it too. It holds at most two reports for one hour. Source refresh/reset
  explicitly invalidate models because features can change even if rating counts stay constant.
- Streamlit 1.66 tracked tabs provide `.open`. Only the selected ranking metric is drawn, and
  evaluation runs only in About the model. Other lightweight Settings controls remain rendered.
- Reset closes the old cached SQLite connection before clearing its resource cache; private files
  are untouched. A taxonomy mismatch exposes the same confirmed recovery without automatic reset.
- About WeekWise uses “WeekWise project team” because individual member names were not supplied.

## Deviations from SPEC/CLAUDE.md

No feature-scope or dependency deviation. The existing venv/uv setup uses Python 3.11 and the
specified exact pins. AI citations name the actual agent, OpenAI Codex (GPT-6). Tests for model
evaluation live in `test_evaluation.py` alongside existing recommender/training test modules.

## Course topics to study

| File → function | Topic | Course |
| --- | --- | --- |
| `rankings.py` → `get_ranking`, `category_overview` | JOIN/GROUP BY/HAVING, bound parameters, null averages | A07, A08 |
| `rankings.py` → plot functions | Horizontal/grouped bars and labels | A09 |
| `recommender.py` → `_split_for_evaluation`, `evaluate_models` | Stratification, leakage prevention, RobustScaler | A10, A11 |
| `recommender.py` → `_classification_metrics`, plot functions | Macro metrics, classifier comparison and importances | A10, A11, A09 |
| `ui_components.py`, Settings views | Cache invalidation, tracked tabs and confirmed reset | L04 |

## Open questions for the team

No M6 blocker remains. Supply individual team names when ready and review the citations/code.
The chosen overview meaning is documented above. AppTest drives tracked tabs by their session
label because its Tab wrapper lacks a select method. Manual browser layout and native dialog
fragment behavior, plus earlier Google/Outlook compatibility checks, remain unperformed.

M7 friend/invitation workflows were not started. Source changes are saved in a local commit and
`/workspace/weekwise-tools/weekwise-m6-source.tar.gz`; no push was made. Restoration in another
cloud environment has not been verified.
