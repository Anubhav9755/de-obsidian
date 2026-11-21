## Lesson 1: Reading data from a table (`SELECT` and `FROM`)

- **`SELECT`**: which columns do you want?
- **`FROM`**: which table are they in?

```sql
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    name       VARCHAR(50),
    department VARCHAR(30),
    city       VARCHAR(30),
    salary     INT,
    hire_date  DATE
);

INSERT INTO employees VALUES
(1,  'Aarav',  'Engineering', 'Pune',      85000, '2021-06-15'),
(2,  'Diya',   'Sales',       'Delhi',     52000, '2022-02-10'),
(3,  'Rohan',  'Engineering', 'Bengaluru', 95000, '2020-11-01'),
(4,  'Meera',  'HR',          'Pune',      48000, '2023-01-20'),
(5,  'Kabir',  'Sales',       'Mumbai',    61000, '2021-09-05'),
(6,  'Isha',   'Engineering', 'Pune',      72000, '2022-07-18'),
(7,  'Vikram', 'Sales',       'Delhi',     54000, '2023-03-12'),
(8,  'Anaya',  'HR',          'Mumbai',    50000, '2022-05-30'),
(9,  'Neha',   'Marketing',   'Bengaluru', 58000, '2023-08-01'),
(10, 'Arjun',  'Sales',       'Pune',      47000, '2020-03-25');
(11, 'Sara',   'Marketing',    NULL,       55000, '2023-09-10'),
(12, 'Dev',    'HR',          'Delhi',     NULL,  '2023-10-05');
```


**Question 1:** Show everything in the employees table. `(all columns)`

```sql
SELECT *
FROM employees;
```

**Question 2:** Show only each employee’s name and department. `(chosen columns)`

```sql
SELECT name, department
FROM employees;
```

**Question 3:** Show each employee’s name and a 10% bonus on their salary, with the bonus column labelled `bonus`. `(calculating and renaming)`

```sql
SELECT name, salary*0.10 AS bonus
FROM employees;
```

Aarav’s row shows `8500.00` under `bonus`. The calculation affects only the output, not the stored table.

The bonus isn’t stored in the table, so we calculate it from `salary`. SQL can do arithmetic (`+ - * /`) inside `SELECT`. `AS` gives the result a readable name (an alias).

**Question 4:** Show the name and hire date of every employee.

```sql
SELECT name, hire_date
FROM employees;
```

**Question 5:** Show each employee’s name and their annual salary . Label the column `annual_salary`

```sql
SELECT name, salary*12 AS annual_salary
FROM employees;
```

---

## Lesson 2: Filtering rows (`WHERE`)

`WHERE` chooses **rows**. It keeps only the rows where a condition is true.

```sql
SELECT columns
FROM table
WHERE condition;
```

The order SQL actually runs these in is **FROM** (find the table), then **WHERE** (keep matching rows), then **SELECT** (pick columns). That’s why you can filter on a column you don’t display.

### Comparison operators

|Operator|Meaning|
|---|---|
|`=`|equal to (just one `=` in SQL)|
|`<>` or `!=`|not equal to|
|`>` / `<`|greater / less than|
|`>=` / `<=`|greater or equal / less or equal|

Two formatting rules:

- Text goes in **single quotes**: `'Sales'`. Text comparisons are case-sensitive in PostgreSQL, so `'sales'` won’t match `'Sales'`.
- Numbers have **no quotes**: `salary > 60000`.
- Dates go in single quotes, as `'YYYY-MM-DD'`.

**Question 6:** Show the name and salary of employees earning more than 60000.

```sql
SELECT name, salary
FROM employees
WHERE salary > 6000;

Result: Aarav (85000), Rohan (95000), Kabir (61000), Isha (72000).
```

**Question 7:** Show the names of employees who work in Pune.

```sql
SELECT name
FROM employees
WHERE city ='Pune';

Result: Aarav, Meera, Isha, Arjun.
```

**Question 8:** Show the names of employees who joined on or after 1 January 2022.

```sql
SELECT name
FROM employees
WHERE hire_date >= '2022-01-01';
```

**Question 9:** Show the name and department of every employee who is **not** in the Sales department.

