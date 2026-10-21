# Module 1: Core Querying Foundations

## Practice Setup (Run This First)

Every example in this module uses the `customers` table below. It deliberately contains **NULL values**, **mixed-case text**, and **duplicate countries** so the edge cases in each topic actually show up.

```sql
DROP TABLE IF EXISTS customers;

CREATE TABLE customers (
    customer_id  SERIAL PRIMARY KEY,
    first_name   VARCHAR(50),
    country      VARCHAR(50),
    total_spent  DECIMAL(10, 2),
    email        VARCHAR(100)
);

INSERT INTO customers (first_name, country, total_spent, email) VALUES
('Amit',   'India',  250.00, 'amit@mail.com'),
('Sarah',  'USA',    120.50, 'sarah@mail.org'),
('Aarav',  'India',  NULL,   'aarav@mail.com'),
('Diana',  'UK',     310.00, 'diana@mail.co.uk'),
('Priya',  'India',   95.75, 'priya@mail.com'),
('Ana',    'Canada', 180.00, NULL),
('John',   'USA',    NULL,   'john@mail.com'),
('Maria',  'UK',     300.00, 'maria@mail.org'),
('Hari',   'India',  250.00, 'hari@mail.net'),
('Neha',   'USA',    200.00, 'neha@mail.com');
```

**Clause order you will keep using (write order):**

```text
SELECT -> FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
```

The database does not _run_ them in this order (see Module 10, Logical Query Execution Order), but this is the order you must _write_ them in.

---

## SELECT, FROM, and LIMIT

`SELECT` chooses **which columns** you want to see, `FROM` names **which table** holds them, and `LIMIT` caps **how many rows** come back. Together they form the smallest useful query.

- **`SELECT *`:** Returns every column. Fine for exploring a table, but avoid it in production code because it returns unneeded data and breaks when columns change.
- **Column aliases (`AS`):** Rename a column in the output only (`total_spent AS revenue`). The table is not changed.
- **`LIMIT n`:** Returns at most `n` rows. **`OFFSET m`** skips the first `m` rows (used for pagination).
- **`LIMIT` without `ORDER BY` is not deterministic.** Tables have no guaranteed row order, so "the first 2 rows" can differ between runs.

### Application

- **Data Exploration:** Peeking at a few rows of an unfamiliar table before writing real logic.
- **Report Building:** Selecting only the fields a dashboard needs, with friendly column names.
- **Pagination:** Showing results 10 at a time in an application (`LIMIT 10 OFFSET 20`).

### Real-World Problem and How Topic Solves It

- **Problem:** A data analyst receives a 50-million-row `customers` table and runs `SELECT * FROM customers;` to "see what's in it." The query floods the network, freezes the client tool, and loads the database for everyone else.
- **Solution:** Selecting only the needed columns and capping rows with `LIMIT` returns instantly, uses a fraction of the resources, and still shows the table's structure.

### Industrial Example

An e-commerce admin panel lists customers 25 per page. Each page request runs `SELECT customer_id, first_name, country FROM customers ORDER BY customer_id LIMIT 25 OFFSET ...`, fetching only the columns the screen displays instead of the whole row.

### Code and Explanation

```sql
-- 1. Choose specific columns and cap the rows
SELECT first_name, country
FROM customers
LIMIT 2;

-- 2. Rename an output column with an alias
SELECT first_name, total_spent AS revenue
FROM customers
LIMIT 3;

-- 3. Deterministic pagination: ORDER BY + LIMIT + OFFSET
SELECT customer_id, first_name
FROM customers
ORDER BY customer_id
LIMIT 3 OFFSET 3;       -- rows 4 to 6
```

#### Code Breakdown:

- **`SELECT first_name, country`:** Returns only these two columns, in this order.
- **`AS revenue`:** Gives the output column a new label without touching the table.
- **`LIMIT 3 OFFSET 3`:** Skips 3 rows, then returns the next 3. Always pair it with `ORDER BY` so "page 2" is the same rows every time.

### Interview Questions

#### Question 1: Why is `SELECT *` discouraged in production queries?

- **Solution Approach:**
    - It returns every column, wasting network and memory on unused data.
    - It prevents the optimizer from using **covering indexes** (an index-only scan is impossible if you ask for all columns).
    - It breaks downstream code silently when columns are added, removed, or reordered.

#### Question 2: Is `SELECT * FROM t LIMIT 5` guaranteed to return the same 5 rows each time?

- **Solution Approach:**
    - No. Relational tables are unordered sets; without `ORDER BY` the engine returns rows in whatever order its plan produces (which can change after updates, `VACUUM`, or a different plan).
    - Always add `ORDER BY` (on a unique or tie-broken column) when the specific rows matter.

#### Question 3: What is the downside of `OFFSET`-based pagination on large tables?

- **Solution Approach:**
    - The database must still read and discard all skipped rows, so deep pages (`OFFSET 1000000`) get progressively slower.
    - **Keyset pagination** (`WHERE id > last_seen_id ORDER BY id LIMIT 25`) jumps straight to the right place using an index.

---

## DISTINCT

`DISTINCT` removes duplicate rows from the output so each unique combination appears once.

- **Single column:** `SELECT DISTINCT country` returns each country once.
- **Multiple columns:** `DISTINCT` applies to the **whole combination** of selected columns, not to each column separately.
- **`COUNT(DISTINCT col)`:** Counts unique non-NULL values instead of returning them.
- **NULL handling:** `DISTINCT` treats all NULLs as one group, so NULL appears once.

### Application

- **Building Dropdowns and Filters:** Listing every unique country or category in a product UI.
- **Data Profiling:** Checking how many distinct values a column has before deciding how to model it.
- **Cardinality Checks:** Verifying a column that should be unique actually is.

### Real-World Problem and How Topic Solves It

- **Problem:** A marketing team wants to know which countries they have customers in. A plain `SELECT country FROM customers` returns thousands of repeated values.
- **Solution:** `SELECT DISTINCT country` returns one row per country, instantly showing the market footprint.

### Industrial Example

A telecom analytics team counts how many unique subscribers used the network each day with `COUNT(DISTINCT subscriber_id)`. Counting raw rows would overstate usage because one subscriber generates many events.

### Code and Explanation

```sql
-- 1. Unique countries
SELECT DISTINCT country
FROM customers;

-- 2. Unique combinations (DISTINCT applies to the pair)
SELECT DISTINCT country, first_name
FROM customers
LIMIT 5;

-- 3. How many unique countries?
SELECT COUNT(DISTINCT country) AS unique_countries
FROM customers;
```

#### Code Breakdown:

- **`SELECT DISTINCT country`:** Returns India, USA, UK, and Canada once each.
- **`DISTINCT country, first_name`:** A row is a duplicate only if **both** values match another row. It does **not** give "unique countries with a sample name."
- **`COUNT(DISTINCT country)`:** Returns one number (4) instead of a list.

### Interview Questions

#### Question 1: Does `SELECT DISTINCT a, b` deduplicate `a` and `b` independently?

- **Solution Approach:**
    - No. It deduplicates the **combination** `(a, b)`. Two rows with the same `a` but different `b` both survive.

#### Question 2: How does `COUNT(DISTINCT col)` differ from `COUNT(col)` and `COUNT(*)`?

- **Solution Approach:**
    - `COUNT(*)` counts all rows. `COUNT(col)` counts non-NULL values. `COUNT(DISTINCT col)` counts unique non-NULL values.

#### Question 3: What is a performance concern with `DISTINCT` on large tables?

- **Solution Approach:**
    - It forces a sort or hash aggregation of the entire result to find duplicates, which can spill to disk. Often a sign that a join is producing duplicate rows that should be fixed at the source instead of hidden with `DISTINCT`.

---

## WHERE

`WHERE` filters rows **before** they are returned, keeping only the rows where the condition is true.

- **Comparison operators:** `=`, `<>` (or `!=`), `<`, `>`, `<=`, `>=`. SQL uses a **single `=`** for equality, not `==`.
- **Text values use single quotes** (`'India'`). Double quotes are for identifiers (column and table names).
- **Text comparison is case-sensitive in PostgreSQL:** `'India'` and `'INDIA'` are different values.
- **Three-valued logic:** A condition can be TRUE, FALSE, or UNKNOWN (when NULL is involved). `WHERE` keeps **only TRUE** rows.

### Application

- **Targeted Retrieval:** "Customers from India," "orders above $100," "events from yesterday."
- **Data Quality Checks:** Finding rows that violate expectations (negative amounts, impossible dates).
- **Reducing Work Early:** Filtering first means less data flows into later steps such as joins and aggregates.

### Real-World Problem and How Topic Solves It

- **Problem:** A finance analyst needs only high-value Indian customers for an audit, but the table holds customers from every country and spend level.
- **Solution:** `WHERE country = 'India' AND total_spent > 100` returns exactly the audit population and nothing else.

### Industrial Example

A fraud-monitoring job scans a transactions table every minute with `WHERE amount > 10000 AND created_at >= NOW() - INTERVAL '1 minute'`, filtering millions of rows down to a handful of alerts that need review.

### Code and Explanation

```sql
-- 1. Single condition
SELECT first_name, country, total_spent
FROM customers
WHERE country = 'India';

-- 2. Two conditions that must both be true
SELECT first_name, country, total_spent
FROM customers
WHERE country = 'India' AND total_spent > 100.00;

-- 3. Case sensitivity: this returns ZERO rows in PostgreSQL
SELECT *
FROM customers
WHERE country = 'INDIA';

-- 4. Not equal
SELECT first_name, country
FROM customers
WHERE country <> 'India';
```

#### Code Breakdown:

- **`country = 'India'`:** Keeps only rows whose country is exactly `India`.
- **`country = 'INDIA'`:** Matches nothing because stored values are `India`. Use `LOWER(country) = 'india'` or `ILIKE` for case-insensitive matching.
- **`<>`:** Excludes the given value. Rows where `country` is NULL are **also excluded** (NULL <> 'India' is UNKNOWN, not TRUE).

### Interview Questions

#### Question 1: Why does `WHERE country <> 'India'` not return rows where `country` is NULL?

- **Solution Approach:**
    - Comparing NULL to anything yields UNKNOWN, and `WHERE` keeps only TRUE rows. To include NULLs, write `WHERE country <> 'India' OR country IS NULL`, or use `IS DISTINCT FROM 'India'`.

#### Question 2: What is the difference between `WHERE` and `HAVING`?

- **Solution Approach:**
    - `WHERE` filters individual rows **before** grouping and cannot use aggregates. `HAVING` filters groups **after** aggregation (covered in Module 2).

#### Question 3: Can you use a column alias defined in `SELECT` inside `WHERE`?

- **Solution Approach:**
    - No. `WHERE` is evaluated before `SELECT`, so the alias does not exist yet. Repeat the expression, or wrap the query in a subquery or CTE.

---

## Logical Operators (AND, OR, NOT)

Logical operators combine multiple conditions in a single `WHERE` clause.

- **`AND`:** Both conditions must be true.
- **`OR`:** At least one condition must be true.
- **`NOT`:** Reverses a condition.
- **Precedence:** `NOT` is evaluated first, then `AND`, then `OR`. **Use parentheses** to make intent explicit.

### Application

- **Multi-Criteria Filtering:** "Indian customers who spent over $200, or any customer from the UK."
- **Exclusion Rules:** Removing flagged or test records with `NOT`.

### Real-World Problem and How Topic Solves It

- **Problem:** A manager asks for customers who are from India **or** the USA **and** have spent over $100. The analyst writes the condition without parentheses and unknowingly returns every Indian customer regardless of spend, because `AND` binds tighter than `OR`.
- **Solution:** Explicit parentheses, `(country = 'India' OR country = 'USA') AND total_spent > 100`, force the intended logic.

### Industrial Example

A compliance report flags accounts that are `(high_risk = true OR country IN (...)) AND NOT verified`. Misplaced parentheses in rules like this have caused real over-reporting and under-reporting, which is why analytics teams enforce parentheses in code review.

### Code and Explanation

```sql
-- 1. OR: either condition qualifies
SELECT first_name, country, total_spent
FROM customers
WHERE country = 'India' OR total_spent > 200.00;

-- 2. NOT: exclude a group
SELECT first_name, country
FROM customers
WHERE NOT country = 'India';

-- 3. Precedence trap: AND runs before OR
SELECT first_name, country, total_spent
FROM customers
WHERE country = 'India' OR country = 'USA' AND total_spent > 200;
-- Read as: India (any spend) OR (USA AND spend > 200)

-- 4. Parentheses make the intended logic explicit
SELECT first_name, country, total_spent
FROM customers
WHERE (country = 'India' OR country = 'USA') AND total_spent > 200;
```

#### Code Breakdown:

- **Query 3 vs 4:** Same conditions, different results. Query 3 returns every Indian customer (Amit, Aarav, Priya, Hari) plus only Americans who spent over 200 (none in this data). Query 4 returns only customers from either country who spent over 200 (Amit and Hari). Aarav drops out because his spend is NULL.
- **`NOT country = 'India'`:** Equivalent to `country <> 'India'` and has the same NULL behavior.

### Interview Questions

#### Question 1: What does `WHERE a = 1 OR b = 2 AND c = 3` evaluate as?

- **Solution Approach:**
    - `AND` has higher precedence, so it is `a = 1 OR (b = 2 AND c = 3)`. Always add parentheses to remove ambiguity.

#### Question 2: What does `TRUE AND NULL` evaluate to? What about `TRUE OR NULL`?

- **Solution Approach:**
    - `TRUE AND NULL` = UNKNOWN (so the row is dropped). `TRUE OR NULL` = TRUE. `FALSE AND NULL` = FALSE.

---

## IN

`IN` checks whether a value matches **any value in a list**, replacing a chain of `OR` conditions.

- **`IN (...)`:** Shorthand for multiple `=` comparisons joined by `OR`.
- **`NOT IN (...)`:** True when the value matches none of the list.
- **The NULL trap:** If the list for `NOT IN` contains a NULL, the whole condition never returns TRUE and the query returns **zero rows**.

### Application

- **Filtering by Category Lists:** "Orders with status in (`shipped`, `delivered`)."
- **Reusable Filters:** A list is easier to read, edit, and generate from code than a long `OR` chain.

### Real-World Problem and How Topic Solves It

- **Problem:** A report needs customers from USA, UK, and Canada. The `OR` version is long and error-prone: `country = 'USA' OR country = 'UK' OR country = 'Canada'`.
- **Solution:** `country IN ('USA', 'UK', 'Canada')` says the same thing in one readable line.

### Industrial Example

A data pipeline filters events with `WHERE event_type IN ('purchase', 'refund', 'chargeback')` to build a revenue-events table, with the list maintained in one place as new event types appear.

### Code and Explanation

```sql
-- 1. Match any value in the list
SELECT *
FROM customers
WHERE country IN ('USA', 'UK');

-- 2. Exclude a list
SELECT *
FROM customers
WHERE country NOT IN ('USA', 'UK');

-- 3. The NULL trap: this returns ZERO rows
SELECT *
FROM customers
WHERE country NOT IN ('USA', NULL);
```

#### Code Breakdown:

- **`IN ('USA', 'UK')`:** Equivalent to `country = 'USA' OR country = 'UK'`.
- **Query 3:** `country NOT IN ('USA', NULL)` expands to `country <> 'USA' AND country <> NULL`. The second part is always UNKNOWN, so no row can pass.
- **Real danger:** This usually appears as `NOT IN (SELECT ...)` where the subquery unexpectedly returns a NULL. Prefer `NOT EXISTS` for safety (see Module 8).

### Interview Questions

#### Question 1: Why can `NOT IN` return no rows even when matches obviously exist?

- **Solution Approach:**
    - If the list or subquery contains a NULL, `x NOT IN (...)` evaluates to UNKNOWN for every row. Fix by filtering NULLs out of the subquery or by using `NOT EXISTS`.

#### Question 2: Is `IN` with a long list slower than `OR`?

- **Solution Approach:**
    - No. The optimizer treats them equivalently. For very large lists, join against a temporary table or use `= ANY(array)` instead.

---

## BETWEEN

`BETWEEN` tests whether a value falls within a range and is **inclusive of both endpoints**.

- **`x BETWEEN a AND b`:** Same as `x >= a AND x <= b`.
- **`NOT BETWEEN`:** Values outside the range.
- **Order matters:** `BETWEEN 300 AND 100` returns nothing, because the lower bound must come first.
- **Dates and timestamps:** `BETWEEN '2026-01-01' AND '2026-01-31'` treats `'2026-01-31'` as midnight, so a timestamp later that day is excluded. Prefer `>= start AND < next_day` for timestamps.

### Application

- **Range Filters:** Price bands, salary ranges, age brackets.
- **Date Windows:** Reports for a given month or quarter.

### Real-World Problem and How Topic Solves It

- **Problem:** Marketing wants customers who spent between $100 and $300. Writing `>= 100 AND <= 300` works but is wordy, and analysts sometimes mistakenly use `>` and `<` and silently drop the boundary customers.
- **Solution:** `BETWEEN 100 AND 300` states the range clearly and includes both endpoints.

### Industrial Example

A billing system assigns tiered pricing with `CASE WHEN usage BETWEEN 0 AND 100 THEN 'tier_1' ...`. Knowing that `BETWEEN` is inclusive prevents boundary customers (usage exactly 100) from being placed in two tiers or none.

### Code and Explanation

```sql
-- 1. Inclusive range
SELECT first_name, total_spent
FROM customers
WHERE total_spent BETWEEN 100 AND 300;

-- 2. Equivalent long form
SELECT first_name, total_spent
FROM customers
WHERE total_spent >= 100 AND total_spent <= 300;

-- 3. Outside the range
SELECT first_name, total_spent
FROM customers
WHERE total_spent NOT BETWEEN 100 AND 300;
```

#### Code Breakdown:

- **Customers at exactly 300 (Maria) and 250 (Amit, Hari):** All included, since both endpoints are inclusive.
- **`NOT BETWEEN`:** Does **not** return rows with NULL `total_spent` (NULL comparisons are UNKNOWN).

### Interview Questions

#### Question 1: Is `BETWEEN` inclusive or exclusive?

- **Solution Approach:**
    - Inclusive on both ends: `x BETWEEN 1 AND 5` includes 1 and 5.

#### Question 2: Why is `BETWEEN` risky for timestamp ranges?

- **Solution Approach:**
    - A date literal like `'2026-01-31'` means `2026-01-31 00:00:00`, so events later on January 31 are missed. Use `ts >= '2026-01-01' AND ts < '2026-02-01'`.

---

## LIKE and ILIKE (Pattern Matching)

`LIKE` matches text against a pattern built from two wildcards. `ILIKE` (PostgreSQL) is the **case-insensitive** version.

- **`%`:** Matches zero, one, or many characters.
- **`_`:** Matches exactly one character.
- **`LIKE` is case-sensitive in PostgreSQL;** use `ILIKE` for case-insensitive matching.
- **Escaping:** To match a literal `%` or `_`, escape it with a backslash (`LIKE '100\%'`).
- **Performance:** A pattern with a **leading wildcard** (`'%n%'`) generally cannot use a normal B-tree index and may scan the whole table.

|Pattern|Meaning|Example match|
|---|---|---|
|`'A%'`|Starts with A|Amit, Aarav, Ana|
|`'%a'`|Ends with a|Diana, Priya, Ana, Maria, Neha|
|`'%n%'`|Contains a lowercase n|Diana, Ana, John|
|`'_a%'`|Second letter is a|Sarah, Aarav, Maria, Hari|
|`'__d%'`|Third letter is d|India|

### Application

- **Search Boxes:** "Customers whose name contains `ana`."
- **Data Cleaning:** Finding email addresses ending in a given domain.
- **Code Lookups:** Product SKUs starting with a prefix.

### Real-World Problem and How Topic Solves It

- **Problem:** Support staff need every customer with a `.com` email, but they only remember the ending. An exact `=` cannot express that.
- **Solution:** `WHERE email LIKE '%.com'` matches any email ending in `.com`, whatever comes before it.

### Industrial Example

A CRM's autocomplete runs `WHERE first_name ILIKE 'ana%'` as the user types. Because the pattern has a trailing (not leading) wildcard, a suitable index can serve it quickly.

### Code and Explanation

```sql
-- 1. Starts with 'A'
SELECT first_name, country
FROM customers
WHERE first_name LIKE 'A%';

-- 2. Ends with 'a'
SELECT first_name
FROM customers
WHERE first_name LIKE '%a';

-- 3. Contains 'n' anywhere
SELECT first_name
FROM customers
WHERE first_name LIKE '%n%';

-- 4. Second letter must be 'a'
SELECT first_name
FROM customers
WHERE first_name LIKE '_a%';

-- 5. Third letter of country is 'd'
SELECT DISTINCT country
FROM customers
WHERE country LIKE '__d%';

-- 6. Case-insensitive match (PostgreSQL)
SELECT first_name
FROM customers
WHERE first_name ILIKE 'a%';

-- 7. Emails ending with .com
SELECT first_name, email
FROM customers
WHERE email LIKE '%.com';
```

#### Code Breakdown:

- **Position matters:** `'A%'` anchors the start, `'%a'` anchors the end, `'%n%'` anchors nothing.
- **`'_a%'`:** One unknown character, then `a`, then anything. It matches _second letter is a_, **not** _ends with a_. (A common mix-up.)
- **`ILIKE 'a%'`:** Matches `Amit`, `Aarav`, and `Ana` regardless of case. Plain `LIKE 'a%'` (lowercase `a`) returns nothing, because every stored name starts with a capital letter.
- **NULL emails:** Ana's email is NULL, so she is excluded from any `LIKE` match.

### Interview Questions

#### Question 1: What is the difference between `%` and `_`?

- **Solution Approach:**
    - `%` matches any number of characters (including none). `_` matches exactly one character.

#### Question 2: Why can `LIKE '%smith'` be slow on a large indexed column?

- **Solution Approach:**
    - A leading wildcard means the engine cannot use the index's sorted order to jump to a starting point, so it scans every row. Options: trigram indexes (`pg_trgm`), full-text search, or storing a reversed column.

#### Question 3: How do you do a case-insensitive match in PostgreSQL vs other databases?

- **Solution Approach:**
    - PostgreSQL: `ILIKE`, or `LOWER(col) LIKE 'x%'`. MySQL's default collation is already case-insensitive for `LIKE`. SQL Server depends on the collation. Never assume; test.

---

## IS NULL and IS NOT NULL

`NULL` means **missing or unknown**, not zero and not an empty string. Because NULL is "unknown," it cannot be tested with `=`.

- **`IS NULL` / `IS NOT NULL`:** The only correct tests for missing values.
- **`col = NULL` never returns rows;** the result is always UNKNOWN.
- **NULL vs empty string:** `''` is a known empty value; NULL is no value. They behave differently.
- **Aggregates skip NULLs** (except `COUNT(*)`), which affects averages and counts.

### Application

- **Data Quality Audits:** Finding records with missing emails or amounts.
- **Distinguishing "Zero" from "Unknown":** A customer who spent $0 is different from one with no recorded spend.

### Real-World Problem and How Topic Solves It

- **Problem:** A data analyst writes `WHERE total_spent = NULL` to find customers with missing spend data and gets an empty result, wrongly concluding the data is complete.
- **Solution:** `WHERE total_spent IS NULL` correctly finds Aarav and John.

### Industrial Example

A nightly data-quality job counts missing values per column with `COUNT(*) - COUNT(column)` and alerts when the NULL percentage crosses a threshold.

### Code and Explanation

```sql
-- 1. Find missing values
SELECT first_name, total_spent
FROM customers
WHERE total_spent IS NULL;

-- 2. Find present values
SELECT first_name, total_spent
FROM customers
WHERE total_spent IS NOT NULL;

-- 3. The common mistake: returns ZERO rows
SELECT *
FROM customers
WHERE total_spent = NULL;

-- 4. Count NULLs per column
SELECT COUNT(*) - COUNT(total_spent) AS missing_spend,
       COUNT(*) - COUNT(email)       AS missing_email
FROM customers;
```

#### Code Breakdown:

- **`= NULL`:** Evaluates to UNKNOWN for every row, so nothing is returned and no error is raised, which makes the bug easy to miss.
- **`COUNT(*) - COUNT(col)`:** `COUNT(*)` counts all rows, `COUNT(col)` counts non-NULL values, so the difference is the number of NULLs.

### Interview Questions

#### Question 1: Why does `WHERE col = NULL` return nothing?

- **Solution Approach:**
    - NULL means unknown, so `col = NULL` is UNKNOWN for every row, and `WHERE` keeps only TRUE rows. Use `IS NULL`.

#### Question 2: How do NULLs affect `AVG`, `COUNT`, and `SUM`?

- **Solution Approach:**
    - `AVG`, `SUM`, `MIN`, `MAX`, and `COUNT(col)` ignore NULLs. `COUNT(*)` counts every row. So `AVG` is computed over non-NULL values only, which can overstate the typical value if NULL should mean zero.

#### Question 3: How would you compare two columns that may contain NULLs and treat NULL = NULL as equal?

- **Solution Approach:**
    - `a IS NOT DISTINCT FROM b` (NULL-safe equality). `a IS DISTINCT FROM b` is the NULL-safe "not equal."

---

## ORDER BY

`ORDER BY` sorts the result rows by one or more columns.

- **`ASC`:** Smallest to largest, or A to Z. This is the default.
- **`DESC`:** Largest to smallest, or Z to A.
- **Multiple columns:** Later columns break ties from earlier ones.
- **Position in the query:** After `WHERE` and `GROUP BY`, before `LIMIT`.
- **NULL ordering in PostgreSQL:** NULLs sort as **larger than every value**, so they come **last in ASC** and **first in DESC**. Control it with `NULLS FIRST` or `NULLS LAST`.
- **Aliases are allowed in `ORDER BY`** (unlike `WHERE`), because sorting happens after `SELECT`.

### Application

- **Leaderboards and Rankings:** Top customers by spend.
- **Reproducible Output:** Stable ordering for reports and for pagination.
- **Time-Based Views:** Most recent events first.

### Real-World Problem and How Topic Solves It

- **Problem:** A sales manager asks for the top 3 customers by spend, but the analyst sorts `DESC` and the top rows are customers with **NULL** spend, because PostgreSQL places NULLs first in descending order.
- **Solution:** `ORDER BY total_spent DESC NULLS LAST` pushes unknown values to the bottom, so real top spenders appear first.

### Industrial Example

A product-ranking service orders items with `ORDER BY rating DESC NULLS LAST, review_count DESC, product_id`. The third column guarantees a deterministic order when ratings and counts tie, so pages do not shuffle between requests.

### Code and Explanation

```sql
-- 1. Highest spenders first (NULLs appear at the TOP in PostgreSQL)
SELECT first_name, total_spent
FROM customers
ORDER BY total_spent DESC;

-- 2. Fix: push NULLs to the end
SELECT first_name, total_spent
FROM customers
ORDER BY total_spent DESC NULLS LAST;

-- 3. Multi-column sort: country A-Z, then spend high-to-low within each country
SELECT first_name, country, total_spent
FROM customers
ORDER BY country ASC, total_spent DESC NULLS LAST;

-- 4. Top 3 customers by spend
SELECT first_name, total_spent
FROM customers
ORDER BY total_spent DESC NULLS LAST
LIMIT 3;
```

#### Code Breakdown:

- **Query 1 vs 2:** Same data, different top rows. Query 1 starts with Aarav and John (NULL spend). Query 2 starts with Diana (310).
- **`ORDER BY country ASC, total_spent DESC`:** Sorts by country first, then orders customers within each country.
- **Clause order:** `ORDER BY` comes before `LIMIT`, so the sort happens first and the top rows are then taken.

### Interview Questions

#### Question 1: Where do NULLs appear with `ORDER BY` in PostgreSQL?

- **Solution Approach:**
    - Last for ascending, first for descending. Use `NULLS FIRST` / `NULLS LAST` to override. Other databases differ (MySQL treats NULL as smallest).

#### Question 2: Why add a unique column as the last `ORDER BY` key?

- **Solution Approach:**
    - Ties between rows leave their relative order undefined, which breaks pagination and test reproducibility. A unique tiebreaker such as the primary key makes the order fully deterministic.

#### Question 3: Can you `ORDER BY` a column that is not in the `SELECT` list?

- **Solution Approach:**
    - Yes, in most databases (including PostgreSQL), except when using `SELECT DISTINCT`, where the sort key must appear in the select list.

---

## Aggregate Functions (COUNT, SUM, AVG, MAX, MIN)

Aggregate functions collapse many rows into a **single summary value**.

- **`COUNT()`:** Counts rows. `COUNT(*)` counts all rows, `COUNT(col)` counts non-NULL values.
- **`SUM()`:** Adds numeric values.
- **`AVG()`:** Mean of non-NULL values.
- **`MAX()` / `MIN()`:** Highest and lowest value (works on numbers, text, and dates).
- **NULLs are ignored** by every aggregate except `COUNT(*)`.
- **Mixing rule:** A plain column cannot appear next to an aggregate in `SELECT` unless the data is grouped (`GROUP BY`) or the aggregate is a window function (`OVER`).

### Application

- **KPIs:** Total revenue, average order value, number of customers.
- **Boundaries:** Earliest signup date, largest order.
- **Sanity Checks:** Row counts before and after a data load.

### Real-World Problem and How Topic Solves It

- **Problem:** Leadership asks for total revenue, average spend, and the biggest and smallest purchase, and wants them without exporting the table to a spreadsheet.
- **Solution:** One query with `SUM`, `AVG`, `MAX`, and `MIN` returns all four numbers in a single pass over the data.

### Industrial Example

A daily dashboard job runs `SELECT COUNT(*), SUM(amount), AVG(amount), MAX(amount) FROM orders WHERE order_date = CURRENT_DATE`, producing the headline tiles from one query.

### Code and Explanation

```sql
-- 1. One-row summary of the whole table
SELECT COUNT(*)            AS total_rows,
       COUNT(total_spent)  AS rows_with_spend,
       SUM(total_spent)    AS total_revenue,
       AVG(total_spent)    AS average_spend,
       MAX(total_spent)    AS highest_purchase,
       MIN(total_spent)    AS lowest_purchase
FROM customers;

-- 2. AVG ignores NULLs; treat NULL as 0 when that is the business meaning
SELECT AVG(total_spent)                AS avg_ignoring_nulls,
       AVG(COALESCE(total_spent, 0))   AS avg_nulls_as_zero
FROM customers;

-- 3. This FAILS: plain column mixed with an aggregate, no GROUP BY
-- SELECT first_name, SUM(total_spent) FROM customers;
-- ERROR: column "customers.first_name" must appear in the GROUP BY clause
--        or be used in an aggregate function
```

#### Code Breakdown:

- **`COUNT(*)` vs `COUNT(total_spent)`:** 10 rows, but only 8 have a spend value.
- **`AVG` NULL behavior:** The first average divides by 8, the second by 10, so the numbers differ. Which is correct depends on whether "no record" means "unknown" or "zero."
- **The error in query 3:** The database cannot decide which single `first_name` to show next to one total. Section below shows the two valid fixes.

### Interview Questions

#### Question 1: What is the difference between `COUNT(*)`, `COUNT(col)`, and `COUNT(DISTINCT col)`?

- **Solution Approach:**
    - All rows; non-NULL values; unique non-NULL values.

#### Question 2: Why does `AVG(col)` differ from `SUM(col) / COUNT(*)`?

- **Solution Approach:**
    - `AVG` divides by the count of non-NULL values; `COUNT(*)` includes rows where `col` is NULL, so the denominator is larger.

#### Question 3: What does `SUM` return on an empty set, and what does `COUNT` return?

- **Solution Approach:**
    - `SUM`, `AVG`, `MIN`, and `MAX` return NULL (not 0). `COUNT` returns 0. Wrap with `COALESCE(SUM(x), 0)` when a report needs zero.

---

## Row Detail + Totals Together: `OVER()` vs `GROUP BY`

Aggregates normally **collapse** rows. Two tools control that behavior, and choosing the right one is a core SQL skill.

- **`GROUP BY`:** Collapses rows into one row per group (per country, per month). Individual rows disappear.
- **`OVER()` (window function):** Computes the aggregate but **keeps every row**, attaching the result as an extra column.

|You need|Use|
|---|---|
|Every individual row **plus** a total or average beside it|`aggregate(...) OVER ()`|
|One summary row **per category**|`GROUP BY`|

### Application

- **Percent-of-Total Reports:** Each customer's share of overall revenue.
- **Benchmarking:** Showing each row next to the company average or maximum.
- **Category Summaries:** Customers and revenue by country.

### Real-World Problem and How Topic Solves It

- **Problem:** A manager wants a customer list that also shows each person's spend next to the company total. Adding `SUM(total_spent)` beside `first_name` throws an error, and using `GROUP BY` first_name loses the company-wide total.
- **Solution:** `SUM(total_spent) OVER ()` attaches the company total to every row without collapsing anything. For a by-country summary, `GROUP BY country` is the right tool.

### Industrial Example

A sales dashboard shows each rep's revenue next to the team total and each rep's percentage of it (`revenue / SUM(revenue) OVER ()`), while a separate regional tile uses `GROUP BY region` for one row per region.

### Code and Explanation

```sql
-- Scenario A: keep every row, add totals beside them
SELECT first_name,
       country,
       total_spent,
       SUM(total_spent) OVER ()  AS company_total,
       AVG(total_spent) OVER ()  AS company_average,
       MAX(total_spent) OVER ()  AS highest_purchase
FROM customers;

-- Scenario A+: percent of total per customer
SELECT first_name,
       total_spent,
       ROUND(100.0 * total_spent / SUM(total_spent) OVER (), 1) AS pct_of_total
FROM customers
WHERE total_spent IS NOT NULL
ORDER BY pct_of_total DESC;

-- Scenario B: collapse into one row per country
SELECT country,
       COUNT(*)           AS total_customers,
       SUM(total_spent)   AS country_revenue
FROM customers
GROUP BY country
ORDER BY country_revenue DESC NULLS LAST;
```

#### Code Breakdown:

- **`SUM(total_spent) OVER ()`:** The empty parentheses mean "the whole table is one window." Every row gets the same total, and no row is removed.
- **`GROUP BY country`:** Produces one row per country, so individual names can no longer appear unless aggregated.
- **Decision rule:** Individual rows + totals → `OVER()`. Categorical rows + totals → `GROUP BY`. (`PARTITION BY` inside `OVER()` gives per-group totals while keeping rows; see Module 6.)

### Interview Questions

#### Question 1: What is the difference between `GROUP BY` and a window function with `OVER()`?

- **Solution Approach:**
    - `GROUP BY` reduces the number of rows (one per group). A window function keeps all input rows and adds the computed value as a new column.

#### Question 2: Why does `SELECT name, SUM(amount) FROM t` fail?

- **Solution Approach:**
    - `SUM` returns one value for the whole table while `name` has many; the engine cannot pair them. Fix with `GROUP BY name` (one row per name) or `SUM(amount) OVER ()` (keep all rows).

#### Question 3: Can you filter on a window function result in `WHERE`?

- **Solution Approach:**
    - No. Window functions are evaluated after `WHERE`. Wrap the query in a subquery or CTE and filter in the outer query.

---

- "Name ends with A" used `LIKE '_a%'`, which actually means _second letter is a_. The correct pattern is `'%a'`.
- `WHERE country = 'INDIA'` returns nothing in PostgreSQL because text comparison is case-sensitive.

----------------------------------------------------------------

**String Manipulation Functions**

String manipulation functions are built-in SQL operations designed to alter, combine, inspect, format, or extract specific parts of textual data stored inside text columns (VARCHAR, TEXT, CHAR).

**Application**

- Normalizing user inputs (e.g., standardizing email addresses or names to lower/upper case).
- Masking or truncating sensitive data like phone numbers, passwords, or credit card details.
- Concatenating separate columns (e.g., combining first_name and last_name into full_name).
- Cleaning extra whitespace or unwanted characters from user inputs before generating reports.

**Real-World Problem & Solution**

Problem:  Data entering modern web systems comes from various input fields, mobile apps, or legacy batch imports. Names often end up with inconsistent casing (JOHn, alice), trailing whitespace ("Sita  "), or split across multiple fields. Generating clean, professional invoice labels or automated communication becomes difficult without clean text formatting.

