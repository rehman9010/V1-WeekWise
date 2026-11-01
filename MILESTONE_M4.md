<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# Milestone M4 report – ML and feedback loop

Status: done

## Acceptance criteria

1. ✅ Sixty deterministic synthetic users, preferences, symmetric friendships and interactions.
   At the fixed clock 2026-11-02 10:00 there are **674 rated interactions**, **50.7% positive**,
   covering all three directions. Feature construction plus fitting the specified 200-tree,
   depth-10 forest took **0.42 seconds** here (target: under three seconds).
   Fresh/upgrade checks across Sunday and midweek clocks also pass the sanity checks.
2. ✅ Controlled fixtures and the actual generated dataset produce different top threes for
   Sports and Career profiles scoring six candidates in the same gap. The model uses explicit
   features, weights, aligned one-hot columns and a heuristic fallback for insufficient or
   single-class data. Prior bundle averages exclude the current rating and same-start peers.
3. ✅ Demo history creates three ratings (5, 4, 2), one no-show and two pending events in the
   all-directions fixture. The popup opens after onboarding; Later suppresses it for the session;
   a fresh session reopens unanswered feedback. At most five oldest activities appear; a sixth
   remains for the next session. Limited input produces as much history as possible.
   Closing uses Streamlit's dismissal callback; Done/Later and new sessions were tested.
4. ✅ Popup and inline attendance → stars → persistence → toast work. Streamlit values 0 and 4
   save ratings 1 and 5. Instrumented AppTest verifies one training call per new or corrected
   rating, none for ordinary reruns, attendance alone or no-shows. Genuine local ratings have
   weight five; synthetic/local-demo rows have weight one. Details allows rating edits.

All **125 unit tests pass**. Two isolated offline AppTest harnesses pass, including onboarding,
calendar validation, favorites, filters, confirm/export/cancel, navigation and privacy.
Streamlit started on port 8501; both `/` and `/_stcore/health` returned HTTP 200.
The server was then stopped. No dependency was added or changed.

## Files created/changed

- `recommender.py`: features, labels, chronological weighted training, Random Forest,
  probability scoring with capped favorite boosts and reasons.
- `synthetic_data.py`, `config.py`: sixty users, hidden in-memory affinities, bounded sampling,
  friendships, attendance/ratings, sanity checks, local demo history and showcase plans.
- `database.py`, `friends.py`: training/pending/history reads, rating revision, targeted demo
  deletion, preserving synthetic upgrades, sharing-aware friend counts and seed-only social writes.
- `ui_components.py`, `app.py`, `views/week.py`, `views/onboarding.py`, `views/settings.py`:
  model cache, learned cards, history choice, one queued dialog per run, inline feedback, popup,
  rating correction, toast and confirmed demo-history removal.
- New `tests/test_training.py`, `tests/test_synthetic_data.py`; updated README, study guide and
  AI usage record. Original `SPEC.md` and `CLAUDE.md` remain unchanged.
- Environment-only harnesses: `/workspace/weekwise-tools/check_m4_app.py` and
  `/workspace/weekwise-tools/check_m4_regression.py`.

## How to check (about five minutes)

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python /workspace/weekwise-tools/check_m4_app.py
.venv/bin/python /workspace/weekwise-tools/check_m4_regression.py
.venv/bin/python -m streamlit run app.py --server.headless true --server.port 8501
```

On a fresh database, finish onboarding with all directions, demo calendar and **Load demo
history** checked. Click **Later**, reload into a new session, then answer one pending event with
✓ and stars and the other with ✗. Look for the toast and the historical week states.
Edit stars in Details. In Settings → Profile, confirm and clear demo history.
Returning to Week must keep the app working without recreating sample history.
Both harnesses perform these checks with temporary data, leaving your actual profile untouched.

The explicit command `.venv/bin/python synthetic_data.py` resets SQLite and regenerates the
whole demo, including users/histories. Stop the app first: this removes the local profile and
interactions. Normal startup upgrades M3 data while preserving it. Remembered private files
are retained by an explicit reset.

## Decisions I made

- The cache key includes a monotonic rating revision alongside the specified rating count.
  Correcting existing stars leaves the count unchanged; the revision covers corrections and
  deletion. Saving unchanged stars does not invalidate the model.
- `synthetic_setup_done` prevents duplicate seeds. An interrupted seed can be replaced while
  preserving genuine local profiles, ratings and private files; a test verifies this.
- Demo history reserves two recent eligible events for pending feedback before choosing the
  rated/no-show examples. Repeated setup preserves existing demo rows.
- The app dispatches dialogs after page rendering, preventing feedback and details from opening
  two dialogs in one run. Feedback stays open across its own writes.
- An explicit anchor fixes generation to its Monday at midnight unless `now` is also passed;
  ordinary first-start generation uses current Zurich time. Reproducibility tests fix both.
- The six showcase codes are literal SPEC values; generated codes retain the restricted alphabet.
  This preserves NOAH26, MIAK26 and JONAS6 despite O/I being excluded from random codes.

## Deviations from SPEC/CLAUDE.md

- The extra cache revision resolves the count-only key's inability to refresh corrected ratings.
- This cloud environment uses Python 3.11 via `venv`/`uv`, with the exact dependency pins.
- Citation headers identify the actual coding agent, OpenAI Codex (GPT-6), instead of copying
  the different provider/model in the specification's example.

## Course topics to study

| File → function | Topic | Course |
| --- | --- | --- |
| `recommender.py` → `event_feature_row`, `to_feature_matrix` | Features, target separation, one-hot alignment | L02, A08, A10 |
| `recommender.py` → `build_training_data`, `train_production_model` | Chronology, leakage, weights, seeded forest | U03 s1, A10 |
| `synthetic_data.py` → `_weighted_events`, `_synthetic_rating` | Bounded loops, RNG, sampling and noise | U02 s5, U03 s1, L02 |
| `database.py` → `get_pending_feedback`, `get_training_interactions_df` | Joins and parameterized queries | A07 |
| `ui_components.py` → `render_feedback_widget`, `cached_recommender` | State transitions, reruns, caching | L03, L04 |

## Open questions for the team

No M4 implementation blocker remains. Team code review is pending. Manual browser/layout review
and the earlier Google/Outlook import checks remain team checks; no private feed was supplied or
claimed tested. AppTest uses full-script reruns, so event-dialog tests explicitly requeue the
open details dialog before triggering its widget handler.

The own API is M5; Rankings/model evaluation is M6; Friends, invitation acceptance and invitation
pins are M7. M4 seeds showcase data but does not add those later workflows. No next milestone
was started.

The completed source is committed locally and archived at
`/workspace/weekwise-tools/weekwise-m4-source.tar.gz`. There was no push; restoration in another
cloud environment has not been verified. M4 is finished and needs no repeated implementation command.
