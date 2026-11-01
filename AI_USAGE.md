<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# AI usage record

| Date | Tool/model | Purpose | Output/files | Checks and team review |
| --- | --- | --- | --- | --- |
| 2026-11-01 | Planning/specification agent, exact model not supplied | Product planning and specification | Uploaded `SPEC.md`, `SPEC.pdf`, `CLAUDE.md`; taxonomy v1 defined in SPEC | User-provided documents; kept unchanged. Team confirms their own earlier AI-use details. |
| 2026-11-01 | OpenAI Codex (GPT-6), M0/M1 | Environment and original data foundation | Python environment, config, app, models, database, categories, importers, demo sources and tests | Earlier run reported 63 passing tests and an offline app check. The source checkout was lost after environment replacement; those files are not claimed to have been restored byte for byte. Team review pending. |
| 2026-11-01 | OpenAI Codex (GPT-6), M2 | Recover the foundation, implement calendar/gaps and verify the app | `calendar_import.py`, `calendar_export.py`, `gaps.py`, `views/week.py`, `ui_components.py`, `synthetic_data.py`, M1 foundation modules, `config.py`, `app.py`, `tests/*.py`, requirements, taxonomy, source example and project documentation | Current unit tests, isolated Streamlit AppTest and actual HTTP startup check; see `MILESTONE_M2.md`. Google/Outlook import and a real secret feed still require the team. |

The citation headers identify the actual coding agent as OpenAI Codex (GPT-6), rather than
copying the Anthropic name in the specification's example. No team review is claimed: headers
remain `pending`, with team changes recorded as `none` until the team edits them.

AI is used during development only. WeekWise does not call an AI/LLM service at runtime.
The team must include AI on its list of aids in the project video. The voice-over must not be
generated. Add later AI use here, including any real-data taxonomy review.
