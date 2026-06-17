# Chapter 6: Exploratory Data Analysis (EDA) Case Study

Exploratory Data Analysis (EDA) is the process of examining a newly acquired dataset to understand its characteristics, uncover hidden patterns, detect anomalies, test hypotheses, and verify assumptions using summary statistics and graphical representations.

In this chapter, we walk through a complete, end-to-end EDA case study on a mock e-commerce transactions dataset.

---

## 1. The EDA Workflow Framework
A systematic approach prevents you from getting lost in the code:

1.  **Define Business Objectives**: What questions are we trying to answer? (e.g. *Why are sales dropping? Which channels bring high-value users?*).
2.  **Inspect Data**: Look at dimensions, data types, missing fields, and basic distributions.
3.  **Clean Data**: Cast data types, treat missing values, drop duplicates, and filter outliers.
4.  **Univariate Analysis**: Analyze variables one by one (distribution of sales, distribution of ages).
5.  **Bivariate/Multivariate Analysis**: Analyze relationships between variables (Sales vs. Category, Age vs. Spend).
6.  **Synthesize Insights**: Translate statistical metrics into recommendations for business stakeholders.

---

## 2. The Case Study Dataset
Imagine we receive a CSV file named `ecommerce_transactions.csv` containing:
*   `TransactionID`: Unique identifier.
*   `Date`: String format transaction dates.
*   `CustomerID`: Unique customer key.
*   `CustomerAge`: Age of customer (has missing values).
*   `Category`: Product category.
*   `SalesAmount`: Revenue amount (stored as string with "$" sign).
*   `Rating`: Customer rating from 1 to 5.

---

## 3. Step-by-Step Implementation

### Step 1: Loading and Inspection
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Load dataset
df = pd.read_csv("ecommerce_transactions.csv")

# Print initial diagnostics
print("Dataset Dimensions:", df.shape)
print("\nData Info:")
print(df.info())
print("\nNull Values Count:")
print(df.isna().sum())
```

### Step 2: Data Cleaning
```python
# 1. Parse dates
df['Date'] = pd.to_datetime(df['Date'], format='%Y-%m-%d')

# 2. Clean SalesAmount (remove '$' and commas, convert to float)
df['SalesAmount'] = df['SalesAmount'].astype(str).str.replace('$', '', regex=False)
df['SalesAmount'] = df['SalesAmount'].str.replace(',', '', regex=False)
df['SalesAmount'] = pd.to_numeric(df['SalesAmount'], errors='coerce')

# 3. Handle missing values
# Impute missing ages with the median age
median_age = df['CustomerAge'].median()
df['CustomerAge'] = df['CustomerAge'].fillna(median_age)

# 4. Handle duplicates
df.drop_duplicates(inplace=True)
```

### Step 3: Univariate Analysis (Summary Statistics)
```python
# Inspect distributions of numerical features
print(df[['CustomerAge', 'SalesAmount', 'Rating']].describe())

# Inspect categorical counts
print("\nTransactions by Category:")
print(df['Category'].value_counts(normalize=True) * 100)
```

### Step 4: Bivariate & Multivariate Analysis (Visualization)
Let's ask: *Does customer age impact how much they spend? Which category is most profitable?*

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# A. Category Sales Distribution (Boxplot)
sns.boxplot(data=df, x="Category", y="SalesAmount", palette="Set2", ax=axes[0, 0])
axes[0, 0].set_title("Sales Amount Range by Category")

# B. Age vs Sales Correlation (Scatter Plot)
sns.scatterplot(data=df, x="CustomerAge", y="SalesAmount", alpha=0.5, color="purple", ax=axes[0, 1])
axes[0, 1].set_title("Customer Age vs Sales Amount")

# C. Monthly Sales Trend (Line Plot)
df['Month'] = df['Date'].dt.to_period('M')
monthly_sales = df.groupby('Month')['SalesAmount'].sum().reset_index()
monthly_sales['Month'] = monthly_sales['Month'].dt.to_timestamp()

sns.lineplot(data=monthly_sales, x="Month", y="SalesAmount", marker="o", color="blue", ax=axes[1, 0])
axes[1, 0].set_title("Monthly Revenue Trend")
axes[1, 0].tick_params(axis='x', rotation=45)

# D. Average Rating by Category (Bar Plot)
sns.barplot(data=df, x="Category", y="Rating", color="teal", errorbar=None, ax=axes[1, 1])
axes[1, 1].set_title("Average Customer Rating by Category")

plt.tight_layout()
plt.show()
```

---

## 4. Synthesizing Insights (The Analyst's Report)
Based on the generated outputs, we draft a brief report for the executive team:

1.  **Revenue Performance**: Monthly revenue showed a downward trend in Q2. *Recommendation: Check if this is a seasonal trend or if a marketing campaign ended.*
2.  **Age Demographics**: The scatter plot shows a dense cluster of high-value sales ($>500$) for customers aged 30-45. *Recommendation: Target marketing budgets towards this age demographic.*
3.  **Customer Satisfaction**: The Average Rating bar plot shows that the "Electronics" category has a significantly lower rating (avg: 2.1 stars) compared to other categories. *Recommendation: Audit Electronics suppliers immediately to inspect product quality.*

---

## 5. Practice Exercises

### Exercise: Outlier Detection Challenge
In data analysis, outliers (extreme values) can skew statistics like the mean. Write a Python function that calculates the upper and lower limits for outliers in the `SalesAmount` column using the Interquartile Range (IQR) method:
*   $\text{IQR} = Q3 - Q1$
*   $\text{Lower Limit} = Q1 - 1.5 \times \text{IQR}$
*   $\text{Upper Limit} = Q3 + 1.5 \times \text{IQR}$
Filter the dataset to exclude these outliers.

*   **Answer**:
    ```python
    def remove_outliers_iqr(df, column):
        Q1 = df[column].quantile(0.25)
        Q3 = df[column].quantile(0.75)
        IQR = Q3 - Q1
        
        lower_limit = Q1 - 1.5 * IQR
        upper_limit = Q3 + 1.5 * IQR
        
        # Filter rows
        clean_df = df[(df[column] >= lower_limit) & (df[column] <= upper_limit)]
        return clean_df, lower_limit, upper_limit

    # Run function
    df_clean, low, high = remove_outliers_iqr(df, 'SalesAmount')
    print(f"Outlier limits: {low} to {high}")
    print(f"Rows removed: {len(df) - len(df_clean)}")
    ```