```sql
SELECT name, department
FROM employees
WHERE department <> 'Sales';
```

**Question 10:** Show the name, city and salary of employees earning **at least** 50000.

```sql
SELECT name, city, salary
FROM employees
WHERE salary >= 50000;
```

---

## Lesson 3: Combining conditions (`AND`, `OR`, `NOT`)

- **`AND`**: both conditions must be true.
- **`OR`**: at least one condition must be true.
- **`NOT`**: flips a condition (`NOT city = 'Pune'` is the same as `city <> 'Pune'`).

**Question 11:** Show the names of Engineering employees who work in Pune.

```sql
SELECT name
FROM employees
WHERE department = 'Engineering' AND city = 'Pune';

Result: Aarav, Isha.
```

AND narrows results whereas OR widens the result.

**Question 12:** Show the name and department of employees in Sales or HR.

```sql
SELECT name, department
FROM employees
WHERE department = 'Sales' OR department = 'HR';

Result: Diya, Meera, Kabir, Vikram, Anaya, Arjun.
```

**Question 13:** Show the names of employees who work in Pune or Delhi and earn more than 50000.

```sql
SELECT name
FROM employees
WHERE (city = 'Pune' OR city = 'Delhi') AND salary > 50000;

Result: Aarav, Diya, Isha, Vikram.
```

SQL evaluates `AND` before `OR`.

**The habit to build:** whenever you mix `AND` and `OR`, add parentheses to show exactly what you mean, even when they aren’t strictly required. Then check the output against a few rows you can verify by eye.

**Question 14:** Show the name and salary of Engineering employees earning more than 80000.

```sql
SELECT name, salary
FROM employees
WHERE department = 'Engineering' AND salary > 80000;
```

**Question 15:** Show the name, department and city of employees who work in Mumbai or Delhi and were hired on or after 2022-01-01.

```sql
SELECT name, department, city
FROM employees
WHERE (city = 'Mumbai' OR city = 'Delhi') AND hire_date >= '22-01-01';
```

---
## Lesson 4: Shortcuts for ranges and lists (`IN`, `BETWEEN`)

- **`IN (...)`** checks whether a value matches any item in a list. It replaces a chain of `OR`s on the same column.
- **`BETWEEN a AND b`** checks whether a value is in a range. It includes both ends, so it’s the same as `>= a AND <= b`.

Add `NOT` in front of either to flip it: `NOT IN`, `NOT BETWEEN`.

**Question 16:** Show the names of employees who work in Pune or Mumbai.

```sql
SELECT name
FROM employees
WHERE city IN ('Mumbai','Pune');

Result: 6 rows (Aarav, Meera, Kabir, Isha, Anaya, Arjun)
```


**Question 17:** Show the name and salary of employees earning between 50000 and 60000.

```sql
SELECT name, salary
FROM employee
WHERE salary BETWEEN 50000 AND 60000;

Result: Diya (52000), Vikram (54000), Anaya (50000), Neha (58000).
```

Always put the smaller value first: `BETWEEN 60000 AND 50000` returns nothing.

**Question 18:** Show the names of employees hired during 2022.

```sql
SELECT name
FROM employees
WHERE hire_date BETWEEN '22-01-01' AND '22-12-31';

Result: Diya, Isha, Anaya.
```

**Question 19:** Show the names of employees who are not in Sales or HR.

```sql
SELECT name
FROM employees
WHERE department NOT IN ('Sales','HR');

Result: Aarav, Rohan, Isha, Neha.
```

**The habit to build:** when notice the same column compared to several values, reach for `IN`. When you see “from X to Y” or “between”, reach for `BETWEEN`, and remember both ends are included.

**Question 20:** Show the name and city of employees who work in Delhi, Mumbai or Bengaluru. Use `IN`.

```sql
SELECT name, city
FROM employees
WHERE city IN ('Delhi','Mumbai','Bengaluru'); 

result - 6 rows (Diya, Vikram, Kabir, Anaya, Rohan, Neha)
```

**Question 21:** Show the name and salary of employees earning between 55000 and 90000, **and** who work in Engineering or Marketing. 

```sql
SELECT name, salary
FROM employees
WHERE salary 55000 BETWEEN 90000 AND department IN ('Engineering','Marketing');
```

