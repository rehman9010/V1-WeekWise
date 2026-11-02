<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-02, SPEC.md v1.0; reviewed by: pending -->
# Milestone M7 report – Friends and invitations

Status: done

## Acceptance criteria

1. ✅ Friends displays the local code/share text and adds `LUCA26`. Unknown, own and duplicate
   codes show clear errors. Friendships are stored symmetrically and survive a new session.
2. ✅ The fresh current-week demo shows at least two friends-going badges and Lena's pinned
   invitation. Noah's invitation is also visible. Pins precede normal scored suggestions without
   altering scores; current opted-in friend counts remain part of the recommendation features.
3. ✅ Confirmed future event details provide a friend multiselect and Send invitations action.
   Synthetic responses immediately follow focus/overlap rules and display `(simulated)`.
   Accepting Noah's invitation confirms the event and opens its normal ICS/Google/Outlook export.
4. ✅ Sharing defaults off for the local user. Turning it off excludes the owner from friends'
   upcoming-plan queries, event-detail names and friends-going counts. Private calendar entries
   are never part of the shared data. All these paths are exercised in tests.

**170 unit tests pass** (13 new M7 tests). A forced SQLite trigger failure confirms that saving
an invitation answer and confirming its activity roll back together. Tests cover recipient
authorization, duplicate sends, repeat answers, expiry, matching/mismatched focus, overlapping
and touching plans, existing confirmations, scoped queries, sharing and invitation priority.

The M7 AppTest passed the full offline widget flow, including adding Luca,
invalid-code messages, opt-in/out, accepting Noah and exporting, simulated answers, duplicate
sends, decline, non-fitting invitations, hidden friend plans/names and session restoration.
The existing planning/calendar, feedback and M6 evaluation/reset AppTest harnesses also pass.

The actual Streamlit process started on port 8501. Its page and health endpoint returned HTTP
200, and the process was stopped afterward. AppTest checks executed the real app script with
temporary fresh databases and mocked external network. HTTP checks independently verify startup.

## Files created/changed

- `friends.py`: code-based friendships, sharing accessors, public plans/names, received/sent
  queries, pending-invitation mapping, validated sending, deterministic simulation and responses.
- New `views/friends.py`: two-column profile/sharing/code UI, demo codes, friend plans and
  invitation history with fit indicators, Accept/Decline and Details buttons.
- `database.py`: extract the existing interaction state machine into
  `apply_interaction_in_transaction`; the normal wrapper still manages its own transaction.
- `recommender.py`: optional inviter metadata and stable invitation-first sorting.
- `ui_components.py`: inviter badges, shared friend names, calendar-fit assessment, invitation
  actions and friend selection/results in confirmed-event dialogs.
- `views/week.py`: reuse a pending-invitation map across eligible gaps.
- `app.py`: register Friends between Rankings and Settings.
- `config.py`: shared invitation badge colour.
- New `tests/test_friends.py`; README, study guide and AI usage record updated.
- Environment-only `/workspace/weekwise-tools/check_m7_app.py`; logs are in
  `/tmp/weekwise-m7-*.log`. Existing M4 showcase generation and social features were reused.

## How to check (about five minutes)

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python -m streamlit run app.py --server.headless true --server.port 8501
```

On a cloud host with a read-only home, prefix commands with
`MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache` for writable plotting caches.

1. Complete onboarding with Load demo history checked. Dismiss pending feedback with Later.
   In Week, find friends-going badges and “Lena invited you” among the first suggestions in a gap.
2. Open Friends. Copy the displayed code message. Add `LUCA26`, then try the same code, your own
   code and an unknown one to see the specific validation messages.
3. Accept Noah's invitation. Check Confirmed, Download .ics and provider export links in Details.
   In the same dialog, select Lena and/or Luca under Invite friends and send. Their saved response
   appears with `(simulated)`. Sending again does not create another invitation.
4. Return to Friends. Review sent outcomes, decline another incoming invitation, and toggle
   Share my WeekWise plans with friends. Upcoming friend plans have Details links.
5. Stop the app with Ctrl+C. The optional local API is not required for any of these steps.

For automated UI checks without changing the app's real profile:

```bash
MPLCONFIGDIR=/tmp/weekwise-mpl XDG_CACHE_HOME=/tmp/weekwise-cache \
  .venv/bin/python /workspace/weekwise-tools/check_m7_app.py
