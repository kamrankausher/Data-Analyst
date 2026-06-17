# Excel Data Analyst Interview Prep Guide

This guide contains the most frequently asked Excel interview questions in Data Analyst interviews at top tech companies, finance firms, and consultancies. Each question includes a detailed, professional answer.

---

## 1. Lookups and Array Mechanics

### Question 1: What is the difference between VLOOKUP, INDEX & MATCH, and XLOOKUP? When and why would you choose one over another?
*   **VLOOKUP**: Searches vertically down the first column of a table and returns a value from a specified column index to the right.
    *   *Limitations*: It cannot look to the left. It breaks if you insert/delete columns because of hardcoded column indexes. It is slow on large tables because it evaluates the entire table array in memory.
*   **INDEX & MATCH**: A combination where `MATCH` finds the row position of the key, and `INDEX` retrieves the value from the target column at that position.
    *   *Advantages*: Can look left or right. Does not break when columns are inserted/deleted. It is faster on large tables because it only references two columns in memory, not the whole grid.
*   **XLOOKUP**: The modern replacement that defaults to exact match, can look in any direction, allows horizontal and vertical searches, and handles errors and binary searches natively.
    *   *Advantages*: Simple syntax, safer default behavior.
*   **Interview Verdict**: In an interview, state: *"I use XLOOKUP by default for its simplicity and safety. If I am working on legacy models that need compatibility with older versions of Excel, I use INDEX & MATCH for its flexibility and speed over VLOOKUP."*

---

## 2. Cell Referencing & Grid Architecture

### Question 2: Explain relative, absolute, and mixed references. In what scenario would you use a mixed reference like `A$1` vs. `$A1`?
*   **Relative (`A1`)**: Updates row and column coordinates when the formula is copied or dragged.
*   **Absolute (`$A$1`)**: Locks both the column and the row. It remains fixed wherever the formula is copied.
*   **Mixed Reference**: Locks only one dimension.
    *   `A$1` (Row Locked): When you copy the formula down, the row stays locked at `1`. When you copy it right, the column changes (`B$1`, `C$1`). Use this when you have **column headers** containing parameters (e.g. tax rates by year listed horizontally).
    *   `$A1` (Column Locked): When you copy the formula down, the row changes (`$A2`, `$A3`). When you copy it right, the column stays locked at `A`. Use this when you have **row headers** containing parameters (e.g. product names listed vertically).
*   **Case Scenario**: Creating a 2D matrix (like a multiplication table or a cohort grid) requires combining both: `=$A2*B$1` so that a single formula can be copied across the entire grid without error.

---

## 3. Data Cleaning and Preparation

### Question 3: You imported customer emails, but lookups are failing. You notice some cells have extra spaces, mixed casing, or are stored in different tabs. Walk through your step-by-step debugging checklist.
*   **Answer Checklist**:
    1.  **Standardize Casing**: Wrap the lookup columns in `LOWER()` or `UPPER()` to ensure capitalization matches (e.g., `john@company.com` matches `JOHN@COMPANY.COM`).
    2.  **Strip Whitespaces**: Wrap columns in `TRIM()` to remove leading, trailing, or double spaces.
    3.  **Non-Printable Characters**: Use `CLEAN()` to remove non-printable characters often imported from database logs.
    4.  **Data Validation check**: Check if any email is stored as a hyperlink or rich text. Convert to plain text.
    5.  **Lookups rewrite**: Implement a safe lookup:
        `=XLOOKUP(LOWER(TRIM(A2)), LOWER(TRIM(LookupRange)), ReturnRange, "Not Found")`
        *(Note: If using XLOOKUP, you can pass arrays through functions directly).*

---

## 4. Modeling & Power Pivot

### Question 4: How does Excel's internal Data Model (Power Pivot) bypass the standard worksheet limit, and how does it compress data?
*   **Bypassing Limits**: Standard Excel sheets are limited to **1,048,576 rows**. Power Pivot loads data into an in-memory database engine called **VertiPaq**. VertiPaq runs in RAM, bypasses the grid sheet limits, and can store **100+ million rows** in a single workbook.
*   **Compression Mechanism**:
    1.  **Column-Oriented Storage**: Instead of storing data row-by-row, it stores column-by-column. Since columns contain many repeating values (e.g. region names), they compress heavily.
    2.  **Dictionary Encoding**: Replaces long strings with small integer index keys.
    3.  **Run-Length Encoding (RLE)**: Compresses consecutive repeating values by recording the value and its count.

---

## 5. Practice Interview Test

### Scenario:
You are analyzing monthly sales. Cell `B1` contains the current year target: `$50,000`. Column A contains Month, Column B contains Sales, and Column C contains Category.
Write a single formula to output `"Target Met - High Value"` if the Month is in Q4 (Oct, Nov, Dec), sales are greater than or equal to the target in `B1`, and the category is `"Technology"`. Otherwise, return `"Standard"`.
*   **Solution Walkthrough**:
    1.  We need to check multiple conditions using `AND()`.
    2.  Condition 1: Q4 Month. This means Month must be "October", "November", or "December". We can check this using `OR(A2="October", A2="November", A2="December")`.
    3.  Condition 2: Sales must be $\geq$ target in `B1`. The target cell is a single baseline cell, so it must be locked absolutely: `B2>=$B$1`.
    4.  Condition 3: Category is Technology: `C2="Technology"`.
    5.  Combine in an `IF` statement.
    6.  **Correct Formula**:
        ```excel
        =IF(AND(OR(A2="October", A2="November", A2="December"), B2>=$B$1, C2="Technology"), "Target Met - High Value", "Standard")
        ```
