# Chapter 6: Database Design & Query Optimization

When datasets grow to millions or billions of rows, unoptimized SQL code can lock database resources, crash applications, or take hours to complete. This chapter covers database normalization, dimensional modeling (Star/Snowflake), B-Tree indexes, reading execution plans, and query tuning strategies.

---

## 1. Database Schema Design & Normalization
Normalization is the process of structuring relational database tables to minimize data redundancy and prevent data anomalies.

### A. The Normal Forms
*   **First Normal Form (1NF)**: Values in each cell must be atomic (single, indivisible values). No arrays, lists, or multiple comma-separated values in a cell.
*   **Second Normal Form (2NF)**: Meets 1NF, and all non-key columns must depend entirely on the Primary Key (no partial dependencies on a composite PK).
*   **Third Normal Form (3NF)**: Meets 2NF, and no non-key columns are transitively dependent on the Primary Key (non-key columns cannot depend on other non-key columns).
    *   *Analogy*: In 3NF, every column must depend on the primary key, the whole primary key, and nothing but the primary key (so help me Codd!).

### B. Analytical Modeling: Star vs. Snowflake Schema

#### 1. Star Schema (Industry Standard for BI)
Fact tables (containing numeric measurements like sales) connect directly to Dimension tables (descriptive attributes like customers).
*   *Pros*: Fast query performance, simple joins.
*   *Cons*: Redundant data in dimensions (e.g. repeating cities in a customer table).

#### 2. Snowflake Schema
Similar to a Star Schema, but dimension tables are further normalized into separate lookup tables (e.g. `Customers` table links to a `Cities` table, which links to a `Countries` table).
*   *Pros*: Eliminates data redundancy in dimension tables.
*   *Cons*: Degrades query performance because the engine must traverse multiple joins.

---

## 2. Under the Hood: How Indexes Work
An Index is an on-disk structure that speeds up row retrieval.

### A. The B-Tree Index (Balanced Tree)
The default index structure in relational databases is the B-Tree.

```
                         [ Root Node ]
                            /     \
                     [ Page 1 ]   [ Page 2 ]
                       /    \       /    \
                     [A-D]  [E-H] [I-L]  [M-P] <-- Leaf Nodes (Pointers to Disk)
```

Instead of scanning all pages sequentially from disk, the database engine traverses the B-Tree from the Root node to a Leaf node.
*   **Binary Search Math**: Searching a sorted index tree takes $O(\log N)$ time, compared to $O(N)$ for a full table scan.
*   **The Write Tax**: Every index added to a table speeds up `SELECT` queries but slows down `INSERT`, `UPDATE`, and `DELETE` queries, because the database must update the index tree every time data changes.

---

## 3. Reading Execution Plans (`EXPLAIN`)
To optimize queries, you must look at how the compiler plans to run them. Prefix your query with `EXPLAIN` or `EXPLAIN ANALYZE` (runs the query and prints metrics).

### Key Terms in Execution Plans:
*   **Sequential Scan (Table Scan)**: Bad. The engine reads the entire table from disk.
*   **Index Scan**: The engine reads the entire index tree.
*   **Index Seek**: Good. The engine uses the B-Tree index to jump directly to the target record.
*   **Hash Match / Hash Join**: The engine builds an in-memory hash table of the smaller table to join keys.

---

## 4. Query Tuning Best Practices for Analysts

### Rule 1: Stop Writing `SELECT *`
*   *Why*: Selecting all columns forces the engine to read useless data from disk, clogging memory and network bandwidth.

### Rule 2: Keep Queries "SARGable" (Search Argument Able)
Applying functions on indexed columns in the `WHERE` clause prevents the optimizer from using the index.

*   *Non-SARGable (Bad - full table scan)*:
    ```sql
    SELECT name FROM employees WHERE YEAR(hire_date) = 2026;
    ```
*   *SARGable (Good - uses index seek)*:
    ```sql
    SELECT name FROM employees WHERE hire_date BETWEEN '2026-01-01' AND '2026-12-31';
    ```

### Rule 3: Avoid `OR` on Different Columns
The engine struggles to optimize index searches with `OR`. Replace with `UNION ALL`:
*   *Unoptimized*:
    ```sql
    SELECT * FROM users WHERE status = 'Active' OR country = 'Canada';
    ```
*   *Optimized*:
    ```sql
    SELECT * FROM users WHERE status = 'Active'
    UNION ALL
    SELECT * FROM users WHERE country = 'Canada' AND status <> 'Active';
    ```

---

## 5. Practice Question Bank

### Beginner Question
You have an `orders` table. There is an index on the column `order_date`. The following query is running slowly. Identify the performance bug and rewrite it.
```sql
SELECT order_id, customer_id
FROM orders
WHERE DATE_TRUNC('month', order_date) = '2026-06-01';
```
*   **Solution Walkthrough**:
    1.  The column `order_date` is wrapped in the `DATE_TRUNC()` function.
    2.  This makes the query non-SARGable. The database must compute `DATE_TRUNC` for every row in the table, disabling the index.
    3.  Rewrite the filter to check a raw range on the `order_date` column.
    4.  **Correct Query**:
        ```sql
        SELECT order_id, customer_id
        FROM orders
        WHERE order_date >= '2026-06-01' AND order_date < '2026-07-01';
        ```

### Intermediate Question
Explain the difference between a **Table Scan (Sequential Scan)** and an **Index Seek**, using an analogy.
*   **Solution Walkthrough**:
    *   **Table Scan**: Imagine entering a library with 1,000,000 books and looking for a specific title by starting at book 1, walking shelf-by-shelf, and reading every book cover until you find it. This is $O(N)$ complexity and consumes maximum energy/time.
    *   **Index Seek**: Imagine walking up to the library's computerized card catalog, searching the title, getting the exact shelf address (e.g. Section C, Row 4, Shelf 2), and walking directly to that book. This is $O(\log N)$ complexity and is highly efficient.

### Advanced Question
You have an e-commerce database. You notice duplicates of customer addresses. The table `orders` contains columns: `order_id`, `customer_id`, `customer_name`, `customer_address`, `customer_city`, `product_id`, `product_price`.
1.  Explain which Normal Form this table violates and why.
2.  Provide the schema of the split tables to achieve 3rd Normal Form (3NF).
*   **Solution Walkthrough**:
    1.  This table violates 2NF and 3NF.
    2.  `customer_name`, `customer_address`, and `customer_city` depend on `customer_id`, not the primary key `order_id` (partial dependency).
    3.  `product_price` depends on `product_id`, not `order_id` (partial dependency).
    4.  To achieve 3NF, we must isolate these entities into separate tables:
        *   `customers` table: `customer_id` (PK), `customer_name`, `customer_address`, `customer_city`.
        *   `products` table: `product_id` (PK), `product_price`.
        *   `orders` table: `order_id` (PK), `customer_id` (FK), `product_id` (FK).
