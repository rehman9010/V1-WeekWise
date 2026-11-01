# CLAUDE.md – Binding rules for the WeekWise coding agent

You are the coding agent for **WeekWise**, the group project of the HSG course *Fundamentals and Methods of Computer Science* (FCS-BWL, autumn 2026). You write all of the code. The team will read that code to prepare for the exam, so **readability beats cleverness everywhere**. Everything in this file is binding unless the team explicitly tells you otherwise in chat.

Version 1.0 · 1 November 2026 · companion of `SPEC.md` v1.0

---

## 1 Sources of truth

1. **`SPEC.md`** defines *what* WeekWise does: behaviour, data, algorithms, UI, milestones. Its section numbers are cited here as SPEC §n.
2. **This file** defines *how* the code is written and how you work.
3. If `SPEC.md` and this file contradict each other, or the spec is ambiguous in a way that changes behaviour, **stop and ask the team**. Do not guess on behaviour.
4. Small technical choices the spec leaves open (a helper's name, the wording of a caption) you decide yourself and list in the milestone report.
5. You do not need the course files: the relevant course rules are digested in SPEC.md and here. Never invent course content; if you cite a course topic in a comment, use the codes in §8.
6. Do not edit `SPEC.md` or `CLAUDE.md` unless the team asks you to.

---

## 2 Working protocol

* Build **one milestone at a time, in order** (M0 → M8, overview in §13, details and acceptance criteria in SPEC §20). Do not start the next milestone until the team says so. Task R (real-data integration) is done when the team hands you real input.
* Before starting a milestone: re-read the SPEC sections it names. Plan briefly in your reply (files, functions, tests), then implement.
* The app must start and work at the end of **every** milestone (`streamlit run app.py`, fresh database, no network). Never leave the repository broken.
* Write the tests for a module in the same milestone as the module.
* At the end of every milestone, run all tests and the app, then deliver a **milestone report** in this format:

```
## Milestone Mx report – <title>
Status: done / done with open points
Acceptance criteria: 1 ✅ … 2 ✅ … 3 ⚠️ (explain)
Files created/changed: <list>
How to check (≈5 min): <exact commands and clicks>
Decisions I made (not specified in SPEC): <list>
Deviations from SPEC/CLAUDE.md: <list or "none">
Course topics to study in this milestone: <file → function → course code>
Open questions for the team: <list>
```

* Update `STUDY_GUIDE.md` (new files/functions → course topics) and `AI_USAGE.md` (one row for the milestone: date, your model name, files) in every milestone.

---

## 3 Environment and commands

* **Python 3.11** (the course's Anaconda version). Do not use features newer than 3.11.
* Dependencies exactly as pinned in `requirements.txt` (SPEC §19). Do not add a dependency without asking the team; if you believe one is needed, explain why in the report.

```bash
conda create -n weekwise python=3.11
conda activate weekwise
pip install -r requirements.txt
streamlit run app.py                     # the app (required)
python api_server.py                     # own API, second terminal (optional)
python -m unittest discover -s tests -v  # all tests
python synthetic_data.py                 # regenerate all demo data
python importers.py                      # run the importers and print a report
```

---

## 4 Toolbox – what you may use

The team decided: build as much as possible with what the course taught, so that reading the code doubles as exam preparation. Extras only where the app needs them.

### Tier A – Python language features taught (use freely)

| Area | Features |
|---|---|
| Basics | variables; `int`, `float`, `str`, `bool`; arithmetic incl. `//`, `%`, `**`; comparisons and chained comparisons; f-strings with format specifiers; escape sequences; triple-quoted strings; type conversion |
| Control flow | `if/elif/else`; conditional expression; `while`; `for`; `range` (1–3 args); `break`/`continue`; `and`/`or`/`not`; sentinel loops; recursion |
| Functions | `def`; docstrings; `return` (≠ `print`); default parameters; keyword arguments; `*args`; unpacking with `*`; local scope; `lambda`; `map`/`filter`; `min`/`max`/`sorted(key=…)` |
| Data structures | lists (methods, slicing, `copy()` vs reference, `sort()` vs `sorted()`); tuples; unpacking; `enumerate`; `del`; dictionaries; sets; **single-level** list comprehensions; generator expressions |
| OOP | classes; `__init__`; instance and class attributes; constants in `UPPER_CASE`; methods; `@property` + setter with validation; `_internal` names; `__str__`/`__repr__`; composition; inheritance; `super()`; overriding; polymorphism; `isinstance` |
| Errors | `raise ValueError(...)`; `try/except` with specific exceptions; `unittest.TestCase` assertions |

### Tier B – Standard library taught

`math` (incl. `isclose`), `random` (`seed`, `randrange`, `choice`, `shuffle`; a `random.Random(seed)` object with the same methods is allowed for reproducibility), `statistics`, `datetime` (`date`, `datetime`, `time`, `timedelta`, `strftime`/`strptime`), `decimal.Decimal`, `pathlib.Path`, `json`, `sqlite3`, `os`, `urllib.parse`, `io`, `itertools`, `unittest`, `zoneinfo` (one constant: `ZoneInfo(TIMEZONE_NAME)`).

### Tier C – Third-party libraries taught

| Library | Use | Allowed in |
|---|---|---|
| `streamlit` | the whole UI, incl. built-in widgets (`st.columns`, `st.container`, `st.button`, `st.selectbox`, `st.multiselect`, `st.checkbox`, `st.radio`, `st.pills`, `st.segmented_control`, `st.text_input`, `st.file_uploader`, `st.expander`, `st.tabs`, `st.sidebar`, `st.session_state`, `st.dialog`, `st.feedback`, `st.badge`, `st.download_button`, `st.link_button`, `st.pyplot`, `st.dataframe`, `st.toast`, `st.cache_data`, `st.cache_resource`, `st.navigation`/`st.Page`) | `app.py`, `views/*.py`, `ui_components.py` only |
| `requests` | `get`/`post` with `params`, `headers`, `json`, `timeout` | `api_client.py`, `importers.py`, `calendar_import.py` |
| `python-dateutil` | `dateutil.parser.parse` for API date strings | `importers.py`, `calendar_import.py` |
| `pandas` (alias `pd`) | DataFrames, `read_sql`, `read_excel`, selection, `groupby`/`agg`, `merge`, `get_dummies`, sorting | any module that needs tables |
| `numpy` (alias `np`) | arrays where scikit-learn needs them | `recommender.py` |
| `matplotlib.pyplot` (alias `plt`) | bar charts; functions **return** a `Figure` | `rankings.py`, `recommender.py` |
| `seaborn` (alias `sns`) | optional charts | `rankings.py`, `recommender.py` |
| `scikit-learn` | `train_test_split`, the five classifiers, `RobustScaler`, metrics | `recommender.py` only |

### Tier D – Accepted extras (not taught, each justified)

| Extra | Why | Keep it simple by |
|---|---|---|
| `icalendar` | Robust reading of the student's `.ics` (time zones, all-day, escaping) | Reading only – **writing `.ics` uses plain f-strings** |
| `recurring-ical-events` | Expanding repeating calendar entries (RRULE, exceptions) | One call: `recurring_ical_events.of(cal).between(a, b)` |
| `flask` | The team's own small Web API | A handful of routes, one decorator each, JSON in/out |
| `openpyxl` | Engine behind `pd.read_excel` / `to_excel` | Never imported directly |
| `tzdata` | Time-zone data for `zoneinfo` on Windows | Never imported directly |

`icalendar` and `recurring_ical_events` only in `calendar_import.py`; `flask` only in `api_server.py`.

### Tier E – Do not use

* `async`/`await`, threading, multiprocessing, background schedulers.
* Custom decorators, metaclasses, abstract base classes, descriptors. Allowed decorators: `@property`/`.setter`, `@st.cache_data`, `@st.cache_resource`, `@st.dialog`, Flask's `@app.route` / `@app.get` / `@app.post` – each explained in a comment at first use.
* `dataclasses`, `typing` generics, protocols, `pydantic`, type-hint-heavy code. Write explicit `__init__` methods; document types in docstrings.
* ORMs (SQLAlchemy etc.) – plain `sqlite3` and SQL strings.
* Walrus operator `:=`, `match` statements, nested or multi-level comprehensions, dictionary/set comprehensions with conditions that need decoding, heavy regular expressions (`re` only if unavoidable and commented), one-liners that need a comment to understand.
* Wildcard imports; shadowing built-ins (`list`, `sum`, `set`, `id`, `type`, `input`, `max`, `min`, `filter`, `map`, `range`, `str`, `dict`); `global`.
* Custom HTML, JavaScript or CSS; `unsafe_allow_html`; third-party Streamlit components (incl. `streamlit-calendar`); Plotly.
* Any runtime call to an LLM or third-party AI service.
* `print` for program output outside `if __name__ == '__main__':` blocks and command-line scripts (`importers.py`, `synthetic_data.py`). Logic functions **return** values.
* Saving trained models to files (`pickle`, `joblib`) – retraining is fast and cached.

---

## 5 Coding conventions

### 5.1 Layout and naming

* PEP 8: 4-space indentation; spaces around `=` and binary operators, none around keyword-argument `=`; lines ≤ 100 characters; two blank lines between top-level definitions.
* **Single quotes** for strings; triple double quotes for docstrings; f-strings for formatting.
* `snake_case` for variables and functions, `CapWords` for classes, `UPPER_CASE` for constants, leading `_` for internal attributes and helper functions.
* **No magic numbers or strings**: every threshold, colour, path, URL and mapping comes from `config.py`. `0`, `1`, `''` and obvious unit conversions (`60` minutes per hour) are fine.
* Imports at the top: standard library, then third-party, then own modules, separated by a blank line; standard aliases `pd`, `np`, `plt`, `sns`; import own modules as modules (`import gaps`, then `gaps.find_gaps_for_day(...)`) so the origin of each function is visible. Exception: constants may be imported by name (`from config import MIN_GAP_MINUTES, OVERLAP_TOLERANCE_MINUTES`; Course: U03 s2 – `from … import`), never with `*`.
* Names and signatures as in SPEC §4.3.

### 5.2 Docstrings and comments (grading requirement 6)

* **Every** module, class, method and function has a docstring. First line: what it does, in one sentence. Then parameters and return value in the style of the course (Assignment 06 `Connection` class):

```python
def find_gaps_for_day(day, busy_intervals, window_start=DAY_WINDOW_START,
                      window_end=DAY_WINDOW_END, min_gap_minutes=MIN_GAP_MINUTES):
    """Return the free gaps of one day that are long enough for an activity.

    A gap is a period inside the day window without any busy interval that
    lasts at least min_gap_minutes (SPEC §10).

    :param day: datetime.date of the day to analyse
    :param busy_intervals: list of (start, end) datetime tuples, in any order
    :param window_start: datetime.time at which the day window opens
    :param window_end: datetime.time at which the day window closes
    :param min_gap_minutes: minimum length of a gap in minutes
    :return: list of Gap objects, sorted by start time
    """
```

* Comments explain **why**, not only what. Comment every non-obvious decision, every business rule (with its SPEC section) and every sanity check.
* Where it fits, mark the course's three phases with `# initialization phase`, `# processing phase`, `# termination phase` (U03 s1).
* `# Course:` tags [D-14]: at the **first** use of a course concept in each file, add a short tag, e.g. `# Course: U03 s5 – enumerate()` or `# Course: A07 – ? placeholders with params tuple`. Use only the codes in §8. Do not tag every line.
* No commented-out code, no TODOs in delivered milestones (list open points in the report instead).

### 5.3 Structure

* **Separate logic from UI.** Gap detection, ranking, ICS writing, metrics, the model, importers and friends logic live in plain Python modules without Streamlit; Streamlit files only call them and display results.
* **Short functions with one job each** (guideline: ≤ 30 lines). Prefer a clear loop with a comment over a clever one-liner.
* Classes as specified in SPEC §5; explicit `__init__`; validation through properties.
* Every function that depends on the current time takes `now=None` and uses `datetime.now()` only when `now` is `None`.
* Functions that use the database receive the connection as the first parameter (`connection`), except `get_connection()` itself and the API client.

---

## 6 Correctness rules (course "reality checks")

* **Floats:** never compare with `==`; use `math.isclose` or an explicit tolerance. **Money** is `Decimal` created from strings (`Decimal('25.00')`), never float; stored as integer Rappen; converted only through `models.price_from_rappen()` / `price_to_rappen()`. Remember that `round()` rounds half to even and `int()` truncates.
* **Every loop needs a way out; every automated action needs a sanity check:** retrain at most once per rating; cap suggestions per gap; reject imports above `MAX_RECORDS_PER_SOURCE` or empty live imports; cap calendar downloads at `MAX_CALENDAR_BYTES`; every network call has a `timeout`.
* **Membership tests** with sets or dicts (favorite IDs, friend codes, recognised WeekWise IDs), not lists.
* **Definitions are business decisions:** each term of SPEC §3 is implemented once, in the named function, and reused everywhere.
* **Validation:** invalid arguments raise `ValueError` with a clear English message; validated attributes use properties with setters. The UI catches `ValueError` and shows the message with `st.error`.
* **SQL:** always `?` placeholders with a params **tuple** (`params=(value,)` for one value); never f-strings, `%` or `+` to build SQL from values. Every write inside `with connection:`. `PRAGMA foreign_keys = 1` right after connecting. Reads with `pd.read_sql(..., index_col=['id'])` or `fetchone()`/`fetchall()`. Where a query has optional parts (rankings), choose between **fixed** SQL fragments with a dictionary – never insert user input into SQL text.
* **Time:** all datetimes inside WeekWise are naive Europe/Zurich; conversion only in `calendar_import.to_local_naive()` and `calendar_export.format_utc()` (SPEC §4.5).
* **Reproducibility:** `RANDOM_SEED` for `random`, `numpy` and every `random_state`.
* **Secrets:** the student's calendar link, API URLs and tokens are never written into code, never logged, never shown in full, never committed. They live in `st.session_state` or in git-ignored files under `data/private/` (SPEC §9.1, §7.3).
* **Privacy:** titles and details of private calendar entries are never written to the database.
* **Errors:** catch specific exceptions (`requests.RequestException`, `ValueError`, `sqlite3.Error`, `KeyError`), never a bare `except:`. Network and data errors fall back to stored data and produce a friendly message; they never crash the app.

---

## 7 AI-citation header (mandatory in every file)

**Course rule (lecture 02):** code generated by AI must be cited and quoted in the source code; HSG's rules on citation and authorship apply to source code accordingly. Because you generate the code, **every file you create or change carries a citation block**. [D-13: the team still checks this format against HSG's citation guideline.]

### 7.1 Python files

The first line names the file, followed by the module docstring (course style, e.g. `# fig02_01.py`), followed by the citation block:

```python
# gaps.py
"""Find free time slots in a student's week and check which events fit into them.

Part of WeekWise – HSG course "Fundamentals and Methods of Computer Science", autumn 2026.
"""
#
# AI CITATION
# Generated by: AI coding agent Anthropic Claude (<your exact model name>), <YYYY-MM-DD>
# Last AI change: <YYYY-MM-DD>, milestone <Mx>
# Based on: WeekWise SPEC.md v1.0, sections <§10, §5.2>
# Prompt summary: "<one sentence: what you were asked to build in this file>"
# Scope: entire file, except blocks marked "# Written by the team"
# Reviewed by: WeekWise team (<initials or 'pending'>)
# Changes by the team: <none | short list>
```

Rules:

* Fill in your real model name and the real dates; never leave `<…>` placeholders in delivered files except `Reviewed by` / `Changes by the team`, which stay `pending` / `none` until the team edits them.
* When you change an existing file, update `Last AI change` and, if needed, `Based on` and `Prompt summary`. Never remove the team's entries.
* If the team writes code themselves, they wrap it in `# Written by the team – start` / `# Written by the team – end`; keep those markers when editing nearby.

### 7.2 Other files

| File type | Citation |
|---|---|
| `.md` (README, STUDY_GUIDE, AI_USAGE) | First lines: an HTML comment `<!-- AI CITATION: generated by Anthropic Claude (<model>), <date>, from SPEC.md v1.0; reviewed by: pending -->` |
| `.json` (`taxonomy.json`, placeholder data, `sources.example.json`) | A top-level `"_meta"` object with an `"ai_citation"` string (JSON has no comments); parsers ignore `_meta` |
| `requirements.txt`, `.gitignore` | A `#` comment line with the citation |
| Generated data files (`.xlsx`, `.ics`, database) | No header; they are produced by `synthetic_data.py`, whose header covers them. The demo `.ics` contains `PRODID:-//WeekWise//…` and a comment line is not possible – that is fine. |

### 7.3 Project-level record

`AI_USAGE.md` lists every AI use in the project (SPEC §19). Add one row per milestone with date, model, files and what the team should check. `tests/test_project_rules.py` verifies the header in every `.py` file.

---

## 8 Course reference codes (for `# Course:` tags and STUDY_GUIDE)

| Code | Topic |
|---|---|
| L02 | Programming 1: ML intro (features, target, training vs. use), float pitfalls, `Decimal`, project rules |
| L03 | Programming 2: control statements, functions, lists, tuples; "every loop needs a way out" |
| L04 | Programming 3: recursion, lambda, comprehensions, slicing, dicts and sets, first Streamlit apps; hashing reality check |
| L05 | Programming 4: OOP – class, object, inheritance, polymorphism, encapsulation |
| L06 | Networks and distributed systems: HTTP methods, status codes, headers, URIs, Web APIs |
| U02 s3 / s4 / s5 | Python basics / control statements / functions, `random.seed` |
| U03 s1 – s5 | More control (phases, sentinel loops) / more functions (defaults, keyword args, scope) / lists / tuples / sequences (unpacking, `enumerate`, slicing) |
| U05 s2 | OOP in Python: properties with validation, `__repr__`/`__str__`, inheritance, `super()` |
| A01 – A05 | Basics, loops, functions, recursion/lambda/Streamlit setup, classes and sorting with `key=` |
| A06 | Web APIs with `requests`, Bearer tokens, `dateutil`, `timedelta`, wrapping API data in classes |
| A07 | SQLite: schema with PK/FK/NOT NULL, `PRAGMA foreign_keys`, `pd.read_sql` with `?` params, `with DB:` writes, JOIN/GROUP BY |
| A08 | pandas: reading files, dtypes, selection, `dropna`, `groupby` |
| A09 | pandas `merge`/`agg` and matplotlib/seaborn charts |
| A10 | ML: train/test split, classifiers, `accuracy_score`, `precision_recall_fscore_support`, `classification_report`, KNN |
| A11 | ML 2: stratified 80/10/10 split with seed 13, `RobustScaler` fitted on train only, comparing ≥ 5 classifiers scaled vs. unscaled |

---

## 9 Streamlit patterns

* `app.py` follows the start-up sequence of SPEC §4.6. Pages are files in `views/`, registered with `st.Page` and `st.navigation`.
* All `st.session_state` keys start with `ww_` (list in SPEC §16.1). Initialise each key once with a default (`if 'ww_week_start' not in st.session_state:`).
* Widget keys are unique and readable: `f'details_{event_id}_{gap_id}'`.
* **Dialogs:** buttons only set `st.session_state['ww_open_event_id']`; the dialog function is called once at the end of the page script. Only one dialog per script run.
* **Caching** lives only in `ui_components.py`: `@st.cache_resource` for the connection and the trained model (key = number of ratings, `max_entries=2`), `@st.cache_data(ttl=...)` for event and calendar loading and for the model evaluation. Every cached function has a comment explaining what invalidates it.
* After every database write call `st.rerun()` (optionally after `st.toast`).
* Colours only through `st.badge(color=…)` and coloured Markdown (`:blue[…]`), names from `config.DIRECTION_COLORS`. Never rely on colour alone – always write the direction name.
* Wrap each page body in one `try/except` that shows a friendly `st.error` (SPEC §16.9).

---

## 10 Tests

* `unittest.TestCase` classes in `tests/test_<module>.py`, test methods `test_<behaviour>` with a one-line docstring; assertions as in the assignments (`assertEqual`, `assertTrue`, `assertRaises`, `assertAlmostEqual`).
* No network, no real database file: use `database.get_connection(':memory:')` + `init_db()`; pass fixed `now` values; parse fixed strings instead of downloading.
* Minimum coverage per module as listed in SPEC §18; `tests/test_project_rules.py` enforces headers and import rules.
* All tests pass at the end of every milestone: `python -m unittest discover -s tests -v`.

---

## 11 Definition of done

**A file is done when:** it has the header (§7) and docstrings everywhere; it follows §§4–6; its logic is covered by tests; it appears in `STUDY_GUIDE.md`.

**A milestone is done when:** all its SPEC §20 acceptance criteria are met; the app starts on a fresh database without network; all tests pass; the milestone report is delivered; `AI_USAGE.md` has its row.

---

## 12 Never

* Never require a key, account, secret link or network connection to start or demo the app.
* Never commit or print secrets; never store private calendar titles in the database.
* Never call an LLM or AI service at runtime.
* Never write to the student's calendar through an API, and never implement sign-up inside WeekWise.
* Never build SQL with f-strings or string concatenation.
* Never use float for money.
* Never change taxonomy bundle keys after the freeze (SPEC §8.4).
* Never treat *not attended* as a negative training example.
* Never add a dependency, a page or a feature outside SPEC.md without asking.
* Never remove or weaken the AI-citation header.

---

## 13 Milestones (summary – details and acceptance criteria in SPEC §20)

| # | Milestone | Target date |
|---|---|---|
| M0 | Project skeleton: structure, `config.py`, requirements, README stub, rules test | Tue 03.11.2026 |
| M1 | Data foundation: models, database, taxonomy, importers, placeholder sources | Fri 06.11.2026 |
| M2 | Calendar import, gap detection, calendar export, week grid with busy blocks | Mon 09.11.2026 |
| M3 | **MVP**: onboarding, suggestions in gaps (heuristic), confirm/export/cancel, filters | Thu 12.11.2026 (MVP slot GE07 13.11.) |
| M4 | Machine learning and feedback loop: synthetic users, Random Forest, ✓/✗ + stars, pop-up, retrain | Wed 18.11.2026 (MVP slot GE08 20.11.) |
| M5 | Own Web API with database fallback | Mon 23.11.2026 |
| M6 | Rankings page and *About the model* (A10–A11 evaluation) | Fri 27.11.2026 |
| M7 | Friends page, friends going, invitations | Wed 02.12.2026 |
| M8 | Real data, hardening, documentation, end-to-end scenario, code freeze | Mon 07.12.2026 (upload deadline Thu 10.12.2026, 23:59) |
| R | Real-data integration whenever an input arrives (after M1) | – |
