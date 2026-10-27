---
title: Python Master Checklist v2 (Lesson 10 onward)
tags: [python, checklist, placements, data-roles]
created: 2026-10-27
---

# Python Master Checklist v2

**Goal:** placement-ready, interview-ready, job-ready Python, at a level where you can open an empty file and solve a real problem.
**Scope:** Python language + standard library. Only two third-party tools are included, both marked ⚠: `pytest` and `requests` (every real job uses them). Left out on purpose: SQL, NumPy, pandas, Git, Docker, web frameworks.

---

## The rules (read once, follow always)

1. **A lesson is ticked `[x]` only when you pass its Level 3 task from a blank file**, with no peeking at the lesson, and you can explain the concept aloud in 60 seconds.
2. **Every lesson has 3 levels:** L1 = predict the output, L2 = fix the bug, L3 = write it yourself. L3 is never solved for you.
3. **Blank-page protocol for every L3:** signature + docstring → tests first → plain-English comments → one comment at a time becomes one line of code, run after every line.
4. **Keep `Mistakes.md`** in Obsidian: one line per error you made and why. Re-read it every Sunday.
5. **Sunday review (30 min):** redo 3 old L3 tasks from scratch. Skipped reviews are why things feel forgotten.
6. **From Lesson 17 on, add a 20-minute daily warm-up:** one small problem from the pattern you just learned.

---

## Where you stand now

- [x] **Lessons 1–9 (Part I):** variables/types, strings, lists/tuples, dicts/sets, loops/comprehensions, functions, exceptions/files, modules/venv, classes (first look). These are in your notes. Lessons 10 onward revisit them through practice, so gaps get caught naturally.
- **Part II is reset.** OOP, dataclasses, ABC, context managers, generators, closures and decorators are treated as **not learned** and are re-taught practice-first in Phase C.
- **Lesson 10 is delivered; its Level 3 task is pending.**

---

## Phase A: Fix the foundations (Lessons 10–16)

- [ ] **10 · Names, objects, identity & copying**
  covers: variable = name, `id`, `is` vs `==`, mutable vs immutable, aliasing, pass-by-object-reference, shallow vs deep copy, the `[[0]*3]*3` trap
  *L3:* `add_audit_tag` (returns new records, originals untouched)

- [ ] **11 · Operators, truthiness & numbers**
  covers: `//` `%` `**` (negatives!), chained comparisons, `and`/`or` short-circuit and what they return, truthy/falsy, `None`, `is None`, bitwise `& | ^ << >>`, ints never overflow, float precision, `Decimal`/`Fraction` for money, banker's rounding, `math` (`isclose`, `inf`, `nan`), `divmod`
  *L3:* a bill calculator using `Decimal`; a safe `to_number()` cleaner

- [ ] **12 · Strings in full & text encoding**
  covers: every method (`find`, `partition`, `splitlines`, `zfill`, `casefold`, `translate`), escapes, raw and triple-quoted strings, format-spec mini-language (`:,` `:>10` `:08.2f` `:%` `!r` `=`), `str` vs `bytes`, `encode`/`decode`, UTF-8, `+=` vs `join`
  *L3:* clean and normalise a messy names/phone/amount column; print an aligned report table

- [ ] **13 · Containers in full: list, tuple, dict, set**
  covers: every method, slice assignment, `namedtuple`, single-element tuple, nested data, safe nested access, merge (`|`, `**`), sorting with `key`/`reverse`, stability, `max(d, key=d.get)`, set methods, `frozenset`, dict views, insertion order
  *L3:* flatten and group nested order data; top-3 per group

- [ ] **14 · Control-flow & looping toolkit**
  covers: `for-else`, `while-else`, `pass`, walrus `:=`, `match-case`, `enumerate(start=)`, `zip`/`zip_longest`/`zip(*matrix)`, nested comprehensions, `if` vs `if/else` position in comprehensions, `range` details, why you never modify a list while looping, `iter(callable, sentinel)`
  *L3:* matrix transpose + a command parser using `match-case`

- [ ] **15 · Functions, deeper**
  covers: positional-only `/` and keyword-only `*`, `global`/`nonlocal`, LEGB properly, nested functions, `callable()`, docstrings, `reduce`, returning multiple values, `locals()`/`globals()`, why `eval`/`exec` are avoided (one page)
  *L3:* a small validator toolkit built from inner functions

