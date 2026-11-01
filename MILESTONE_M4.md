<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# Milestone M4 report – ML and feedback loop

Status: in progress (checkpoint; M4 is not yet complete)

Implemented and checked: explicit ML features, chronological training rows without own-row or
same-start leakage, sample weights, seeded Random Forest, heuristic fallback, learned scoring,
privacy-aware friend counts, pending-feedback queries, demo-history deletion and rating cache
revision. All 116 unit tests pass at this checkpoint. The existing M3 UI is still wired to its
baseline scorer; M4 app verification follows once the new UI is connected.

Remaining: generate 60 synthetic profiles and histories; verify dataset sanity and training time;
enable onboarding demo history and showcase friends; wire model scoring into cards; implement
inline feedback and the session popup; run full tests, Streamlit AppTest and an actual server.

Decision: the cache key includes a monotonic rating revision alongside the specified rating
count. Correcting an existing rating does not change the count; this extra key ensures that
corrections and demo-history removal also invalidate the model. Saving an unchanged rating
does not retrain. No extra dependencies or work on M5–M8.
