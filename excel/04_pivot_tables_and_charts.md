# Chapter 4: Pivot Tables & Charts for Data Summarization

A Pivot Table is Excel’s most powerful tool. It allows you to take millions of rows of transaction data and summarize it into dynamic, interactive reports in seconds. This chapter explains the architecture of Pivot Tables, advanced calculated values, and charting best practices.

---

## 1. Pivot Table Architecture
To insert a Pivot Table: Select your data range, click **Insert** > **PivotTable**.
*   **Source Data Integrity**:
    1.  Every column **must** have a header.
    2.  No empty rows or columns inside the table.
    3.  Columns must contain consistent data types (e.g. do not mix dates and text in the same column).

### The Four Quadrants:

```
+-----------------------------------------------------+
|                  PivotTable Fields                  |
+-----------------------------------------------------+
| FILTERS                  | COLUMNS                  |
| (Filters the entire      | (Creates horizontal      |
| report by this field)    | headings in columns)     |
+--------------------------+--------------------------+
| ROWS                     | VALUES                   |
| (Creates vertical lists  | (Calculates data:        |
| of unique row keys)      | SUM, COUNT, AVERAGE...)  |
+--------------------------+--------------------------+
```

---

## 2. Advanced Pivot Calculations

### A. Summarize Values By
By default, Excel sums numbers and counts text. You can change this by right-clicking a value in your Pivot Table and selecting **Summarize Values By**:
*   `SUM`: Total.
*   `AVERAGE`: Mean.
*   `COUNT`: Count of records.
*   `PRODUCT`: Multiplies values.

### B. Show Values As (Percentage & Growth Calculations)
Instead of displaying raw sums, you can calculate ratios. Right-click the value column > **Show Values As**:
*   `% of Grand Total`: Useful for market share calculations.
*   `% of Column Total`: Useful for understanding category mix.
*   `% of Row Total`: Useful for demographic distributions.
*   `Difference From` / `% Difference From`: Compares values to a baseline (e.g., comparing monthly sales to January's sales).
*   `Running Total In`: Computes a cumulative running sum (essential for tracking budget spend over time).

---

## 3. Dynamic Grouping (Binning)
You can group data in a Pivot Table without changing the source table.

### A. Grouping Dates
If you drag a `Date` field to Rows, Excel might display every single day.
1.  Right-click any date cell in your Pivot Table.
2.  Select **Group...**.
3.  Choose your granularity: **Years**, **Quarters**, or **Months**. Excel will automatically group the rows.

### B. Grouping Numbers (Bins)
Used to group numeric values into brackets (e.g. analyzing sales counts by price brackets):
1.  Drag your numeric variable (e.g. `Price`) to **Rows**.
2.  Right-click any price value in the Pivot Table and select **Group...**.
3.  Set the starting value (e.g., `0`), ending value (e.g., `100`), and increment size (e.g., `10`).
4.  Excel will create ranges: `0-10`, `10-20`, `20-30`, etc.

---

## 4. Visual Filters: Slicers & Timelines
Slicers and Timelines are visual, click-to-filter dashboards elements.

*   **Slicers**: Categorical filters (e.g. Filter by Region, Sales Rep).
*   **Timelines**: Interactive date sliders.
*   **The Report Connections Rule**: By default, a slicer only filters the Pivot Table it was created from. To make a single slicer filter multiple Pivot Tables:
    1.  Right-click the Slicer and select **Report Connections...**.
    2.  Check the boxes for all Pivot Tables on the sheet that you want to filter with this Slicer.

---

## 5. Visualizing Data: Dashboard Design Principles
A Pivot Chart is a visual representation of your Pivot Table.

### The Chart Selection Matrix:
*   **Line Chart**: Best for displaying trends over time. (Never use line charts for categorical data).
*   **Bar/Column Chart**: Best for comparing categories (e.g. Sales by Product). Use horizontal bars if category names are long.
*   **Pie Chart**: Avoid unless you have fewer than 3 categories. Never use pie charts for time-series.
*   **Scatter Plot**: Best for showing relationships between two continuous numeric variables.

---

## 6. Practice Question Bank

### Beginner Question
You have transaction-level sales data. You want to build a Pivot Table that displays the average order value and the total order count for each Product Category. Explain which fields to drag into which quadrants.
*   **Solution Walkthrough**:
    1.  We need unique Product Categories as row headers. Drag `Product Category` to **Rows**.
    2.  We need to calculate both average order value and total order count.
    3.  Drag `Revenue` to **Values** first. Right-click it, select **Summarize Values By** > **AVERAGE**.
    4.  Drag `Revenue` (or `OrderID`) to **Values** a second time. Right-click it, select **Summarize Values By** > **COUNT**.

### Intermediate Question
You have monthly sales data for 2026. You want to display month-on-month sales growth as a percentage (i.e. how much sales changed in February compared to January). Explain the steps to set up the value field calculation.
*   **Solution Walkthrough**:
    1.  Drag `Date` (grouped by Months) to **Rows**.
    2.  Drag `Sales` to **Values**.
    3.  Right-click the Sales value in the Pivot Table, select **Show Values As** > **% Difference From...**.
    4.  Set the Base Field to **Date (Months)**.
    5.  Set the Base Item to **(previous)**.
    6.  Excel will calculate the percentage change between the current month and the prior month.

### Advanced Question
You have a Pivot Table showing total sales by customer. You want to analyze the distribution of customer purchases. Specifically, you want to see the count of customers who have made total purchases in brackets of $1,000 (e.g., $0-$1,000, $1,000-$2,000). How do you configure this?
*   **Solution Walkthrough**:
    1.  Create a Pivot Table.
    2.  Drag `CustomerID` to **Rows**.
    3.  Drag `Sales` to **Values** (set to SUM).
    4.  Since we need to group the sum of sales *by customer*, we cannot group directly in this pivot. We must first copy the summary data or load it to the Data Model.
    5.  *Alternatively (if we have customer-level sales in our source table)*: Drag `Sales` directly to **Rows** and `CustomerID` (set to COUNT) to **Values**.
    6.  Right-click any value in the Rows column (Sales) and select **Group...**.
    7.  Set Starting at `0`, Ending at max sales (e.g., `10000`), and By `1000`.
    8.  Excel will create the bins: `0-1000`, `1000-2000`, etc.
