# Lecture 4: Linear Mixed Effects Model

## Linear Fixed Effects Model

- **Linear Fixed Effects Model (LFE)** is a generalization of the linear regression model
- **Fixed Effects**: the parameters of the model
  - constant for all observations

### Limitations

- **Data Hierarchy**: the data is organized in a hierarchy
  - Can be due to **sampling levels**
  - e.g. investmests in different firms, students in different schools (sampling schemes may be different in different schools)
- Might have some correlation between datapoints in firms/ schools
  - violates the independence assumption (i.i.d. observations)

#### Example: Investments in different firms

- Goal: assessing the association of gross investment with market_value and capital in the population of American firms.
- Data: 11 firms, 20 observations per firm
  - 2 heirachical levels: firm and observation

1. Trial 1: ignore firm

```R
ordinary_model <- lm(
  formula = investment ~ market_value + capital,
  data = Grunfeld)
```

2. Trial 2: Different intercepts for different firms

```R
model_varying_intercept <- lm(
  # -1: so that baseline is not included as first intercept
    formula = investment ~ market_value + capital + firm - 1,
    data = Grunfeld)
```

3. Trial 3: OLS regeression for each firm

- This does NOT solve our goal.
- We want to find out among all firms, not one specific firm.

```R
model_by_firm <- lm(
  investment ~ market_value * firm + capital * firm,
  data = Grunfeld)
```

## Linear Mixed Effects Model

- Fundamental idea:
  - data subsets of elements share a correlation structure
  - i.e. all n rows of training data are not independent

$$ \text{mixed effect} = \text{fixed effect} + \text{random effect} $$

$$\beta_{0j} = \beta_0 + b_{0j}$$

- $\beta_{0j}$/ mixed effect: the intercept for the $j$th school/ firm
- $\beta_0$/ fixed effect: the average intercept
- $b_{0j}$/ random effect: the deviation of the $j$th school/ firm from the average intercept
  - $b_{0j} \sim N(0, \sigma^2_{0})$
  - independent of the error term $\epsilon$
- **Variance of the $i$th observation**:
  - $\sigma^2_{0} + \sigma^2_{\epsilon}$

### Full Equation for LME

$$
y_{ij} = \beta_{0j} + \beta_{1j}x_{1ij} + \beta_{2j}x_{2ij} + \epsilon_{ij} \\ = (\beta_0 + b_{0j}) + (\beta_1 + b_{1j})x_{1ij} + (\beta_2 + b_{2j})x_{2ij} + \epsilon_{ij}
$$

For $i$ in $1, 2, \ldots, n_j$ and $j$ in $1, 2, \ldots, J$

**Note**: $(b_{0j}, b_{1j}, b_{2j}) \sim N(\textbf{0}, \textbf{D})$

- $\textbf{0}$: vector of zero, e.g. $(0, 0, 0)^T$
- $\textbf{D}$: generic covariance matrix

$$\textbf{D} = \begin{bmatrix} \sigma^2_{0} & \sigma_{01} & \sigma_{02} \\ \sigma_{10} & \sigma^2_{1} & \sigma_{12} \\ \sigma_{20} & \sigma_{21} & \sigma^2_{2} \end{bmatrix} = \begin{bmatrix} \sigma^2_{0} & \rho_{01}\sigma_{0}\sigma_{1} & \rho_{02}\sigma_{0}\sigma_{2} \\ \rho_{10}\sigma_{0}\sigma_{1} & \sigma^2_{1} & \rho_{12}\sigma_{1}\sigma_{2} \\ \rho_{20}\sigma_{0}\sigma_{2} & \rho_{21}\sigma_{1}\sigma_{2} & \sigma^2_{2} \end{bmatrix}$$

- $\rho_{uv}$: pearson correlation between uth and vth random effects

### Model Fitting of LME

- use the `lmer` function from the `lme4` package

```R
mixed_intercept_model <- lmer(
  response ~ regressor_1 + regressor_2 +
    (1 | school), # random intercept by firm
  data
)

full_model <- lmer(
  response ~ regressor_1 + regressor_2 +
    (regressor_1 + regressor_2| school),
    # random intercept and slope by firm
  data
)
```

- Equation for mixed intercept model:

$$y_{ij} = (\beta_0 + b_{0j}) + \beta_1x_{1ij} + \beta_2x_{2ij} + \epsilon_{ij}$$

- Equation for full model:

$$y_{ij} = (\beta_0 + b_{0j}) + (\beta_1 + b_{1j})x_{1ij} + (\beta_2 + b_{2j})x_{2ij} + \epsilon_{ij}$$

### Inference of LME

- Cannot do inference using normal t-test

```R
summary(mixed_intercept_model)
summary(full_model)

# obtain coefficients
coef(mixed_intercept_model)$firm
coef(full_mixed_model)$firm
```

### Prediction with LME

1. Predict on _existing_ group

2. Predict on _new_ group

```R
predict(full_model,
  newdata = tibble(school = "new_school", regressor_1 = 1, regressor_2 = 2))
```
