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



SQL runs in a different order than you write it:
**FROM** (which table) → **WHERE** (which rows) → **SELECT** (which columns) → **DISTINCT** (remove duplicate rows) → **LIMIT** (keep the first N rows)

So when read a question, ask in this order: _which table? which rows? which columns? duplicates? how many?_ That is the thinking pattern for every query this week.

**Question :** Show the name and city of all employees in the Engineering department.

1. Table: `employees`
2. Rows: only where `department = 'Engineering'`
3. Columns: `name`, `city`
4. Duplicates or limit: not asked

```sql
SELECT name, city
FROM employees
WHERE department = 'Engineering';

Result: Aarav–Pune, Rohan–Bengaluru, Isha–Pune.
```

**Question :** Which departments have employees working in Pune? Each department once.

1. Table: `employees`
2. Rows: `city = 'Pune'`
3. Column: `department`
4. Duplicates: Pune has two Engineering employees, so the words “each once” mean `DISTINCT`

```sql
SELECT department
FROM employees
WHERE city = 'Pune';

Result: Engineering, HR, Sales.
```