- [ ] **16 · Gotcha sweep I ("what does this print?")**
  covers: 25 mixed questions on lessons 10–15 (identity, `+=` on lists, defaults, scope, truthiness, float equality, tuple-with-list, shadowing built-ins)
  *L3:* write 5 of your own tricky snippets with explanations

---

## Phase B: Complexity & algorithmic Python (Lessons 17–23)

- [ ] **17 · Hashing & how dict/set really work**
  covers: hashable rules, why keys must be immutable, hash table idea, collisions, `__hash__`/`__eq__` contract, O(1) average, insertion order guarantee
  *L3:* build a tiny hash map class from a list of buckets

- [ ] **18 · Big-O & measuring code**
  covers: O(1), O(log n), O(n), O(n log n), O(n²), space complexity, complexity table for list/dict/set/str/deque, `timeit`, proving a claim with a measurement
  *L3:* benchmark list vs set membership and write the result in your own words

- [ ] **19 · Recursion & backtracking**
  covers: base case, call stack, recursion limit, recursion vs loop, subsets/permutations/combinations by hand
  *L3:* recursive flatten, permutations, N-queens (small)

- [ ] **20 · `collections`, stacks & queues**
  covers: `deque`, `Counter` (full API), `defaultdict`, `OrderedDict`, `ChainMap`; list as a stack; why a list is a bad queue
  *L3:* balanced-brackets checker; sliding-window max with `deque`

- [ ] **21 · Sorting, `heapq` & `bisect`**
  covers: Timsort, stability, `key`, `cmp_to_key`, min-heap, `nlargest`/`nsmallest`, heap of tuples, `bisect_left`/`insort`
  *L3:* top-K frequent items; merge K sorted lists

- [ ] **22 · Patterns I: two pointers, sliding window, prefix sums, hashmap counting**
  covers: when to recognise each pattern from the problem statement
  *L3:* 6 problems, 2 per pattern

- [ ] **23 · Memoization & intro to dynamic programming**
  covers: manual dict memo → `functools.lru_cache`, top-down vs bottom-up, 1D DP
  *L3:* fibonacci, climbing stairs, coin change, house robber

---

## Phase C: Pro-level language features (Lessons 24–36), re-taught practice-first

- [ ] **24 · Iterators & the iterator protocol**
  covers: iterable vs iterator vs generator, `__iter__`/`__next__`, `StopIteration`, exhaustion (loop twice = empty), `iter`/`next` with default
  *L3:* a class-based iterator `Countdown`; a batch iterator `batched(iterable, n)`

- [ ] **25 · Generators & streaming pipelines**
  covers: `yield`, generator expressions vs lists (memory), `yield from`, infinite generators, `send`/`close` (awareness), chaining generators into a pipeline, reading huge files lazily
  *L3:* a 3-stage pipeline `read_lines → parse → filter` over a big text file

- [ ] **26 · `itertools`, `functools` & `operator`**
  covers: `chain`, `islice`, `groupby` (sort first!), `product`, `permutations`, `combinations`, `accumulate`, `cycle`, `tee`, `starmap`; `partial`, `reduce`, `cache`, `cached_property`, `singledispatch`; `itemgetter`, `attrgetter`
  *L3:* group-and-summarise records using only these tools

- [ ] **27 · Closures**
  covers: functions as first-class objects, inner function remembering outer variables, late-binding trap in loops, `nonlocal`, closure as lightweight object
  *L3:* `make_counter`, `make_rate_limiter`

- [ ] **28 · Decorators**
  covers: manual vs `@` syntax, `functools.wraps`, decorators with arguments (factories), stacking order, class-based decorators, decorating methods
  *L3:* write `@timer`, `@retry(times, delay)`, `@cache_with_ttl` from blank

- [ ] **29 · Context managers**
  covers: `with` protocol (`__enter__`/`__exit__`), exception handling inside `__exit__`, `@contextmanager`, `suppress`, `closing`, `redirect_stdout`, `ExitStack`
  *L3:* a `Timer` context manager; a `temporary_directory` context manager; a DB-style transaction (commit/rollback)

