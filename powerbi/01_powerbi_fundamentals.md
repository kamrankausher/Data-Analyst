# Chapter 1: Power BI Fundamentals

Power BI is Microsoft's premier business intelligence and data visualization platform. It allows data analysts to aggregate, model, and visualize data into interactive dashboards. This chapter covers the product ecosystem, storage connectivity modes, licensing profiles, and the VertiPaq database engine.

---

## 1. The Power BI Suite Ecosystem
Power BI is a distributed cloud and desktop ecosystem:

*   **Power BI Desktop**: A free Windows application where 100% of report development occurs (data ingestion, modeling, DAX scripting, visual design).
*   **Power BI Service (SaaS)**: A cloud platform (`app.powerbi.com`) where completed reports are published, shared, secured (via RLS), and scheduled to refresh.
*   **Power BI Gateways**: Secure on-premises software that links your cloud-hosted dashboards to local SQL servers or files, allowing scheduled data refreshes.
*   **Power BI Mobile**: App used by executives to monitor dashboards on smartphones.

---

## 2. Under the Hood: The VertiPaq Columnar Engine
To understand Power BI’s speed, we must explore its internal storage engine, **VertiPaq**. Traditional databases (like MySQL) store data row-by-row. VertiPaq is a **column-oriented, in-memory database**.

```
   Row-Store Database (MySQL):
   Row 1: [ 101, Alice, Tech, USA ]
   Row 2: [ 102, Bob,   Tech, Canada ]
   
   Column-Store Database (VertiPaq):
   Column ID:   [ 101, 102 ]
   Column Name: [ Alice, Bob ]
   Column Dept: [ Tech, Tech ]  ===> Compresses easily because of duplicates!
```

### How VertiPaq Compresses Data:
1.  **Value Encoding**: Math transformations that scale numbers down to smaller ranges.
2.  **Dictionary Encoding**: Replaces long strings with small integer index keys. For example, if "United States" is written 1,000,000 times, VertiPaq stores the string once in a dictionary index under ID `1`, and then writes `1` in the column matrix.
3.  **Run-Length Encoding (RLE)**: Compresses repeating values. If a column contains 10,000 repeating values of `1`, instead of writing `1` ten thousand times, RLE writes: `(1, 10000 times)`.

---

## 3. Storage & Connection Modes
When you connect to data, you must choose a storage mode:

### A. Import Mode (Default)
Data is extracted from the source and loaded directly into your computer's RAM.
*   *Pros*: Maximum performance. Unlocks all DAX and Power Query operations.
*   *Cons*: File size limit of 1GB (for Pro licensing). Data is static until refreshed.

### B. DirectQuery Mode
No data is loaded into Power BI. Instead, whenever a user clicks a visual, Power BI translates the click into a live SQL query and runs it directly on the source database.
*   *Pros*: Can query tables containing billions of rows (exceeding RAM limits). Real-time data updates.
*   *Cons*: Visual rendering is slow (limited by database speed). Restricts advanced DAX functions.

### C. Live Connection Mode
Connects directly to an external, pre-built SQL Server Analysis Services (SSAS) cube or Power BI Semantic Model.
*   *Pros*: Promotes a single version of truth across the enterprise. Offloads computations to a central server.
*   *Cons*: Disables Power Query ETL. You cannot modify the data structure.

---

## 4. Licensing Comparison
*   **Power BI Free**: Design reports locally and publish to a personal workspace. **Cannot share reports** with other users.
*   **Power BI Pro** ($10/user/month): Required to collaborate and publish/share reports in shared workspaces. File sizes capped at 1GB.
*   **Power BI Premium Capacity**: Server capacity leased by a company. Allows users with **Free licenses** to view dashboards, which is cost-effective for large organizations with thousands of report consumers.

---

## 5. Practice Question Bank

### Beginner Question
You have built a sales dashboard in Power BI Desktop connecting to local Excel files. You publish it to your colleague's workspace, but they receive an error saying they cannot view the data. What license must they have to view this shared dashboard?
*   **Solution Walkthrough**:
    1.  Power BI enforces workspace sharing rules.
    2.  To view a report shared in a collaborative workspace (non-premium), both the publisher and the viewer must have a paid license.
    3.  Therefore, your colleague must have a **Power BI Pro** (or Premium Per User) license to view the dashboard.

### Intermediate Question
A company has a database containing 500 million rows of historical sales data. They want to construct a dashboard. The data changes continuously, and executives require real-time updates. Visual performance is important, but data timeliness is the absolute priority. Which storage mode should you select?
*   **Solution Walkthrough**:
    1.  First check the data size: 500 million rows. This is too large for standard Import Mode RAM limitations.
    2.  Check the business requirement: Real-time updates. Import Mode requires scheduled refreshes, so it cannot be real-time.
    3.  Therefore, **DirectQuery** is the correct choice. It queries the live SQL database on every visual interaction, ensuring real-time data access.

### Advanced Question
Explain how VertiPaq's **Run-Length Encoding (RLE)** would compress a column containing sorted customer locations: `['US', 'US', 'US', 'US', 'CA', 'CA', 'MX']`. Contrast this with unsorted data.
*   **Solution Walkthrough**:
    1.  RLE works by compressing consecutive duplicate values.
    2.  For the sorted array `['US', 'US', 'US', 'US', 'CA', 'CA', 'MX']`, RLE compresses it to three instructions:
        *   Value: `US`, Count: 4
        *   Value: `CA`, Count: 2
        *   Value: `MX`, Count: 1
    3.  This reduces the storage footprint from 7 elements to 3 value-count pairs.
    4.  If the array was unsorted: `['US', 'CA', 'US', 'MX', 'US', 'CA', 'US']`, RLE cannot compress it because there are no consecutive duplicates. The footprint would remain 7 elements.
    5.  *Analyst Takeaway*: Sorting your columns or designing tables with clean hierarchies (low cardinality) before loading them into Power BI significantly improves compression and visual speeds.