Solution:  SQL string functions allow us to trim trailing spaces, force casing consistency, and merge values dynamically at query execution time without altering raw source data permanently.

**Industrial Example**

An enterprise healthcare system generates automated SMS appointment reminders. The database stores patient first and last names separately, often with erratic spacing entered by front-desk registration staff. The application uses string functions to generate a unified, properly capitalized, clean greeting line for the SMS API payload.

```sql
SELECT 
    patient_id,
    -- 1. Combines first and last names with a space
    CONCAT(TRIM(first_name), ' ', TRIM(last_name)) AS formatted_full_name,

    -- 2. Converts the email address to lowercase for standard matching
    LOWER(email) AS normalized_email,

    -- 3. Extracts the first 3 letters of the city for visual tagging
    UPPER(SUBSTRING(city, 1, 3)) AS city_code,

    -- 4. Replaces outdated area identifiers in the address
    REPLACE(address, 'Old Wing', 'Main Building') AS updated_address
FROM Patient;

SELECT 
    TRIM(LOWER('  User@Domain.COM  ')) AS cleaned_email,
    SPLIT_PART('user@domain.com', '@', 2) AS domain_name,
    SUBSTRING('PROMO_2026_SUMMER' FROM 7 FOR 4) AS promo_year,
    LENGTH('Data Engineer') AS char_count;
```

- CONCAT(): Joins string arguments into a single continuous text string.

- TRIM(): Strips leading and trailing spaces that could cause layout or comparison issues.

- LOWER() / UPPER(): Converts text characters into consistent lowercase or uppercase letters.

- SUBSTRING(string, start, length): Cuts out a specific segment of text starting at position 1 for a length of 3.

- REPLACE(string, target, replacement): Scans text for a specific substring and substitutes it with the target string.

- TRIM(LOWER(...)) strips leading/trailing whitespaces and converts characters to lowercase.

- SPLIT_PART(string, delimiter, index) splits the text on @ and extracts the 2nd part (domain.com).

- SUBSTRING(string FROM start FOR length) extracts 4 characters starting at index 7 (2026).

- LENGTH(string) returns total character count (13).

**Important Points**

- SQL string indexing is 1-based, not 0-based like Python or C++. SUBSTRING('Data', 1, 1) returns 'D'.

- CONCAT() automatically converts NULL values to empty strings in standard PostgreSQL, whereas simple concatenation using first_name || ' ' || last_name will return NULL if any operand is NULL.

- Heavy use of non-sargable functions like LOWER(column_name) in WHERE clauses prevents SQL from using standard indexes, leading to slower full table scans.

----

**Date & Time Functions**

Date and Time functions operate on temporal data types (DATE, TIME, TIMESTAMP, INTERVAL). They enable extraction of specific calendar units, calculation of durations, and alignment of event timestamps into operational periods

**Application**

- Calculating elapsed time (e.g., patient length-of-stay, order delivery duration).

- Truncating precise event timestamps down to day, week, or month levels for aggregations.

- Scheduling future events (e.g., adding follow-up intervals to a discharge date).

- Filtering data dynamically relative to the current date (CURRENT_DATE).

**Real-World Problem & Solution**

Problem: Raw application data records exact event occurrences using precise timestamps (e.g., 2026-03-14 14:22:09.102). Executing executive monthly revenue or throughput analysis directly on precise timestamps yields fragmented groups instead of unified daily or monthly totals.

Solution: Date truncation and extraction functions collapse sub-day noise, permitting clean temporal aggregations over months, quarters, or years.

**Industrial Example**

A hospital operations platform tracks patient readmission rates within a mandatory 30-day monitoring window. The query must compute the exact integer count of days between a patient's initial discharge timestamp and their subsequent admission timestamp to flag 30-day readmissions automatically.

```sql
SELECT 
    appointment_id,
    appointment_date,

    -- 1. Extracts specific integer components from a timestamp
    EXTRACT(YEAR FROM appointment_date) AS appt_year,
    EXTRACT(MONTH FROM appointment_date) AS appt_month,

    -- 2. Truncates time precision down to the starting midnight of the month
    DATE_TRUNC('month', appointment_date) AS month_start,

    -- 3. Calculates follow-up date by adding an interval
    appointment_date + INTERVAL '14 days' AS two_week_followup,

    -- 4. Computes exact age in days from current date
    CURRENT_DATE - appointment_date::DATE AS days_elapsed
FROM Appointment;
```

- EXTRACT(part FROM timestamp): Isolates a numeric date unit (e.g., YEAR, MONTH, DAY, DOW for day of week).

- DATE_TRUNC('unit', timestamp): Sets all temporal units finer than the specified target unit to their minimum starting values (e.g., truncating to 'month' sets the day to 01 and time to 00:00:00).

- INTERVAL: Represents a duration of time (e.g., '14 days', '2 hours') that can be added or subtracted directly from temporal values.

Important- 

- Time zones matter: Always store raw event data in standard TIMESTAMP WITH TIME ZONE (UTC) and convert to local time zones at the reporting layer.

- Subtracting two DATE types directly returns an integer representing total days, whereas subtracting two TIMESTAMP types yields an INTERVAL.

---

**NULL-Handling Functions**

NULL-handling functions evaluate, replace, or manipulate NULL values—a state in SQL representing unknown, missing, or inapplicable data.

**Application**

- Providing default fallback values for missing attributes in user interfaces or reports.

- Preventing catastrophic run-time errors such as Division-by-Zero (NULLIF).

- Normalizing data across optional table attributes prior to mathematical operations.

**Real-World Problem & Solution**

Problem: In SQL, any arithmetic operation involving a NULL value evaluates directly to NULL (500 + NULL = NULL). If a patient billing query sums base_fee + procedure_fee + emergency_surcharge, and emergency_surcharge is NULL for standard patients, the calculated total bill defaults entirely to NULL, resulting in uncollected revenue.

Solution: COALESCE converts missing NULL values into a safe numeric fallback (like 0.00) before running mathematical calculations.

**Industrial Example**

A financial audit dashboard calculates the average collection efficiency ratio: (paid_amount / total_amount). If an unpaid invoice has a paid_amount recorded as 0, or an incomplete draft bill has a total_amount recorded as 0, standard division throws a critical division-by-zero database exception.

```sql
SELECT 
    bill_id,
    patient_id,
    total_amount,

    -- 1. Replaces NULL paid_amount with 0.00 for clean mathematical operations
    COALESCE(paid_amount, 0.00) AS safe_paid_amount,

    -- 2. Provides default text string when contact details are absent
    COALESCE(payment_mode, 'Pending Selection') AS display_payment_mode,

    -- 3. Prevents division-by-zero error by evaluating denominator to NULL if zero
    total_amount / NULLIF(paid_amount, 0) AS payment_multiple
FROM Bill;
```

**COALESCE(val1, val2, ...)**: Evaluates parameters in sequential order and returns the first non-NULL value.

**NULLIF(arg1, arg2):** Compares two expressions. If arg1 == arg2, it returns NULL; otherwise, it returns arg1.

**Important Points**

- NULL does not equal NULL in standard SQL conditional expressions (NULL = NULL evaluates to UNKNOWN, not TRUE). Always use IS NULL or IS NOT NULL.

- Aggregate functions (like SUM(), AVG(), COUNT()) systematically ignore NULL values, except COUNT(*), which counts physical rows regardless of content.

---

**Math & Numeric Functions**

Math functions perform standard mathematical, statistical, and rounding operations on numeric columns (INT, DECIMAL, NUMERIC, FLOAT).

**Application**

- Rounding floating-point currency calculations to standard financial precision (2 decimal places).

- Binning data into fixed numerical cohorts or buckets using floor and ceiling logic.

- Determining parity or routing logic using modulo logic (MOD).

- Calculating absolute distance or error variances using ABS().

**Real-World Problem & Solution**

Problem: Financial systems calculating percentage-based taxes or discounts generate long decimal values (e.g., $104.34891). Inserting unrounded floating-point calculations into downstream accounting balances causes ledger mismatch errors due to fractional cent discrepancies.

Solution: SQL numeric functions apply deterministic rounding rules (ROUND, CEIL, FLOOR) to guarantee precision across analytical queries.

**Industrial Example**

A hospital logistics system distributes inventory across specialized transport bins with fixed capacities. Using modulo operations (MOD), the system assigns incoming shipment batch numbers to alternating loading docks, ensuring equal load balance.

```sql
SELECT 
    bill_id,
    total_amount,

    -- 1. Rounds GST calculation precisely to 2 decimal places
    ROUND(total_amount * 0.18, 2) AS gst_amount,

    -- 2. Forces upward rounding to the next full integer
    CEIL(total_amount) AS billed_ceiling,

    -- 3. Forces downward rounding to the preceding integer
    FLOOR(total_amount) AS billed_floor,

    -- 4. Determines whether the bill ID is odd or even for sample routing
    MOD(bill_id, 2) AS audit_bucket,

    -- 5. Evaluates absolute difference relative to a target baseline
    ABS(total_amount - 5000.00) AS deviation_from_target
FROM Bill;
```

ROUND(numeric_expression, decimal_places): Rounds a number to the specified decimal precision.

CEIL(val) / CEILING(val): Returns the smallest integer greater than or equal to the argument.

FLOOR(val): Returns the largest integer less than or equal to the argument.

MOD(dividend, divisor): Returns the remainder of the integer division.

ABS(val): Eliminates the negative sign, returning absolute distance.

**Important Points**

- Floating-point types (FLOAT, REAL) introduce binary rounding inaccuracies. Always store currency, tax, and precise calculations using exact numeric types (NUMERIC or DECIMAL).