- [ ] **30 · OOP I: classes, attributes & methods**
  covers: class vs object, `__init__`, `self`, instance vs class attributes (shared-list trap), instance/class/static methods, alternate constructors
  *L3:* `Account`, `Inventory`, `Transaction.from_csv_line()`

- [ ] **31 · OOP II: encapsulation, `@property` & dunder methods**
  covers: `_x` and `__x` conventions, name mangling, `@property` + setter validation, `__str__`/`__repr__`, `__eq__`/`__hash__`, `__lt__` + `total_ordering`, `__add__`, `__len__`, `__getitem__`, `__contains__`, `__bool__`, `__call__`
  *L3:* a `Money` class (add, compare, format, hashable); a `Matrix` or `Vector` class

- [ ] **32 · OOP III: inheritance, composition & abstract classes**
  covers: `super()`, overriding, MRO and the diamond, mixins, `abc.ABC` + `@abstractmethod`, polymorphism, duck typing, "composition over inheritance"
  *L3:* a `Reader` ABC with `CsvReader`/`JsonReader`; a pipeline built from composed components

- [ ] **33 · `dataclasses`, `Enum`, `NamedTuple`, `__slots__`**
  covers: `@dataclass` (`field`, `default_factory`, `frozen`, `order`, `__post_init__`, `slots`), `Enum`/`IntEnum`/`auto`, when to use dataclass vs dict vs namedtuple
  *L3:* model a transaction record with validation and status enum

- [ ] **34 · Type hints**
  covers: parameter/return hints, `list[int]`, `X | Y`, `Optional`, `Callable`, `TypedDict`, `Protocol`, `TypeVar`/`Generic` (light), `Literal`, `Final`; hints are not enforced at runtime; what `mypy` checks
  *L3:* fully type-annotate one of your earlier L3 solutions

- [ ] **35 · Exceptions, pro level**
  covers: `BaseException` vs `Exception` and the built-in tree, multiple `except`, order of clauses, bare `except` danger, re-raise, `finally` overriding `return`, `raise from`, custom exception hierarchies, `assert` and when not to use it, `warnings`, reading tracebacks
  *L3:* an exception hierarchy for a payment pipeline with wrapping and retry logic

- [ ] **36 · Design patterns & SOLID (interview talk) + metaclass/descriptor awareness**
  covers: factory, singleton (`__new__`), strategy, observer; SOLID in one example each; one page each on metaclasses and descriptors (what they are, where frameworks use them), `__init_subclass__`
  *L3:* implement strategy + observer in under 40 lines each

---

## Phase D: Real-world scripting (Lessons 37–48)

- [ ] **37 · Files & paths, pro level**
  covers: all modes (`r+` `w+` `a+` `x` `rb` `wb`), `seek`/`tell`, buffering, `pathlib` vs `os.path`, `shutil`, `tempfile`, `glob`/`fnmatch`, walking a tree, `gzip`/`zipfile`, `pickle` (and why it is unsafe on untrusted data), `io.StringIO`/`BytesIO`, atomic writes
  *L3:* a folder organiser (sort files by extension, dry-run mode, undo log)

- [ ] **38 · CSV & JSON, pro level**
  covers: `csv` dialects and quoting, `DictReader` edge cases, `json` options (`default=`, `ensure_ascii`, `sort_keys`, custom encoder), JSON Lines, flattening nested JSON, `configparser`, `tomllib` (awareness)
  *L3:* CSV → cleaned JSONL converter with a rejects file

- [ ] **39 · Command-line scripts & configuration**
  covers: `sys.argv`, `argparse` (positional, optional, flags, choices, subcommands), exit codes and `sys.exit`, `os.environ`, `.env` idea, `python -m`, `if __name__ == "__main__"`
  *L3:* a CLI `logstat --file x.log --level ERROR --top 5`

- [ ] **40 · `logging`, properly**
  covers: levels, loggers, handlers, formatters, hierarchy, file + console logging, rotating files, `logging` vs `print`, logging exceptions with tracebacks
  *L3:* add production-style logging to your folder organiser

- [ ] **41 · `datetime`, `time` & time zones**
  covers: `date`/`time`/`datetime`/`timedelta`, `strftime`/`strptime`, ISO 8601, epoch, `zoneinfo`, naive vs aware datetimes, `time.sleep`/`monotonic`/`perf_counter`
  *L3:* parse mixed-format timestamps, convert IST ↔ UTC, group events by day

