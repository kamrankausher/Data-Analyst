# Chapter 4: Data Aggregation & Grouping in Pandas

Data analysis is the science of summarization. Pandas provides a powerful "Split-Apply-Combine" mechanism to aggregate data, alongside advanced pivot table and rolling window features that replicate the most advanced SQL and Excel operations.

---

## 1. The Split-Apply-Combine Mechanism
When you run `.groupby()` in Pandas:
1.  **Split**: Pandas splits the DataFrame into sub-tables (groups) based on key column values.
2.  **Apply**: Pandas applies an aggregation function (like sum or mean) to each group.
3.  **Combine**: Pandas compiles the results back into a single output DataFrame.

```
                  Split            Apply           Combine
               +---------+       +-------+
               | Tech 10 | =====>| Tech  |
   Tech  10    | Tech 20 |       | 30    |       +-------+------+
   Sales 15 => +---------+       +-------+ ====> | Tech  | 30   |
   Tech  20    +---------+       +-------+       | Sales | 15   |
   Sales 15    |Sales 15 | =====>| Sales |       +-------+------+
               |Sales 15 |       | 30    |
               +---------+       +-------+
```

*   *Basic syntax*:
    ```python
    # Calculate total sales by region
    regional_sales = df.groupby('Region')['Sales'].sum()
    ```

---

## 2. Advanced Aggregations (`.agg()`)
To calculate different metrics for different columns, pass a dictionary to `.agg()`:

```python
summary = df.groupby('Category').agg({
    'ProductID': 'count',       # Count of products in category
    'SalesPrice': ['min', 'max', 'mean'], # Multiple stats for price
    'Revenue': 'sum'            # Total revenue
})
```

---

## 3. Pivot Tables & Crosstabs

### A. Pivot Tables (`pd.pivot_table()`)
Replicates Excel's pivot grids.
*   `index`: Row headers.
*   `columns`: Column headers.
*   `values`: Column to aggregate.
*   `aggfunc`: Aggregation function (defaults to mean).

```python
pivot = df.pivot_table(
    index='Region',
    columns='ProductCategory',
    values='Revenue',
    aggfunc='sum',
    margins=True # Adds Grand Totals
)
```

### B. Cross-Tabulations (`pd.crosstab()`)
Calculates frequency counts (occurrences) between categorical variables.
```python
# Count of customer signups by country and traffic channel
crosstab_df = pd.crosstab(df['Country'], df['Channel'])
```

---

## 4. Window Functions in Pandas

### A. Shifting (Lag & Lead)
*   `.shift(1)`: Lag (grabs value from the row above).
*   `.shift(-1)`: Lead (grabs value from the row below).

```python
# Calculate day-over-day stock price change
df['Prior_Day_Price'] = df['Close_Price'].shift(1)
df['Price_Change'] = df['Close_Price'] - df['Prior_Day_Price']
```

### B. Rolling Windows
Calculates rolling metrics (e.g. moving averages) over a sliding window of rows.
```python
# Calculate 7-day moving average of sales
df['Sales_7day_MA'] = df['Sales'].rolling(window=7, min_periods=1).mean()
```
*   `min_periods=1`: Allows calculations to run even if the window has fewer than 7 rows (e.g. on row 3, it averages the 3 available rows instead of returning `NaN`).

---

## 5. Practice Question Bank

### Beginner Question
You have a DataFrame of transactions. Write a script to calculate the total revenue and the average customer rating for each store branch.
*   **Solution Walkthrough**:
    1.  Group by `Branch`.
    2.  Use `.agg()` with a dictionary mapping `Revenue` to `'sum'` and `Rating` to `'mean'`.
    3.  **Correct Code**:
        ```python
        branch_stats = df.groupby('Branch').agg({
            'Revenue': 'sum',
            'Rating': 'mean'
        })
        ```

### Intermediate Question
You have daily transaction records. You want to calculate the 30-day running total of sales. Write the script, and explain what steps you must take to ensure the running total is accurate.
*   **Solution Walkthrough**:
    1.  To calculate a running total, the data must be sorted chronologically.
    2.  Sort by `Date` first.
    3.  Use `.cumsum()` on the `Sales` column to compute the running sum.
    4.  **Correct Code**:
        ```python
        # Step 1: Sort chronologically
        df = df.sort_values('Date')
        # Step 2: Cumulative sum
        df['Running_Total_Sales'] = df['Sales'].cumsum()
        ```

### Advanced Question
You have customer activity logs with columns: `CustomerID`, `SessionDate`, and `PageViews`.
Write a script to calculate the change in `PageViews` for each customer compared to their *previous* session. Sort the data correctly so that the lag is calculated chronologically for each customer.
*   **Solution Walkthrough**:
    1.  First, sort the entire DataFrame by `CustomerID` and `SessionDate`.
    2.  If we apply `.shift(1)` directly to the entire pageviews column, we will accidentally calculate the difference using the last row of the *previous customer* for the first row of the *current customer*.
    3.  To prevent this, use `groupby('CustomerID')` first, and apply `.shift(1)` to each customer group independently.
    4.  **Correct Code**:
        ```python
        # Step 1: Sort data
        df = df.sort_values(['CustomerID', 'SessionDate'])
        
        # Step 2: Group by customer and shift within groups
        df['Prior_Session_Views'] = df.groupby('CustomerID')['PageViews'].shift(1)
        
        # Step 3: Compute difference
        df['PageView_Change'] = df['PageViews'] - df['Prior_Session_Views']
        ```
        *(This ensures that a customer's first session displays NaN for prior views instead of pulling data from a different customer).*
