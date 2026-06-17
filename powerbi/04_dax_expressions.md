# Chapter 4: DAX (Data Analysis Expressions) in Power BI

DAX is the programming language that powers Power BI's calculations. To write accurate reports, you must master Row Context, Filter Context, and Context Transition. This chapter covers these concepts in detail, alongside advanced filter overrides and time-intelligence calculations.

---

## 1. The Core Engine: DAX Contexts

### A. Row Context (Row-by-Row Loop)
*   **What it is**: The concept of the "current row". 
*   **Where it exists**:
    1.  Inside **Calculated Columns** (e.g. `Profit = Sales[Revenue] - Sales[Cost]`).
    2.  Inside **Iterator Functions** (like `SUMX`, `AVERAGEX`).
*   **Limitation**: A Row Context does **not** see the filter context. It cannot calculate aggregates across rows on its own.

### B. Filter Context (Visual Coordinates)
*   **What it is**: The active filters on your report page (defined by slicers, table row/column headers, and page filters).
*   **Where it exists**: Inside **Measures**.
*   *Example*: If you place a measure `Total Revenue = SUM(Sales[Amount])` in a matrix visual with `Region` as the row header, the filter context filters the `Sales` table to only include rows for the active region *before* calculating the sum.

### C. Context Transition (The Bridge)
Context Transition occurs when you convert a Row Context into a Filter Context.
*   **How to trigger it**: Wrap your calculation in the `CALCULATE` function.
*   *Without Context Transition*: If you write a calculated column `DeptAvg = AVG(Employees[Salary])`, every row will display the average of the *entire company*.
*   *With Context Transition*: If you write `DeptAvg = CALCULATE(AVG(Employees[Salary]))`, the row context (which knows the department of the current row) is converted into a filter context, filtering the table to only that department. The column will display the average for each employee's specific department.

---

## 2. Calculated Columns vs. Measures

| Feature | Calculated Column | Measure |
| :--- | :--- | :--- |
| **Evaluation Time**| During data refresh (Static). | When visual is rendered or clicked (Dynamic). |
| **Storage** | Stored in RAM & Disk (increases file size). | Calculated on the fly (no disk overhead). |
| **Context** | Operates in **Row Context**. | Operates in **Filter Context**. |
| **Use Case** | Slicing data (e.g. Age brackets, region groups).| Quantitative calculations (e.g. Sales, Margins). |

---

## 3. The Power of `CALCULATE`
`CALCULATE` is the most important function in DAX. It is the only function that can add, remove, or modify the Filter Context.
*   **Formula**: `CALCULATE(Expression, Filter1, Filter2, ...)`

### Overriding Page Filters:
If you have a slicer on the page filtering for the year `2025`, but you want a card visual to display sales for `2026`:
```dax
Sales 2026 = CALCULATE(SUM(Sales[Amount]), 'Calendar'[Year] = 2026)
```
*CALCULATE replaces the page filter on the Year column with 2026 for this measure.*

---

## 4. Iterators & Filter Modifiers

### A. Iterators (X-Functions)
Standard functions like `SUM` only accept a single column. If you need to perform a row-by-row calculation first (e.g. multiplying `Quantity` by `Price`) and then sum the results, you must use an iterator.
*   `SUMX(Table, Expression)`: Loops through the table, evaluates the expression for each row, and then sums the results.
    `Total Revenue = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])`

### B. Filter Modifiers: `ALL` and `ALLEXCEPT`
*   `ALL(Table/Column)`: Removes all filters from the target, returning the grand total.
*   `ALLEXCEPT(Table, Column)`: Removes all filters *except* the one on the specified column.

```dax
% Share of Total Sales = 
VAR CurrentCategorySales = SUM(Sales[Amount])
VAR AllCategorySales = CALCULATE(SUM(Sales[Amount]), ALL(Products[Category]))
RETURN
DIVIDE(CurrentCategorySales, AllCategorySales, 0)
```

---

## 5. Time Intelligence Functions
Time intelligence allows you to compare current sales against prior periods (requires a marked Calendar table).

*   **Year-to-Date (YTD)**:
    `Sales YTD = TOTALYTD(SUM(Sales[Amount]), 'Calendar'[Date])`
*   **Prior Year (YoY)**:
    `Sales LY = CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Calendar'[Date]))`
*   **Rolling 12-Month Sales**:
    ```dax
    Rolling 12M Sales = 
    CALCULATE(
        SUM(Sales[Amount]),
        DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), -1, YEAR)
    )
    ```

---

## 6. Practice Question Bank

### Beginner Question
You write a calculated column in the `Sales` table: `LineTotalColumn = Sales[Quantity] * Sales[UnitPrice]`. You also write a measure: `LineTotalMeasure = SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])`.
Explain the difference in performance and storage between these two approaches.
*   **Solution Walkthrough**:
    1.  `LineTotalColumn` is calculated row-by-row during data refresh. It adds a new column of data to your `Sales` table on disk and in memory (RAM). If the table has 10 million rows, this consumes significant memory.
    2.  `LineTotalMeasure` is a measure. It consumes zero disk space. It calculates the product row-by-row on the fly only when you place it in a visual.
    3.  *Recommendation*: Use the measure to preserve system resources.

### Intermediate Question
You want to display a card visual showing the total revenue for the category "Electronics" regardless of what category a user selects in a slicer. Write the DAX measure.
*   **Solution Walkthrough**:
    1.  We need to calculate `SUM(Sales[Amount])`.
    2.  We need to override the category filter using `CALCULATE`.
    3.  We must use `ALL` to clear any category filters applied by page slicers, then apply the "Electronics" filter.
    4.  **Correct Measure**:
        ```dax
        Electronics Sales = 
        CALCULATE(
            SUM(Sales[Amount]),
            ALL(Products[Category]),
            Products[Category] = "Electronics"
        )
        ```

### Advanced Question
Write a measure to calculate the month-on-month growth rate percentage of Sales. Ensure the calculation handles the first month of data (when prior month sales is null) without returning errors.
*   **Solution Walkthrough**:
    1.  Define a variable for current month sales: `VAR CurrentSales = SUM(Sales[Amount])`.
    2.  Define a variable for prior month sales using `DATEADD` to shift the calendar back by 1 month:
        `VAR PriorSales = CALCULATE(SUM(Sales[Amount]), DATEADD('Calendar'[Date], -1, MONTH))`.
    3.  Compute the growth rate: `(CurrentSales - PriorSales) / PriorSales`.
    4.  Use `DIVIDE` to handle cases where `PriorSales` is 0 or null.
    5.  **Correct Measure**:
        ```dax
        MoM Growth % = 
        VAR CurrentSales = SUM(Sales[Amount])
        VAR PriorSales = CALCULATE(SUM(Sales[Amount]), DATEADD('Calendar'[Date], -1, MONTH))
        RETURN
        DIVIDE(CurrentSales - PriorSales, PriorSales, 0)
        ```
