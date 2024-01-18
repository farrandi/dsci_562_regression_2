# Lecture 2: Model Selection and Multinomial Logistic Regression

## Likelihood-based Model Selection

### Deviance Test

- The deviance ($D_k$) is used to compare a given model with k regressors ($l_k$) with the **baseline/ saturated model** ($l_0$).
  - The baseline model is the "perfect" fit to the data (overfitted), it has a distinct poisson mean ($\lambda_i$) for each $i$th observation.

$$D_k = 2 log \frac{\hat{l}_k}{\hat{l}_0}$$

- **Interpretation of $D_k$**

  - Large value of $D_k$ => poor fit compared to baseline model
  - Small value of $D_k$ => good fit compared to baseline model

#### $D_k$ in Poisson Regression

$$D_k = 2 \sum_{i=1}^n \left[ y_i log \left( \frac{y_i}{\hat{\lambda}_i} \right) - (y_i - \hat{\lambda}_i) \right]$$

\*note: when $y_i = 0$, log term is defined to be 0.

- Hypothesises are as follows (opposite of normal hypothesis):
  - $H_0$: Our model with k regressors fits the data better than the saturated model.
  - $H_A$: Otherwise

```R
glance(model) # D_k is "deviance" col

# to get p-value
pchisq(summary_poisson_model_2$deviance,
  df = summary_poisson_model_2$df.residual,
  lower.tail = FALSE
)
```

- Formally `deviance` is **residual deviance**, this is a test statistic.
- Asmptomatically, it has a null distribution of:

$$D_k \sim \chi^2_{n-k-1}$$

- dof: $n-k-1$
  - $n$ is the number of observations
  - $k$ is the number of regressors (including intercept)

#### Deviance for nested models

```R
anova(model_1, model_2, test = "Chisq")
# deviance column is \delta D_k
```

- model_1 is nested in model_2
- $H_0$: model_1 fits the data better as model_2
- $H_A$: model_2 fits the data better as model_1

$$\Delta D_k = D_{k_1} - D_{k_2} \sim \chi^2_{k_2 - k_1}$$

### Akaike Information Criterion (AIC)

$$AIC_k = D_k + 2k$$

- AIC can be used to compare models that are not nested.
- Smaller AIC is better (means better fit)
- Can get from `glance()` function

### Bayesian Information Criterion (BIC)

$$BIC_k = D_k + k log(n)$$

- BIC tends to select models with fewer regressors than AIC.
- smaller BIC is better (means better fit)
- Can get from `glance()` function

## Multinomial Logistic Regression

- Is a MLE-based GLM for when the response is **categorical** and **nominal**.
  - **Nominal**: unordered categories
    - e.g. red, green, blue
  - **Ordinal**: ordered categories
    - e.g. low, medium, high
- Similar to binomial logistic regression, but with more than 2 categories.
- Link function is the **logit** function.
  - need more than 1 logit function to model the probabilities of each category.
  - One category is the **baseline category**, the other categories are **compared to the baseline category**.

$$\eta_i^{(model 2, model 1)} = \log\left[\frac{P(Y_i = \texttt{model 2} \mid X_{i, 1}, X_{i,2}, X_{i,3})}{P(Y_i = \texttt{model 1} \mid X_{i, 1}, X_{i,2}, X_{i,3})}\right] = \beta_0^{(\texttt{model 2},\texttt{model 1})} + \beta_1^{(\texttt{model 2},\texttt{model 1})} X_{i, 1} + \beta_2^{(\texttt{model 2},\texttt{model 1})} X_{i, 2} + \beta_3^{(\texttt{model 2},\texttt{model 1})} X_{i, 3}$$

$$\eta_i^{(model 3, model 1)} = \log\left[\frac{P(Y_i = \texttt{model 3} \mid X_{i, 1}, X_{i,2}, X_{i,3})}{P(Y_i = \texttt{model 1} \mid X_{i, 1}, X_{i,2}, X_{i,3})}\right] = \beta_0^{(\texttt{model 3},\texttt{model 1})} + \beta_1^{(\texttt{model 3},\texttt{model 1})} X_{i, 1} + \beta_2^{(\texttt{model 3},\texttt{model 1})} X_{i, 2} + \beta_3^{(\texttt{model 3},\texttt{model 1})} X_{i, 3}$$

With some algebra, we can get the following (For m categories):

$$p_{i, \texttt{model 1}} = \frac{1}{1 + \sum_{j=2}^m e^{\eta_i^{(\texttt{model j}, \texttt{model 1})}}}$$

$$p_{i, \texttt{model 2}} = \frac{e^{\eta_i^{(\texttt{model 2}, \texttt{model 1})}}}{1 + \sum_{j=2}^m e^{\eta_i^{(\texttt{model j}, \texttt{model 1})}}}$$

- All probabilities sum to 1.

### Nuances: Baseline Category

- The baseline level is the level that is not included in the model.
  - can find using `levels()` function, the first level is the baseline level.

```R
levels(data$response) # to check levels

# to change levels
data$response <- recode_factor(data$response,
  "0" = "new_level_0",
  "1" = "new_level_1",
)
```

### Estimation of MLR

```R
model <- multinom(response ~ regressor_1 + regressor_2 + regressor_3,
  data = data)

# to get test statistics
mlr_output <- tidy(model,
  conf.int = TRUE, # to get confidence intervals (default is 95%)
  exponentiate = TRUE) # to get odds ratios
# default result is log odds ratios

# can filter p-values
mlr_output |> filter(p.value < 0.05)

# predict
predict(model, newdata = data, type = "probs")
# sum of all probabilities is 1
```

### Inference of MLR

- Check if regressor is significant using **Wald test**.

$$z_j^{(u,v)} = \frac{\hat{\beta}_j^{(u,v)}}{SE(\hat{\beta}_j^{(u,v)})}$$

- For large sample sizes, $z_j^{(u,v)} \sim N(0,1)$
- To test the hypothesis:
  - $H_0$: $\beta_j^{(u,v)} = 0$
  - $H_A$: $\beta_j^{(u,v)} \neq 0$

### Coefficient Interpretation for MLR

e.g. $\beta_1^{(b,a)} = 0.5$

- For a 1 unit increase in $X_1$, the odds of being in category $b$ is $e^{0.5} = 1.65$ times the odds of being in category $a$.

e.g. $\beta_2^{(c,a)} = -0.5$

- For a 1 unit increase in $X_2$, the odds of being in category $c$ decrease by $39\%$ [$ 1 - (e^{-0.5}) = 1 - 0.61 = 0.39$] less than being in category $a$.
