# Chapter 3: Data Cleaning & Analysis in Excel

Data cleaning is the most critical phase of any analysis project. Real-world business data is messy — it has duplicates, missing cells, bad casing, and corrupt formats. This chapter covers the tools and mathematical logic used to clean datasets inside Excel.

---

## 1. Importing & Formatting Text Data

### A. The Text-to-Columns Delimiter Split
When importing raw CSV, log, or text files, data often opens compressed inside Column A.
*   **Delimiter**: A character (like a comma, semicolon, tab, or pipe `|`) that separates data fields.
*   **The Wizard**: Select Column A, go to **Data** > **Text to Columns**, choose *Delimited*, and check your delimiter.
*   *Critical Analyst Rule*: In the final step of the wizard, inspect your columns. If a column contains IDs with leading zeros (e.g. `007421`), change the column data format from *General* to **Text**. If you leave it as General, Excel will strip the leading zeros, converting the ID to `7421`, which corrupts the lookup key.

### B. Standardizing Text Cases
*   `UPPER("apple")` -> `"APPLE"`
*   `LOWER("APPLE")` -> `"apple"`
*   `PROPER("john smith")` -> `"John Smith"`

---

## 2. Handling Missing and Duplicate Values

### A. Removing Duplicates
Select your data table, navigate to **Data** > **Remove Duplicates**.
*   **Single-Column Deduplication**: Removes rows based on duplicates in a single unique identifier (e.g., keeping only one row per `CustomerID`).
*   **Multi-Column Deduplication**: Removes rows only if *every* column value matches.
*   *Warning*: Deduplication is destructive. Always copy your raw data tab before performing this action.

### B. Finding and Filling Blank Cells
Blank cells can skew averages and break calculations.
1.  **Locate Blanks**: Select your dataset. Press `Ctrl` + `G` (or `F5`), click **Special...**, select **Blanks**, and click **OK**. Excel will highlight only the empty cells in your selected range.
2.  **Mass-Fill Action**:
    *   *Direct Value*: Immediately type `0` or `"Unknown"` and press `Ctrl` + `Enter`. This writes the value to all selected blank cells simultaneously.
    *   *Formula Copy*: Type `=` and press the `Up Arrow` key. Press `Ctrl` + `Enter`. Excel will copy the value from the cell directly above for every blank cell.

---

## 3. Advanced Sorting & Filtering

### A. Multi-Level Sort Hierarchy
Sorting on a single column is simple, but analysts often need to sort hierarchically.
*   *Scenario*: Sort transaction data first by **Region** (A to Z), then within each region by **Sales Channel** (A to Z), and finally within those groups by **Profit** (Largest to Smallest).
*   *Path*: Go to **Data** > **Sort** to add sorting levels.

### B. Wildcard Filters
Activate filters with `Ctrl` + `Shift` + `L`.
*   `*` (Asterisk): Matches any sequence of characters (e.g., filtering `*inc*` returns "Apple Inc.", "Incorporated", "Incentive").
*   `?` (Question Mark): Matches exactly one character (e.g., filtering `202?` returns "2020", "2021", "2022", but not "202").

---

## 4. Conditional Formatting using Formulas
Conditional formatting colors cells based on conditions. While basic highlighting is simple, data analysts must write formulas to format **entire rows** based on a single column's value.

### Step-by-Step Row Highlighting:
Imagine you have an orders table `A2:D100`. Column D contains the status: `"Cancelled"`, `"Pending"`, or `"Shipped"`. You want to highlight the **entire row** in light red if the status is `"Cancelled"`.

1.  Select the entire range `A2:D100` (do **not** include the header row 1).
2.  Go to **Home** > **Conditional Formatting** > **New Rule...**.
3.  Select **Use a formula to determine which cells to format**.
4.  Write the formula: `=$D2="Cancelled"`
    *   *Why `$D2`?* The dollar sign (`$`) locks Column D. When Excel checks cell `A2`, it evaluates the status in `D2`. When Excel checks cell `B2`, the column remains locked, so it also evaluates `D2`. This ensures the entire row formats consistently.
    *   *Why no `$` on `2`?* The row must remain relative so that when Excel evaluates row 3, it checks `=$D3="Cancelled"`.
5.  Set the formatting fill color and click OK.

---

## 5. Data Validation (Data Entry Integrity)
Prevent bad data from entering your sheets.
*   *Path*: **Data** > **Data Validation**.
*   **Drop-Down Lists**: Select cell range, set Allow to *List*, and type list items in the source field: `Low, Medium, High` (or refer to a worksheet range: `=$F$2:$F$4`).
*   **Custom Validation Formulas**: You can use logical formulas to restrict inputs. For example, to prevent users from entering a shipping date that is earlier than the order date:
    *   If order date is in `A2` and shipping date is in `B2`.
    *   Set Validation on `B2` to *Custom* and use formula: `=B2>=A2`.

---

## 6. Practice Question Bank

### Beginner Question
You have a table of product names in Column A. Some names contain leading or trailing spaces (e.g. `"  Laptop "`).
Write a formula to clean the names, format them in UPPERCASE, and explain what happens if you don't clean these spaces before doing a lookup.
*   **Solution Walkthrough**:
    1.  We need to remove stray spaces: `TRIM(A2)`.
    2.  We must cast to uppercase: `UPPER(TRIM(A2))`.
    3.  If we do not clean the spaces, lookup functions like `VLOOKUP` or `XLOOKUP` will fail to find matches (e.g. `"Laptop"` does not match `"  Laptop "`).
    4.  **Correct Formula**: `=UPPER(TRIM(A2))`

### Intermediate Question
You have an inventory table spanning `A2:E200`. Column C has the quantity in stock. You want to highlight the **entire row** in light orange if the quantity in stock (`Column C`) falls below 15. Write the exact conditional formatting formula and specify what range you should select before applying the rule.
*   **Solution Walkthrough**:
    1.  Select the data range: `A2:E200`.
    2.  Write the formula locking column C but keeping the row relative.
    3.  Formula starts with `=`. Point to the first cell in Column C: `=$C2`.
    4.  Add the comparison: `<15`.
    5.  **Correct Formula**: `=$C2<15`

### Advanced Question
You are designing an expense log sheet. Column A is the `Department` and Column B is the `Expense Amount`.
You want to set up data validation on Column B to ensure that:
1.  The expense amount is a positive number.
2.  If the department in Column A is "Marketing", the expense amount in Column B cannot exceed $5,000.
Write the custom validation formula for cell `B2`.
*   **Solution Walkthrough**:
    1.  Condition 1: Amount must be positive: `B2>0`.
    2.  Condition 2: If `A2` is `"Marketing"`, then `B2` must be `<=5000`. This can be written as `IF(A2="Marketing", B2<=5000, TRUE)`.
    3.  Combine both conditions using an `AND` statement.
    4.  **Correct Formula**: `=AND(B2>0, IF(A2="Marketing", B2<=5000, TRUE))`
        *(Or equivalently: `=AND(B2>0, OR(A2<>"Marketing", B2<=5000))`)*.
