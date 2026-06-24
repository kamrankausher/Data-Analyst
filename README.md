# Complete Data Analyst Curriculum (Zero to Hero)

Welcome to the ultimate Data Analyst self-study guide! This repository contains comprehensive, textbook-quality notes, diagrams, and practice question banks designed to take you from absolute zero to a super-advanced, interview-ready level. 

No video tutorials required — every topic is explained in-depth with logical analogies for beginners, low-level mechanics for advanced learners, and practical coding exercises with detailed solutions.

---

## 📂 Repository Contents

### 1. 📊 Excel for Data Analysis (`/excel`)
*   **01_excel_fundamentals.md**: Excel interface grid architecture, cell reference locking models (`$A1` vs. `A$1`), and data types.
*   **02_formulas_and_functions.md**: Logical evaluation (`IF`/`IFS`), lookup functions (`VLOOKUP`, `INDEX & MATCH`, `XLOOKUP`), and text/date manipulation.
*   **03_data_cleaning_and_analysis.md**: Text-to-columns delimiter parsing, deduplication, handling blanks, multi-level sorting, conditional formatting, and validation rules.
*   **04_pivot_tables_and_charts.md**: Aggregation grids, value calculations (% of total, diff from), numeric/date grouping bins, slicers, and charting.
*   **05_power_query_and_modeling.md**: ETL steps, wide-to-tall unpivoting, joins (merging) and unions (appending), Power Pivot relational data model, and DAX.
*   **interview_questions.md**: Real interview questions on referencing, lookups, and VertiPaq engine with detailed explanations.

### 2. 🗄️ SQL for Data Analysis (`/sql`)
*   **01_sql_fundamentals.md**: Relational model (PK/FK), compilation execution order (from `FROM` to `LIMIT`), operators, and three-valued logic.
*   **02_joins_and_unions.md**: Horizontal joins (INNER, LEFT, RIGHT, FULL, SELF, CROSS), vertical stacking (`UNION` vs. `UNION ALL`), and database join algorithms.
*   **03_aggregations_and_grouping.md**: Count variations, `GROUP BY` RAM mechanics, `WHERE` vs. `HAVING` filters, and date truncations.
*   **04_subqueries_and_ctes.md**: Nested subqueries (FROM, WHERE, SELECT), correlated evaluations, CTE optimization, and Recursive CTEs.
*   **05_window_functions.md**: Windows partition syntax, rank functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`), lag/lead growth, and sliding windows (`ROWS BETWEEN`).
*   **06_optimization_and_design.md**: 1NF/2NF/3NF normalization, Star vs. Snowflake dimensional schemas, B-Tree index heights, execution plans (`EXPLAIN`), and SARGable queries.
*   **interview_questions.md**: Advanced SQL questions on join types, parser execution order, index seeks, and grouping.

### 3. 📉 Power BI for Data Analysis (`/powerbi`)
*   **01_powerbi_fundamentals.md**: Desktop vs. Service ecosystem, connection modes (Import, DirectQuery, Live), and VertiPaq columnar compression.
*   **02_power_query_etl.md**: ETL workspace transformations, unpivoting categories, merging/appending, M-language let-blocks, and Query Folding.
*   **03_data_modeling.md**: Schema layouts, relationship cardinality, cross-filter pitfalls, Date dimensions, and inactive relationships (`USERELATIONSHIP`).
*   **04_dax_expressions.md**: Row context vs. Filter context, columns vs. measures, context transition via `CALCULATE`, iterators (`SUMX`), and time intelligence.
*   **05_visualization_and_publishing.md**: Design grids, tooltips, drill-throughs, Power BI Service deployments, gateways, and static/dynamic Row-Level Security.
*   **interview_questions.md**: Standard Power BI questions on connections, calculated columns vs. measures, RLS, and Query Folding.

### 4. 🐍 Python for Data Analysis (`/python`)
*   **01_numpy_arrays.md**: Python lists vs. NumPy arrays, contiguous memory allocation, vectorization (SIMD), broadcasting rules, and axes calculations.
*   **02_pandas_dataframes.md**: Series/DataFrame layouts, reading datasets, health inspection checks, `loc` vs. `iloc` indexing, and boolean queries.
*   **03_pandas_data_cleaning.md**: Missing value treatments, float type coercions, datetime extractions, custom lambdas, and merges/concatenations.
*   **04_pandas_aggregation.md**: Split-apply-combine, custom aggregation matrices, pivot tables, crosstabs, shifts (lag/lead), and rolling sliding windows.
*   **05_data_visualization.md**: Matplotlib Figure/Axes objects, customized plots, Seaborn statistical plots, and correlation matrix heatmaps.
*   **06_eda_case_study.md**: Step-by-step Exploratory Data Analysis (EDA) on a mock transactions dataset, from ingestion to reporting.
*   **interview_questions.md**: Real python questions on NumPy speed, loc vs iloc, GroupBy aggregations, and memory optimization.

### 5. 🧮 Statistics for Data Analysis (`/statistics`)
*   **01_descriptive_statistics.md**: Central tendency (mean/median/mode), dispersion (sample variance with Bessel's correction, standard dev, IQR), skewness, and Z-scores.
*   **02_probability_and_distributions.md**: Conditional probability, Bayes' Theorem math, discrete distributions (Binomial, Poisson), Normal distributions, and CLT.
*   **03_hypothesis_testing.md**: H0/Ha hypotheses, Type I/II errors, T-tests, ANOVA, Chi-Square, and A/B testing parameters (Sample sizing, MDE).
*   **04_regression_and_correlation.md**: Pearson/Spearman correlation coefficients, Simple/Multiple Linear Regression OLS, R-squared model evaluation, and multicollinearity (VIF).
*   **interview_questions.md**: Real statistics questions on mean vs median, hypothesis errors, p-values, regression evaluation, and multicollinearity.

---

