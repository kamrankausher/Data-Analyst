# Chapter 1: NumPy (Numerical Python) for Data Analysis

NumPy is the foundational library for scientific computing in Python. While you will spend most of your time in Pandas as a Data Analyst, Pandas is built on top of NumPy. Understanding how NumPy manages arrays, indices, and vectorization is crucial for high-performance data processing.

---

## 1. Why NumPy? Lists vs. Arrays

### A. Python List Memory Structure
A Python list is an array of pointers to objects in memory. Each object is boxed (wrapped with metadata like data type and reference counts). Because of this:
*   Items can be scattered across your RAM, causing cache misses.
*   Python must check the data type of every element during a loop.

### B. NumPy Array (ndarray) Memory Structure
A NumPy array is a contiguous block of raw homogeneous data (every element is of the exact same data type).
*   **Contiguous Memory Layout**: Elements are stored next to each other in memory, allowing CPUs to read blocks of data into cache memory at once.
*   **Vectorization (SIMD)**: Vectorized operations run compiled C code behind the scenes. This allows the CPU to perform operations on multiple data points in a single instruction (Single Instruction Multiple Data).

```
   Python List (Array of Pointers):
   List Box -> [ Pointer1, Pointer2, Pointer3 ]
                  |         |         |
                  v         v         v
               [Obj 10]  [Obj 20]  [Obj 30]  <-- Scattered in RAM
   
   NumPy Array (Contiguous Block):
   Array Box -> [ 10 | 20 | 30 ]             <-- Stored together in memory
```

---

## 2. Array Creation & Data Types
Specify data types (`dtypes`) to optimize memory usage:

```python
import numpy as np

# A. Creation
arr_1d = np.array([1, 2, 3], dtype=np.int32) # Uses 32-bit integers
zeros = np.zeros((3, 4))                     # 3x4 grid of 0.0 floats
ones = np.ones((2, 2))                       # 2x2 grid of 1.0 floats
range_arr = np.arange(0, 10, 2)              # [0, 2, 4, 6, 8]
linear_space = np.linspace(0, 1, 5)          # 5 numbers from 0 to 1: [0., 0.25, 0.5, 0.75, 1.]
```

---

## 3. Slicing and Indexing (1D and 2D arrays)

```
   2D Array: data
   
          Col 0  Col 1  Col 2
Row 0   [[  10,    20,    30  ],
Row 1    [  40,    50,    60  ],
Row 2    [  70,    80,    90  ]]
```

### A. Syntax: `array[row_start:row_end, col_start:col_end]`
*   *Cell lookup*: `data[1, 2]` -> `60`
*   *Row slice*: `data[0, :]` -> `[10, 20, 30]`
*   *Column slice*: `data[:, 0]` -> `[10, 40, 70]`
*   *Subgrid extraction (top-left 2x2)*:
    `data[0:2, 0:2]` -> `[[10, 20], [40, 50]]`

### B. Boolean Indexing (Filtering)
Pass boolean conditions inside brackets:
```python
spend = np.array([50, 120, 200, 80])
# Filter for values above 100
mask = spend > 100 # Returns [False, True, True, False]
high_spend = spend[mask] # Returns [120, 200]
```

---

## 4. Vectorized Operations & Broadcasting
Vectorization means operations are automatically applied element-wise.
*   `arr1 + arr2`
*   `arr1 * 2` (Scalar multiplication)

### Broadcasting Rules:
Broadcasting allows arithmetic operations on arrays of different shapes. NumPy matches shapes starting from the trailing dimension:
1.  **Rule 1**: If dimensions are equal, or if one of the dimensions is `1`, they are compatible.
2.  **Rule 2**: If they are incompatible, a `ValueError` is raised.

*   *Example: Adding a 1D array of shape `(3,)` to a 2D array of shape `(2,3)`:*
    ```python
    A = np.array([[10, 20, 30],
                  [40, 50, 60]]) # Shape: (2, 3)
    B = np.array([1, 2, 3])      # Shape: (3,) -> broadcasted to (2, 3) by duplicating rows
    
    print(A + B)
    # Output: [[11, 22, 33],
    #          [41, 52, 63]]
    ```

---

## 5. Aggregations & The Axis Concept
*   `axis=0`: Calculations run **vertically** down the rows (column-wise summaries).
*   `axis=1`: Calculations run **horizontally** across the columns (row-wise summaries).

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

# Sum down columns
print(np.sum(matrix, axis=0)) # Output: [5, 7, 9]

# Sum across rows
print(np.sum(matrix, axis=1)) # Output: [6, 15]
```

---

## 6. Practice Question Bank

### Beginner Question
You have an array of product prices in USD: `prices_usd = np.array([10, 25, 50, 100])`.
1.  Write a vectorized expression to convert all prices to EUR, assuming an exchange rate of `0.92`.
2.  Filter the resulting array to display only prices that are greater than 20 EUR.
*   **Solution Walkthrough**:
    1.  Convert using vectorization: `prices_eur = prices_usd * 0.92`.
    2.  Create a filter mask: `mask = prices_eur > 20`.
    3.  Apply mask to array: `prices_eur[mask]`.
    4.  **Correct Code**:
        ```python
        prices_usd = np.array([10, 25, 50, 100])
        prices_eur = prices_usd * 0.92
        filtered_eur = prices_eur[prices_eur > 20]
        ```

### Intermediate Question
Explain what **Broadcasting** is, and trace whether the following operation runs successfully or fails. If it succeeds, show the output.
```python
x = np.array([[1], [2], [3]]) # Shape (3, 1)
y = np.array([10, 20])        # Shape (2,)
result = x + y
```
*   **Solution Walkthrough**:
    1.  Check the dimensions:
        *   `x` shape is `(3, 1)`
        *   `y` shape is `(2,)` -> prepended with 1 to become `(1, 2)` for matching.
    2.  Compare dimension sizes:
        *   Dimension 2: `x` has `1`, `y` has `2`. Since one size is `1`, they are compatible (dimension 2 broadcasts to size 2).
        *   Dimension 1: `x` has `3`, `y` has `1`. Since one size is `1`, they are compatible (dimension 1 broadcasts to size 3).
    3.  Both dimensions broadcast to a final shape of `(3, 2)`.
    4.  `x` broadcasts to:
        `[[1, 1], [2, 2], [3, 3]]`
    5.  `y` broadcasts to:
        `[[10, 20], [10, 20], [10, 20]]`
    6.  Sum them:
        `[[11, 21], [12, 22], [13, 23]]`
    7.  **Answer**: Yes, it succeeds. Output:
        ```python
        [[11, 21],
         [12, 22],
         [13, 23]]
        ```

### Advanced Question
You have a 2D array of sensor readings where rows represent hours of the day (24 hours) and columns represent different locations (5 locations). Some readings contain invalid values represented by `-999`.
Write a script to:
1.  Replace all `-999` values with `np.nan` (floats).
2.  Calculate the mean reading for each location (each column), ignoring the `np.nan` values.
*   **Solution Walkthrough**:
    1.  Convert array to float to support NaN values: `readings = readings.astype(float)`.
    2.  Use boolean indexing to replace values: `readings[readings == -999] = np.nan`.
    3.  To calculate column-wise means while ignoring NaNs, use `np.nanmean()` instead of `np.mean()`.
    4.  Calculate column-wise: set `axis=0`.
    5.  **Correct Code**:
        ```python
        # Convert to float to allow NaN values
        readings = readings.astype(float)
        # Replace placeholders
        readings[readings == -999.0] = np.nan
        # Column means
        location_means = np.nanmean(readings, axis=0)
        ```
