# Lecture 8: Missing Data

## Missing Data

### Types of Missing Data

#### Missing Completely at Random (MCAR)

- The probability of missing data is the same for all observations
- Missingness is independent of data (Ideal case because there is no pattern)
- No systematic differences between missing and non-missing data

#### Missing at Random (MAR)

- The probability of missing data depends on observed data
- Can use imputation to fill in missing data:
  - **Hot deck**: Replace missing value with a value from the _same dataset_
  - **Cold deck**: Replace missing value with a value from a _different dataset_

#### Missing Not at Random (MNAR)

- The probability of missing data depends on unobservable quantities
- E.g. missing data on income for people who are unemployed

### Handling Missing Data

#### Listwise Deletion

- Remove all observations with missing data
- If data is MCAR, this is unbiased
  - Also increases standard errors since we're using less data
- If data is MAR or MNAR, this is **biased** (CAREFUL)
  - e.g. if missing data is related to income, lower income will omit telling us their income, so removing them will bias our data to higher income.

#### Mean Imputation

- Replace missing data with the mean of the observed data
  - Can only be used on continuous/ count data
- Artificially reduces standard errors (drawback)
  - Also reduces variance, which is not good

```r
library(mice)

data <- mice(data, seed = 1, method = "mean")
complete(data)
```

#### Regression Imputation

- Use a regression model to predict missing data
- Use the predicted value as the imputed value
- We will **reinforce** the relationship between the predictor and the variable with missing data
  - This is not good if the relationship is not strong
  - Will change inference results

```r
data <- mice(data, seed = 1, method = "norm.predict")
complete(data)
```

#### Multiple Imputation

- Idea: Impute missing data multiple times to account for uncertainty
- Use `mice` package in R. Stands for **M**ultivariate **I**mputation by **C**hained **E**quations

- Steps:
  1. Create `m` copies of the dataset
  2. In each copy, impute missing data (different values)
  3. Carry out analysis on each dataset
  4. Combine models to one pooled model

```r
imp_data <- mice(data, seed = 1, m = 15, printFlag = FALSE)

complete(data, 3) # Get the third imputed dataset

# estimate OLS regression on each dataset
models <- with(imp_data, lm(y ~ x1 + x2))

# get third model
models$analyses[[3]]

# combine models
pooled_model <- pool(models)

summary(pooled_model) # remember to exp if using log model
```