- ROUND() behavior on exact half-points (e.g., 2.5) can vary by database engine (Banker's rounding vs. Round away from zero). Verify your database engine's specifications when building financial pipelines.

---

**Deep Aggregations (SUM, AVG, MIN, MAX)**

**Real-World Problem and Solution**   
Problem: A financial analytics team needs to compute product performance metrics from a transactions table. The raw data contains missing ratings (NULL), duplicate transaction logs due to partial retries, and needs breakdown totals for completed vs. refunded amounts in one report. Simple AVG() or SUM() functions yield incorrect results because standard AVG() drops NULL rows (skewing averages upward), duplicate payment entries inflate SUM(), and calculating conditional totals requires multiple costly table scans using subqueries.

Solution: Applying advanced aggregation patterns solves all three issues:
1. COALESCE inside AVG() forces missing values to evaluate as 0, representing true dataset averages.
2. SUM(DISTINCT ...) prevents double-counting duplicate payment IDs.
3. FILTER (or conditional CASE WHEN within SUM()) computes completed and refunded amounts in a single table pass.

**Industrial Example**  
An e-commerce payment gateway evaluates monthly fraud risk profiles across customer accounts. The database engine needs to evaluate total transaction volume, distinct count of used payment cards, average purchase value (accounting for zero-dollar promo redemptions), and total refunded amounts per user—all computed dynamically over millions of streaming events per hour without degrading performance.

```sql
SELECT 
    user_id,

    -- 1. SUM(DISTINCT ...): Excludes duplicate payment amounts caused by row fan-out
    SUM(DISTINCT payment_amount) AS unique_payment_sum,

    -- 2. Standard AVG vs. COALESCE AVG:
    AVG(rating) AS avg_rating_ignoring_nulls,
    AVG(COALESCE(rating, 0)) AS avg_rating_including_nulls,

    -- 3. Filtered Aggregation (ANSI standard / PostgreSQL / SQLite / Snowflake):
    SUM(payment_amount) FILTER (WHERE status = 'completed') AS completed_revenue,

    -- 4. Conditional Aggregation (Universal dialect compatibility using CASE WHEN):
    SUM(CASE WHEN status = 'refunded' THEN payment_amount ELSE 0 END) AS refunded_revenue,

    -- 5. Combining Aggregations with Windowing (Running Cumulative Total):
    SUM(payment_amount) OVER (
        PARTITION BY user_id 
        ORDER BY transaction_timestamp 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_user_spend

FROM transactions
GROUP BY user_id, transaction_timestamp, payment_amount;
```

- SUM(DISTINCT payment_amount) deduplicates identical values within the group before summing, protecting against fan-out errors.AVG(rating) evaluates $\frac{\text{sum of   non-null ratings}}{\text{count of non-null ratings}}$. 

- AVG(COALESCE(rating, 0)) converts NULL entries to 0, evaluating 
$\frac{\text{sum of non-null ratings}}{\text{total count of all rows}}$, preventing upward skew.

- SUM(...) FILTER (WHERE ...) evaluates the aggregate function only over rows that satisfy the specified boolean predicate.
- SUM(CASE WHEN status = 'refunded' ...) provides universal dialect support for conditional aggregation: non-matching rows return 0 (or NULL), which adds nothing to the running total.
- SUM(...) OVER (...) demonstrates aggregate functions acting as window functions to construct running cumulative metrics without folding rows.

NULL Elimination Warning: All standard aggregate functions (SUM, AVG, MIN, MAX, COUNT(col)) automatically ignore NULL values, with one notable exception: COUNT(*), which counts table rows regardless of content.

COUNT(column) vs. COUNT(ashtrick) COUNT(column) only counts rows where column IS NOT NULL. COUNT(ashtrick) counts all matching table rows.

FILTER Clause vs. WHERE Clause: Filtering inside an aggregate (e.g., SUM(amount) FILTER (WHERE status = 'completed')) isolates logic per column. A global query WHERE clause filters rows before any aggregation occurs across all columns.

Empty Set Behavior: Aggregating over an empty result set returns NULL for SUM, AVG, MIN, and MAX. However, COUNT over an empty set returns 0.

---

**HAVING Clause** - 

For example: "Show me only the countries where the total revenue is greater than 100."

If you try to use WHERE country_total > 100, the database will throw an error because WHERE filters rows before any groupings or math calculations happen. To filter grouped buckets, we use HAVING.

```sql
SELECT country,
        SUM(total_spent) AS country_total,
        COUNT(*) AS total_customers
FROM customers
GROUP BY country
HAVING country_total > 100.00;
```

WHERE filters individual records before GROUP BY executes.  
HAVING filters aggregated summaries after GROUP BY executes. 

The strict order of execution always goes: FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT

--------------------------------------------------

**COUNT(DISTINCT)**- When aggregating, there will be need to count how many unique items exist within a group, rather than just counting every row.

Example - If we want to know how many unique customers from each country made a purchase, we wrap DISTINCT inside a COUNT() function.

```sql
SELECT country,
        COUNT(customer_id) AS total_rows,
        COUNT(DISTINCT customer_id) AS unique_customer_count
FROM customers
GROUP BY country;
```

COUNT() to measure raw transactional activity or event frequency (e.g., total clicks), but COUNT(DISTINCT) to find our actual unique user base (e.g., Daily Active Users / DAU).

------------------------------------

**Multi-Column Grouping** - e.g., Total revenue by Country AND by Year, or Total sales by Store AND by Department.

```sql
SELECT country,
        first_name,
        SUM(total_spent) AS total_amount
FROM customers
GROUP BY country, first_name;
```

When we pass multiple columns to a GROUP BY clause, the database looks for unique combinations of those values.

- If "Amit" had three rows for "India", they would collapse into a single row.

- The rows are only kept separate if the combination is completely unique (e.g., India | Amit vs India | Ananya).

-----------------

**Conditional Logic using CASE WHEN** - 

CASE WHEN acts exactly like an if-elif-else statement in Python. It allows you to create new categorical buckets or flags on the fly based on conditions.

Ex- classify customers into spend categories based on their total_spent column:

1. If they spent more than $300, they are a 'VIP'.

2. If they spent between $1 and $300, they are 'Regular'.

3. If their spend is missing (NULL), they are 'Inactive'

```sql
SELECT first_name,
        total_spent,
        CASE 
        WHEN total_spent > 300.00 THEN 'VIP'
        WHEN total_spent BETWEEN 1.00 AND 300.00 THEN 'Regular'
        ELSE 'Inactive'
    END AS customer_segment
FROM customers;
```

--------------------

**The Core Joins** - 

Right now, all data lives in a single customers table. In production databases, data is split up into multiple tables to keep things organized (Normalization). To do anything meaningful, we have to connect them horizontally using matching keys.

```sql
-- Insert mock transactional data

DROP TABLE IF EXISTS orders;

CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    product_name VARCHAR(50),
    order_date DATE
);

INSERT INTO orders VALUES (101, 1, 'Laptop', '2026-06-01');

INSERT INTO orders VALUES (102, 1, 'Mouse', '2026-06-02');

INSERT INTO orders VALUES (103, 2, 'Phone', '2026-06-03');

INSERT INTO orders VALUES (104, 3, 'Book', '2026-06-04');

INSERT INTO orders VALUES (105, 99, 'Keyboard', '2026-06-05');
```

**THE INNER JOIN** - returns rows where there is a match in both tables.

```sql
SELECT c.customer_id, c.first_name, o.product_name, o.order_date
FROM customers AS c
INNER JOIN orders AS o
        ON c.customer_id = o.customer_id;
```

The INNER JOIN Command tells the database engine that we want to combine rows from customers with rows from orders horizontally based on a matching relationship.

In INNER JOIN order doesn't matter at all. It cares only about finding a strict match between both tables. Because a match is a mutual relationship, it doesn’t matter if Table A looks for Table B, or Table B looks for Table A.

ON - The ON clause is the matching condition or the "bridge" between the two tables. It tells the database engine exactly how the rows in the left table connect to the rows in the right table

----------------------------------------------

**LEFT JOIN** (keep everything on the left) - A LEFT JOIN takes all rows from the left table (the one right after the FROM clause), and brings in matching information from the right table if it exists.  
If there is no match, it still keeps the left row but fills the right table's columns with None (NULL).

```sql
SELECT c.customer_id,c.first_name,o.product_name,o.order_date
        FROM customers AS c
        LEFT JOIN orders AS o
        ON c.customer_id =o. customer_id;
```

LEFT JOIN is ORDER dependent. It treats the first table as the absolute source of truth (the master list) and makes the second table completely optional.

**Case 1: customers on the Left**

FROM customers AS c  
LEFT JOIN orders AS o  
ON c.customer_id = o.customer_id

Meaning: "Show me every single customer we have in our registry, and append their order details if they have any."
Result: You will see all 5 customers, including Ananya and Kevin (who have 0 orders and show up with None).

**Case 2: orders on the Left**

FROM orders AS o  
LEFT JOIN customers AS c  
ON o.customer_id = c.customer_id

Meaning: "Show me every single order that was ever placed in our system, and append the customer's name next to it if we know who they are."
Result: Ananya and Kevin disappear completely (because they don't have rows in the orders table). Instead, Order 105 (the Keyboard bought by customer_id = 99) stays in the results with a customer name of None.

-------------------

**RIGHT JOIN** - A RIGHT JOIN is the exact mirror image of a LEFT JOIN. It takes all rows from the right table (the one specified after the RIGHT JOIN keyword), and brings in matching information from the left table if it exists.  
If there is no match, it keeps the right row and fills the left table's columns with None (NULL).

```sql
SELECT c.customer_id, 
       c.first_name, 
       o.product_name, 
       o.order_date
FROM customers AS c
RIGHT JOIN orders AS o 
    ON c.customer_id = o.customer_id;
```

------------

**FULL OUTER JOIN** - combination of both left join and right join. returns all rows from both tables.  
- If there is a match based on the ON key, it matches them horizontally.

- If a row in the left table has no match, it keeps it and fills the right side with None.

- If a row in the right table has no match, it keeps it and fills the left side with None.

RIGHT JOIN, native SQLite does not support the explicit keyword FULL OUTER JOIN. However,  in production warehouses (BigQuery, Snowflake, etc.), it is natively supported.

Since both RIGHT JOIN and FULL OUTER JOIN is not supported in SQLite. JUGAAD to let me understand -

```sql
-- Left side: Keeps all customers
SELECT c.customer_id AS customer_id, c.first_name, o.product_name, o.order_date
FROM customers AS c
LEFT JOIN orders AS o ON c.customer_id = o.customer_id

UNION

-- Right side: Keeps all orders
SELECT o.customer_id AS customer_id, c.first_name, o.product_name, o.order_date
FROM orders AS o
LEFT JOIN customers AS c ON o.customer_id = c.customer_id;
```

-------

**UNION vs UNION ALL** - allows to glue tables vertically (stacking rows on top of each other), whereas JOINS connect tables horizontally.

1. UNION ALL (The Fast Stack)
UNION ALL takes two queries and just stacks them together. It doesn't care if there are duplicate rows; it just copies everything over. Because it doesn't check for duplicates, it is incredibly fast.

2. UNION (The Deduplicated Stack)
UNION stacks the queries, but then it runs an extra sorting process under the hood to look for duplicate rows and destroys them, leaving only unique rows. It is slower because of this extra scanning step.

```sql
SELECT first_name, country 
        FROM customers 
        WHERE country = 'India'

UNION ALL

SELECT first_name, country 
        FROM customers 
        WHERE total_spent > 100.00
```

-----------

**INTERSECT & EXCEPT**

INTERSECT is a set operator that compares the results of two SELECT statements and returns only the distinct rows that are present in both query result sets.

EXCEPT or MINUS is a set operator that compares two SELECT statements and returns the distinct rows from the first query that do not exist in the second query's result set.

**Application**

INTERSECT:
- Finding overlapping records across systems (e.g., patients who visited both the Outpatient Department and the Inpatient Department).

- Identifying dual-role entities (e.g., employees who are also registered customers).

- Cross-validating data consistency across multiple regional databases.

EXCEPT:
- Finding missing or non-converted records (e.g., registered patients who have never booked an appointment).

- Data quality auditing (e.g., finding orphaned billing records that lack matching primary patient entries).

- Excluding specific blacklisted or flagged records from an analytical cohort.

**Example -**

INTERSECT:
An enterprise healthcare platform analyzes patient participation across two separate clinical trials. They use INTERSECT to generate a report of patients enrolled in both Trial A and Trial B simultaneously to monitor for contraindications.

EXCEPT:
A medical billing system compares generated insurance claims against settled insurance payments. Running EXCEPT between Claims_Issued and Claims_Paid isolates unpaid or pending claims that require immediate staff follow-up.

```sql
-- INTERSECT

-- EXCEPT

SELECT patient_id, patient_name, city FROM Q1_Emergency_Patients
INTERSECT
SELECT patient_id, patient_name, city FROM Q2_Inpatient_Patients;

SELECT patient_id, patient_name, city FROM Q1_Emergency_Patients
EXCEPT
SELECT patient_id, patient_name, city FROM Q2_Inpatient_Patients;
```

---

**SELF JOIN** - Table is joined to itself.

Why on earth would you join a table to itself? This is a mandatory pattern whenever your data has a hierarchy stored within a single table—such as an organizational chart where an employee row contains a pointer column to their manager's employee ID.

Temporary company directory table:-

```sql
DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    emp_id INT,
    emp_name VARCHAR(50),
    manager_id INT
);

INSERT INTO employees VALUES (1, 'Rohan (CEO)', NULL);

INSERT INTO employees VALUES (2, 'Vikram (VP)', 1);

INSERT INTO employees VALUES (3, 'Amit (Analyst)', 2);

INSERT INTO employees VALUES (4, 'Ananya (Engineer)', 2);
```

PROBLEM - print a list showing every employee's name along with their manager's name.

We can't get it from a single row. You have to take the table, duplicate it in memory, and look up the manager_id against the emp_id.

```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
        FROM employees AS e
        LEFT JOIN employees AS m
        ON e.manager_id = m.emp_id;
```

----------

**CROSS JOIN** - A CROSS JOIN is a unique intermediate operation that doesn't use an ON clause. It matches every single row from Table A with every single row from Table B.  

CROSS JOIN are used to generate "master combinations" or base matrices.  

Example: If you have a table of 100 products and a table of 12 months, and you want to build a dashboard that tracks inventory status for every product in every single month (even if sales were zero), you CROSS JOIN products with months to create your initial grid, then LEFT JOIN your actual sales records against it!

```sql
SELECT c.first_name, o.product_name
FROM customers AS c
CROSS JOIN orders AS o;
```

***REST ALL JOINS ARE IN DBMS NOTES IN DETAIL***

----

**IMPORTANT JOINS**

**Semi-Join:** A filtering pattern that returns rows from a left table if at least one matching row exists in a right table. Crucially, it never duplicates rows from the left table, regardless of how many matches exist in the right table.  
Used to filter an primary entity list based on the existence of related activity (e.g., "Find all users who have placed at least one order") without altering the cardinality (row count) of the primary dataset.

**Anti-Join:** The inverse of a semi-join. It returns rows from a left table if no matching row exists in a right table.  
Used to identify missing references, non-responders, or inactive entities (e.g., "Find all registered users who have never placed an order").

Example -   
An enterprise cybersecurity platform checks system logins against a table of flagged IP addresses. The engine uses a Semi-Join to isolate internal accounts that have logged in from any flagged IP address. It uses an Anti-Join to isolate corporate servers that have never received security patch updates.

```sql
-- SEMI-JOIN

-- ANTI-JOIN

-- Pattern A: Correlated EXISTS Subquery (Preferred for Performance)
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
);

-- Pattern B: IN Subquery
SELECT c.customer_id, c.customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id 
    FROM orders
);

-- Pattern A: Correlated NOT EXISTS Subquery (Safest and Preferred)
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
);

-- Pattern B: LEFT JOIN ... IS NULL
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

**Code Breakdown:**

EXISTS (Semi-Join): The query engine processes each row in customers. As soon as it encounters a single matching customer_id in orders, the internal subquery short-circuits (TRUE), emits the customer row, and immediately moves to the next customer.

NOT EXISTS (Anti-Join): Reverses the boolean evaluation. If a match is found in orders, the subquery returns TRUE, making NOT EXISTS false; thus, the customer row is excluded.  
LEFT JOIN ... WHERE o.customer_id IS NULL (Anti-Join): Performs a standard outer join, populating unmatched right-side attributes with NULL. The WHERE clause then retains only the unmatched NULL rows.

---

**Multiple/Composite Keys**   

Joining on composite keys means linking two or more tables using multiple column conditions simultaneously inside the ON clause (ON tableA.col1 = tableB.col1 AND tableA.col2 = tableB.col2).

**Application**  
Used whenever an entity's primary key or unique identifier consists of two or more combined attributes:  
- Multi-Tenant Schemas: Joining shared enterprise datasets that are segregated by tenant_id alongside an entity ID (tenant_id, user_id).
- Time-Series / Partitioned Data: Linking historical event partitions defined by combinations like (year, month, region_id).
- Composite Bridge Tables: Joining junction tables in many-to-many relationships (e.g., matching on both order_id and product_id).

**Real-World Problem and Solution**    
A global multi-tenant SaaS platform stores client data in a shared orders table and inventory table. Order IDs are not globally unique; they are unique only within each specific client's account (tenant_id). Joining solely on order_id causes cross-tenant data leaks and incorrect metrics by matching tenant A's Order #1001 with tenant B's Order #1001.

Solution: Joining across both the tenant_id and order_id composite key guarantees that data matches exclusively within the boundary of each client's isolated tenant space.

**Industrial Example**   
A logistics network tracks regional package routing across international hubs. Key lookup relies on a composite key composed of origin_hub_id, destination_hub_id, and carrier_code. Merging manifest schedules with package routing logs requires evaluating equality across all three composite keys simultaneously.

```sql
SELECT 
    o.tenant_id,
    o.order_id,
    o.customer_id,
    i.warehouse_location,
    i.stock_status
FROM orders o
INNER JOIN inventory i 
    ON  o.tenant_id  = i.tenant_id 
    AND o.product_id = i.product_id;
```

**Code Breakdown:**   
- o.tenant_id = i.tenant_id: Ensures the query only matches records belonging to the same enterprise client account.
- AND o.product_id = i.product_id: Ensures the lookup isolates the specific item within that client's inventory scope.
- Both predicates in the ON clause must evaluate to TRUE for a row pair to be combined and returned in the output set.

---

**Non-Equi Joins (Joins with >, <, BETWEEN instead of =)**  

A non-equi join is any join operation where the linkage condition in the ON clause uses inequality operators (>, <, >=, <=, BETWEEN, or !=) instead of standard value equality (=).

**Application**  
- Range & Banding Analysis: Categorizing continuous metrics into discrete buckets (e.g., mapping salary figures to tax brackets or credit scores to risk tiers).
- Time-Series Overlaps: Matching transaction timestamps against variable validity windows or promotional campaign dates.
- Session Attribution: Linking user clickstream events to active user sessions based on timestamp boundaries.
- Rolling Analytics without Window Functions: Pairing sequential records within specific relative offsets.

**Real-World Problem and Solution**   
Problem: A financial team needs to apply tax percentages to employee base salaries. The tax rates are defined in a reference table with salary ranges (min_salary, max_salary). Because base salaries contain exact continuous values (e.g., $84,250.00), performing an equality join (=) fails because exact salary values do not exist in the tax reference table.

Solution: Applying a non-equi join with BETWEEN evaluates whether an employee's salary falls within the lower and upper bounds of a tax bracket, mapping each employee to their correct rate without requiring hardcoded logic.

**Industrial Example**   
An e-commerce platform tracks dynamic hotel pricing. When a user books a stay, the system matches the booking date range (check_in_date to check_out_date) against a seasonal pricing table using non-equi date ranges (booking_date >= season_start_date AND booking_date <= season_end_date) to calculate nightly rates.

```sql
-- Pattern 1: Range Banding (Tax Brackets)
SELECT 
    e.employee_name,
    e.salary,
    t.bracket_name,
    t.tax_rate
FROM employees e
INNER JOIN tax_brackets t
    ON e.salary >= t.min_salary 
   AND e.salary <= t.max_salary;

-- Pattern 2: Time Window Overlap (Event Attribution)
SELECT 
    c.click_id,
    c.user_id,
    c.click_time,
    s.session_id
FROM user_clicks c
INNER JOIN user_sessions s
    ON c.user_id = s.user_id
   AND c.click_time BETWEEN s.session_start AND s.session_end;
```

**Code Breakdown:**
- Pattern 1: The ON clause checks if salary falls within the boundary defined by min_salary and max_salary. Every employee record pairs with the single tax bracket row that satisfies both inequality conditions.

- Pattern 2: Merges composite key logic (c.user_id = s.user_id) with a non-equi condition (c.click_time BETWEEN s.session_start AND s.session_end). Click records match only with the session active during that click event.

---

**LATERAL JOIN / CROSS APPLY (For "Top N Per Group" Problems)**

A LATERAL join (PostgreSQL, Snowflake, Redshift, MySQL 8.0.14+) or CROSS APPLY (SQL Server, Oracle) allows an inline subquery in the FROM or JOIN clause to reference columns from preceding tables defined earlier in the query. It acts like a SQL for-each loop, executing the right-side subquery once for every row produced by the left-side table.

**Application**  
- Top $N$ Per Group Queries: Fetching the most recent $N$ orders, top 3 highest transactions, or latest logs for every user in a table.
- Correlated Table Functions: Passing a column value from a parent table into a table-valued function or array unnesting function (e.g., parsing JSON arrays or string splits per row).
- Complex Per-Row Calculations: Computing expensive multi-step metrics per row where intermediate results need to be reused across columns.
 
**Real-World Problem and Solution**  
Problem: An e-commerce dashboard needs to display every customer along with their top 2 highest value orders.
Using window functions like ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC) requires scanning and ranking the entire orders table first before filtering on rank <= 2. If the orders table has 100 million rows, this full table scan and partition sort becomes extremely slow.

Solution: Using a LATERAL join evaluates an inner query that orders and fetches LIMIT 2 records for each specific customer individually. Combined with an index on (customer_id, amount DESC), the query performs direct index lookups per customer, avoiding a full table scan over millions of unneeded order rows.

**Industrial Example**  
A financial audit system processes corporate accounts. For each enterprise customer, the system retrieves the 3 most recent suspicious log events. Using LATERAL, the engine performs an index scan on the logs table for each account ID, returning the 3 latest security flags per customer in milliseconds.

```sql
-- PostgreSQL / Snowflake / MySQL 8.0.14+ Syntax (LATERAL)
SELECT 
    c.customer_id,
    c.customer_name,
    top_orders.order_id,
    top_orders.amount,
    top_orders.order_date
FROM customers c
LEFT JOIN LATERAL (
    SELECT o.order_id, o.amount, o.order_date
    FROM orders o
    WHERE o.customer_id = c.customer_id  -- Reference to left table row!
    ORDER BY o.amount DESC
    LIMIT 2
) top_orders ON TRUE;

-- SQL Server Syntax Equivalent (CROSS APPLY / OUTER APPLY)
SELECT 
    c.customer_id,
    c.customer_name,
    top_orders.order_id,
    top_orders.amount,
    top_orders.order_date
FROM customers c
OUTER APPLY (
    SELECT TOP 2 o.order_id, o.amount, o.order_date
    FROM orders o
    WHERE o.customer_id = c.customer_id
    ORDER BY o.amount DESC
) top_orders;
```

**Code Breakdown:**
- LEFT JOIN LATERAL (...) top_orders ON TRUE: Defines a lateral subquery. The ON TRUE clause is used because the matching logic is already handled inside the subquery's WHERE clause.
- WHERE o.customer_id = c.customer_id: Connects the inner subquery to the current customer row of the outer table (c). Standard subqueries in joins cannot reference table c here, but LATERAL explicitly enables this correlation.
- ORDER BY o.amount DESC LIMIT 2: Fetches only the top 2 records for the current customer before moving to the next.

**LEFT JOIN LATERAL vs. CROSS JOIN LATERAL:**  
CROSS JOIN LATERAL: If a customer has 0 orders, that customer is excluded from the output.  
LEFT JOIN LATERAL ... ON TRUE: If a customer has 0 orders, the customer row is retained with NULL order columns.

------------

**DDL & Schema Management**

**CREATE TABLE and Core Constraints (NOT NULL, FOREIGN KEY)**  

CREATE TABLE is the foundational Data Definition Language (DDL) command used to define the structure of a relational database table, including column names, data types, and integrity constraints.

- NOT NULL: Ensures that a column cannot accept NULL (missing or unknown) values.
- FOREIGN KEY: Enforces referential integrity by linking a column (or columns) in a child table to the primary key column in a parent table, ensuring child rows point to valid parent records.

**Application**  

- Data Completeness: Enforcing mandatory fields (e.g., user emails or transaction amounts) using NOT NULL.
- Relational Integrity: Linking transactional records (like orders) to master records (like users or products) via FOREIGN KEY constraints to prevent orphan data creation.

**Real-World Problem and Solution** 

Problem: An application allows users to create orders. Without schema-level rules, a bug in the application code might insert an order without an order date, or insert an order for a customer_id (e.g., 9999) that does not exist in the customers table. This creates broken analytics and orphan data.

Solution: Creating the orders table with a NOT NULL constraint on order_date and a FOREIGN KEY constraint on customer_id forces the database engine to reject incomplete or invalid records at insertion time.

**Industrial Example**

An enterprise inventory management platform uses foreign keys to link every stock adjustment log to an active employee account and a valid warehouse location. If an API request attempts to post an adjustment against a non-existent warehouse ID, the database immediately blocks the operation with a foreign key violation error.

```sql
-- Parent Table
CREATE TABLE customers (
    customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

-- Child Table with NOT NULL and FOREIGN KEY Constraints
CREATE TABLE orders (
    order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    -- NOT NULL ensures order_date must be provided
    order_date DATE NOT NULL,

    -- Foreign key linking to parent customer table
    customer_id INT NOT NULL,

    -- Named foreign key constraint definition
    CONSTRAINT fk_orders_customers 
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
);
```

**Code Breakdown:**

- customer_name VARCHAR(100) NOT NULL: Declares that every customer must have a name; attempts to insert NULL will throw an error.
- customer_id INT NOT NULL: Declares the column in the child table that will reference the parent table.
- CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id): Enforces that any value inserted into orders.customer_id must already exist in customers.customer_id.

**Important Points**

- Order of Deletion: You cannot drop or truncate a parent table (customers) while a child table (orders) holds an active foreign key referencing it, unless you drop the constraint first or use CASCADE.
- Foreign Key Indexing: Databases do not automatically create indexes on foreign key columns in child tables (unlike primary keys). You should manually create indexes on foreign keys (CREATE INDEX idx_orders_customer_id ON orders(customer_id);) to ensure efficient JOIN performance and fast cascading checks.
- Constraint Naming: Always explicitly name foreign key constraints (e.g., CONSTRAINT fk_tablename_parenttable) so that diagnostic logs provide clean error context when violations occur.

---

**ALTER TABLE (ADD COLUMN, MODIFY/ALTER COLUMN, DROP COLUMN, RENAME COLUMN/TABLE)**

ALTER TABLE is the DDL command used to modify the structure of an existing database table without destroying the table or its underlying data. It allows you to add new columns, modify existing column definitions or data types, remove unneeded columns, and rename columns or tables.

**Application**

- Schema Evolution: Adapting production database schemas to new business requirements (e.g., adding a loyalty_points column to an existing customers table).
- Data Type Migration: Increasing character limits or changing numeric precision to prevent data overflow (e.g., changing VARCHAR(50) to VARCHAR(255)).
- Schema Refactoring: Renaming columns or tables during code cleanup to match modern naming conventions.
- Storage Optimization: Removing deprecated or redundant columns to free up disk space and clean up table layouts.

**Industrial Example**

A financial processing engine updates its compliance system. The engineering team must increase the precision of balance tracking from two decimal places to four (DECIMAL(12, 4)) across billions of archived transaction rows. Using ALTER TABLE transactions ALTER COLUMN amount TYPE DECIMAL(12, 4);, the system modifies the data type in place without taking down accounting services.

```sql
-- 1. RENAME TABLE: Changes the table name
ALTER TABLE user_accounts RENAME TO customers;

-- 2. ADD COLUMN: Adds a new column with a default value
ALTER TABLE customers 
ADD COLUMN phone_number VARCHAR(20) DEFAULT NULL;

-- 3. MODIFY/ALTER COLUMN: Changes data type (Dialect-specific syntax)
-- PostgreSQL / Snowflake:
ALTER TABLE customers 
ALTER COLUMN phone_number TYPE VARCHAR(30);

-- MySQL / MariaDB equivalent:
-- ALTER TABLE customers MODIFY COLUMN phone_number VARCHAR(30);

-- 4. RENAME COLUMN: Renames a specific column
ALTER TABLE customers 
RENAME COLUMN phone_number TO contact_number;

-- 5. DROP COLUMN: Removes a column and its data permanently
ALTER TABLE customers 
DROP COLUMN contact_number;
```

**Code Breakdown**

- RENAME TO customers: Updates table metadata in the system catalog to direct future queries to the new table name.
- ADD COLUMN: Appends a new column attribute to the end of the table definition. Adding DEFAULT NULL ensures existing rows populate safely without raising constraint errors.
- ALTER COLUMN ... TYPE / MODIFY COLUMN: Alters the storage classification or scale of a column.
- RENAME COLUMN: Updates the column identifier while maintaining all stored data, indexes, and constraints attached to it.
- DROP COLUMN: Drops the specified column and deallocates its associated storage.

**Important Points Related to It**

- Table Lock Overhead: On large tables (millions of rows), executing ALTER TABLE commands—especially dropping or altering column data types—can require exclusive table locks (ACCESS EXCLUSIVE LOCK). This can block read/write queries and cause temporary application latency.

- Dialect Differences for Column Modifications:
    - PostgreSQL / Snowflake: ALTER TABLE table_name ALTER COLUMN column_name TYPE new_type;
    - MySQL / Oracle: ALTER TABLE table_name MODIFY COLUMN column_name new_type;
    - SQL Server: ALTER TABLE table_name ALTER COLUMN column_name new_type;

---

**Cascading Constraints (ON DELETE CASCADE / ON UPDATE CASCADE)**

Cascading constraints define automated rules that the database engine enforces when a primary or referenced key record in a parent table is deleted or updated. Instead of raising a foreign key constraint violation error, the system automatically propagates (cascades) the corresponding delete or update action to all child records that reference that parent key.

**Application**

- Used to maintain referential integrity across relational schemas without needing manual multi-query cleanup or custom database triggers:
- Child Record Cleanup: Automatically removing dependent line items when an order is deleted.
- Session and Profile Teardown: Purging user session tokens or preference entries when a user account is deleted.
- Key Migration: Propagating key changes throughout child tables when a natural key is modified in a parent lookup table.

**Industrial Example**

A multi-tenant SaaS application manages projects and associated project tasks. When an enterprise client cancels their account and requests immediate tenant data deletion, executing a single DELETE FROM tenants WHERE tenant_id = 'TNT-8921' automatically cascades through projects, tasks, comments, and file_attachments, cleanly removing all associated operational data without leaving orphan rows across millions of database records.

```sql
-- Parent Table
CREATE TABLE customers (
    customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

-- Child Table with Cascading Actions
CREATE TABLE orders (
    order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE DEFAULT CURRENT_DATE,

    -- Foreign key declaration with ON DELETE and ON UPDATE actions
    CONSTRAINT fk_customer 
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Demonstration:
INSERT INTO customers (customer_name) VALUES ('Alice');
INSERT INTO orders (customer_id) VALUES (1), (1);

-- Executing parent deletion automatically removes rows in the orders table:
DELETE FROM customers WHERE customer_id = 1;
```

**Code Breakdown:**

- REFERENCES customers(customer_id) establishes the foreign key relationship between child (orders) and parent (customers).
- ON DELETE CASCADE instructs the database engine: "When a row in customers is deleted, automatically delete every matching row in orders where customer_id matches."
- ON UPDATE CASCADE ensures that if the customer_id primary key value is updated in the parent table, all matching customer_id foreign key entries in orders update to match automatically.

**Important Points**

- Alternative Cascade Actions:
    - ON DELETE SET NULL: Sets the foreign key column in child rows to NULL when the parent row is deleted (requires the child foreign key column to allow NULL).
    - ON DELETE SET DEFAULT: Sets the child foreign key column to its assigned DEFAULT value upon parent row deletion.
    - ON DELETE RESTRICT / NO ACTION (Default): Rejects the parent deletion and throws a constraint violation error if child records exist.

- Accidental Mass Deletion Risk: ON DELETE CASCADE can be dangerous in production setups. A single unintended DELETE FROM parent_table; statement can silently wipe out millions of rows across multiple nested child tables without warning.
- Performance Considerations: Deleting a parent row that triggers cascades across large child tables can lock multiple tables and cause temporary performance bottlenecks. Indexes should always be created on foreign key columns (CREATE INDEX idx_orders_customer_id ON orders(customer_id);) to ensure the engine can quickly locate dependent child rows during a cascade.

---

**(VARCHAR vs TEXT, DECIMAL vs FLOAT, DATE vs TIMESTAMP vs TIMESTAMPTZ)**

Data types define the storage size, structural domain, precision rules, and operation constraints of column values within a database schema:

- VARCHAR(N) vs TEXT: Character string types that store variable-length text data.
- DECIMAL(P, S) vs FLOAT: Numeric types distinguishing exact fixed-point precision from approximate floating-point values.
- DATE vs TIMESTAMP vs TIMESTAMPTZ: Temporal types measuring calendar dates, local wall-clock times, and time zone-aware points in time.

**Application**

- Character Data: Use VARCHAR(N) when enforcing mandatory maximum character length boundaries (e.g., state codes or phone numbers); use TEXT for arbitrary length content (e.g., blog posts or JSON payloads).
- Numeric Data: Use DECIMAL for financial metrics, billing totals, or currency values; use FLOAT for scientific computations, sensor telemetry, or statistical approximations.
- Temporal Data: Use TIMESTAMPTZ for transaction logs and global user events; use DATE for birthday entries or billing start dates; use TIMESTAMP for static local display times.

**Industrial Example**

An e-commerce marketplace stores order payments using DECIMAL(10, 2) to guarantee exact accounting balances. Product descriptions use TEXT to accommodate unstructured content. Payment audit timestamps are logged as TIMESTAMPTZ, allowing international sellers to view order histories rendered accurately in their local time zones.

```sql
CREATE TABLE transaction_audit (
    audit_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    -- 1. Character Types
    country_code VARCHAR(3) NOT NULL,       -- Max length enforced (e.g., 'USA')
    internal_notes TEXT,                    -- Unlimited length text

    -- 2. Numeric Types
    exact_amount DECIMAL(12, 2) NOT NULL,   -- Precision = 12 total digits, Scale = 2 decimal places
    approx_ratio FLOAT,                      -- Floating-point scientific value

    -- 3. Temporal Types
    scheduled_date DATE,                    -- YYYY-MM-DD
    local_event_time TIMESTAMP,             -- YYYY-MM-DD HH:MM:SS (No timezone awareness)
    global_event_time TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP -- Stored internally as UTC
);
```

Code Breakdown:
- VARCHAR(3) restricts inputs to a maximum length of 3 characters.
- TEXT accepts arbitrary text without needing length constraints.
- DECIMAL(12, 2) allocates 12 total digits of precision, reserving 2 digits to the right of the decimal point (storing up to $99,999,999,999.99).
- FLOAT stores floating-point numeric values using IEEE 754 binary arithmetic.
- TIMESTAMPTZ automatically converts input times to UTC for storage and converts them back to the client session's local time zone upon retrieval.

**Important Points**

- VARCHAR(N) vs TEXT Performance: In PostgreSQL, VARCHAR(N) and TEXT use the identical underlying storage engine (varlena) and perform at identical speeds. The only difference is that VARCHAR(N) enforces maximum character length validation on insert.
- DECIMAL vs FLOAT Rule: Never store currency or money values using FLOAT or DOUBLE PRECISION. Always use DECIMAL / NUMERIC.

- Dialect Variations:
    - PostgreSQL: Prefers TEXT, DECIMAL, and TIMESTAMPTZ (short for TIMESTAMP WITH TIME ZONE).
    - MySQL: Uses VARCHAR(N), DECIMAL, and handles time zones using TIMESTAMP (auto-converted to UTC) versus DATETIME (fixed string representation).
    - SQL Server: Uses VARCHAR(N), DECIMAL, and DATETIMEOFFSET for time zone awareness.

---

**Surrogate / Auto-Increment Keys (SERIAL / IDENTITY / AUTO_INCREMENT)**

A surrogate key is an artificially generated numeric identifier used as a table's primary key. It has no business meaning (unlike a natural key like a Social Security Number or ISBN). Auto-increment mechanisms automatically assign sequential integer values to surrogate key columns whenever a new row is inserted.

**Application**

- Unique Identification: Providing a simple, immutable, non-business unique identifier for every row in a table.
- Join Optimization: Accelerating JOIN operations across relational tables using compact integer keys instead of wide string/text columns.
- Simplifying Application Logic: Allowing applications to insert rows without needing to calculate or pass the next available ID manually.

**Real-World Problem and Solution**

Problem: An application uses a natural key like email as the primary key in a users table. If a user changes their email address, the system must perform costly cascading updates across dozens of foreign-key child tables (orders, payments, logs). Furthermore, joining on wide string columns degrades join performance and increases index size.

Solution: Using an auto-increment integer surrogate key (user_id) as the primary key isolates internal database relationships from external business data. Changing an email address becomes a simple single-row update on the users table, leaving all foreign key references intact.

**Industrial Example**

An e-commerce system generates millions of order records daily. The engine automatically assigns sequential primary key IDs (order_id) during high-concurrency peak sales events without requiring write transactions to block each other to find the next available ID.

```sql
-- 1. Standard ANSI SQL / PostgreSQL 10+ / Snowflake / Oracle 12c (GENERATED AS IDENTITY)
CREATE TABLE customers_ansi (
    customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

-- 2. Legacy PostgreSQL Syntax (SERIAL)
CREATE TABLE customers_postgres (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

-- 3. MySQL / MariaDB Syntax (AUTO_INCREMENT)
CREATE TABLE customers_mysql (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);

-- 4. SQL Server Syntax (IDENTITY)
CREATE TABLE customers_sqlserver (
    customer_id INT IDENTITY(1,1) PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);
```

**Code Breakdown**

- GENERATED ALWAYS AS IDENTITY: The standard ANSI SQL approach. Prevents manual overrides by throwing an error if an application explicitly tries to pass a value into customer_id during an INSERT.
- GENERATED BY DEFAULT AS IDENTITY: Allows the database to assign sequential values automatically, while still allowing manual explicit ID overrides when needed.
- SERIAL: PostgreSQL shorthand that implicitly creates an underlying sequence object and sets the column default to nextval('sequence_name').
- AUTO_INCREMENT: MySQL implementation that assigns the next sequential integer starting from 1.
- IDENTITY(1,1): SQL Server implementation specifying (seed, increment) where the value starts at 1 and increases by 1 for each row.

**Important Points**

- Gaps in Sequences: Auto-increment values are not guaranteed to be gapless. If a transaction is rolled back or an insert fails due to a constraint check, the sequence number generated during that attempt is lost forever.
- GENERATED ALWAYS vs GENERATED BY DEFAULT:
- ALWAYS: Standard safety rule. Prevents application code from accidentally inserting explicit primary key values.
- BY DEFAULT: Useful for data migration scripts where historical IDs must be preserved.

---

**INSERT: Single-Row, Multi-Row, and INSERT...SELECT**

The INSERT statement is the core DML command used to add new rows of data into an existing database table.

- Single-Row INSERT: Appends a single record into the target table.
- Multi-Row INSERT: Appends multiple explicit rows within a single SQL statement execution.
- INSERT...SELECT: Queries data from one or more source tables and inserts the resulting dataset directly into a target table.

**Application**

- Single-Row: Inserting a single event, user registration, or standalone entity record submitted via an API or user interface.
- Multi-Row: Batch inserting structured records (e.g., seeding lookup tables or processing bulk form submissions) to minimize network round-trips.
- INSERT...SELECT: Populating aggregation/reporting tables, migrating data between tables, backing up specific records, or executing ETL (Extract, Transform, Load) transformations.

**Industrial Example**

A logistics application archives completed delivery records. At midnight, an automated batch process runs an INSERT...SELECT query that reads all deliveries marked 'COMPLETED' from the live active_deliveries table and populates the archived_deliveries historical table in a single atomic transaction.

```sql
-- Target Table Setup
CREATE TABLE product_catalog (
    product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE
);

-- 1. Single-Row INSERT
INSERT INTO product_catalog (product_name, category, price)
VALUES ('Wireless Mouse', 'Electronics', 29.99);

-- 2. Multi-Row INSERT (Batching)
INSERT INTO product_catalog (product_name, category, price)
VALUES 
    ('Mechanical Keyboard', 'Electronics', 89.99),
    ('Ergonomic Chair', 'Office', 249.50),
    ('Desk Mat', 'Office', 15.00);

-- 3. INSERT...SELECT (Populating from another table query)
CREATE TABLE high_value_products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10, 2)
);

INSERT INTO high_value_products (product_id, product_name, price)
SELECT product_id, product_name, price
FROM product_catalog
WHERE price > 50.00 AND is_active = TRUE;
```

**Code Breakdown:**

- Explicit Column List (product_name, category, price): Specifying column target names ensures that omitted columns with DEFAULT or auto-increment rules (like product_id and is_active) populate automatically and protects queries from breaking if table column order changes.

- Multi-Row VALUES (...), (...): Groups multiple row tuples in a single statement, reducing network latency and transaction overhead compared to executing multiple individual INSERT commands.

- INSERT INTO ... SELECT ...: Eliminates the need to retrieve data back to the application layer before writing it. The database engine streams rows directly from the source query into the target table on disk.

**Important Points**

- Explicit Column Mapping: Always specify target column names INSERT INTO target (col1, col2) rather than relying on implicit positional inserts (INSERT INTO target VALUES (...)). Omitting explicit column names makes code fragile to column reordering or future table schema alterations.

- Multi-Row Insert Limits: Databases impose parameters and memory boundaries on single statements (e.g., MySQL's max_allowed_packet or SQL Server's limit of 1,000 row values per single VALUES clause). For bulk loading hundreds of thousands of rows, split batch inserts into chunks (e.g., 1,000–5,000 rows per batch) or use bulk load tools (COPY in PostgreSQL, LOAD DATA INFILE in MySQL).

- Data Type Matching: In INSERT...SELECT, the output columns of the SELECT query must match the target table's columns in position, count, and compatible data types. Column names in the subquery do not need to match target names, but position and compatibility are strictly enforced.

---

**UPDATE: Basic, JOIN-Based, and UPDATE...FROM Syntax**

The UPDATE statement is the DML command used to modify existing column values in table records based on matching conditions.

- Basic UPDATE: Modifies column values using static values or expressions computed directly from the row being updated.
- JOIN-Based UPDATE / UPDATE...FROM: Updates rows in a target table using data dynamically retrieved from secondary source tables linked via join conditions.

**Application**

- State Changes: Changing an order's status from 'processing' to 'shipped'.
- Metric Adjustments: Applying bulk price increases or incrementing user reward balances.
- Cross-Table Synchronization: Syncing customer tier classifications based on aggregated lifetime spending stored in a separate metrics table.

**Industrial Example**

A subscription platform recalculates active user pricing. At billing cycle reset, an UPDATE...FROM query joins the subscriptions table with the usage_logs table, applying dynamic tier adjustments based on each customer's total data usage over the past month.

```sql
-- Target Table: products | Source Table: vendors

-- 1. Basic UPDATE
UPDATE products
SET price = price * 0.95,
    updated_at = CURRENT_TIMESTAMP
WHERE category = 'Electronics';

-- 2. PostgreSQL / Snowflake / Redshift Syntax (UPDATE ... FROM)
UPDATE products p
SET price = p.price * 0.90,
    is_discounted = TRUE
FROM vendors v
WHERE p.vendor_id = v.vendor_id
  AND v.vendor_tier = 'Preferred';

-- 3. MySQL Syntax Equivalent (UPDATE with explicit JOIN)
UPDATE products p
INNER JOIN vendors v ON p.vendor_id = v.vendor_id
SET p.price = p.price * 0.90,
    p.is_discounted = TRUE
WHERE v.vendor_tier = 'Preferred';

-- 4. SQL Server Syntax Equivalent (UPDATE ... FROM ... JOIN)
UPDATE p
SET p.price = p.price * 0.90,
    p.is_discounted = TRUE
FROM products p
INNER JOIN vendors v ON p.vendor_id = v.vendor_id
WHERE v.vendor_tier = 'Preferred';
```

**Code Breakdown:**

- Basic UPDATE: The SET clause specifies columns to modify, while the WHERE clause isolates matching rows. Omitting WHERE updates every row in the table.

-  UPDATE ... FROM (PostgreSQL Dialect):
    - UPDATE products p: Specifies the target table being modified. (Do not join the target table again in the FROM clause).
    - FROM vendors v: Declares secondary tables needed to source values or criteria.
    - WHERE p.vendor_id = v.vendor_id: Defines the join link between target and source tables.

**Important Points**

- The Missing WHERE Clause Disaster: Running an UPDATE without a WHERE clause modifies every single record in the table. In production environments, always execute a SELECT query with identical WHERE conditions first to verify the target row count before running the UPDATE.

- Target Alias Rules in PostgreSQL: In PostgreSQL UPDATE...FROM, never write UPDATE products p ... FROM products p JOIN vendors v. Repeating the target table in the FROM clause creates a self-join, leading to unexpected behavior.

- Non-Deterministic Multi-Match Updates: If a single row in the target table matches multiple rows in the source table during a join-based update, SQL standard rules state that the resulting value is non-deterministic (the engine will pick one matching source row arbitrarily). Ensure join criteria maintain a 1:1 or N:1 relationship relative to the target table.

---

**DELETE: Basic and JOIN/Subquery-Based**

The DELETE statement is the DML command used to remove existing rows from a table based on logical criteria.
- Basic DELETE: Evaluates simple conditions on target table attributes to purge specific rows.

- JOIN / Subquery-Based DELETE: Uses data from related secondary tables (via subqueries or dialect-specific join constructs) to filter which target rows should be deleted.

**Application**

- Targeted Cleanup: Removing canceled orders, expired temporary guest sessions, or obsolete records.
- Data Pruning: Removing records based on external parent status (e.g., deleting orders associated with suspended vendor accounts).
- Duplicate Elimination: Purging duplicate records from staging tables prior to loading data into production.

**Industrial Example**

A fraud prevention engine isolates compromised user accounts. An automated nightly pipeline executes a subquery-driven DELETE that scans login attempt logs, identifies IP addresses flagged on a blacklists table, and purges unverified session tokens linked to those blacklisted IPs.

```sql
-- 1. Basic DELETE
DELETE FROM shopping_carts
WHERE updated_at < CURRENT_TIMESTAMP - INTERVAL '30 days';

-- 2. ANSI-Compliant Subquery DELETE (Portable across PostgreSQL, MySQL, SQL Server, Oracle)
DELETE FROM shopping_carts
WHERE user_id IN (
    SELECT user_id 
    FROM users 
    WHERE account_status = 'DEACTIVATED'
);

-- 3. PostgreSQL Dialect Syntax (USING Clause)
DELETE FROM shopping_carts sc
USING users u
WHERE sc.user_id = u.user_id
  AND u.account_status = 'DEACTIVATED';

-- 4. MySQL Dialect Syntax (Explicit JOIN)
DELETE sc
FROM shopping_carts sc
INNER JOIN users u ON sc.user_id = u.user_id
WHERE u.account_status = 'DEACTIVATED';
```

**Code Breakdown**

- Basic DELETE FROM ... WHERE: Scans the table and deletes rows satisfying the predicate. Omitting the WHERE clause permanently deletes all rows in the table.

- WHERE user_id IN (...) (Subquery): Evaluates an independent list of IDs from the users subquery and removes matching cart entries.

- USING users u (PostgreSQL): Introduces secondary lookup tables into the DELETE pipeline. The join logic connects target table sc to secondary table u inside the WHERE clause.

- DELETE sc FROM ... (MySQL): The table alias immediately following DELETE specifies which table's rows will be removed, while the FROM ... INNER JOIN constructs the matching filter logic.

**Important Points** 

- The Missing WHERE Clause Risk: A DELETE FROM table_name; statement without a WHERE clause removes every record in the table sequentially. Unlike TRUNCATE, row-by-row DELETE logs individual row deletions in the transaction log, making it slow on massive tables.

- Checking Rows Before Deletion: Always verify target records first by replacing DELETE with SELECT * using identical criteria:

```sql
-- Step 1: Verify output rows
SELECT * FROM shopping_carts WHERE updated_at < CURRENT_TIMESTAMP - INTERVAL '30 days';

-- Step 2: Execute actual deletion
DELETE FROM shopping_carts WHERE updated_at < CURRENT_TIMESTAMP - INTERVAL '30 days';
```

- Performance Considerations: Deleting large volumes of rows (e.g., millions of records) in a single transaction can saturate the transaction log, lock tables, and impair database execution speeds. For mass purges, process deletions in smaller iterative batches (e.g., deleting 5,000 rows per transaction loop).

---

**UPSERT Patterns (ON CONFLICT in PostgreSQL vs MERGE in ANSI/SQL Server/Snowflake)**

An UPSERT (a blend of UPdate and inSERT) is a conditional DML pattern that attempts to insert a record into a table. If the row violates an existing PRIMARY KEY or UNIQUE constraint (meaning the record already exists), the database engine intercepts the conflict and performs an UPDATE on the existing row instead (or skips insertion entirely).

**Application**

- Idempotent Data Ingestion: Safely ingesting API feeds or streaming logs without creating duplicate records on pipeline retries.
- Inventory & Telemetry Counters: Incrementing views, click counts, or stock levels where a key might already exist.
- Profile & Settings Sync: Updating existing user preferences if an account exists, or creating a new record if it does not.

**Industrial Example**

A global identity provider syncs user metadata from single sign-on (SSO) providers. When a user logs in via Google/Okta, the system executes an UPSERT on users matching on email. If the email is recognized, it updates last_login_at and display_name; if it is a new user, it creates the account profile.

```sql
-- Pattern 1: PostgreSQL / SQLite Syntax (ON CONFLICT)

-- Pattern 2: ANSI Standard / SQL Server / Snowflake / Oracle Syntax (MERGE)

-- Target table with unique constraint on product_id
CREATE TABLE daily_product_views (
    product_id INT NOT NULL,
    view_date DATE NOT NULL,
    view_count INT DEFAULT 1,
    PRIMARY KEY (product_id, view_date)
);

-- Executing UPSERT using ON CONFLICT
INSERT INTO daily_product_views (product_id, view_date, view_count)
VALUES (101, CURRENT_DATE, 1)
ON CONFLICT (product_id, view_date) 
DO UPDATE SET 
    view_count = daily_product_views.view_count + EXCLUDED.view_count;

-- Alternative: Ignore duplicate inserts without throwing an error
INSERT INTO daily_product_views (product_id, view_date, view_count)
VALUES (101, CURRENT_DATE, 1)
ON CONFLICT (product_id, view_date) 
DO NOTHING;

MERGE INTO daily_product_views AS target
USING (VALUES (101, CURRENT_DATE, 1)) AS source (product_id, view_date, view_count)
ON target.product_id = source.product_id 
AND target.view_date = source.view_date

-- When match exists -> UPDATE
WHEN MATCHED THEN
    UPDATE SET target.view_count = target.view_count + source.view_count

-- When no match exists -> INSERT
WHEN NOT MATCHED THEN
    INSERT (product_id, view_date, view_count)
    VALUES (source.product_id, source.view_date, source.view_count);
```

**Code Breakdown**
- ON CONFLICT (product_id, view_date): Specifies the unique target constraint/index to watch for collisions.

- EXCLUDED Keyword (PostgreSQL): A virtual table representing the row values that were attempted to be inserted. EXCLUDED.view_count references the incoming candidate value.

- DO NOTHING: Silently ignores the insert attempt if a conflict occurs, avoiding thrown exceptions.

- MERGE INTO ... USING: Evaluates a join condition between a target table and a incoming source batch. The WHEN MATCHED and WHEN NOT MATCHED clauses route execution cleanly based on join resolution.

**Important Points**

- Constraint Prerequisite for ON CONFLICT: PostgreSQL requires an explicit UNIQUE index or PRIMARY KEY constraint on the columns listed in ON CONFLICT (...). You cannot use ON CONFLICT on arbitrary non-unique columns.

- Concurrency Safety: Both ON CONFLICT and MERGE acquire internal database locks during execution, preventing duplicate key errors under parallel thread workloads.

- Dialect Summary:

    - PostgreSQL / SQLite: INSERT ... ON CONFLICT (col) DO UPDATE / DO NOTHING

    - MySQL / MariaDB: INSERT ... ON DUPLICATE KEY UPDATE col = VALUES(col)

    - SQL Server / Snowflake / Oracle / BigQuery: MERGE INTO target USING source ON ...

---

**TRUNCATE vs DELETE vs DROP**

TRUNCATE, DELETE, and DROP are database commands used to remove data or database objects, operating at distinct architectural layers:

- DELETE (DML): Removes specific or all rows from a table row-by-row based on a WHERE clause, logging each deleted row individually.

- TRUNCATE (DDL): Removes all rows from a table instantly by deallocating the table's underlying storage data pages, bypassing row-by-row deletion logging.

- DROP (DDL): Completely destroys the entire table object definition along with all its data, constraints, indexes, and grants from the database catalog.

Note: TRUNCATE can be rolled back inside explicit transactions in PostgreSQL and SQL Server, but causes an implicit commit in MySQL.

**Application**

- DELETE: Use when selectively purging subset rows via WHERE conditions, or when operating inside fine-grained transaction controls requiring row-level triggers to fire.

- TRUNCATE: Use when resetting staging tables, clearing temporary batch tables, or clearing massive operational datasets quickly without dropping schema structures.

- DROP: Use during schema migrations, teardowns, or lifecycle removal of obsolete database objects.

**Industrial Example**

An e-commerce testing environment undergoes daily automated system resets. The deployment pipeline uses TRUNCATE to clear customer cart sessions and inventory caches instantly, while leaving table structures intact. When deprecating old legacy analytics tables, the pipeline uses DROP TABLE to permanently release disk storage back to the database cluster.

```sql
-- 1. DELETE: Selective row removal with logging
BEGIN;
DELETE FROM orders 
WHERE order_status = 'CANCELLED';
-- Row-level triggers fire; action is fully logged
COMMIT;

-- 2. TRUNCATE: Fast full-table data purge
BEGIN;
TRUNCATE TABLE order_staging;
-- Table structure remains; auto-increment keys reset; fast page deallocation
COMMIT;

-- 3. DROP: Complete schema destruction
DROP TABLE IF EXISTS obsolete_orders_2024;
-- Table metadata, constraints, indexes, and data are permanently removed
```

**Code Breakdown**

- DELETE FROM orders WHERE ...: Evaluates rows individually against the WHERE predicate. Retains non-matching rows.

- TRUNCATE TABLE order_staging: Empties all data pages from disk instantly. Table metadata, constraints, and column definitions remain intact for future INSERT queries.

- DROP TABLE IF EXISTS ...: Instructs the database catalog to unregister the table object entirely. Future queries referencing this table will return a table does not exist error.

**Summary Comparison Matrix**

- Command Category:

DELETE: DML (Data Manipulation Language).  
TRUNCATE: DDL (Data Definition Language).  
DROP: DDL (Data Definition Language).  

- Operation Scope:

DELETE: Removes specific rows based on criteria, or all rows if unfiltered.  
TRUNCATE: Removes all rows from the table indiscriminately.  
DROP: Destroys the table object entirely, along with all of its data, indexes, and constraints.  

- WHERE Clause Support:

DELETE: Fully supported.  
TRUNCATE: Not Allowed.  
DROP: Not Allowed.  

- Execution Speed:

DELETE: Slow (scans, checks, and logs each row individually).  
TRUNCATE: Ultra-Fast (deallocates storage data pages directly in system metadata).  
DROP: Instant (removes metadata entries from the system catalog).  

- Transaction Logging:

DELETE: Generates high log volume by logging every individual row deletion.  
TRUNCATE: Minimal logging (logs page deallocations rather than individual row contents).  
DROP: Minimal logging (logs catalog metadata modifications).  

- Auto-Increment / Sequence Reset:

DELETE: No (the identity/sequence counter continues from the last value).  
TRUNCATE: Yes (resets auto-increment and sequence counters back to their initial seed value).  
DROP: N/A (the underlying sequence or identity column is deleted with the table).  

- Triggers Fired:

DELETE: Fires associated ON DELETE row-level and statement-level triggers.  
TRUNCATE: Does NOT fire row-level DELETE triggers.  
DROP: Does NOT fire row-level DELETE triggers.  

- Table Structure Retention:

DELETE: Preserved intact for future inserts.  
TRUNCATE: Preserved intact for future inserts.  
DROP: Destroyed completely.  

- Rollback Ability:

DELETE: Fully rollbackable within explicit transactions across all relational databases.  
TRUNCATE: Rollbackable in PostgreSQL and SQL Server; causes an implicit transaction commit in MySQL.  
DROP: Rollbackable in PostgreSQL and SQL Server; causes an implicit transaction commit in MySQL.  

**Important Points**

- Foreign Key Blockers: You cannot run TRUNCATE TABLE parent_table; if parent_table is referenced by an active foreign key constraint in a child table, even if the child table is empty. You must either drop the foreign key, disable it, or use TRUNCATE parent_table, child_table CASCADE; (in PostgreSQL).

- Identity / Auto-Increment Resets:

    - DELETE: Deleting all rows via DELETE FROM users; does not reset the primary key sequence. The next insert will receive ID N+1.

    - TRUNCATE: Automatically resets primary key identities/sequences back to 1 (or default seed) in most SQL dialects.

- Storage Space Reclaim: Running DELETE on millions of rows does not immediately return free disk space to the OS; it marks internal table pages as reusable. TRUNCATE and DROP immediately deallocate storage structures, returning space to the storage pool.

---

**The RETURNING Clause (Standalone & DML Integration)**

The RETURNING clause is a DML extension (standard in PostgreSQL, SQLite, Oracle, and available as OUTPUT in SQL Server) that allows modification statements—INSERT, UPDATE, DELETE, and UPSERT—to return modified row data directly back to the calling application within the exact same execution turn.

**Application**

- Retrieving Generated Keys: Capturing auto-incremented primary keys (user_id, order_id) or server-side defaults (created_at, UUID) immediately upon row insertion without running a separate SELECT query.

- Audit and Archival Logs: Returning the original values of deleted or updated rows to feed directly into audit logging streams.

- Pipeline Chaining: Pairing RETURNING with Common Table Expressions (CTEs) to feed modified rows directly into downstream operations within a single multi-table transaction.

**Industrial Example**

An inventory engine processes order cancellations. When items are removed from an order via a DELETE FROM order_items query, appending RETURNING product_id, quantity allows the application to capture the exact restock quantities immediately and update inventory ledgers in real time without querying the database twice.

```sql
-- Target Schema
CREATE TABLE users (
    user_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    account_status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- 1. INSERT with RETURNING (Capturing generated ID & default values)
INSERT INTO users (email)
VALUES ('alice@example.com')
RETURNING user_id, created_at, account_status;

-- 2. UPDATE with RETURNING (Capturing updated state)
UPDATE users
SET account_status = 'active'
WHERE email = 'alice@example.com'
RETURNING user_id, email, account_status AS new_status;

-- 3. DELETE with RETURNING (Capturing deleted snapshot data)
DELETE FROM users
WHERE account_status = 'pending'
RETURNING user_id, email;

-- 4. Dialect Equivalent: SQL Server (OUTPUT Clause)
-- INSERT INTO users (email) 
-- OUTPUT inserted.user_id, inserted.created_at 
-- VALUES ('bob@example.com');
```

**Code Breakdown**

- RETURNING user_id, created_at: Instructs the execution engine to return a record set containing the auto-generated user_id and server-side computed created_at timestamp immediately after insertion completes.

- RETURNING ... AS new_status: Allows returned columns to be aliased, just like a standard SELECT query.

- DELETE ... RETURNING: Streams the purged row values back to the client buffer before removing them from table storage.

**Important Points**

- Performance & Network Efficiency: Eliminates round-trip network hops between the application layer and database layer by combining write operations and data retrieval into a single query turn.

- Use with Data Modifying CTEs: PostgreSQL allows INSERT, UPDATE, or DELETE statements with RETURNING to live inside Common Table Expressions (WITH blocks). This enables powerful patterns like moving rows from active tables to historical archive tables in a single atomic SQL statement:

```sql
WITH deleted_orders AS (
    DELETE FROM orders 
    WHERE status = 'cancelled'
    RETURNING *
)
INSERT INTO archived_orders 
SELECT * FROM deleted_orders;
```

- Dialect Differences:

    - PostgreSQL / SQLite / Oracle 18c+ / CockroachDB: Uses standard RETURNING col1, col2 (or RETURNING *).

    - SQL Server: Uses the OUTPUT clause (OUTPUT inserted.col1, OUTPUT deleted.col1).

    - MySQL: Does not support RETURNING or OUTPUT natively (requires using driver-level functions like LAST_INSERT_ID()).

---

**Primary Key Design (Single-Column vs Composite Keys)**

A primary key is a column or a set of columns that uniquely identifies each row in a database table. A single-column primary key relies on one unique column (typically a surrogate key like an integer ID or UUID), while a composite primary key combines two or more columns to form uniqueness across the row.

**Application**

- Single-Column Primary Key: Used for entity-centric tables where each record represents an independent object (e.g., users, products, orders).
- Composite Primary Key: Used in junction/bridge tables for many-to-many relationships (e.g., order_items, student_courses) or multi-tenant architectures where uniqueness is scoped per tenant (e.g., tenant_id + user_id).

**Real-World Problem and Solution**

Problem: An e-commerce system tracks line items in shopping carts. If the table uses a single auto-increment cart_item_id, a bug in the application code might allow duplicate rows for the same cart_id and product_id pair (e.g., inserting two separate rows for Product #50 inside Cart #12 instead of updating the quantity).

Solution: Defining a composite primary key on (cart_id, product_id) forces uniqueness at the database schema level. Any attempt to insert a duplicate product_id for the same cart_id fails instantly with a primary key constraint violation.

**Industrial Example**

An enterprise SaaS platform segregates customer data by tenant. The database uses a composite primary key of (tenant_id, user_id) across all user tables. This guarantees that user IDs are unique per corporate client while enforcing proper indexing for tenant-isolated database queries.

```sql
-- Pattern 1: Single-Column Primary Key
CREATE TABLE users (
    user_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- Pattern 2: Composite Primary Key (Junction Table)
CREATE TABLE order_items (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,

    -- Defining composite primary key across two columns
    PRIMARY KEY (order_id, product_id),

    -- Foreign key constraints referencing parent tables
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id),
    CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Code Breakdown:**

- user_id INT ... PRIMARY KEY: Establishes a single surrogate key column as the primary key
- PRIMARY KEY (order_id, product_id): Creates a composite key. The combination of order_id = 101 and product_id = 50 must be unique. The same order_id can appear multiple times, and the same product_id can appear multiple times, but the pair cannot repeat.

**Important Points Related to It**

- Implicit Constraints: Declaring a primary key automatically enforces NOT NULL and UNIQUE constraints on all involved columns.

- Clustered Index Creation: In most relational database engines (such as MySQL InnoDB or SQL Server), declaring a primary key automatically creates a clustered index on those columns, dictating the physical storage order of rows on disk.

- Foreign Key Overhead: Joining child tables to a composite key table requires foreign keys in child tables to store all composite key columns, increasing table storage size and join complexity.

- Column Order in Composite Keys: Place the most frequently queried or highest-cardinality column first in a composite primary key (e.g., PRIMARY KEY (tenant_id, user_id)) to optimize B-tree index lookup efficiency.

---------

**Window Functions**  
Unlike standard aggregation (GROUP BY) which collapses individual rows into a single summary row, *Window Functions* compute aggregate or analytical values across a set of rows while preserving the identity and detail of every single row in the final output

GROUP BY (Collapses Rows):  
[Row 1: Engineering, 100k] \   
[Row 2: Engineering, 120k]  -->  [Engineering, 110k Avg] (Original rows are lost)  
[Row 3: Engineering, 110k] /  

WINDOW FUNCTION (Preserves Rows):  
[Row 1: Engineering, 100k]  -->  [Row 1: Engineering, 100k, 110k Avg]  
[Row 2: Engineering, 120k]  -->  [Row 2: Engineering, 120k, 110k Avg]  
[Row 3: Engineering, 110k]  -->  [Row 3: Engineering, 110k, 110k Avg]

To control how a window function behaves, we use the OVER() clause. This clause accepts three optional sub-clauses: 
   
**The Core Components Explained:**  
- PARTITION BY: Defines the boundaries. It slices your dataset into isolated groups (e.g., by department or country). The window function calculates its values independently within each slice.

- ORDER BY: Determines the physical order of rows within each partition before the function executes.

- ROWS/RANGE BETWEEN (The Frame): Limits the calculation to a moving subset of rows relative to the current row.

```sql
DROP TABLE IF EXISTS employee_salaries;

CREATE TABLE employee_salaries(
            emp_id INT,
            emp_name VARCHAR(50),
            department VARCHAR(50),
            salary INT
);

INSERT INTO employee_salaries VALUES
(1, 'Amit', 'Data Science', 95000),
(2, 'Neha', 'Engineering', 110000),
(3, 'Ananya', 'Marketing', 70000),
(4, 'Rohan', 'Data Science', 80000),
(5, 'Pooja', 'Engineering', 11500),
(6, 'Vikram', 'Engineering', 105000);
```

**The Query: Partitioned vs. Global Aggregates**  
Query to compare a global table average against a partitioned department average, all while retaining employee-level detail.

```sql
SELECT 
    emp_name,
    department,
    salary,
    -- Global Average (No partition) -- 
    AVG(salary) OVER() AS company_avg_salary,
    -- Partitioned Average (Calculated per department slice) -- 
    AVG(salary) OVER(PARTITION BY department) AS dept_avg_salary
FROM employee_salaries;
```

----------------

**Ranking Functions & Dealing with Ties**
  
When ranking rows within a partition, standard SQL provides three functions. They behave identically when values differ, but diverge completely when they encounter ties (identical values).  

Mechanics of the Big Three  
- ROW_NUMBER(): Strictly sequential (1, 2, 3, 4). Ties are assigned arbitrary unique numbers based on engine optimization.  
- RANK(): Assigns identical ranks to ties, but skips values to maintain chronological count (1, 1, 3, 4).  
- DENSE_RANK(): Assigns identical ranks to ties, but never skips a rank value (1, 1, 2, 3).

**Visual Comparison of Ranking Functions**  
Within the 'Data Science' partition, both Amit and Neha earn $95,000.

```sql
SELECT department,
emp_name,
salary,
ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) AS seq_num,
RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS rank_val,
DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS dense_rank_val
FROM employee_salaries;
```

**Value Navigation (LEAD, LAG) & Running Totals**  
In financial and time-series analysis, we frequently need to compare the current row against adjacent rows or compute a running summation over time.  

1. Value Navigation  
- LAG(column, offset): Looks backward in the sorted partition by a specified number of rows (default is 1).  
- LEAD(column, offset): Looks forward in the sorted partition.

2. The Default Running Total Frame  
When we use an aggregate function like SUM() paired with an ORDER BY clause inside OVER(), SQL implicitly assumes a dynamic window frame:

Accumulates values sequentially from the top of the partition down to the active row.

**Time-Series Sandbox**

```sql
DROP TABLE IF EXISTS daily_sales;

CREATE TABLE daily_sales (
    sale_date DATE,
    revenue INT
);

INSERT INTO daily_sales VALUES 
('2026-07-01', 5000),
('2026-07-02', 7000),
('2026-07-03', 6000),
('2026-07-04', 9000),
('2026-07-05', 8500);
```

```sql
SELECT 
    sale_date,
    revenue,
    -- 1. Running Cumulative Total
    SUM(revenue) OVER (ORDER BY sale_date) AS cumulative_revenue,

    -- 2. Previous Day's Value
    LAG(revenue, 1) OVER (ORDER BY sale_date) AS yesterday_revenue,

    -- 3. Day-over-Day Variance Calculation
    revenue - LAG(revenue, 1) OVER (ORDER BY sale_date) AS revenue_delta
FROM daily_sales;
```

----------

**Value Window Functions (FIRST_VALUE, LAST_VALUE, NTH_VALUE, NTILE)**

Value window functions retrieve values from specific relative rows within a defined window frame without changing the row grain of the query.

**Application**  
- Comparing an individual row against the baseline first or last transaction of a entity.- Segmenting numeric distributions into equal parts (quartiles, percentiles) for statistical analysis.
- Finding the $N$-th event in a customer's activity timeline.

**Real-World Problem & Solution**  
Problem: A healthcare analytics team needs to calculate the difference between every patient's bill and their first initial bill amount, while keeping every individual bill visible in the output.

Solution: Standard GROUP BY collapses individual billing rows into summary rows. FIRST_VALUE() OVER (...) retrieves the first bill's value across the partition while retaining all individual bill records.

**Industrial Example**  
A financial trading platform evaluates stock trades by comparing each executing price against the opening price (FIRST_VALUE) and the closing price (LAST_VALUE) for that specific trading session.

```sql
-- Query demonstrating Value Window Functions
SELECT 
    patient_id,
    bill_id,
    bill_date,
    total_amount,

    -- 1. Retrieves the first bill amount for each patient
    FIRST_VALUE(total_amount) OVER (
        PARTITION BY patient_id 
        ORDER BY bill_date ASC
    ) AS initial_bill_amount,

    -- 2. Calculates difference relative to the first bill
    total_amount - FIRST_VALUE(total_amount) OVER (
        PARTITION BY patient_id 
        ORDER BY bill_date ASC
    ) AS delta_from_initial,

    -- 3. Retrieves the 2nd bill amount for each patient
    NTH_VALUE(total_amount, 2) OVER (
        PARTITION BY patient_id 
        ORDER BY bill_date ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_bill_amount,

    -- 4. Divides all bills across the whole table into 4 equal distribution buckets (Quartiles)
    NTILE(4) OVER (
        ORDER BY total_amount DESC
    ) AS revenue_quartile
FROM Bill;
```

FIRST_VALUE(col): Returns the value of col from the first row in the window frame.

NTH_VALUE(col, N): Returns the value of col from the $N$-th row in the window frame.

NTILE(buckets): Splits ordered rows across the partition into $N$ roughly equal buckets and assigns a bucket number ($1$ to $N$) to each row.

**Important**  
The Frame Trap with LAST_VALUE: The default window frame for ORDER BY is RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW. This means LAST_VALUE() will return the current row's value unless you explicitly define the frame as ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING.

---

**Gaps-and-Islands Analysis Pattern**

"Gaps-and-Islands" is a classic SQL analytical pattern used to detect contiguous sequences of consecutive records ("Islands") and identify missing intervals or breaks ("Gaps") within ordered time-series or sequential data.

**Application**
- Finding consecutive days a patient took their medication vs. missed days.
- Calculating maximum streaks (e.g., active daily user streaks, consecutive days of hospital room occupancy).
- Identifying missing sequential primary key sequence IDs or invoice numbers.

**Real-World Problem & Solution**  
Problem: An IoT monitoring network records daily temperature logs for hospital vaccine storage. If sensor reporting drops for multiple days, finding those missing continuous ranges ("Gaps") or identifying uninterrupted valid temperature monitoring periods ("Islands") using standard WHERE filters is mathematically impossible.

Solution: Combining ROW_NUMBER() or LAG() with value difference calculations groups contiguous dates into shared sequence markers.

**Industrial Example**  
A hospital bed management dashboard needs to identify continuous uninterrupted date ranges when an Intensive Care Unit (ICU) bed was continuously occupied ("Islands") to compute occupancy efficiency.

```sql
-- Step 1: Generate consecutive group identifiers using ROW_NUMBER subtraction
WITH Consecutive_Groups AS (
    SELECT 
        patient_id,
        visit_date,
        -- Subtracting row numbers from the calendar date creates a static date anchor for continuous runs
        visit_date - (ROW_NUMBER() OVER (PARTITION BY patient_id ORDER BY visit_date) * INTERVAL '1 day') AS group_anchor
    FROM Patient_Visits
)
-- Step 2: Aggregate by the generated anchor to discover the "Islands"
SELECT 
    patient_id,
    MIN(visit_date) AS island_start_date,
    MAX(visit_date) AS island_end_date,
    COUNT(*) AS consecutive_days_count
FROM Consecutive_Groups
GROUP BY patient_id, group_anchor
HAVING COUNT(*) >= 2
ORDER BY patient_id, island_start_date;
```

**Code Breakdown -**   
- PARTITION BY patient_id: Restarts the row numbering independently for every unique patient.
- ORDER BY visit_date: Assigns sequential numbers (1, 2, 3...) to visits ordered chronologically.
- ROW_NUMBER() * INTERVAL '1 day': Converts the integer row sequence into a day-based duration format.
- visit_date - INTERVAL: Calculates the static group_anchor. All consecutive visit dates within the same streak evaluate to the exact same group_anchor value.

&&

- GROUP BY patient_id, group_anchor: Groups all continuous visit days belonging to the same streak together.
- MIN(visit_date): Identifies the start date of the continuous streak ("Island Start").
- MAX(visit_date): Identifies the end date of the continuous streak ("Island End").
- COUNT(*): Calculates the length of the streak in total days.
- HAVING COUNT(*) >= 2: Filters out isolated single-day visits, keeping only streaks of 2 or more consecutive days.
- ORDER BY patient_id, island_start_date: Displays output chronologically per patient.


**Important**
- Gaps Technique: To find missing sequences ("Gaps"), pair LAG(event_date) with a filter checking where event_date - LAG(event_date) > 1 day.
- Standard Compatibility: This anchor math works universally across PostgreSQL, SQL Server, MySQL, Oracle, Snowflake, and BigQuery.

----

**Overview: Window Function Categories**   

**Ranking Functions**  
- Functions: ROW_NUMBER(), RANK(), DENSE_RANK()
- Purpose: Assigns positional order to rows within a partition without collapsing them.

**Value / Offset Functions**  
- Functions: LAG(), LEAD(), FIRST_VALUE(), LAST_VALUE()
- Purpose: Compares records across adjacent or boundary rows within a defined window frame.

**Distribution Functions**  
- Functions: NTILE(), CUME_DIST(), PERCENT_RANK()
- Purpose: Segments row distributions statistically into buckets, cumulative percentages, or relative ranks.

**Aggregate Functions**  
- Functions: SUM() OVER(), AVG() OVER(), COUNT() OVER()
- Purpose: Calculates running totals, moving averages, and cumulative counts across rows.

---

**WINDOW FRAMES(The Physical Moving Window)** 

When working with window functions, using PARTITION BY and ORDER BY defines the broad group and the order of rows. A Window Frame goes one step further: it defines the exact, dynamic boundary of physical rows relative to the current row that SQL should look at to perform a calculation.

Don't just look at the whole partition; look only at a specific moving slice of rows around the active row.

**The Boundary Keywords**
- UNBOUNDED PRECEDING: Starts from the very first row of the partition.  
- N PRECEDING: Reaches back N rows prior to the active row.
- CURRENT ROW: Stops or starts at the current row being processed. 
- N FOLLOWING: Reaches forward N rows ahead of the active row.  
- UNBOUNDED FOLLOWING: Extends all the way to the last row of the partition.

**REAL-WORLD SCENARIO**

1. 7-Day Rolling Revenue Average  

Practical Business Need: Smooths out weekend volatility and daily spikes in e-commerce traffic to see clear underlying trends.
Frame Boundary Definition: ROWS BETWEEN 6 PRECEDING AND CURRENT ROW

2. 3-Day Centered Moving Average  

Practical Business Need: Analyzes daily metrics by averaging yesterday's performance, today's performance, and tomorrow's projected performance.
Frame Boundary Definition: ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING

3. Year-to-Date (YTD) Cumulative Sales  

Practical Business Need: Tracks the continuous accumulation of total revenue starting from the first day of the year up to the current day.
Frame Boundary Definition: ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

4. Forward Inventory Coverage  

Practical Business Need: Aggregates upcoming demand forecasts over the next 3 days to manage stock and avoid supply shortages.
Frame Boundary Definition: ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING

**The Scenario & Solution**  
Suppose a business tracks daily store revenue and wants to calculate a 3-Day Rolling Average Revenue (taking the current day and the previous 2 days) to smooth out sudden spikes and monitor underlying sales trends.

```sql
DROP TABLE IF EXISTS store_sales;

CREATE TABLE store_sales (
    sale_date DATE,
    revenue INT
);

INSERT INTO store_sales VALUES 
('2026-07-01', 10000),
('2026-07-02', 12000),
('2026-07-03', 15000),
('2026-07-04', 9000),
('2026-07-05', 18000),
('2026-07-06', 20000);
```

```sql
SELECT 
    sale_date,
    revenue,
    -- 3-Day Rolling Average: Current Day + 2 Days Prior
    AVG(revenue) OVER (
        ORDER BY sale_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_3day_avg
FROM store_sales;
```

Explanation -   
Step-by-Step Row Calculation Logic 

Row 1 (2026-07-01): No preceding rows exist. Frame contains [10000].  
{Average} = {10000}/{1} = {10000.0}  

Row 2 (2026-07-02): 1 preceding row exists. Frame contains [10000, 12000].  
{Average} = {10000 + 12000}/{2} = {11000.0} 

Row 3 (2026-07-03): 2 preceding rows exist. Frame contains [10000, 12000, 15000]  
{Average} = {10000 + 12000 + 15000}/{3} ={12333.33}  

Row 4 (2026-07-04): Frame drops the oldest row (10000) and shifts   forward to [12000, 15000, 9000]  
{Average} = {12000 + 15000 + 9000}/{3} = {12000.0}

**ROWS VS RANGE in Window Frame**  
-- Physical Row Count  
ROWS BETWEEN 1 PRECEDING AND CURRENT ROW  

ROWS (Physical Position): Counts physical rows exact distance away (N rows back/forward), regardless of row values or duplicate timestamps.  

RANGE (Logical Value): Evaluates the actual value inside the ORDER BY column. If two rows have identical dates or timestamps (ties), RANGE treats them as a single bucket and includes all tied rows together in the calculation.

-- Logical Value Range  
RANGE BETWEEN INTERVAL '7' DAY PRECEDING AND CURRENT ROW

Production Tip: Always default to ROWS when calculating moving averages over strict physical row counts, as it avoids unexpected frame expansions caused by duplicate values and performs significantly faster in database engines.

---------------------

-----

**COMMON TABLE EXPRESSIONS (CTE)**

A Common Table Expression (CTE) is a temporary, named result set defined using a WITH clause. It exists strictly within the execution scope of a single SELECT, INSERT, UPDATE, or DELETE statement, serving as a clean alternative to complex subqueries.

*Application*
- Breaking down long, multi-stage data pipelines into modular, readable logic steps.

- Eliminating redundant subqueries across complex JOIN operations.

- Implementing self-referencing recursive operations for hierarchical data structures (e.g., organizational charts, product categories).

- Preparing data for window functions and post-aggregation filtering.

*Real-World Scenario*

Scenario:  
An e-commerce platform needs to evaluate customer retention. Management wants a list of "High-Value Active Customers" (customers who spent over $1,000 in total across at least 3 distinct orders) alongside their average transaction amount.

Problem Solved:  
Writing this with nested subqueries requires wrapping aggregate queries inside FROM and WHERE clauses, making the SQL hard to read and debug. A CTE isolates the user-level aggregations upfront, allowing the main query to remain clean and declarative.

```sql
-- Step 1: Stage customer metrics using a CTE
WITH CustomerMetrics AS (
    SELECT 
        customer_id,
        COUNT(order_id) AS total_orders,
        SUM(amount) AS total_spent,
        AVG(amount) AS avg_order_value
    FROM orders
    GROUP BY customer_id
),
-- Step 2: Filter for high-value active customers
HighValueCustomers AS (
    SELECT 
        customer_id,
        total_orders,
        total_spent,
        avg_order_value
    FROM CustomerMetrics
    WHERE total_spent > 1000 
      AND total_orders >= 3
)
-- Step 3: Final output query
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    h.total_orders,
    h.total_spent,
    ROUND(h.avg_order_value, 2) AS avg_order_value
FROM HighValueCustomers h
JOIN customers c ON h.customer_id = c.customer_id
ORDER BY h.total_spent DESC;
```

*Explanation* - 
- WITH CustomerMetrics AS (...): Computes aggregate order count, total spend, and average spend per customer_id directly from the base orders table.

- HighValueCustomers AS (...): References CustomerMetrics as if it were a physical table, applying business logic rules (total_spent > 1000 and total_orders >= 3).

- Final Query: Joins HighValueCustomers back to the customers table to pull demographic fields (first_name, last_name) and displays formatted metrics sorted by total spend.

*Important Point*-

- Scope & Lifecycle: CTEs exist only for the duration of the query execution and do not persist in database storage or system catalogs.

- Chaining Capability: Multiple CTEs can be defined sequentially separated by commas under a single WITH keyword.

- Optimization: Modern SQL query engines (including SQLite 3.25+) optimize CTEs by either inlining them into the main execution plan or materializing them dynamically.

- Recursive Variant: Adding the RECURSIVE keyword (WITH RECURSIVE) enables iterative processing, allowing queries to traverse trees, organizational hierarchies, or graph models.

----------

**Multiple / Chained CTEs**-

A Multiple or Chained Common Table Expression (CTE) is a sequence of two or more temporary named result sets defined under a single WITH keyword, separated by commas. Each subsequent CTE in the chain can directly reference any CTE defined before it in the same statement, allowing complex queries to be built as a step-by-step logical pipeline within a single execution scope.

*APPLICATIONS -*

- Multi-Stage Data Pipelines: Breaking down long, multi-stage data processing pipelines into modular, clean logic steps.

- Eliminating Nested Subqueries: Replacing complex, unreadable subqueries wrapped inside FROM and WHERE clauses.

- Separation of Aggregation & Business Logic: Performing raw data aggregations in an initial stage, applying threshold business rules in a secondary stage, and handling presentation joins in the main query.

- Preparing Data for Downstream Functions: Pre-aggregating or filtering datasets before applying window functions, conditional aggregations, or complex sorting rules.

*Real-World Scenario & Problem Solved*  

*Scenario:*   
An e-commerce platform needs to evaluate customer retention. Management requests a list of "High-Value Active Customers"—defined as customers who have spent over $1,000 in total across at least 3 distinct orders—alongside their average transaction amount and customer demographic details.

*Problem Solved:*  
Writing this with traditional subqueries requires nesting aggregate queries inside FROM and HAVING clauses, making the query difficult to maintain, read, and debug. Chaining CTEs isolates the user-level aggregations upfront in CustomerMetrics, applies the business filters cleanly in HighValueCustomers, and keeps the outer query dedicated to joining demographic data.

```sql
-- 1. Reset tables to avoid conflicts with earlier notebook cells

-- 2. Create tables and populate sample data

DROP TABLE IF EXISTS orders;

DROP TABLE IF EXISTS customers;

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10,2)
);

INSERT INTO customers VALUES 
    (1, 'Sarah', 'Miller'),
    (2, 'Aaron', 'Smith'),
    (3, 'Paul', 'Jones');

INSERT INTO orders VALUES 
    (101, 1, 450.00),
    (102, 1, 350.00),
    (103, 1, 300.00),
    (104, 2, 600.00),
    (105, 2, 500.00),
    (106, 3, 100.00);
```

```sql
-- 1. Execute the chained CTE query

-- Step 1: Stage customer metrics using an initial CTE
WITH CustomerMetrics AS (
    SELECT
        customer_id,
        COUNT(order_id) AS total_orders,
        SUM(amount) AS total_spent,
        AVG(amount) AS avg_order_value
    FROM orders
    GROUP BY customer_id
),
-- Step 2: Filter for high-value active customers in a chained CTE
HighValueCustomers AS (
    SELECT
        customer_id,
        total_orders,
        total_spent,
        avg_order_value
    FROM CustomerMetrics
    WHERE total_spent > 1000
      AND total_orders >= 3
)
-- Step 3: Final output query joining demographic data
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    h.total_orders,
    h.total_spent,
    ROUND(h.avg_order_value, 2) AS avg_order_value
FROM HighValueCustomers h
JOIN customers c ON h.customer_id = c.customer_id
ORDER BY h.total_spent DESC;
```

Explanation - 
- WITH CustomerMetrics AS (...): Begins the CTE block and computes aggregate order count, total spend, and average spend per customer_id directly from the base orders table.

- Comma Separator (,): Ends the definition of CustomerMetrics and allows a second CTE to be declared without repeating the WITH keyword.

- HighValueCustomers AS (...): References CustomerMetrics as if it were a physical database table, applying business logic filtering (total_spent > 1000 and total_orders >= 3).

- Final Output Query: Queries HighValueCustomers (h), joins it with the customers base table (c) on customer_id to retrieve user demographic fields (first_name, last_name), rounds avg_order_value to two decimal places, and sorts the final output by total_spent in descending order.

-------

**Recursive CTEs(Hierarchical & Tree Traversal -**

A Recursive CTE is a Common Table Expression that references its own name within its own definition. Declared using WITH RECURSIVE, it consists of two distinct components linked by UNION ALL (or UNION): 
1. Anchor Member (the base query that runs once to establish the initial result set) 
2. Recursive Member (a query that repeatedly executes against the previous iteration's result set until no new rows are returned).

*Application -*

- Hierarchical Tree Traversal: Navigating multi-level parent-child structures like organizational charts, reporting lines, and management chains.
- Category Taxonomies: Querying deeply nested e-commerce category trees (e.g., Electronics -> Computers -> Laptops -> Gaming Laptops).
- Bill of Materials (BOM): Breaking down complex manufactured products into sub-assemblies and individual raw component parts.
- Dynamic Series Generation: Generating ranges of dates, sequences of numbers, or schedules without relying on physical tally tables.

*Real-World Scenario & Problem Solved*

*Scenario:* An HR analytics department needs to generate an organizational hierarchy chart. Management requires a report listing every employee, their direct manager, and their exact depth level in the company hierarchy, starting with the CEO at Level 1 down to front-line individual contributors.

*Problem Solved:* Relational databases store parent-child links across rows (manager_id pointing to employee_id). Standard JOIN statements require knowing the exact depth of the tree in advance (e.g., writing 5 explicit LEFT JOINs for 5 levels). If the tree deepens to 6 levels, a static query fails. A Recursive CTE dynamically explores all hierarchy levels automatically regardless of depth.

```sql
DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    manager_id INT
);

INSERT INTO employees VALUES
    (1, 'Alice (CEO)', NULL),      -- Top of hierarchy (Level 1)
    (2, 'Bob (VP)', 1),           -- Reports to Alice (Level 2)
    (3, 'Charlie (VP)', 1),       -- Reports to Alice (Level 2)
    (4, 'David (Manager)', 2),    -- Reports to Bob (Level 3)
    (5, 'Eve (Developer)', 4);    -- Reports to David (Level 4)
```

```sql
WITH RECURSIVE OrgHierarchy AS (
    -- Anchor Member: Select the top-level entity (CEO, manager_id IS NULL)
    SELECT 
        employee_id,
        first_name,
        manager_id,
        1 AS depth_level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive Member: Join employees with the previous CTE iteration
    SELECT 
        e.employee_id,
        e.first_name,
        e.manager_id,
        h.depth_level + 1 AS depth_level
    FROM employees e
    JOIN OrgHierarchy h ON e.manager_id = h.employee_id
)
SELECT 
    employee_id,
    first_name,
    manager_id,
    depth_level
FROM OrgHierarchy
ORDER BY depth_level, employee_id;
```

--------

**Advanced Recursive CTEs (Path Tracking & Cycle Prevention)**

An Advanced Recursive CTE extends standard recursion by maintaining dynamic state-tracking variables (such as concatenated path strings or arrays/lists of visited nodes) across iterative steps. This enables the query to construct full end-to-end traversal paths and detect cyclic references (infinite loops) to safely terminate execution when handling network graph datasets.

An Advanced Recursive CTE is a recursive query that maintains a running "notepad" (an array or list of visited items) as it loops. It uses this notepad to build a history of the journey (Path Tracking) and to stop automatically if it tries to visit a place it has already been (Cycle Prevention).

*Application*  

Route Finding: Showing the full step-by-step path from start to finish (e.g., flight connections, package tracking, or transit routes).

Crash Prevention: Preventing database crashes caused by circular links in your data (e.g., employee circular management loops or circular product dependencies).

*Real-World Scenario & Problem Solved*

Scenario: We want to find all possible flight routes starting from NYC.  
Flight 1: NYC -> Chicago
Flight 2: Chicago -> Miami
Flight 3: Miami -> NYC (Notice Miami connects back to NYC!)

Problem: If we run a standard recursive query, SQL will keep searching indefinitely: 
NYC -> Chicago -> Miami -> NYC -> Chicago -> Miami... until the database crashes or runs out of memory.

How It Solves the Problem: The query carries a list variable (visited_airports). When searching for the next flight, SQL asks: "Is this destination airport already in my visited_airports list?" When it reaches NYC again, the answer is Yes, so SQL stops searching that branch.

```sql
-- 1. Setup table with circular flight data

-- 2. Run Recursive CTE with Path Tracking & Cycle Prevention

DROP TABLE IF EXISTS flights;

CREATE TABLE flights (
    origin VARCHAR(10),
    destination VARCHAR(10)
);

INSERT INTO flights VALUES
    ('NYC', 'ORD'),
    ('ORD', 'LAX'),
    ('LAX', 'NYC'),  -- Circular loop! LAX points back to NYC
    ('ORD', 'DFW');

WITH RECURSIVE FlightPaths AS (
    -- STEP 1: ANCHOR (Start at NYC)
    SELECT 
        origin,
        destination,
        -- Path Tracker: Visual string of route
        CAST(origin || ' -> ' || destination AS VARCHAR(500)) AS full_path,
        -- The "Notepad": A list initialized with ['NYC', 'ORD']
        [origin, destination] AS visited_airports
    FROM flights
    WHERE origin = 'NYC'

    UNION ALL

    -- STEP 2: RECURSIVE STEP (Find connecting flights)
    SELECT 
        f.origin,
        f.destination,
        -- Update Path Tracker: Append next stop to the visual string
        p.full_path || ' -> ' || f.destination AS full_path,
        -- Update Notepad: Add the new destination to our list
        list_append(p.visited_airports, f.destination) AS visited_airports
    FROM flights f
    JOIN FlightPaths p ON f.origin = p.destination

    -- STEP 3: CYCLE PREVENTION (The Guard Rule)
    -- "Only take this flight if the destination is NOT already on our notepad!"
    WHERE NOT list_contains(p.visited_airports, f.destination)
)
SELECT 
    full_path,
    visited_airports
FROM FlightPaths;
```

*Line-by-Line Breakdown*

- full_path Construction (origin || ' -> ' || destination): Initializes and appends strings at each iteration step to build a human-readable visual route string (e.g., NYC -> ORD -> LAX).

- visited_airports List ([origin, destination]): Creates a DuckDB list data structure containing all airports visited in the current path execution branch.

- list_append(p.visited_airports, f.destination): Dynamically appends each newly reached destination airport to the list of visited locations during each recursive step.

- WHERE NOT list_contains(...): Acts as the cycle prevention filter. Before joining a candidate connecting flight, it checks whether f.destination is already stored inside p.visited_airports. If it is present, that recursive path branch terminates immediately, preventing an infinite loop.

*Important Points*  

Path Tracking: Uses string concatenation (|| ' -> ' ||) or list appending to keep a history of every step taken.

Cycle Prevention: Uses list/array searching (list_contains) inside the WHERE clause to block infinite loops.

Database Syntax: DuckDB uses list_append() and list_contains(). Other databases use similar functions (e.g., PostgreSQL uses array functions like array_append()).

---

**Writable / Data-Modifying CTEs**

A Writable (or Data-Modifying) CTE is a Common Table Expression that contains data-modifying SQL statements (INSERT, UPDATE, or DELETE) combined with a RETURNING clause. Instead of merely reading data, it alters data in tables and captures the modified or deleted rows in memory, making them immediately available to be queried or inserted elsewhere within the exact same statement.

**Application**

Atomic Archiving & Purging:  Moving old or cancelled records from an active database table into an archive table in a single query step.

Audit Logging:  Automatically capturing modified or deleted rows and inserting them into an audit log table.

Cascading Data Modifications:  Capturing primary key IDs generated by an INSERT operation in the CTE to insert related child records in the main statement.

**Real-World Scenario & Problem Solved**  

*Scenario:* A hospital system needs to purge cancelled appointments from the active appointment table while concurrently moving those exact cancelled records into an appointment_archive table for compliance tracking.

*Problem Solved:* Traditionally, this requires two separate queries: first INSERT INTO archive SELECT ... FROM appointment, followed by DELETE FROM appointment. If the database crashes between these steps or if new records are modified concurrently, data can be duplicated or lost. A Writable CTE performs both the DELETE and the INSERT atomically in a single execution step—eliminating race conditions and preventing data mismatch.

```sql
DROP TABLE IF EXISTS appointment_archive;

DROP TABLE IF EXISTS appointment;

CREATE TABLE appointment ( 
    appointment_id INT PRIMARY KEY,
    patient_id INT,
    fee DECIMAL(10,2),
    status VARCHAR(20)
);

CREATE TABLE appointment_archive(
    appointment_id INT,
    patient_id INT,
    fee DECIMAL(10,2),
    status VARCHAR(20),
    archived_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO appointment VALUES
    (1, 101, 150.00, 'Completed'),
    (2, 102, 200.00, 'Cancelled'),
    (3, 103, 100.00, 'Cancelled'),
    (4, 104, 250.00, 'Completed');
```

```sql
WITH moved_appointment AS(
DELETE FROM appointment
WHERE status = 'Cancelled'
RETURNING appointment_id, patient_id, fee, status
)

INSERT INTO appointment_archive(appointment_id, patiend_id, fee, status)
SELECT appointment_id, patient_id, fee, status
FROM moved_appointments;
```

**Line-by-Line Breakdown**  

WITH moved_appointments AS (...): Begins the CTE block wrapping a data modification statement.

DELETE FROM appointment WHERE status = 'Cancelled': Removes rows matching the criteria directly from the active appointment table.

RETURNING appointment_id, patient_id, fee, status: Intercepts the deleted rows before they vanish and outputs them as a temporary CTE result set.

INSERT INTO appointment_archive ... SELECT ... FROM moved_appointments: Takes the returned rows directly out of memory and writes them into the appointment_archive table.

*Important Points*

Atomicity: The operation is completely atomic—either both the deletion and insertion succeed together, or the entire statement rolls back if an error occurs.

RETURNING Clause Requirement: A data-modifying CTE must include a RETURNING clause so downstream queries can consume the affected rows.

----

**CTE Materialization Control (MATERIALIZED vs NOT MATERIALIZED)**

CTE Materialization Control allows you to explicitly instruct the database's query optimizer whether to compute a CTE once and store its result set in memory as a temporary cached table (MATERIALIZED), or to inline/substitute the CTE code directly into every place it is referenced in the outer query (NOT MATERIALIZED).

**Application** 

Query Performance Tuning: Preventing the database engine from recalculating heavy, expensive aggregations multiple times when referenced repeatedly within a single statement.

Optimization Fence Control: Controlling whether predicate pushdown (applying outer WHERE filters directly inside the CTE definition before execution) should be enabled or blocked.

Resource Optimization: Managing temporary memory consumption during large-scale analytical transformations.

**Real-World Scenario & Problem Solved** 

Scenario: An e-commerce reporting pipeline runs a heavy daily aggregate query over millions of transactions to calculate daily revenue metrics. This daily summary CTE is referenced twice within the same overall query: once in a branch pulling high-performing days, and again in a UNION ALL branch pulling low-performing days.

Problem Solved: Without explicit materialization control, query planners might inline the CTE code into both branches. This forces the engine to scan millions of rows and compute the GROUP BY aggregation twice. Forcing MATERIALIZED guarantees that the CTE executes once into memory, allowing both query branches to read directly from the cached result set.

```sql
DROP TABLE IF EXISTS sales;

CREATE TABLE sales(
sale_id INT PRIMARY KEY,
sale_date DATE,
amount DECIMAL(10,2)
);

INSERT INTO sales VALUES
    (1, '2026-08-01', 500.00),
    (2, '2026-08-01', 700.00),  -- Aug 1 Total: 1200
    (3, '2026-08-02', 150.00),  -- Aug 2 Total: 150
    (4, '2026-08-03', 2000.00); -- Aug 3 Total: 2000
```

```sql
WITH daily_summary AS MATERIALIZED (
SELECT 
    sale_date,
    COUNT(sale_id) AS total_transations,
    SUM(amount) AS total_revenue
    FROM sales
    GROUP BY sale_date
    )

SELECT sale_date, total_revenue, 'High Performance' AS performance_category
FROM daily_summary
WHERE total_revenue >= 1000

UNION ALL

SELECT sale_date, total_revenue, 'Low Performance' AS performance_category
FROM daily_summary
WHERE total_revenue < 1000
```

**Line-by-Line Breakdown**

WITH daily_summary AS MATERIALIZED (...): The AS MATERIALIZED clause forces DuckDB to evaluate the CTE immediately, storing the output in memory as a temporary result set before running the outer query.

SELECT sale_date, COUNT(...), SUM(...) FROM sales GROUP BY sale_date: Computes daily aggregations. Because of MATERIALIZED, this grouping calculation executes exactly once.

Branch 1 (WHERE total_revenue >= 1000): Queries the cached daily_summary table in memory to filter high-revenue days.

UNION ALL: Combines results from both query branches.

Branch 2 (WHERE total_revenue < 1000): Queries the same in-memory cached daily_summary result set without re-evaluating the raw sales table.

**Important Points**

MATERIALIZED (Optimization Fence): Evaluates the CTE once and caches the result. It blocks predicate pushdown (outer filters are NOT pushed inside the CTE).

NOT MATERIALIZED (Inlining): Allows the query engine to collapse the CTE logic directly into the main query, enabling predicate pushdown for maximum filtering before aggregation.

__________

**Subquery Fundamentals (Nested Queries)**

A subquery (also known as an inner query or nested statement) is a SELECT query embedded inside another outer SQL statement (SELECT, INSERT, UPDATE, or DELETE).

**Application**  
Used to perform multi-step data retrieval in a single query by feeding the intermediate results of an inner operation directly into the conditional logic of an outer statement.

**Real-World Problem and How Particular Topic Solves It**  
Problem: Retrieving orders placed on the absolute latest available date in the system without manually hardcoding or copying dates from a separate lookup query.

How It Solves It: Automatically computes the maximum date inside an inner query and hands that result directly to the outer filtering condition in a single execution step.

**Industrial Example**    
In financial audit systems, pulling all transactions processed on the most recent accounting close date without needing human intervention to manually supply the date variable.

```sql
CREATE TABLE IF NOT EXISTS orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10, 2)
);

SELECT 
    order_id, 
    customer_id, 
    order_date, 
    total_amount
FROM orders
WHERE order_date = (
    SELECT MAX(order_date) 
    FROM orders
);
```

The database evaluates the inner subquery (SELECT MAX(order_date) FROM orders) first.  
The inner query computes the single latest date present in the table (e.g., '2026-08-16').  
The outer query substitutes the inner block with that value (WHERE order_date = '2026-08-16').  
The main query executes and returns all rows matching that date.

----

**Multi-Column Subqueries**

A multi-column subquery is a nested query that evaluates and returns more than one column per row to the outer statement, using row/tuple comparison operators like (col1, col2) IN (...).

**Application**  
Used to evaluate composite matching conditions where two or more attributes must match simultaneously as a unit across tables or grouped sets.

**Real-World Problem & How It Solves It**  

Problem: Finding the employee who holds the maximum salary within each department. Filtering by salary IN (SELECT MAX(salary) FROM employees GROUP BY department_id) fails because an employee in Department A might happen to earn the exact same salary amount as the maximum salary of Department B, incorrectly adding them to the results.

How It Solves It: Binds the department_id and salary together in a single tuple (department_id, salary) IN (...), ensuring the salary strictly pairs with its corresponding department.

**Industrial Example**  
In e-commerce fraud prevention, identifying transactions where the combination of (user_id, ip_address) matches a blacklisted list of fraudulent account-IP pairs.

```sql
DROP TABLE IF EXISTS employees;

CREATE TABLE IF NOT EXISTS employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    department_id INT,
    salary DECIMAL(10, 2)
);

INSERT INTO employees (employee_id, first_name, department_id, salary) VALUES
(1, 'Alice', 101, 90000.00),
(2, 'Bob', 101, 80000.00),
(3, 'Charlie', 102, 95000.00),
(4, 'Diana', 102, 95000.00),
(5, 'Evan', 103, 60000.00);
```

```sql
SELECT 
    employee_id, 
    first_name, 
    department_id, 
    salary
FROM employees
WHERE (department_id, salary) IN (
    SELECT 
        department_id, 
        MAX(salary)
    FROM employees
    GROUP BY department_id
);
```

The inner subquery groups the table by department_id and generates a paired matrix of (department_id, max_salary).

The outer query checks each row's (department_id, salary) combination against that list of pairs.

A row is included only if both the department ID and salary match the exact pair returned for that group.

**IMPORTANT**

DBMS Support & Column Order: The number, sequence, and data types of columns in the outer tuple must strictly match the column list inside the inner subquery's SELECT statement. (Note: Most major databases like PostgreSQL, MySQL, and Oracle support tuple subqueries, but SQL Server uses alternative syntax like CTEs or joins).

----

**Subqueries in the FROM Clause (Derived Tables)**

**Definition**  
A subquery placed inside the FROM clause that functions as a temporary, dynamically generated virtual table (also known as an inline view or derived table) for the outer query to select from.

**Application**  
Used when you need to perform multi-stage aggregations (e.g., taking an average of totals) or pre-filter and transform data before joining it with other tables.

**Real-World Problem and How Particular Topic Solves It**  
Problem: Calculating the average total spend per customer. SQL throws a syntax error if you attempt to nest aggregate functions directly like AVG(SUM(total_amount)).

How It Solves It: The subquery in the FROM clause calculates SUM(total_amount) grouped by customer first. The outer query then runs AVG() over those resulting customer totals.

**Industrial Example**  
In subscription analytics, calculating the average Monthly Recurring Revenue (MRR) per user tier by first aggregating individual account spends in a derived table, then computing the tier average in the main statement.

```sql
SELECT 
    AVG(customer_total) AS avg_spend_per_customer
FROM (
    SELECT 
        customer_id, 
        SUM(total_amount) AS customer_total
    FROM orders
    GROUP BY customer_id
) AS customer_summaries;
```

The inner subquery groups the orders table by customer_id and computes the sum of orders for each user.

It outputs a temporary result set labeled with the alias customer_summaries.

The outer SELECT treats customer_summaries like a physical table and calculates the average of the customer_total column.

____

**Subqueries in the SELECT Clause**

**Definition**  
A subquery placed directly inside the SELECT list of an outer query to calculate and project an additional single-value column alongside standard table attributes for every output row.

**Application**  
Used to append aggregated benchmark metrics (like global averages, max values, or department counts) next to individual row-level detail without altering the row grain of the main table.

**Real-World Problem and How Particular Topic Solves It**   
Problem: Displaying individual employee names and salaries alongside the company-wide average salary for side-by-side comparison. Running a standard AVG(salary) forces a GROUP BY that collapses individual employee rows.

How It Solves It: Embeds a scalar subquery inside the SELECT line. The subquery computes the global average once and projects it as an added column on every detailed row without collapsing dataset records.

**Industrial Example**  
In e-commerce inventory management, displaying each product's price alongside the highest price in its category to determine relative positioning on catalog listing pages.

```sql
SELECT 
    employee_id,
    first_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS company_avg_salary
FROM employees;
```

The main query scans the employees table row by row to retrieve employee_id, first_name, and salary.

For every output row, the embedded subquery (SELECT AVG(salary) FROM employees) evaluates to a single numeric value.

The resulting scalar average (e.g., 65000.00) is assigned the alias company_avg_salary and appended as a new column to every row in the output.

Must Return a Single Value: Subqueries in the SELECT clause must return strictly one row and one column per outer row execution. Returning multiple rows or columns triggers a database runtime error.

---

**Correlated Subqueries**

A correlated subquery is a nested query that evaluates repeatedly—once for every candidate row processed by the outer query—because it references column values passed down from the outer statement.

**Application**

Used to perform row-level contextual filtering where each record must be compared against an aggregate metric calculated dynamically for its specific group or entity.

**Real-World Problem and How Particular Topic Solves It**

Problem: Finding employees who earn more than the average salary of their own specific department (not the company-wide average).

How It Solves It: The inner query receives the current row's department_id from the outer query, calculates that specific department's average salary, and compares it against the employee's salary row by row.

**Industrial Example**

In fraud monitoring, identifying transactions where a payment amount exceeds three times that specific user's historical median spend.

```sql
-- Setup schema and sample data

-- Execute Correlated Subquery

DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    employee_id INT,
    first_name VARCHAR(50),
    department_id INT,
    salary DECIMAL(10, 2)
);

INSERT INTO employees VALUES
(1, 'Alice', 101, 90000.00),
(2, 'Bob', 101, 70000.00),
(3, 'Charlie', 102, 95000.00),
(4, 'Diana', 102, 60000.00);

SELECT 
    e1.employee_id,
    e1.first_name,
    e1.department_id,
    e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

**Explanation:**

1. The outer query alias e1 evaluates row 1 (Alice, department_id = 101).

2. e1.department_id (101) is passed into the inner subquery (e2).

3. The inner subquery calculates the average salary for department 101 (80000.00).

4. The outer query checks if Alice's salary (90000.00 > 80000.00) and outputs the row. This loop repeats for every row in e1.

**Important Point**

Performance Impact: Because a correlated subquery executes once per outer row, running it against large datasets can lead to O(N X M) performance bottlenecks. Window functions (AVG(salary) OVER (PARTITION BY department_id)) are usually a more performant alternative.

---

**EXISTS and NOT EXISTS**

EXISTS is a logical operator that checks whether a subquery returns any rows. It returns a boolean value (TRUE or FALSE) and short-circuits evaluation as soon as a single matching record is found.

**Application**

Used to test for record presence or absence across related tables without generating duplicate rows or loading large ID lists into memory.

**Real-World Problem and How Particular Topic Solves It**

Problem: Finding customers who have placed at least one order. Using JOIN can duplicate customer records if they have hundreds of orders, requiring a DISTINCT fix. Using IN loads entire order ID lists into memory.

How It Solves It: EXISTS checks the relationship per customer and stops searching the orders table the exact millisecond it finds 1 matching row for that customer.

**Industrial Example**

In user retention workflows, identifying churned accounts using NOT EXISTS to target users who created an account but have zero logged activity in the last 90 days.

```sql
-- Setup tables

-- Execute EXISTS query

DROP TABLE IF EXISTS customers;

DROP TABLE IF EXISTS orders;

CREATE TABLE customers (customer_id INT, customer_name VARCHAR(50));

CREATE TABLE orders (order_id INT, customer_id INT);

INSERT INTO customers VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Charlie');

INSERT INTO orders VALUES (101, 1), (102, 1);

SELECT 
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
);
```

**Explanation:**

1. For Alice (customer_id = 1), EXISTS scans orders, finds order_id = 101, returns TRUE immediately, and moves to the next customer.

2. For Charlie (customer_id = 3), EXISTS finds no matching rows in orders and returns FALSE.

3. SELECT 1 is standard convention because EXISTS only cares about row existence, ignoring column values returned.

**Important Point Related To It**

NULL Safety: Unlike NOT IN, which completely breaks and returns 0 rows if the subquery contains a single NULL value, NOT EXISTS handles NULL values in subqueries safely without failing or producing false negatives.

----

**ANY / ALL / SOME Operators**

The ANY, ALL, and SOME operators are logical comparison operators used in SQL WHERE or HAVING clauses to compare a single scalar value against a single-column set of values returned by a subquery:

- ANY / SOME: Evaluates to TRUE if the comparison is true for at least one value in the subquery result set. (ANY and SOME are exact logical equivalents).

- ALL: Evaluates to TRUE if the comparison is true for every single value in the subquery result set.

**Application**

- Set Threshold Comparisons: Comparing an entity metric against a group benchmark (e.g., finding products priced higher than all products in a competitor category, or finding sales reps who earned more than any rep in region B).

- Alternative to IN and EXISTS: = ANY provides identical functional logic to the IN operator, while <> ALL provides identical functional logic to NOT IN.

**Real-World Problem and Solution**

Problem: An e-commerce system needs to flag high-performing sellers. Specifically, management wants to identify products whose individual price is higher than every product listed in the 'Budget' category. Writing multiple queries or hardcoding numeric thresholds creates fragile application code as product catalog prices change dynamically.

Solution: Using 
price > ALL (SELECT price FROM products WHERE category = 'Budget') 
allows the database engine to resolve the comparative threshold dynamically in a single step.

**Industrial Example**

A human resources platform handles employee compensation audits. To identify potential salary compression or equity issues, the system runs a report listing non-management employees who earn a higher salary than at least one manager (
    
    salary > ANY (SELECT salary FROM employees WHERE role = 'Manager')
).

```sql
-- Target Table Setup
CREATE TABLE products (
    product_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

-- 1. ANY / SOME Operator Example
-- Finds products that are priced higher than AT LEAST ONE product in the 'Office' category
SELECT product_name, category, price
FROM products
WHERE price > ANY (
    SELECT price 
    FROM products 
    WHERE category = 'Office'
);

-- Note: '= ANY' is functionally identical to 'IN'
SELECT product_name, price
FROM products
WHERE category = ANY (VALUES ('Electronics'), ('Office'));

-- 2. ALL Operator Example
-- Finds products that are priced higher than ALL products in the 'Office' category
SELECT product_name, category, price
FROM products
WHERE price > ALL (
    SELECT price 
    FROM products 
    WHERE category = 'Office'
);

-- Note: '<> ALL' is functionally identical to 'NOT IN'
SELECT product_name, price
FROM products
WHERE category <> ALL (VALUES ('Obsolete'), ('Discontinued'));
```

**Code Breakdown:**

- price > ANY (...): Evaluates as price > MIN(subquery_value). If a product's price is greater than even the cheapest item in the subquery, the row is included.

- price > ALL (...): Evaluates as price > MAX(subquery_value). A product's price must exceed the single highest value returned by the subquery to be included.

- = ANY vs IN: Both syntax choices construct identical logical comparison plans inside database query planners.

**Important Points Related to It**

- Equivalence Rules:

    - = ANY is identical to IN.
    - <> ALL is identical to NOT IN.
    - ANY and SOME are exact synonyms in standard SQL grammar.

- NULL Value Trap with ALL (<> ALL / NOT IN Hazard): If the subquery evaluated by an ALL operator returns even a single NULL value, any <> comparison against it will evaluate to UNKNOWN (due to three-valued logic). As a result, the entire WHERE clause fails, returning zero rows.

SELECT * FROM products WHERE price <> ALL (SELECT price FROM discount_list);

- Performance Consideration: Modern database query optimizers typically flatten simple ANY/ALL constructs into equivalent join routines (like semi-joins or anti-joins). However, explicitly handling subquery NULL filters is crucial to ensure optimal index usage and prevent unexpected zero-row result sets.

-----

**Advanced Aggregation using ROLLUP**

**Definition**  
ROLLUP is an extension of the GROUP BY clause that automatically calculates hierarchical sub-totals and a grand total for a specified set of columns in a single query.

**Application**  
Used for generating hierarchical summary reports where you need data rolled up from detailed levels to higher aggregated levels (e.g., Year ➔ Quarter ➔ Month, or Region ➔ Country ➔ City).

**Real-World Problem and How Particular Topic Solves It**  
Problem: Creating a sales report that requires the total sales per store, the sub-total of sales per region, and a grand total of all sales company-wide. Normally, this requires writing three separate GROUP BY queries and stacking them together using UNION ALL.

How It Solves It: ROLLUP computes the standard group aggregations plus all the intermediate sub-totals and the final grand total in one single scan of the data, drastically reducing query complexity.

**Industrial Example**  
In Business Intelligence (BI) dashboards, feeding data to matrix visuals or pivot tables that allow executives to drill down from Global Revenue to Regional Revenue to Individual Store Revenue.

```sql
DROP TABLE IF EXISTS sales;

CREATE TABLE sales (
    region VARCHAR(50),
    store_location VARCHAR(50),
    revenue DECIMAL(10, 2)
);

INSERT INTO sales VALUES
('North', 'Store A', 1000.00),
('North', 'Store B', 1500.00),
('South', 'Store C', 2000.00);

SELECT 
    region, 
    store_location, 
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY ROLLUP (region, store_location);
```

The database first groups by both (region, store_location) and calculates the revenue for each specific store.

It then rolls up one level (dropping store_location to NULL) and calculates the sub-total for each (region).

Finally, it rolls up again (dropping region to NULL) and calculates the grand total for the entire table ().

The sequence of columns inside ROLLUP() is strictly hierarchical from left to right. ROLLUP(region, store_location) gives region sub-totals. ROLLUP(store_location, region) would incorrectly give store sub-totals across different regions.

---

**Advanced Aggregation using CUBE**

**Definition**  
CUBE is an extension of the GROUP BY clause that automatically calculates aggregations across every possible combination (permutation) of the specified columns.

**Application**  
Used for multi-dimensional data analysis (OLAP dynamic slicing and dicing) where you need to view metrics across all possible dimensions and intersections without relying on a strict hierarchy.

**Real-World Problem and How Particular Topic Solves**   
Problem: An analyst needs total sales grouped by region and category, but also needs overall totals by region alone, overall totals by category alone, and a grand total. Computing this manually requires writing 4 separate GROUP BY statements merged together using UNION ALL.

How It Solves It: CUBE(region, category) generates all 2^N possible grouping combinations (where N is the number of columns) in a single pass over the dataset.

**Industrial Example** 
In financial and retail data warehousing, pre-aggregating metrics across store locations, product lines, and customer age tiers to feed cross-tabular dashboards in tools like Power BI or Tableau.

```sql
DROP TABLE IF EXISTS sales_data;

CREATE TABLE sales_data (
    region VARCHAR(50),
    category VARCHAR(50),
    sales_amount DECIMAL(10, 2)
);

INSERT INTO sales_data VALUES
('North', 'Electronics', 1000.00),
('North', 'Clothing', 500.00),
('South', 'Electronics', 1500.00);

SELECT 
    region, 
    category, 
    SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY CUBE (region, category);
```

(region, category): Calculates total sales for every distinct region and category pair.  
(region, NULL): Calculates total sales per region across all categories.  
(NULL, category): Calculates total sales per category across all regions.  
(NULL, NULL): Calculates the overall grand total sales for the dataset.

Because CUBE generates 2^N combinations, adding too many columns can drastically slow down query execution. For example, passing 5 columns into CUBE generates 2^5 = 32 separate grouping sets in a single query execution.

---

**GROUPING SETS**

GROUPING SETS is a specialized SQL clause that allows you to specify exact, custom combinations of columns to aggregate within a single GROUP BY query, avoiding the full permutation overhead of CUBE or the rigid hierarchy of ROLLUP.

**Application** 

Used when you need specific sub-totals across multiple dimensions (e.g., total by Region, total by Category) without computing every intermediate combination or grand total.

**Real-World Problem and How Particular Topic Solves It**

Problem: Generating a summary dashboard that requires sales totals grouped by region alone AND sales totals grouped by category alone, but DOES NOT need totals for (region, category) combined or a grand total.

How It Solves It: GROUPING SETS executes only the requested groupings, eliminating unnecessary computation and keeping output datasets lean.

**Industrial Example**

In reporting platforms, generating multi-dimension KPI cards for executive summaries where distinct metrics must be calculated at non-hierarchical aggregate levels in a single query pass.

```sql
-- Setup table and sample data

-- Execute GROUPING SETS query directly

DROP TABLE IF EXISTS sales_data;

CREATE TABLE sales_data (
    region VARCHAR(50),
    category VARCHAR(50),
    sales_amount DECIMAL(10, 2)
);

INSERT INTO sales_data VALUES
('North', 'Electronics', 1000.00),
('North', 'Clothing', 500.00),
('South', 'Electronics', 1500.00);

SELECT 
    region, 
    category, 
    SUM(sales_amount) AS total_sales
FROM sales_data
GROUP BY GROUPING SETS (
    (region),
    (category)
);
```

**Explanation:**

The database aggregates sales_amount grouped strictly by region (setting category to NULL).

The database aggregates sales_amount grouped strictly by category (setting region to NULL).

It combines both results into a single output without computing the combined (region, category) pairs or grand totals.

**Important Point**

Efficiency: Both ROLLUP and CUBE are shortcuts built on top of GROUPING SETS. For example, ROLLUP(a, b) is equivalent to GROUPING SETS ((a, b), (a), ()).

---

**Data Transformation (PIVOT & UNPIVOT)**

**PIVOT (Rows to Columns)**  

PIVOT transforms unique row values from a single column into multiple individual columns, aggregating numerical metrics across those new column headings to create a wide-format cross-tabulation.

**Application**

Used to convert normalized long-format database tables into wide-format matrix reports for dashboards, pivot tables, and financial statements.

**Real-World Problem and How Particular Topic Solves It**

Problem: Raw sales tables store records vertically (Year, Quarter, Revenue). Showing quarterly figures side-by-side for each year requires manual grouping or complex filtering logic.

How It Solves It: PIVOT automatically rotates distinct quarterly row values (Q1, Q2, Q3, Q4) into horizontal columns and populates each cell with aggregated revenue.

**Industrial Example**

In financial reporting, turning vertical daily revenue logs into a horizontal table with months (Jan, Feb, Mar) as columns for side-by-side year-over-year comparison.

```sql
-- Setup sample table and long-format data

-- Execute PIVOT transformation

DROP TABLE IF EXISTS quarterly_sales;

CREATE TABLE quarterly_sales (
    year INT,
    quarter VARCHAR(10),
    revenue DECIMAL(10, 2)
);

INSERT INTO quarterly_sales VALUES
(2025, 'Q1', 10000.00),
(2025, 'Q2', 12000.00),
(2025, 'Q3', 15000.00),
(2025, 'Q4', 18000.00),
(2026, 'Q1', 11000.00),
(2026, 'Q2', 14000.00);

PIVOT quarterly_sales
ON quarter
USING SUM(revenue)
GROUP BY year;
```

**Explanation:**

1. ON quarter identifies the distinct string values (Q1, Q2, Q3, Q4) to turn into new table columns.

2. USING SUM(revenue) specifies the aggregation function used to fill the cells under each new column.

3. GROUP BY year keeps year as the primary vertical row identifier.

**Important Point Related To It**

Standard SQL Alternative: In database engines that lack a native PIVOT keyword (such as older SQLite versions), conditional aggregation using CASE WHEN inside SUM() achieves the exact same pivoting output:
SUM(CASE WHEN quarter = 'Q1' THEN revenue ELSE 0 END) AS Q1.

---

**UNPIVOT (Columns to Rows)**

UNPIVOT performs the inverse of pivoting—it transforms multiple column headers into a single attribute row column, flattening wide tables into normalized long-format records.

**Application**

Used during Data Integration (ETL/ELT) to normalize denormalized spreadsheets or flattened datasets before loading them into relational data warehouses.

**Real-World Problem and How Particular Topic Solves It**

Problem: Ingesting a spreadsheet where quarterly budgets exist in separate horizontal columns (Q1_Budget, Q2_Budget, Q3_Budget, Q4_Budget), making standard SQL filtering or time-series windowing impossible.

How It Solves It: UNPIVOT collapses the 4 quarterly columns into two long-format columns: Quarter_Name (storing column names as strings) and Budget_Amount (storing values).

**Industrial Example**

In marketing analytics, ingesting exported campaign spreadsheets where each target demographic exists as its own wide column, unpivoting them into normalized key-value pairs for database storage.

```sql
-- Setup wide-format table

-- Execute UNPIVOT transformation

DROP TABLE IF EXISTS wide_budgets;

CREATE TABLE wide_budgets (
    department VARCHAR(50),
    Q1_Budget DECIMAL(10, 2),
    Q2_Budget DECIMAL(10, 2)
);

INSERT INTO wide_budgets VALUES
('Engineering', 50000.00, 55000.00),
('Marketing', 30000.00, 32000.00);

UNPIVOT wide_budgets
ON Q1_Budget, Q2_Budget
INTO
    NAME quarter_name
    VALUE budget_amount;
```

**Explanation:**

1. ON Q1_Budget, Q2_Budget identifies the wide columns to dismantle into rows.

2. NAME quarter_name designates the new column that will store the original column names as text strings ('Q1_Budget', 'Q2_Budget').

3. VALUE budget_amount designates the new column that will store the numerical values extracted from those original columns.

**Important Point**

Data Loss on NULLs: By default, standard UNPIVOT operations automatically exclude rows where the source column contains NULL values. To retain missing data entries during transformation, use explicit INCLUDE NULLS options if supported by the engine.

---

**Indexing & Query Execution Performance**

Indexing creates secondary lookup data structures (typically B-Trees or Hash Indexes) to locate rows without scanning an entire table, while EXPLAIN inspects the query optimizer's execution plan to diagnose cost bottlenecks.

**Application**

Used to accelerate WHERE filtering, JOIN conditions, and ORDER BY sorting operations on large tables, converting costly full-table scans into targeted lookups.

**Real-World Problem & How It Solves It**

Problem: Searching for a specific record in a 50-million-row users table using WHERE email = 'user@example.com' forces a Sequential Scan (reading every single page on disk), taking several seconds or minutes.

How It Solves It: A B-Tree index on email reduces lookup complexity from O(N) full table scan to O(log N) tree traversal, fetching the matching record in milliseconds.

**Industrial Example**

In high-throughput authentication services, indexing the username and email columns prevents database CPU spikes during login traffic surges.

```sql
-- Setup table and sample dataset

-- Insert records

-- Create B-Tree index on email

-- Inspect execution plan using EXPLAIN

DROP TABLE IF EXISTS users;

CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100),
    signup_date DATE
);

INSERT INTO users SELECT 
    range AS user_id, 
    'user_' || range || '@example.com' AS email,
    DATE '2026-01-01' + INTERVAL (range % 365) DAYS AS signup_date
FROM range(1, 100000);

CREATE INDEX idx_users_email ON users(email);

EXPLAIN SELECT user_id, signup_date 
FROM users 
WHERE email = 'user_50000@example.com';
```

**Explanation:**

CREATE INDEX idx_users_email builds a sorted pointer structure mapped directly to the email column.

Running EXPLAIN outputs the execution plan graph without running heavy computation.

The optimizer switches the access mechanism from SEQ_SCAN (Full Table Scan) to INDEX_SCAN or SEARCH, skipping irrelevant data blocks completely.

**Important Point Related To It**

Write Overhead Penalty: Indexes are not free. Every INSERT, UPDATE, or DELETE requires updating both the base table and all attached index structures, which degrades write throughput if a table is over-indexed.

-----

**Clustered vs. Non-Clustered Indexes** 


Clustered Index: Defines the physical storage order of table data on disk. The leaf nodes of a clustered index are the actual data rows. Only one clustered index can exist per table (typically auto-created on the Primary Key).

Non-Clustered Index: A separate, standalone lookup structure containing index keys and pointers (row IDs or clustered keys) referencing the actual data rows. Multiple non-clustered indexes can exist on a single table.

**Application**

Clustered indexes optimize sequential range reads (BETWEEN, >, <) and Primary Key lookups. Non-Clustered indexes accelerate secondary queries filtering, joining, or sorting on non-primary attributes (like email, status, or created_at).

**Real-World Problem and How Particular Topic Solves**

Problem: Searching a 10-million-row users table for a specific email forces an O(N) full table scan across disk blocks, even if the table is physically sorted by user_id.

How It Solves It: A non-clustered index acts like a book index at the end of a textbook. It creates a lightweight B-Tree mapping each email directly to its physical row location on disk, turning full table scans into fast O(log N) point lookups.

**Industrial Example**

In financial ledgers, clustering the transactions table physically on transaction_id stores records sequentially by ingestion order. Non-clustered indexes are added to account_id and timestamp to serve user account history dashboards instantly.

```sql
-- Setup table schema

-- Insert sample records

-- Create explicit Non-Clustered Index on secondary lookup column

-- Execute point lookup via Non-Clustered Index

DROP TABLE IF EXISTS accounts;

CREATE TABLE accounts (
    account_id INT PRIMARY KEY, -- Clustered Index automatically created here
    email VARCHAR(100),
    balance DECIMAL(10, 2)
);

INSERT INTO accounts VALUES 
(101, 'alice@example.com', 5000.00),
(102, 'bob@example.com', 1200.00),
(103, 'charlie@example.com', 8500.00);

CREATE INDEX idx_accounts_email ON accounts(email);

SELECT account_id, balance 
FROM accounts 
WHERE email = 'bob@example.com';
```

**Explanation:**

1. Defining PRIMARY KEY builds a clustered index, physically ordering table rows on disk by account_id.

2. CREATE INDEX idx_accounts_email creates a secondary B-Tree mapping distinct email strings to their corresponding account_id pointers.

3. The WHERE email = ... query traverses the non-clustered B-Tree, reads the pointer 102, and retrieves row attributes without scanning unrelated table blocks.

**Important Point Related To It**

Covering Index & Key Lookups: If a query requests columns missing from a Non-Clustered Index, the engine must perform a "Key Lookup" back to the main clustered table. Including secondary columns inside the index creates a Covering Index, eliminating extra disk reads entirely.

---

**EXPLAIN and EXPLAIN ANALYZE**

EXPLAIN and EXPLAIN ANALYZE are database diagnostic commands used to inspect the execution strategy generated by the query optimizer:

- EXPLAIN: Displays the logical execution plan constructed by the optimizer based on catalog statistics without running the query (estimated cost, row counts, and plan nodes).

- EXPLAIN ANALYZE: Actually executes the query, providing a comparison between the optimizer's initial estimates and the actual runtime performance metrics (exact execution time, actual row counts, loop counts, and memory usage).

**Application**

- Identifying Bottlenecks: Detecting slow query operations, such as full Sequential Scans (Seq Scan) on large tables where an Index Scan was expected.

- Diagnosing Stale Statistics: Comparing estimated rows vs actual rows to spot outdated table statistics that cause the optimizer to choose poor join algorithms.

- Join Optimization: Checking whether the engine chooses a Nested Loop, Hash Join, or Merge Join, and optimizing join predicates accordingly.

- Filter Offloading: Identifying whether WHERE clause filters are applied at the index level or evaluated after fetching pages from disk (Filter: condition overhead).

**Real-World Problem and Solution**

Problem: A critical dashboard endpoint that runs a multi-table join query starts taking 12 seconds to load. Application developers cannot tell why the query degraded, as all tables have primary keys.

Solution: Running EXPLAIN ANALYZE reveals a Seq Scan on a 10-million-row orders table because the join column customer_id lacked an index. The engine was executing a Nested Loop join, reading the entire orders table repeatedly. Creating a non-clustered index on orders(customer_id) allowed the optimizer to switch to an Index Scan, dropping query execution time from 12 seconds to 4 milliseconds.

**Industrial Example**

A high-throughput API endpoint suffers from database memory spikes during peak traffic. Running EXPLAIN (ANALYZE, BUFFERS) in PostgreSQL reveals that a specific sorting query spills over from fast RAM (work_mem) to slow disk-based temporary files (Sort Method: external merge  Disk: 15420kB). Increasing work_mem or adding an index that satisfies the ORDER BY clause eliminates disk spilling.

```sql
-- Target Query to Inspect
-- 1. Get Estimated Execution Plan (Does NOT execute query)
EXPLAIN 
SELECT o.order_id, o.order_date, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2026-01-01'
  AND c.country = 'IND';

-- 2. Get Actual Execution Metrics (Executes the query!)
-- PostgreSQL Syntax:
EXPLAIN (ANALYZE, BUFFERS, COSTS, VERBOSE)
SELECT o.order_id, o.order_date, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2026-01-01'
  AND c.country = 'IND';

-- MySQL Syntax Equivalent:
-- EXPLAIN ANALYZE SELECT ... ;
```

**Important Points Related to It**

- Side Effects of ANALYZE: Running EXPLAIN ANALYZE on INSERT, UPDATE, or DELETE queries will modify data because the statement actually runs. To safely analyze write queries, wrap them in a transaction rollback block:

BEGIN;
EXPLAIN ANALYZE DELETE FROM orders WHERE status = 'CANCELLED';
ROLLBACK;

- Common Node Operations:

    - Seq Scan / Full Table Scan: Evaluates every page on disk sequentially. High cost on large tables.

    - Index Scan: Reads index B-tree nodes and navigates to heap/table pages for missing columns.

    - Index Only Scan: Satisfies query entirely from index leaf nodes without visiting main table storage (Covering Index hit).

    - Nested Loop: Iterates outer table rows and performs a lookup on inner table per row. Fast for small row sets.

    - Hash Join: Builds an in-memory hash table of the smaller relation, then probes it with the larger relation. Highly efficient for large, un-indexed joins.

- Optimizer Statistics Staleness: If plans show bad estimations, run explicit database statistics updates (ANALYZE table_name; in Postgres or ANALYZE TABLE table_name; in MySQL) to update catalog metadata

---

**Logical Query Execution Order**

The Logical Query Execution Order represents the conceptual sequence of operations that a relational database engine follows to process a SQL query. While a SQL query is written in a declarative syntax order (starting with SELECT), the database engine executes the query in a completely different logical order to properly filter, group, aggregate, and evaluate clauses.

**Application**

- Disambiguating Scope Rules: Understanding why column aliases created in the SELECT clause cannot be used in WHERE or GROUP BY clauses, but can be used in ORDER BY.

- Filter Placement Optimization: Knowing the difference between filtering before aggregation (WHERE) versus filtering after aggregation (HAVING).

- Query Debugging: Diagnosing syntax and reference errors occurring during complex aggregations, joins, and window function calculations

**Real-World Problem and Solution**

Problem: A developer attempts to write a query calculating total customer order values and filter out customers with total spending under $1,000 using the following query:

    SELECT customer_id, SUM(order_total) AS total_spend
    FROM orders
    WHERE total_spend > 1000  -- ERROR: Column "total_spend" does not exist!
    GROUP BY customer_id;

Solution: Because WHERE is evaluated before GROUP BY and SELECT, the alias total_spend does not exist yet when filtering occurs. Understanding the logical order dictates moving the aggregate filter to the HAVING clause:

    SELECT customer_id, SUM(order_total) AS total_spend
    FROM orders
    GROUP BY customer_id
    HAVING SUM(order_total) > 1000;

**Industrial Example**

An e-commerce analytics engine processes monthly user retention metrics. The query must evaluate tables in strict order: first joining users and orders (FROM / JOIN), filtering out non-paying user records (WHERE), grouping active users by region (GROUP BY), filtering out regions with low total volume (HAVING), calculating row metrics and window function ranks (SELECT), sorting results by rank (ORDER BY), and retrieving top results (LIMIT).

```sql
/* 
WRITTEN SYNTAX ORDER:
1. SELECT
2. FROM
3. JOIN / ON
4. WHERE
5. GROUP BY
6. HAVING
7. WINDOW
8. ORDER BY
9. LIMIT / OFFSET
*/

-- ACTUAL LOGICAL EXECUTION ORDER IN DATABASE ENGINE:

-- Step 1: FROM & JOINs (Identifies source tables & builds virtual working set)
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id

-- Step 2: WHERE (Filters individual rows BEFORE aggregation)
WHERE o.order_status = 'COMPLETED' 
  AND o.order_date >= '2026-01-01'

-- Step 3: GROUP BY (Collapses matching rows into aggregate groups)
GROUP BY c.customer_id, c.country

-- Step 4: HAVING (Filters aggregated groups AFTER GROUP BY)
HAVING SUM(o.order_total) >= 5000.00

-- Step 5: SELECT (Evaluates column expressions, aliases, & Window Functions)
SELECT 
    c.customer_id,
    c.country,
    SUM(o.order_total) AS total_spend,
    DENSE_RANK() OVER (PARTITION BY c.country ORDER BY SUM(o.order_total) DESC) AS country_rank

-- Step 6: DISTINCT (Deduplicates the final projections if requested)
-- DISTINCT

-- Step 7: ORDER BY (Sorts final result set; can reference SELECT aliases)
ORDER BY country_rank ASC

-- Step 8: LIMIT / OFFSET (Paginates final output rows)
LIMIT 10;
```

**Important Points Related to It**

- Clause Access Rules:

    - WHERE vs SELECT: WHERE cannot see SELECT aliases because WHERE runs in Step 2, long before SELECT executes in Step 5.

    - HAVING vs WHERE: WHERE filters individual base rows before aggregation; HAVING filters aggregated summaries after GROUP BY.

    - ORDER BY Special Privilege: ORDER BY runs in Step 7 (after SELECT), which is why it can reference aliases defined in SELECT.

- Window Functions Execution Timing: Window functions (OVER (...)) are evaluated inside Step 5 (SELECT). Consequently, you cannot use window function results in WHERE or HAVING clauses without wrapping the query inside a CTE or subquery.

- Physical vs. Logical Execution: The logical order specifies how the engine must logically compute results. Modern query optimizers may reorder physical execution steps (e.g., pushing WHERE conditions before JOIN execution) via predicate pushdown, provided the final query outcome remains identical.

---

**Query Optimization Techniques**

Query optimization is the process of tuning SQL statements, schema structures, and indexing strategies to reduce system resource consumption—specifically execution time, CPU cycles, disk I/O, and memory usage—while delivering identical query results.

**Application**

- High-Throughput APIs: Reducing response latencies for web endpoints by optimizing repetitive transactional queries.

- Batch & ETL Pipelines: Eliminating full-table scans and disk-spilling sort operations in data processing jobs to speed up runtimes.

- Resource Cost Reduction: Lowering cloud database compute costs by minimizing CPU utilization and disk read operations.

**Real-World Problem and Solution**

Problem: A critical customer reporting query takes 15 seconds to execute because it computes aggregates using non-SARGable functions in the WHERE clause, fetches unused columns (SELECT *), and forces implicit data type conversions:

-- BAD: Non-SARGable, uses SELECT *, functions on indexed columns
SELECT * 
FROM orders 
WHERE DATE_TRUNC('year', order_date) = '2026-01-01'::date
  AND CAST(customer_id AS VARCHAR) = '10045';

Solution: Refactoring the query to be SARGable (Search Argument Able) allows the optimizer to use B-Tree indexes efficiently, dropping query runtime to 3 milliseconds:

-- GOOD: SARGable ranges, explicit projections, matching types
SELECT order_id, order_total, order_date 
FROM orders 
WHERE order_date >= '2026-01-01' AND order_date < '2027-01-01'
  AND customer_id = 10045;

**Industrial Example**

A high-frequency billing platform executes millions of status checks daily. The original implementation used correlated subqueries inside a WHERE IN (...) construct. Replacing the correlated subqueries with an explicit EXISTS check or 
a indexed JOIN allowed the planner to eliminate redundant subquery re-evaluations for every outer row, saving hundreds of gigabytes of disk I/O per day.

```sql
-- Pattern 1: Non-SARGable vs. SARGable Expressions
-- BAD: Applying functions to indexed columns disables index lookups (forces Seq Scan)
SELECT user_id, email FROM users WHERE UPPER(last_name) = 'SMITH';

-- GOOD: Use functional indexes OR query raw values directly
-- Option A: Create functional index: CREATE INDEX idx_users_upper_lastname ON users(UPPER(last_name));
-- Option B: Query directly if data is normalized
SELECT user_id, email FROM users WHERE last_name = 'Smith';


-- Pattern 2: Subquery Refactoring (IN vs. EXISTS vs. JOIN)
-- BAD: IN subquery with potentially large result sets or NULL values
SELECT customer_id, company_name 
FROM customers 
WHERE customer_id IN (SELECT customer_id FROM orders WHERE order_total > 1000);

-- GOOD: Replace with EXISTS (short-circuits on first match) or Semi-Join
SELECT c.customer_id, c.company_name 
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id AND o.order_total > 1000
);


-- Pattern 3: Avoid SELECT * (Covering Index Enabler)
-- BAD: Fetches unused payload columns, forcing disk page lookups (Heap Access)
SELECT * FROM orders WHERE order_date = '2026-09-01';

-- GOOD: Project only required columns, enabling Index-Only Scans
SELECT order_id, customer_id, order_total 
FROM orders 
WHERE order_date = '2026-09-01';
```

**Code Breakdown:**

- SARGable Clauses: Predicates formatted so the database engine can jump straight to a specific B-tree leaf node (e.g., col >= value instead of FUNCTION(col) = value).

- EXISTS Short-Circuiting: Stops searching the subquery as soon as a single matching row is found, whereas IN evaluates and materializes the full subquery dataset.

- Explicit Projection (SELECT col1, col2): Decreases network payload, lowers memory usage, and allows the planner to fulfill queries entirely using Covering Indexes (Index Only Scan) without reading the main table pages.

**Important Points Related to It**

- SARGability Rule: Never wrap indexed table columns inside functions in WHERE, JOIN, or GROUP BY clauses (DATE(), SUBSTR(), CAST(), COALESCE()). Doing so hides the index from the planner and forces a full table scan.

- Implicit Data Type Conversions: Ensure application query parameters match target table column data types exactly (e.g., passing a string parameter to an integer column). Type mismatches force the engine to cast every row before evaluating equality, disabling index usage.

- Pagination Efficiency: Avoid high-offset pagination (LIMIT 100 OFFSET 1000000), which reads and discards 1 million rows. Use Seek/Keyset Pagination (WHERE id > last_seen_id ORDER BY id LIMIT 100) to leverage index lookups directly.

---

**Partitioning Large Tables**

Table partitioning is a database design technique where a single, logically unified large table is physically divided into smaller, independent underlying storage chunks called partitions. The database management system manages this division automatically based on a designated partition key and routing strategy (e.g., Range, List, Hash), allowing queries to target only relevant physical data segments.

**Application**

- Managing Multi-Terabyte Datasets: Maintaining high query performance on massive tables (e.g., logging systems, time-series metrics, transaction archives) where single B-tree indexes become too large to fit in RAM.

- Partition Pruning: Accelerating queries by allowing the query planner to bypass and ignore physical partitions that do not match query WHERE clause filters.

- Fast Data Lifecycle Management (Bulk Drops): Archiving or purging historical data instantly by dropping entire physical partitions (DROP TABLE partition_2024_q1) instead of executing slow, I/O-heavy DELETE operations.

**Real-World Problem and Solution**

- Problem: An analytics table stores 2 billion clickstream event logs (clickstream_events). A night pipeline must purge records older than 3 years to comply with data retention policies. Running DELETE FROM clickstream_events WHERE event_time < '2023-01-01'; takes hours, locks table rows, causes severe transaction log bloat, and degrades application traffic.

- Solution: Structuring clickstream_events as a Range Partitioned Table by month allows the maintenance pipeline to issue DROP TABLE clickstream_events_y2022m12;. Dropping the underlying physical partition unlinks storage metadata instantly in milliseconds, bypassing row-by-row logging entirely.

**Industrial Example**

A global payment network processes hundreds of millions of card authorizations daily. The database partitions the authorizations table by Range on transaction_date (monthly partitions) and sub-partitions by List on region_code. Queries filtering for WHERE transaction_date >= '2026-09-01' AND region_code = 'APAC' trigger Partition Pruning, reading only a tiny fraction of total cluster storage disk pages.

```sql
-- 1. Declarative Range Partitioning (PostgreSQL Syntax)
CREATE TABLE order_logs (
    log_id BIGINT GENERATED ALWAYS AS IDENTITY,
    customer_id INT NOT NULL,
    order_total DECIMAL(10, 2) NOT NULL,
    log_date DATE NOT NULL,
    PRIMARY KEY (log_id, log_date) -- Partition key MUST be part of Primary Key
) PARTITION BY RANGE (log_date);

-- 2. Creating Specific Range Child Partitions
CREATE TABLE order_logs_y2026_q1 PARTITION OF order_logs
    FOR VALUES FROM ('2026-01-01') TO ('2026-04-01');

CREATE TABLE order_logs_y2026_q2 PARTITION OF order_logs
    FOR VALUES FROM ('2026-04-01') TO ('2026-07-01');

CREATE TABLE order_logs_y2026_q3 PARTITION OF order_logs
    FOR VALUES FROM ('2026-07-01') TO ('2026-10-01');

-- 3. Partition Pruning Demonstration
-- The planner skips Q1 & Q2 partitions entirely, scanning ONLY order_logs_y2026_q3
EXPLAIN
SELECT * 
FROM order_logs 
WHERE log_date >= '2026-08-01' AND log_date < '2026-09-01';

-- 4. Fast Purging via Partition Drop
DROP TABLE order_logs_y2026_q1; -- Instant metadata drop, zero row-by-row log bloat
```

**Code Breakdown:**

- PARTITION BY RANGE (log_date): Defines the master routing key for incoming rows.

- PARTITION OF ... FOR VALUES FROM (...) TO (...): Creates discrete physical storage tables that hold specific value ranges.

- PRIMARY KEY (log_id, log_date): In partitioned setups, primary keys and unique constraints must explicitly include the partition key column to enforce uniqueness across physical partitions without cross-table index locks.

**Important Points Related to It**

- Partition Strategies:
    - **Range Partitioning:** Divides data based on continuous value ranges (such as dates, timestamps, or numerical ID intervals). Perfect for time-series and log data.

	- **List Partitioning:** Divides data by matching explicit discrete lists of values (such as country codes, status types, or region identifiers).

	- **Hash Partitioning:** Divides data by applying a hashing algorithm to a partition key column, distributing rows evenly across a predefined number of sub-tables to prevent hot-spotting.

	- **Partition Pruning:** An optimization feature where the query planner analyzes the `WHERE` clause at runtime and completely skips (prunes) scanning partitions that cannot possibly contain matching data.
    
    - Over-Partitioning Anti-Pattern: Avoid creating too many small partitions (e.g., partitioning by minute or second). Thousands of tiny partitions overhead the query planner during execution planning, consuming memory and degrading overall query throughput.

### Interview Questions

#### Question 1: How does partition pruning function at the query execution level, and what query design anti-patterns prevent the optimizer from successfully pruning partitions?

- **Solution Approach:**
    
    - _Mechanism:_ Partition pruning occurs during query planning. The optimizer evaluates predicates in the `WHERE` clause against the partition boundary metadata constraints. If a partition's declared range or list values fall entirely outside the predicate conditions, the planner drops that partition descriptor from the execution tree.
    - _Anti-Patterns:_ Wrapping partition key columns inside SQL functions or implicit type conversions (e.g., `WHERE DATE(recorded_at) = '2026-07-01'` instead of `recorded_at >= '2026-07-01' AND recorded_at < '2026-07-02'`) masks the raw column value, blinding the optimizer and forcing a full scan across all partitions.

#### Question 2: Compare Range, List, and Hash partitioning strategies, and describe a production scenario where choosing Hash partitioning is mandatory over Range partitioning.

- **Solution Approach:**
    
    - _Range:_ Best for continuous chronological or numerical data (dates, financial IDs) where queries frequently filter by date intervals.
    - _List:_ Best for categorical attributes with discrete, well-known values (regions, department codes, tenant IDs).
    - _Hash:_ Best for uniform data distribution when there is no natural range or list boundary, or when a high-velocity single-key table (e.g., `user_id` or `device_id`) suffers from severe sequential write hot-spotting on a single range partition. Hash partitioning ensures writes are pseudo-randomly distributed across equal sub-tables.

#### Question 3: What are the architectural challenges and performance trade-offs associated with maintaining unique constraints and primary keys on partitioned tables?

- **Solution Approach:**
    
    - _The Challenge:_ In relational databases, primary keys enforce uniqueness across the entire table. For partitioned tables, enforcing a global unique constraint (e.g., `PRIMARY KEY (log_id, recorded_at)`) requires the database engine to verify uniqueness across _all_ physical partitions.
    - _The Trade-Off:_ To guarantee global uniqueness, the partition key _must_ be included as part of the primary key or unique index definition. Without including the partition key in the index, the index becomes a global index spanning multiple physical files, creating heavy lock contention and slow distributed writes during partition maintenance operations.

---

**VIEWS vs. Materialized VIEWS**

A View is a saved, virtual table defined by a SELECT query that executes dynamically every time it is queried. A Materialized View physically computes and stores the query results on disk, requiring periodic refreshes to sync with base table changes.

**Application**

Used to simplify complex join operations, enforce column-level security/access control, and pre-compute heavy analytical aggregations for fast retrieval.

***Real-World Problem & How It Solves**

Problem: Running a 10-table analytical join taking 15 seconds every time an executive dashboard loads causes server latency and high compute costs.

How It Solves It: A Materialized View executes the heavy query once and writes the result dataset to disk, serving subsequent dashboard requests instantly in milliseconds (O(1)read time).

**Industrial Example**

In financial platforms, exposing sensitive user tables to reporting teams through restricted Views that hide password hashes, social security numbers, and internal tokens.

```sql
-- Setup base table

-- 1. Standard Virtual View

-- Query the virtual view directly

DROP TABLE IF EXISTS orders;

CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    total_amount DECIMAL(10, 2),
    status VARCHAR(20)
);

INSERT INTO orders VALUES 
(1, 101, 250.00, 'COMPLETED'),
(2, 102, 400.00, 'PENDING'),
(3, 101, 150.00, 'COMPLETED');

CREATE VIEW v_completed_orders AS
SELECT customer_id, total_amount
FROM orders
WHERE status = 'COMPLETED';

SELECT * FROM v_completed_orders;
```

**Explanation:**

1. CREATE VIEW saves the query definition under v_completed_orders without duplicating data on disk.

2. Querying v_completed_orders automatically executes the underlying SELECT statement against orders behind the scenes.

**Important Point Related** 

Trade-off (Freshness vs. Speed): Views guarantee real-time data freshness but rerun underlying computations every execution. Materialized Views offer ultra-fast read speeds but require explicit refresh commands (REFRESH MATERIALIZED VIEW) to capture modified base table data.

---

**Database Transactions & ACID Properties**

A transaction is a logical unit of work consisting of one or more SQL statements executed as a single, atomic operation controlled by BEGIN, COMMIT, and ROLLBACK.

**Application**

Used to maintain strict data integrity during multi-step write, update, or deletion operations where partial completion would corruption state.

**Real-World Problem & How It Solves It**

Problem: Transferring $100 from Account A to Account B requires two operations: deducting $100 from A and adding $100 to B. If the database crashes after step 1, $100 vanishes into thin air.

How It Solves It: Wrapping both updates inside a transaction ensures that if step 2 fails, ROLLBACK undoes step 1 entirely, restoring the original balance.

**Industrial Example**

In banking systems, processing wire transfers, order checkout carts, and inventory reservations under strict ACID guarantees.

**ACID Breakdown**

- Atomicity: "All or nothing." Either every statement succeeds or the entire transaction reverts.

- Consistency: Data must validly satisfy all constraints, foreign keys, and rules before and after execution.

- Isolation: Concurrent transactions execute without interfering with or leaking uncommitted data to each other.

- Durability: Once committed, changes are permanently saved on non-volatile disk storage, surviving power failures or crashes.

```sql
-- Setup accounts table

-- Execute atomic money transfer transaction

-- # Step 1: Deduct from Account 1

-- # Step 2: Add to Account 2

-- # Commit changes if both steps succeed

-- # Rollback everything if any error occurs

-- Inspect updated balances

DROP TABLE IF EXISTS accounts;

CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance DECIMAL(10, 2)
);

INSERT INTO accounts VALUES (1, 500.00), (2, 200.00);

BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100.00 WHERE account_id = 1;

UPDATE accounts SET balance = balance + 100.00 WHERE account_id = 2;

COMMIT;

ROLLBACK;

SELECT * FROM accounts;
```

**Explanation:**

1. BEGIN TRANSACTION marks the boundary of the isolated work block.

2. If both UPDATE queries complete without raising an exception, COMMIT permanently persists changes to disk.

3. If an error occurs midway, ROLLBACK wipes all uncommitted mutations made since BEGIN TRANSACTION.

---

### Isolation Levels & Concurrency Anomalies

Isolation is the third property of ACID transactions ($I$). It defines the degree to which concurrent transactions are shielded from seeing uncommitted or intermediate modifications made by other simultaneous queries. Without isolation, concurrent database access leads to data corruption, race conditions, and unpredictable application states.

#### Application

- **Preventing Data Corruption:** Ensuring that simultaneous financial transfers, inventory deductions, or account updates do not overwrite each other's changes.

- **Balancing Consistency vs. Concurrency:** Choosing the correct isolation tier to prevent read/write anomalies while maximizing the number of simultaneous transactions a database engine can process.

#### Real-World Problem and Solution

- **Problem:** Two separate banking representatives simultaneously check a customer's account balance of $500. Representative A applies a $100 debit, while Representative B applies a $50 debit. If both transactions read the initial snapshot of $500 concurrently without proper isolation, Representative A writes $400 ($500 - $100) and Representative B writes $450 ($500 - $50), completely overwriting Representative A's debit. This phenomenon is known as a **Lost Update**, causing the financial institution to lose track of the $100 deduction.

- **Solution:** Enforcing proper transaction isolation ensures that concurrent write transactions either lock conflicting records or abort when modifications collide, preserving all updates accurately.

#### Industrial Example

An e-commerce flash sale processes thousands of concurrent checkout requests for a limited stock item. Strict transaction isolation ensures that inventory decrement counters evaluate sequentially under heavy contention, preventing overselling past inventory limits.

#### Concurrency Anomalies Explained

- **Dirty Read:** Transaction A modifies a row without committing. Transaction B reads that uncommitted change. If Transaction A subsequently rolls back, Transaction B has acted on data that never officially existed.

- **Non-Repeatable Read:** Transaction A reads a row. Transaction B updates or deletes that row and commits. If Transaction A reads the exact same row again within the same transaction, we notice modified data or missing records.

- **Phantom Read:** Transaction A queries a range of rows matching a condition (e.g., `WHERE total > 100`). Transaction B inserts a new row that matches Transaction A's search criteria and commits. If Transaction A re-runs the range query, new "phantom" records appear.

- **Lost Update:** Two concurrent transactions read the same base value, compute updates independently, and write back changes, causing one transaction's modifications to be silently overwritten by the other.

- **Write Skew:** Two transactions read overlapping datasets, make independent decisions based on those reads, and modify disjoint subsets of data such that a global business constraint is violated.

#### Isolation Levels and Anomaly Prevention Matrix

| **Isolation Level**  | **Dirty Read** | **Non-Repeatable Read** | **Phantom Read** | **Lost Update** |
| -------------------- | -------------- | ----------------------- | ---------------- | --------------- |
| **READ UNCOMMITTED** | Possible       | Possible                | Possible         | Possible        |
| **READ COMMITTED**   | Prevented      | Possible                | Possible         | Possible        |
| **REPEATABLE READ**  | Prevented      | Prevented               | Possible*        | Prevented       |
| **SERIALIZABLE**     | Prevented      | Prevented               | Prevented        | Prevented       |

*Note: Some database engines like MySQL InnoDB prevent phantom reads even at the REPEATABLE READ level using next-key locking.

#### Default Isolation Levels Across Databases

- **PostgreSQL:** `READ COMMITTED`

- **MySQL (InnoDB):** `REPEATABLE READ`

- **SQL Server:** `READ COMMITTED` (supports Snapshot Isolation when configured)

#### Code and Explanation

```sql
-- 1. Setting Transaction Isolation Level explicitly
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Execute transactional read/write workloads here...
SELECT balance FROM accounts WHERE account_id = 1;

COMMIT;
```

- `SET TRANSACTION ISOLATION LEVEL`: Instructs the database engine to enforce specific concurrency rules for the active session or transaction block.

---

#### MVCC (Multi-Version Concurrency Control), Snapshot Isolation, and Demo

**MVCC (Multi-Version Concurrency Control)** is an architectural concurrency model used by modern database engines (like PostgreSQL, Oracle, and MySQL InnoDB) to avoid read-write locking conflicts. Instead of locking data rows during read operations, MVCC maintains multiple physical versions (or tuples) of a single logical row.

**Snapshot Isolation:** A specific implementation of MVCC where a transaction reads a consistent point-in-time snapshot of the database as it existed at the exact moment the transaction began, entirely ignoring subsequent uncommitted or committed modifications made by concurrent transactions.

#### Application

- **High-Concurrency Reads:** Allowing long-running analytical read queries to execute concurrently alongside rapid write operations without blocking each other.

- **Eliminating Read Locks:** Preventing readers from blocking writers and writers from blocking readers, drastically improving database throughput.

#### Real-World Problem and Solution

- **Problem:** In legacy locking databases, if an analytical report queries the `orders` table, it acquires shared read locks, blocking all incoming checkout transactions until the report finishes. This causes application timeouts during peak hours.

- **Solution:** MVCC creates a private version snapshot for the reporting transaction. The reporting query reads older row versions directly from storage data pages without acquiring locks, while concurrent checkout transactions write new row versions simultaneously without obstruction.

#### Industrial Example

An e-commerce order tracking dashboard queries real-time item status metrics while warehouse workers update package statuses concurrently. Thanks to MVCC snapshot isolation, neither workflow blocks the other, ensuring zero dashboard lag and zero checkout locks.

#### Code and Demonstration: Reproducing a Non-Repeatable Read

```sql
-- SESSION 1: Begins transaction under READ COMMITTED isolation
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Step 1: Read initial balance
SELECT balance FROM accounts WHERE account_id = 1; 
-- Returns: 500.00

---------------------------------------------------------
-- SESSION 2 (Concurrently executed in separate window):
BEGIN TRANSACTION;
UPDATE accounts SET balance = 750.00 WHERE account_id = 1;
COMMIT; -- Session 2 commits changes successfully!
---------------------------------------------------------

-- Step 2: Re-read the exact same row in Session 1
SELECT balance FROM accounts WHERE account_id = 1; 
-- Returns: 750.00 (Value changed! This is a Non-Repeatable Read)

COMMIT;
```

- **Session 1** reads a baseline value (`500.00`).

- **Session 2** updates that row and commits.

- Under `READ COMMITTED`, **Session 1** re-reads the row and sees the newly committed value (`750.00`), demonstrating a **Non-Repeatable Read** because two identical queries within the same transaction returned different results.
#### Interview Questions & Answers

- **Q1: How does MVCC work under the hood?**
    
    - _Answer:_ MVCC stores system-identifier metadata columns on every physical row—specifically `xmin` (the transaction ID that created the row version) and `xmax` (the transaction ID that deleted or superseded the row version). When a transaction reads data, it evaluates these transaction IDs against its own transaction snapshot visibility rules, seeing only rows committed before it started.
    
- **Q2: Does MVCC eliminate all locks?**
    
    - _Answer:_ No. MVCC eliminates locks between **readers and writers**, but concurrent **writers still lock each other** (exclusive write locks) on the same rows to prevent lost updates and race conditions.
    
- **Q3: What is "Bloat" in MVCC databases like PostgreSQL, and how is it resolved?**
    
    - _Answer:_ Because MVCC retains old row versions on disk until no active transactions need them, frequent updates and deletes cause **table and index bloat** (wasted storage disk space). This is cleaned up automatically or manually via the background **VACUUM** process.

---

### Locking & Deadlocks (`SELECT ... FOR UPDATE`, `SKIP LOCKED`, and Deadlock Resolution)

#### Definition

While MVCC handles read-write concurrency without blocking, database **locks** are explicit synchronization primitives used to arbitrate conflicting write operations and prevent data corruption.

- **Shared (S) Locks:** Permissive locks acquired during read operations. Multiple transactions can hold Shared locks on the same resource simultaneously.

- **Exclusive (X) Locks:** Restrictive locks acquired during write operations (`INSERT`, `UPDATE`, `DELETE`). Only one transaction can hold an Exclusive lock on a specific row or table, blocking all other conflicting read or write attempts.

- **Row-Level vs. Table-Level Locks:** Row-level locks target fine-grained individual records for maximum concurrency, whereas table-level locks restrict entire table structures (typically during schema alterations or bulk DDL operations).

#### Application and Advanced Locking Clauses

- **`SELECT ... FOR UPDATE`:** Explicitly acquires exclusive row-level locks on selected rows during a read operation, preventing concurrent transactions from updating or deleting those specific rows until our transaction commits.

- **`FOR SHARE`:** Acquires shared locks on selected rows, allowing concurrent reads but blocking external updates until our transaction releases the lock.

- **`NOWAIT`:** Instructs the database engine to fail immediately with an error rather than waiting in line if a target row is already locked by another transaction.

- **`SKIP LOCKED`:** Essential for high-concurrency **Job Queue patterns**. It tells the database to skip over rows currently locked by other workers instead of waiting, allowing multiple background workers to pull independent pending tasks from a queue simultaneously without contention.

- **Optimistic Locking:** A concurrency pattern that avoids database locks entirely. Instead, we include a `version` integer column in our table; when updating a row, we check that `version` has not changed since we read it, aborting the write if a conflict is detected.

#### Real-World Problem and Solution

- **Problem:** A background job queue worker processes pending email notifications from a `jobs` table. If multiple worker threads query `SELECT * FROM jobs WHERE status = 'PENDING' LIMIT 1`, they often grab the exact same job row concurrently, resulting in duplicate processing and race conditions.
    
- **Solution:** Using `SELECT * FROM jobs WHERE status = 'PENDING' LIMIT 1 FOR UPDATE SKIP LOCKED;` ensures that each worker locks and pulls a distinct, un-claimed job row instantly, bypassing locked rows and eliminating queue contention entirely.

#### Industrial Example

An enterprise flight ticket booking platform uses `SELECT ... FOR UPDATE` when a customer initiates seat selection. This locks the specific seat row immediately so that no other user can book the same seat during checkout payment processing.

#### Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE job_queue (
    job_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    task_name VARCHAR(100) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING'
);

-- 1. Job Queue Pattern using SKIP LOCKED (Zero-wait concurrency)
BEGIN TRANSACTION;

SELECT job_id, task_name 
FROM job_queue 
WHERE status = 'PENDING'
LIMIT 1 
FOR UPDATE SKIP LOCKED;

-- Update the grabbed job status to 'PROCESSING'
UPDATE job_queue 
SET status = 'PROCESSING' 
WHERE job_id = 1; -- (Using the grabbed ID)

COMMIT;

-- 2. Strict Locking with NOWAIT (Fails instantly if locked)
BEGIN TRANSACTION;
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE NOWAIT;
COMMIT;
```

- **`FOR UPDATE`:** Places an exclusive lock on the selected row tuples.

- **`SKIP LOCKED`:** Excludes locked records from the result set dynamically, making it ideal for distributed worker pools.

- **`NOWAIT`:** Replaces the default blocking behavior (which waits indefinitely for locks to release) with an immediate exception throw if a lock conflict is encountered.

#### Blocking vs. Deadlocks and Resolution

- **Blocking:** Occurs naturally when Transaction A holds a lock on a resource and Transaction B is forced to wait in line until Transaction A commits or rolls back. Blocking is normal and healthy in transactional systems.
    
- **Deadlock:** A catastrophic circular dependency where **Transaction A** holds a lock on Resource 1 and waits for Resource 2, while **Transaction B** holds a lock on Resource 2 and waits for Resource 1. Neither transaction can proceed.
    
- **Detection & Victim Selection:** Database engines run background deadlock detector threads that analyze lock graphs. Once a circular dependency is detected, the engine automatically selects one transaction as the **victim**, rolls back its uncommitted changes, and throws a deadlock error, allowing the surviving transaction to complete successfully.

**Prevention Strategies:**

1. **Consistent Lock Order:** Always access multi-table resources in the exact same logical sequence across all application code paths.

2. **Short Transactions:** Keep transaction blocks as short as possible to minimize lock holding times.

3. **Proper Indexing:** Ensure foreign keys and lookup columns are indexed to prevent unnecessary table-level or range-level lock escalations.

4. **Lock Timeouts:** Configure `lock_timeout` parameters so transactions fail gracefully rather than hanging indefinitely.

#### Demonstration: Creating and Resolving a Deadlock

```sql
-- SESSION 1:
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 1; -- Locks Account 1

-----------------------------------------------------------------
-- SESSION 2 (Concurrently executed in separate window):
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 2; -- Locks Account 2
-----------------------------------------------------------------

-- SESSION 1 tries to access Account 2 (Blocked waiting for Session 2)
UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;

-----------------------------------------------------------------
-- SESSION 2 tries to access Account 1 (Blocked waiting for Session 1)
-- DEADLOCK DETECTED! Database aborts one session automatically.
UPDATE accounts SET balance = balance + 50 WHERE account_id = 1;
-----------------------------------------------------------------
```

#### Interview Questions & Answers

- **Q1: What is the difference between Optimistic Locking and Pessimistic Locking?**
    
    - _Answer:_ **Pessimistic locking** (`SELECT ... FOR UPDATE`) assumes conflicts are frequent and locks records immediately when read. **Optimistic locking** assumes conflicts are rare, uses no database locks during reads, and checks a `version` column upon write to detect if another transaction modified the data in the meantime.
    
- **Q2: How does `SKIP LOCKED` improve queue performance compared to standard queries?**

	- _Answer:_ Standard queries without `SKIP LOCKED` cause worker threads to block each other waiting for locked rows, creating serialization bottlenecks. `SKIP LOCKED` allows parallel workers to bypass locked tasks instantly, scaling queue processing linearly across multiple threads.
	
- **Q3: How can we prevent deadlocks in our application architecture?**

	- _Answer:_ We prevent deadlocks by establishing strict ordering rules for multi-resource acquisitions, keeping transactions small, adding indexes to avoid heavy lock escalations, and implementing application-level retry logic when deadlock exceptions occur.

----

### SAVEPOINT

#### Definition

A `SAVEPOINT` is a structural sub-division marker set inside an active database transaction. It establishes a logical recovery checkpoint, allowing developers to partially roll back a transaction to a specific point without discarding all work completed earlier in the same transaction block.

#### Application

- **Complex Batch Processing:** Executing a series of independent insert or update steps within a single transaction where the failure of one minor step should not abort the entire batch.

- **Conditional Error Recovery:** Gracefully handling optional row insertions or secondary data writes without losing primary transactional progress.

#### Real-World Problem and Solution

- **Problem:** An ETL script processes a batch of 10 customer records inside a single transaction. Records 1 through 5 insert successfully, but Record 6 triggers a check constraint violation due to malformed data. Under standard transactional rules, an error aborts the entire transaction, rolling back records 1 through 5 and wasting valid progress.

- **Solution:** Setting a `SAVEPOINT` before each individual record insertion allows the script to catch errors per record, execute a `ROLLBACK TO SAVEPOINT` for the failed record only, and continue processing records 7 through 10 successfully.

#### Industrial Example

An enterprise order processing pipeline handles multi-item checkouts. When adding optional warranty add-ons to an order, if the warranty inventory table throws an exception, the pipeline rolls back only to the warranty savepoint, preserving the primary product purchase transaction.

#### Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE batch_logs (
    log_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    message VARCHAR(255) NOT NULL
);

-- 1. Standard Transaction with SAVEPOINT (PostgreSQL, MySQL, Oracle)
BEGIN TRANSACTION;

INSERT INTO batch_logs (message) VALUES ('Step 1: Initialized batch pipeline');

-- Set a named savepoint
SAVEPOINT sp_step1_complete;

INSERT INTO batch_logs (message) VALUES ('Step 2: Processing core records');

-- Simulate an issue or conditional failure requiring partial rollback
-- ROLLBACK TO SAVEPOINT sp_step1_complete;

-- Release savepoint memory resources when no longer needed
RELEASE SAVEPOINT sp_step1_complete;

INSERT INTO batch_logs (message) VALUES ('Step 3: Finalizing transaction');

COMMIT;
```

SQL Server uses SAVE TRANSACTION instead of SAVEPOINT:

- **`SAVEPOINT name`:** Places a marker inside the current transaction log stream.

- **`ROLLBACK TO SAVEPOINT name`:** Reverts all database changes made _after_ the specified savepoint while keeping modifications made _before_ it intact.

- **`RELEASE SAVEPOINT name`:** Destroys the savepoint marker, freeing up internal transaction state memory. (Note: Committing or rolling back the parent transaction destroys all savepoints automatically).

#### Important Points Related to It

- **Not a Nested Transaction:** Savepoints are **not** independent nested transactions. If the parent transaction rolls back (`ROLLBACK`), everything—including all work protected by savepoints—is undone.

- **Dialect Differences:**
    
    - **PostgreSQL / Oracle / MySQL / SQLite:** Uses `SAVEPOINT name` and `ROLLBACK TO SAVEPOINT name`.
    
    - **SQL Server:** Uses `SAVE TRANSACTION name` and `ROLLBACK TRANSACTION name`.

#### Interview Questions & Answers

- **Q1: Does rolling back to a savepoint automatically commit the preceding operations?**
    
    - _Answer:_ No. Rolling back to a savepoint only undoes changes made _after_ that savepoint within the active transaction. The entire transaction remains uncommitted ntil an explicit `COMMIT` statement is executed.
    
- **Q2: Can we roll back to a savepoint after the parent transaction has already been committed?**
    
    - _Answer:_ No. Committing or rolling back a transaction destroys all savepoints and releases their associated resources automatically.

---

### Autocommit, Updatable Views & Materialized Views 

#### Autocommit vs. Explicit Transactions

- **Autocommit Mode:** The default behavior in most relational databases where every single SQL statement (`INSERT`, `UPDATE`, `DELETE`) is automatically treated as an independent, fully committed transaction.
    
- **Explicit Transactions:** Initiated using `BEGIN` or `START TRANSACTION`, requiring an explicit `COMMIT` to persist changes or `ROLLBACK` to discard them.

#### Updatable Views and `WITH CHECK OPTION`

- **Updatable Views:** Simple views that permit `INSERT`, `UPDATE`, or `DELETE` operations directly against underlying base tables through the view interface.
    
- **`WITH CHECK OPTION`:** A constraint applied to a view definition that prevents updates or inserts if the resulting data rows violate the view's own `WHERE` clause filter criteria.

```sql
CREATE VIEW v_active_users AS
SELECT user_id, email, status
FROM users
WHERE status = 'ACTIVE'
WITH CHECK OPTION;

-- This INSERT will fail because status is 'PENDING', violating the view filter rule!
INSERT INTO v_active_users (user_id, email, status) 
VALUES (101, 'test@example.com', 'PENDING');
```

#### Materialized View Maintenance (PostgreSQL/Oracle Style)

**`REFRESH MATERIALIZED VIEW CONCURRENTLY`:** Updates a materialized view in the background without locking read access to the view, provided the materialized view has a unique index defined on it.

```sql
-- Unique index required for CONCURRENTLY refreshing in PostgreSQL
CREATE UNIQUE INDEX idx_mv_summary_id ON mv_daily_sales(store_id);

-- Refresh without blocking readers
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```

---

# ==🔵Module 15: Statistical & Analytical SQL==

### PERCENTILE_CONT and PERCENTILE_DISC (Percentiles, Medians, and Quartiles)

#### Definition

`PERCENTILE_CONT` and `PERCENTILE_DISC` are analytical window/aggregate functions used to calculate statistical percentiles, medians, and quartiles across a dataset:

- **`PERCENTILE_CONT` (Continuous):** Calculates the exact percentile by interpolating between adjacent data values if the target percentile falls between two physical rows.

- **`PERCENTILE_DISC` (Discrete):** Returns the actual physical value from the dataset whose cumulative distribution matches or exceeds the requested percentile threshold.

- **`WITHIN GROUP (ORDER BY ...)`:** The mandatory clause used with these functions to sort the dataset before computing the percentile rank.

#### Application

- **SLA & Latency Monitoring:** Computing p95, p99, or p99.9 response times for API gateways or server logs.

- **Financial & Salary Analytics:** Finding median household incomes, salary distributions, or customer spending quartiles.

- **Outlier Detection:** Calculating Interquartile Ranges (IQR) to identify anomalous values or fraudulent spikes.

#### Real-World Problem and How Topic Solves It

- **Problem:** An analytics team needs to find the median delivery time for e-commerce orders. Using the standard `AVG()` function is misleading because a few delayed shipments skew the average upward, failing to represent typical customer experiences.

- **Solution:** Using `PERCENTILE_CONT(0.5)` calculates the exact median (50th percentile) value, providing an accurate representation of central tendency unaffected by extreme high or low outliers.

#### Industrial Example

A cloud infrastructure provider tracks API request latency. To meet Service Level Agreements (SLAs), the engineering team runs weekly SQL reports using `PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY latency_ms)` partitioned by service region to monitor p99 latency spikes.

#### Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE order_deliveries (
    order_id INT PRIMARY KEY,
    region VARCHAR(50),
    delivery_days DECIMAL(5, 2)
);

INSERT INTO order_deliveries VALUES 
(1, 'North', 2.0), (2, 'North', 3.5), (3, 'North', 4.0),
(4, 'South', 1.5), (5, 'South', 2.0), (6, 'South', 9.0);

-- 1. Calculating Median and Quartiles using PERCENTILE_CONT and PERCENTILE_DISC
SELECT 
    region,
    -- Continuous Median (interpolated if necessary)
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY delivery_days) AS median_cont,
    
    -- Discrete Median (picks actual value from the dataset)
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY delivery_days) AS median_disc,
    
    -- First Quartile (Q1 - 25th percentile)
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY delivery_days) AS q1_val,
    
    -- Third Quartile (Q3 - 75th percentile)
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY delivery_days) AS q3_val
FROM order_deliveries
GROUP BY region;
```

#### Code Breakdown:

- **`PERCENTILE_CONT(0.5)`:** Requests the 50th percentile (median). If the dataset has an even number of rows, it interpolates the average between the two middle values.

- **`WITHIN GROUP (ORDER BY delivery_days)`:** Sorts the underlying rows in ascending order before the percentile calculation is applied.

- **`GROUP BY region`:** Computes regional statistical benchmarks independently for each geographic territory.

#### Interview Questions

- **Q1: What is the primary difference between `PERCENTILE_CONT` and `PERCENTILE_DISC`?**
    
    - _Answer:_ `PERCENTILE_CONT` uses linear interpolation to calculate a precise decimal value between two adjacent rows if the target percentile does not land on an exact row. `PERCENTILE_DISC` restricts its output strictly to an actual value present in the source dataset.
        
- **Q2: Can we use `PERCENTILE_CONT` as a window function with `OVER()`?**
    
    - _Answer:_ Yes. In modern database engines (like PostgreSQL, Snowflake, and BigQuery), we can write `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY col) OVER (PARTITION BY category)` to calculate rolling or partitioned medians while retaining row-level details.

```sql
-- 1. Setup Table and Sample Data
CREATE TABLE employee_salaries (
    emp_id INT,
    emp_name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
);

INSERT INTO employee_salaries VALUES
(1, 'Amit', 'Data Science', 95000.00),
(2, 'Neha', 'Engineering', 110000.00),
(3, 'Ananya', 'Marketing', 70000.00),
(4, 'Rohan', 'Data Science', 80000.00),
(5, 'Pooja', 'Engineering', 115000.00),
(6, 'Vikram', 'Engineering', 105000.00),
(7, 'Sita', 'Data Science', 105000.00);

-- 2. Execute Query with Windowed PERCENTILE_CONT
SELECT 
    emp_name,
    department,
    salary,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER (PARTITION BY department) AS dept_median_salary
FROM employee_salaries;
```

| **emp_name** | **department** | **salary** | **dept_median_salary** |
| ------------ | -------------- | ---------- | ---------------------- |
| Rohan        | Data Science   | 80000.00   | 95000.00               |
| Amit         | Data Science   | 95000.00   | 95000.00               |
| Sita         | Data Science   | 105000.00  | 95000.00               |
| Neha         | Engineering    | 110000.00  | 110000.00              |
| Vikram       | Engineering    | 105000.00  | 110000.00              |
| Pooja        | Engineering    | 115000.00  | 110000.00              |
| Ananya       | Marketing      | 70000.00   | 70000.00               |

---

### `STDDEV` / `VARIANCE` / `CORR` (Outlier Detection & Z-Scores)

Statistical functions used to measure data spread, variability, and linear correlation across distributions:

- **`STDDEV_POP` / `STDDEV_SAMP`:** Calculates the population standard deviation vs. sample standard deviation, measuring how widely values deviate from the mean.

- **`VAR_POP` / `VAR_SAMP`:** Calculates population and sample variance (the squared standard deviation).

- **`CORR`:** Measures the statistical correlation coefficient between two numeric columns.

#### Application

- **Anomaly & Fraud Detection:** Flagging transactions or metrics that deviate excessively from baseline patterns.

- **Risk Management:** Measuring market volatility or system performance standard errors.

- **Feature Engineering:** Normalizing variables using **Z-scores** (`(value - mean) / stddev`) for machine learning models or statistical control limits.

#### Real-World Problem and How Topic Solves It

- **Problem:** An e-commerce platform detects a sudden surge in order amounts, but cannot easily tell if it is normal holiday traffic or fraudulent activity because average order values vary widely across different product categories.

- **Solution:** Calculating the **Z-score** for each transaction using population standard deviation (`STDDEV_POP`) identifies transactions that sit more than 3 standard deviations away from the category mean, isolating genuine anomalies automatically.

#### Industrial Example

A cloud infrastructure telemetry pipeline monitors server CPU loads. Using `STDDEV_SAMP` grouped by server cluster, the system flags instances where current CPU utilization exceeds $\mu + 2\sigma$ (two standard deviations above average), triggering automated scaling alerts.

#### Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE server_metrics (
    server_id INT,
    metric_hour INT,
    cpu_load_pct DECIMAL(5, 2)
);

INSERT INTO server_metrics VALUES 
(1, 1, 45.0), (1, 2, 47.0), (1, 3, 46.5), (1, 4, 98.0), -- 98.0 is an anomaly
(2, 1, 20.0), (2, 2, 22.0), (2, 3, 21.5), (2, 4, 23.0);

-- 1. Calculating Standard Deviation, Variance, and Z-Scores
WITH ServerStats AS (
    SELECT 
        server_id,
        metric_hour,
        cpu_load_pct,
        AVG(cpu_load_pct) OVER (PARTITION BY server_id) as mean_load,
        STDDEV_SAMP(cpu_load_pct) OVER (PARTITION BY server_id) as stddev_load
    FROM server_metrics
)
SELECT 
    server_id,
    metric_hour,
    cpu_load_pct,
    -- Calculate Z-Score: (Value - Mean) / Standard Deviation
    CASE 
        WHEN stddev_load = 0 THEN 0 
        ELSE (cpu_load_pct - mean_load) / stddev_load 
    END AS z_score
FROM ServerStats;
```

- **`STDDEV_SAMP(...) OVER (PARTITION BY server_id)`:** Computes sample standard deviation independently for each server's historical load logs without collapsing the original rows.

- **`Z-Score Calculation`:** Measures how many standard deviations a specific hourly CPU load sits away from that server's normal operating average. A Z-score greater than 2 or 3 typically indicates a performance anomaly.

#### Interview Questions - 

- **Q1: What is the difference between `STDDEV_POP` and `STDDEV_SAMP`?**
    
    - _Answer:_ `STDDEV_POP` computes the population standard deviation dividing by $N$ (total items), whereas `STDDEV_SAMP` computes the sample standard deviation dividing by $N - 1$ (Bessel's correction) to account for sample bias.
        
- **Q2: How do we use standard deviation to spot anomalies in SQL?**
    
    - _Answer:_ We calculate the mean ($\mu$) and standard deviation ($\sigma$) per group, compute the Z-score for each record, and filter rows where the absolute Z-score exceeds a statistical threshold (such as $\vert{}Z\vert{} > 3$).

---
## Cohort, Retention & Funnel Queries

- **Cohort Analysis:** Grouping users or entities based on a shared characteristic or initial timeline (such as their sign-up month or first purchase date) and tracking their behavior over subsequent time periods.

- **Retention Analysis:** Measuring the percentage of users from a specific cohort who return to perform key actions (like logging in or making a purchase) across consecutive periods (Period 0, Period 1, Period 2...).

- **Funnel Analysis:** Tracking user progression through a sequential series of steps (e.g., View Product $\rightarrow$ Add to Cart $\rightarrow$ Checkout $\rightarrow$ Purchase) to measure conversion rates and identify drop-off points.

### Application

- **Product Health Tracking:** Evaluating whether new user cohorts retain better or worse than previous product releases.

- **Marketing ROI Optimization:** Identifying which acquisition channels yield users with long-term retention versus short-lived engagement.

- **Conversion Rate Optimization (CRO):** Pinpointing exact friction points in multi-step user workflows where users abandon the process.

### Real-World Problem and How Topic Solves It

- **Problem:** An e-commerce platform notices total monthly active users remain flat, but management cannot tell if the business is successfully retaining older customers or just frantically replacing high churn with expensive new sign-ups.

- **Solution:** Grouping customers into monthly signup cohorts and tracking their retention percentages over time separates true product stickiness from raw acquisition growth, exposing actual user lifecycle retention trends.

### Industrial Example

A subscription-based SaaS platform tracks monthly user retention. By extracting each user's signup month (`DATE_TRUNC('month', signup_date)`) as their cohort anchor and joining it against subsequent activity logs, the analytics team builds a retention matrix heatmap to monitor churn rate trends.

**QUERY**

``` sql
-- Target Schema Setup
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    signup_date DATE
);

