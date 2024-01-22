# Lecture 3: Ordinal Logistic Regression

## Ordinal Logistic Regression

- **Ordinal**: has a natural ordering
- There might be loss of information when using MLR for ordinal data
- We are going to use the **proportional odds model** for ordinal data
  - It is a **cumulative logit model**

### Preprocessing for Ordinal Data

- Reorder the levels of the response variable

```R
data$response <- as.ordered(data$response)
data$response <- fct_relevel(
  data$response,
  c("unlikely", "somewhat likely", "very likely")
)
levels(data$response)
```

### Data Model for OLR

- For a response with **$m$** responses and **$k$** regressors, the model is:

- We will have:
  - $m-1$ equations (link functions: logit)
  - $m-1$ intercepts
  - $k$ regression coefficients

#### Link Functions for m responses OLR

$$
\begin{gather*}
\text{Level } m - 1 \text{ or any lesser degree versus level } m\\
\text{Level } m - 2 \text{ or any lesser degree versus level } m - 1 \text{ or any higher degree}\\
\vdots \\
\text{Level } 1 \text{ versus level } 2 \text{ or any higher degree}\\
\end{gather*}
$$

$$
\begin{gather*}
\eta_i^{(m - 1)} = \log\left[\frac{P(Y_i \leq m - 1 \mid X_{i,1}, \ldots, X_{i,k})}{P(Y_i = m \mid X_{i,1}, \ldots, X_{i,k})}\right] = \beta_0^{(m - 1)} - \beta_1 X_{i, 1} - \beta_2 X_{i, 2} - \ldots - \beta_k X_{i, k} \\
\eta_i^{(m - 2)} = \log\left[\frac{P(Y_i \leq m - 2 \mid X_{i,1}, \ldots, X_{i,k})}{P(Y_i > m - 2 \mid X_{i,1}, \ldots, X_{i,k})}\right] = \beta_0^{(m - 2)} - \beta_1 X_{i, 1} - \beta_2 X_{i, 2} - \ldots - \beta_k X_{i, k} \\
\vdots \\
\eta_i^{(1)} = \log\left[\frac{P(Y_i = 1 \mid X_{i,1}, \ldots, X_{i,k})}{P(Y_i > 1 \mid X_{i,1}, \ldots, X_{i,k})}\right] = \beta_0^{(1)} - \beta_1 X_{i, 1} - \beta_2 X_{i, 2} - \ldots - \beta_k X_{i, k}.
\end{gather*}
$$

#### Probability that $Y_i$ is in level j

$$p_{i,j} = P(Y_i = j \mid X_{i,1}, \ldots, X_{i,k}) = P(Y_i \leq j \mid ...) - P(Y_i \leq j - 1 \mid ...)$$

- $i$ is the index of the observation
- $j$ is the level of the response variable

$$\sum_{j = 1}^{m} p_{i,j} = 1$$

### Estimation of OLR

- use `MASS::polr` function

```R
ordinal_model <- polr(
  formula = response ~ regressor_1 + regressor_2,
  data = data,
  Hess = TRUE # Hessian matrix of log-likelihood
)
```

### Inference of OLR

- Similar to MLR using **Wald test**

```R
cbind(
  tidy(ordinal_model),
  p.value = pnorm(abs(tidy(ordinal_model)$statistic),
    lower.tail = FALSE
  ) * 2
)
# confidence intervals

confint(ordinal_model) # default is 95%
```

### Coefficient Interpretation of OLR

- e.g. $\beta_1 = 0.6$
  - For a one unit increase in $X_1$, the odds of being in a higher category is $e^{0.6} = 1.82$ times the odds of being in a lower category, holding all other variables constant.

### Predictions

```R
predict(ordinal_model, newdata = data, type = "probs")
# returns probabilities for each level
```

- To get the **corresponding predicted** cumulative odds for a new observation, use `VGAM::vglm` function

```R
olr <- vglm(
  response ~ regressor_1 + regressor_2,
  propodds, # for proportional odds model
  data,
)

# can also predict using this model, same as code block above
predict(olr, newdata = data, type = "response")

# get predicted cumulative odds
predict(olr, newdata = data, type = "link") |>
  exp() # to get odds instead of log odds
```

- Interpret the predicted cumulative odds as:
  - e.g. $logitlink(P[Y_i \geq j]) = 2.68$
    - A student with [data for $X_i$] is 2.68 times more likely to be in $j$ or higher category than in category $j - 1$ or lower, holding all other variables constant.
  - e.g. $logitlink(P[Y_i \geq 2]) = 0.33$
    - A student with [data for $X_i$] is 3.03 (1/0.33) times more likely to be in $j$ category or lower than in category j or higher, holding all other variables constant.

### Non-proportional Odds

- If the proportional odds assumption is not met, we can use the **partial proportional odds model**.
- Test for proportional odds assumption using the **Brant-Wald test**.
  - $H_0$: Our OLR model globally fulfills the proportional odds assumption.
  - $H_A$: Our OLR model does not globally fulfill the proportional odds assumption.

```R
brant(ordinal_model)
```

- If the proportional odds assumption is not met, we can use the **generalized ordinal logistic regression model**.
  - Basically all $\beta$'s are allowed to vary across the different levels of the response variable.
