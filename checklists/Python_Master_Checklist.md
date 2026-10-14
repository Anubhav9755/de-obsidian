---
title: Python Master Checklist — Zero to Pro (Data Roles)
tags: [python, checklist, data-engineering, data-science, data-analyst]
created: 2026-10-14
status: in-progress
---

# 🐍 Python Master Checklist — Zero to Pro (Data Engineer · Data Scientist · Analyst)

Converted from *The Complete Python Checklist* and **re-prioritised for data jobs**. Progress ticks come from what your `Python_building_updated.ipynb` notebook actually covers. This is a living document: edit it freely.

> [!info] How to read this
> - `- [x]` = done (seen in your notebook) · `- [ ]` = to do
> - **Priority:** 🔴 must-have for data jobs · 🟡 should learn · 🟢 optional / awareness only
> - **Roles:** `DE` data engineer · `DS` data scientist · `DA` analyst
> - Items marked *(partial)* are touched in the notebook but not yet complete. Leave unticked until you can do them from memory, without notes.

> [!warning] Basics not tracked
> Your notebook starts at generators and OOP, so Section 1–3 basics (variables, loops, lists, dicts, etc.) are unticked here. If you already did them elsewhere, tick them yourself.

## 📊 Progress Snapshot

<!-- PROGRESS_START -->
| Part | Done | Total | % |
|---|---|---|---|
| PART I — The Core Language | 16 | 110 | 15% |
| PART II — Standard Library Tour | 0 | 19 | 0% |
| PART III — Testing, Tooling & Professional Practice | 0 | 15 | 0% |
| PART IV — Data Stack (pick your track, but DE/DS/DA share the base) | 0 | 39 | 0% |
| PART V — Polars, In Depth | 0 | 13 | 0% |
| PART VI — Other Domains (🟢 awareness only for data careers) | 0 | 13 | 0% |
| PART VII — Adjacent Skills Data Jobs Test Every Time | 0 | 5 | 0% |
| PART VIII — Projects (proof of skill) | 0 | 6 | 0% |
| **Overall** | **16** | **220** | **7%** |

🔴 must-haves done: **14 / 108** (13%)

_Update this table manually as you tick boxes (counts above are as of 2026-10-14)._
<!-- PROGRESS_END -->

## 🎯 Next Up (suggested order)