**Question 22:** Show the name, department and hire date of employees hired between 2021-01-01 and 2021-12-31 who work in Sales or Engineering.

```sql
SELECT name, department, hire_date
FROM employees
WHERE hire_date 2021-01-01 BETWEEN 2021-12-31 AND department IN ('Engineering','Sales');
```

---

## Lesson 5: Matching text patterns (`LIKE`) 

Only know part of the text: “names that start with A” or “cities containing ‘ba’”. `LIKE` matches a pattern, and the pattern uses two wildcards:

|Wildcard|Meaning|
|---|---|
|`%`|any number of characters (including none)|
|`_`|exactly one character|

Two things to remember:

- `LIKE` in PostgreSQL is **case-sensitive**. `ILIKE` ignores case. In MySQL and SQL Server, `LIKE` is already case-insensitive by default.
- `NOT LIKE` flips the match.


**Question 23:** Show the names of employees whose name starts with ‘A’.

```SQL
SELECT name
FROM employees
WHERE name LIKE 'A%';

Result: Aarav, Anaya, Arjun.
```

**Question 24:** Show the names of employees whose name anywhere with ‘a’.

```sql
SELECT name
FROM employees
WHERE name LIKE '%a%';

Result: Diya, Meera, Isha, Anaya, Neha, Aarav
```

**Question 24:** Show the names of employees whose second letter is ‘a’.

```sql
SELECT name
FROM employees
WHERE name LIKE '_a%';

Result: Aarav, Kabir.
```

**Question 26.** Show the name and department of employees whose name starts with ‘A’ and who are **not** in the Sales department.

```sql
SELECT name, department
FROM employees
WHERE name LIKE 'A%' AND department <> 'Sales';

Result: Aarav, Anaya
```

---
### Lesson 6: Removing duplicates (`DISTINCT`)

A table can have many rows with the same value. If you select `department` from `employees`, you get 10 rows, with “Sales” appearing 4 times. Often you only want the **unique** values. Put `DISTINCT` right after `SELECT`.

```sql
SELECT DISTINCT column
FROM table;
```

**Key rule:** `DISTINCT` looks at the **entire row of selected columns**, not just the first column. If you select two columns, a row is a duplicate only when **both** values match another row.

**Question 27:** List the departments that exist in the company, each once.

```sql
SELECT DISTINCT department
FROM employees;

Result: Engineering, Sales, HR, Marketing (4 rows, in no guaranteed order)
```


**Question 28:** List each unique combination of department and city.

```sql
SELECT DISTINCT department, city
FROM employees;
```

This returns 8 rows, not 10. Only two combinations repeat: Engineering in Pune (Aarav and Isha) and Sales in Delhi (Diya and Vikram). Notice that Engineering appears 3 times in the table, but “Engineering + Pune” and “Engineering + Bengaluru” are different combinations, so both survive.

**Question 29:** In which cities do Sales employees work? Each city once.

```sql
SELECT DISTINCT city
FROM employees
WHERE department = 'Sales';

Result: Delhi, Mumbai, Pune (Delhi would appear twice without `DISTINCT`).
```

**Words that signal `DISTINCT`:** “unique”, “each once”, “different”, “without repetition”, “list the departments/cities that...”.

**Question 30.** List the unique department and city combinations for employees hired on or after 2021-01-01.

```sql
SELECT DISTINCT city,department
FROM employees
WHERE hire_date >= 2021-01-01;
```

---
### Lesson 7: Sorting and limiting (`ORDER BY`, `LIMIT`)

Rows come back in no guaranteed order unless you ask. `ORDER BY` sorts them, and `LIMIT` keeps only the first N rows of the sorted result.

```sql
SELECT columns 
FROM table 
WHERE condition 
ORDER BY column [ASC|DESC]
LIMIT n;
```

- `ASC` is ascending (smallest first) and is the default. `DESC` is descending (largest first).
- `ORDER BY` comes after `WHERE`; `LIMIT` goes last.
- `LIMIT` without `ORDER BY` gives you arbitrary rows, so always sort first when you want “top” or “first”.

**Question 31:** Show employee names and salaries, highest salary first.

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;

