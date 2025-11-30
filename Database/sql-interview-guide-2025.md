# SQL Interview Guide - Frequently Asked Questions 2025

---

## Table of Contents
1. [DELETE vs TRUNCATE vs DROP](#1-delete-vs-truncate-vs-drop)
2. [Types of Joins](#2-types-of-joins)
3. [WHERE vs HAVING](#3-where-vs-having)
4. [Primary Key vs Unique Key vs Foreign Key](#4-primary-key-vs-unique-key-vs-foreign-key)
5. [Find Duplicate Records](#5-find-duplicate-records)
6. [Find Nth Highest Salary](#6-find-nth-highest-salary)
7. [UNION vs UNION ALL vs INTERSECT vs MINUS](#7-union-vs-union-all-vs-intersect-vs-minus)
8. [Aggregate Functions](#8-aggregate-functions)
9. [GROUP BY with ROLLUP, CUBE, GROUPING SETS](#9-group-by-with-rollup-cube-grouping-sets)
10. [Subqueries](#10-subqueries)
11. [Views](#11-views)
12. [Indexes](#12-indexes)
13. [NULL Handling](#13-null-handling)
14. [CASE Statement](#14-case-statement)
15. [Window Functions - Ranking](#15-window-functions---ranking)
16. [Window Functions - Analytic](#16-window-functions---analytic)
17. [Transactions & ACID](#17-transactions--acid)
18. [Normalization](#18-normalization)
19. [Common Query Patterns](#19-common-query-patterns)
20. [Quick Reference - Common Functions](#20-quick-reference---common-functions)

---

## 1. DELETE vs TRUNCATE vs DROP

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| WHERE clause | ✅ Yes | ❌ No | ❌ No |
| Rollback | ✅ Yes | ❌ No | ❌ No |
| Triggers fired | ✅ Yes | ❌ No | ❌ No |
| Speed | Slow | Fast | Fast |
| Space released | ❌ No | ✅ Yes | ✅ Yes |
| Table exists after | ✅ Yes | ✅ Yes | ❌ No |
| Resets identity | ❌ No | ✅ Yes | N/A |

```sql
-- DELETE: Removes specific rows, can rollback
DELETE FROM employees WHERE dept_id = 10;

-- TRUNCATE: Removes all rows, keeps structure, faster
TRUNCATE TABLE employees;

-- DROP: Removes entire table including structure
DROP TABLE employees;
DROP TABLE employees CASCADE CONSTRAINTS;  -- Also drops dependent objects
```

---

## 2. Types of Joins

```
INNER JOIN           LEFT JOIN            RIGHT JOIN           FULL JOIN
  ┌───┬───┐           ┌───┬───┐            ┌───┬───┐           ┌───┬───┐
  │ A │███│ B         │███│███│ B          │ A │███│███│       │███│███│███│
  └───┴───┘           └───┴───┘            └───┴───┘           └───┴───┘
  Only matching       All A +              All B +             All rows
  rows                matching B           matching A          from both
```

### INNER JOIN
Returns only matching rows from both tables.

```sql
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from left table + matching rows from right table.

```sql
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
Returns all rows from right table + matching rows from left table.

```sql
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### FULL OUTER JOIN
Returns all rows from both tables.

```sql
SELECT e.name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

### SELF JOIN
Table joined to itself.

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

### CROSS JOIN
Cartesian product - every combination of rows.

```sql
SELECT e.name, d.dept_name
FROM employees e
CROSS JOIN departments d;
```

---

## 3. WHERE vs HAVING

| Feature | WHERE | HAVING |
|---------|-------|--------|
| When executed | Before GROUP BY | After GROUP BY |
| Works on | Individual rows | Groups |
| Aggregate functions | ❌ Cannot use | ✅ Can use |
| Use with | Any query | GROUP BY queries |

```sql
-- WHERE: Filters rows BEFORE grouping
-- HAVING: Filters groups AFTER grouping

SELECT dept_id, COUNT(*) AS emp_count, AVG(salary) AS avg_salary
FROM employees
WHERE status = 'ACTIVE'        -- Filters individual rows first
GROUP BY dept_id
HAVING COUNT(*) > 5;           -- Filters groups after aggregation
```

### SQL Execution Order
```
1. FROM
2. WHERE (filter rows)
3. GROUP BY
4. HAVING (filter groups)
5. SELECT
6. ORDER BY
7. LIMIT/FETCH
```

---

## 4. Primary Key vs Unique Key vs Foreign Key

| Feature | Primary Key | Unique Key | Foreign Key |
|---------|-------------|------------|-------------|
| Purpose | Identifies row uniquely | Ensures unique values | Links tables |
| NULL allowed | ❌ No | ✅ Yes (one NULL) | ✅ Yes |
| Duplicates | ❌ No | ❌ No | ✅ Yes |
| Per table | Only 1 | Multiple | Multiple |
| Index created | Clustered (auto) | Non-clustered (auto) | No (manual) |

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,                    -- Primary Key
    email VARCHAR(100) UNIQUE,                 -- Unique Key
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)  -- Foreign Key
);
```

---

## 5. Find Duplicate Records

### Find duplicates
```sql
SELECT name, email, COUNT(*)
FROM employees
GROUP BY name, email
HAVING COUNT(*) > 1;
```

### Find duplicate rows with details
```sql
SELECT *
FROM employees
WHERE email IN (
    SELECT email
    FROM employees
    GROUP BY email
    HAVING COUNT(*) > 1
);
```

### Delete duplicates (keep one with lowest ID)
```sql
DELETE FROM employees
WHERE id NOT IN (
    SELECT MIN(id)
    FROM employees
    GROUP BY email
);
```

### Oracle: Delete duplicates using ROWID
```sql
DELETE FROM employees
WHERE ROWID NOT IN (
    SELECT MIN(ROWID)
    FROM employees
    GROUP BY email
);
```

---

## 6. Find Nth Highest Salary

### 2nd Highest Salary - Subquery Method
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

### Nth Highest Salary - DENSE_RANK Method (Recommended)
```sql
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) WHERE rank = 2;  -- Change to N for Nth highest
```

### Nth Highest Salary - LIMIT/OFFSET Method
```sql
-- MySQL / PostgreSQL
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;  -- For 2nd highest (N-1)

-- Oracle 12c+
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
OFFSET 1 ROWS FETCH NEXT 1 ROWS ONLY;
```

### Top 3 Salaries Per Department
```sql
SELECT * FROM (
    SELECT dept_id, name, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank
    FROM employees
) WHERE rank <= 3;
```

---

## 7. UNION vs UNION ALL vs INTERSECT vs MINUS

| Operation | Duplicates | Performance |
|-----------|------------|-------------|
| UNION | Removes | Slower |
| UNION ALL | Keeps | Faster |
| INTERSECT | Common rows only | Medium |
| MINUS/EXCEPT | Difference | Medium |

### UNION - Removes duplicates
```sql
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

### UNION ALL - Keeps duplicates (faster)
```sql
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

### INTERSECT - Only rows present in both
```sql
SELECT name FROM employees
INTERSECT
SELECT name FROM contractors;
```

### MINUS (Oracle) / EXCEPT (SQL Server)
Rows in first query but not in second.

```sql
SELECT name FROM employees
MINUS
SELECT name FROM contractors;
```

---

## 8. Aggregate Functions

```sql
SELECT 
    COUNT(*) AS total_rows,           -- Count all rows
    COUNT(commission) AS with_comm,   -- Count non-NULL values
    COUNT(DISTINCT dept_id) AS depts, -- Count distinct values
    SUM(salary) AS total_salary,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees;
```

> **Important:** Aggregate functions ignore NULL values (except COUNT(*))

---

## 9. GROUP BY with ROLLUP, CUBE, GROUPING SETS

### Basic GROUP BY
```sql
SELECT dept_id, job_id, SUM(salary)
FROM employees
GROUP BY dept_id, job_id;
```

### ROLLUP - Hierarchical subtotals
```sql
SELECT dept_id, job_id, SUM(salary)
FROM employees
GROUP BY ROLLUP(dept_id, job_id);
-- Returns: (dept, job), (dept, NULL), (NULL, NULL)
```

### CUBE - All possible combinations
```sql
SELECT dept_id, job_id, SUM(salary)
FROM employees
GROUP BY CUBE(dept_id, job_id);
-- Returns: (dept, job), (dept, NULL), (NULL, job), (NULL, NULL)
```

### GROUPING SETS - Specific combinations
```sql
SELECT dept_id, job_id, SUM(salary)
FROM employees
GROUP BY GROUPING SETS(
    (dept_id, job_id),
    (dept_id),
    ()
);
```

---

## 10. Subqueries

### Scalar Subquery - Returns single value
```sql
SELECT name, salary,
       (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees;
```

### Single-row Subquery
```sql
SELECT * FROM employees
WHERE dept_id = (SELECT dept_id FROM departments WHERE name = 'IT');
```

### Multi-row Subquery - Use IN, ANY, ALL
```sql
SELECT * FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'NYC');
```

### ANY - Greater than minimum value in list
```sql
SELECT * FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE dept_id = 10);
```

### ALL - Greater than maximum value in list
```sql
SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE dept_id = 10);
```

### Correlated Subquery - References outer query
```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id
);
```

### EXISTS - Check if rows exist
```sql
SELECT d.dept_name
FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.dept_id);
```

---

## 11. Views

| Feature | View | Table |
|---------|------|-------|
| Stores data | ❌ No (query definition) | ✅ Yes |
| Takes space | ❌ No | ✅ Yes |
| Can insert/update | Sometimes* | ✅ Yes |
| Performance | Query executes each time | Direct access |

*Simple views without joins/aggregates can be updatable

### Create View
```sql
CREATE VIEW active_employees AS
SELECT emp_id, name, salary, dept_id
FROM employees
WHERE status = 'ACTIVE';
```

### Use View
```sql
SELECT * FROM active_employees;
```

### Create or Replace
```sql
CREATE OR REPLACE VIEW active_employees AS
SELECT emp_id, name, salary, dept_id, hire_date
FROM employees
WHERE status = 'ACTIVE';
```

### Drop View
```sql
DROP VIEW active_employees;
```

---

## 12. Indexes

### Create Index
```sql
CREATE INDEX idx_emp_name ON employees(last_name);
```

### Composite Index
```sql
CREATE INDEX idx_emp_dept_sal ON employees(dept_id, salary);
```

### Unique Index
```sql
CREATE UNIQUE INDEX idx_emp_email ON employees(email);
```

### Drop Index
```sql
DROP INDEX idx_emp_name;
```

### When to use indexes:
- ✅ Columns in WHERE clause
- ✅ Columns in JOIN conditions
- ✅ Columns in ORDER BY
- ✅ High-cardinality columns

### When NOT to use:
- ❌ Small tables
- ❌ Frequently updated columns
- ❌ Low-cardinality columns

---

## 13. NULL Handling

### Check for NULL
```sql
SELECT * FROM employees WHERE commission IS NULL;
SELECT * FROM employees WHERE commission IS NOT NULL;
```

### NVL (Oracle)
```sql
SELECT NVL(commission, 0) FROM employees;
```

### ISNULL (SQL Server)
```sql
SELECT ISNULL(commission, 0) FROM employees;
```

### IFNULL (MySQL)
```sql
SELECT IFNULL(commission, 0) FROM employees;
```

### COALESCE - Returns first non-NULL value (ANSI SQL)
```sql
SELECT COALESCE(phone, mobile, email, 'N/A') FROM contacts;
```

### NULLIF - Returns NULL if values are equal
```sql
SELECT NULLIF(current_salary, previous_salary) FROM employees;
```

### NVL2 (Oracle) - Different values for NULL vs NOT NULL
```sql
SELECT NVL2(commission, 'Has Comm', 'No Comm') FROM employees;
```

> **Important NULL Rules:**
> - `NULL = NULL` → NULL (not TRUE!)
> - `NULL <> NULL` → NULL (not TRUE!)
> - Always use `IS NULL` or `IS NOT NULL`

---

## 14. CASE Statement

### Simple CASE
```sql
SELECT name,
       CASE status
           WHEN 'A' THEN 'Active'
           WHEN 'I' THEN 'Inactive'
           WHEN 'T' THEN 'Terminated'
           ELSE 'Unknown'
       END AS status_desc
FROM employees;
```

### Searched CASE (with conditions)
```sql
SELECT name, salary,
       CASE
           WHEN salary < 30000 THEN 'Low'
           WHEN salary BETWEEN 30000 AND 70000 THEN 'Medium'
           WHEN salary > 70000 THEN 'High'
           ELSE 'Unknown'
       END AS salary_band
FROM employees;
```

### CASE in ORDER BY
```sql
SELECT * FROM employees
ORDER BY CASE WHEN status = 'A' THEN 1
              WHEN status = 'I' THEN 2
              ELSE 3
         END;
```

### CASE in Aggregation
```sql
SELECT 
    SUM(CASE WHEN status = 'A' THEN 1 ELSE 0 END) AS active_count,
    SUM(CASE WHEN status = 'I' THEN 1 ELSE 0 END) AS inactive_count
FROM employees;
```

---

## 15. Window Functions - Ranking

```sql
SELECT 
    emp_id,
    name,
    salary,
    
    -- ROW_NUMBER: Unique sequential number
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    
    -- RANK: Same rank for ties, gaps after
    RANK() OVER (ORDER BY salary DESC) AS rank,
    
    -- DENSE_RANK: Same rank for ties, no gaps
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
    
FROM employees;
```

### Example output for salaries: 100, 100, 90, 80

| salary | row_num | rank | dense_rank |
|--------|---------|------|------------|
| 100 | 1 | 1 | 1 |
| 100 | 2 | 1 | 1 |
| 90 | 3 | 3 | 2 |
| 80 | 4 | 4 | 3 |

### Ranking within partitions
```sql
SELECT dept_id, name, salary,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dept_rank
FROM employees;
```

---

## 16. Window Functions - Analytic

```sql
SELECT 
    emp_id,
    salary,
    dept_id,
    
    -- Running total
    SUM(salary) OVER (ORDER BY emp_id) AS running_total,
    
    -- Total per department
    SUM(salary) OVER (PARTITION BY dept_id) AS dept_total,
    
    -- LAG: Previous row value
    LAG(salary) OVER (ORDER BY emp_id) AS prev_salary,
    
    -- LEAD: Next row value
    LEAD(salary) OVER (ORDER BY emp_id) AS next_salary,
    
    -- FIRST_VALUE / LAST_VALUE
    FIRST_VALUE(salary) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS highest_in_dept

FROM employees;
```

### LAG and LEAD with offset and default
```sql
SELECT 
    emp_id,
    salary,
    LAG(salary, 1, 0) OVER (ORDER BY emp_id) AS prev_salary,    -- 1 row back, default 0
    LAG(salary, 2, 0) OVER (ORDER BY emp_id) AS prev_2_salary,  -- 2 rows back
    LEAD(salary, 1, 0) OVER (ORDER BY emp_id) AS next_salary    -- 1 row forward
FROM employees;
```

---

## 17. Transactions & ACID

### Transaction Example
```sql
BEGIN TRANSACTION;  -- or START TRANSACTION

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;    -- Save changes
-- or
ROLLBACK;  -- Undo changes
```

### SAVEPOINT
```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
ROLLBACK TO sp1;  -- Undo only second update
COMMIT;
```

### ACID Properties

| Property | Description |
|----------|-------------|
| **A**tomicity | All or nothing - complete transaction or rollback |
| **C**onsistency | Database remains in valid state |
| **I**solation | Concurrent transactions don't interfere |
| **D**urability | Committed changes are permanent |

---

## 18. Normalization

### 1NF (First Normal Form)
- Eliminate duplicate columns
- Each column contains atomic (indivisible) values
- Each row is unique

### 2NF (Second Normal Form)
- Must be in 1NF
- No partial dependencies (non-key columns depend on full primary key)

### 3NF (Third Normal Form)
- Must be in 2NF
- No transitive dependencies (non-key columns depend only on primary key)

### BCNF (Boyce-Codd Normal Form)
- Must be in 3NF
- Every determinant is a candidate key

### Example - Unnormalized to 3NF

**Unnormalized:**
```
Orders(OrderID, CustomerName, CustomerCity, Product1, Product2, Product3)
```

**1NF (remove repeating groups):**
```
Orders(OrderID, CustomerName, CustomerCity, ProductName)
```

**2NF (remove partial dependencies):**
```
Orders(OrderID, CustomerID, ProductName)
Customers(CustomerID, CustomerName, CustomerCity)
```

**3NF (remove transitive dependencies):**
```
Orders(OrderID, CustomerID)
OrderItems(OrderID, ProductID)
Customers(CustomerID, CustomerName, CityID)
Cities(CityID, CityName)
Products(ProductID, ProductName)
```

---

## 19. Common Query Patterns

### Find Employees Without Department
```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;

-- OR using NOT EXISTS
SELECT * FROM employees e
WHERE NOT EXISTS (SELECT 1 FROM departments d WHERE d.dept_id = e.dept_id);
```

### Find Departments Without Employees
```sql
SELECT d.*
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;
```

### Employees Earning More Than Manager
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

### Employees Earning More Than Department Average
```sql
-- Using correlated subquery
SELECT e.name, e.salary, e.dept_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id
);

-- Using window function
SELECT * FROM (
    SELECT name, salary, dept_id,
           AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
    FROM employees
) WHERE salary > dept_avg;
```

### Count by Multiple Conditions
```sql
SELECT 
    COUNT(*) AS total,
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_count,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_count,
    SUM(CASE WHEN salary > 50000 THEN 1 ELSE 0 END) AS high_earners
FROM employees;
```

### Year-over-Year Comparison
```sql
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    SUM(amount) AS total_sales,
    LAG(SUM(amount)) OVER (ORDER BY EXTRACT(YEAR FROM order_date)) AS prev_year,
    SUM(amount) - LAG(SUM(amount)) OVER (ORDER BY EXTRACT(YEAR FROM order_date)) AS diff
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date);
```

### Find Consecutive Values
```sql
-- Find employees hired on consecutive days
SELECT e1.name, e1.hire_date, e2.name AS next_hire, e2.hire_date
FROM employees e1
JOIN employees e2 ON e2.hire_date = e1.hire_date + 1;
```

---

## 20. Quick Reference - Common Functions

### String Functions

| Function | Example | Result |
|----------|---------|--------|
| LENGTH | `LENGTH('Hello')` | 5 |
| UPPER | `UPPER('hello')` | HELLO |
| LOWER | `LOWER('HELLO')` | hello |
| SUBSTR | `SUBSTR('Hello', 1, 3)` | Hel |
| TRIM | `TRIM('  Hello  ')` | Hello |
| REPLACE | `REPLACE('Hello', 'l', 'x')` | Hexxo |
| CONCAT | `CONCAT('Hello', ' World')` | Hello World |
| INSTR | `INSTR('Hello', 'l')` | 3 |
| LPAD | `LPAD('Hi', 5, '*')` | ***Hi |
| RPAD | `RPAD('Hi', 5, '*')` | Hi*** |

### Date Functions

| Function | Description | Database |
|----------|-------------|----------|
| SYSDATE | Current date/time | Oracle |
| GETDATE() | Current date/time | SQL Server |
| NOW() | Current date/time | MySQL |
| ADD_MONTHS(date, n) | Add months | Oracle |
| DATEADD(month, n, date) | Add months | SQL Server |
| TRUNC(date, 'MM') | First day of month | Oracle |
| EXTRACT(YEAR FROM date) | Get year | ANSI SQL |
| DATEDIFF(d1, d2) | Days between dates | MySQL/SQL Server |
| MONTHS_BETWEEN(d1, d2) | Months between | Oracle |

### Conversion Functions

| Function | Example | Description |
|----------|---------|-------------|
| TO_CHAR | `TO_CHAR(date, 'YYYY-MM-DD')` | Date to string (Oracle) |
| TO_DATE | `TO_DATE('2024-01-15', 'YYYY-MM-DD')` | String to date (Oracle) |
| TO_NUMBER | `TO_NUMBER('123')` | String to number (Oracle) |
| CAST | `CAST(value AS datatype)` | ANSI SQL conversion |
| CONVERT | `CONVERT(datatype, value)` | SQL Server |

### Date Format Codes (Oracle)

| Code | Meaning | Example |
|------|---------|---------|
| YYYY | 4-digit year | 2024 |
| MM | Month (01-12) | 01 |
| DD | Day (01-31) | 15 |
| HH24 | Hour (00-23) | 14 |
| MI | Minute (00-59) | 30 |
| SS | Second (00-59) | 45 |
| DAY | Day name | MONDAY |
| MONTH | Month name | JANUARY |
| Q | Quarter | 1 |
| WW | Week of year | 03 |

---

## Summary Cheat Sheet

```
DDL: CREATE, ALTER, DROP, TRUNCATE (auto-commit)
DML: SELECT, INSERT, UPDATE, DELETE (can rollback)
DCL: GRANT, REVOKE
TCL: COMMIT, ROLLBACK, SAVEPOINT

Execution Order:
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

JOINs:
• INNER: Only matching rows
• LEFT:  All left + matching right
• RIGHT: All right + matching left
• FULL:  All rows from both
• CROSS: Cartesian product
• SELF:  Table joined to itself

NULL Handling:
• IS NULL, IS NOT NULL (never use = NULL)
• COALESCE(a, b, c) - first non-NULL
• NVL(a, b) - Oracle specific

Ranking Functions:
• ROW_NUMBER: 1, 2, 3, 4 (unique, no ties)
• RANK:       1, 1, 3, 4 (ties allowed, gaps)
• DENSE_RANK: 1, 1, 2, 3 (ties allowed, no gaps)

Common Patterns:
• Find duplicates: GROUP BY + HAVING COUNT(*) > 1
• Nth highest: DENSE_RANK() or subquery
• Running total: SUM() OVER (ORDER BY ...)
• Previous/Next row: LAG() / LEAD()
• Anti-join: LEFT JOIN + WHERE key IS NULL

Index Best Practices:
• Create on WHERE, JOIN, ORDER BY columns
• Use composite index for multi-column queries
• Avoid on frequently updated columns
• Consider cardinality (high = good for B-tree)

Transaction ACID:
• Atomicity: All or nothing
• Consistency: Valid state maintained
• Isolation: No interference
• Durability: Permanent after commit
```

---

*Good luck with your SQL interview!*
