# Power BI Data Analyst Interview Prep Guide

This guide contains the most frequently asked Power BI interview questions in Data Analyst and Business Intelligence interviews. Each question includes a detailed, professional answer.

---

## 1. Storage & Architecture

### Question 1: What is Query Folding in Power Query? Why is it important, and what actions can break it?
*   **Query Folding**: The process where Power Query translates data transformations (like filters, joins, groupings) into a single SQL statement and pushes it to the source database.
*   **Importance**: It optimizes performance. Instead of downloading millions of raw rows and processing them in your computer's RAM, the database server runs the SQL query and sends only the filtered, aggregated results to Power BI.
*   **What Breaks It**:
    1.  Writing a custom SQL statement in the connection box when loading data.
    2.  Using transformations not supported by the source database's SQL dialect (e.g. `Index Column` or certain string splits).
    3.  Merging queries from two different data sources (e.g. SQL Server and an Excel sheet).
*   **Optimization Strategy**: Place all folding-compatible steps (like filters, column selections, renaming) at the very top of your Applied Steps list, before any non-folding steps.

---

## 2. DAX & Modeling

### Question 2: What is Context Transition in DAX? How do you trigger it, and why is it dangerous in Calculated Columns?
*   **Context Transition**: The process of converting a Row Context (row-by-row loop) into a Filter Context.
*   **How to Trigger It**: Wrap a calculation inside the `CALCULATE` function.
*   **Calculated Column Danger**: Inside a calculated column, a Row Context is active. If you write a measure reference (which has an implicit `CALCULATE` wrapper) or write `CALCULATE` directly, Context Transition occurs.
    *   The engine filters the *entire table* to only match the values of the current row.
    *   If your table has duplicate key values, this filters the table to those duplicates, which can result in incorrect summaries.
    *   It is CPU and RAM intensive, which can significantly slow down your data refresh.

---

## 3. Calculated Columns vs. Measures

### Question 3: When would you create a Calculated Column instead of a Measure?
*   **Use Calculated Columns when**:
    1.  You need to use the resulting values as a **slicer** or filter on the report page.
    2.  You need to use the values as **row or column headers** in a matrix or table visual.
    3.  You need to categorize data into buckets (e.g. grouping ages into "Under 18", "18-35", "35+").
*   **Use Measures when**:
    1.  You are calculating quantitative metrics (e.g., total sales, average profit, year-on-year growth).
    2.  The calculation must react dynamically to slicers and filters clicked by the user on the dashboard.
*   **Interview Verdict**: *"I write measures by default to minimize memory footprint and file size. I only create calculated columns when I need to slice or group data."*

---

## 4. Security & Administration

### Question 4: How do you set up dynamic Row-Level Security (RLS) so that sales managers only see data for their own region?
*   **Answer Checklist**:
    1.  **Model Prep**: Ensure the data model contains an employee table (e.g., `Employees`) containing emails and regions, linked to the main `Sales` table by region.
    2.  **Define Role**: In Power BI Desktop, go to **Manage Roles** and create a new role (e.g., `RegionalManager`).
    3.  **DAX Filter**: Apply a DAX filter on the `Employees` table:
        ```dax
        [Email] = USERPRINCIPALNAME()
        ```
    4.  **Publish**: Publish the report to the Power BI Service.
    5.  **Assign Users**: In the Service, go to the Semantic Model security settings, and add the target Active Directory group or individual emails to the `RegionalManager` role.
    6.  *Result*: When a manager logs in, `USERPRINCIPALNAME()` returns their login email, filtering the employee table, which in turn filters the sales transactions.

---

## 5. DAX Code Challenge

### Scenario:
Write a DAX measure to calculate the percentage of total company sales contributed by the current filtered category, ensuring that selecting subcategories in a slicer does not break the calculation.
*   **Solution Walkthrough**:
    1.  We need the sales for the current category: `VAR CurrentSales = SUM(Sales[Amount])`.
    2.  We need the grand total sales across all categories, overriding the active category filter:
        `VAR TotalSales = CALCULATE(SUM(Sales[Amount]), ALL(Products[Category]))` or `ALL(Products)`.
    3.  Divide the two values safely using `DIVIDE()`.
    4.  **Correct Measure**:
        ```dax
        Category Share % = 
        VAR CurrentSales = SUM(Sales[Amount])
        VAR TotalSales = CALCULATE(SUM(Sales[Amount]), ALL(Products[Category]))
        RETURN
        DIVIDE(CurrentSales, TotalSales, 0)
        ```