Result: Rohan (95000), Aarav (85000), Isha (72000).
```

**Question 32:** Show the 3 highest-paid employees.

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;

Result: Rohan, Aarav, Isha.
```

**Question 33:** Show name, department and salary, sorted by department A to Z, and within each department by salary highest first.

```sql
SELECT name, department, salary
FROM employees
ORDER BY department ASC, salary DESC
```

Engineering comes first (Rohan, Aarav, Isha), then HR, Marketing, Sales. `ASC` or `DESC` applies to the column right before it, so each column needs its own direction.

**Question 34** Show the name and hire date of the 4 most recently hired employees.

```sql
SELECT name, hire_date
FROM employees
ORDER BY hire_date ASC
LIMIT 4;
```

**Question 35.** Show the name, city and salary of employees in Pune or Delhi, sorted by city A to Z, and within each city by salary highest first.

```sql
SELECT name, city, salary
FROM employees
WHERE city IN ('Pune','Delhi')
ORDER BY city ASC, salary DESC;
```

---

### Lesson 8: Missing values (`NULL`)

`NULL` means **no value / unknown**. It is not zero and not an empty string. A salary of `NULL` means “we don’t know the salary,” not “the salary is 0.”

Because NULL means “unknown,” normal comparisons don’t work with it:

- `salary = NULL` is never true, even for NULL rows. It returns 0 rows.
- To test for NULL, use **`IS NULL`** and **`IS NOT NULL`**.

**Question 36 :** Show the name of employees whose city is missing.

```sql
SELECT name
FROM employees
WHERE city IS NULL;

Result: Sara
```

**Question 37:** Show the name and salary of employees whose salary is recorded.

```sql
SELECT name, salary
FROM employees
WHERE salary IS NOT NULL;

Result: 11 rows
```

**Question 38:** Show the names of employees who are not in Pune.

```sql
SELECT name
FROM employees
WHERE city <> 'Pune' OR city IS NULL;

Result: Now it returns 8 rows, including Sara.
```

**Question 39.** Show the name and department of employees whose salary is missing.

```sql
SELECT name, department
FROM employees
WHERE salary IS NULL;
```

**Question 40.** Show the name and city of employees who are not in Delhi, **including** those with no city recorded.

```sql
SELECT name, city 
FROM employees
WHERE city <> 'Delhi' OR city IS NULL;
```

---
## Lesson 9: Summarising data (aggregate functions)

Until now, every query returned individual rows. Aggregate functions **collapse 
many rows into one summary value**:

|Function|What it gives|
|---|---|
|`COUNT(*)`|number of rows|
|`COUNT(column)`|number of rows where that column is **not NULL**|
|`SUM(column)`|total|
|`AVG(column)`|average|
|`MIN(column)` / `MAX(column)`|smallest / largest value|

**Important:** `SUM`, `AVG`, `MIN`, `MAX` and `COUNT(column)` all **ignore NULLs**. Only `COUNT(*)` counts every row.

**Question 41:** How many employees are there, and how many have a recorded salary?

```SQL
SELECT COUNT(*) AS total_employees,
       COUNT(salary) AS with_salary
FROM employees;

Result: 12 and 11
```

**Question 42:** Show the highest, lowest and average salary in the company.

```sql
SELECT MAX(salary) AS HIGHEST,
		MIN(salary) AS LOWEST,
		AVG(salary) AS AVERAGE
FROM employees;

Result: 95000, 47000, 61545.45
```

**Question 43:** What is the average salary of Sales employees?

```sql
SELECT AVERAGE(salary) AS avg_salary
FROM employees
WHERE department = 'Sales';

Result: 53500
```

**Question 44:** How many different departments are there?

```sql
SELECT COUNT(DISTINCT department) AS dept_count
FROM employees;

Result: 4
```

**One rule for now:** when you aggregate, the result has **one row**. You can’t mix an aggregate with a normal column (like `SELECT name, MAX(salary)`) yet. That needs `GROUP BY`.

**Question 45:** How many employees are there in total, and how many have a recorded city? (Show both in one query.)

```sql
SELECT COUNT(*) as total_emp,
		COUNT(city) AS with_city
FROM employees
```