CREATE TABLE user_events (
    event_id INT PRIMARY KEY,
    user_id INT,
    event_date DATE
);

-- 1. Cohort & Retention Matrix Query
WITH UserCohorts AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
ActivityPeriods AS (
    SELECT DISTINCT
        u.user_id,
        u.cohort_month,
        DATE_TRUNC('month', e.event_date) AS activity_month,
        -- Calculate month offset (Period 0, 1, 2...)
        (EXTRACT(YEAR FROM e.event_date) - EXTRACT(YEAR FROM u.signup_date)) * 12 +
        (EXTRACT(MONTH FROM e.event_date) - EXTRACT(MONTH FROM u.signup_date)) AS period_number
    FROM UserCohorts u
    JOIN user_events e ON u.user_id = e.user_id
)
SELECT 
    cohort_month,
    period_number,
    COUNT(DISTINCT user_id) AS active_users
FROM ActivityPeriods
GROUP BY cohort_month, period_number
ORDER BY cohort_month, period_number;


-- 2. Conversion Funnel Query using Conditional Aggregation
CREATE TABLE funnel_logs (
    user_id INT,
    step_name VARCHAR(50),
    event_time TIMESTAMP
);

SELECT 
    COUNT(DISTINCT user_id) AS total_visitors,
    COUNT(DISTINCT CASE WHEN step_name = 'VIEW_PRODUCT' THEN user_id END) AS viewed_product,
    COUNT(DISTINCT CASE WHEN step_name = 'ADD_TO_CART' THEN user_id END) AS added_to_cart,
    COUNT(DISTINCT CASE WHEN step_name = 'PURCHASE' THEN user_id END) AS completed_purchase,
    -- Conversion Rate Calculation
    ROUND(
        COUNT(DISTINCT CASE WHEN step_name = 'PURCHASE' THEN user_id END) * 100.0 / 
        NULLIF(COUNT(DISTINCT user_id), 0), 2
    ) AS overall_conversion_pct
