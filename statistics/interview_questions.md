# Statistics Data Analyst Interview Prep Guide

This guide contains the most frequently asked Statistics and Experimental Design (A/B Testing) interview questions in Data Analyst interviews. Each question includes a detailed, professional answer.

---

## 1. Descriptive Statistics & Distributions

### Question 1: In what scenario would you prefer using the Median over the Mean to report typical performance?
*   **Answer**: In highly skewed distributions containing extreme outliers.
    *   *Why*: The Mean is calculated by summing all values and dividing by the count. A single extreme outlier pulls the mean in its direction. The Median is the middle value of the sorted dataset, making it robust to outliers.
    *   *Business Example*: Household income or housing prices. A few billionaires or multi-million dollar mansions will skew the mean income/price upwards, making it look much higher than what a typical family experiences. The median provides a more representative center of the distribution.

---

## 2. Inferential Statistics & Hypothesis Testing

### Question 2: Explain the difference between Type I and Type II errors. How do you control for these in an A/B test?
*   **Type I Error (False Positive)**: Rejecting the Null Hypothesis when it is actually true. (e.g. declaring a checkout layout change increased conversion rate when the change was actually due to random noise).
    *   *Control*: Controlled by setting the significance level ($\alpha$), typically **0.05 (5%)**. This means there is a 5% chance of committing a Type I error.
*   **Type II Error (False Negative)**: Failing to reject the Null Hypothesis when it is false. (e.g. claiming a checkout change had no effect when it actually increased conversion rate).
    *   *Control*: Controlled by ensuring the test has sufficient **Statistical Power ($1-\beta$)**, typically targeted at **80%**. You increase power by increasing the sample size or extending the test duration.

---

## 3. A/B Testing & p-values

### Question 3: How would you explain what a "p-value" is to a non-technical business stakeholder?
*   **Answer**: *"Imagine we run a coin-flip experiment. The coin lands heads 9 times out of 10. We want to know if the coin is rigged (biased).*
    *   *The p-value is the probability that a fair coin would land heads 9 or more times by pure random luck.*
    *   *In business, a p-value of 0.03 means: if the new website change actually had zero real impact, there is only a 3% chance that we would see a conversion increase this large by pure random luck.*
    *   *Because 3% is a very small chance, we reject the idea that this was luck and conclude the change had a real impact."*

---

## 4. Regression & Modeling

### Question 4: What is Multicollinearity in a Multiple Linear Regression model? Why is it problematic, and how do you resolve it?
*   **Multicollinearity**: Occurs when two or more independent variables are highly correlated with each other.
*   **Why it is problematic**: It makes it difficult to isolate the individual effect of each predictor on the dependent variable. It inflates the standard errors of the coefficients, making them unstable and reducing the statistical power of the model.
*   **How to detect**: Calculate the **Variance Inflation Factor (VIF)**. A VIF value of $>5$ suggests moderate collinearity, and a VIF $>10$ indicates severe multicollinearity.
*   **How to resolve**:
    1.  Remove one of the highly correlated variables.
    2.  Combine the correlated variables into a single index (e.g. using Principal Component Analysis).

---

## 5. Statistical Challenge

### Scenario:
You are analyzing an A/B test. The p-value of the test is `0.045` (using $\alpha = 0.05$). The product manager is excited and wants to deploy the feature immediately. However, you notice that the sample size was very small, and the test only ran for 3 days. What is your recommendation, and why?
*   **Solution Walkthrough**:
    1.  A p-value below $\alpha$ indicates statistical significance. However, a small sample size coupled with a short duration poses significant risks.
    2.  **Novelty Effect**: Users often interact with a new feature simply because it is new, creating a temporary surge in conversion that fades after a few days.
    3.  **Weekday/Weekend Bias**: Running a test for only 3 days fails to capture a full weekly business cycle. User behavior on weekends is often different from weekdays.
    4.  **Sample Size issues**: Small samples have lower statistical power, meaning they are prone to inflating the effect size (exaggerating the real impact).
    5.  **Recommendation**: Do not deploy the feature. Recommend letting the test run for at least 1-2 full weeks to collect more data and eliminate weekly cycle and novelty bias.
