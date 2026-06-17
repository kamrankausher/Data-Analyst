# Chapter 2: Probability & Probability Distributions

Probability is the study of uncertainty. Because data analysts use historical samples to make decisions about the future, understanding probability rules and distribution models is essential for forecasting and risk management.

---

## 1. Probability Fundamentals & Bayes' Theorem

### A. Core Axioms
*   **Independent Events**: The outcome of Event A does not affect the probability of Event B (e.g. flipping a coin twice).
    $$P(A \text{ and } B) = P(A) \times P(B)$$
*   **Dependent Events**: The outcome of Event A affects the probability of Event B (e.g. drawing cards from a deck without replacement).
    $$P(A \text{ and } B) = P(A) \times P(B|A)$$
*   **Conditional Probability**: The probability of Event A occurring, given that Event B has already occurred:
    $$P(A|B) = \frac{P(A \text{ and } B)}{P(B)}$$

### B. Bayes' Theorem
Bayes' Theorem calculates the probability of a hypothesis ($H$) given new evidence ($E$):
$$P(H|E) = \frac{P(E|H) \times P(H)}{P(E)}$$
*   *Prior Probability $P(H)$*: The initial estimate of the probability of the hypothesis.
*   *Likelihood $P(E|H)$*: The probability of the evidence given the hypothesis.
*   *Posterior Probability $P(H|E)$*: The updated probability of the hypothesis after observing the evidence.

---

## 2. Discrete Probability Distributions
Discrete distributions model events with distinct, countable outcomes.

### A. Binomial Distribution
Models the number of successes in $n$ independent trials, where each trial has only two possible outcomes (success or failure) and a constant probability of success ($p$).
*   **Formula**:
    $$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$
*   *Use Case*: Calculating the probability of getting exactly 5 conversions from 20 sales calls, with a conversion rate of 10%.

### B. Poisson Distribution
Models the number of times an event occurs within a fixed interval of time or space, assuming events occur independently at a constant average rate ($\lambda$).
*   **Formula**:
    $$P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}$$
*   *Use Case*: Calculating the probability of a server receiving 100 requests in a minute, given that the historical average is 80 requests per minute.

---

## 3. Continuous Distributions: The Normal Distribution
Continuous variables (like revenue, weight, time) can take any fractional value.

### The Normal (Gaussian) Distribution
A symmetric, bell-shaped distribution defined by its Mean ($\mu$, center) and Standard Deviation ($\sigma$, spread).

```
                      Normal Distribution Curve
                              /\
                             /  \
                            /    \
                           /      \
                          /        \
               __________/__________\__________
               -3s   -2s  -1s   m   1s   2s   3s
```

### The Empirical Rule (68-95-99.7% Rule):
In a normal distribution:
1.  **68.2%** of all data points fall within $\pm 1$ standard deviation of the mean ($\mu \pm 1\sigma$).
2.  **95.4%** of all data points fall within $\pm 2$ standard deviations of the mean ($\mu \pm 2\sigma$).
3.  **99.7%** of all data points fall within $\pm 3$ standard deviations of the mean ($\mu \pm 3\sigma$).

---

## 4. The Central Limit Theorem (CLT)
The Central Limit Theorem is the foundation of inferential statistics.
*   **The Theorem**: If you draw random samples of size $n$ from *any* population distribution (even if the population is highly skewed or non-normal), the distribution of the sample means ($\bar{x}$) will approach a **Normal Distribution** as the sample size $n$ increases.
*   *Rule of thumb*: A sample size of $n \geq 30$ is generally sufficient for the CLT to apply.
*   *Why it matters*: It allows us to calculate confidence intervals and perform hypothesis tests on sample data without knowing the shape of the underlying population.

---

## 5. Practice Question Bank

### Beginner Question
A call center converts leads at a rate of 10% ($p = 0.10$). An agent makes 5 cold calls.
1.  Identify the distribution model.
2.  Write the expression to calculate the probability of getting exactly 2 sales conversions.
*   **Solution Walkthrough**:
    1.  The scenario consists of independent trials (calls), each with two outcomes (convert/fail) and a constant conversion probability. This matches the **Binomial Distribution**.
    2.  Parameters: $n = 5$, $k = 2$, $p = 0.10$.
    3.  Formula:
        $$P(X = 2) = \binom{5}{2} (0.10)^2 (0.90)^3 = 10 \times 0.01 \times 0.729 = 0.0729$$
    4.  **Answer**: There is a **7.29%** chance of getting exactly 2 conversions.

### Intermediate Question
A website experiences server errors at a rate of 2 per day ($\lambda = 2$). Calculate the probability that the server experiences exactly 3 errors tomorrow.
*   **Solution Walkthrough**:
    1.  The scenario counts occurrences over a fixed time interval (one day) at a constant average rate. This matches the **Poisson Distribution**.
    2.  Parameters: $\lambda = 2$, $k = 3$.
    3.  Formula:
        $$P(X = 3) = \frac{2^3 \times e^{-2}}{3!} = \frac{8 \times 0.1353}{6} \approx 0.1804$$
    4.  **Answer**: There is an **18%** chance of experiencing exactly 3 server errors tomorrow.

### Advanced Question
A marketing team notices that 15% of visitors to their site are high-intent buyers ($P(\text{Buyer}) = 0.15$). Of these buyers, 70% click on a specific promo banner ($P(\text{Promo}|\text{Buyer}) = 0.70$). Meanwhile, 10% of non-buyers also click on the banner ($P(\text{Promo}|\text{Non-Buyer}) = 0.10$).
If a visitor clicks on the promo banner, what is the updated probability that they are a high-intent buyer?
*   **Solution Walkthrough**:
    1.  Use Bayes' Theorem:
        $$P(\text{Buyer}|\text{Promo}) = \frac{P(\text{Promo}|\text{Buyer}) \times P(\text{Buyer})}{P(\text{Promo})}$$
    2.  Identify probabilities:
        *   $P(\text{Buyer}) = 0.15$
        *   $P(\text{Non-Buyer}) = 0.85$
        *   $P(\text{Promo}|\text{Buyer}) = 0.70$
        *   $P(\text{Promo}|\text{Non-Buyer}) = 0.10$
    3.  Calculate total probability of clicking the promo banner ($P(\text{Promo})$):
        $$P(\text{Promo}) = (P(\text{Promo}|\text{Buyer}) \times P(\text{Buyer})) + (P(\text{Promo}|\text{Non-Buyer}) \times P(\text{Non-Buyer}))$$
        $$P(\text{Promo}) = (0.70 \times 0.15) + (0.10 \times 0.85) = 0.105 + 0.085 = 0.19$$
    4.  Apply Bayes' Theorem:
        $$P(\text{Buyer}|\text{Promo}) = \frac{0.70 \times 0.15}{0.19} = \frac{0.105}{0.19} \approx 0.5526$$
    5.  **Answer**: There is a **55.3%** probability that a visitor who clicks the promo banner is a high-intent buyer.
