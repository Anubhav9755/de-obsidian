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

