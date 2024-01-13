# Lecture 2: Model Selection and Multinomial Logistic Regression

## Likelihood-based Model Selection

### Deviance Test

- The deviance ($D_k$) is used to compare a given model with k regressors with the **baseline model**.
  - The baseline model is the "perfect" fit to the data (overfitted), it has a distinct poisson mean ($\lambda_i$) for each $i$th observation.

$$D_k = 2 log \frac{\hat{l}_k}{\hat{l}_0}$$

### Akaike Information Criterion (AIC)

$$AIC_k = D_k + 2k$$

### Bayesian Information Criterion (BIC)

$$BIC_k = D_k + k log(n)$$

- BIC tends to select models with fewer regressors than AIC.

## Multinomial Logistic Regression

- Is a MLE-based GLM for when the response is **categorical** and **nominal**.

---

#### Catagorical Type Responses

- **Nominal**: unordered categories
  - e.g. red, green, blue
- **Ordinal**: ordered categories
  - e.g. low, medium, high

---
