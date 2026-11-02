<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-02, SPEC.md v1.0; reviewed by: pending -->
# Milestone M5 report – Own Web API

Status: done

## Acceptance criteria

1. ✅ The local Flask API exposes health, categories, an event list, event details and interaction
   writes. Streamlit displays **Event data: own API ✓** and uses the client for suggestion reads,
   details, confirmation/cancellation and attendance/ratings. A real temporary API process logged
   successful GET /events and POST /interactions requests from actual Streamlit widget actions.
2. ✅ Stopping that API led to a successful SQLite cancellation in **0.905 seconds including the
   UI rerun**; the storage operation itself took about **0.001 seconds**. The sidebar immediately
   showed the database fallback. Following offline actions skipped HTTP during the cooldown.
   Restarting the API and advancing the test clock past 30 seconds restored API reads/writes.
3. ✅ All specified route/status combinations are tested: health/categories/list/detail 200;
   event-range 400; unknown event 404; interaction 201, 400 and unknown user/event 404.
   Additional checks cover JSON 405/503, malformed JSON, bounds, filters, nullable/integer money,
   request connection closure, client transport failures and lost responses after a commit.

**144 unit tests pass.** Offline Streamlit feedback and planning regression harnesses pass.
The live API/AppTest harness verifies HTTP-based feedback still causes exactly one model retrain.
The documented API and Streamlit commands both started successfully. Final HTTP checks returned
200 for API health/events and Streamlit page/health. All servers started for the checks were stopped.

## Files created/changed

- New `api_server.py`: five routes, strict input validation, JSON responses/errors, per-request
  SQLite connections, preserving schema initialization when started before Streamlit.
- New `api_client.py`: Event conversion, exact time-range filtering, one bounded HTTP attempt,
  equivalent database fallback and recovery when a write committed but its response was lost.
- `database.py`: raw event dictionaries; object-typed frames that retain integer Rappen/nulls;
  shared interaction input checks; user existence lookup; optional SQLite URI support for tests.
- `config.py`: protocol fields, date/range/identifier bounds, 30-second health cache and labels.
- `ui_components.py`, `views/week.py`: API-aware event caches, reads/writes and sidebar status;
  a recent operation failure overrides a stale healthy cache without a second wait on rerun.
- New `tests/test_api.py`; README, study guide and AI usage record updated.
- Environment-only harnesses in `/workspace/weekwise-tools/`: `check_m5_live.py`,
  `check_m5_feedback.py`, `check_m5_regression.py`.

## How to check (about five minutes)

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m unittest discover -s tests -v
.venv/bin/python api_server.py
```

In a second terminal:

```bash
cd /workspace/V1-WeekWise
.venv/bin/python -m streamlit run app.py --server.headless true --server.port 8501
```

Open the app and complete onboarding if needed. The sidebar should show the API. Choose an event
in a gap, open Details and confirm it; the API terminal should log GET /events and POST /interactions.
Stop the API with Ctrl+C, then cancel the future confirmation. The action should work immediately
and the sidebar should show the database. Restart the API; after 30 seconds, the next UI rerun
detects it. Stop both processes when finished.

The API-only first start creates an empty schema; GET /events returns an empty list until the
first Streamlit page run prepares demo events. Existing profiles and history are preserved.

For the automated UI checks, stop any API occupying port 5001 first:

```bash
.venv/bin/python /workspace/weekwise-tools/check_m5_live.py
.venv/bin/python /workspace/weekwise-tools/check_m5_feedback.py
.venv/bin/python /workspace/weekwise-tools/check_m5_regression.py
```

These harnesses use temporary databases and do not alter the app's profile. The live harness
starts and stops only its own API processes; unit tests use in-memory SQLite and no network.

## Decisions I made

- `/categories` returns a `directions` list with key, name, colors and nested bundle ID/key/name
  lists. Optional event filters combine with AND; an unknown filter value returns an empty list.
- `get_event` returns `(Event or None, origin)`, consistent with the other client operations.
  Date-only API queries are widened when necessary, then narrowed to exact client datetime bounds.
- The UI caches health for 30 seconds. Each operation makes at most one HTTP attempt, with the
  configured two-second timeout. Failed operations suppress further probes for 30 seconds so the
  write-triggered rerun does not wait on the same unavailable API again.
- The event cache includes availability and returns origin along with events. A failure clears
  cached event results so restarting the API cannot retain a cached database fallback.
- The shared database resolves uncertain writes: if the requested state appeared since the
  attempt began, the client returns that committed row. An unchanged duplicate confirmation
  still fails validation. A lost rating response increments the model revision only once.
- Nullable event money/ID fields use an object-typed DataFrame to preserve SQL integers exactly;
  a test covers a Rappen integer above 2**53, which would otherwise lose precision as float.
- SQLite URI mode is an optional test seam for multiple connections to one in-memory database.
  Production still uses the existing SQLite file with its ten-second connection timeout.
- API CLI startup initializes only the schema; it neither resets data nor generates a local user.
  Importing the API module starts no server and opens no real database.

## Deviations from SPEC/CLAUDE.md

No functional scope deviation. No dependency changed. Invalid transport responses also fall back
when they contain malformed JSON or invalid event fields. Startup/storage failures use a friendly
JSON 503 response in addition to the specified status codes.

The existing cloud setup uses Python 3.11 via venv/uv with the exact dependency pins. AI citations
name the actual coding agent, OpenAI Codex (GPT-6), rather than the model in the example template.

## Course topics to study

| File → function | Topic | Course |
| --- | --- | --- |
| `api_server.py` → `create_app`, route functions | HTTP verbs, routes, JSON and status codes | L06, A06 |
| `api_client.py` → `get_events`, `save_interaction` | Requests, timeouts and failure handling | L06, A06 |
| `api_client.py` → `_fallback_interaction` | Uncertain delivery and shared committed state | L06, A07 |
| `database.py` → `get_event_records`, `validate_interaction_input` | Exact storage values and shared validation | L02, A07, A08 |
| `ui_components.py` → `api_available`, `load_events` | Cache invalidation, session state and reruns | L04 |

## Open questions for the team

No M5 blocker remains. Team review is pending. AppTest exercises widget handlers through
full-script reruns; event-detail tests requeue the open dialog before its next action. Manual
browser/layout review and the earlier Google/Outlook compatibility checks remain unperformed.

M6 rankings/model evaluation and M7 friend/invitation workflows were not started. Source changes
are committed locally and archived at `/workspace/weekwise-tools/weekwise-m5-source.tar.gz`.
No push was made; restoration in another cloud environment has not been verified.
