# Python Data Analyst Interview Prep Guide

This guide contains the most frequently asked Python and Pandas interview questions in Data Analyst and Data Science interviews. Each question includes a detailed, professional answer.

---

## 1. Memory Management & Vectorization

### Question 1: What is the difference between a Python List and a NumPy Array? Why is NumPy faster?
*   **Python List**: An array of pointers to objects scattered in memory. Each object is boxed (contains metadata for type and references).
    *   *Downside*: Iterating over lists requires Python to check the data type of each element, which adds significant overhead. It also suffers from cache misses because the data points are not contiguous in RAM.
*   **NumPy Array**: A contiguous block of raw homogeneous data (same data type).
    *   *Why it is faster*: Homogeneity allows the compiler to perform calculations without type checking. Contiguous memory layout allows CPU caches to load blocks of data at once. It uses compiled C code and supports vectorization (SIMD) to run operations on multiple data points simultaneously.

---

## 2. Pandas Slicing & Indexing

### Question 2: Explain the difference between `loc` and `iloc` in Pandas. What happens if you slice a range like `0:3` using both?
*   **loc**: Label-based indexing.
    *   `df.loc[0:3, 'ColumnName']` returns rows with index **labels** `0, 1, 2, 3` (inclusive of the end boundary).
*   **iloc**: Integer-based indexing.
    *   `df.iloc[0:3, 1]` returns rows at index **positions** `0, 1, 2` (exclusive of the end boundary).
*   **Key Interview Point**: *"Always specify column names explicitly in loc for readability and model stability, rather than relying on index positions in iloc, which can break if the source data schema changes."*

---

## 3. Data Wrangling & Optimization

### Question 3: How do you optimize memory usage in Pandas when loading a large dataset?
*   **Answer Checklist**:
    1.  **Select columns**: Only load the columns you need: `pd.read_csv('data.csv', usecols=['ID', 'Sales'])`.
    2.  **Cast Categorical Data**: Convert columns with low cardinality (repeating text values like Country or Segment) to the `category` data type: `df['Country'] = df['Country'].astype('category')`. This replaces strings with small integer codes.
    3.  **Optimize Numerics**: Downcast numeric types. For example, convert 64-bit floats (`float64`) to 32-bit floats (`float32`), or `int64` to `int16`/`int8` using `pd.to_numeric(..., downcast='float')`.
    4.  **Chunking**: Read the file in smaller chunks: `pd.read_csv('data.csv', chunksize=100000)`.

---

## 4. Aggregations & Grouping

### Question 4: Explain the Split-Apply-Combine mechanism in Pandas GroupBy. How do you apply different aggregation functions to different columns simultaneously?
*   **Split**: Splits the DataFrame into subgroups based on key column values.
*   **Apply**: Evaluates an aggregation function (like sum or mean) for each group independently.
*   **Combine**: Merges the results back into a single output DataFrame.
*   **Custom Aggregations**: Pass a dictionary to the `.agg()` method:
    ```python
    df.groupby('Department').agg({
        'EmployeeID': 'count',
        'Salary': 'mean',
        'Sales': 'sum'
    })
    ```

---

## 5. Python Code Challenge

### Scenario:
You have a DataFrame `sales` with columns: `Date`, `CustomerID`, `Revenue`.
Write a script to calculate the month-on-month revenue growth rate percentage for the year 2026.
*   **Solution Walkthrough**:
    1.  Convert the `Date` column to datetime type.
    2.  Filter the DataFrame to include only rows from the year 2026.
    3.  Group transactions by month, and sum the `Revenue`.
    4.  Calculate prior month revenue using `.shift(1)`.
    5.  Compute the growth rate: `(Current - Prior) / Prior * 100`.
    6.  **Correct Code**:
        ```python
        import pandas as pd
        
        # 1. Parse date
        sales['Date'] = pd.to_datetime(sales['Date'])
        
        # 2. Filter 2026
        sales_2026 = sales[sales['Date'].dt.year == 2026].copy()
        
        # 3. Monthly sum
        sales_2026['Month'] = sales_2026['Date'].dt.to_period('M')
        monthly_revenue = sales_2026.groupby('Month')['Revenue'].sum().reset_index()
        
        # 4. Shift to get prior month
        monthly_revenue['Prior_Revenue'] = monthly_revenue['Revenue'].shift(1)
        
        # 5. Compute growth rate
        monthly_revenue['MoM_Growth_Pct'] = (
            (monthly_revenue['Revenue'] - monthly_revenue['Prior_Revenue']) / 
            monthly_revenue['Prior_Revenue']
        ) * 100
        print(monthly_revenue)
        ```
        *(Using .copy() prevents SettingWithCopyWarnings when creating the Month column).*