**Question 46:** Show the total salary and the average salary of employees in the HR department. (Look closely at the HR rows before you predict the average.)

```sql
SELECT SUM(salary), AVG(salary)
FROM employees
WHERE department = 'HR';
```

---

### Lesson 10: Summarising per group (`GROUP BY`)

Last lesson, an aggregate gave **one answer for the whole table**. But most real questions are “per” questions: the average salary **per department**, the number of employees **per city**. `GROUP BY` splits the rows into groups and runs the aggregate **once per group**.

```SQL
SELECT group_column, AGGREGATE(column)
FROM table
GROUP BY group_column;
```

**The rule that causes most errors:** every column in `SELECT` that is _not_ inside an aggregate must also be in `GROUP BY`. The words “per”, “for each” and “by” in a question are your signal to use it.

**Question 47:** How many employees are in each department?

```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department;

Result: Engineering 3, Sales 4, HR 3, Marketing 2
```

**Question 48:** Show the average salary of each department.

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

Result: Engineering 84000, Sales 53500, HR 49000, Marketing 56500
```

**Question 49:** Show the number of employees in each city, largest first.

```sql
SELECT city, COUNT(*) AS emp_count
FROM employees
GROUP BY city
ORDER BY emp_count DESC;
```

Result: Pune 4, Delhi 3, Bengaluru 2, Mumbai 2, and then a row with an empty city and a count of 1. NULLs don’t disappear in `GROUP BY`; they form their own group (that’s Sara). `ORDER BY` can use the alias, since it runs after `SELECT`.

**Question 50.** Show each city and the highest salary in that city.

```sql
SELECT city, MAX(salary) AS highest_salary
FROM employees
GROUP BY city;

RESULT - Pune 85000, Delhi 54000, Bengaluru 95000, Mumbai 61000, and an empty city (Sara) 55000.
```

**Question 51.** Show each department and the number of employees hired on or after 2022-01-01. (Think about the order: which rows should be removed before the grouping happens?)

```sql
SELECT department, COUNT(*) AS recent_hires
FROM employees
WHERE hire_date >= '2022-01-01'
GROUP BY department;

Result: Sales 2, HR 3, Engineering 1, Marketing 2
```

**Question 52:** For each department, how many employees earn more than 50000?

```sql
SELECT department, COUNT(*) AS total_emp
FROM employees
WHERE salary > 50000; 
GROUP BY department

Result: Engineering 3, Sales 3, Marketing 2, which is 3 rows
```

**Question 53.** Show each department and its lowest salary.

```sql
SELECT department, MIN(salary) AS min_salary
FROM employees
GROUP BY department;
```

`RESULT:` 4 rows (Engineering 72000, Sales 47000, HR 48000, Marketing 55000). HR is 48000 because Dev’s NULL salary is ignored.

**Question 54.** Show each city and the number of employees hired **before** 2022-01-01, with the city that has the most such employees first.

```sql
SELECT city, COUNT(*) AS new_joinee
FROM employees
WHERE hire_date < '2022-01-01'
GROUP BY city
ORDER BY new_joinee DESC;
```

---

## Lesson 11: Filtering groups (HAVING)

WHERE filters rows before grouping. But sometimes you want to filter groups based on their result, like “departments with more than 2 employees.” The count only exists after grouping, so WHERE can’t see it. That’s what HAVING is for.

	WHERE	HAVING
Filters	individual rows	whole groups
Runs	before GROUP BY	after GROUP BY
Can use aggregates?	No	Yes

Full order of execution: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT.

**Here are the three patterns side by side, using the same table.**

**Pattern 1:** only WHERE (filter people, then summarise)

**Question 55:** For employees hired on or after 2022-01-01, how many are in each department?

```sql
SELECT department, COUNT(*) AS recent_hires
FROM employees
WHERE hire_date >= '2022-01-01'
GROUP BY department;
```

Result: 4 rows (Sales 2, HR 3, Engineering 1, Marketing 2). Every group is shown, because nothing tests the group’s result. The only condition is about a single person’s hire date.

**Pattern 2:** only HAVING (summarise everyone, then filter groups)

**Question 56:** Which departments have more than 2 employees?

```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

