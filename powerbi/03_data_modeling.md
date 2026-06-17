# Chapter 3: Data Modeling in Power BI

Data modeling is the process of defining relationships between tables. A clean data model organizes your data in a Star Schema, optimizing query performance and making it intuitive to write DAX measures. This chapter covers relationship cardinality, cross-filtering, calendar tables, and active vs. inactive relationships.

---

## 1. Schema Design: Star vs. Snowflake
Power BI is optimized around the **Star Schema**.

```
            +-------------------+
            |   Customers Dim   |
            |   (Lookup Table)  |
            +---------+---------+
                      | 1
                      |
                      | *
+---------------+     |     +--------------------+
| Products Dim  +-----+-----+     Sales Fact     |
| (Lookup Table)| *   *     | (Transaction Table)|
+---------------+           +----------+---------+
                                       | *
                                       |
                                       | 1
                            +---------+----------+
                            |     Date Dim       |
                            |   (Lookup Table)   |
                            +--------------------+
```

*   **Fact Table**: Stores quantitative transactions (e.g. Sales, Inventories). Fact tables have many rows, few columns, and link to dimension tables.
*   **Dimension Table**: Stores descriptive attributes (e.g. Customers, Products). Dimension tables have fewer rows and many columns.
*   **Snowflake Schema**: A variations of the Star Schema where dimension tables are normalized (e.g. `Product` links to `Sub-Category`, which links to `Category`).
    *   *Analyst Rule*: Avoid Snowflake schemas by default. They force the database engine to traverse multiple joins, which slows down visual rendering and complicates DAX calculations. Flatten hierarchies in Power Query before loading them.

---

## 2. Cardinality & Cross-Filter Direction

### A. Cardinality
*   **One-to-Many (`1:*`)**: The standard relationship. One row in the Dimension table matches multiple rows in the Fact table (e.g., one customer makes multiple purchases).
*   **Many-to-Many (`*:*`)**: Occurs when both columns contain duplicate values.
    *   *Warning*: Many-to-Many relationships should be avoided. They introduce ambiguity and can lead to incorrect calculations. Resolve this by creating a **Bridge Table** containing unique values of the key to connect both tables via two `1:*` relationships.

### B. Cross Filter Direction
This controls the direction that filters flow between tables.
*   **Single (Default)**: Filters flow from the 1-side of the relationship to the *-side. (e.g., Selecting a customer name filters the Sales table to show only their purchases).
*   **Both (Bidirectional)**: Filters flow in both directions.
    *   *Warning*: Never set cross-filtering to *Both* globally. It can cause circular dependencies, performance degradation, and unexpected double-counting. Keep it set to *Single* and activate bidirectional filtering on the fly inside specific DAX measures using `CROSSFILTER`.

---

## 3. The Date Dimension (Calendar Table)
You must **never** use Power BI’s default auto-date hierarchies (which generate hidden, bloated tables behind the scenes for every date column). Always create a dedicated Date table.

### Date Table Requirements:
1.  Must contain continuous dates without gaps (covering all days of all years in your dataset).
2.  Must be marked as a Date table (Right-click table > **Mark as date table**).
3.  Must contain columns for Year, Month, Month Name, Quarter, and Day.

---

## 4. Active vs. Inactive Relationships
A Fact table can have multiple date columns (e.g., `OrderDate`, `ShipDate`, `DeliveryDate` in a Sales table). However, Power BI only allows **one active relationship** between any two tables.

*   **Active Relationship**: Represented by a solid line. By default, date slicers will filter your fact table based on this connection.
*   **Inactive Relationship**: Represented by a dashed line.
*   **USERELATIONSHIP**: You can activate the inactive connection inside a DAX measure on the fly using `USERELATIONSHIP`:

```dax
Shipped Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    USERELATIONSHIP(Sales[ShipDate], 'Calendar'[Date])
)
```

---

## 5. Practice Question Bank

### Beginner Question
You have a `Sales` table and a `Product` table. You drag a relationship between `ProductID` in both tables. Power BI prompts you that this relationship has a many-to-many (`*:*`) cardinality. What does this tell you about your `Product` table, and how do you resolve it?
*   **Solution Walkthrough**:
    1.  A dimension lookup table (like `Product`) must have unique primary keys.
    2.  If Power BI flags a many-to-many relationship, it means your `Product` table contains duplicate `ProductID` values.
    3.  To resolve this, go to Power Query, select the `ProductID` column in the `Product` query, right-click, and select **Remove Duplicates**.
    4.  Apply changes. You can now establish a clean `1:*` relationship.

### Intermediate Question
You want to display a table visual showing customer names from the `Customers` table, and the count of products they bought from the `Sales` table. Currently, selecting a customer in a slicer filters the table correctly, but selecting a product in a product slicer does not filter the customer list. How do you resolve this?
*   **Solution Walkthrough**:
    1.  The relationship between `Customers` (1-side) and `Sales` (*-side) has a cross-filter direction set to **Single**. Filters flow from `Customers` to `Sales`.
    2.  Selecting a product filters the `Sales` table. But because the filter cannot flow backwards (from the *-side to the 1-side), the `Customers` table is unaffected.
    3.  To solve this, you can change the cross-filter direction of the relationship to **Both** (not recommended globally), or write a measure that uses `CROSSFILTER`:
        ```dax
        Filtered Customer Count = 
        CALCULATE(
            DISTINCTCOUNT(Sales[CustomerID]),
            CROSSFILTER(Customers[CustomerID], Sales[CustomerID], Both)
        )
        ```
    4.  This forces bidirectional filtering only for this specific calculation, preserving model performance.

### Advanced Question
You have a `Sales` table with columns `OrderDate` and `DeliveryDate`. Both columns are linked to the `Calendar` table (`OrderDate` is active, `DeliveryDate` is inactive).
Write three DAX measures:
1.  `Ordered Revenue`: Revenue based on the order date.
2.  `Delivered Revenue`: Revenue based on the delivery date.
3.  `Revenue in Transit`: Revenue that has been ordered but not yet delivered.
*   **Solution Walkthrough**:
    1.  `Ordered Revenue` uses the active relationship:
        `Ordered Revenue = SUM(Sales[Amount])`
    2.  `Delivered Revenue` must activate the inactive relationship on `DeliveryDate` using `USERELATIONSHIP`:
        `Delivered Revenue = CALCULATE(SUM(Sales[Amount]), USERELATIONSHIP(Sales[DeliveryDate], 'Calendar'[Date]))`
    3.  `Revenue in Transit` is the difference. However, we must ensure we compute this difference over a consistent date context:
        `Revenue in Transit = [Ordered Revenue] - [Delivered Revenue]`
        *(This calculates the value of orders placed but not yet delivered in the selected time window)*.
        *Alternatively (using filter states)*:
        `Revenue in Transit = CALCULATE(SUM(Sales[Amount]), ISBLANK(Sales[DeliveryDate]))`
        *(This sums orders where the delivery date is blank).*
        Both are valid depending on the exact business definition.