FROM funnel_logs;
```

- `DATE_TRUNC('month', signup_date)`: Converts a date to the beginning of its month. Example: `'2025-01-15'` → `2025-01-01 00:00:00`.

- `EXTRACT(YEAR FROM event_date)`: Extracts the year from a date. Example: `'2025-03-10'` → `2025`.

- `UserCohorts` assigns each user a cohort based on their signup month using `DATE_TRUNC()`.

- `ActivityPeriods` joins users with their events and calculates the month offset using `EXTRACT()`.

- `COUNT(DISTINCT user_id)` counts unique active users in each cohort and period.

- `GROUP BY` and `ORDER BY` organize the results by cohort month and period number.

**INTERVIEW QUESTION**

- **Q1: What is the difference between Period 0 and Period 1 in a retention matrix?**
    - _Answer:_ **Period 0** represents the initial baseline period when the user signed up or made their first purchase (typically yielding 100% retention). **Period 1** represents the very first subsequent cycle (such as month 1 or week 1) where returning activity is measured.

- **Q2: How do we calculate drop-off conversion rates across multi-step funnels in SQL?**
    - _Answer:_ We use conditional aggregation (`COUNT(DISTINCT CASE WHEN step = 'X' THEN user_id END)`) across sequential funnel stages, allowing us to evaluate all step counts in a single pass and compute percentage conversions relative to prior steps.

---

## Time-Series Patterns (`DATE_TRUNC`, `generate_series`, and Period-over-Period Growth)

Time-series patterns involve manipulating temporal data to align events into consistent chronological intervals, fill missing calendar gaps, compute running totals, and calculate growth velocities across comparative periods:

- **`DATE_TRUNC`:** Truncates a timestamp down to a specified precision unit (e.g., `'month'`, `'week'`, `'day'`).

- **`generate_series`:** A set-returning function used to generate continuous sequences of dates or numbers over a specified range.

- **Period-over-Period (PoP):** Comparing metrics across relative historical windows (such as Year-over-Year or Month-over-Month growth) using window offset functions like `LAG`.

### Application

- **Filling Temporal Gaps:** Ensuring charts and dashboards do not distort trends when specific days or months record zero transactions.

- **Financial & Operational Auditing:** Calculating Year-to-Date (YTD) running totals, moving averages, and growth deltas.

- **Trend Analysis:** Smoothing out daily volatility to uncover long-term seasonal or cyclical trajectories.

### Real-World Problem and How Topic Solves It

- **Problem:** An analytics dashboard plots daily sales. If a store records zero sales on a specific Tuesday, the database drops that date entirely from the query output. When rendered on a line graph, the chart connects Monday directly to Wednesday, misrepresenting the timeline scale and creating a misleading visual drop.

- **Solution:** Generating a continuous calendar table using `generate_series` and performing a `LEFT JOIN` against actual sales data ensures that missing dates are preserved and populated with a `0` value instead of disappearing.

### Industrial Example

A retail banking platform tracks monthly transaction volumes. To present a clean executive report, the data pipeline uses `DATE_TRUNC('month', transaction_date)` to aggregate raw timestamps, pairs it with `LAG(revenue, 12)` to compare performance against the exact same month last year, and computes the Year-over-Year (YoY) growth percentage.

```sql

