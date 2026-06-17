# Chapter 1: Descriptive Statistics

Descriptive statistics is the branch of mathematics that summarizes and describes the characteristics of a dataset. Before building models or testing hypotheses, an analyst must understand the shape, center, and spread of their data.

---

## 1. Measures of Central Tendency
Central tendency identifies the single central value that represents the middle of a distribution.

*   **Mean (Arithmetic Average)**: Sum of all values divided by the number of values ($\mu$ for population, $\bar{x}$ for sample).
    *   *Limit*: Highly sensitive to outliers. A single extreme value shifts the mean.
*   **Median**: The middle value when data is sorted from low to high. If there is an even number of values, it is the average of the two middle numbers.
    *   *Strength*: Robust to outliers. Extreme values do not shift the median.
*   **Mode**: The value that occurs most frequently in the dataset. Useful for categorical data.

### Outlier Sensitivity Example:
Consider a team of 5 developers with salaries: `[50k, 55k, 58k, 60k, 62k]`.
*   Mean: `57k`
*   Median: `58k`

Now, a CEO joins with a salary of `500k`. The dataset becomes `[50k, 55k, 58k, 60k, 62k, 500k]`.
*   Mean: `130.8k` (Does not represent the typical team salary anymore!)
*   Median: `59k` (Remains robust and representative of the team).

---

## 2. Measures of Dispersion (Spread)
Dispersion measures how spread out the values are from the center.

### A. Range
The difference between the maximum and minimum value. Highly sensitive to outliers.

### B. Variance ($\sigma^2$)
Measures the average squared deviation of values from their mean.
*   **Population Variance**:
    $$\sigma^2 = \frac{\sum (X - \mu)^2}{N}$$
*   **Sample Variance**: (We divide by $n-1$ to correct for bias, called Bessel's Correction)
    $$s^2 = \frac{\sum (x - \bar{x})^2}{n-1}$$

### C. Standard Deviation ($\sigma$ or $s$)
The square root of the variance. It returns the metric to the original unit of measurement (e.g. if variance is in dollars-squared, standard deviation is in dollars).
$$\sigma = \sqrt{\sigma^2}$$

### D. Interquartile Range (IQR)
The spread of the middle 50% of the data. It is calculated by subtracting the first quartile ($Q1$) from the third quartile ($Q3$):
$$\text{IQR} = Q3 - Q1$$
*   **Quartiles**:
    *   $Q1$ (25th percentile): 25% of data is below this value.
    *   $Q2$ (50th percentile): The Median.
    *   $Q3$ (75th percentile): 75% of data is below this value.

---

## 3. Shape of Data: Skewness & Kurtosis

### A. Skewness (Asymmetry)
Measures the asymmetry of the probability distribution.

```
       Right Skew (Positive)                Left Skew (Negative)
             /\                                    /\
            /  \                                  /  \
           /    \                                /    \
  ________/______\______                ________/______\______
         /        \----                         ----/        \
  Tail points to the RIGHT              Tail points to the LEFT
  Mean > Median > Mode                  Mode > Median > Mean
```

*   **Symmetric (Normal)**: Skewness is 0. Mean = Median = Mode.
*   **Right-Skewed (Positive Skew)**: Tail extends to the right. Mean is pulled towards the tail by high outliers. (e.g. Household income).
*   **Left-Skewed (Negative Skew)**: Tail extends to the left. Mean is pulled down by low outliers. (e.g. Age of death).

### B. Kurtosis (Peakedness)
Measures the "tailedness" of the distribution (frequency of extreme outliers).
*   **Mesokurtic**: Normal distribution shape.
*   **Leptokurtic (High Kurtosis)**: Sharp peaked profile with heavy, thick tails (high probability of extreme outliers — common in financial market returns).
*   **Platykurtic (Low Kurtosis)**: Flat, broad profile with thin tails.

---

## 4. Z-Scores
A Z-score represents how many standard deviations a data point is away from the mean. It is used to standardize different variables for comparison.
$$Z = \frac{x - \mu}{\sigma}$$
*   A Z-score of `0` means the value is exactly average.
*   A Z-score of `2` means the value is 2 standard deviations above the average.
*   **Outlier detection**: Any value with a Z-score of $> 3$ or $< -3$ is typically flagged as an outlier.

---

## 5. Practice Question Bank

### Beginner Question
You have a sample dataset of customer satisfaction scores (scale of 1-10): `[5, 7, 8, 8, 10]`.
Compute the Sample Variance and the IQR.
*   **Answer**:
    1.  **Mean ($\bar{x}$)**: $(5+7+8+8+10) / 5 = 7.6$
    2.  **Deviations from Mean ($x - \bar{x}$)**: `[-2.6, -0.6, 0.4, 0.4, 2.4]`
    3.  **Squared Deviations**: `[6.76, 0.36, 0.16, 0.16, 5.76]`
    4.  **Sum of Squares**: $6.76 + 0.36 + 0.16 + 0.16 + 5.76 = 13.2$
    5.  **Sample Variance ($s^2$)**: $13.2 / (5-1) = 3.3$
    6.  **IQR**:
        *   Sorted: `[5, 7, 8, 8, 10]`
        *   Median ($Q2$) = `8`
        *   $Q1$ (25th percentile) = `6`
        *   $Q3$ (75th percentile) = `9`
        *   $\text{IQR} = 9 - 6 = 3$

### Exercise 2: Z-Score Outlier Check
The average height of a population is 170 cm with a standard deviation of 10 cm. A person is measured at 205 cm tall. Compute their Z-score. Is this height mathematically classified as an outlier?
*   **Answer**:
    *   $Z = (205 - 170) / 10 = 35 / 10 = 3.5$
    *   Yes. Because the Z-score is $3.5$ (which is $>3$), this height is classified as a statistical outlier.
