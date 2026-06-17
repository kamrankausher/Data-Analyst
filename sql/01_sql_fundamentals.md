# Chapter 1: SQL Fundamentals

SQL (Structured Query Language) is the universal interface to data. This chapter covers the mathematical foundations of relational databases, the exact execution order of SQL compilation, three-valued logic, and pattern filtering.

---

## 1. Relational Database Architecture
A Relational Database Management System (RDBMS) stores data in tables linked by keys.

```
       [Parent Table: Departments]
       Primary Key: dept_id
       +---------+-----------------+
       | dept_id | dept_name       |
       +---------+-----------------+
       | 10      | Finance         | <---+
       +---------+-----------------+     |
                                         | 1-to-Many Relationship
       [Child Table: Employees]          |
       Foreign Key: department_id        |
       +--------+-----------+---------------+
       | emp_id | name      | department_id |
       +--------+-----------+---------------+
       | 101    | Alice     | 10            | ----> Links to dept_id 10
       +--------+-----------+---------------+
```

*   **Primary Key (PK)**: A column containing unique values that identify each row. PK columns cannot contain `NULL` or duplicate values.
*   **Foreign Key (FK)**: A column in one table that references the Primary Key of another table, maintaining referential integrity.
*   **Database Storage Pages**: Under the hood, databases do not read individual rows from disk. They read **pages** (usually 8KB blocks in SQL Server or 16KB in MySQL). If you write unoptimized queries that scan entire tables, the database has to pull thousands of pages into memory, causing disk I/O bottlenecks.

---

## 2. The Compiler's Journey: SQL Execution Order
When you write a SQL query, you write it in one sequence, but the database parser executes it in a completely different sequence. Understanding this is key to debugging.

```
               HOW YOU WRITE IT              HOW SQL RUNS IT
               
               1. SELECT                     1. FROM & JOINs
               2. FROM                       2. WHERE (Row Filter)
               3. JOIN ... ON                3. GROUP BY (Aggregation)
               4. WHERE                      4. HAVING (Group Filter)
               5. GROUP BY                   5. SELECT
               6. HAVING                     6. DISTINCT
               7. ORDER BY                   7. ORDER BY
               8. LIMIT                      8. LIMIT / OFFSET
```

### The Impact of Compilation Order:
1.  **Aliases**: Because `WHERE` (Step 2) runs before `SELECT` (Step 5), you **cannot** filter by an alias created in the SELECT clause.
    *   *Fails*: `SELECT price * qty AS total FROM sales WHERE total > 100`
    *   *Succeeds*: `SELECT price * qty AS total FROM sales WHERE (price * qty) > 100`
2.  **Order By**: Because `ORDER BY` (Step 7) runs after `SELECT` (Step 5), you **can** sort by an alias created in the SELECT clause.
    *   *Succeeds*: `SELECT price * qty AS total FROM sales ORDER BY total DESC`

---

## 3. Row Filtering & Pattern Matching

### A. Comparison Operators
Standard comparisons: `=`, `<>` (Not Equal), `>`, `<`, `>=`, `<=`

### B. Logical Operators & Wildcards
*   `IN`: Evaluates if a value exists within a comma-separated list.
    `WHERE country IN ('USA', 'Canada', 'Mexico')`
*   `BETWEEN`: Range check (inclusive).
    `WHERE salary BETWEEN 50000 AND 80000` (Equivalent to `salary >= 50000 AND salary <= 80000`).
*   `LIKE`: Pattern match using wildcards:
    *   `%`: Matches zero or more characters.
    *   `_` (Underscore): Matches exactly one character.
    *   *Example*: `LIKE 'A_b%'` matches "Albert", "Abby", but not "Ab".

---

## 4. Ternary (Three-Valued) Logic & NULLs
In mathematics, values are either True or False. In SQL, because data can be missing (`NULL`), logic has three states: **True, False, or Unknown**.

*   `NULL` represents the *absence of data*. It is not equal to `0` or an empty string `""`.
*   Any comparison with `NULL` (even `NULL = NULL`) evaluates to **Unknown**.
*   Because of this, you must never use `=` or `<>` to check for Nulls. You must use `IS NULL` or `IS NOT NULL`.

```sql
-- This returns 0 rows! (NULL = NULL evaluates to Unknown, and WHERE only keeps TRUE rows)
SELECT * FROM employees WHERE manager_id = NULL;

-- This is the correct way
SELECT * FROM employees WHERE manager_id IS NULL;
```

### Default Fallbacks: `COALESCE` and `IFNULL`
*   `COALESCE(val1, val2, ...)`: Returns the first non-null argument.
    `SELECT COALESCE(bonus, 0) FROM employees` (If bonus is null, returns 0).

---

## 5. Practice Question Bank

### Beginner Question
You have an `orders` table. You want to retrieve the `order_id` and `customer_id` for all orders placed in the year 2026 where the `shipped_date` is missing. Sort the results by `order_date` descending. Write the query.
*   **Solution Walkthrough**:
    1.  Select target columns: `order_id`, `customer_id`.
    2.  Filter by date range for 2026: `order_date BETWEEN '2026-01-01' AND '2026-12-31'`.
    3.  Filter for missing shipping dates: `shipped_date IS NULL`.
    4.  Sort by order date descending: `ORDER BY order_date DESC`.
    5.  **Correct Query**:
        ```sql
        SELECT order_id, customer_id
        FROM orders
        WHERE order_date BETWEEN '2026-01-01' AND '2026-12-31'
          AND shipped_date IS NULL
        ORDER BY order_date DESC;
        ```

### Intermediate Question
Trace why the following query fails and rewrite it to make it execute correctly.
```sql
SELECT employee_id, salary + COALESCE(commission, 0) AS total_pay
FROM employees
WHERE total_pay > 50000;
```
*   **Solution Walkthrough**:
    1.  The database executes clauses in this order: `FROM` -> `WHERE` -> `SELECT`.
    2.  When the `WHERE` clause runs, the expression `salary + COALESCE(commission, 0)` has not yet been computed or aliased as `total_pay`. The compiler does not know what `total_pay` is.
    3.  To fix it, we must replace the alias in the `WHERE` clause with the actual calculation.
    4.  **Correct Query**:
        ```sql
        SELECT employee_id, salary + COALESCE(commission, 0) AS total_pay
        FROM employees
        WHERE (salary + COALESCE(commission, 0)) > 50000;
        ```

### Advanced Question
You have a table `customers` with columns `customer_id` and `phone_number`.
Write a query to find all customers whose phone numbers match the format `###-###-####` (where each `#` represents a single digit between 0 and 9, separated by hyphens). Note: assume standard SQL pattern matching.
*   **Solution Walkthrough**:
    1.  We need to match exact characters and positions.
    2.  The hyphen `-` must appear at positions 4 and 8.
    3.  A single digit placeholder is the underscore `_`.
    4.  The pattern is `___-___-____` (three underscores, hyphen, three underscores, hyphen, four underscores).
    5.  **Correct Query**:
        ```sql
        SELECT customer_id, phone_number
        FROM customers
        WHERE phone_number LIKE '___-___-____';
        ```
        *(For more rigid database-specific validations, regex like `phone_number ~ '^[0-9]{3}-[0-9]{3}-[0-9]{4}$'` is used, but standard SQL uses `LIKE` wildcards).*
