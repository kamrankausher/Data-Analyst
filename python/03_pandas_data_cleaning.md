# Chapter 3: Data Cleaning with Pandas

Real-world datasets contain anomalies. Before running models or producing reports, you must treat missing values, drop duplicates, cast wrong data types, and merge lookup tables together. This chapter covers the engineering mechanics of data cleaning in Pandas.

---

## 1. Missing Value Treatment (`NaN` / `None`)
Pandas represents missing data using `NaN` (Not a Number, a float object) or `None` (Python object).

### A. Diagnosis
*   `df.isna().sum()`: Counts missing values per column.

### B. Treatment Strategies
1.  **Drop Rows/Columns**:
    *   `df.dropna(subset=['Salary'])`: Drops rows only if the `Salary` column is missing.
2.  **Imputation (Filling)**:
    *   `df['Salary'] = df['Salary'].fillna(df['Salary'].median())`: Fills missing values with the column median.
    *   `df['Category'] = df['Category'].fillna('Unknown')`: Fills with a string literal.
    *   `df['Stock'].ffill()`: Forward-fills (propagates the last valid observation forward). Excellent for time-series.

---

## 2. Removing Duplicates
*   `df.duplicated()`: Returns a boolean series where `True` indicates a duplicate row.
*   `df.drop_duplicates(subset=['Email'], keep='first', inplace=True)`: Removes duplicate rows based on the `Email` column, keeping the first occurrence.

---

## 3. Data Type Parsing & Standardizing

### A. Safe Numeric Casting (`errors='coerce'`)
If a numeric column contains text values (e.g. `['10.5', 'FREE', '20.0']`), a standard conversion will fail.
*   *The Coerce Solution*: Set `errors='coerce'`. This converts non-numeric strings into `NaN`, which can then be filled.
    ```python
    df['Price'] = pd.to_numeric(df['Price'], errors='coerce')
    ```

### B. Datetime Casting & Extraction
Parse strings to timestamps to unlock date-based logic:
```python
# Convert to Datetime
df['Date'] = pd.to_datetime(df['Date'], format='%Y-%m-%d')

# Extract components using the .dt accessor
df['Year'] = df['Date'].dt.year
df['MonthName'] = df['Date'].dt.strftime('%B')
```

---

## 4. Lambda Expressions & Custom row mappings
For custom, element-wise transformations:

### A. Element-wise Mapping
```python
# Map categories
df['Gender'] = df['Gender'].map({'M': 'Male', 'F': 'Female'})
```

### B. Lambda Operations (`.apply()`)
```python
# Calculate custom shipping fee based on distance
df['ShippingFee'] = df['Distance'].apply(lambda x: 0.0 if x <= 5 else 5.0 + (x - 5) * 0.5)
```

---

## 5. Combining DataFrames (Joins & Unions)

### A. Merging (Horizontal Join)
Replicates SQL Joins.
```python
df_merged = pd.merge(df_orders, df_customers, on='CustomerID', how='left')
```
*   `how` options: `'inner'`, `'left'`, `'right'`, `'outer'`.

### B. Concatenating (Vertical Stack)
Replicates SQL Unions.
```python
df_all = pd.concat([df_2025, df_2026], axis=0, ignore_index=True)
```
*   `ignore_index=True`: Resets the index numbers consecutively for the combined DataFrame.

---

## 6. Practice Question Bank

### Beginner Question
You have a DataFrame of transactions. The `Amount` column contains values stored as strings with currency symbols and commas (e.g., `"$1,250.50"`). Write the code to clean this column and convert it to float.
*   **Solution Walkthrough**:
    1.  The column is a string (object) type. Use the `.str` accessor.
    2.  Remove the dollar sign: `.str.replace('$', '', regex=False)`.
    3.  Remove the commas: `.str.replace(',', '', regex=False)`.
    4.  Cast to float: `pd.to_numeric(..., errors='coerce')`.
    5.  **Correct Code**:
        ```python
        df['Amount'] = df['Amount'].str.replace('$', '', regex=False)
        df['Amount'] = df['Amount'].str.replace(',', '', regex=False)
        df['Amount'] = pd.to_numeric(df['Amount'], errors='coerce')
        ```

### Intermediate Question
Explain what `errors='coerce'` does in `pd.to_numeric()`. What is the status of a cell that originally contained `"N/A"` after this function is applied, and how do you handle it?
*   **Solution Walkthrough**:
    1.  By default, `pd.to_numeric()` raises a ValueError and stops execution if it encounters a non-numeric string like `"N/A"`.
    2.  Setting `errors='coerce'` forces Pandas to convert any non-numeric value into `NaN` (Not a Number) instead of throwing an error.
    3.  The cell containing `"N/A"` becomes a `NaN` float value.
    4.  To handle the resulting `NaN`, use `.fillna()` to set a default value (e.g., `0.0`).
    5.  **Correct Code**:
        ```python
        df['Column'] = pd.to_numeric(df['Column'], errors='coerce').fillna(0.0)
        ```

### Advanced Question
You have two DataFrames:
- `sales`: contains columns `order_id`, `customer_id`, `amount`.
- `customers`: contains columns `customer_id`, `country`, `segment`.
Write a script to:
1.  Perform a left join to add customer country and segment to the sales records.
2.  Identify the count of sales transactions that have missing customer details (unmatched IDs).
3.  Impute missing countries with `"International"` and missing segments with `"Unknown"`.
*   **Solution Walkthrough**:
    1.  Perform left merge: `merged = pd.merge(sales, customers, on='customer_id', how='left')`.
    2.  Identify transactions missing customer details: these will have `NaN` values in the `country` column. Count them: `merged['country'].isna().sum()`.
    3.  Impute missing values using `.fillna()` with a dictionary.
    4.  **Correct Code**:
        ```python
        # 1. Left join
        merged = pd.merge(sales, customers, on='customer_id', how='left')
        
        # 2. Count missing details
        missing_count = merged['country'].isna().sum()
        print("Transactions missing customer details:", missing_count)
        
        # 3. Impute specific columns
        merged = merged.fillna({
            'country': 'International',
            'segment': 'Unknown'
        })
        ```
        *(Using a dictionary inside fillna is the cleanest way to apply different default values to different columns).*
