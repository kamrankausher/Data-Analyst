# Chapter 2: Formulas & Functions for Data Analysis

Writing formulas is where Excel transforms from a static data entry sheet into an analytical engine. This chapter details logical criteria, advanced lookups, string parsing, and date math. We will explore not just *how* to write functions, but how Excel processes them under the hood.

---

## 1. Logical Functions & Boolean Evaluation
Logical functions evaluate whether conditions are true or false, routing calculations accordingly.

### A. The `IF` Logic Gate
*   **Formula**: `=IF(logical_test, value_if_true, [value_if_false])`
*   *Analogy*: Think of `IF` as a fork in the road. If the sign says `Age >= 18`, adult traffic goes left (`value_if_true`), while minor traffic goes right (`value_if_false`).

### B. Logical Operators (`AND`, `OR`, `NOT`)
These are used to combine multiple criteria:
*   `AND(cond1, cond2)`: Returns `TRUE` only if **both** are true.
*   `OR(cond1, cond2)`: Returns `TRUE` if **at least one** is true.
*   *Example*: `=IF(AND(Sales >= 10000, ReturnRate < 0.05), "High Bonus", "Standard")`

### C. Nested IFs vs. `IFS`
Historically, handling multiple conditions required nesting:
`=IF(Score>=90, "A", IF(Score>=80, "B", IF(Score>=70, "C", "F")))`
This is hard to read and prone to missing parenthesis errors. The modern `IFS` function resolves this:
*   **Formula**: `=IFS(condition1, value1, condition2, value2, ...)`
*   *Rule*: Excel evaluates conditions **from left to right** and stops at the *first* condition that evaluates to `TRUE`.
*   *Example*: `=IFS(A1>=90, "A", A1>=80, "B", A1>=70, "C", TRUE, "F")` (The final `TRUE, "F"` acts as the default catch-all).

---

## 2. Lookup & Reference Functions
Lookup functions allow you to join two tables together by matching a key.

### A. `VLOOKUP` (Vertical Lookup)
*   **Formula**: `=VLOOKUP(lookup_value, table_array, col_index_num, [range_lookup])`

```
   Lookup Value: 102
   
   Table Array: F2:H4
   
      Col 1 (Key)  Col 2 (Name)  Col 3 (Price)
F2: [  101        |  Apple      |  $1.50        ]
F3: [  102  =====>|  Banana ===>|  $0.80        ]  <-- Returns $0.80 (Col Index 3)
F4: [  103        |  Orange     |  $1.20        ]
```

*   **The Crucial Exact Match Rule**: Always set the fourth argument `[range_lookup]` to `FALSE` (or `0`). If you leave it blank or set to `TRUE`, Excel assumes the first column is sorted and returns an approximate match (often the wrong record if your key isn't sorted).
*   **Major Flaws of VLOOKUP**:
    1.  **No Left Lookups**: It can only search the *first* column of the table and return columns to the right. If Employee ID is in Column B and Name is in Column A, VLOOKUP fails.
    2.  **Column Insertion Vulnerability**: If you insert a new column into the table range, your column index index number (e.g. `3`) is now wrong, and your formulas will display incorrect data.

### B. The Professional Standard: `INDEX & MATCH`
This method splits row finding and column extraction into two independent functions, resolving all flaws of VLOOKUP.
*   `MATCH(lookup_value, lookup_array, [match_type])`: Finds the *position number* of the search item in a column or row. (Set `match_type` to `0` for exact match).
*   `INDEX(array, row_num, [column_num])`: Returns the value of a cell at a specific position in a column or grid.
*   *Combined Syntax*: `=INDEX(Return_Column_Range, MATCH(Lookup_Value, Lookup_Column_Range, 0))`
*   *Benefit*: It doesn't care where the columns are (left or right), and inserting columns doesn't break the references.

### C. The Modern Solution: `XLOOKUP`
Introduced in Office 365, `XLOOKUP` replaces VLOOKUP and INDEX/MATCH.
*   **Formula**: `=XLOOKUP(lookup_value, lookup_array, return_array, [if_not_found], [match_mode])`
*   *Default*: It defaults to an exact match, can search in any direction, and handles missing values natively.
*   *Example*: `=XLOOKUP(A2, Employees[ID], Employees[Name], "Not Found")`

---

## 3. Text & Date Wrangling

### A. Text Cleaning
*   `TRIM(text)`: Deletes all leading, trailing, and double spaces. Essential because `"John "` does not match `"John"`.
*   `CONCAT(text1, text2...)`: Merges strings together (or use ampersand: `=A1 & " - " & B1`).
*   `LEFT(text, n)`, `RIGHT(text, n)`, `MID(text, start, n)`: Extract slices of text.
*   `FIND(find_text, within_text)`: Returns the position number of a character (case-sensitive).

### B. Date Calculations
*   `TODAY()`: Returns current date.
*   `NETWORKDAYS(start, end, [holidays])`: Returns the count of working days (excludes weekends and custom holiday lists).
*   `DATEDIF(start, end, unit)`: Returns difference in years (`"Y"`), months (`"M"`), or days (`"D"`).

---

## 4. Error Handling: `IFERROR`
When formulas fail, they output codes like `#N/A`, `#VALUE!`, or `#DIV/0!`. These ruin dashboards.
*   **Formula**: `=IFERROR(Value_To_Try, Value_If_Error)`
*   *Example*: `=IFERROR(Sales / Units, 0)`

---

## 5. Practice Question Bank

### Beginner Question
You have a table of employees. Column A contains First Name, and Column B contains Last Name. In Column C, write a formula to combine them into a single string formatted as `Lastname, Firstname` (e.g. `Smith, John`) in all lowercase, and clean any stray spaces.
*   **Solution Walkthrough**:
    1.  We need to merge the columns: `B2 & ", " & A2`.
    2.  We must clean stray spaces using `TRIM`.
    3.  We must convert to lowercase using `LOWER`.
    4.  **Correct Formula**: `=LOWER(TRIM(B2) & ", " & TRIM(A2))`

### Intermediate Question
You are looking up product prices. The Product ID is in cell `A2`. The price catalog is in another sheet called `Catalog`, spanning `A2:B100` where column A is Price and column B is Product ID.
1.  Explain why `VLOOKUP` cannot find the price.
2.  Write the correct formula to retrieve the price using `INDEX & MATCH`.
*   **Solution Walkthrough**:
    1.  `VLOOKUP` searches the *first* column of the table range. In `Catalog`, column A contains Price and column B contains Product ID. `VLOOKUP` would look up the Product ID in the Price column, which fails.
    2.  `INDEX & MATCH` allows us to search column B and return column A.
    3.  Lookup Range: `Catalog!$B$2:$B$100` (Product ID column).
    4.  Return Range: `Catalog!$A$2:$A$100` (Price column).
    5.  **Correct Formula**: `=INDEX(Catalog!$A$2:$A$100, MATCH(A2, Catalog!$B$2:$B$100, 0))`

### Advanced Question
You have transaction records. Column A has `Transaction Date` and Column B has `Shipping Date`.
1.  Write a formula to calculate the number of workdays between transaction and shipping.
2.  If the shipping date is missing, return `"Not Shipped"`.
3.  If any calculation error occurs, return `"Error"`.
*   **Solution Walkthrough**:
    1.  Test if shipping date is blank: `IF(B2="", "Not Shipped", ...)`
    2.  Calculate working days: `NETWORKDAYS(A2, B2)`.
    3.  Wrap everything in `IFERROR` to handle errors.
    4.  **Correct Formula**: `=IFERROR(IF(B2="", "Not Shipped", NETWORKDAYS(A2, B2)), "Error")`
