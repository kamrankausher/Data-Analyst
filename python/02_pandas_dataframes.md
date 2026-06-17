# Chapter 2: Pandas DataFrames & Series

Pandas is the core library for data analysis in Python. It provides high-performance, easy-to-use data structures (Series and DataFrames) and data analysis tools. This chapter covers the architecture of Series and DataFrames, file importing, indexing (loc/iloc), and logical filtering.

---

## 1. Pandas Data Structures

### A. Series (1-Dimensional Labeled Array)
A **Series** is similar to a 1D NumPy array but has an **Index** that labels each element. The index can be integers, strings, or datetimes.
```python
import pandas as pd
s = pd.Series([100, 200, 300], index=['Store A', 'Store B', 'Store C'])
```

### B. DataFrame (2-Dimensional Tabular Grid)
A **DataFrame** is a 2D table (rows and columns) similar to an Excel sheet. Each column in a DataFrame is a Series that shares the same row index.

```
       DataFrame structure:
       
                 Column Index (Headers)
                 'Name'    'Age'     'Salary'
       Row    0  [ Alice |  25   |   70000   ]
       Index  1  [ Bob   |  30   |   85000   ]
```

---

## 2. Ingesting & Exporting Datasets
Pandas can natively interface with dozens of data formats:

```python
# Reading files
df = pd.read_csv("sales.csv")
df_excel = pd.read_excel("financials.xlsx", sheet_name="Sheet1")
df_sql = pd.read_sql("SELECT * FROM orders", connection_object)

# Writing files
df.to_csv("clean_sales.csv", index=False) # index=False prevents writing the row index numbers
```

---

## 3. Data Inspection Checklist
Run this checklist immediately after loading a dataset:

1.  `df.head(n)`: Returns the first `n` rows (defaults to 5).
2.  `df.tail(n)`: Returns the last `n` rows.
3.  `df.shape`: Returns a tuple `(rows, columns)`.
4.  `df.info()`: Displays data types, non-null counts, and memory usage. Essential for identifying missing values and incorrect types.
5.  `df.describe()`: Generates summary statistics (mean, standard dev, min, max, percentiles) for all numeric columns.
6.  `df['Column'].value_counts()`: Counts the frequency of unique values in a categorical column.

---

## 4. Selecting and Filtering Data

### A. loc vs. iloc Indexing
*   `loc`: **Label-based** indexing. You reference rows and columns by their name/labels.
*   `iloc`: **Integer-based** indexing. You reference rows and columns by their numerical index position (starting from 0).

```python
# Sample DataFrame df:
#    EmpID   Name   Department   Salary
# 0  101     Alice  Tech         70000
# 1  102     Bob    HR           85000

# Using loc: Get Name of row index 0
print(df.loc[0, 'Name']) # Output: Alice

# Using iloc: Get Name of row index 0 (Col index 1)
print(df.iloc[0, 1]) # Output: Alice
```

*   **Slicing ranges**:
    *   `df.loc[0:2, 'Name':'Salary']` (loc is **inclusive** of the end boundary!).
    *   `df.iloc[0:2, 1:3]` (iloc is **exclusive** of the end boundary!).

### B. Boolean Indexing (Filtering Rows)
To filter rows, pass conditions inside brackets:
*   `&` (AND)
*   `|` (OR)
*   `~` (NOT)

> [!WARNING]
> You must wrap each individual condition in parentheses `()` to avoid operator precedence errors (since bitwise operators have higher priority than comparison operators in Python).

```python
# Find employees in Tech earning more than $60,000
tech_high_earners = df[(df['Department'] == 'Tech') & (df['Salary'] > 60000)]
```

---

## 5. Practice Question Bank

### Beginner Question
You have loaded a dataset. The column `ID` contains values that should be treated as strings, but Pandas has loaded them as integers.
1.  Explain why this is an issue.
2.  Write the code to convert the column data type to String.
*   **Solution Walkthrough**:
    1.  If IDs are stored as integers, you cannot perform string operations (like checking if an ID starts with a specific prefix). Additionally, leading zeros will be stripped.
    2.  To convert types, use `.astype(str)`.
    3.  **Correct Code**:
        ```python
        df['ID'] = df['ID'].astype(str)
        ```

### Intermediate Question
Trace the difference between the outputs of these two indexing commands:
1.  `df.loc[0:3, 'Name']`
2.  `df.iloc[0:3, 1]` (Assuming 'Name' is column index 1).
*   **Solution Walkthrough**:
    1.  `loc` is label-based. The slice `0:3` returns row indexes labeled `0, 1, 2, 3` (4 rows total).
    2.  `iloc` is integer-based. The slice `0:3` returns row index positions `0, 1, 2` (3 rows total, excluding the end boundary).
    3.  **Answer**:
        *   `loc` returns 4 rows.
        *   `iloc` returns 3 rows.

### Advanced Question
You have a customer transactions DataFrame `df` with columns: `Country`, `Revenue`, `Age`, `Status`.
Write a query to retrieve only the `Age` and `Revenue` columns for customers who meet **all** of the following conditions:
1.  Country is either "Germany", "France", or "Spain".
2.  Revenue is greater than $10,000.
3.  Status is not "Cancelled".
*   **Solution Walkthrough**:
    1.  Condition 1: Use `.isin()` for matching a list: `df['Country'].isin(['Germany', 'France', 'Spain'])`.
    2.  Condition 2: `df['Revenue'] > 10000`.
    3.  Condition 3: `df['Status'] != 'Cancelled'` or `~(df['Status'] == 'Cancelled')`.
    4.  Combine all conditions using `&` inside `loc` to filter rows and select columns:
        `df.loc[Row_Conditions, Column_List]`.
    5.  **Correct Code**:
        ```python
        row_mask = (
            (df['Country'].isin(['Germany', 'France', 'Spain'])) &
            (df['Revenue'] > 10000) &
            (df['Status'] != 'Cancelled')
        )
        filtered_df = df.loc[row_mask, ['Age', 'Revenue']]
        ```
