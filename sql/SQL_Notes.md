---
tags:
---

# SQL Notes


SELECT: Tells the database which columns you want to look at.

FROM: Tells the database which table contains those columns.

LIMIT - Tells the database exactly how many rows of data to show.

```sql
SELECT first_name, country 
FROM customers
LIMIT 2;
```

DISTINCT - Removes duplicate values from your output. It gives you a list of unique values.

```sql
SELECT DISTINCT country
FROM customers;
```

WHERE - It filters rows based on specific conditions.

EX- "Show me only customers from India" or "Show me transactions over $100"

```sql
SELECT first_name, country, total_spent
        FROM customers
        WHERE country = 'India' and total_spent > 100.00;
```

LOGIC OPERATORS - AND, OR, NOT

```sql
SELECT *
    FROM customers
    WHERE country = 'INDIA' OR total_spent > 200.00;
```

IN - instead of writing "WHERE country = 'INDIA' OR country = 'India', it can group into a simple list using parentheses;

```sql
SELECT *
        FROM customers
        Where country IN('USA','UK');
```

BETWEEN -   Instead of "WHERE total_spent >= 100 AND total_spent <= 300",  use BETWEEN to target continuous range.

```sql
SELECT *
        FROM customers
        WHERE total_spent BETWEEN 100 AND 300;
```

Condition - Find all customers whose names start with the letter 'A'" or "Find all email addresses ending in '.com'

LIKE - relies on two special placeholder symbols:

% (Percent sign): Represents zero, one, or multiple characters.

_ (Underscore): Represents exactly one single character.

NAME STARTS WITH 'A'-

```sql
SELECT first_name, country
        FROM customers
        WHERE first_name LIKE 'A%';
```

NAME ENDS WITH 'A'

```sql
SELECT * 
FROM customers 
WHERE first_name LIKE '_a%';
```

ANYWHERE INSIDE THE TEXT-

```sql
SELECT *
        FROM customers
            WHERE first_name LIKE '%n%';
```

SECOND LETTER MUST BE 'A'-

```sql
SELECT * 
FROM customers 
WHERE country LIKE '__d%';
```

IS NULL or IS NOT NULL - Finds rows where data is completely missing or present.

```sql
SELECT *
        FROM customers
        WHERE total_spent IS NULL;
```

```sql
SELECT *
        FROM customers
        WHERE total_spent IS NOT NULL;
```

ORDER BY - Tells the database which column to use for sorting the rows.

ASC (Ascending): Sorts numbers from smallest to largest, or text from A to Z. This is the default if you don't specify.

DESC (Descending): Sorts numbers from largest to smallest, or text from Z to A.

Rule (Order): It always goes after the WHERE clause, but before the LIMIT clause.

```sql
SELECT *
        FROM customers
        ORDER BY total_spent DESC;
```

COUNT(): Counts how many rows contain data.

SUM(): Adds up all the numeric values in a column.

AVG(): Calculates the mathematical average (mean) of a column.

MAX(): Finds the highest value in a column.

MIN(): Finds the lowest value in a column.

Never mix * (or any raw column name) with an aggregate function (SUM, AVG, COUNT) unless you are grouping the data.

```sql
SELECT *, 
       SUM(total_spent) OVER() AS total_revenue,
       AVG(total_spent) OVER() AS average_spend,
       MAX(total_spent) OVER() AS highest_purchase,
       MIN(total_spent) OVER() AS lowest_purchase
FROM customers;
```

what if so many thing are happening inside and where we have to use * as well as SUM and other function. so what i do there

1. Keep Every Row and Add Totals Next to Them -

 Use a Window Function. Do this by adding the keyword OVER() right after your aggregate function. This tells the database: "Calculate the sum, but don't collapse my table."

```sql
SELECT *, 
       SUM(total_spent) OVER() AS company_total,
       MAX(total_spent) OVER() AS max_spent
FROM customers;
```

Scenario B: Collapse Rows into Categorical Buckets

If you don't care about seeing individual people like Amit or Sarah, but you want to see the totals broken down by Country, you drop the * and use GROUP BY.

```sql
SELECT country, 
       COUNT(*) AS total_customers
FROM customers
GROUP BY country;
```

If you need individual rows (*) + Totals  ->  Use OVER().

If you need categorical rows + Totals ->  Use GROUP BY.

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
    - Range Partitioning: Routes rows based on numeric or date ranges (e.g., monthly/yearly metrics).

    - List Partitioning: Routes rows based on discrete key values (e.g., region IN ('US-EAST', 'US-WEST')).

    - Hash Partitioning: Distributes rows across $N$ partitions using a modulo hash on keys (e.g., balancing tenant workloads evenly across nodes).
    
    - Primary Key / Unique Constraint Requirement: Unique and primary key constraints on a partitioned table must include all columns specified in the partition key.
    
    - Over-Partitioning Anti-Pattern: Avoid creating too many small partitions (e.g., partitioning by minute or second). Thousands of tiny partitions overhead the query planner during execution planning, consuming memory and degrading overall query throughput.

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

```
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

```
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

```
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

```
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

```
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