-- 1. Filling Missing Dates using generate_series and LEFT JOIN (PostgreSQL style)
WITH DateCalendar AS (
    -- Generates a continuous sequence of dates between start and end bounds
    SELECT generate_series(
        DATE '2026-07-01', 
        DATE '2026-07-05', 
        INTERVAL '1 day'
    )::DATE AS calendar_date      -- `::` is a type-casting operator. It converts a value from one data type to another. `::DATE`: Converts the generated timestamp into a date.
)

SELECT 
    c.calendar_date,
    -- COALESCE replaces missing sales gaps with 0.00 instead of NULL
    COALESCE(s.revenue, 0.00) AS revenue,
    -- Calculates a cumulative running total without collapsing the daily rows.
    SUM(COALESCE(s.revenue, 0.00)) OVER (ORDER BY c.calendar_date)   
AS running_total
FROM DateCalendar c
LEFT JOIN daily_sales s ON c.calendar_date = s.sale_date;


-- 2. Period-over-Period (MoM) Growth Calculation using LAG
WITH MonthlyRevenue AS (
    SELECT 
	    -- DATE_TRUNC(): Groups dates into their respective months.
        DATE_TRUNC('month', sale_date) AS month_start,
        SUM(revenue) AS total_revenue
    FROM daily_sales
    GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT 
    month_start,
    total_revenue,
    -- LAG() fetches the previous month's revenue for comparison.
    LAG(total_revenue, 1) OVER (ORDER BY month_start) AS prev_month_revenue,
    -- Month-over-Month Growth Percentage Calculation
    ROUND(
        (total_revenue - LAG(total_revenue, 1) OVER (ORDER BY month_start)) * 100.0 / 
        NULLIF(LAG(total_revenue, 1) OVER (ORDER BY month_start), 0), 2
        -- NULLIF() avoids division by zero, and ROUND() formats the percentage to two decimal places.
    ) AS mom_growth_pct
FROM MonthlyRevenue;

```

- **`generate_series(...)`:** Dynamically builds a complete reference calendar sequence so no dates are omitted from analysis.

- **`COALESCE(s.revenue, 0.00)`:** Ensures missing transaction days evaluate cleanly to zero rather than dropping out or turning into `NULL`.

- **`LAG(total_revenue, 1)`:** Retrieves the revenue value from the immediately preceding row (previous month) to compute relative growth deltas side-by-side.

### Interview Corner: Time-Series Patterns

- **Q1: Why do we use `generate_series` with a `LEFT JOIN` in time-series queries?**
    - _Answer:_ Base tables only store rows where events actually occurred. If we want to report metrics over continuous intervals (like every day of the month), a `LEFT JOIN` against a generated calendar sequence ensures that days with zero activity are included rather than missing.

- **Q2: How do we calculate Month-over-Month (MoM) growth percentage in SQL?**
    - _Answer:_ We aggregate metrics by month, use the `LAG()` window function to fetch the prior month's value, and compute the percentage difference using the formula: `((Current - Previous) / Previous) * 100`.

---

## Semi-structured & Modern Data — JSON Querying

JSON (JavaScript Object Notation) data types allow relational databases to store unstructured or semi-structured key-value document payloads natively inside standard table columns. Specialized operators are used to traverse, extract, and cast nested data directly within SQL statements:

- **`->` (Arrow Operator):** returns the extracted value as a JSON/JSONB data type (preserving structural quotes or formatting)

- **`->>` (Double Arrow Operator):** extracts and coerces the value into a standard SQL scalar string text type.

- **`#>` and `#>>`:** Extracts nested JSON objects or text fields using a path array (e.g., `{address, city}`).

### Application

- **Dynamic Schema Storage:** Storing flexible user profile settings, form submissions, or third-party API webhook payloads where column schemas change frequently.

- **Hybrid Modeling:** Combining structured relational metadata (like `user_id` and `created_at`) with flexible document attributes inside a single table.

### Real-World Problem and How Topic Solves It

- **Problem:** An e-commerce system captures diverse product specifications (such as shoe sizes, clothing colors, laptop RAM capacities, and camera resolutions). Creating separate relational columns for every possible product attribute results in a sparse table with hundreds of `NULL` columns.

- **Solution:** Storing variable product specifications inside a single JSON/JSONB column allows the system to store flexible attributes without altering table schemas, while JSON extraction operators query specific nested keys instantly.

### Industrial Example

An enterprise telemetry system ingests user app logs containing nested device information. The analytics pipeline uses `->>` operators to filter and group query results based on device operating systems extracted dynamically from the raw JSON payload.

## QUERY

```sql
-- Target Table Setup
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    metadata JSONB -- Stores semi-structured profile data
);

INSERT INTO user_profiles VALUES 
(1, 'alice', '{"age": 28, "city": "New York", "preferences": {"theme": "dark", "notifications": true}}'),
(2, 'bob', '{"age": 34, "city": "London", "preferences": {"theme": "light", "notifications": false}}');

-- 1. Extracting JSON Fields using -> (JSON) and ->> (Text)
SELECT 
    username,
    metadata -> 'city' AS city_json_type,       -- Returns JSON format ("New York")
    metadata ->> 'city' AS city_text_type,     -- Returns raw text (New York)
    -- Casting extracted JSON text to an integer
    (metadata ->> 'age')::INT AS user_age
FROM user_profiles;

-- 2. Extracting Nested Keys using Path Operators (#> and #>>)
SELECT 
    username,
    metadata #> '{preferences, theme}' AS theme_json,
    metadata #>> '{preferences, theme}' AS theme_text
FROM user_profiles
WHERE (metadata ->> 'age')::INT > 30;
```

- **`metadata -> 'city'`:** Extracts the `city` field while preserving its JSON string formatting (including quotes).

- **`metadata ->> 'city'`:** Extracts the `city` value and unquotes it into a clean SQL `TEXT` string.

- **`(metadata ->> 'age')::INT`:** Casts the extracted text value explicitly into an integer type so it can be used safely in mathematical comparisons or `WHERE` clauses.

- **`#>> '{preferences, theme}'`:** Traverses nested JSON document branches using a path array to pull deep sub-keys directly.

**Q1: How do we handle missing keys or null values when querying JSON columns?**

- _Answer:_ If a requested JSON key does not exist, extraction operators return `NULL` rather than throwing a syntax error. We can use `COALESCE` or standard `IS NULL` checks to handle missing document attributes safely.

---

## Semi-structured & Modern Data — JSONB

**JSONB** (Binary JSON) is an advanced semi-structured data storage format. Unlike standard `JSON` text types (which store raw character strings complete with original whitespace and duplicate keys), `JSONB` parses the document on input and decomposes it into a binary representation.

- **Containment (`@>`):** Checks if the left JSONB document contains all key-value pairs or elements of the right JSONB document.

- **Key Existence (`?`):** Checks if a specific string key exists as a top-level field in the JSONB document.

- **`jsonb_set`:** A built-in function used to insert, update, or delete nested values within a JSONB document dynamically.

- **GIN Index:** A Generalized Inverted Index designed specifically to index keys and values inside JSONB documents for rapid lookups. The GIN index helps PostgreSQL locate rows whose JSONB data contains the specified brand. You get the same results with or without the index; the index is intended to improve search performance.

### Application

- **High-Performance Document Filtering:** Querying and filtering large tables based on nested semi-structured attributes without incurring text-parsing overhead.

- **Flexible E-commerce Catalogs:** Managing product attribute variations where different items share few common columns.

- **Indexing Semi-structured Data:** Accelerating complex key-value searches using GIN indexes.

### Real-World Problem and How Topic Solves It

- **Problem:** An e-commerce platform stores dynamic product specifications in a standard `JSON` text column. When users filter products by nested criteria (e.g., `specs ->> 'waterproof' = 'true'`), the database must re-parse the raw string text of every single row sequentially, resulting in severe CPU bottlenecks and slow query speeds at scale.

- **Solution:** Migrating the column to `JSONB` parses the document into binary format upfront. Combined with a **GIN index**, the database can locate matching documents instantly via index pointer lookups instead of scanning and parsing text strings row-by-row.

### Industrial Example

A multi-tenant SaaS application stores organization-specific UI layout preferences and feature toggles in a `JSONB` column. The application relies on containment operators (`@>`) and GIN indexes to quickly filter organizations that have specific feature flags enabled, executing complex preference checks in milliseconds.

### Code and Explanation

```sql
-- 1. Target Table Setup with JSONB
CREATE TABLE catalog_items (
    item_id INT PRIMARY KEY,
    item_name VARCHAR(100),
    specs JSONB
);

INSERT INTO catalog_items VALUES 
(1, 'Smart Watch', '{"brand": "Apple", "price": 399, "tags": ["electronics", "wearable"], "connectivity": {"bluetooth": true, "wifi": true}}'),
(2, 'Running Shoes', '{"brand": "Nike", "price": 120, "tags": ["sports", "footwear"], "connectivity": {"bluetooth": false, "wifi": false}}');

-- 2. Containment Operator (@>) Lookup
-- Finds items that contain the specified key-value pair structure
SELECT item_name 
FROM catalog_items 
WHERE specs @> '{"brand": "Apple"}';     -- item_name |Smart Watch|

-- 3. Key Existence Operator (?) Lookup
-- Checks if the top-level document contains a specific key name
SELECT item_name 
FROM catalog_items 
WHERE specs ? 'tags';      -- item_name |Smart Watch||Running Shoes|

-- 4. Modifying Nested Data using jsonb_set
UPDATE catalog_items
SET specs = jsonb_set(specs, '{price}', '429')
WHERE item_id = 1;   

-- 5. Creating a GIN Index for High-Performance JSONB Querying
CREATE INDEX idx_catalog_specs_gin ON catalog_items USING GIN (specs);     -- CREATE INDEX
```

Output 4:

| item_name     | price |
| ------------- | ----- |
| Smart Watch   | 429   |
| Running Shoes | 120   |

#### Code Breakdown:

- **`specs @> '{"brand": "Apple"}'`:** Uses the containment operator to check if the JSONB document contains the sub-document `{"brand": "Apple"}`. This query can leverage a GIN index.

- **`specs ? 'tags'`:** Tests whether the string `'tags'` exists as a top-level key.

- **`jsonb_set(specs, '{price}', '429')`:** Navigates to the `{price}` path within the binary JSONB structure and updates its value to `429`.

- **`USING GIN (specs)`:** Builds a Generalized Inverted Index over all keys and values in the JSONB column, transforming document lookups from full table scans into lightning-fast index searches.

### Interview Corner:

- **Q1: What is the key performance difference between `JSON` and `JSONB` storage types?**
    - _Answer:_ `JSON` stores data as an exact textual copy of the input string, requiring text re-parsing on every read query. `JSONB` parses data into a decomposed binary format upon insertion, eliminating parsing overhead during queries and enabling indexing via GIN.

- **Q2: How does the containment operator (`@>`) differ from the text extraction operator (`->>`) when filtering rows?**
    - _Answer:_ The extraction operator (`->>`) pulls out a scalar value which often requires explicit type casting and prevents efficient index usage unless functional indexes are built. The containment operator (`@>`) evaluates structural inclusion across the entire JSONB document and is fully optimized to leverage GIN indexes.

- **Q3: Explain how a GIN index processes a JSONB containment query (`@>`), and describe the primary write-performance trade-off associated with maintaining JSONB GIN indexes.**
    - _Answer:_ A GIN (Generalized Inverted Index) breaks down a JSONB document into individual key-value element pairs (e.g., path, key, and value entries) and stores them in an index lookup structure pointing to the matching row IDs. When a containment query (`@>`) runs, the engine looks up the target elements directly in the GIN index. **The trade-off:** Because a single JSONB document can contain dozens of keys and values, updating or inserting a row forces the database to write multiple index entries for every single key/value pair, causing significantly heavier write amplification and slower `INSERT`/`UPDATE` performance compared to standard B-Tree indexes.

---

## Semi-structured & Modern Data — Arrays & Structs

Relational databases support native **Array** and **Struct (Composite/Row)** data types to handle multi-valued attributes and nested compound objects directly within columns:

- **Array Columns:** Ordered lists of homogeneous elements (e.g., `INT[]`, `VARCHAR[]`) stored in a single cell.

- **1-Based Indexing:** SQL array indexing starts at `1` (unlike Python or C++, which start at `0`).

- **`unnest()`:** A set-returning function that expands an array column into a separate row for each element.

- **`array_agg()`:** An aggregate function that collapses rows back into a single array.

- **`ANY` / `ALL` with Arrays:** Operators used to evaluate scalar values against array elements.

### Application

- **Multi-Valued Attributes:** Storing lists of tags, phone numbers, permission roles, or category IDs inside a single table row without requiring a separate mapping table.

- **Flattening & Re-aggregating:** Transforming vertical lists into horizontal arrays for reporting or matrix operations.

### Real-World Problem and How Topic Solves It

- **Problem:** An e-commerce system tracks category tags for products. If a product belongs to multiple categories, a fully normalized relational design requires creating a separate junction table (`product_tags`) and executing frequent `JOIN` operations just to filter products matching a list of tags.

- **Solution:** Storing tags in a native array column (`VARCHAR[]`) allows the system to store multi-valued data directly within the product row and query it efficiently using array operators, eliminating unnecessary join overhead.

### 4. Industrial Example

A content management platform stores user permission roles as a native array (`roles VARCHAR[]`) inside a `users` table. Authorization checks use the `ANY` operator to verify user permissions instantly during API request validation.

### QUERY


```sql
-- Target Table Setup with Array Columns
CREATE TABLE articles (
    article_id INT PRIMARY KEY,
    title VARCHAR(100),
    tags VARCHAR(50)[] -- Native array column
);

INSERT INTO articles VALUES 
(1, 'Introduction to SQL', ARRAY['database', 'sql', 'backend']),
(2, 'Advanced Python Decorators', ARRAY['python', 'programming', 'backend']);

-- 1. Querying Arrays using 1-Based Indexing and ANY
-- Fetch articles where 'sql' is present anywhere in the tags array
SELECT title 
FROM articles 
WHERE 'sql' = ANY(tags);

-- Accessing a specific array element (1-based indexing: gets the first element)
SELECT title, tags[1] AS primary_tag
FROM articles;

-- 2. Array Modification Functions (array_append, array_length)
UPDATE articles
SET tags = array_append(tags, 'tutorial')
WHERE article_id = 1;

-- 3. Unnesting Arrays (Turning array elements into individual rows)
SELECT 
    article_id,
    title,
    unnest(tags) AS individual_tag     -- Expands the array elements vertically, turning a single row with an array of 3 tags into 3 individual rows.
FROM articles;
```

### Interview Corner: 

- **Q1: What is the starting index of an array in standard SQL?**
    - _Answer:_ SQL arrays use **1-based indexing** (the first element is accessed via `arr[1]`), unlike languages like Python or C++ which use 0-based indexing.

- **Q2: What happens to a query's row count when you use the `unnest()` function in the `SELECT` clause?**
    - _Answer:_ `unnest()` is a set-returning function that triggers **row explosion**. If a row contains an array of 5 elements, `unnest()` expands that single row into 5 separate output rows, multiplying the row count for that record.


- **Q3: When should you choose a native SQL array column over a traditional normalized child table with a foreign key?**
    - _Answer:_
        - Choose **Native Arrays** when dealing with small, fixed-shape, homogeneous multi-value attributes (like tags or email lists) where child items do not require independent metadata, complex foreign key constraints, or individual updates.
            
        - Choose a **Normalized Child Table** when child elements represent independent entities that need rigorous referential integrity, unique constraints, independent timestamps, frequent updates, or multi-column indexing.

---
## Semi-structured & Modern Data — Flattening Nested Data

Flattening nested data is the process of converting hierarchical, multi-valued arrays or nested JSON collections into flat, tabular row formats. This is typically achieved using set-returning functions (like `jsonb_array_elements` or `unnest`) combined with a `LATERAL` join to unpack elements while retaining relational context.

### Application

- **ETL and Data Pipelines:** Transforming nested JSON API responses into flat staging tables for data warehouse loading.

- **Analytical Reporting:** Enabling standard group-by and aggregate operations on data elements that were previously locked inside nested arrays or documents.

### Real-World Problem and How Topic Solves It

- **Problem:** An order management database stores customer orders where each order contains a nested JSON array of purchased line items (`items: [{item_id, qty, price}, ...]`). Management wants to calculate total revenue per product across all orders. Running standard aggregations fails because the items are locked inside a nested array blob.

- **Solution:** Using a `LATERAL` join with `jsonb_array_elements` unpacks each nested item into its own discrete row while keeping the parent `order_id` attached, allowing standard relational aggregations to run smoothly.

### Industrial Example

A food delivery application ingests multi-vendor order payloads from external partners. The data engineering pipeline uses flattening functions to explode nested delivery milestone logs into flat rows, tracking precise step-by-step driver timing metrics in the data warehouse.

## QUERY 

```sql
-- Target Table Setup with Nested JSONB Data
CREATE TABLE customer_orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_data JSONB
);

INSERT INTO customer_orders VALUES 
(101, 501, '{"order_date": "2026-06-01", "items": [{"product": "Laptop", "qty": 1, "price": 1200}, {"product": "Mouse", "qty": 2, "price": 25}]}'),
(102, 502, '{"order_date": "2026-06-02", "items": [{"product": "Keyboard", "qty": 1, "price": 80}]}'),
(103, 503, '{"order_date": "2026-06-03", "items": []}'); -- Order with zero items

-- 1. Flattening Nested JSON Arrays using LEFT JOIN LATERAL
SELECT 
    o.order_id,
    o.customer_id,
    item->>'product' AS product_name,
    (item->>'qty')::INT AS quantity, -- extract values from JSON and convert them into numeric data types.
    (item->>'price')::DECIMAL AS unit_price  -- extract values from JSON and convert them into numeric data types.
FROM customer_orders o
-- LEFT JOIN LATERAL ensures orders with empty items arrays are still preserved
LEFT JOIN LATERAL jsonb_array_elements(o.order_data->'items') AS item ON TRUE;
```
## Expected output

|order_id|customer_id|product_name|quantity|unit_price|
|---|---|---|---|---|
|101|501|Laptop|1|1200|
|101|501|Mouse|2|25|
|102|502|Keyboard|1|80|
|103|503|`NULL`|`NULL`|`NULL`|

- `o.order_data->'items'` extracts the JSON array containing the order's products.

- `jsonb_array_elements()` expands that array into one row per product.

- `LEFT JOIN LATERAL ... ON TRUE` evaluates the array for each order and preserves the order even when its array has no elements.

- `item->>'product'` extracts the product name as text, while `::INT` and `::DECIMAL` convert quantity and price into numeric types.

### Interview Questions

#### Question 1:  You are flattening a table of user profiles containing a nested array of addresses using an inner join with `jsonb_array_elements`. You notice that users who haven't added an address completely disappear from your final output report.

- **The Task:** Explain why this happens and how to modify your query to keep those users visible.
- **Solution Approach:**
    - _Why it happens:_ A standard inner `LATERAL` join drops parent rows if the evaluated array is empty or `NULL`, because the function returns zero rows to join against.
    - _The Fix:_ Replace the inner `LATERAL` join with a `LEFT JOIN LATERAL` so that parent entities with empty or missing collections are preserved with `NULL` values.

#### Question 2: Your data pipeline flattens a table containing 1 million customer records, where each customer record contains an average of 10 nested transaction logs.
    
- **The Task:** What structural performance issue occurs when you unnest these arrays, and how can you safeguard your downstream queries against it?
- **Solution Approach:**
    - _The Issue:_ Unnesting causes **row explosion** (multiplication of rows). 1 million rows multiplied by 10 elements explodes the dataset into 10 million rows, increasing memory pressure, temporary disk space usage, and join costs.
    - _Safeguard:_ Filter or aggregate the nested data _before_ or _during_ the unnest operation (e.g., filtering out irrelevant history or performing conditional checks) rather than unnesting the entire raw history blindly across wide table joins.

#### Question 3: After flattening a nested JSON log table to perform specific calculations on individual elements, you need to aggregate the processed rows back into a single structured JSON object per user session.
    
- **The Task:** Which aggregation function would you use to pack flat rows back into a JSON/JSONB array?
- **Solution Approach:**
    - Use **`jsonb_agg()`** (or `json_agg()`). This aggregate function collects individual rows or scalar values back into a single JSONB array format. Combined with `jsonb_build_object()`, it allows you to reconstruct customized hierarchical structures from flattened tables.

---
## Programmability — Stored Procedures

A **stored procedure** is a precompiled collection of one or more SQL statements and procedural logic (such as variables, conditional branches, loops, and exception handling) stored directly inside the database catalog. It is executed on the database server via an explicit invocation command (e.g., `CALL` or `EXEC`).

- **Parameters (`IN`, `OUT`, `INOUT`):** Input parameters pass data into the procedure, output parameters return computed results back to the caller, and inout parameters handle both.

- **Procedural Block:** Extends declarative SQL with programmatic control flows like `IF/ELSE` statements and loops.

### Application

- **Encapsulating Complex Workflows:** Grouping multi-step business transactions, data transformations, and validation rules into a single server-side script.

- **Network Round-Trip Reduction:** Executing heavy multi-statement logic on the database server with a single network call rather than sending dozens of individual queries back and forth from the application layer.

### Real-World Problem and How Topic Solves It

- **Problem:** An application performs a complex monthly payroll calculation requiring sequential inserts, updates, and validations across five different tables. Executing these steps individually from the application tier creates heavy network latency and risks leaving data in a half-updated state if a network timeout occurs midway.

- **Solution:** Encapsulating the entire payroll workflow inside a single database stored procedure ensures that all procedural logic and transactional states execute securely on the database engine side in a single network round-trip.

### Industrial Example

An enterprise healthcare system executes daily patient insurance claims processing via database stored procedures. The procedure loops through pending claims, validates billing codes against reference tables, updates ledger statuses, and handles transaction commits or rollbacks autonomously.

### QUERY

```sql
-- Target Table Setup
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance DECIMAL(10, 2)
);

INSERT INTO accounts VALUES (1, 1000.00), (2, 500.00);

-- 1. Creating a Stored Procedure with Transaction Control (PostgreSQL PL/pgSQL style)
CREATE OR REPLACE PROCEDURE transfer_funds(
    IN source_acc INT,
    IN target_acc INT,
    IN transfer_amount DECIMAL(10, 2)
)
LANGUAGE plpgsql
AS $$
DECLARE
    current_balance DECIMAL(10, 2);
BEGIN
    -- Check source account balance
    SELECT balance INTO current_balance 
    FROM accounts 
    WHERE account_id = source_acc;

    IF current_balance < transfer_amount THEN
        RAISE EXCEPTION 'Insufficient funds in source account: %', source_acc;
    END IF;

    -- Deduct from source
    UPDATE accounts 
    SET balance = balance - transfer_amount 
    WHERE account_id = source_acc;

    -- Add to target
    UPDATE accounts 
    SET balance = balance + transfer_amount 
    WHERE account_id = target_acc;

    -- Explicit commit of the transfer transaction
    COMMIT;

EXCEPTION
    WHEN OTHERS THEN
        -- Rollback changes if any error occurs
        ROLLBACK;
        RAISE;
END;
$$;

-- 2. Calling the Stored Procedure
CALL transfer_funds(1, 2, 250.00);
```

- `CREATE OR REPLACE PROCEDURE`: Creates the procedure, or replaces it if it already exists.
- `IN source_acc INT`: Input parameter containing the sender's account ID.
- `IN target_acc INT`: Input parameter containing the receiver's account ID.
- `IN transfer_amount DECIMAL(10, 2)`: Input amount, with up to 10 total digits and 2 decimal places.
- `LANGUAGE plpgsql`: Says the procedure is written in PostgreSQL's procedural SQL language.
- `AS $$ ... $$`: Delimits the procedure's body.
- `DECLARE`: Defines variables used inside the procedure.
- `RAISE EXCEPTION`: Stops execution and reports an error if the condition is true.
- `%`: Placeholder replaced with the source account ID.
- `COMMIT`: Saves the transaction's changes permanently.
- `EXCEPTION`: Starts the error-handling section.
- `WHEN OTHERS`: Catches most runtime errors.
- `ROLLBACK`: Attempts to undo the transaction.
- `RAISE`: Rethrows the error so the caller knows the operation failed.

### Interview Questions

Question 1: You are designing an order fulfillment workflow that updates inventory levels, logs audit events, and charges a payment gateway. The team is debating whether to implement this logic inside an application-layer service or a database stored procedure.

- The Task: Compare the trade-offs between stored procedures and application-layer code regarding performance, maintenance, and debugging.
- Solution Approach: Stored procedures run close to the storage engine, reducing network overhead for multi-statement batch jobs and centralizing database logic. However, they are harder to version-control, scale horizontally across application servers, debug with standard application tracing tools, and tie business logic tightly to a specific database vendor dialect.

Question 2: A junior developer writes a stored procedure that performs 50 individual row updates inside a loop, executing a separate database transaction commit after every single iteration.

- The Task: Identify the performance and data integrity risks associated with this pattern, and propose an optimized approach.
- Solution Approach: The Issue: Committing inside a loop creates severe overhead due to constant transaction log flushing (frequent disk I/O). Furthermore, it violates atomicity; if the script fails on iteration 45, the first 44 iterations remain permanently committed, corrupting data consistency. The Fix: Wrap the entire loop execution block inside a single atomic transaction transaction block (`BEGIN` ... `COMMIT` at the procedure level) so that changes commit together or roll back entirely on failure.

---
## User-Defined Functions (UDFs)

A **User-Defined Function (UDF)** is a database routine that accepts parameters, performs calculations or data manipulations, and returns a value or a set of rows. Unlike stored procedures, functions are designed to be evaluated inline within SQL statements (`SELECT`, `WHERE`, `JOIN`).

- **Scalar Functions:** Return a single atomic value (e.g., an integer, string, or boolean) per invocation.

- **Table-Valued / Set-Returning Functions:** Return a table or a set of rows that can be queried like a physical table.

- **Deterministic vs. Non-Deterministic:**

    - _Deterministic:_ Always returns the exact same output given the same input parameters (e.g., mathematical operations, string manipulation).
    - _Non-Deterministic:_ Can return different outputs on separate calls even with identical inputs (e.g., `NOW()`, `RANDOM()`).

### Application

- **Reusable Computations:** Encapsulating complex mathematical formulas, string formatting, or currency conversions into reusable modules that can be called across multiple queries.

- **Inline Filtering:** Passing parameters into table-valued functions inside `FROM` or `JOIN` clauses to generate dynamic result sets on the fly.

### Real-World Problem and How Topic Solves It

- **Problem:** An analytics team repeatedly writes identical complex tax-calculation formulas across twenty different financial reports. If tax laws change, developers must manually update the formula in all twenty SQL scripts, leading to maintenance errors and inconsistent reporting.

- **Solution:** Encapsulating the formula inside a scalar UDF (`calculate_tax()`) allows all reports to reference the single function definition, ensuring that updating the function logic automatically updates every dependent report instantly.

### Industrial Example

An e-commerce pricing engine uses a deterministic scalar UDF to compute discounted unit prices based on customer tier and seasonal promotion codes, embedding the function directly into core inventory and checkout query pipelines.

## CODE

```sql
-- 1. Creating a Scalar UDF (PostgreSQL style)
CREATE OR REPLACE FUNCTION calculate_final_price(
    base_price DECIMAL(10, 2), 
    discount_pct DECIMAL(5, 2)
)
RETURNS DECIMAL(10, 2)
LANGUAGE plpgsql
DETERMINISTIC
AS $$
BEGIN
    RETURN base_price * (1.0 - (discount_pct / 100.0));
END;
$$;

-- Using the Scalar Function inside a standard SELECT query
SELECT 
    item_name,
    base_price,
    calculate_final_price(base_price, 15.00) AS discounted_price
FROM catalog_items;


-- 2. Creating a Table-Valued / Set-Returning Function
CREATE OR REPLACE FUNCTION get_items_by_price_threshold(max_price DECIMAL(10, 2))
RETURNS TABLE (
    item_id INT,
    item_name VARCHAR,
    price DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT c.item_id, c.item_name, (c.specs->>'price')::DECIMAL AS price
    FROM catalog_items c
    WHERE (c.specs->>'price')::DECIMAL <= max_price;
END;
$$;

-- Querying the Table-Valued Function like a table
SELECT * FROM get_items_by_price_threshold(200.00);
```

- **`RETURNS DECIMAL(10, 2)`:** Declares that the scalar function outputs a single numeric value.

- **`DETERMINISTIC`:** Tells the query optimizer that this function produces identical outputs for identical inputs, enabling execution plan optimizations.

- **`RETURNS TABLE (...) AS $$ RETURN QUERY ... $$`:** Defines a set-returning function that executes an internal query and returns tabular results.

### Interview Questions

#### Question 1: What are the fundamental differences between a Stored Procedure and a User-Defined Function (UDF) in SQL?

- **Solution Approach:**
    - _Execution Context:_ Functions can be called inline directly inside `SELECT`, `WHERE`, or `JOIN` clauses as part of a query execution tree. Procedures cannot be called inline; they must be invoked explicitly using a standalone execution command (`CALL` or `EXEC`).
    - _Transaction Control:_ Procedures can contain transaction control statements (`BEGIN`, `COMMIT`, `ROLLBACK`). Standard SQL functions generally cannot execute explicit transaction control commands.
    - _Side Effects:_ Functions are typically expected to be side-effect-free (read-only data access), whereas procedures frequently perform data modifications, mutations, and administrative tasks.

#### Question 2: What is the performance pitfall of using scalar UDFs in large-scale relational queries, and how do database engines handle them?

- **Solution Approach:**
    - _The Pitfall:_ Row-by-row execution. If a scalar UDF is called inside the `SELECT` clause of a query scanning 1 million rows, the database engine may execute the function code 1 million times sequentially. This triggers context switches between the SQL execution engine and the procedural language runtime, bypassing vectorization and causing severe CPU bottlenecks.
    - _Mitigation:_ Modern engines support inlineable functions or recommend rewriting scalar UDF logic into native SQL expressions, joins, or window functions so the optimizer can vectorize execution.

#### Question 3: Why does the query optimizer care whether a user-defined function is declared as `DETERMINISTIC` (or `IMMUTABLE`) versus `NON-DETERMINISTIC` (or `VOLATILE`)?

- **Solution Approach:**
    
    - If a function is marked **Deterministic / Immutable**, the optimizer knows its output depends solely on its input parameters. This allows the optimizer to cache or pre-compute function results, evaluate expressions once per query instead of once per row, or apply index-based optimizations.
    - If a function is marked **Non-Deterministic / Volatile** (e.g., calling `NOW()` or `RANDOM()`), the optimizer must re-evaluate the function expression for every single row processed, preventing aggressive caching and restricting specific execution plan optimizations.

---

## Triggers

### 1. Definition

A **database trigger** is a stored procedural routine configured to execute automatically (fire) in response to a specific Data Modification Language (DML) event—such as an `INSERT`, `UPDATE`, or `DELETE`—occurring on a target table or view.

- **Timing (`BEFORE`, `AFTER`, `INSTEAD OF`):** Determines whether the trigger runs before the table modification is written, after the change is committed, or in place of the standard operation (commonly used on views).

- **Granularity (`FOR EACH ROW` vs. `FOR EACH STATEMENT`):** Row-level triggers execute once for every individual row affected by the statement, whereas statement-level triggers execute exactly once per DML statement regardless of how many rows are modified.

- **Transition Variables (`NEW` and `OLD`):** Special record references available inside row-level triggers. `NEW` holds the incoming row state (for `INSERT` and `UPDATE`), and `OLD` holds the existing row state prior to modification (for `UPDATE` and `DELETE`).

### 2. Application

- **Automated Audit Logging:** Capturing historical snapshots or change logs whenever sensitive rows are modified or deleted.
- **Enforcing Invariant Business Rules:** Automatically maintaining derived columns, such as auto-updating `updated_at` timestamps or recalculating aggregate balances across related tables.

### 3. Real-World Problem and How Topic Solves It

- **Problem:** An application updates sensitive user records across five different microservices. Developers frequently forget to update the `updated_at` timestamp column or log modification events in application code, leading to stale audit trails and missing data lineage.
- **Solution:** Attaching a `BEFORE UPDATE` trigger directly to the database table ensures that every time an `UPDATE` statement executes—regardless of which application service sent it—the `updated_at` timestamp is automatically refreshed at the storage layer.

### 4. Industrial Example

A financial ledger system uses an `AFTER INSERT OR UPDATE` trigger to write transaction event payloads into an immutable audit table. This guarantees strict regulatory compliance without depending on application-tier logging reliability.

### 5. Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE employee_records (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    salary DECIMAL(10, 2),
    updated_at TIMESTAMP
);

