# SQL Cheat Sheet

> Dialect notes are called out throughout: **MySQL**, **MSSQL** (SQL Server), **Oracle**, and **PostgreSQL** where relevant.

---

## Table of Contents

1. [Database & Table Basics](#1-database--table-basics)
2. [Data Types (Dialect Comparison)](#2-data-types-dialect-comparison)
3. [CRUD — Insert, Select, Update, Delete](#3-crud--insert-select-update-delete)
4. [Filtering, Sorting, Limiting](#4-filtering-sorting-limiting)
5. [Joins](#5-joins)
6. [Aggregation & Grouping](#6-aggregation--grouping)
7. [Subqueries](#7-subqueries)
8. [Set Operations](#8-set-operations)
9. [Constraints & Keys](#9-constraints--keys)
10. [Indexes](#10-indexes)
11. [Views](#11-views)
12. [Common Table Expressions (CTEs) & Recursion](#12-common-table-expressions-ctes--recursion)
13. [Window Functions](#13-window-functions)
14. [Transactions & Concurrency](#14-transactions--concurrency)
15. [Stored Procedures & Functions](#15-stored-procedures--functions)
16. [Triggers](#16-triggers)
17. [String, Date & Null Functions (Dialect Comparison)](#17-string-date--null-functions-dialect-comparison)
18. [Pagination (Dialect Comparison)](#18-pagination-dialect-comparison)
19. [JSON Support](#19-json-support)
20. [Pivoting Data](#20-pivoting-data)
21. [Query Execution Order](#21-query-execution-order)
22. [Performance & EXPLAIN](#22-performance--explain)
23. [Security Basics](#23-security-basics)

---

## 1. Database & Table Basics

```sql
CREATE DATABASE company;
USE company;                         -- MySQL / MSSQL
-- Oracle has no USE; you connect to a schema/user directly instead of switching DBs.

CREATE TABLE employees (
    id          INT PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    salary      DECIMAL(10,2),
    dept_id     INT,
    hire_date   DATE DEFAULT CURRENT_DATE
);

ALTER TABLE employees ADD COLUMN email VARCHAR(100);
ALTER TABLE employees DROP COLUMN email;
ALTER TABLE employees RENAME COLUMN first_name TO fname;   -- MySQL 8+/PostgreSQL syntax

DROP TABLE employees;
TRUNCATE TABLE employees;            -- fast delete-all, resets identity, minimal logging
```

| Task | MySQL | MSSQL | Oracle |
|---|---|---|---|
| Rename column | `ALTER TABLE t RENAME COLUMN a TO b;` (8+) | `EXEC sp_rename 't.a', 'b', 'COLUMN';` | `ALTER TABLE t RENAME COLUMN a TO b;` |
| Rename table | `RENAME TABLE t TO t2;` | `EXEC sp_rename 't', 't2';` | `ALTER TABLE t RENAME TO t2;` |
| Show tables | `SHOW TABLES;` | `SELECT * FROM sys.tables;` | `SELECT table_name FROM user_tables;` |
| Describe table | `DESCRIBE t;` / `SHOW COLUMNS FROM t;` | `sp_help t;` or query `INFORMATION_SCHEMA.COLUMNS` | `DESC t;` |

---

## 2. Data Types (Dialect Comparison)

| Category | MySQL | MSSQL | Oracle | PostgreSQL |
|---|---|---|---|---|
| Auto-increment integer | `INT AUTO_INCREMENT` | `INT IDENTITY(1,1)` | `NUMBER` + `IDENTITY` (12c+) or sequence | `SERIAL` / `GENERATED ALWAYS AS IDENTITY` |
| Variable text | `VARCHAR(n)` | `VARCHAR(n)` / `NVARCHAR(n)` (unicode) | `VARCHAR2(n)` | `VARCHAR(n)` |
| Large text | `TEXT` | `VARCHAR(MAX)` | `CLOB` | `TEXT` |
| Boolean | `BOOLEAN` (alias of `TINYINT(1)`) | `BIT` (0/1) | `NUMBER(1)` (no native boolean pre-23c) | `BOOLEAN` |
| Date/time | `DATETIME`, `TIMESTAMP` | `DATETIME2`, `DATETIMEOFFSET` | `DATE` (has time!), `TIMESTAMP` | `TIMESTAMP`, `TIMESTAMPTZ` |
| Decimal | `DECIMAL(p,s)` | `DECIMAL(p,s)` / `NUMERIC(p,s)` | `NUMBER(p,s)` | `NUMERIC(p,s)` |
| JSON | `JSON` | `NVARCHAR(MAX)` + `JSON` functions (or native `JSON` type SQL Server 2025+) | `JSON` (21c+) / `BLOB`/`CLOB` before | `JSON` / `JSONB` |

---

## 3. CRUD — Insert, Select, Update, Delete

```sql
-- INSERT
INSERT INTO employees (id, first_name, last_name, salary, dept_id)
VALUES (1, 'Ada', 'Lovelace', 95000, 10);

INSERT INTO employees (id, first_name, last_name, salary, dept_id)
VALUES (2, 'Alan', 'Turing', 98000, 10),
       (3, 'Grace', 'Hopper', 91000, 20);

-- SELECT
SELECT first_name, last_name FROM employees;
SELECT * FROM employees;

-- UPDATE
UPDATE employees SET salary = salary * 1.1 WHERE dept_id = 10;

-- DELETE
DELETE FROM employees WHERE id = 3;
```

### Upsert (Insert-or-Update)

| Dialect | Syntax |
|---|---|
| MySQL | `INSERT ... ON DUPLICATE KEY UPDATE col = VALUES(col);` |
| MSSQL | `MERGE` statement (see below) |
| Oracle | `MERGE` statement |
| PostgreSQL | `INSERT ... ON CONFLICT (id) DO UPDATE SET col = EXCLUDED.col;` |

```sql
-- MERGE (MSSQL / Oracle standard form)
MERGE INTO employees AS tgt
USING (SELECT 1 AS id, 100000 AS salary) AS src
ON tgt.id = src.id
WHEN MATCHED THEN UPDATE SET tgt.salary = src.salary
WHEN NOT MATCHED THEN INSERT (id, salary) VALUES (src.id, src.salary);
```

---

## 4. Filtering, Sorting, Limiting

```sql
SELECT * FROM employees
WHERE dept_id = 10 AND salary > 90000;

SELECT * FROM employees WHERE dept_id IN (10, 20);
SELECT * FROM employees WHERE salary BETWEEN 90000 AND 100000;
SELECT * FROM employees WHERE last_name LIKE 'H%';       -- starts with H
SELECT * FROM employees WHERE email IS NULL;

SELECT * FROM employees ORDER BY salary DESC, last_name ASC;
```

### Row Limiting Differences

| Goal | MySQL | MSSQL | Oracle (12c+) |
|---|---|---|---|
| Top N rows | `SELECT * FROM t LIMIT 10;` | `SELECT TOP 10 * FROM t;` | `SELECT * FROM t FETCH FIRST 10 ROWS ONLY;` |
| Offset + limit | `LIMIT 10 OFFSET 20;` | `ORDER BY ... OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;` | `OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;` |

---

## 5. Joins

```sql
-- INNER JOIN
SELECT e.first_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT (OUTER) JOIN
SELECT e.first_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- RIGHT JOIN
SELECT e.first_name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;

-- FULL OUTER JOIN
SELECT e.first_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- SELF JOIN
SELECT e1.first_name AS employee, e2.first_name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;

-- CROSS JOIN (Cartesian product)
SELECT * FROM employees CROSS JOIN departments;
```

> **MySQL note:** No native `FULL OUTER JOIN` — emulate with `LEFT JOIN UNION RIGHT JOIN`.
> **Oracle legacy syntax:** `WHERE e.dept_id = d.id(+)` for outer join (avoid; use ANSI `JOIN`).

```sql
-- MySQL FULL OUTER JOIN emulation
SELECT e.first_name, d.dept_name FROM employees e LEFT JOIN departments d ON e.dept_id = d.id
UNION
SELECT e.first_name, d.dept_name FROM employees e RIGHT JOIN departments d ON e.dept_id = d.id;
```

---

## 6. Aggregation & Grouping

```sql
SELECT dept_id, COUNT(*) AS headcount, AVG(salary) AS avg_salary, MAX(salary), MIN(salary), SUM(salary)
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5
ORDER BY avg_salary DESC;
```

- `WHERE` filters rows **before** grouping.
- `HAVING` filters groups **after** aggregation.

### Grouping Extensions

```sql
-- Subtotals + grand totals
SELECT dept_id, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP (dept_id, job_title);      -- MySQL 8+, MSSQL, Oracle all support ROLLUP

SELECT dept_id, job_title, SUM(salary)
FROM employees
GROUP BY CUBE (dept_id, job_title);        -- MSSQL, Oracle; MySQL lacks CUBE
```

---

## 7. Subqueries

```sql
-- Scalar subquery
SELECT first_name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- IN subquery
SELECT * FROM employees WHERE dept_id IN (SELECT id FROM departments WHERE region = 'EU');

-- EXISTS
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- Correlated subquery
SELECT e.first_name, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.dept_id = e.dept_id);

-- Derived table (subquery in FROM)
SELECT dept_id, avg_sal
FROM (SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id) AS dept_avg
WHERE avg_sal > 80000;
```

---

## 8. Set Operations

```sql
SELECT name FROM current_employees
UNION                      -- removes duplicates
SELECT name FROM former_employees;

SELECT name FROM current_employees
UNION ALL                  -- keeps duplicates, faster
SELECT name FROM former_employees;

SELECT name FROM current_employees
INTERSECT
SELECT name FROM contractors;

SELECT name FROM current_employees
EXCEPT                     -- MySQL: use MINUS-style workaround or MySQL 8.0.31+ supports EXCEPT
SELECT name FROM contractors;
```

> **Oracle** uses `MINUS` instead of `EXCEPT`. **MySQL** added `INTERSECT`/`EXCEPT` only from 8.0.31+; earlier versions need `NOT IN`/`NOT EXISTS` workarounds.

---

## 9. Constraints & Keys

```sql
CREATE TABLE departments (
    id        INT PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE employees (
    id        INT PRIMARY KEY,
    dept_id   INT,
    salary    DECIMAL(10,2) CHECK (salary >= 0),
    CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each row (implies `NOT NULL` + `UNIQUE`) |
| `FOREIGN KEY` | Enforces referential integrity to another table |
| `UNIQUE` | No duplicate values allowed |
| `NOT NULL` | Column cannot be empty |
| `CHECK` | Enforces a boolean condition per row |
| `DEFAULT` | Value used when none supplied |

---

## 10. Indexes

```sql
CREATE INDEX idx_dept ON employees (dept_id);
CREATE UNIQUE INDEX idx_email ON employees (email);
CREATE INDEX idx_dept_salary ON employees (dept_id, salary);   -- composite index
DROP INDEX idx_dept ON employees;          -- MySQL
DROP INDEX idx_dept;                       -- MSSQL / Oracle (index names are schema/table-scoped differently)
```

| Index type | Notes |
|---|---|
| B-Tree (default) | Good for equality & range queries; supported everywhere |
| Clustered | Determines physical row order — **MSSQL/MySQL(InnoDB)**: 1 per table. Oracle: implemented as Index-Organized Table (IOT) |
| Non-clustered / Secondary | Separate structure pointing to row location |
| Full-text | `FULLTEXT` (MySQL), `CONTAINS`/`FREETEXT` (MSSQL), Oracle Text (Oracle) |
| Bitmap | Oracle-specific, great for low-cardinality columns in data warehouses |

---

## 11. Views

```sql
CREATE VIEW high_earners AS
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 90000;

SELECT * FROM high_earners;

CREATE OR REPLACE VIEW high_earners AS ...   -- MySQL, Oracle, PostgreSQL
-- MSSQL: use ALTER VIEW instead of CREATE OR REPLACE

DROP VIEW high_earners;
```

**Materialized views** (store pre-computed results physically):

```sql
CREATE MATERIALIZED VIEW mv_dept_totals AS
SELECT dept_id, SUM(salary) AS total_salary FROM employees GROUP BY dept_id;

REFRESH MATERIALIZED VIEW mv_dept_totals;   -- PostgreSQL / Oracle (Oracle: dbms_mview.refresh)
```
> **MySQL has no native materialized views** — simulate with a real table refreshed via an `EVENT` or scheduled job.

---

## 12. Common Table Expressions (CTEs) & Recursion

```sql
WITH dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
)
SELECT e.first_name, e.salary, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.dept_id = d.dept_id;
```

### Recursive CTE (e.g., org chart / hierarchy traversal)

```sql
WITH RECURSIVE org_chart AS (          -- MySQL 8+, PostgreSQL: RECURSIVE keyword required
    SELECT id, first_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.first_name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level;
```

> **MSSQL:** omit the `RECURSIVE` keyword — just `WITH org_chart AS (...)`.
> **Oracle:** supports the same `WITH ... UNION ALL` recursive form, or its legacy `CONNECT BY PRIOR` syntax:
```sql
SELECT id, first_name, LEVEL
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR id = manager_id;
```

---

## 13. Window Functions

```sql
SELECT
    first_name, dept_id, salary,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn,
    RANK()       OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dense_rnk,
    NTILE(4)     OVER (ORDER BY salary) AS quartile,
    LAG(salary)  OVER (PARTITION BY dept_id ORDER BY salary) AS prev_salary,
    LEAD(salary) OVER (PARTITION BY dept_id ORDER BY salary) AS next_salary,
    SUM(salary)  OVER (PARTITION BY dept_id) AS dept_total,
    AVG(salary)  OVER (ORDER BY salary ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM employees;
```

| Function | Purpose |
|---|---|
| `ROW_NUMBER()` | Unique sequential number per row |
| `RANK()` | Same rank for ties, gaps after ties |
| `DENSE_RANK()` | Same rank for ties, no gaps |
| `NTILE(n)` | Splits rows into n buckets |
| `LAG()`/`LEAD()` | Access prior/next row's value |
| `FIRST_VALUE()`/`LAST_VALUE()` | First/last value in the window frame |

> Widely supported in MySQL 8+, MSSQL 2012+, Oracle, PostgreSQL. **MySQL 5.7 and earlier have no window function support.**

---

## 14. Transactions & Concurrency

```sql
BEGIN TRANSACTION;                 -- MSSQL / (MySQL: START TRANSACTION;)

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
-- or ROLLBACK;

SAVEPOINT sp1;
ROLLBACK TO sp1;
```

| Property | Meaning |
|---|---|
| **A**tomicity | All-or-nothing execution |
| **C**onsistency | Valid state before & after |
| **I**solation | Concurrent transactions don't interfere |
| **D**urability | Committed data survives crashes |

### Isolation Levels

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;  -- default for most: MSSQL, Oracle, PostgreSQL
-- MySQL (InnoDB) default: REPEATABLE READ
```

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | ✔ possible | ✔ possible | ✔ possible |
| Read Committed | ✘ | ✔ possible | ✔ possible |
| Repeatable Read | ✘ | ✘ | ✔ possible (not in MySQL InnoDB, which prevents it via gap locks) |
| Serializable | ✘ | ✘ | ✘ |

---

## 15. Stored Procedures & Functions

```sql
-- MySQL
DELIMITER //
CREATE PROCEDURE GiveRaise(IN emp_id INT, IN pct DECIMAL(5,2))
BEGIN
    UPDATE employees SET salary = salary * (1 + pct/100) WHERE id = emp_id;
END //
DELIMITER ;

CALL GiveRaise(1, 10);
```

```sql
-- MSSQL (T-SQL)
CREATE PROCEDURE GiveRaise
    @emp_id INT, @pct DECIMAL(5,2)
AS
BEGIN
    UPDATE employees SET salary = salary * (1 + @pct/100) WHERE id = @emp_id;
END;

EXEC GiveRaise @emp_id = 1, @pct = 10;
```

```sql
-- Oracle (PL/SQL)
CREATE OR REPLACE PROCEDURE give_raise(p_emp_id IN NUMBER, p_pct IN NUMBER) AS
BEGIN
    UPDATE employees SET salary = salary * (1 + p_pct/100) WHERE id = p_emp_id;
END;
/

EXEC give_raise(1, 10);
```

### Functions (return a scalar value)

```sql
-- MySQL
CREATE FUNCTION GetAnnualSalary(monthly DECIMAL(10,2)) RETURNS DECIMAL(10,2)
DETERMINISTIC
RETURN monthly * 12;

-- MSSQL
CREATE FUNCTION dbo.GetAnnualSalary(@monthly DECIMAL(10,2))
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @monthly * 12;
END;

-- Oracle
CREATE OR REPLACE FUNCTION get_annual_salary(p_monthly NUMBER) RETURN NUMBER IS
BEGIN
    RETURN p_monthly * 12;
END;
/
```

| Concept | MySQL | MSSQL | Oracle |
|---|---|---|---|
| Procedural language | Basic SQL/PSM extensions | T-SQL | PL/SQL (most powerful/mature) |
| Statement delimiter | `DELIMITER //` needed for blocks | `;` (blocks via `BEGIN...END`) | `/` to execute a PL/SQL block |
| Packages (grouped procs/functions) | Not supported natively | Not native (use schemas) | `PACKAGE` / `PACKAGE BODY` supported |

---

## 16. Triggers

Triggers automatically run code in response to `INSERT`, `UPDATE`, or `DELETE` events.

```sql
-- MySQL
DELIMITER //
CREATE TRIGGER trg_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO salary_audit (emp_id, old_salary, new_salary, changed_at)
        VALUES (OLD.id, OLD.salary, NEW.salary, NOW());
    END IF;
END //
DELIMITER ;
```

```sql
-- MSSQL (T-SQL)
CREATE TRIGGER trg_salary_audit
ON employees
AFTER UPDATE
AS
BEGIN
    INSERT INTO salary_audit (emp_id, old_salary, new_salary, changed_at)
    SELECT d.id, d.salary, i.salary, GETDATE()
    FROM deleted d
    JOIN inserted i ON d.id = i.id
    WHERE d.salary <> i.salary;
END;
```

```sql
-- Oracle (PL/SQL)
CREATE OR REPLACE TRIGGER trg_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
WHEN (OLD.salary != NEW.salary)
BEGIN
    INSERT INTO salary_audit (emp_id, old_salary, new_salary, changed_at)
    VALUES (:OLD.id, :OLD.salary, :NEW.salary, SYSDATE);
END;
/
```

| Concept | MySQL | MSSQL | Oracle |
|---|---|---|---|
| Row-level access | `OLD.col`, `NEW.col` | Pseudo-tables `deleted`, `inserted` (set-based, not per-row by default) | `:OLD.col`, `:NEW.col` |
| Timing options | `BEFORE`, `AFTER` | `AFTER`, `INSTEAD OF` | `BEFORE`, `AFTER`, `INSTEAD OF` |
| Statement vs row-level | Row-level only | Statement-level (default) | Both supported (`FOR EACH ROW` for row-level) |
| DDL triggers | Not supported | Supported (`CREATE TRIGGER ... ON DATABASE`) | Supported (`ON SCHEMA`/`ON DATABASE`) |

---

## 17. String, Date & Null Functions (Dialect Comparison)

| Task | MySQL | MSSQL | Oracle |
|---|---|---|---|
| Concatenate strings | `CONCAT(a, b)` | `CONCAT(a, b)` or `a + b` | `a \|\| b` or `CONCAT(a,b)` |
| Substring | `SUBSTRING(s, 1, 3)` | `SUBSTRING(s, 1, 3)` | `SUBSTR(s, 1, 3)` |
| String length | `LENGTH(s)` | `LEN(s)` | `LENGTH(s)` |
| Uppercase/lowercase | `UPPER(s)`, `LOWER(s)` | same | same |
| Current date/time | `NOW()`, `CURDATE()` | `GETDATE()`, `SYSDATETIME()` | `SYSDATE`, `SYSTIMESTAMP` |
| Add interval | `DATE_ADD(d, INTERVAL 1 DAY)` | `DATEADD(day, 1, d)` | `d + 1` (or `d + INTERVAL '1' DAY`) |
| Date difference | `DATEDIFF(d1, d2)` | `DATEDIFF(day, d2, d1)` | `d1 - d2` |
| Format date | `DATE_FORMAT(d, '%Y-%m-%d')` | `FORMAT(d, 'yyyy-MM-dd')` | `TO_CHAR(d, 'YYYY-MM-DD')` |
| Null coalescing | `IFNULL(a, b)` / `COALESCE(a,b)` | `ISNULL(a, b)` / `COALESCE(a,b)` | `NVL(a, b)` / `COALESCE(a,b)` |
| Conditional | `IF(cond, a, b)` | `IIF(cond, a, b)` | `DECODE`/`CASE WHEN` |
| Cast type | `CAST(x AS type)` | `CAST(x AS type)` / `CONVERT(type, x)` | `CAST(x AS type)` |

`CASE` (standard across all dialects):
```sql
SELECT first_name,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 70000  THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM employees;
```

---

## 18. Pagination (Dialect Comparison)

```sql
-- MySQL
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 20;

-- MSSQL (2012+)
SELECT * FROM employees ORDER BY id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Oracle (12c+)
SELECT * FROM employees ORDER BY id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Oracle (pre-12c) - wrap with ROWNUM
SELECT * FROM (
    SELECT a.*, ROWNUM rnum FROM (SELECT * FROM employees ORDER BY id) a WHERE ROWNUM <= 30
) WHERE rnum > 20;
```

---

## 19. JSON Support

```sql
-- MySQL
SELECT JSON_EXTRACT(data, '$.name') FROM users;
SELECT data->>'$.name' FROM users;                 -- shorthand, unquoted result
UPDATE users SET data = JSON_SET(data, '$.age', 31) WHERE id = 1;

-- PostgreSQL
SELECT data->>'name' FROM users;                   -- ->> returns text, -> returns json
SELECT data->'address'->>'city' FROM users;

-- MSSQL
SELECT JSON_VALUE(data, '$.name') FROM users;
SELECT JSON_QUERY(data, '$.address') FROM users;

-- Oracle
SELECT JSON_VALUE(data, '$.name') FROM users;
```

---

## 20. Pivoting Data

```sql
-- MSSQL native PIVOT
SELECT * FROM (
    SELECT dept_id, job_title, salary FROM employees
) AS src
PIVOT (
    SUM(salary) FOR job_title IN ([Engineer], [Manager], [Analyst])
) AS pvt;

-- Oracle native PIVOT (11g+)
SELECT * FROM employees
PIVOT (
    SUM(salary) FOR job_title IN ('Engineer' AS Engineer, 'Manager' AS Manager)
);
```

> **MySQL has no native `PIVOT`** — emulate with conditional aggregation:
```sql
SELECT dept_id,
    SUM(CASE WHEN job_title = 'Engineer' THEN salary ELSE 0 END) AS Engineer,
    SUM(CASE WHEN job_title = 'Manager'  THEN salary ELSE 0 END) AS Manager
FROM employees
GROUP BY dept_id;
```

---

## 21. Query Execution Order

SQL is written in one order but **executed** in another — important for understanding why aliases behave the way they do:

```
1. FROM / JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. DISTINCT
7. ORDER BY
8. LIMIT / OFFSET / FETCH
```

This is why you can't reference a `SELECT` alias in a `WHERE` clause, but you *can* in `ORDER BY`.

---

## 22. Performance & EXPLAIN

```sql
-- MySQL
EXPLAIN SELECT * FROM employees WHERE dept_id = 10;
EXPLAIN ANALYZE SELECT * FROM employees WHERE dept_id = 10;   -- 8.0.18+

-- MSSQL
SET SHOWPLAN_ALL ON;
-- or use "Include Actual Execution Plan" in SSMS

-- Oracle
EXPLAIN PLAN FOR SELECT * FROM employees WHERE dept_id = 10;
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

**General tips:** index columns used in `WHERE`/`JOIN`/`ORDER BY`; avoid `SELECT *` in production code; avoid wrapping indexed columns in functions (`WHERE YEAR(date_col) = 2024` prevents index use); prefer `EXISTS` over `IN` for large subqueries; batch large `INSERT`/`UPDATE`/`DELETE` operations to avoid lock/log bloat.

---

## 23. Security Basics

```sql
-- Users & privileges
CREATE USER 'analyst'@'%' IDENTIFIED BY 'strong_password';   -- MySQL
GRANT SELECT ON company.* TO 'analyst'@'%';
REVOKE SELECT ON company.* FROM 'analyst'@'%';

CREATE LOGIN analyst WITH PASSWORD = 'strong_password';       -- MSSQL
CREATE USER analyst FOR LOGIN analyst;
GRANT SELECT ON employees TO analyst;

CREATE USER analyst IDENTIFIED BY strong_password;            -- Oracle
GRANT SELECT ON employees TO analyst;
```

- Always use **parameterized queries / prepared statements** in application code to prevent SQL injection — never concatenate raw user input into SQL strings.
- Apply the **principle of least privilege**: grant only the specific rights (`SELECT`, `INSERT`, etc.) a role actually needs.
- Use **views** or **row-level security** (MSSQL: `CREATE SECURITY POLICY`; Oracle: Virtual Private Database) to restrict data exposure without duplicating tables.

---

### Quick Reference: Dialect Nicknames & Notes

| Dialect | Full Name | Notable Traits |
|---|---|---|
| MySQL | MySQL / MariaDB | Open-source, widely used for web apps; `LIMIT`, `AUTO_INCREMENT`, `DELIMITER` blocks |
| MSSQL | Microsoft SQL Server (T-SQL) | `TOP`, `IDENTITY`, `GETDATE()`, strong integration with .NET/Windows ecosystem |
| Oracle | Oracle Database (PL/SQL) | Most feature-rich procedural language, `DUAL` table, `SYSDATE`, `CONNECT BY` |
| PostgreSQL | PostgreSQL | Standards-compliant, rich JSON/array types, `RETURNING` clause on DML |

```sql
-- Handy: SELECT without a FROM table
SELECT 1 + 1;                 -- MySQL, PostgreSQL, MSSQL — works directly
SELECT 1 + 1 FROM DUAL;       -- Oracle requires a FROM clause, DUAL is the dummy table
```