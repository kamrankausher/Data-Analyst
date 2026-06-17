# Chapter 4: Correlation & Regression

Correlation and Regression are statistical techniques used to analyze relationships between variables and build predictive models. This chapter covers Pearson vs. Spearman correlation, Simple and Multiple Linear Regression using Ordinary Least Squares (OLS), model evaluation metrics ($R^2$, RMSE), residual diagnostics, and multicollinearity.

---

## 1. Correlation (Strength & Direction)

### A. Covariance
Measures how two variables change together. If one increases and the other increases, covariance is positive.
*   *Limit*: Dependent on scale (e.g., measuring weight in grams vs. kilograms changes the value), making it hard to compare.

### B. Pearson Correlation Coefficient ($r$)
Standardizes covariance to a scale between **-1.0 and +1.0** by dividing by the product of the standard deviations:
$$r = \frac{\sum (X - \bar{X})(Y - \bar{Y})}{\sqrt{\sum (X - \bar{X})^2 \sum (Y - \bar{Y})^2}}$$
*   $r = 1.0$: Perfect positive linear relationship.
*   $r = 0.0$: No linear relationship.
*   $r = -1.0$: Perfect negative linear relationship.

```
       Positive Correlation           Negative Correlation
               /                             \
              /                               \
             /                                 \
            /                                   \
```

### C. Spearman Rank Correlation
Measures the monotonic relationship between ranked variables. Use this when your variables are ordinal (ranked), or when the relationship is non-linear but moves in a constant direction.

---

## 2. Simple Linear Regression
Simple linear regression models the relationship between a dependent variable ($Y$) and a single predictor variable ($X$).

### The Regression Equation:
$$Y = \beta_0 + \beta_1 X + \epsilon$$

*   $Y$: Dependent variable (what we want to predict, e.g. Sales).
*   $X$: Independent variable (what we use to predict, e.g. Marketing Spend).
*   $\beta_0$ (Y-Intercept): The value of $Y$ when $X = 0$.
*   $\beta_1$ (Slope Coefficient): The change in $Y$ for every 1-unit increase in $X$.
*   $\epsilon$ (Error Term / Residual): The difference between the actual value and the value predicted by the line.

### Ordinary Least Squares (OLS)
OLS calculates the regression line by **minimizing the sum of squared residuals** (differences between observed data points and the line).
$$\text{Minimize} \sum \epsilon_i^2 = \sum (Y_i - \hat{Y}_i)^2$$

---

## 3. Evaluating Regression Models

### A. Coefficient of Determination ($R^2$ or R-squared)
$R^2$ measures the proportion of variance in the dependent variable that is explained by the independent variable(s).
*   Scale: **0.0 to 1.0** (0% to 100%).
*   *Interpretation*: An $R^2$ of `0.85` means that 85% of the variation in Sales is explained by the variation in Marketing Spend.

### B. Prediction Errors: MSE & RMSE
*   **Mean Squared Error (MSE)**: The average of the squared errors:
    $$\text{MSE} = \frac{1}{n}\sum (Y_i - \hat{Y}_i)^2$$
*   **Root Mean Squared Error (RMSE)**: The square root of the MSE. It returns the error to the original unit of measurement:
    $$\text{RMSE} = \sqrt{\text{MSE}}$$
    *   *Interpretation*: An RMSE of 5 means that, on average, the model's predictions are off by $\pm 5$ units.

### C. Residual Diagnostics (Regression Assumptions)
To trust your model, you must check the residuals (errors):
1.  **Mean of Residuals**: Must be close to 0.
2.  **Homoscedasticity**: The variance of the residuals must be constant across all levels of the predictor variable. If the errors spread out like a cone (variance increases as $X$ increases), it is called **Heteroscedasticity**, which violates regression assumptions.

---

## 4. Multiple Linear Regression
When you use multiple independent variables to predict $Y$:
$$Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \dots + \beta_k X_k + \epsilon$$

### A. Multicollinearity & VIF
Multicollinearity occurs when independent variables are highly correlated with each other (e.g., trying to predict weight using both height in inches and height in centimeters).
*   **Problem**: It inflates the variance of coefficients, making it hard to determine which variable is actually driving the prediction.
*   **Detection**: Calculate the **Variance Inflation Factor (VIF)**. A VIF value of $> 5$ or $10$ indicates problematic multicollinearity.

### B. Dummy Variables (Categorical Inputs)
Linear regression only accepts numbers. To include a categorical column like `Region` (North, South, West):
1.  Convert the categories into binary columns (e.g. `Region_North` [1 or 0], `Region_South` [1 or 0]).
2.  **The Dummy Variable Trap**: Always exclude one category column (reference category) from the model to prevent perfect multicollinearity. If you have 3 regions, include only 2 in the regression equation.

---

## 5. Practice Question Bank

### Beginner Question
You fit a linear regression model to predict employee sales performance (in thousands of dollars) based on hours of training received:
$$\text{Sales} = 15.2 + 2.4 \times \text{TrainingHours}$$
1.  What is the expected sales score for an employee with 0 training hours?
2.  What is the expected sales performance increase for an employee who receives 5 hours of training?
*   **Solution Walkthrough**:
    1.  When $\text{TrainingHours} = 0$, $\text{Sales} = 15.2$ (Intercept). The expected sales is $15,200.
    2.  For 5 hours of training, the increase is $2.4 \times 5 = 12$ (equivalent to $12,000). The total expected sales would be $15.2 + 12 = 27.2$ ($27,200).

### Intermediate Question
Explain what **Multicollinearity** is, why it is problematic for multiple linear regression models, and how to detect it.
*   **Solution Walkthrough**:
    *   **Multicollinearity**: Occurs when two or more independent variables in a regression model are highly correlated with each other.
    *   **Why it is problematic**: It makes it difficult to isolate the individual effect of each predictor on the dependent variable. It inflates the standard errors of the coefficients, making them unstable and reducing the statistical power of the model.
    *   **Detection**: Calculate the **Variance Inflation Factor (VIF)** for each predictor. A VIF $>5$ suggests moderate collinearity, and a VIF $>10$ indicates severe multicollinearity that should be resolved (e.g. by dropping one of the correlated columns).

### Advanced Question
You are evaluating two regression models built to predict customer lifetime value (CLV):
- **Model A**: $R^2 = 0.65$, $\text{RMSE} = \$120$, Predictors: `Spend`, `Age`.
- **Model B**: $R^2 = 0.66$, $\text{RMSE} = \$118$, Predictors: `Spend`, `Age`, `Height`, `ShoeSize`.
1.  Which model is a better fit for the data?
2.  Explain why you should look at the **Adjusted R-squared** instead of the standard R-squared when comparing these models.
*   **Solution Walkthrough**:
    1.  Model B has a slightly higher $R^2$ and lower RMSE. However, it includes two predictors (`Height`, `ShoeSize`) that are theoretically unrelated to customer lifetime value.
    2.  Standard $R^2$ is mathematically guaranteed to increase (or stay the same) whenever a new predictor is added to a model, even if that predictor is completely random noise.
    3.  **Adjusted R-squared** penalizes the model for adding extra variables that do not improve prediction quality.
    4.  If we calculate Adjusted R-squared, Model B's adjusted value will likely be lower than Model A's because the slight increase in $R^2$ (from 0.65 to 0.66) is offset by the penalty of adding two useless variables.
    5.  **Answer**: Model A is the better, more parsimonious model. You should use Adjusted R-squared to prevent overfitting when comparing models with different numbers of predictors.
