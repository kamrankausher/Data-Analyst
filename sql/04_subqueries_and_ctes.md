# Chapter 4: Subqueries & Common Table Expressions (CTEs)

As data analysis requirements grow, a single SELECT query is rarely sufficient. You will need to nest queries, feeding the output of one dataset into another. This chapter covers the syntax, evaluation paths, and performance differences between Subqueries and Common Table Expressions (CTEs).

---

## 1. What is a Subquery?
A subquery (or nested query) is a query enclosed inside another SQL query. It can return a single value (scalar), a single column (vector), or a full table.

### A. Subquery in `WHERE` (Scalar/List Filter)
Used to filter rows against calculated thresholds.
*   *Query*: Find employees who earn more than the company average.
    ```sql
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees); -- Nested query runs first, returns one number
    ```

### B. Subquery in `FROM` (Derived Table / Inline View)
Used to create a temporary table.
*   *Critical Rule*: You **must** assign an alias to a subquery placed in the FROM clause.
*   *Query*: Find the average sales across all branches, using their regional totals.
    ```sql
    SELECT AVG(total_regional_sales) AS national_average
    FROM (
        SELECT branch_id, SUM(sales_amount) AS total_regional_sales
        FROM transactions
        GROUP BY branch_id
    ) AS branch_summaries; -- Alias mandatory here
    ```

### C. Subquery in `SELECT` (Calculated Field)
Used to compute columns dynamically. Note: The subquery must return a single scalar value.
```sql
SELECT 
    name, 
    salary, 
    (SELECT AVG(salary) FROM employees) AS company_average
FROM employees;
```

---

## 2. Correlated vs. Non-Correlated Subqueries

### A. Non-Correlated Subqueries
The nested query is completely independent of the outer query.
*   **Execution**: The database optimizer runs the subquery **exactly once**, extracts the result, and passes it to the outer query. Very fast.

### B. Correlated Subqueries
The nested query references a column from the outer query.
*   **Execution**: The database engine cannot run the subquery in isolation. It must run the subquery **once for every single row** evaluated by the outer query.
*   *Query*: Find employees who earn more than the average salary of *their specific department*.
    ```sql
    SELECT outer_emp.name, outer_emp.salary, outer_emp.department_id
    FROM employees outer_emp
    WHERE outer_emp.salary > (
        SELECT AVG(inner_emp.salary)
        FROM employees inner_emp
        WHERE inner_emp.department_id = outer_emp.department_id -- Link to outer row
    );
    ```
    *Performance warning*: If the outer query returns 100,000 rows, the subquery executes 100,000 times! Avoid correlated subqueries on large datasets where possible (rewrite them using Window Functions or Joins).

---

## 3. Common Table Expressions (CTEs)
A CTE is a temporary named result set defined before your main query. It acts as a modular, readable building block.

```sql
-- Defining a CTE
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
-- Main query referencing the CTE
SELECT * 
FROM cte_name
WHERE column1 > 100;
```

### The Benefits of CTEs:
1.  **Readability**: Subqueries read from the "inside out". CTEs read from "top to bottom", matching human logic.
2.  **Reusability**: You can reference the same CTE multiple times within your main query (e.g., joining a CTE to itself).
3.  **Modularity**: You can define multiple CTEs separated by commas and join them:
    ```sql
    WITH cte_sales AS (...),
         cte_targets AS (...)
    SELECT * 
    FROM cte_sales s 
    LEFT JOIN cte_targets t ON s.product_id = t.product_id;
    ```

---

## 4. Subqueries vs. CTEs: Performance Differences
In modern SQL database engines (PostgreSQL 12+, MySQL 8+, SQL Server, BigQuery), the optimizer compiles CTEs and subqueries into the **same execution plan**. There is generally zero difference in query execution speed.
*   *Historically*: Some database engines materialized (cached) CTEs, which could slow down queries. Today, optimizers merge CTEs directly into the query tree.
*   *Rule of thumb*: Use CTEs by default for code maintainability, and only rewrite to subqueries if profiling shows optimization advantages.

---

## 5. Practice Question Bank

### Beginner Question
You have an `employees` table (employee_id, name, hire_date). Write a query using a **subquery** in the WHERE clause to retrieve the names of all employees who were hired *after* the employee with ID `105` was hired.
*   **Solution Walkthrough**:
    1.  We need to find the hire date of employee 105: `(SELECT hire_date FROM employees WHERE employee_id = 105)`.
    2.  Filter the main table for hire dates greater than this subquery result.
    3.  **Correct Query**:
        ```sql
        SELECT name, hire_date
        FROM employees
        WHERE hire_date > (SELECT hire_date FROM employees WHERE employee_id = 105);
        ```

### Intermediate Question
You have tables `products` (product_id, name, price) and `order_items` (order_id, product_id, quantity).
Write a query using a **CTE** to find the name and price of the product(s) that have generated the highest total revenue (calculated as `quantity * price`).
*   **Solution Walkthrough**:
    1.  Create a CTE `product_revenue` that sums `quantity * price` for each product.
    2.  In the main query, find the product(s) where the revenue matches the maximum revenue calculated in the CTE.
    3.  **Correct Query**:
        ```sql
        WITH product_revenue AS (
            SELECT 
                p.product_id, 
                p.name, 
                p.price,
                SUM(oi.quantity * p.price) AS total_revenue
            FROM order_items oi
            INNER JOIN products p ON oi.product_id = p.product_id
            GROUP BY p.product_id, p.name, p.price
        )
        SELECT name, price, total_revenue
        FROM product_revenue
        WHERE total_revenue = (SELECT MAX(total_revenue) FROM product_revenue);
        ```

### Advanced Question
You have a table `employees` (employee_id, manager_id, name). This table represents an organization hierarchy.
Write a query using a **Recursive CTE** to display all employees under manager ID `10`, listing their organizational level (manager 10 is level 1, their direct reports are level 2, etc.).
*   **Solution Walkthrough**:
    1.  A Recursive CTE uses two parts separated by `UNION ALL`.
    2.  **Anchor Member**: The baseline start point (Manager ID 10):
        `SELECT employee_id, manager_id, name, 1 AS level FROM employees WHERE employee_id = 10`
    3.  **Recursive Member**: The loop that joins the CTE back to the employees table to find the next level down:
        `SELECT e.employee_id, e.manager_id, e.name, r.level + 1 FROM employees e INNER JOIN org_hierarchy r ON e.manager_id = r.employee_id`
    4.  **Correct Query**:
        ```sql
        WITH RECURSIVE org_hierarchy AS (
            -- Anchor Member
            SELECT employee_id, manager_id, name, 1 AS level
            FROM employees
            WHERE employee_id = 10
            
            UNION ALL
            
            -- Recursive Member
            SELECT e.employee_id, e.manager_id, e.name, r.level + 1
            FROM employees e
            INNER JOIN org_hierarchy r ON e.manager_id = r.employee_id
        )
        SELECT * FROM org_hierarchy;
        ```
