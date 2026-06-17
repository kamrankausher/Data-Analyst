# SQL Data Analyst Interview Prep Guide

This guide contains the most frequently asked SQL interview questions in Data Analyst and Analytics Engineering interviews at top tech companies. Each question includes a detailed, professional answer.

---

## 1. Query Execution & Parser Compilation

### Question 1: Explain the logical execution order of a SQL query. Why does this order prevent you from using column aliases in the WHERE clause?
*   **Logical Execution Order**:
    1.  `FROM` (with `JOIN`s)
    2.  `WHERE`
    3.  `GROUP BY`
    4.  `HAVING`
    5.  `SELECT` (with aliases and window functions)
    6.  `DISTINCT`
    7.  `ORDER BY`
    8.  `LIMIT` / `OFFSET`
*   **The Alias Problem**: Because the `WHERE` clause is compiled and executed in **Step 2**, but the column aliases are only created in the `SELECT` clause in **Step 5**, the database engine does not know what the alias refers to during row filtering.
*   *Incorrect*: `SELECT age * 12 AS months FROM users WHERE months > 200;`
*   *Correct*: `SELECT age * 12 AS months FROM users WHERE (age * 12) > 200;`

---

## 2. Aggregations & Grouping

### Question 2: What is the difference between WHERE and HAVING? Can you write a query using both in the same statement?
*   **WHERE**: Filters individual rows **before** any grouping or aggregation takes place. It cannot reference aggregate functions (like `SUM` or `AVG`).
*   **HAVING**: Filters grouped summary rows **after** the `GROUP BY` clause has been executed. It is used to filter based on aggregated metrics.
*   **Combined Query Example**: Find departments where the average salary of employees earning more than $40,000 exceeds $80,000.
    ```sql
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    WHERE salary > 40000 -- Step 1: Filter out individual low earners
    GROUP BY department_id -- Step 2: Group by department
    HAVING AVG(salary) > 80000; -- Step 3: Filter out departments with low average
    ```

---

## 3. Window Functions & Ranking

### Question 3: Explain the difference between ROW_NUMBER(), RANK(), and DENSE_RANK(). When would a business scenario require one over the others?
*   **ROW_NUMBER()**: Assigns a unique, consecutive integer to every row, regardless of duplicates.
    *   *Business Scenario*: Pagination (e.g. displaying items 1-10 on page 1).
*   **RANK()**: Assigns the same rank to identical values, but **skips** ranks. If two rows rank 2nd, the next row gets 4th.
    *   *Business Scenario*: Classic competitions (e.g., if two people share silver, the next person gets bronze).
*   **DENSE_RANK()**: Assigns the same rank to identical values, but **does not skip** ranks. If two rows rank 2nd, the next row gets 3rd.
    *   *Business Scenario*: Financial salary bands or sales performance tiers (e.g., finding the top 3 highest unique salaries).

---

## 4. Query Performance & Indexing

### Question 4: What is a "Non-SARGable" query? Explain how writing `WHERE YEAR(date_column) = 2026` affects query speed compared to `WHERE date_column BETWEEN ...`.
*   **Non-SARGable**: A query where the database optimizer cannot use an index to speed up the search because a function is applied to the indexed column.
*   **Index Scan vs. Seek**:
    *   If you write `WHERE YEAR(date_column) = 2026`, the database must run the `YEAR()` function on **every single row** in the table (Full Table Scan / Sequential Scan) to check if it equals 2026, even if there is an index on `date_column`.
    *   If you write `WHERE date_column BETWEEN '2026-01-01' AND '2026-12-31'`, the database can use the index to jump directly to the first row of 2026 and read until the end of 2026 (Index Seek).
*   **Impact**: On a 100-million-row table, the first query can take minutes and lock the table, while the second query completes in milliseconds.

---

## 5. Coding Challenge (Live Coding Practice)

### Scenario:
You have a table `user_sessions` (session_id, user_id, start_time, page_views).
Write a query to find the users who have page views higher than the average page views of all users, but only for sessions that occurred in the last 30 days. Display `user_id` and their total page views.

*   **Solution Walkthrough**:
    1.  We need to calculate the average page views of all users for sessions in the last 30 days.
    2.  Create a CTE `recent_user_views` to sum page views per user over the last 30 days.
    3.  In the main query, calculate the overall average from the CTE.
    4.  Filter the CTE to show only users whose total views exceed this average.
    5.  **Correct Query**:
        ```sql
        WITH recent_user_views AS (
            SELECT 
                user_id,
                SUM(page_views) AS total_views
            FROM user_sessions
            WHERE start_time >= CURRENT_DATE - INTERVAL '30 days'
            GROUP BY user_id
        )
        SELECT user_id, total_views
        FROM recent_user_views
        WHERE total_views > (SELECT AVG(total_views) FROM recent_user_views);
        ```
