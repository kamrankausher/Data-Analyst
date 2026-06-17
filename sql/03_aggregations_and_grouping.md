# Chapter 3: Aggregations & Grouping

Aggregation is the process of summarizing transaction-level records into business metrics. This chapter covers standard and conditional aggregations, the behind-the-scenes memory mechanics of grouping, and filtering aggregated results.

---

## 1. Aggregate Functions & COUNT Logic
Aggregate functions process multiple column values and return a single value.

*   `SUM(col)`: Totals values. Ignores `NULL` values.
*   `AVG(col)`: Computes mean. Ignores `NULL` values (which changes the denominator!).
*   `MIN(col)` & `MAX(col)`: Returns boundary values. Works on numbers, dates, and alphabetic strings.

### The COUNT Variations (Crucial for Data Quality):
*   `COUNT(*)`: Counts **every row** in the partition, including rows containing `NULL` values.
*   `COUNT(column)`: Counts only rows where the specified column is **not null**.
*   `COUNT(DISTINCT column)`: Counts unique, non-null values.

#### Case Study: `sales_bonus` table
| emp_id | department | bonus_pct |
| :--- | :--- | :--- |
| 101 | Tech | 0.10 |
| 102 | Tech | 0.10 |
| 103 | Tech | `NULL` |

*   `COUNT(*)` = `3`
*   `COUNT(bonus_pct)` = `2` (emp 103 is skipped because bonus is NULL)
*   `COUNT(DISTINCT bonus_pct)` = `1` (Unique values is only `0.10`)
*   `AVG(bonus_pct)`: Calculated as $(0.10 + 0.10) / 2 = 0.10$. (Note: The division is by 2, not 3, because the NULL row is ignored. If you want to treat NULL as 0, you must write `AVG(COALESCE(bonus_pct, 0))`, which evaluates to $(0.10 + 0.10 + 0.00) / 3 = 0.066$).

---

## 2. Under the Hood: GROUP BY Memory Mechanics
When the database runs `GROUP BY`, it partitions your data in memory.

```
   Raw Table:
   Dept  | Salary
   Tech  | 100
   Sales | 80
   Tech  | 120
   
   Step 1: Partitioning in Memory (Hashing/Sorting)
   [Tech]  ==> [100, 120]
   [Sales] ==> [80]
   
   Step 2: Apply aggregate function (e.g. SUM)
   [Tech]  ==> 220
   [Sales] ==> 80
```

### The Selection Constraint:
If you use `GROUP BY`, **every single column** in your `SELECT` list must either be:
1.  Listed in your `GROUP BY` clause, or
2.  Wrapped inside an aggregate function.
*   *Why?* If you select a column (like `employee_name`) that is not grouped and not aggregated, the database does not know which employee name to display for the unified group row (e.g., in the Tech group above, should it show Alice or Bob?).

---

## 3. Filtering Data: `WHERE` vs. `HAVING`
Analysts must understand the difference in execution order:

*   `WHERE`: Runs **before** the grouping step. It filters out raw rows. It cannot reference aggregate functions.
*   `HAVING`: Runs **after** the grouping step. It filters out aggregate buckets.

```sql
SELECT 
    department_id, 
    AVG(salary) AS avg_sal
FROM employees
WHERE salary > 30000       -- Filter individual high-earning rows FIRST
GROUP BY department_id      -- Group them SECOND
HAVING AVG(salary) > 60000; -- Keep departments where group average is > 60k THIRD
```

---

## 4. Scalar Date & Time Functions (Standard SQL)
Data analysts constantly group metrics by date intervals:

*   `DATE(timestamp)`: Extracts the date portion.
*   `EXTRACT(PART FROM date)`: Extracts component numbers (e.g., `EXTRACT(MONTH FROM order_date)` returns values from 1 to 12).
*   `DATE_TRUNC('interval', timestamp)` (PostgreSQL / Snowflake standard): Truncates dates down to the start of the interval. Essential for cohort analysis.
    *   `DATE_TRUNC('month', '2026-06-17')` -> `'2026-06-01'` (All transactions in June collapse to June 1st, allowing easy monthly grouping).

---

## 5. Practice Question Bank

### Beginner Question
You have a table `website_logs` with columns `log_id`, `user_id`, and `visit_time` (timestamp). Write a query to find the count of unique visitors who visited the website for each day.
*   **Solution Walkthrough**:
    1.  We need to group by day. Extract date from timestamp: `DATE(visit_time)` or `CAST(visit_time AS DATE)`.
    2.  We need to count unique visitors: `COUNT(DISTINCT user_id)`.
    3.  Group by the date expression.
    4.  **Correct Query**:
        ```sql
        SELECT 
            DATE(visit_time) AS visit_day,
            COUNT(DISTINCT user_id) AS unique_visitors
        FROM website_logs
        GROUP BY DATE(visit_time)
        ORDER BY visit_day;
        ```

### Intermediate Question
You have an `employees` table (employee_id, name, department_id, salary). Write a query to find all department IDs where the average salary is higher than $80,000, but exclude employees who earn less than $20,000 from the calculation entirely.
*   **Solution Walkthrough**:
    1.  We must exclude employees earning $<20,000$ before calculating averages. This requires a `WHERE` clause: `WHERE salary >= 20000`.
    2.  Group by `department_id`.
    3.  Filter the resulting averages using a `HAVING` clause: `HAVING AVG(salary) > 80000`.
    4.  **Correct Query**:
        ```sql
        SELECT department_id, AVG(salary) AS average_salary
        FROM employees
        WHERE salary >= 20000
        GROUP BY department_id
        HAVING AVG(salary) > 80000;
        ```

### Advanced Question
You have a table `sales_orders` (order_id, customer_id, order_date, amount).
Write a query to classify customers into three segments based on their total spend:
- `"VIP"`: Spend $> \$10,000$
- `"Active"`: Spend between $\$2,000$ and $\$10,000$
- `"Standard"`: Spend $< \$2,000$
Display the count of customers in each segment. (Hint: Use a CASE statement inside a CTE or subquery).
*   **Solution Walkthrough**:
    1.  Create a CTE to calculate the total spend for each customer.
    2.  In the main query, use a `CASE` statement on the total spend to classify each customer.
    3.  Group by the `CASE` statement classification to count how many customers fall into each category.
    4.  **Correct Query**:
        ```sql
        WITH customer_totals AS (
            SELECT 
                customer_id,
                SUM(amount) AS total_spend
            FROM sales_orders
            GROUP BY customer_id
        )
        SELECT 
            CASE 
                WHEN total_spend > 10000 THEN 'VIP'
                WHEN total_spend BETWEEN 2000 AND 10000 THEN 'Active'
                ELSE 'Standard'
            END AS customer_segment,
            COUNT(customer_id) AS customer_count
        FROM customer_totals
        GROUP BY 
            CASE 
                WHEN total_spend > 10000 THEN 'VIP'
                WHEN total_spend BETWEEN 2000 AND 10000 THEN 'Active'
                ELSE 'Standard'
            END;
        ```
