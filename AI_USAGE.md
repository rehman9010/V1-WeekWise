<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# AI usage record

| Date | Tool/model | Purpose | Output/files | Checks and team review |
| --- | --- | --- | --- | --- |
| 2026-11-01 | Planning/specification agent, exact model not supplied | Product planning and specification | Uploaded `SPEC.md`, `SPEC.pdf`, `CLAUDE.md`; taxonomy v1 defined in SPEC | User-provided documents; kept unchanged. Team confirms their own earlier AI-use details. |
| 2026-11-01 | OpenAI Codex (GPT-6), M0/M1 | Environment and original data foundation | Python environment, config, app, models, database, categories, importers, demo sources and tests | Earlier run reported 63 passing tests and an offline app check. The source checkout was lost after environment replacement; those files are not claimed to have been restored byte for byte. Team review pending. |
| 2026-11-01 | OpenAI Codex (GPT-6), M2 | Recover the foundation, implement calendar/gaps and verify the app | `calendar_import.py`, `calendar_export.py`, `gaps.py`, `views/week.py`, `ui_components.py`, `synthetic_data.py`, M1 foundation modules, `config.py`, `app.py`, `tests/*.py`, requirements, taxonomy, source example and project documentation | Current unit tests, isolated Streamlit AppTest and actual HTTP startup check; see `MILESTONE_M2.md`. Google/Outlook import and a real secret feed still require the team. |
| 2026-11-01 | OpenAI Codex (GPT-6), M3 | Implement onboarding and the planning MVP | `app.py`, `config.py`, `database.py`, `recommender.py`, `friends.py` (code generation only), `calendar_import.py` documentation, `ui_components.py`, `views/onboarding.py`, `views/week.py`, `views/settings.py`, `tests/test_profiles.py`, `tests/test_recommender.py`, README, study guide and M3 report | 104 unit tests; isolated Streamlit flows for onboarding, favorites, filters, confirm/export/cancel, settings and session restoration; actual HTTP server startup. Team review pending; external Google/Outlook compatibility remains a manual check. |

| 2026-11-01 | OpenAI Codex (GPT-6), M4 (in progress) | Train the recommender and implement feedback | `config.py`, `database.py`, `friends.py`, `recommender.py`, `ui_components.py`, `tests/test_training.py`, study guide and M4 report | First checkpoint: 116 unit tests pass. Synthetic histories and UI integration remain; see `MILESTONE_M4.md`. Team review pending. |

The citation headers identify the actual coding agent as OpenAI Codex (GPT-6), rather than
copying the Anthropic name in the specification's example. No team review is claimed: headers
remain `pending`, with team changes recorded as `none` until the team edits them.

AI is used during development only. WeekWise does not call an AI/LLM service at runtime.
The team must include AI on its list of aids in the project video. The voice-over must not be
generated. Add later AI use here, including any real-data taxonomy review.