- [ ] **42 · Regular expressions I**
  covers: `match`/`search`/`fullmatch`/`findall`/`finditer`/`sub`/`split`, character classes, quantifiers, anchors, groups, `compile`
  *L3:* extract emails, phone numbers and dates from messy text

- [ ] **43 · Regular expressions II**
  covers: named groups, greedy vs lazy, lookahead/lookbehind, flags (`re.I`, `re.M`, `re.X`), `sub` with a function, catastrophic backtracking (awareness)
  *L3:* parse real-looking web-server log lines into dicts

- [ ] **44 · Running other programs: `subprocess`**
  covers: `run`, capture output, `check=True`, return codes, timeouts, passing args safely (never build shell strings from user input), `shutil.which`
  *L3:* a script that runs a command, captures output and logs failures

- [ ] **45 · HTTP & APIs** ⚠ `requests`
  covers: how HTTP works (methods, status codes, headers), `urllib` once, then `requests`: params, JSON, auth headers, `timeout`, `raise_for_status`, pagination, retries with backoff, rate limits, sessions
  *L3:* fetch every page of a paginated API with retry and a polite delay

- [ ] **46 · Utility standard library**
  covers: `random` (`seed`, `choice`, `sample`, `shuffle`), `secrets`, `hashlib`, `uuid`, `base64`, `statistics`, `textwrap`, `pprint`, `string`
  *L3:* a dedupe-by-file-hash tool; a reproducible random sampler

- [ ] **47 · Modules, packages & project structure**
  covers: absolute vs relative imports, `__init__.py`, `__all__`, `import *` rules, `importlib`/`reload`, circular imports and how to break them, `venv`, `pip`, `pip freeze`, `requirements.txt`, `pyproject.toml` (awareness), running a package with `-m`
  *L3:* turn your folder organiser into a proper installable-looking package

- [ ] **48 · Automation craft**
  covers: idempotent scripts (safe to run twice), dry-run flags, checkpoint/resume, retries, atomic writes, validating inputs, plain-Python grouping/aggregation/joins/dedupe without pandas
  *L3:* a "daily report" script that survives being run twice and half-way crashes

---

## Phase E: Quality, speed & internals (Lessons 49–56)

- [ ] **49 · Debugging**
  covers: reading a traceback bottom-up, `pdb`/`breakpoint()`, bisecting a bug, print vs log vs debugger, reproducing with a minimal example
  *L3:* debug 5 broken scripts I give you

- [ ] **50 · Testing I** ⚠ `pytest`
  covers: `assert`, `unittest` (awareness), `pytest` discovery, fixtures, `parametrize`, `pytest.raises`, testing pure functions first
  *L3:* a test suite for your `Money` class and your CSV cleaner

- [ ] **51 · Testing II**
  covers: `unittest.mock` (`patch`, `Mock`, `MagicMock`), `tmp_path`, `monkeypatch`, `doctest`, testing code that touches files, time and HTTP, what coverage does and doesn't tell you
  *L3:* test the paginated API fetcher with no real network

- [ ] **52 · Profiling & optimisation**
  covers: `timeit`, `cProfile`, `tracemalloc`, measure-first rule, common wins (set lookups, `join`, generators, `lru_cache`, avoiding repeated work, local variable lookup)
  *L3:* profile a slow script, find the hot spot, fix it, show before/after numbers

- [ ] **53 · Memory & CPython internals**
  covers: reference counting, garbage collector and cycles, `gc`, `weakref` (awareness), small-int cache and string interning, list resizing, dict/set internals, bytecode with `dis` (awareness), CPython vs PyPy
  *L3:* write a one-page "what happens when I do `a = b`" explainer in your own words, with code

- [ ] **54 · Concurrency I: threads**
  covers: CPU-bound vs I/O-bound, the GIL (what it is and why it exists), `threading`, `join`, daemon threads, `Lock`, race conditions, deadlock, `Event`, `queue.Queue`, producer–consumer
  *L3:* a multi-threaded downloader with a worker queue