-- 1. Create a Trigger Function to auto-update timestamps
CREATE OR REPLACE FUNCTION set_updated_timestamp()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Automatically stamp the current time on the incoming row
    NEW.updated_at := CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$;

-- 2. Bind the Function to a Table via a BEFORE UPDATE Trigger
CREATE TRIGGER trg_auto_timestamp
    BEFORE UPDATE ON employee_records
    FOR EACH ROW
    EXECUTE FUNCTION set_updated_timestamp();

-- Test Insert
INSERT INTO employee_records VALUES (1, 'Alice', 75000.00, NULL);

-- Test Update (Trigger fires automatically)
UPDATE employee_records SET salary = 80000.00 WHERE emp_id = 1;
```

- `CREATE OR REPLACE FUNCTION`: Creates the function or replaces its existing definition.
- `RETURNS TRIGGER`: Indicates that this function is designed to be called by a trigger.
- `NEW`: Represents the new version of the row being updated.
- `NEW.updated_at := CURRENT_TIMESTAMP`: Sets the timestamp to the current transaction's time. `:=` → assigns a value to that field.
- `RETURN NEW`: Sends the modified row back so PostgreSQL can save it.
- `BEFORE UPDATE`: Runs the trigger before an update is saved.
- `ON employee_records`: Applies it to this table.
- `FOR EACH ROW`: Runs the function separately for every row being updated.
- `EXECUTE FUNCTION`: Calls the trigger function.
### 6. Interview Questions

#### Question 1: What is the operational difference between a row-level trigger (`FOR EACH ROW`) and a statement-level trigger (`FOR EACH STATEMENT`)?

- **Solution Approach:**
    - _Row-Level Triggers_ execute once for every individual record impacted by the DML statement. They have access to `NEW` and `OLD` transition variables, making them ideal for row-specific validation or column mutations.
    - _Statement-Level Triggers_ execute only once per SQL query, regardless of whether the query modifies 0 rows or 10 million rows. They are typically used for bulk auditing, batch validation checks, or resetting session caches.
        
#### Question 2: Explain how `NEW` and `OLD` transition variables behave during an `UPDATE` operation versus an `INSERT` or `DELETE` operation.

- **Solution Approach:**
    - During an `INSERT`, only the `NEW` variable is populated with the incoming row data (`OLD` is `NULL`).
    - During a `DELETE`, only the `OLD` variable is populated with the deleted row data (`NEW` is `NULL`).
    - During an `UPDATE`, both `OLD` (containing pre-image values) and `NEW` (containing post-image values) are available, allowing triggers to compare old and new values to detect if specific columns were altered.
        
#### Question 3: What are the primary architectural risks of relying heavily on database triggers for core business logic in large-scale applications ("hidden logic")?

- **Solution Approach:**
    - _Hidden Logic & Debugging Difficulty:_ Triggers execute implicitly behind the scenes. Developers debugging application code may not realize database-level triggers are mutating data, causing unexpected side effects or difficult-to-trace bugs.
    - _Performance Impact:_ Complex or poorly written row-level triggers executing across large bulk updates can severely degrade database write performance due to recursive overhead and transaction lock contention.

---
## Security & Administration — `GRANT` / `REVOKE`

`GRANT` and `REVOKE` are Data Control Language (DCL) commands used to manage access privileges and security permissions on database objects (such as tables, views, schemas, and procedures).

- **Privilege Types:** Specific operations allowed on an object, including `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, and `USAGE`.

