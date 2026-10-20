# SQL Notes (MySQL)

A topic-wise guide built from my practice session. Each section has the concept, the query, and a short explanation.

## Table of Contents

1. [Database Basics](#1-database-basics)
2. [Creating Tables and Inserting Data](#2-creating-tables-and-inserting-data)
3. [Basic SELECT Queries](#3-basic-select-queries)
4. [Filtering with WHERE](#4-filtering-with-where)
5. [Sorting, LIMIT and OFFSET](#5-sorting-limit-and-offset)
6. [Aggregate Functions](#6-aggregate-functions)
7. [GROUP BY and HAVING](#7-group-by-and-having)
8. [Conditional Logic: IF and CASE](#8-conditional-logic-if-and-case)
9. [ALTER TABLE (Changing Structure)](#9-alter-table-changing-structure)
10. [UPDATE and Safe Mode](#10-update-and-safe-mode)
11. [Constraints](#11-constraints)
12. [Joins and Foreign Keys](#12-joins-and-foreign-keys)
13. [UNION and UNION ALL](#13-union-and-union-all)
14. [Triggers](#14-triggers)
15. [Stored Procedures](#15-stored-procedures)
16. [Common Mistakes and Fixes](#16-common-mistakes-and-fixes)
17. [Quick Reference Cheat Sheet](#17-quick-reference-cheat-sheet)

---

## 1. Database Basics

```sql
SHOW DATABASES;          -- list all databases
CREATE DATABASE IF NOT EXISTS EMPLOYEES;
USE EMPLOYEES;           -- select the database to work in
SHOW TABLES;             -- list tables in the current database
```

- `IF NOT EXISTS` prevents an error if the database already exists.
- You must run `USE <db>` before working with tables in that database.
- Tables whose names contain special characters (like `student-data`) need backticks: `` SELECT * FROM `student-data`; ``

---

## 2. Creating Tables and Inserting Data

### Create a table

```sql
CREATE TABLE employees(
    namee      VARCHAR(50),
    email      VARCHAR(50),
    salary     INT,
    department VARCHAR(70)
);
```

### Insert one row

```sql
INSERT INTO employees(namee, email, salary, department)
VALUES ('KAIF', 'Kaif@xyz.com', 50000, 'IT');
```

### Insert multiple rows at once

```sql
INSERT INTO employees(namee, email, salary, department) VALUES
('Rahul', 'rahul@xyz.com', 45000, 'IT'),
('Sara',  'sara@xyz.com',  70000, 'CS'),
('Ali',   'ali@xyz.com',   55000, 'IT');
```

Multi-row inserts are faster and cleaner than many single `INSERT` statements.

### Drop a table

```sql
DROP TABLE employees;   -- permanently deletes the table and its data
```

---

## 3. Basic SELECT Queries

```sql
SELECT * FROM employees;                         -- all columns
SELECT namee, email FROM employees;              -- specific columns
SELECT namee, salary * 12 AS annual_salary
FROM employees;                                  -- calculated column with alias
SELECT salary AS employees_salary FROM employees;
SELECT DISTINCT department FROM employees ORDER BY department;
```

- `AS` creates an **alias** (a temporary name for a column or table).
- `DISTINCT` removes duplicate values from the result.
- Expressions such as `salary * 12` can be computed directly in `SELECT`.

---

## 4. Filtering with WHERE

```sql
-- Equality
SELECT * FROM employees WHERE salary = 50000;

-- Range (inclusive on both ends)
SELECT namee, salary FROM employees
WHERE salary BETWEEN 50000 AND 60000;

-- Multiple values
SELECT namee FROM employees WHERE department IN ('IT', 'CS');

-- Same result using OR
SELECT namee FROM employees
WHERE department = 'IT' OR department = 'CS';

-- Combining conditions
SELECT * FROM employees WHERE department = 'cs' AND salary > 45000;
```

### Pattern matching with LIKE

| Wildcard | Meaning                      |
|----------|------------------------------|
| `%`      | Zero or more characters      |
| `_`      | Exactly one character        |

```sql
SELECT * FROM employees WHERE namee LIKE 'A%';    -- starts with A
SELECT * FROM employees WHERE namee LIKE '%a_';   -- second-to-last letter is 'a'
```

---

## 5. Sorting, LIMIT and OFFSET

```sql
SELECT namee, salary FROM employees ORDER BY salary;        -- ascending (default)
SELECT namee, salary FROM employees ORDER BY salary DESC;   -- descending
```

### LIMIT and OFFSET (pagination)

```sql
SELECT * FROM employees LIMIT 3;              -- first 3 rows
SELECT * FROM employees LIMIT 10 OFFSET 3;    -- skip 3 rows, then return up to 10
```

### Useful patterns

```sql
-- 3rd lowest salary (skip 2, take 1)
SELECT namee, salary FROM employees ORDER BY salary LIMIT 1 OFFSET 2;

-- Top 3 highest-paid IT employees
SELECT namee, salary FROM employees
WHERE department = 'IT'
ORDER BY salary DESC
LIMIT 3;

-- Employees earning > 70000, highest first
SELECT namee, department, salary FROM employees
WHERE salary > 70000
ORDER BY salary DESC;
```

**Order of clauses:** `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT`

---

## 6. Aggregate Functions

Aggregate functions collapse many rows into a single value.

| Function  | Purpose                  |
|-----------|--------------------------|
| `COUNT()` | Number of rows           |
| `SUM()`   | Total                    |
| `AVG()`   | Average                  |
| `MAX()`   | Largest value            |
| `MIN()`   | Smallest value           |

```sql
SELECT COUNT(salary) FROM employees;   -- counts non-NULL salaries
SELECT COUNT(namee)  FROM employees;
SELECT SUM(salary)   FROM employees;
```

`COUNT(*)` counts all rows, while `COUNT(column)` ignores NULLs in that column.

---

## 7. GROUP BY and HAVING

`GROUP BY` splits rows into groups, and aggregates are calculated per group.

```sql
SELECT department, COUNT(*)      FROM employees GROUP BY department;
SELECT department, AVG(salary)   FROM employees GROUP BY department;
SELECT department, MAX(salary)   FROM employees GROUP BY department;
SELECT department, MIN(salary)   FROM employees GROUP BY department;
SELECT department, SUM(salary)   FROM employees GROUP BY department;
```

### HAVING: filter groups

`WHERE` filters **rows before** grouping. `HAVING` filters **groups after** aggregation.

```sql
-- Departments with more than 4 employees
SELECT department, COUNT(*) FROM employees
GROUP BY department
HAVING COUNT(*) > 4;

-- Departments whose total salary exceeds 230000
SELECT department, SUM(salary) FROM employees
GROUP BY department
HAVING SUM(salary) > 230000;

-- Departments with average salary above 50000
SELECT department, AVG(salary) FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### Combined example

```sql
-- Top 2 departments with avg salary > 40000, highest first
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 40000
ORDER BY avg_salary DESC
LIMIT 2;
```

---

## 8. Conditional Logic: IF and CASE

### IF(condition, value_if_true, value_if_false)

```sql
SELECT namee, salary,
       IF(salary >= 50000, 'High', 'Low') AS category
FROM employees;
```

### CASE (multiple conditions)

```sql
SELECT namee, department,
       CASE
           WHEN department = 'IT'  THEN 'Technical'
           WHEN department = 'cs'  THEN 'Human Resource'
           WHEN department = 'bio' THEN 'Finance Team'
           ELSE 'Other'
       END AS department_type
FROM employees;
```

- Use `IF()` for a simple two-way choice.
- Use `CASE` when there are more than two outcomes.
- MySQL string comparison is case-insensitive by default, so `'cs'` matches `'CS'`.

---

## 9. ALTER TABLE (Changing Structure)

```sql
-- Add a column
ALTER TABLE employees ADD age INT;

-- Drop a column
ALTER TABLE employees DROP COLUMN age;

-- Change a column's type and move it to the first position
ALTER TABLE employees MODIFY COLUMN department VARCHAR(50) FIRST;

-- Move a column after another column
ALTER TABLE employees MODIFY COLUMN namee VARCHAR(50) AFTER id;

-- Rename a column (MySQL 8+)
ALTER TABLE employees RENAME COLUMN old_name TO new_name;

-- Rename a table
RENAME TABLE old_table TO new_table;

-- Add a primary key
ALTER TABLE employees ADD PRIMARY KEY (id);

-- Make a column auto-increment
ALTER TABLE employees MODIFY id INT NOT NULL AUTO_INCREMENT;

-- View the full table definition
SHOW CREATE TABLE employees;
```

### Generating sequential IDs for existing rows

```sql
SET @i = 0;
UPDATE employees SET id = (@i := @i + 1);
```

`@i` is a user variable. It starts at 0 and increases by 1 for each row, which gives every row a unique number.

---

## 10. UPDATE and Safe Mode

```sql
SET SQL_SAFE_UPDATES = 0;   -- allow updates/deletes without a key-based WHERE
```

MySQL Workbench blocks `UPDATE` or `DELETE` statements that don't filter on a key column. Setting this to `0` disables that protection for the session.

```sql
UPDATE employees SET age = 24 WHERE department = 'IT';
UPDATE employees SET salary = 50000 WHERE namee = 'Rahul';
```

### Random numbers

```sql
UPDATE employees SET marks = FLOOR(20 + (RAND() * 11));
```

- `RAND()` returns a decimal between 0 and 1.
- `RAND() * 11` gives 0 to 10.99, and `20 + ...` shifts it to 20 to 30.99.
- `FLOOR()` rounds down, so the result is an integer from **20 to 30**.

**Warning:** always include a `WHERE` clause on `UPDATE` unless you really want to change every row.

---

## 11. Constraints

Constraints enforce rules on the data.

| Constraint    | Purpose                                      |
|---------------|----------------------------------------------|
| `PRIMARY KEY` | Unique and non-NULL identifier for each row  |
| `UNIQUE`      | No duplicate values (NULL allowed)           |
| `NOT NULL`    | Value must be provided                       |
| `DEFAULT`     | Value used when none is supplied             |
| `CHECK`       | Value must satisfy a condition               |
| `FOREIGN KEY` | Links to a primary key in another table      |

### PRIMARY KEY and UNIQUE

```sql
CREATE TABLE students (
    id    INT PRIMARY KEY,
    email VARCHAR(50) UNIQUE
);
```

### NOT NULL

```sql
CREATE TABLE students2 (
    id   INT,
    namee VARCHAR(50) NOT NULL
);

INSERT INTO students2 VALUES (1, 'kaif');   -- works
INSERT INTO students2 VALUES (2, NULL);     -- ERROR: namee cannot be NULL
```

### DEFAULT

```sql
CREATE TABLE hiring (
    id    INT,
    namee VARCHAR(50),
    city  VARCHAR(50) DEFAULT 'DELHI'
);

INSERT INTO hiring VALUES (1, 'kaif', 'Noida');          -- city = Noida
INSERT INTO hiring (id, namee) VALUES (3, 'Aman');       -- city = DELHI (default)
```

### CHECK

```sql
CREATE TABLE voting (
    id  INT,
    age INT CHECK (age >= 18)
);

INSERT INTO voting VALUES (1, 24);   -- works
INSERT INTO voting VALUES (2, 15);   -- ERROR: violates the CHECK constraint
INSERT INTO voting VALUES (3, 34);   -- works
```

---

## 12. Joins and Foreign Keys

### Setup: two related tables

```sql
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    emp_id   INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id  INT,
    salary   INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

The `FOREIGN KEY` guarantees that every `dept_id` in `employees` exists in `departments`.

Sample data used: 5 departments (IT, HR, Finance, Marketing, Sales) and 10 employees spread across them.

### Join types at a glance

| Join          | Returns                                                         |
|---------------|-----------------------------------------------------------------|
| `INNER JOIN`  | Only rows that match in both tables                             |
| `LEFT JOIN`   | All rows from the left table, plus matches from the right (else NULL) |
| `RIGHT JOIN`  | All rows from the right table, plus matches from the left (else NULL) |
| `CROSS JOIN`  | Every row of table A combined with every row of table B         |

### INNER JOIN

```sql
SELECT e.emp_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- With a filter
SELECT e.emp_name, e.salary, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > 50000;

-- Employees per department
SELECT d.dept_name, COUNT(e.emp_id) AS total_employees
FROM departments d
INNER JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;
```

`e` and `d` are table aliases, which keep queries short and readable.

### LEFT JOIN

```sql
-- Every employee, even if they have no department
SELECT e.emp_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### RIGHT JOIN

```sql
-- Every department, even if it has no employees
SELECT d.dept_name, e.emp_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;

-- Total salary per department; empty departments show 0
SELECT d.dept_name, COALESCE(SUM(e.salary), 0) AS total_salary
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id
GROUP BY d.dept_id, d.dept_name;

-- Highest salary per department
SELECT d.dept_name, MAX(e.salary) AS highest_salary
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id
GROUP BY d.dept_id, d.dept_name;
```

- `COALESCE(x, 0)` returns the first non-NULL value, so it replaces NULL with 0.

### Condition in ON vs WHERE

```sql
SELECT d.dept_name, COUNT(e.emp_id) AS total_employees
FROM employees e
RIGHT JOIN departments d
    ON e.dept_id = d.dept_id AND e.salary > 50000
GROUP BY d.dept_id, d.dept_name;
```

Putting `e.salary > 50000` in the `ON` clause keeps **all departments** in the result (with a count of 0 where nobody qualifies). Putting it in `WHERE` would remove those departments entirely.

### CROSS JOIN

```sql
SELECT e.emp_name, d.dept_name
FROM employees e
CROSS JOIN departments d;

SELECT COUNT(*) AS total_combinations
FROM employees CROSS JOIN departments;   -- 10 x 5 = 50
```

A cross join produces a **Cartesian product**: rows(A) × rows(B).

---

## 13. UNION and UNION ALL

Both combine the results of two `SELECT` statements. The queries must return the **same number of columns** with compatible types.

| Operator    | Duplicates                         |
|-------------|------------------------------------|
| `UNION`     | Removed (slower, sorts internally) |
| `UNION ALL` | Kept (faster)                      |

```sql
SELECT emp_name AS namee FROM employees
UNION
SELECT dept_name FROM departments;

SELECT emp_name FROM employees
UNION ALL
SELECT dept_name FROM departments;

-- With ordering (ORDER BY applies to the whole combined result)
SELECT emp_id AS allids FROM employees
UNION
SELECT dept_id FROM departments
ORDER BY allids DESC;
```

Column names in the result come from the **first** query.

---

## 14. Triggers

A **trigger** is code that runs automatically when an `INSERT`, `UPDATE`, or `DELETE` happens on a table. Here it logs every salary change.

### Step 1: audit table

```sql
CREATE TABLE SalaryAudit (
    audit_id   INT AUTO_INCREMENT PRIMARY KEY,
    emp_id     INT,
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Step 2: trigger

```sql
DELIMITER //

CREATE TRIGGER AfterSalaryUpdate
AFTER UPDATE ON Employees
FOR EACH ROW
BEGIN
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO SalaryAudit (emp_id, old_salary, new_salary)
        VALUES (OLD.emp_id, OLD.salary, NEW.salary);
    END IF;
END //

DELIMITER ;
```

- `DELIMITER //` temporarily changes the statement terminator so the `;` inside the body doesn't end the statement early.
- `AFTER UPDATE` means it fires after the row is updated.
- `FOR EACH ROW` means it runs once per affected row.
- `OLD` holds the values before the update, and `NEW` holds the values after.
- `<>` means "not equal".

### Step 3: test

```sql
UPDATE Employees SET salary = 65000 WHERE emp_id = 102;
SELECT * FROM SalaryAudit;   -- shows old (40000) and new (65000) salary
```

---

## 15. Stored Procedures

A **stored procedure** is a saved, reusable block of SQL that you run with `CALL`.

### Procedure with an input parameter

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeeById(IN p_id INT)
BEGIN
    SELECT * FROM Employees WHERE emp_id = p_id;
END //

DELIMITER ;

CALL GetEmployeeById(105);
```

### Procedure with a variable

```sql
DELIMITER //

CREATE PROCEDURE CalculateBonus(IN p_id INT)
BEGIN
    DECLARE v_salary DECIMAL(10,2);

    SELECT salary INTO v_salary
    FROM Employees
    WHERE emp_id = p_id;

    SELECT v_salary AS Salary,
           v_salary * 0.10 AS Bonus;   -- 10% bonus
END //

DELIMITER ;

CALL CalculateBonus(103);
```

- `IN` marks an input parameter.
- `DECLARE` creates a local variable.
- `SELECT ... INTO` stores a query result into a variable.

---

## 16. Common Mistakes and Fixes

| Issue in the notes | Why it is a problem | Fix |
|---|---|---|
| `SELECT namee FROM employees GROUP BY salary;` | Selecting a non-grouped, non-aggregated column fails under `ONLY_FULL_GROUP_BY` mode | Select the grouped column: `SELECT salary, COUNT(*) FROM employees GROUP BY salary;` |
| `ALTER TABLE employees RENAME COLUMN sal TO salary;` | Column `sal` doesn't exist in this table | Use the real current column name |
| `RENAME TABLE student TO employees;` | Fails if `student` doesn't exist, or `employees` already exists | Check with `SHOW TABLES;` first |
| `INSERT INTO students2 VALUES (2, NULL);` | Violates `NOT NULL` (this was intentional to demo the error) | Provide a real value |
| `INSERT INTO voting VALUES (2, 15);` | Violates `CHECK (age >= 18)` (intentional demo) | Use age 18 or above |
| `UPDATE employees SET salary = 4500.00 ...` | `salary` is `INT`, so decimals get rounded | Use `DECIMAL` if you need decimals |
| `ADD age INT` run twice | Error: duplicate column name | Drop the column first, or run it once |
| `CALL GetEmployeeById(105)` with no `;` | Statement may not execute or can merge with the next one | Always end with `;` |
| Creating `employees` again after `DROP TABLE` | Works, but it's a new table with a different structure | Make sure to recreate it intentionally |
| `UPDATE` without `WHERE` | Changes **every** row | Always filter, and test with `SELECT` first |

---

## 17. Quick Reference Cheat Sheet

```sql
-- Query structure
SELECT   columns / aggregates
FROM     table
JOIN     other_table ON condition
WHERE    row_filter
GROUP BY grouping_columns
HAVING   group_filter
ORDER BY column [ASC|DESC]
LIMIT    n OFFSET m;
```

| Task                       | Syntax                                        |
|----------------------------|-----------------------------------------------|
| Unique values              | `SELECT DISTINCT col`                         |
| Range                      | `col BETWEEN a AND b`                         |
| Match a list               | `col IN (a, b, c)`                            |
| Pattern                    | `col LIKE 'A%'`                               |
| Replace NULL               | `COALESCE(col, 0)`                            |
| Two-way condition          | `IF(cond, a, b)`                              |
| Multi-way condition        | `CASE WHEN ... THEN ... ELSE ... END`         |
| Combine results            | `UNION` / `UNION ALL`                         |
| Add / remove column        | `ALTER TABLE t ADD / DROP COLUMN c`           |
| Auto log changes           | `CREATE TRIGGER ... AFTER UPDATE ...`         |
| Reusable logic             | `CREATE PROCEDURE ... CALL ...`               |

---

*Notes compiled from hands-on MySQL practice: DDL, DML, filtering, aggregation, joins, constraints, triggers and stored procedures.*
