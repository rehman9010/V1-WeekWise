<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
## Milestone M3 report – Onboarding and weekly planning MVP

Status: **done with open points** — all M3 acceptance checks pass locally; external calendar
compatibility remains the manual team check carried forward from M2.

Acceptance criteria:

1. ✅ Three-step onboarding validates name/focus, tests an unchanged link or parses an upload,
   and opens the current week with labeled, color-coded suggestions. The automated path completed
   in 1.2 seconds, including invalid-input checks. Demo works without network or credentials.
2. ✅ Confirming one occurrence creates a confirmed block and removes competing suggestions.
   The dialog offers ICS, Google, Outlook and source links. Cancellation restores the available
   suggestions and offers the cancellation file and manual-delete hint. Favorites work at all
   three levels. Known exported IDs remain recognized after cancellation.
3. ✅ Bundle filtering shows every fitting match with source links, up to the stated cap.
   Clear returns to top three plus dropdown. Both checkbox and sports multiselect controls,
   overflow messaging and favorite-or-bundle semantics were checked.
4. ✅ An invalid link produces a friendly message and demo fallback. During onboarding it
   prevents Finish until a successful test or an explicit switch to Demo. In Calendar settings
   a failed source visibly falls back to Demo. Secret URLs are absent from messages and SQLite.

Files created/changed:

- `database.py`: atomic onboarding, profile updates, favorite validation, interaction reads,
  full state-machine persistence and known export IDs.
- `recommender.py`: the exact M3 heuristic, capped favorite bonus, stable ranking and OR filters.
- `friends.py`: only the unique friend-code generator required for new profiles.
- `app.py`, `views/onboarding.py`, `views/week.py`, `views/settings.py`, `ui_components.py`:
  navigation, onboarding, cards/dialogs, confirm/export/cancel, filters, Week/Day layout, Beyond
  your gaps, Profile and Calendar settings, caching and privacy controls.
- `config.py`: M3 thresholds and shared labels/formulas; `calendar_import.py`: known-ID
  contract documentation now includes locally cancelled exports.
- `tests/test_profiles.py`, `tests/test_recommender.py`: new regression and behavior tests.
- `README.md`, `STUDY_GUIDE.md`, `AI_USAGE.md`, this report. SPEC.md and CLAUDE.md are unchanged.

How to check (≈5 min):

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/streamlit run app.py
```

On a fresh database, select Next, enter a name and focus, select Next, keep Demo and Finish.
On the Week page choose a future suggestion → Details → favorite it → Confirm. Check the new
confirmed block and missing competing options, then download its ICS or open a compose link.
Cancel it in Details and check the restored suggestions and cancellation download. Choose a
bundle filter and clear it; try More options and Layout → Day. Open Beyond your gaps to see the
placement reason and warning. In Settings & About, save a new focus/name and test an invalid
calendar link. The Calendar tab also provides title hiding, all-day busy and Forget controls.

Validation performed:

- **104 unit tests passed**, zero skipped/expected failures, with Python 3.11.17 and unchanged
  pinned dependencies. Tests use fixed clocks, in-memory databases, temporary files and mocked
  HTTP. The state-machine tests cover valid and invalid transitions, exact time boundaries,
  rating constraints, reconfirmation identity and cancellation with an exported feed entry.
- `.venv/bin/python /workspace/weekwise-tools/check_m3_app.py` passed against a fresh temporary
  database, blocking unexpected HTTP requests. It exercised invalid/valid focus, failed and
  changed link tests, malformed/valid uploads, demo completion, top-three/dropdown behavior,
  three favorite kinds, confirmation/export/cancellation, competing-slot removal/restoration,
  all-match/capped filters, sports multiselect, Beyond confirmation, profile edits, fallback,
  privacy switches, day layout, navigation and a second session with no duplicate user/events.
- AppTest requests whole scripts. The harness requeues the already-open dialog before each
  dialog-widget action so the real handler executes; it does not substitute database writes.
  Browser rendering and provider-account imports were not tested.
- Started the actual server with:

```bash
.venv/bin/streamlit run app.py --server.headless=true --server.address=127.0.0.1 --server.port=8501 --browser.gatherUsageStats=false
```

  Local HTTP checks returned health `200 / ok` and the Streamlit HTML/asset entry point.
  The server was stopped after verification.

Decisions I made (not specified in SPEC):

- Initially select all three focus directions, while enforcing at least one on save.
- Keep the allowed demo-history checkbox checked but inactive until the feedback milestone.
  The UI explains that history becomes available with attendance/rating features.
- Implement the complete database interaction state machine now so its single write API is
  consistent and tested; expose only confirm/cancel in M3.
- Use `friends.generate_friend_code()` now for onboarding, leaving social features for M7.
- Validate onboarding again at Finish, and save the user, directions and non-secret flags in
  one transaction. A failed or changed private input cannot silently complete onboarding.
- Preserve baseline tie order and retain session choices across page navigation. Public event
  caches expire after one minute and are cleared after source imports; confirmations and
  profiles are read fresh. No trained model or model cache is introduced.
- Recognize every locally stored export ID, including cancelled occurrences, per SPEC §9.2.
  Confirmed blocks come from the database. This extends M2's conservative confirmation-only
  caller policy so cancellation still frees the slot before the external feed refreshes.

Deviations from SPEC/CLAUDE.md:

- Continue M2's documented deferral of startup history cleanup until M4 creates demo history.
- Use the actual OpenAI Codex (GPT-6) identity in citation headers. No dependency was added.
- Cloud UI checks remain outside the checkout to respect the project's Streamlit import
  boundaries; the committed unit tests cover the pure logic and data contracts.

Course topics to study in this milestone:

| File | Function | Course code |
| --- | --- | --- |
| `database.py` | `complete_onboarding`, `add_favorite`, `save_interaction` | A07, L03 |
| `recommender.py` | `heuristic_score`, `favorite_boost`, `filter_suggestions` | L02, L04 |
| `friends.py` | `generate_friend_code` | U02 s5, U03 s1 |
| `ui_components.py` | cards, dialogs, top-three slicing and dropdown | L04, U03 s5 |
| `views/onboarding.py`, `views/settings.py` | staged input and saved preferences | A04, L04 |
| `views/week.py` | gap recomputation and view composition | L04, L05 |

Open questions for the team:

- Perform the Google/Outlook account import and real secret-feed checks carried from M2.
- Review the code and milestone behavior; AI citation review status remains pending.

M4 has not been started. No model training, attendance/rating UI, synthetic user history,
Rankings or Friends page is included. A local source backup is retained outside the checkout.