- **`WITH GRANT OPTION`:** Allows the recipient user or role to grant those exact same privileges to other users.

- **Column-Level Privileges:** Restricting access to specific columns within a table rather than granting access to the entire row structure.

### Application

- **Least Privilege Enforcement:** Ensuring that users, applications, and reporting tools possess only the minimum necessary permissions required to perform their specific tasks.

- **Access Governance:** Restricting data modification rights (`INSERT`, `UPDATE`, `DELETE`) to application service roles while granting read-only access (`SELECT`) to analytics teams.

### Real-World Problem and How Topic Solves It

- **Problem:** An analytics team connects directly to the production database to build weekly reports. If an analyst accidentally executes an improper `UPDATE` or `DROP TABLE` statement, it can cause catastrophic production data loss or downtime.

- **Solution:** Using `GRANT SELECT` gives the analytics user base read access to the necessary tables, while explicitly omitting `INSERT`, `UPDATE`, and `DELETE` permissions, physically preventing unauthorized data modifications at the database engine level.

### Industrial Example

An enterprise SaaS platform maintains strict database security boundaries. The backend application role is granted `SELECT`, `INSERT`, and `UPDATE` on transaction tables, while third-party BI analytics tools connect using a separate database user restricted exclusively to `SELECT` privileges via explicit `GRANT` statements.

### Code and Explanation

```sql
-- 1. Granting Table-Level Privileges
GRANT SELECT ON customers TO reporting_analyst;
GRANT INSERT, UPDATE ON customers TO app_service_role;

-- 2. Granting Column-Level Privileges (PostgreSQL / Oracle style)
-- Restricts the analyst to reading only non-sensitive columns
GRANT SELECT (customer_id, signup_date, country) ON customers TO junior_analyst;

-- 3. Granting Privileges with GRANT OPTION
GRANT SELECT ON financial_ledger TO security_admin WITH GRANT OPTION;

-- 4. Revoking Privileges
REVOKE INSERT ON customers FROM reporting_analyst;
```

- **`GRANT SELECT ON customers TO reporting_analyst`:** Assigns read-only access rights to the specified user or role.

- **`GRANT SELECT (customer_id, country) ...`:** Limits access strictly to named columns, hiding sensitive fields (like email or phone numbers) from the user.

- **`WITH GRANT OPTION`:** Empowers the grantee to pass these same permissions on to other system users.

- **`REVOKE INSERT ... FROM ...`:** Strips previously assigned permissions from a user or role immediately.

### Interview Questions

#### Question 1: What is the purpose of the `WITH GRANT OPTION` clause, and what security risk does it introduce?

- **Solution Approach:**
    - _Purpose:_ It allows the grantee to delegate the assigned privileges to other users or roles in the database.
    - _Security Risk:_ It introduces the risk of permission escalation and privilege proliferation, where tracking who holds access becomes difficult, potentially creating unintended backdoors or unauthorized privilege chains.

#### Question 2: How do table-level privileges differ from column-level privileges in relational databases?

- **Solution Approach:**
    - _Table-Level Privileges_ grant or revoke access to the entire relational table (all rows and all columns).
    - _Column-Level Privileges_ restrict access to specific subsets of columns within a table, enabling fine-grained security control (e.g., hiding PII columns like salaries or SSNs while permitting read access to general attributes).

#### Question 3: What happens to dependent objects or permissions when you execute a `REVOKE` command on a base table?

- **Solution Approach:**
    - When privileges are revoked from a user or role, their ability to query or modify the target object is immediately terminated. If the revoked user had extended permissions via `GRANT OPTION`, cascading revokes may also strip permissions from downstream users who received access through them, depending on the database engine's DCL implementation.

---

## Roles & Permissions

A **database role** is a named collection of privileges and permissions used as a security container to manage access control across multiple users.

- **Users vs. Roles:** In modern database engines (like PostgreSQL), users and roles are effectively unified (users are simply roles with login privileges).

- **Role Inheritance:** The ability for one role to inherit all permissions assigned to another role (`GRANT parent_role TO child_role`).

- **Principle of Least Privilege (PoLP):** A security concept dictating that every user or service account must be given only the minimum necessary privileges required to complete its tasks and nothing more.

### Application

- **Centralized Access Management:** Grouping permissions by operational job functions (such as `data_engineer`, `bi_analyst`, or `app_service`) instead of assigning permissions to individual user accounts one by one.

- **Streamlined Onboarding & Offboarding:** Adding a new employee simply requires assigning them to the appropriate role, instantly granting all required permissions without manual grant scripts.

### Real-World Problem and How Topic Solves It

- **Problem:** A company hires 20 new data analysts simultaneously. If the database administrator grants individual table permissions to each analyst one by one, it creates administrative overhead and leaves room for human error—such as accidentally granting write access where only read access is permitted.

- **Solution:** Creating a centralized `analyst_role` with predefined read permissions (`GRANT SELECT`) allows the administrator to add all 20 users to that single role instantly via inheritance, ensuring uniform and secure access governance.

### Industrial Example

An enterprise cloud data warehouse organizes access using hierarchical roles. The `reporting_base` role holds `SELECT` permissions on staging tables, the `bi_analyst` role inherits from `reporting_base`, and individual analyst accounts inherit from `bi_analyst`, establishing clean permission lineage.

### Code and Explanation


```sql
-- 1. Creating Roles (PostgreSQL style)
CREATE ROLE bi_analyst NOLOGIN;
CREATE ROLE data_engineer NOLOGIN;

-- 2. Assigning Privileges to Roles
GRANT SELECT ON ALL TABLES IN SCHEMA public TO bi_analyst;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO data_engineer;

-- 3. Creating a User Account with Login Privileges
CREATE USER alex_analyst WITH PASSWORD 'SecurePassword123!';

-- 4. Granting Role Membership (Inheritance)
GRANT bi_analyst TO alex_analyst;

-- 5. Setting Default Role Behavior for the User
ALTER USER alex_analyst SET ROLE bi_analyst;
```

#### Code Breakdown:

- **`CREATE ROLE ... NOLOGIN`:** Creates a security group container that holds privileges but cannot be used to log in directly by clients.

- **`GRANT bi_analyst TO alex_analyst`:** Assigns the role to a user account, enabling the user to inherit all privileges associated with `bi_analyst`.

- **`NOLOGIN`:** A security best practice ensuring that operational roles cannot be used as direct login entry points by unauthorized actors.

#### Question 1: What is the functional difference between a database User and a database Role in enterprise database management?

- **Solution Approach:**
    - A _User_ represents an identity capable of authenticating and logging into the database (possessing login credentials).
    - A _Role_ represents a collection of permissions and privileges. In modern database architectures (like PostgreSQL), users and roles are unified—a user is simply a role with login privileges (`LOGIN` attribute enabled), allowing permissions to be managed collectively via role inheritance rather than per individual.

#### Question 2: How does role inheritance work in relational database management systems?

- **Solution Approach:**
    - Role inheritance allows a child role or user to inherit all privileges assigned to a parent role. When `GRANT parent_role TO child_role` is executed, any permissions granted to `parent_role` automatically apply to `child_role` without needing duplicate grant statements, simplifying multi-layered security hierarchies.
        
#### Question 3: What is the Principle of Least Privilege (PoLP), and why is it critical when configuring database user permissions?

- **Solution Approach:**
    - The Principle of Least Privilege dictates that users, applications, and system processes should operate with the bare minimum permissions necessary to perform their functions. Applying PoLP in databases prevents lateral movement during security breaches, stops rogue scripts or compromised application credentials from executing unauthorized DDL/DML commands, and protects sensitive data from internal exposure.

---

## Row-Level Security (RLS)

**Row-Level Security (RLS)** is an advanced database access control mechanism that allows database administrators to restrict which data rows are visible or modifiable by a user or role during standard DML operations (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).

- **`ENABLE ROW LEVEL SECURITY`:** Activates the security enforcement layer on a specified relational table.

- **`CREATE POLICY`:** Defines specific security rules governing row access.

- **`USING` Clause:** Evaluates existing rows for read, update, or delete operations (acts as a dynamic `WHERE` filter).

- **`WITH CHECK` Clause:** Validates incoming rows during `INSERT` or `UPDATE` operations to ensure new/modified data complies with policy boundaries.

### Application

- **Multi-Tenant Architectures:** Hosting multiple corporate clients in a single shared database table while mathematically guaranteeing that Tenant A can never view or modify Tenant B’s data.

- **Territorial Data Partitioning:** Restricting regional sales managers to query only customer records belonging to their assigned geographic territory.

### Real-World Problem and How Topic Solves It

- **Problem:** A multi-tenant SaaS application stores all customer orders in a single monolithic `orders` table. If a developer forgets to append `WHERE tenant_id = X` in an application query, or if an analyst runs an ad-hoc query, data leaks across tenants, resulting in a severe security breach.

- **Solution:** Enabling Row-Level Security directly on the database table automatically injects security predicates into _every_ query executed by database roles, ensuring that data isolation is enforced at the storage engine level rather than relying entirely on application-tier code.

### Industrial Example

An enterprise healthcare platform manages patient records. Database administrators implement RLS policies linked to session variables (`current_setting('app.current_user_id')`). When a doctor queries the records table, the database transparently filters out all rows except those associated with the doctor's authorized patient list.

### Code and Explanation

```sql
-- Target Table Setup
CREATE TABLE tenant_invoices (
    invoice_id INT PRIMARY KEY,
    tenant_id VARCHAR(50),
    amount DECIMAL(10, 2),
    status VARCHAR(20)
);

-- 1. Enable Row-Level Security on the Table
ALTER TABLE tenant_invoices ENABLE ROW LEVEL SECURITY;

-- 2. Create a Security Policy for Multi-Tenant Isolation
CREATE POLICY tenant_isolation_policy ON tenant_invoices
    AS RESTRICTIVE
    FOR ALL
    TO PUBLIC
    USING (tenant_id = current_setting('app.current_tenant_id', true));

-- 3. Setting the session variable and querying the table
-- (Only rows matching the current session tenant variable will be returned)
SET LOCAL app.current_tenant_id = 'tenant_alpha_99';

SELECT * FROM tenant_invoices;
```

#### Code Breakdown:

- **`ALTER TABLE ... ENABLE ROW LEVEL SECURITY`:** Turns on the RLS enforcement switch. Without this, security policies are ignored.
- **`USING (tenant_id = current_setting(...))`:** Dynamically evaluates every row against a runtime session variable, acting as an invisible security filter on all queries.
- **`AS RESTRICTIVE`:** Ensures that multiple policies on the same table are combined using logical `AND` operators, tightening security boundaries.

### Advanced Interview Questions

#### Question 1: How does Row-Level Security (RLS) impact query optimization and execution plans at scale, and how do database optimizers handle implicit predicate injection?

- **Solution Approach:**
    - _Execution Impact:_ RLS injects dynamic security predicates (such as function calls like `current_setting()`) into the query parse tree during execution planning. If not carefully designed, these dynamic filters can prevent the optimizer from leveraging indexes effectively, causing sequential table scans (`Seq Scan`) across large datasets.
    - _Mitigation:_ Indexing the tenant/filtering column explicitly and ensuring security functions are marked `IMMUTABLE` or optimized allows the query planner to treat RLS predicates like standard `WHERE` clauses during cost-based optimization.

#### Question 2: By default, who bypasses Row-Level Security policies in PostgreSQL, and what architectural considerations must be addressed when setting up application connection pools (like PgBouncer in transaction mode) where database sessions are reused across different end-users?

- **Solution Approach:**
    - _Bypass:_ Table owners and superusers bypass RLS policies by default (`BYPASSRLS` attribute).
    - _Connection Pooling Pitfall:_ In connection poolers operating in _transaction mode_ (e.g., PgBouncer), database sessions are pooled and recycled across different clients. If session-level variables (like `SET LOCAL app.current_user_id`) are used to drive RLS policies, a connection returned to the pool can accidentally retain the previous user's security context, leaking data to the next client.
    - _Solution:_ Explicitly reset session state, issue `RESET ALL` upon connection checkout, or pass user context per query statement rather than relying on persistent session variables.

#### Question 3: Compare the execution behavior and operational scope of the `USING` clause versus the `WITH CHECK` clause when defining a security policy for an `UPDATE` operation.

- **Solution Approach:**
    - The **`USING` clause** determines which existing rows are visible and eligible to be targeted or modified by the operation (acting as the pre-image filter).
    - The **`WITH CHECK` clause** validates the newly mutated row state _after_ the update or insert is applied, ensuring that the modified row does not violate policy boundaries (preventing a user from updating a row and shifting its ownership to another tenant). If omitted during an `UPDATE`, the `USING` expression is used for `WITH CHECK` validation by default.

---
## SQL Injection & Parameterized Queries

**SQL Injection (SQLi)** is a critical security vulnerability that occurs when untrusted user input is directly concatenated into a dynamic SQL query string instead of being safely isolated from the query structure.

- **Mechanism:** Malicious input manipulates the syntax of the SQL statement, altering its intended logic (e.g., bypassing authentication, extracting sensitive data via `UNION`, or executing administrative commands like `DROP TABLE`).

- **Parameterized Queries (Prepared Statements):** A defense mechanism where the SQL query structure is compiled and sent to the database engine _first_ with placeholders (`?`, `$1`). User inputs are then sent separately as raw data parameters, ensuring the database treats input strictly as literal values rather than executable code.

### Application

- **Secure Application Integration:** Preventing attackers from compromising web applications, APIs, and microservices connected to the database.

- **Safe Dynamic SQL:** Executing dynamic database routines, stored procedures, or ORM queries without exposing schema boundaries to injection attacks.

### Real-World Problem and How Topic Solves It

- **Problem:** A login query concatenates user-supplied form fields: `SELECT * FROM users WHERE username = '` + `userInput` + `' AND password = '` + `passInput` + `'`. An attacker inputs `' OR '1'='1` into the username field, transforming the query into: `SELECT * FROM users WHERE username = '' OR '1'='1' AND password = ''`. The condition evaluates to `TRUE`, granting administrative access without a valid password.

- **Solution:** Using a parameterized query sends `SELECT * FROM users WHERE username = $1 AND password = $2` to the database engine. The database binds `' OR '1'='1` strictly as a literal text parameter for `$1`, matching zero users and blocking the bypass attempt completely.

### Industrial Example

An enterprise web application uses parameterized queries (prepared statements) across all API endpoints. When an attacker attempts to pass payload strings containing nested SQL commands (`admin'; DROP TABLE orders; --`) into a search endpoint, the database engine escapes and treats the entire payload as a raw search string parameter, protecting table integrity.

### Code and Explanation


```sql
-- Vulnerable Dynamic Query Pattern (Do NOT use in production)
-- String concatenation allows structural manipulation:
-- Query string: 'SELECT * FROM accounts WHERE acc_id = ' || user_input;

-- 1. Prepared Statement (PostgreSQL Syntax)
PREPARE fetch_account (INT) AS
SELECT account_id, balance, status 
FROM accounts 
WHERE account_id = $1;

-- 2. Safe Execution with Parameter Binding
-- The parameter value (101) is bound strictly as a literal value
EXECUTE fetch_account(101);

-- 3. Deallocating Prepared Statements
DEALLOCATE fetch_account;
```

#### Code Breakdown:

- **`PREPARE fetch_account (INT)`:** Pre-compiles the SQL command structure on the database server and specifies explicit data types for parameters (`$1`).

- **`EXECUTE fetch_account(101)`:** Passes input parameters safely to the prepared query payload. The input cannot break out of its scalar type boundary.

- **`DEALLOCATE`:** Frees server memory allocated to the pre-compiled execution plan.

### 6. Interview Questions

#### Question 1: Explain the fundamental technical difference between how a relational database processes dynamic string-concatenated SQL queries versus parameterized queries (prepared statements).

- **Solution Approach:**
    - _Dynamic String Concatenation:_ The database treats the incoming string as a single unit of text, parsing, compiling, and building an execution tree _after_ user inputs are mixed into the statement. This allows user input to inject SQL tokens (`OR`, `UNION`, `;`, `--`) directly into the parser.
    - _Parameterized Queries:_ Query compilation occurs in two separate phases:
        1. The query structure containing placeholders (`$1`, `?`) is parsed, validated, and compiled into an execution tree.
        2. User parameters are transmitted separately over the wire as raw scalar data values. The execution engine binds these values directly to pre-allocated nodes in the execution tree without re-parsing them, rendering code injection physically impossible.

#### Question 2: Can stored procedures or ORM frameworks still be vulnerable to SQL injection? If so, give a concrete example of how it occurs.

- **Solution Approach:**
    - Yes. Stored procedures and ORMs are not inherently immune if they use dynamic SQL concatenation internally.
    - _Stored Procedure Example:_ If a procedure uses `EXECUTE IMMEDIATE 'SELECT * FROM ' || user_table_var;` or PostgreSQL's `EXECUTE 'SELECT * FROM users WHERE name = ''' || input_str || '''';`, string concatenation inside the procedure reinstates the vulnerability.
    - _Fix:_ Use explicit identifier/literal escaping built-ins (e.g., `quote_ident()` or `quote_literal()` in PL/pgSQL, or parameterized dynamic execution via `USING`).

#### Question 3: What is the difference between In-band (Classic) SQL Injection, Inferential (Blind) SQL Injection, and Out-of-Band SQL Injection?

- **Solution Approach:**
    
    - _In-band (Classic):_ The attacker uses the same channel of communication to launch the attack and gather results (e.g., displaying stolen database records directly on the web application page via `UNION` queries or raw database error messages).
    - _Inferential (Blind):_ No data or error messages are returned directly on the page. The attacker reconstructs data byte-by-byte by asking true/false questions and observing application behavior (Boolean-based) or injecting database delay functions like `pg_sleep()` or `WAITFOR DELAY` (Time-based).
    - _Out-of-Band:_ Used when server responses are asynchronous or unstable. The attacker crafts payloads that force the database server to make external network calls (e.g., DNS requests or HTTP requests via database extensions) to a server controlled by the attacker to exfiltrate data.

---

## Module 17: Dialect Fluency — Multi-Engine Comparison & Cheat Sheet

**Dialect Fluency** is the ability to write, translate, and optimize SQL code across major database engines—specifically **PostgreSQL**, **MySQL**, **SQL Server (T-SQL)**, and Cloud Data Warehouses (**Snowflake / Google BigQuery**). While ANSI SQL defines standard syntax rules, each engine implements proprietary extensions, functions, and configuration behaviors.

### Comprehensive Engine Comparison Reference

|**Feature / Operation**|**PostgreSQL**|**MySQL (8.0+)**|**SQL Server (T-SQL)**|**Snowflake / BigQuery**|
|---|---|---|---|---|
|**Row Limiting**|`LIMIT 10`|`LIMIT 10`|`SELECT TOP 10 ...`|`LIMIT 10` (Both)|
|**String Concatenation**|`col1 \| col2` or `CONCAT()`|`CONCAT(c1, c2)` or `c1 + c2`|`c1 + c2` or `CONCAT()`|`col1 \| col2` or `CONCAT()`|
|**Current Timestamp**|`CURRENT_TIMESTAMP` / `NOW()`|`NOW()` / `CURRENT_TIMESTAMP()`|`GETDATE()` / `SYSDATETIME()`|`CURRENT_TIMESTAMP()`|
|**NULL Substitution**|`COALESCE(c1, val)`|`IFNULL()` / `COALESCE()`|`ISNULL(c1, val)` / `COALESCE()`|`NVL()` / `COALESCE()`|
|**Auto-Increment ID**|`SERIAL` / `GENERATED ALWAYS AS IDENTITY`|`AUTO_INCREMENT`|`IDENTITY(1,1)`|`AUTOINCREMENT` (Snowflake) / `AUTO` (BQ)|
|**String Aggregation**|`STRING_AGG(col, ',')`|`GROUP_CONCAT(col SEPARATOR ',')`|`STRING_AGG(col, ',')`|`LISTAGG(col, ',')` (Snowflake) / `STRING_AGG()` (BQ)|
|**Safe Type Casting**|`CAST(col AS TYPE)` _(throws error on failure)_|`CAST` / `CONVERT`|`TRY_CAST(col AS TYPE)`|`TRY_CAST(col AS TYPE)`|
|**Identifier Quoting**|`"column_name"`|`` `column_name` ``|`[column_name]`|`"column_name"`|
|**Filter Window Results**|Requires Subquery / CTE|Requires Subquery / CTE|Requires Subquery / CTE|**`QUALIFY`** clause supported natively|

### Key Syntax Variations Deep-Dive

#### A. UPSERT Syntax

- **PostgreSQL:**

```sql
    INSERT INTO users (id, name) VALUES (1, 'Alice')
    ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
```

- **MySQL:**

```sql
    INSERT INTO users (id, name) VALUES (1, 'Alice')
    ON DUPLICATE KEY UPDATE name = VALUES(name);
```

- **SQL Server (T-SQL) / Snowflake:**
 
 ```sql
    MERGE INTO users AS target
    USING (SELECT 1 AS id, 'Alice' AS name) AS source
    ON target.id = source.id
    WHEN MATCHED THEN UPDATE SET target.name = source.name
    WHEN NOT MATCHED THEN INSERT (id, name) VALUES (source.id, source.name);
 ```

#### B. The `QUALIFY` Clause (Snowflake & BigQuery)

In standard SQL, you cannot filter window function results directly in the same `WHERE` clause because of logical execution order. Snowflake and BigQuery provide the `QUALIFY` clause to filter window function outputs cleanly:

```sql
SELECT 
    employee_id,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rk
FROM employees
QUALIFY rk = 1; -- Filters directly without needing an outer CTE wrapper
```

#### C. Safe Casting (`TRY_CAST`)

When cleaning dirty ETL text data, standard `CAST` fails the entire query if a row cannot be converted. `TRY_CAST` safely returns `NULL` on failure:

```sql
-- Returns NULL instead of throwing a syntax/conversion error if string is non-numeric
SELECT TRY_CAST('abc' AS INT); 
```

###  Interview Questions

#### Question 1: How does MySQL's string concatenation behavior differ from PostgreSQL and SQL Server when handling `NULL` values?

- **Solution Approach:**
    - In PostgreSQL and standard SQL, using the concatenation operator (`||`) with any `NULL` operand evaluates the entire expression to `NULL` (unless using the `CONCAT()` function which treats `NULL` as an empty string).
    - In MySQL, the `+` operator treats `NULL` mathematically (resulting in `NULL`), but MySQL's proprietary `CONCAT()` function automatically skips or converts `NULL` arguments to empty strings, whereas SQL Server's `+` operator evaluates to `NULL` if any input is `NULL`.

#### Question 2: What are the performance and architectural implications of migrating a large data pipeline from PostgreSQL's `ON CONFLICT` UPSERT mechanism to SQL Server's `MERGE` statement?

- **Solution Approach:**
    - `ON CONFLICT` in PostgreSQL is optimized specifically for single-table insert-or-update concurrency using index locks and snapshot isolation.
    - `MERGE` in SQL Server processes relational sources against target tables through execution plan joins. If the source dataset is large and lacks proper indexing on the join keys, `MERGE` can trigger costly Hash Joins or table scans, and historically suffered from race conditions under high concurrent concurrency unless serialized or isolated carefully.

#### Question 3: Why is the `QUALIFY` clause in Snowflake and BigQuery considered a major syntax improvement over traditional ANSI SQL when working with analytical window functions?

- **Solution Approach:**
    - In traditional ANSI SQL, to filter rows based on a window function result (e.g., finding the top 1 row per partition), you must wrap the query inside a CTE or derived subquery, then apply a `WHERE rk = 1` filter in the outer scope.
    - The `QUALIFY` clause evaluates window functions and applies filters in the same logical evaluation step, eliminating the need for boilerplate CTE nesting and making complex analytical queries significantly more readable and maintainable.

---

## Module 18: Performance Tuning & Diagnostic Architecture — Query Execution Plans & Join Strategies

The **Query Execution Planner** (or Optimizer) is the core database component that transforms a declarative SQL query into an optimized procedural execution plan. It evaluates statistical data, index availability, and table sizes to choose the most cost-effective access paths and join algorithms:

**Access Paths:**

- _Sequential Scan (Seq Scan):_ Reads the entire table block-by-block from disk/buffer cache.

- _Index Scan:_ Traverses a B-Tree index to find row pointers (`TIDs`), then fetches corresponding table blocks (risk of random I/O).

- _Bitmap Index Scan:_ Combines multiple index lookups into an in-memory bitmap of physical block locations, converting random disk reads into sequential block access.

**Join Algorithms:**

- _Nested Loop Join:_ Iterates through an outer table and searches the inner table for matching rows. Ideal for small datasets with indexes on the join key.

- _Hash Join:_ Builds an in-memory hash table from the smaller relation, then probes it with the larger relation. Highly efficient for large, unindexed datasets.

- _Merge Join (Sort-Merge):_ Sorts both input relations on the join key and merges them sequentially. Requires sorted inputs or pre-existing indexes.

