# Lecture 6: Local Regression

## Piecewise Local Regression

- Recall that classical linear regression (parametric) favours interpreatbility when aiming to make inference
- If goal is **accurate prediction**, then we can use **local regression** (non-linear)

### Piecewise Constant Regression

- Use step function to fit a piecewise constant function

$$
C_0(X_i) = I(X_i < c_1) = \begin{cases} 1 & \text{if } X_i < c_1 \\ 0 & \text{otherwise} \end{cases} \\
C_1(X_i) = I(c_1 \leq X_i < c_2) = \begin{cases} 1 & \text{if } c_1 \leq X_i < c_2 \\ 0 & \text{otherwise} \end{cases} \\
\vdots \\
C_{k-1}(X_i) = I(c_{k-1} \leq X_i < c_k) = \begin{cases} 1 & \text{if } c_{k-1} \leq X_i < c_k \\ 0 & \text{otherwise} \end{cases} \\
C_k(X_i) = I(X_i \leq c_k) = \begin{cases} 1 & \text{if } X_i \leq c_k \\ 0 & \text{otherwise} \end{cases}
$$

$$Y_i = \beta_0 + \beta_1 C_1(X_i) + \beta_2 C_2(X_i) + \cdots + \beta_k C_k(X_i) + \epsilon_i$$

- No need $C_0$ just by definition of $C_0$

```R
breakpoints <- c(10, 20, 30, 40, 50) # or 5 (number of breakpoints)

# create steps
data <- data |> mutate(
    steps = cut(data$var_to_split,
                breaks = breakpoints,
                right = FALSE))
levels(data$steps) # check levels

model <- lm(Y ~ steps, data = data)
```

### Non-Continous Piecewise Linear Regression

- Add interaction terms to the model

$$Y_i = \beta_0 + \beta_1C_1(X_i) + \cdots + \beta_kC_k(X_i) \\ + \beta_{k+1}X_i + \beta_{k+2}X_iC_1(X_i) + \cdots + \beta_{2k}X_iC_k(X_i) + \epsilon_i$$

```R
model_piecewise_linear <- lm(Y ~ steps * var_to_split, data = data)
```

### Continous Piecewise Linear Regression

$$Y_i = \beta_0 + \beta_1X_i + \beta_2(X_i - c_1)_+ + \cdots + \beta_k(X_i - c_{k-1})_+ + \epsilon_i$$

Where:

$$(X_i - c_j)_+ = \begin{cases} X_i - c_j & \text{if } X_i > c_j \\ 0 & \text{otherwise} \end{cases}$$

```R
model_piecewise_cont_linear <- lm(Y ~ var_to_split +
    I(var_to_split - breakpoint[2]) * I(var_to_split >= breakpoint[2]) +
    I(var_to_split - breakpoint[3]) * I(var_to_split >= breakpoint[3]) +
    I(var_to_split - breakpoint[4]) * I(var_to_split >= breakpoint[4]) +
    I(var_to_split - breakpoint[5]) * I(var_to_split >= breakpoint[5]),
    data = data)
```

## kNN Regression

- In this section:
  - $k$: number of neighbours
  - $p$: number of regressors
- **kNN** is a non-parametric method
- no training phase (lazy learner)
- finds $k$ closest neighbours to the query point $x_0$ and predicts the average of the neighbours' responses
- $k=1$ means no training error but overfitting

```R
model_knn <- knnreg(Y ~ X, data = data, k = 5)
```

## Locally Weighted Scatterplot Smoothing (LOWESS)

- Idea:

  - Find closest points to $x_i$ (query point)
  - assign weights based on distance
    - closer -> more weight
  - use weighted least squares for **second degree polynomial** fit

- Minimize sum of squares of weighted residuals

$$\sum_{i=1}^n w_i(y_i - \beta_0 - \beta_1x_i - \beta_2x_i^2)^2$$

- This model can deal with **heteroscedasticity** (non-constant variance)
  - Weighted least squares allows different variance for each observation
- Things to consider:
  - `span`: between 0 and 1, specifies the proportion of points considered as neighbours (more neighbours -> smoother fit)
  - `degree`: degree of polynomial to fit

```R
model_lowess <- lowess(Y ~ X, data = data, span = 0.5, degree = 2)
```
