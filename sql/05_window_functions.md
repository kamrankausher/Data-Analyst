# Chapter 5: Window Functions

Window functions perform calculations across a set of table rows related to the current row. Unlike `GROUP BY` which collapses rows, window functions preserve the individual identity of each row. This chapter covers the syntax, mechanics, and advanced partition configurations of windowing.

---

## 1. Anatomy of a Window Function
A window function is identified by the `OVER` clause, which instructs the compiler to compute calculations across a defined window (subset) of rows.

```sql
SELECT 
    column_name,
    AGGREGATE_OR_WINDOW_FUNCTION() OVER (
        PARTITION BY partition_column
        ORDER BY sort_column
        ROWS_OR_RANGE_FRAME_SPECIFICATION
    ) AS alias_name
FROM table_name;
```

*   `PARTITION BY`: Splits the rows into distinct buckets (e.g., partition sales by `region`). If omitted, the window is the entire dataset.
*   `ORDER BY`: Sorts the rows within each partition. This is mandatory for rank, lag, and running totals.
*   `ROWS_OR_RANGE_FRAME`: Defines boundaries relative to the current row (e.g. "include 2 rows before and 2 rows after").

---

## 2. Ranking Functions
Ranking functions assign integer ranks to rows within partitions.

### A. ROW_NUMBER(), RANK(), and DENSE_RANK()
*   `ROW_NUMBER()`: Assigns a unique, sequential integer. If values are identical, it still assigns consecutive numbers arbitrarily.
*   `RANK()`: Assigns the same rank to identical values, but **skips** ranks.
*   `DENSE_RANK()`: Assigns the same rank to identical values, but **does not skip** ranks.

#### Visual Comparison (Order by score descending):
| Student | Score | ROW_NUMBER() | RANK() | DENSE_RANK() |
| :--- | :--- | :--- | :--- | :--- |
| Alice | 98 | 1 | 1 | 1 |
| Bob | 95 | 2 | 2 | 2 |
| Charlie | 95 | 3 | 2 | 2 |
| Dan | 90 | 4 | **4** | **3** |

---

## 3. Value Functions: LAG & LEAD
Lags and leads retrieve values from surrounding rows without self-joins.

*   `LAG(column, offset, default)`: Accesses a column value from `offset` rows **before** the current row.
*   `LEAD(column, offset, default)`: Accesses a column value from `offset` rows **after** the current row.

```
   Row Index   Sales   LAG(Sales, 1)   LEAD(Sales, 1)
   1           $100    NULL            $150
   2           $150    $100            $200
   3           $200    $150            NULL
```

---

## 4. Running Totals & Frame Specifications
When you write `SUM(amount) OVER (ORDER BY date)`, SQL calculates a running total. It does this because of a default frame boundary.

### The Frame Syntax (`ROWS` vs. `RANGE`):
```sql
ROWS BETWEEN [Start_Boundary] AND [End_Boundary]
```

Boundary options include:
*   `UNBOUNDED PRECEDING`: The very first row of the partition.
*   `CURRENT ROW`: The active row.
*   `N PRECEDING`: $N$ rows before.
*   `UNBOUNDED FOLLOWING`: The very last row of the partition.

#### 1. Running Total (Default Frame):
`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
This calculates the sum from the first row up to the current row.

#### 2. Moving Average (Sliding Window):
`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`
This sums the current row and the 2 prior rows, dividing by 3 (calculating a 3-row moving average).

---

## 5. Practice Question Bank

### Beginner Question
You have a table `employees` (employee_id, name, department_id, salary). Write a query using a window function to display each employee's name, department, salary, and the average salary of their department.
*   **Solution Walkthrough**:
    1.  We need to display individual rows, so we cannot use `GROUP BY`.
    2.  We use the window function `AVG(salary) OVER (...)`.
    3.  We partition by `department_id` to get regional averages.
    4.  **Correct Query**:
        ```sql
        SELECT 
            name, 
            department_id, 
            salary,
            AVG(salary) OVER (PARTITION BY department_id) AS dept_average_salary
        FROM employees;
        ```

### Intermediate Question
You have a table `monthly_sales` (year, month, sales). Write a query to display:
1.  The sales for the current month.
2.  The sales for the previous month (using `LAG`).
3.  The absolute revenue growth compared to the prior month.
Sort the final output chronologically.
*   **Solution Walkthrough**:
    1.  Sort the window by year and month to ensure correct lag ordering.
    2.  Calculate the lag: `LAG(sales, 1) OVER (ORDER BY year, month)`.
    3.  Subtract the lag from the current month's sales to get growth.
    4.  **Correct Query**:
        ```sql
        WITH sales_lag AS (
            SELECT 
                year, 
                month, 
                sales,
                LAG(sales, 1) OVER (ORDER BY year, month) AS prior_month_sales
            FROM monthly_sales
        )
        SELECT 
            year, 
            month, 
            sales,
            prior_month_sales,
            sales - COALESCE(prior_month_sales, 0) AS revenue_growth
        FROM sales_lag
        ORDER BY year, month;
        ```

### Advanced Question
You have a table `stock_prices` (ticker, date, close_price).
Write a query to calculate:
1.  The 5-day moving average of the closing price for each ticker.
2.  Filter the results to show only dates where the closing price was *above* the 5-day moving average.
*   **Solution Walkthrough**:
    1.  Create a CTE to calculate the moving average.
    2.  Partition by `ticker` and sort by `date` inside the window.
    3.  Specify the frame: `ROWS BETWEEN 4 PRECEDING AND CURRENT ROW` (5 days total).
    4.  In the main query, filter where `close_price > moving_average`.
    5.  **Correct Query**:
        ```sql
        WITH moving_averages AS (
            SELECT 
                ticker,
                date,
                close_price,
                AVG(close_price) OVER (
                    PARTITION BY ticker 
                    ORDER BY date 
                    ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
                ) AS moving_avg_5day
            FROM stock_prices
        )
        SELECT ticker, date, close_price, moving_avg_5day
        FROM moving_averages
        WHERE close_price > moving_avg_5day
        ORDER BY ticker, date;
        ```
        *(Note: If a ticker has fewer than 5 rows, the average is calculated using whatever rows are available in the frame).*