- [ ] **55 · Concurrency II: processes & async**
  covers: `multiprocessing` (`Process`, `Pool`), `concurrent.futures` (`ThreadPoolExecutor`/`ProcessPoolExecutor`/`Future`), `asyncio` (`async`/`await`, `gather`, tasks), choosing between them
  *L3:* the same job done 3 ways, with timings and a written verdict

- [ ] **56 · Code quality & refactoring**
  covers: PEP 8, PEP 257 docstring styles, naming, linters/formatters (`ruff`, `black` awareness), the 3-step refactor (draft → spot bottlenecks → upgrade), reading others' code, code-review habits
  *L3:* refactor one ugly script of yours and write what you changed and why

---

## Phase F: Placement & interview sprint (Lessons 57–64)

- [ ] **57 · Linked lists, stacks & queues in Python**
  *L3:* reverse a linked list, detect a cycle, implement a stack/queue class

- [ ] **58 · Trees & BST**
  covers: node class, traversals (recursive + iterative), height, validate BST, level order
  *L3:* 5 problems

- [ ] **59 · Graphs**
  covers: adjacency dict, BFS, DFS, connected components, topological sort, shortest path with `heapq`
  *L3:* 5 problems (grid islands, course schedule, word ladder, ...)

- [ ] **60 · Dynamic programming in Python**
  covers: 2D DP, memo vs tabulation, recognising DP from the statement
  *L3:* LCS, knapsack, edit distance

- [ ] **61 · Machine-coding / OOP design round**
  covers: how to structure a 45-minute design problem
  *L3:* LRU cache, in-memory key-value store with TTL, rate limiter, parking lot, task scheduler

- [ ] **62 · Data-role scripting round (no pandas)**
  covers: log parsing, dedupe, group-by/aggregate, JSON flatten, top-K, merge sorted streams, sessionization, finding duplicates/outliers
  *L3:* 8 problems, timed

- [ ] **63 · Interview Q&A bank**
  covers: ~150 Python questions by topic, with answers, plus a "what does this print" bank; you answer aloud first, then check
  *L3:* record yourself answering 20 questions in 60 seconds each

- [ ] **64 · Mock rounds**
  covers: 3 timed mocks (45 min coding + 15 min theory), review after each, targeted re-study of what failed
  *L3:* pass all 3 with no hints

---

## Phase G: Capstone projects (build these, then put them on your resume)

- [ ] **P1 · Log analyzer CLI**: `argparse`, `re`, `collections`, `datetime`, `logging`, `pytest` tests
- [ ] **P2 · Mini ETL pipeline**: CSV/JSON in → validate/clean → output + rejects file; generators, dataclasses, context managers, retry decorator, logging, tests
- [ ] **P3 · API ingestion tool** ⚠: `requests`, pagination, retry/backoff, caching, `concurrent.futures`, config from environment variables, idempotent re-runs
- [ ] **P4 · Task scheduler / key-value store**: OOP design, `heapq`, threading basics, tests

---

## If time is short: the fast track

Do these first, in this order, and come back to the rest later:
**10 → 11 → 12 → 13 → 14 → 15 → 17 → 18 → 19 → 20 → 21 → 22 → 24 → 25 → 28 → 30 → 31 → 35 → 37 → 38 → 40 → 41 → 42 → 63 → 64**

---

## Deliberately cut or kept to one page

Metaclasses and descriptors (one page, in Lesson 36), `__getattribute__`/`__setattr__`, `ExceptionGroup`/`except*`, `eval`/`exec` (one paragraph, why to avoid), `weakref`, `array`, SOLID in depth, advanced `asyncio` (only the basics in 55). Add any of these later if a job asks for them.

---

## Progress log

| Phase | Lessons | Done |
|---|---|---|
| A: Foundations | 10–16 (7) | 0 |
| B: Complexity & algorithms | 17–23 (7) | 0 |
| C: Pro language | 24–36 (13) | 0 |
| D: Real-world scripting | 37–48 (12) | 0 |
| E: Quality & internals | 49–56 (8) | 0 |
| F: Placement sprint | 57–64 (8) | 0 |
| G: Capstones | P1–P4 (4) | 0 |

Total: 55 lessons + 4 projects. At about one lesson every 1–2 days, expect roughly 3 months. The fast track alone is 25 lessons.
