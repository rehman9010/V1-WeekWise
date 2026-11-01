<!-- AI CITATION: OpenAI Codex (GPT-6), 2026-11-01, from SPEC.md v1.0; reviewed by: pending -->
# M1 recovery note

The original M1 turn implemented the data foundation and reported 63 passing tests, a successful
Streamlit functional check, and a successful HTTP startup check. During the interrupted M2 turn,
the environment was replaced and the repository files were absent. There were no earlier local
commits from which to restore them. Uploaded SPEC.md and CLAUDE.md were available separately.

M1 was reconstructed during M2 from those unchanged documents. The current checks verify:

- all specified domain classes and exact monetary conversions;
- the thirteen-table SQLite schema, foreign keys, transactional upserts and joined queries;
- the frozen 23-bundle taxonomy and deterministic categorization;
- Excel and JSON activity import, normalization, rejection and offline fallback behavior;
- reproducible 480 / 60 / 30 demo occurrence counts and stable IDs across imports;
- preservation of events referenced by history or invitations during cleanup.

See `tests/test_foundation.py` and `MILESTONE_M2.md` for the current implementation and validation.
The seven-column M2 week view replaces M1's temporary event table. Past demo occurrences remain
stored until M4 adds the histories needed before applying startup cleanup. Real source integration
still awaits the team's inputs. This note records recovery, not a byte-identical restoration.
