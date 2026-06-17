# Chapter 5: Power Query & Data Modeling in Excel

As business datasets grow into millions of rows, standard Excel grids crash. To build scalable models, analysts use **Power Query** (to clean and reshape data) and **Power Pivot** (to build relational models and write DAX measures). This chapter covers these enterprise-grade tools.

---

## 1. Power Query: The ETL Engine
Power Query is an ETL (Extract, Transform, Load) tool. Every transformation step you click (e.g., renaming a column, changing a data type) is compiled into a script written in the **M Language** and saved in the **Applied Steps** panel.

*   **Applied Steps Automation**: When new raw data is added to your source, clicking **Refresh** re-runs the M script on the new data automatically.
*   **The Close & Load To Rule**: When closing Power Query, you have two choices:
    1.  *Close & Load*: Loads data into a physical Excel sheet (limited to 1 million rows).
    2.  *Close & Load To...* > *Only Create Connection* + *Add this data to the Data Model*: Bypasses the Excel grid entirely and loads data directly into the internal compressed database memory.

---

## 2. Advanced Power Query Transformations

### A. Unpivoting Columns (Wide to Tall Format)
Human-readable spreadsheets are often wide (e.g. columns for different months). Databases require tall formats.

```
   Wide Format (Bad for Analysis):
   Product_ID | Jan_Sales | Feb_Sales | Mar_Sales
   101        | 100       | 150       | 200
   
   Tall Format (Normalized/Correct):
   Product_ID | Month     | Sales
   101        | Jan_Sales | 100
   101        | Feb_Sales | 150
   101        | Mar_Sales | 200
```

*   **How to Unpivot**: Select the non-value columns (e.g. `Product_ID`), right-click, and choose **Unpivot Other Columns**.

### B. Query Merging vs. Appending
*   **Merge Queries**: Horizontal join. Merges two queries based on a matching key column (similar to SQL Joins).
*   **Append Queries**: Vertical stack. Combines rows from multiple files or queries (similar to SQL Union). Column names must match.

---

## 3. Power Pivot: The Relational Data Model
Power Pivot allows you to create a relational database schema inside Excel.

*   **The VertiPaq Engine**: Power Pivot uses a columnar storage engine. Instead of storing data row-by-row, it stores columns. This allows it to compress repeating data values heavily (e.g., repeating country names are compressed to a single index key). This allows Excel to handle **100+ million rows** in memory.
*   **Diagram View**: In the Power Pivot window, switch to Diagram View to establish relationships. Drag the primary key of your Lookup (Dimension) table (e.g., `ProductID` in the `Products` table) and drop it onto the foreign key of your transaction (Fact) table (e.g., `ProductID` in the `Sales` table).

---

## 4. DAX (Data Analysis Expressions) Basics
DAX is the formula language used in Power Pivot and Power BI.

### A. Calculated Columns vs. Measures
*   **Calculated Column**: Evaluated row-by-row during data refresh. Consumes disk space and RAM. Use only when you need a category to slice data by in filters or row headers.
*   **Measure**: Evaluated on the fly in response to the filter context in your Pivot Table. Consumes no disk space. This is the professional standard for quantitative calculations.

### B. Crucial DAX Functions
*   **Safe Division (`DIVIDE`)**: Prevents `#DIV/0!` errors when calculating ratios.
    `Margin % := DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)`
*   **Filter Override (`CALCULATE`)**: Modifies the filter context.
    `US Sales := CALCULATE(SUM(Sales[Revenue]), Customers[Country] = "USA")`

---

## 5. Practice Question Bank

### Beginner Question
You have two tables: `Sales_2025` and `Sales_2026`. Both tables have columns `Order_ID`, `Product_ID`, and `Amount`. You want to combine these tables into a single master sales table. Explain the step-by-step process in Power Query.
*   **Solution Walkthrough**:
    1.  Load both tables into Power Query (**Data** > **From Table/Range**).
    2.  Check that column names and data types in both queries match exactly (case-sensitive).
    3.  Select `Sales_2025` as the primary query.
    4.  Go to **Home** > **Append Queries** > select **Append Queries as New**.
    5.  Select the two tables and click OK.
    6.  Rename the new query to `Master_Sales`.
    7.  Click **Close & Load**.

### Intermediate Question
You have a `Sales` table containing 5 million transactions and a `Product_Catalog` table containing product cost details. How do you construct a model to calculate total product costs for all transactions without using `VLOOKUP` or running out of memory?
*   **Solution Walkthrough**:
    1.  Load both tables into Power Query.
    2.  Instead of merging them or loading to a sheet, select **Close & Load To...** > select **Only Create Connection** and check **Add this data to the Data Model**.
    3.  Go to **Power Pivot** > **Manage**.
    4.  Switch to Diagram View.
    5.  Drag `ProductID` from the `Product_Catalog` table (Lookup) to `ProductID` in the `Sales` table (Fact), establishing a 1-to-many relationship.
    6.  Insert a Pivot Table using the Data Model.

### Advanced Question
You have built a relationship between `Sales` and `Product_Catalog` (on `ProductID`). You want to write a DAX measure that calculates the total sales revenue (calculated as `Sales[Quantity] * Product_Catalog[UnitPrice]`) for all transactions. Note that `UnitPrice` is only stored in the lookup table `Product_Catalog`.
*   **Solution Walkthrough**:
    1.  Because we need to multiply columns from two different tables row-by-row, we must use an iterator function: `SUMX`.
    2.  `SUMX` takes a table as its first argument: `Sales`.
    3.  The second argument is the row-level expression.
    4.  To pull the `UnitPrice` from the related lookup table during the row loop, we must use the `RELATED` function: `Sales[Quantity] * RELATED(Product_Catalog[UnitPrice])`.
    5.  **Correct DAX Measure**:
        `Total Revenue := SUMX(Sales, Sales[Quantity] * RELATED(Product_Catalog[UnitPrice]))`
        *(Without `RELATED`, the row loop in the Sales table cannot access columns in the Product_Catalog table)*.