```

Existing installations keep their original demo-history choice and answers. Use Settings →
Data sources → confirmed Reset demo data only if you intend to discard your profile/history and
replay onboarding. The automated harness uses a temporary database instead.

## Decisions I made

- Friend codes are trimmed and uppercased before lookup. Generated codes still use the configured
  alphabet; explicit showcase codes such as `NOAH26` retain their specified spelling.
- `get_invitations()` returns a dictionary containing received/sent DataFrames, indexed by
  invitation ID, pending first. Multiple inviters to one occurrence share one suggestion badge.
- A complete recipient batch is validated before any insertion. Repeated sends reuse the unique
  sender/recipient/event row, and answered invitations retain their outcomes. Real recipients
  remain pending. Each simulated answer and its optional confirmation are committed atomically.
- Only the current local recipient can answer an incoming invitation. An identical repeated
  answer is a no-op; changing an answered invitation is rejected. Accepting an already-confirmed
  occurrence records the answer without duplicating the plan. A cancelled plan can be confirmed
  again. Later cancellation of a plan does not rewrite the historical invitation response.
- Simulation checks strict overlap with other confirmed WeekWise activities. Touching endpoints
  and the same occurrence are excluded from clashes. It does not inspect private calendars.
- Explicit invitations share one activity even when general plan sharing is off. Opt-out still
  removes the owner from presence counts, event-detail presence names and shared-plan lists.
- Invites stay subject to category/favorite filters and the usual gap eligibility rules. Every
  received invitation remains on Friends. Calendar clashes show a warning and allow an explicit
  acceptance; unavailable/fallback calendars are identified. Expired/full/closed invites can be
  declined but not accepted. The dialog reuses existing export and single-dialog behavior.
- Friend changes update prediction-time counts immediately on rerun. They do not force retraining
  or change the rating-count/revision cache contract. Source refresh still clears model caches.
- Invitation acceptance uses a single local database transaction, since the existing interaction
  API alone cannot atomically save an invitation response. Other planning/feedback actions keep
  their API/fallback path and pass the same regression tests.

## Deviations from SPEC/CLAUDE.md

No functional scope or dependency deviation. Existing M4 showcase data already provides the
required friends, plans and invitations and needed no regeneration change. The optional
`invited_by_map` scoring parameter carries the existing Suggestion invitation field. Python 3.11,
the pinned dependencies and source files `SPEC.md`/`CLAUDE.md` are unchanged. AI citations identify
the actual coding agent, OpenAI Codex (GPT-6).

## Course topics to study

| File → function | Topic | Course |
| --- | --- | --- |
| `friends.py` → `add_friend`, sharing queries | Sets, symmetric relations, SQL joins and bound parameters | L04, A07 |
| `friends.py` → `send_invitations`, `simulate_friend_response` | Whole-batch validation, unique constraints and deterministic decisions | L03, A07 |
| `friends.py`, `database.py` → response/interaction helpers | Atomic writes and why nested connection contexts can commit too early | A07 |
| `recommender.py` → `score_events` | Stable tuple-key sorting with unchanged numerical scores | A05, U03 s4 |
| `views/friends.py`, `ui_components.py` | State, privacy controls, shared dialogs and composition | L04 |

## Open questions for the team

No M7 blocker remains. Review the code/citations and the documented interaction choices.
AppTest performs whole-script reruns, so the dialog harness requeues the current event before
each action. Manual browser layout/native fragment behavior and the earlier Google/Outlook
service compatibility checks remain unperformed. Codes/invitations are local database features
as specified; multi-device messaging is outside this milestone.

M8 was not started. Source changes are saved in a local commit and the source-only archive
`/workspace/weekwise-tools/weekwise-m7-source.tar.gz`. No push was made; restoration in another
cloud environment has not been verified.
