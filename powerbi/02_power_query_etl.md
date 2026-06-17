# Chapter 2: Power Query ETL in Power BI

Power Query is the Extract, Transform, Load (ETL) engine of Power BI. Before building models, you must clean your data. This chapter covers essential data cleaning transformations, merging/appending, Query Folding, and the M formula language.

---

## 1. Power Query Transformations

### A. Core Cleaning Tasks:
*   **Split Column by Delimiter**: Split text strings (e.g. splitting `John-Smith` into `John` and `Smith` using the hyphen delimiter).
*   **Merge Columns**: Combines multiple text fields into a single column.
*   **Fill Down / Fill Up**: Replaces null values in a column with the first prior non-null value. Excellent for parsing merged header files from legacy reporting systems.
*   **Transform text format**: Trim (removes spaces), Clean (removes non-printable characters), Upper/Lower/Proper casing.

### B. Unpivoting Columns (Wide to Tall)
Wide datasets are human-readable, but they break relational data models. Databases prefer tall structures.
*   *Action*: Select the identifier column (e.g., `Product_ID`), right-click, and choose **Unpivot Other Columns**. Power Query transforms month/year column headers into attribute rows.

---

## 2. Merging and Appending Queries

### A. Merging (SQL Joins)
Combines two queries horizontally based on a key column.
*   **Left Outer**: Keeps all records from the first table and matching records from the second.
*   **Inner**: Keeps only matching rows from both tables.
*   **Left Anti**: Keeps rows from the first table that do **not** have matching records in the second. Perfect for identifying orphan IDs (e.g., transactions referencing non-existent product codes).

### B. Appending (SQL Unions)
Stacks queries vertically. Column names must match exactly (case-sensitive). If columns differ, Power Query creates separate columns filled with `null` values where data is missing.

---

## 3. Query Folding (The Optimization Engine)
Query Folding is the process where Power Query translates your data transformations (filtering, column removal, grouping) into a single SQL statement, pushing the calculation workload down to the source database server.

```
   [ Power Query Steps ]                   [ Source Database ]
   1. Filter Active Users                  Translates to:
   2. Select ID and Name      =======>     SELECT ID, Name 
   3. Group by ID                          FROM Users 
                                           WHERE Status = 'Active'
```

*   **Why it matters**: It prevents Power BI from downloading millions of raw rows and filtering them in your computer's memory. The database server does the work and sends only the filtered results.
*   **How to check**: Right-click a step in the Applied Steps list. If **View Native Query** is clickable, the step is folded. If it is greyed out, the fold is broken, and subsequent steps must run in local memory.
*   **Fold Breakers**: Functions like `Index Column`, certain custom M code, or converting casing of non-SQL sources break folding. Always place folding steps (like filters and column selections) at the very top of your steps list.

---

## 4. The M Language: `let...in` Blocks
Every step in Power Query generates code in the **M Language**. You can view and edit this by clicking **Home** > **Advanced Editor**.

### Structure of an M Script:
```powerquery
let
    Source = Sql.Database("ServerName", "DBName"),
    FilterRows = Table.SelectRows(Source, each [Status] = "Active"),
    RemoveCols = Table.RemoveColumns(FilterRows, {"Password"})
in
    RemoveCols
```

*   `let`: Defines steps (variables). Each step references the name of the prior step.
*   `in`: Defines which step's output is returned to the data model.
*   **Syntax**: M is **case-sensitive** (`Table.SelectRows` is correct; `table.selectrows` returns an error). Step names with spaces are written as `#"Step Name"`.

---

## 5. Practice Question Bank

### Beginner Question
You have loaded a table in Power Query. The date column contains string values formatted as `"06-17-2026"`. You change the data type to Date, but half of the rows return a `[Error]` value.
1.  Explain why this error occurs.
2.  How do you resolve it?
*   **Solution Walkthrough**:
    1.  The error occurs because your computer's local system settings (Locale) use a different date format (e.g., `DD-MM-YYYY`), causing Power Query to fail when parsing the American format `MM-DD-YYYY` (e.g. interpreting 17 as the month).
    2.  To fix it: Right-click the column, delete the failed Change Type step.
    3.  Right-click the column again, select **Change Type** > **Using Locale...**.
    4.  Set Data Type to **Date**, and set the Locale to **English (United States)**.
    5.  Power Query will parse the strings using the US format and convert them without errors.

### Intermediate Question
You want to merge a `Sales` query with a `Rep_Catalog` query based on the `RepCode` column. However, `Sales` contains codes in lowercase (e.g., `"a101"`) and `Rep_Catalog` contains codes in uppercase (e.g., `"A101"`). What will happen if you merge them, and how do you resolve it?
*   **Solution Walkthrough**:
    1.  Power Query is case-sensitive. If you merge them directly, the keys `"a101"` and `"A101"` will not match, returning null values.
    2.  To resolve this, you must standardize the keys before merging.
    3.  In the `Sales` query, select the `RepCode` column, go to **Transform** > **Format** > **Uppercase**.
    4.  In the `Rep_Catalog` query, ensure `RepCode` is also uppercase.
    5.  Merge the queries. The records will now match.

### Advanced Question
You are importing a 100-million-row SQL database. You need to filter the data to only include rows from the year 2026. Explain how you would write this transformation in Power Query to ensure that the database server performs the filter and Power BI does not download all 100 million rows.
*   **Solution Walkthrough**:
    1.  We need to use **Query Folding**.
    2.  When connecting to the SQL Server, do **not** write a custom SQL statement in the connection box (writing custom SQL blocks query folding for subsequent steps in Power BI).
    3.  Instead, connect directly to the table using the GUI.
    4.  In Power Query, immediately select the `Date` column, click the filter arrow, select **Date Filters** > **Between...**, and set the range from `1/1/2026` to `12/31/2026`.
    5.  Because this is a standard filter step, Power Query will fold this step.
    6.  Right-click the filter step and click **View Native Query** to confirm that the generated SQL statement contains a `WHERE date_column BETWEEN ...` filter.
    7.  The database server will filter the data and send only the 2026 subset to Power BI.
