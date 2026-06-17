# Chapter 5: Visualization, Publishing & Security

Designing a data model and writing DAX is only half the battle. The final step is presenting your data. This chapter covers dashboard layout composition, drill-throughs, Power BI Service deployments, gateway connections, Row-Level Security (RLS), and performance profiling.

---

## 1. Visual Composition & Usability

### A. Layout Structure
*   **The Grid**: Align visuals using standard grid margins. Place primary KPI cards (Revenue, margin) at the top-left, filters on the top or left, and detailed charts in the center and bottom.
*   **The F-Shape Pattern**: Design reports to match how humans read screens (scanning horizontally across the top, then down the left).

```
   +---------------------------------------------------+
   | [Logo]   KPI 1 (Rev)   KPI 2 (Qty)   KPI 3 (Goal) |  <-- High level KPI Cards
   +---------------------------------------------------+
   | [Filters] |                                       |
   |           |    Main Trend Line Chart (Sales)      |  <-- Core Visual
   | Region    |                                       |
   | Category  +---------------------------------------+
   |           |    Category Breakdown Bar Chart       |  <-- Supporting Visual
   +-----------+---------------------------------------+
```

---

## 2. Interactive Navigation Features

### A. Drill-Down vs. Drill-Through
*   **Drill-Down**: Navigates down a hierarchical column group within the same visual (e.g. clicking a bar for the year `2026` to see quarterly details).
*   **Drill-Through**: Redirects users to a completely separate report page focused on the clicked category (e.g., right-clicking a customer on the main dashboard to jump to a customer profile page).
    *   *Setup*: Create a destination page, set Page Type to Standard, and drag the target key field (e.g., `CustomerID`) into the **Drill-Through filters** panel.

### B. Custom Page Tooltips
Hovering over a visual element can display a small, custom-designed report page (e.g. hovering over a region on a map to see a mini bar chart of sales reps in that region).
1.  Create a new page. Under Page Information, set **Allow use as tooltip** to On.
2.  Set the Canvas Size to **Tooltip** (small size).
3.  Design your mini chart on this canvas.
4.  On your main page, select your visual > Format > Tooltips > set Page to your custom tooltip page.

---

## 3. Power BI Service & Gateway Refresh
To share reports, click **Publish** in Power BI Desktop to send them to the cloud-based Power BI Service.

### On-Premises Data Gateways
When a dashboard imports data from a local SQL Server or file share, the cloud-based Power BI Service cannot access it.
*   **The Gateway Solution**: Install the On-premises Data Gateway on a local server that is always running. The gateway opens a secure, outbound connection to the Power BI Service.
*   **Configuring Refresh**:
    1.  Open the Semantic Model settings in the Power BI Service.
    2.  Expand **Gateway connection** and ensure your gateway is online.
    3.  Under **Data source credentials**, enter the credentials for your local database.
    4.  Turn on **Scheduled Refresh** (up to 8 times daily for Pro, 48 times for Premium).

---

## 4. Row-Level Security (RLS)
RLS restricts data access for given users. Filters restrict data at the database level, ensuring users only see what they are authorized to see.

### A. Static RLS
Use for fixed role boundaries:
1.  In Power BI Desktop, go to **Modeling** > **Manage Roles**.
2.  Create a role (e.g., "US Managers").
3.  Write a DAX filter expression on your dimension table: `Customers[Country] = "United States"`.
4.  Publish to the Service. Go to the Semantic Model Security settings and add the emails of the US managers to this role.

### B. Dynamic RLS
Use for dynamic data access based on who is logged in.
1.  Add a lookup table in your model that maps employee emails to their allowed regions.
2.  In Manage Roles, create a role (e.g., "User Filter").
3.  Write this DAX filter on your employee map table:
    ```dax
    [EmployeeEmail] = USERPRINCIPALNAME()
    ```
    *(The `USERPRINCIPALNAME()` function returns the login email of the active viewer, dynamically filtering the table to match their record).*

---

## 5. Performance Diagnostics: Performance Analyzer
If a dashboard page loads slowly, use the **Performance Analyzer**:
1.  Go to **View** > **Performance Analyzer**.
2.  Click **Start Recording** and click **Refresh Visuals**.
3.  Power BI displays the load times (in milliseconds) for every visual, split into:
    *   *DAX Query*: Time spent running the DAX code in the engine.
    *   *Visual Display*: Time spent rendering the graphics on screen.
    *   *Other*: Time spent waiting on background tasks or network queries.
*   *Optimization Rule*: If DAX Query time is high, optimize your DAX formulas (e.g., replace calculated columns with measures, or avoid nested iterators). If Visual Display time is high, reduce the number of visuals on the page.

---

## 6. Practice Question Bank

### Beginner Question
You have created an interactive report. You want to add a button that resets all slicers on the page to their default, unfiltered state. How do you implement this?
*   **Solution Walkthrough**:
    1.  First, configure the page with all slicers in their default, unfiltered state.
    2.  Go to **View** > **Bookmarks** to open the bookmarks panel.
    3.  Click **Add** to create a bookmark, and rename it to `ResetFilters`.
    4.  Go to **Insert** > **Buttons** > select **Reset**.
    5.  Select the button, go to the format panel > Turn on **Action**.
    6.  Set type to **Bookmark**, and select `ResetFilters` as the target.
    7.  Now, when users click the button, the page returns to the unfiltered state saved in the bookmark.

### Intermediate Question
Explain the differences between **Drill-Down** and **Drill-Through**, providing a business scenario for each.
*   **Solution Walkthrough**:
    *   **Drill-Down**: Navigates levels of a hierarchy within the *same visual*.
        *   *Scenario*: In a bar chart showing Sales by Year, a user clicks the "2026" bar to see sales by Quarter for that year.
    *   **Drill-Through**: Navigates from one page to a *completely different page* containing detailed context about the selected item.
        *   *Scenario*: In a table listing customers and their total sales, a user right-clicks a customer row, selects Drill-Through, and is redirected to a page displaying that customer's address, contact details, and transaction history.

### Advanced Question
You need to set up dynamic Row-Level Security (RLS) for a sales team.
- The `Sales` table contains transaction records.
- The `AccessMap` table contains columns `Email` and `Region`.
- Both tables are connected by `Region`.
Explain how you would write the DAX filter expression to restrict data access, and how you would test the setup.
*   **Solution Walkthrough**:
    1.  The relationship between `AccessMap` and `Sales` must be configured with a cross-filter direction set to **Single** (from `AccessMap` to `Sales`) or **Both**, depending on model hierarchy.
    2.  In Power BI Desktop, go to **Manage Roles**.
    3.  Create a role named `SalespersonRole`.
    4.  Select the `AccessMap` table and write the DAX filter:
        ```dax
        [Email] = USERPRINCIPALNAME()
        ```
    5.  To test the setup before publishing:
        *   Go to the **Modeling** tab and click **View as**.
        *   Check the box for `SalespersonRole`.
        *   Check the box for **Other User** and enter a sales rep's email (e.g. `john@company.com`).
        *   Click OK. The visuals should display data only for the region associated with John's email in the `AccessMap` table.
