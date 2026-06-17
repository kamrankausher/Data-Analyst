# Chapter 2: Joins & Unions (Combining Datasets)

Relational database design splits data across multiple tables to enforce data integrity. To perform data analysis, you must merge these tables back together. This chapter covers the mathematical and algorithmic execution of Joins (horizontal merging) and Unions (vertical stacking).

---

## 1. Relational Joins (Horizontal Merging)
To understand joins, let's establish two sample tables:

### Table 1: `users` (Left Table)
| user_id | name | country |
| :--- | :--- | :--- |
| 1 | Alice | USA |
| 2 | Bob | Canada |
| 3 | Charlie | UK |

### Table 2: `transactions` (Right Table)
| tx_id | user_id | amount |
| :--- | :--- | :--- |
| 501 | 1 | 100 |
| 502 | 2 | 250 |
| 503 | 5 | 90 |

---

## 2. Inner Join vs. Outer Joins

### A. Inner Join (`INNER JOIN`)
Returns records only when there is a match in **both** tables.
*   **Venn Diagram**: The intersection ($A \cap B$).
*   *Query*:
    ```sql
    SELECT u.user_id, u.name, t.tx_id, t.amount
    FROM users u
    INNER JOIN transactions t ON u.user_id = t.user_id;
    ```
*   *Result*:
    | user_id | name | tx_id | amount |
    | :--- | :--- | :--- | :--- |
    | 1 | Alice | 501 | 100 |
    | 2 | Bob | 502 | 250 |
    *(Charlie is excluded because he has no transactions. Transaction 503 is excluded because user ID 5 is missing in the users table).*

### B. Left Join (`LEFT JOIN` or `LEFT OUTER JOIN`)
Returns **all** records from the left table, and matching records from the right. If no match exists, right-side columns are filled with `NULL`.
*   *Query*:
    ```sql
    SELECT u.user_id, u.name, t.tx_id, t.amount
    FROM users u
    LEFT JOIN transactions t ON u.user_id = t.user_id;
    ```
*   *Result*:
    | user_id | name | tx_id | amount |
    | :--- | :--- | :--- | :--- |
    | 1 | Alice | 501 | 100 |
    | 2 | Bob | 502 | 250 |
    | 3 | Charlie | `NULL` | `NULL` |

### C. Full Outer Join (`FULL JOIN`)
Returns all records when there is a match in either the left or right table.
*   *Result*:
    | user_id | name | tx_id | amount |
    | :--- | :--- | :--- | :--- |
    | 1 | Alice | 501 | 100 |
    | 2 | Bob | 502 | 250 |
    | 3 | Charlie | `NULL` | `NULL` |
    | `NULL` | `NULL` | 503 | 90 |

---

## 3. Special Joins: Self Join and Cross Join

### A. Self Join (Hierarchies)
A table joined with itself. Excellent for parent-child rows stored in a single table (e.g. employees and their managers).
*   *Query*:
    ```sql
    SELECT 
        emp.name AS employee_name,
        mgr.name AS manager_name
    FROM employees emp
    LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id;
    ```

### B. Cross Join (Cartesian Product)
Combines every row of Table A with every row of Table B. It does not use an `ON` clause.
*   *Math*: If Table A has $M$ rows and Table B has $N$ rows, the Cross Join results in $M \times N$ rows.
*   *Use Case*: Generating combinations of products and store locations.

---

## 4. Under the Hood: Join Algorithms
How does a database engine physically match rows in memory? The optimizer chooses one of three algorithms:

1.  **Nested Loop Join**:
    *   *Mechanism*: For each row in the outer table, scan the entire inner table.
    *   *Complexity*: $O(M \times N)$.
    *   *Use Case*: Best for small tables, especially if the inner table has an index on the join key.
2.  **Hash Join**:
    *   *Mechanism*: The engine reads the smaller table into memory and builds a hash table key-value store. It then scans the larger table, hashing the join keys to instantly find matches.
    *   *Complexity*: $O(M + N)$.
    *   *Use Case*: Best for large, unsorted tables.
3.  **Merge Join**:
    *   *Mechanism*: Both tables are sorted by the join key first. The engine then scans both tables in parallel, matching keys.
    *   *Complexity*: $O(M \log M + N \log N)$ for sorting.
    *   *Use Case*: Best when tables are already sorted or indexed.

---

## 5. Unions: Vertical Stacking
Unions stack datasets vertically.

### The Stacking Rules:
1.  Both queries must select the **same number of columns**.
2.  Columns must have **compatible data types** in the exact same index order.

### `UNION` vs. `UNION ALL`:
*   `UNION`: Stacks rows and **removes duplicates**. Behind the scenes, the database performs a sorting and deduplication step in memory, which is CPU-heavy.
*   `UNION ALL`: Stacks rows and **preserves all duplicates**. Very fast because the database just appends the pages. **Always use UNION ALL by default** unless you explicitly need duplicate rows removed.

---

## 6. Practice Question Bank

### Beginner Question
You have a table `customers` (customer_id, name) and a table `orders` (order_id, customer_id, product). Write a query to find the names of all customers who have placed at least one order.
*   **Solution Walkthrough**:
    1.  We need customer names, which are in `customers`.
    2.  We need to verify they have placed an order. An `INNER JOIN` ensures that only customers with matching records in `orders` are returned.
    3.  Because a customer might have placed multiple orders, we should use `DISTINCT` to avoid listing the same name multiple times.
    4.  **Correct Query**:
        ```sql
        SELECT DISTINCT c.name
        FROM customers c
        INNER JOIN orders o ON c.customer_id = o.customer_id;
        ```

### Intermediate Question
Write a query to find the names of all customers who have **never** placed an order.
*   **Solution Walkthrough**:
    1.  Perform a `LEFT JOIN` from `customers` to `orders` on `customer_id`. This keeps all customers, and adds order details where they exist.
    2.  For customers who have never ordered, their order-side columns (like `order_id` or `customer_id` from the orders table) will be `NULL`.
    3.  Filter for these rows in the `WHERE` clause.
    4.  **Correct Query**:
        ```sql
        SELECT c.name
        FROM customers c
        LEFT JOIN orders o ON c.customer_id = o.customer_id
        WHERE o.order_id IS NULL;
        ```

### Advanced Question
You have two tables: `web_leads` (email, signup_time) and `event_attendees` (email, attendance_date).
Write a query to construct a unified master list of all unique email addresses across both marketing channels. Do not use `UNION`. (Hint: Use a full outer join or subquery logic).
*   **Solution Walkthrough**:
    1.  Normally, a `UNION` is the standard way to merge lists vertically.
    2.  To do this without `UNION`, we can perform a `FULL OUTER JOIN` on the `email` column.
    3.  In a `FULL JOIN`, if an email exists in `web_leads` but not `event_attendees`, the `event_attendees.email` is null (and vice-versa).
    4.  We use `COALESCE` to extract the valid email from whichever side is not null.
    5.  **Correct Query**:
        ```sql
        SELECT DISTINCT COALESCE(w.email, e.email) AS unique_email
        FROM web_leads w
        FULL OUTER JOIN event_attendees e ON w.email = e.email;
        ```
        *(This returns a single consolidated column of unique emails across both tables).*
