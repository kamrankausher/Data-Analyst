# Chapter 5: Data Visualization in Python

Data visualization translates statistical metrics into visual stories. This chapter covers the architecture of **Matplotlib** (the core engine) and **Seaborn** (the statistical visualization library built on top of Matplotlib).

---

## 1. The Matplotlib Architecture
Matplotlib uses a hierarchy of components:
1.  **Figure**: The main window or canvas container.
2.  **Axes**: The actual chart or subplot inside the figure. An axes contains title, labels, ticks, and legend.

```
   +---------------------------------------+
   | Figure (The Canvas Frame)             |
   |   +-------------------------------+   |
   |   | Axes (The Plot Grid)          |   |
   |   |                               |   |
   |   |      Line or Scatter Plot     |   |
   |   |                               |   |
   |   | Title, Labels, Ticks, Legend  |   |
   |   +-------------------------------+   |
   +---------------------------------------+
```

### The Object-Oriented Approach (Best Practice):
Avoid using the global state-based `plt.plot()` syntax. Instead, instantiate the Figure and Axes objects explicitly. This gives you absolute control over subplots and formatting.

```python
import matplotlib.pyplot as plt

# Create Figure and Axes
fig, ax = plt.subplots(figsize=(8, 5))

# Plot data on the Axes object
months = ['Jan', 'Feb', 'Mar', 'Apr']
sales = [12000, 15000, 14000, 19000]
ax.plot(months, sales, color='teal', linestyle='-', marker='o', linewidth=2)

# Format the Axes
ax.set_title("Q1 Sales Performance", fontsize=14, fontweight='bold', color='navy')
ax.set_xlabel("Month", fontsize=11)
ax.set_ylabel("Revenue ($)", fontsize=11)
ax.grid(True, linestyle=':', alpha=0.6)

# Display or save
plt.tight_layout()
plt.show()
# fig.savefig("sales_trend.png", dpi=300)
```

---

## 2. Statistical Visualization with Seaborn
Seaborn simplifies chart creation by interfacing directly with Pandas DataFrames.

### A. Distribution Analysis (Histograms & KDE)
Histograms group numerical values into bins to show their frequency distribution.
```python
import seaborn as sns
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 5))
sns.histplot(data=df, x="Age", kde=True, bins=15, color="coral", ax=ax)
ax.set_title("Customer Age Distribution")
plt.show()
```

### B. Category Comparisons (Boxplots & Violinplots)
Boxplots show the 5-number summary (Min, Q1, Median, Q3, Max) and flag outliers.
```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(data=df, x="Department", y="Salary", hue="Gender", palette="pastel", ax=ax)
ax.set_title("Salary Spread by Department")
plt.show()
```

### C. Relationship Analysis (Scatter plots & Heatmaps)
Heatmaps are used to visualize the strength of relationships between all numerical variables in a dataset.
```python
# Calculate correlation matrix
corr = df[['Sales', 'Profit', 'Discount', 'Units']].corr()

# Draw heatmap
fig, ax = plt.subplots(figsize=(6, 5))
sns.heatmap(corr, annot=True, cmap="coolwarm", fmt=".2f", linewidths=0.5, ax=ax)
ax.set_title("Variable Correlation Heatmap")
plt.show()
```

---

## 3. Practice Question Bank

### Beginner Question
You have a DataFrame of user signups. You want to plot a simple bar chart showing the count of signups for each day of the week. Write the code using Seaborn.
*   **Solution Walkthrough**:
    1.  Use `sns.countplot()`.
    2.  Specify the data and the column for the x-axis: `x="DayOfWeek"`.
    3.  **Correct Code**:
        ```python
        import seaborn as sns
        import matplotlib.pyplot as plt
        
        fig, ax = plt.subplots(figsize=(8, 5))
        sns.countplot(data=df, x="DayOfWeek", palette="Blues", ax=ax)
        ax.set_title("User Signups by Day of Week")
        plt.show()
        ```

### Intermediate Question
You want to create a 1-row, 2-column subplot grid.
- Subplot 1 should be a scatter plot showing `Sales` vs. `Profit`.
- Subplot 2 should be a boxplot showing `Profit` across different `Regions`.
Write the Matplotlib/Seaborn script.
*   **Solution Walkthrough**:
    1.  Create a subplot grid: `fig, axes = plt.subplots(1, 2, figsize=(14, 5))`.
    2.  `axes` will be an array containing the two subplot axes: `axes[0]` and `axes[1]`.
    3.  Plot the scatter plot on `axes[0]`.
    4.  Plot the boxplot on `axes[1]`.
    5.  **Correct Code**:
        ```python
        import seaborn as sns
        import matplotlib.pyplot as plt
        
        # Create 1x2 subplot grid
        fig, axes = plt.subplots(1, 2, figsize=(14, 5))
        
        # Subplot 1: Scatter plot
        sns.scatterplot(data=df, x="Sales", y="Profit", color="purple", ax=axes[0])
        axes[0].set_title("Sales vs. Profit")
        
        # Subplot 2: Boxplot
        sns.boxplot(data=df, x="Region", y="Profit", palette="Set3", ax=axes[1])
        axes[1].set_title("Profit Distribution by Region")
        
        plt.tight_layout()
        plt.show()
        ```

### Advanced Question
You are preparing a correlation heatmap for a presentation. The default heatmap contains labels that overlap, and the color scale is hard to read.
Write a script that:
1.  Calculates correlation.
2.  Draws a heatmap using a diverging color palette (`coolwarm`) centered at 0.
3.  Masks the upper triangular part of the matrix (since correlation matrices are symmetric and the top-right triangle contains redundant values).
4.  Rotates the x-axis labels by 45 degrees to prevent overlap.
*   **Solution Walkthrough**:
    1.  Calculate correlation: `corr = df.corr(numeric_only=True)`.
    2.  To generate a mask for the upper triangle, use `np.triu()` on a boolean matrix of the same shape: `mask = np.triu(np.ones_like(corr, dtype=bool))`.
    3.  Pass the mask to the heatmap: `sns.heatmap(..., mask=mask)`.
    4.  Rotate labels using `ax.set_xticklabels(..., rotation=45)`.
    5.  **Correct Code**:
        ```python
        import numpy as np
        import seaborn as sns
        import matplotlib.pyplot as plt
        
        # 1. Correlation
        corr = df.corr(numeric_only=True)
        
        # 2. Generate mask for upper triangle
        mask = np.triu(np.ones_like(corr, dtype=bool))
        
        # 3. Plot
        fig, ax = plt.subplots(figsize=(8, 6))
        sns.heatmap(
            corr, 
            mask=mask, 
            annot=True, 
            cmap="coolwarm", 
            vmin=-1, vmax=1, center=0, 
            linewidths=0.5, 
            ax=ax
        )
        
        # 4. Format labels
        ax.set_title("Symmetric Variable Correlations", fontsize=12, fontweight='bold')
        plt.xticks(rotation=45, ha='right') # ha='right' aligns text cleanly
        plt.tight_layout()
        plt.show()
        ```