Result: 3 rows (Engineering 3, Sales 4, HR 3). Marketing has 2, so it fails. Every employee is counted, so there’s no WHERE. The only condition is about a group’s count.

**Pattern 3:** both (filter people, summarise, filter groups)

**Question 57:** Among employees hired on or after 2021-01-01, which departments have an average salary above 55000?

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= '2021-01-01'
GROUP BY department
HAVING AVG(salary) > 55000;
```

**Question 58:** Which departments have more than 2 employees?

One row per? Department.
Calculate? Count of employees.
Filter rows first? No.
Filter groups? Yes: count > 2.
Expected rows? Engineering 3, Sales 4, HR 3 pass; Marketing (2) fails. 3 rows.

```SQL
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

**Question 59:** Show departments whose average salary is above 55000.

```SQL
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 55000;
```

**Question 60:** Among employees hired on or after 2022-01-01, which departments have at least 2 such employees?

```sql
SELECT department, COUNT(*) AS total_employee
FROM employees
WHERE hire_date >= '2022-01-01'
GROUP BY department
HAVING COUNT(*) >=2;
```

**Question 61:** Show each city and its employee count, but only cities with more than 1 employee.

```sql
select city, count(*) as emp_count
from employees
group by city
having count(*) > 1;
Returns 4 rows: Pune 4, Delhi 3, Bengaluru 2, Mumbai 2
```

**Question 62:** Show each department and its highest salary, but only departments where the highest salary is at least 60000.

```sql
select department, max(salary) as max_salary
from employees
group by department
having max_salary >= 60000
Returns 2 rows: Engineering 95000 and Sales 61000
```

**Question 63:** Considering only employees hired on or after 2021-01-01, show each department’s average salary, but only departments whose average is above 55000.

```sql
select department, avg(salary) as avg_salary
fromemployees
where hire_date >= '2021-01-01'
group by department
having avg(salary) > 55000;
```

**Classify:** label each as WHERE only, HAVING only, or both.

a) Number of employees per department, counting only employees in Pune.
- WHERE city = 'Pune' then GROUP BY department gives Engineering 2, HR 1, Sales 1.

b) Cities whose total salary is above 120000.
- Group by city, then HAVING SUM(salary) > 120000 gives Pune (252000) and Bengaluru (153000).
  
c) For employees hired before 2022-01-01, each department’s maximum salary, but only departments where that maximum exceeds 80000.
- The hire date is a row filter and the max is a group test.

**Question 64:** Considering only employees who earn more than 50000, show each city that has at least 2 such employees, along with the count.

```sql
select city, count(*) as emp_count
from employees
where salary > 50000
group by city
having count(*) >= 2;
```

---

## Lesson 12: Conditional logic (CASE WHEN)

CASE WHEN is SQL’s if/else. It checks conditions in order and returns a value for the first one that is true. It’s how you create labels like “High / Medium / Low” or count rows under a condition.

```sql
CASE
	WHEN condn1 THEN result1
	WHEN condn2 THEN result2
	ELSE default_result
END
```

Two rules:
- Order matters. The first true WHEN wins, and the rest are ignored.
- Without an ELSE, rows that match nothing get NULL.

Question 65: Show each employee’s name, salary and a salary band: High (80000 or more), Medium (55000 up to 80000), Low (below 55000), and Unknown if the salary is missing.

```SQL
SELECT name, salary
	CASE
		WHEN salary IS NULL THEN 'Unknown'
		WHEN salary >= 80000 THEN 'High'
		WHEN salary >= 55000 THEN 'Medium'
		ELSE 'Low'
	END AS salary_band
FROM employees;
```

**Question 66:** For each department, show the total employees and how many of them earn 60000 or more.

Thinking:
- One row per? Department.
- Calculate: a total count, plus a count of only some rows.
- We can’t use WHERE, because it would remove the other employees from the total. So we count conditionally: CASE gives 1 to people who qualify and 0 to the rest, then SUM adds up the 1s.

```sql
SELECT department,
	COUNT(*) AS total_emp
	SUM(CASE WHEN salary >= 60000 THEN 1 ELSE 0 END) AS high_earner
FROM employees
GROUP BY department;
``
```




