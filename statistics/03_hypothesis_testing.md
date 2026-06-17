# Chapter 3: Hypothesis Testing & A/B Testing

Hypothesis testing is the statistical method used to determine whether there is enough evidence in a sample of data to infer that a certain condition is true for the entire population. It prevents analysts from making decisions based on random noise.

---

## 1. The Hypothesis Testing Framework

### A. The Hypotheses
*   **Null Hypothesis ($H_0$)**: The baseline status quo. Assumes there is **no difference**, no effect, or no relationship between variables (e.g. *The new layout does not increase conversion rate*).
*   **Alternative Hypothesis ($H_a$)**: The claim we are testing. Assumes there **is a difference** or effect (e.g. *The new layout increases conversion rate*).

### B. Alpha ($\alpha$), Beta ($\beta$), and Type I/II Errors
*   **Significance Level ($\alpha$)**: The probability of rejecting the Null Hypothesis when it is actually true. Typically set to **0.05 (5%)**.
*   **Type I Error (False Positive)**: Rejecting $H_0$ when it is true. (Represented by $\alpha$).
*   **Type II Error (False Negative)**: Failing to reject $H_0$ when it is false. (Represented by $\beta$).
*   **Statistical Power ($1-\beta$)**: The probability of correctly rejecting the Null Hypothesis when it is false. Typically targeted at **80%**.

```
                      Hypothesis Test Decision Grid
                      
                                  True State of Reality
                              H0 is True       H0 is False
                           +----------------+----------------+
                 Reject H0 |  Type I Error  |    Correct     |
                           | (False Pos.)   |   Decision     |
  Decision Made            +----------------+----------------+
            Fail to Reject |    Correct     |  Type II Error |
                 H0        |  Decision      | (False Neg.)   |
                           +----------------+----------------+
```

### C. The P-Value
The p-value is the probability of obtaining test results at least as extreme as the observed results, assuming the Null Hypothesis is true.
*   **Decision Rule**:
    *   If **$\text{p-value} < \alpha$ (usually 0.05)**: Reject $H_0$. The result is statistically significant.
    *   If **$\text{p-value} \geq \alpha$**: Fail to reject $H_0$. The result is not statistically significant.

---

## 2. Choosing the Right Statistical Test

| Test Type | Input Variables | Output Variable | Business Use Case |
| :--- | :--- | :--- | :--- |
| **One-Sample T-Test** | 1 Continuous | N/A (Compare to Constant)| Checking if average weight of bags exceeds the advertised 50 lbs. |
| **Independent Two-Sample T-Test**| 1 Categorical (2 groups) | 1 Continuous | Comparing average sales of Group A vs. Group B in an A/B test. |
| **Paired T-Test** | 1 Continuous (Before/After) | 1 Continuous (Before/After) | Comparing employee test scores before and after a training seminar. |
| **One-Way ANOVA** | 1 Categorical (3+ groups) | 1 Continuous | Comparing average transaction values across 4 store branches. |
| **Chi-Square Test of Independence**| 2 Categorical | N/A (Frequency Counts) | Checking if customer gender is related to preferred product category. |

---

## 3. A/B Testing in Business
An A/B test is a randomized control experiment used to evaluate changes on digital platforms.

### A/B Test Step-by-Step Procedure:
1.  **Split Traffic**: Split incoming users into Group A (Control, current page) and Group B (Treatment, modified page).
2.  **Define metrics**: Select your Primary Metric (e.g. signup conversion rate) and Guardrail Metrics (e.g. page load speed, checkout errors).
3.  **Calculate Sample Size**: Do **not** stop your test early when a result looks significant (called "peeking"), as this inflates false positive rates. Run the test until you reach the target sample size calculated using:
    *   *Base Conversion Rate* (current performance).
    *   *Minimum Detectable Effect (MDE)*: The smallest change in conversion rate that is business-viable to detect.
    *   *Power (80%)* and *Significance ($\alpha = 5\%$)*.

---

## 4. Practice Question Bank

### Beginner Question
An e-commerce site runs an A/B test to see if a red checkout button increases conversion rate compared to the current blue button.
- $H_0$: Conversion Rate Red = Conversion Rate Blue
- $H_a$: Conversion Rate Red $\neq$ Conversion Rate Blue
The test concludes with a p-value of `0.012`. Using a significance level ($\alpha$) of `0.05`, what is your decision and recommendation?
*   **Solution Walkthrough**:
    1.  Compare the p-value (`0.012`) to the significance level $\alpha$ (`0.05`).
    2.  Since `0.012` is less than `0.05`, we reject the Null Hypothesis.
    3.  The difference in conversion rates is statistically significant.
    4.  **Answer**: Reject $H_0$. Recommendation: Deploy the red checkout button.

### Intermediate Question
Identify the correct statistical test to use for the following scenarios:
1.  Checking if customer satisfaction scores (1-10) differ between users who use Google Chrome, Safari, and Firefox.
2.  Checking if the proportion of users who convert (Yes/No) differs between those who see a pop-up discount vs. those who don't.
3.  Checking if average delivery times changed before and after changing delivery routes (tracking the same drivers).
*   **Solution Walkthrough**:
    1.  Scenario 1: Comparing means of 3 independent groups (Chrome, Safari, Firefox). Use **One-Way ANOVA**.
    2.  Scenario 2: Comparing relationships between two categorical variables (Converted: Yes/No, Pop-up: Yes/No). Use **Chi-Square Test of Independence**.
    3.  Scenario 3: Comparing the same continuous variable (delivery times) for the same group of subjects (same drivers) at two different times (before/after). Use a **Paired T-Test**.

### Advanced Question
You are planning an A/B test. The product manager wants to detect a conversion rate change from 10% to 10.5%.
1.  Explain what the **Minimum Detectable Effect (MDE)** is in this context.
2.  How will lowering the MDE to 10.1% affect the required sample size and duration of your test?
*   **Solution Walkthrough**:
    1.  The MDE is the smallest change in conversion rate that the experiment is designed to detect with statistical significance. In this context, the MDE is 0.5% (absolute change) or 5% (relative change).
    2.  If the team wants to detect an even smaller change (from 10% to 10.1%, an MDE of 0.1%), the signal-to-noise ratio decreases.
    3.  To detect a smaller difference, you need to collect more data to ensure the observed difference is not due to random noise.
    4.  **Answer**: Lowering the MDE will significantly **increase the required sample size** and **lengthen the duration** of the test.