1. Decorators (you've already used `@property`, `@classmethod`, `@contextmanager`, so this is the natural next step)
2. Error handling, deep (custom exceptions, `raise from`, retry logic)
3. File I/O + `pathlib`, `json`, `csv`, `logging`
4. Typing + `dataclasses` + `pydantic`
5. `pytest` basics (start writing tests for what you've already built)
6. Then SQL + `pandas` → `numpy` → Polars

## 🔧 What I changed from the original

- Added **priority + role tags** so you can skip what a data career rarely needs
- Moved **pandas / numpy / SQL drivers / pyarrow** up front; demoted GUI, game dev, embedded, metaclasses/descriptors to 🟢
- Added missing data-role topics: statistics, EDA, visualisation, scikit-learn workflow, APIs/ETL, data quality, cloud SDK (`boto3`), parquet
- Added a **Projects** section (proof of skill beats ticking boxes) and an **Adjacent skills** section (SQL, Git, Linux, Docker)
- Added items the notebook covers that the original list lacked (mixins, diamond problem, streaming-pipeline pattern, production refactoring mindset)

---

# PART I — The Core Language

## 1. Fundamentals
- [ ] 🔴 Variables, dynamic + strong typing, PEP 8 naming
- [ ] 🔴 Numeric types: `int`, `float`, `complex`
- [ ] 🔴 `Decimal` for precise money math (matters in finance data) and `Fraction` 🟢
- [ ] 🔴 Strings: indexing, slicing, immutability
- [ ] 🟡 Encoding: `str` vs `bytes`, `.encode()` / `.decode()`, UTF-8 (real-world messy data!)
- [ ] 🔴 f-strings incl. format specs (`f"{x:.2f}"`, alignment, padding, `=` debug)
- [ ] 🔴 Booleans, truthy/falsy, short-circuit evaluation
- [ ] 🔴 `None`, `is` vs `==`
- [ ] 🔴 Type casting and implicit coercion rules

## 2. Control Flow
- [ ] 🔴 `if / elif / else`, ternary expressions
- [ ] 🔴 `while`, `for`, `break`, `continue`, loop `else`
- [ ] 🟡 `match / case` structural pattern matching (3.10+), guards, destructuring
- [ ] 🟡 Walrus operator `:=` and idiomatic use

## 3. Data Structures — Deep
- [ ] 🔴 Lists: methods, comprehensions, nested lists, slicing tricks (`[::-1]`, step)
- [ ] 🔴 Tuples: packing/unpacking, `namedtuple`, `NamedTuple`
- [ ] 🔴 Dicts: comprehensions, `defaultdict`, `Counter`, `ChainMap`, merge with `|`, ordering guarantee
- [ ] 🔴 Sets: operations, `frozenset`, set comprehensions
- [ ] 🟢 `array` module (typed arrays)
- [ ] 🟡 `collections.deque` (O(1) both ends)
- [ ] 🔴 Choosing the right structure by trade-offs (list vs dict vs set lookup)
- [ ] 🔴 Mutability, aliasing, the mutable-default-argument trap
- [ ] 🟡 Shallow vs deep copy (`copy` module)

## 4. Functions — Deep
- [ ] 🔴 `*args`, `**kwargs`, positional-only (`/`) and keyword-only (`*`) parameters
- [ ] 🟡 Closures and the late-binding closure bug
- [ ] 🔴 Lambda functions: when to use and when not to
- [ ] 🔴 `map`, `filter`, `functools.reduce`
- [ ] 🟡 Recursion, memoization, why deep recursion is risky in Python
- [ ] 🟡 Scope: LEGB, `global`, `nonlocal`
- [ ] 🔴 Type hints for parameters and return values *(partial: `List[str]`, `Generator[str, None, None]`, `Iterator[None]` used in notebook)*
- [ ] 🟡 Docstring conventions (Google / NumPy / reST)
- [x] 🔴 Production refactoring mindset: Draft → find bottleneck → production-quality (the "3-step rule")

## 5. Error Handling — Deep
- [ ] 🔴 `try / except / else / finally`, multiple exception types *(partial: `try/finally` motivation for context managers)*
- [ ] 🟡 Exception hierarchy, and why you never casually catch `BaseException`
- [ ] 🔴 Custom exceptions, chaining (`raise X from Y`), `__cause__` vs `__context__`
- [ ] 🔴 Fail loud vs fail soft, retry logic (huge in data pipelines)
- [ ] 🟡 `warnings` module
- [ ] 🟢 Exception groups (`except*`, 3.11+)

## 6. File I/O & OS Interaction
- [ ] 🔴 `open()` modes (`r/w/a/rb/wb/x`) and the `with` statement *(partial: `with open` used in notebook)*
- [ ] 🔴 `pathlib.Path` (modern) vs `os.path` (legacy)
- [ ] 🔴 Reading large files efficiently (line iteration, chunked reads)
- [ ] 🟢 `mmap` memory mapping
- [ ] 🟡 `os`: env variables, `os.walk`
- [ ] 🟡 `shutil`: copy / move / delete / archive
- [ ] 🟡 `glob` pattern matching
- [ ] 🟡 `tempfile`
- [ ] 🟡 `subprocess` (and why `shell=True` is a security risk)

## 7. Object-Oriented Programming — Deep
- [x] 🔴 Classes, objects, `__init__`, `self`
- [x] 🔴 Instance vs class attributes (and the shadowing gotcha)
- [x] 🔴 Instance methods
- [x] 🔴 `@classmethod` and alternate constructors
- [x] 🔴 `@staticmethod`, and how to choose between the three method types
- [x] 🔴 Inheritance, `super()`, MRO, multiple inheritance
- [x] 🟡 Diamond problem and cooperative inheritance
- [x] 🟡 Mixins and polymorphism (DataPipeline example)
- [x] 🔴 Encapsulation conventions (`_protected`, `__private` name mangling)
- [x] 🔴 Properties (`@property`, setters) — controlled attribute access
- [ ] 🟡 Property deleters (`@x.deleter`)
- [ ] 🔴 Dunder methods *(partial: `__init__`, `__repr__`, `__str__`, `__eq__`, `__len__` covered; still to do below)*
  - [ ] 🔴 `__lt__` / `__gt__` and ordering (`functools.total_ordering`)
  - [ ] 🔴 `__hash__` (and why defining `__eq__` affects it)
  - [ ] 🟡 `__iter__` / `__next__`, `__getitem__` / `__setitem__`, `__contains__`
  - [ ] 🟡 `__call__`, operator overloads (`__add__` etc.)
- [ ] 🔴 Composition vs inheritance
- [ ] 🔴 `dataclasses`: `@dataclass`, `field()`, `frozen=True`, `__post_init__` (used everywhere in modern pipelines)
- [ ] 🟡 Abstract Base Classes (`abc`, `@abstractmethod`)
- [ ] 🟡 Enums (`Enum`, `IntEnum`, `Flag`, `auto()`)
- [ ] 🟢 `__slots__`
- [ ] 🟢 Descriptors (`__get__` / `__set__`)
- [ ] 🟢 Metaclasses (know what they are; ORMs use them)

## 8. Iterators, Generators, Context Managers
- [x] 🔴 Generators (`yield`)
- [x] 🔴 Generator expressions vs list comprehensions (memory efficiency, lazy vs eager)
- [x] 🔴 Streaming pipeline pattern (context manager + chained generators for huge files)
- [x] 🔴 Context managers: `with`, `__enter__` / `__exit__`
- [x] 🔴 `@contextlib.contextmanager`
- [ ] 🔴 Iterator protocol in full (`__iter__`, `__next__`, `StopIteration`) *(partial)*
- [ ] 🟡 `yield from` and generator delegation
- [ ] 🟢 Native `async def` coroutines vs generator-based
- [ ] 🔴 `itertools`: `chain`, `islice`, `groupby`, `product`, `permutations`, `combinations`, `cycle`, `tee`, `accumulate`
- [ ] 🟡 `contextlib.suppress`, multiple context managers in one `with`

## 9. Decorators — Deep
- [ ] 🔴 Basic decorators and `functools.wraps`
- [ ] 🔴 Decorators with arguments (factories)
- [ ] 🟡 Class-based decorators
- [ ] 🟡 Stacking decorators and execution order
- [ ] 🔴 Built-ins cold: `@lru_cache`, `@cache`, `@singledispatch` (you already know `@property`, `@classmethod`, `@staticmethod`)
- [ ] 🔴 Practical patterns: timing, retry, logging, caching

## 10. Functional Programming Toolkit
- [ ] 🟡 `functools`: `partial`, `reduce`, `cmp_to_key`, `wraps`, `lru_cache`
- [ ] 🟡 `operator` module (e.g. `sorted(key=itemgetter(...))`)
- [ ] 🟡 Pure functions and the immutability mindset (easier testing and parallelism)

## 11. Typing System
- [ ] 🔴 Type hints for variables, functions, classes
- [ ] 🔴 `typing`: `Optional`, `Union` (and `|`), `Callable`, `Any`, `TypeVar`, `Generic`, `Literal`, `TypedDict`, `Protocol`
- [ ] 🟡 `mypy` or `pyright`
- [ ] 🔴 Runtime validation vs static hints (hints don't enforce at runtime)
- [ ] 🔴 `pydantic` v2: models, validators, nested models, serialisation

## 12. Concurrency, Parallelism & Async
- [ ] 🟡 Concurrency vs parallelism; the GIL and the 3.13+ free-threaded build
- [ ] 🟡 `threading`: I/O-bound tasks, locks, race conditions, deadlocks
- [ ] 🟡 `multiprocessing`: CPU-bound tasks, pools, pickling requirements
- [ ] 🔴 `concurrent.futures`: `ThreadPoolExecutor`, `ProcessPoolExecutor`
- [ ] 🟡 `asyncio`: event loop, `async/await`, `gather`, `run`, `async with`, `async for`
- [ ] 🟡 When to use which (I/O-bound → asyncio/threads; CPU-bound → multiprocessing)
- [ ] 🟡 Pitfalls: blocking calls inside async code, races, deadlocks

## 13. Memory Management & Internals
- [ ] 🟡 Reference counting + garbage collection (`gc`), cyclic references
- [ ] 🟡 Interning (small ints/strings), and why `is` sometimes "works" for `==`
- [ ] 🟡 Name resolution and variable binding
- [ ] 🟢 Bytecode basics (`dis`)
- [ ] 🟢 CPython vs PyPy and other implementations

## 14. Performance & Profiling
- [ ] 🔴 `timeit` micro-benchmarking
- [ ] 🟡 `cProfile` and reading profiler output
- [ ] 🔴 Big-O awareness for built-ins (list vs dict vs set)
- [ ] 🔴 Vectorisation mindset (numpy/polars over manual loops)
- [ ] 🟢 Cython, numba, `ctypes` (awareness only)

---

# PART II — Standard Library Tour

- [ ] 🔴 `datetime`, `time`, `zoneinfo`: timezones, naive vs aware datetimes
- [ ] 🔴 `re`: `match`, `search`, `findall`, `finditer`, `sub`, named groups, lookahead/lookbehind
- [ ] 🔴 `json`: load/dump, custom encoders/decoders
- [ ] 🔴 `csv`: `DictReader` / `DictWriter`
- [ ] 🔴 `sqlite3`: great for scripting and practising SQL from Python
- [ ] 🔴 `argparse`: build real CLI tools
- [ ] 🔴 `logging`: levels, handlers, formatters, config *(partial: used in timer context-manager example)*
- [ ] 🟡 `hashlib` (MD5/SHA1 broken for security; fine for checksums/dedup keys)
- [ ] 🟡 `secrets` (vs `random`, which is NOT secure)
- [ ] 🟡 `uuid`
- [ ] 🟡 `heapq`, `bisect`
- [ ] 🟢 `struct`
- [ ] 🟡 `pickle` (never unpickle untrusted data)
- [ ] 🟢 `socket`
- [ ] 🟡 `xml.etree.ElementTree`
- [ ] 🟡 `configparser`
- [ ] 🟡 `sys`: args, `sys.exit`, `sys.path`
- [ ] 🟡 `traceback`
- [ ] 🟢 `inspect`

---

# PART III — Testing, Tooling & Professional Practice

- [ ] 🔴 `pytest`: fixtures, `parametrize`, marks, `conftest.py`
- [ ] 🔴 Mocking (`unittest.mock`, `pytest-mock`): APIs, DBs, file systems
- [ ] 🟢 Property-based testing (`hypothesis`)
- [ ] 🟡 Coverage (`pytest-cov`) and its limits
- [ ] 🟢 `tox` / `nox`
- [ ] 🔴 Formatting & linting: `black`, `isort`, `ruff`
- [ ] 🟡 Pre-commit hooks
- [ ] 🔴 Git fundamentals for Python projects (`.gitignore`, branching)
- [ ] 🔴 Virtual environments: `venv` + one modern tool (`uv` or `poetry`)
- [ ] 🔴 `requirements.txt` vs `pyproject.toml`
- [ ] 🟢 Packaging & publishing to PyPI
- [ ] 🟡 `pyenv`
- [ ] 🔴 Docker basics for Python apps (clean `Dockerfile`, multi-stage, `.dockerignore`)
- [ ] 🟡 CI/CD basics (GitHub Actions: run tests/lint)
- [ ] 🟡 Design patterns: Singleton, Factory, Strategy, Observer, Decorator, Dependency Injection

---

# PART IV — Data Stack (pick your track, but DE/DS/DA share the base)

## A. Shared Foundation (all data roles)
- [ ] 🔴 `numpy`: arrays, broadcasting, vectorisation, why it's fast
- [ ] 🔴 `pandas`: DataFrames, `.loc` / `.iloc`, boolean filtering
- [ ] 🔴 `pandas`: `groupby`, aggregations, `pivot_table`
- [ ] 🔴 `pandas`: merges/joins, `concat`
- [ ] 🔴 `pandas`: missing data, duplicates, dtype fixes, string/datetime accessors
- [ ] 🔴 `pandas`: `apply` vs vectorised ops (and when apply is a trap)
- [ ] 🔴 Reading/writing CSV, JSON, Excel, Parquet
- [ ] 🔴 SQL from Python (`sqlite3`, `sqlalchemy`, `pd.read_sql`)
- [ ] 🔴 Data cleaning workflow (profile → clean → validate → export)
- [ ] 🔴 Jupyter notebooks properly (and their limits for production code)

## B. Data Engineer track `DE`
- [ ] 🔴 `pyarrow` and Parquet (columnar format, partitioning)
- [ ] 🔴 `sqlalchemy` Core vs ORM
- [ ] 🔴 DB drivers: `psycopg2`/`psycopg3` (Postgres), `pymysql` (MySQL), `pyodbc` (SQL Server)
- [ ] 🔴 Calling REST APIs: `requests` / `httpx`, pagination, rate limits, retries
- [ ] 🔴 Building an ETL/ELT script: extract → transform → load, idempotent re-runs
- [ ] 🔴 Incremental loading, batching, chunked processing
- [ ] 🔴 Cloud SDK basics: `boto3` (S3 read/write from Python)
- [ ] 🔴 Orchestration: Airflow (one deeply), Prefect/Dagster by name
- [ ] 🟡 Data quality checks (`great_expectations` or custom validation + `pydantic`)
- [ ] 🟡 `pyspark` basics (DataFrame API, lazy execution, partitions)
- [ ] 🟡 `dbt` (Jinja + SQL modelling, Python-adjacent)
- [ ] 🟡 Config & secrets management (`.env`, `python-dotenv`, never hardcode credentials)
- [ ] 🟡 Logging, monitoring and alerting for pipelines

## C. Data Scientist track `DS`
- [ ] 🔴 Statistics in code: mean/variance/distributions, hypothesis tests (`scipy.stats`)
- [ ] 🔴 Visualisation: `matplotlib`, `seaborn` (and `plotly` 🟡)
- [ ] 🔴 EDA workflow (distributions, correlations, outliers, missing-value strategy)
- [ ] 🔴 `scikit-learn`: preprocessing, train/test split, pipelines, cross-validation, metrics
- [ ] 🔴 Core models: linear/logistic regression, trees, random forest, gradient boosting (`xgboost` / `lightgbm`)
- [ ] 🔴 Feature engineering, leakage awareness
- [ ] 🟡 Model evaluation, tuning, overfitting/underfitting
- [ ] 🟡 `pytorch` or `tensorflow` (pick one, after classical ML)
- [ ] 🟡 Experiment tracking (`mlflow`)
- [ ] 🟢 Model serving basics (FastAPI)

## D. Data Analyst track `DA`
- [ ] 🔴 Advanced `pandas`: window ops (`rolling`, `expanding`), `resample`, time series
- [ ] 🔴 Visualisation and storytelling (`matplotlib`, `seaborn`, `plotly`)
- [ ] 🔴 Reporting automation: Excel via `openpyxl` / `xlsxwriter`
- [ ] 🟡 Dashboards: `streamlit` (quick) · BI tools by name (Power BI, Tableau)
- [ ] 🟡 A/B testing basics, descriptive vs inferential stats
- [ ] 🟡 Scheduling scripts (`schedule`, cron)

---

# PART V — Polars, In Depth

> [!tip] Learn after pandas fundamentals are solid, in parallel with DE project work. Don't replace pandas with it.

- [ ] 🟡 Why Polars: Rust, multi-threaded, lazy engine, memory-efficient
- [ ] 🟡 Core objects: `pl.DataFrame`, `pl.Series`, `pl.LazyFrame`
- [ ] 🟡 **Eager vs Lazy API** (the most important concept): `read_csv` vs `scan_csv` → `.collect()`
- [ ] 🟡 Expression API: `pl.col()`, chaining, how it differs from pandas
- [ ] 🟡 `.select()`, `.filter()`, `.with_columns()`
- [ ] 🟡 `.group_by()`, `.agg()`
- [ ] 🟡 `.join()` (inner / left / outer / cross)
- [ ] 🟡 Window functions with `.over()` (SQL `PARTITION BY`)
- [ ] 🟡 Reading/writing: CSV, Parquet, JSON, Arrow IPC
- [ ] 🟡 Streaming: `.collect(streaming=True)` for larger-than-memory data
- [ ] 🟡 Query optimisation: predicate/projection pushdown
- [ ] 🟡 Interop: `.to_pandas()`, numpy, Arrow
- [ ] 🟡 When to still use pandas vs when Polars is clearly better

---

# PART VI — Other Domains (🟢 awareness only for data careers)

## Web / APIs (useful for serving data & ingestion)
- [ ] 🟡 `fastapi` (async, pydantic, auto OpenAPI docs)
- [ ] 🟢 `flask`, `django` (know they exist)
- [ ] 🟡 REST principles, status codes, auth basics (JWT, OAuth)
- [ ] 🟢 WSGI vs ASGI, CORS, static files

## Scraping & Automation (useful for ingestion)
- [ ] 🟡 `BeautifulSoup`
- [ ] 🟢 `scrapy`, `selenium` / `playwright`
- [ ] 🟡 `smtplib` / `email` for pipeline alerts

## Security Essentials (keep the practical ones)
- [ ] 🔴 Parameterised queries (never string-format SQL)
- [ ] 🔴 Secrets via env vars / `.env`, never hardcoded
- [ ] 🟢 `cryptography` library basics

## Not needed for data roles (🟢 skip unless curious)
- [ ] 🟢 GUI: `tkinter`, `PyQt` / `PySide`, `kivy`
- [ ] 🟢 Game dev: `pygame`
- [ ] 🟢 Embedded: MicroPython / CircuitPython

---

# PART VII — Adjacent Skills Data Jobs Test Every Time

- [ ] 🔴 SQL: joins, aggregations, CTEs, window functions, query optimisation
- [ ] 🔴 Git + GitHub workflow
- [ ] 🔴 Linux/CLI basics
- [ ] 🟡 Data modelling: star schema, normalisation, SCDs (DE)
- [ ] 🟡 DSA for interviews: arrays, hashmaps, sorting, two pointers, basic recursion

---

# PART VIII — Projects (proof of skill)

- [ ] 🔴 **Project 1: CSV → clean → Parquet pipeline** using generators, context managers, logging, pytest
- [ ] 🔴 **Project 2: API → database ETL** (requests + sqlalchemy/Postgres, idempotent, scheduled)
- [ ] 🔴 **Project 3: EDA + dashboard** on a real dataset (pandas + seaborn + streamlit)
- [ ] 🟡 **Project 4: ML model** end-to-end with sklearn Pipeline and evaluation write-up
- [ ] 🟡 **Project 5: Same pipeline in pandas vs Polars**, benchmarked on a large file
- [ ] 🟡 **Project 6: Airflow DAG** orchestrating Project 2 (Dockerised)

---

# 📝 Study Notes & Review Log

| Date | Topic | Confidence (1–5) | Notes |
|------|-------|------------------|-------|
|      |       |                  |       |

## Honest note

You don't need to finish every box before starting real work. Finish the 🔴 items in Part I (sections 1–9) properly, learn Parts II–III through real projects, pick your Part IV track deliberately, and treat everything marked 🟢 as "known to exist." The most common way ambitious learners stall is gold-plating every domain before building anything.
