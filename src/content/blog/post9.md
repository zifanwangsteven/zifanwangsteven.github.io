---
title: "Regression and Other Stories"
description: "An ongoing list of nuggets I find interesting from Andrew Gelman's magnum opus"
pubDate: "Oct 4 2025"
heroImage: "/itemPreview.webp"
tags: ["stats"]
---
## Chapter 4
### Question 4.3
```A multiple-choice test item has four options. Assume that a student taking this question either knows the answer or does a pure guess. A random sample of 100 students take the item, and 60% get it correct. Give an estimate and 95% confidence interval for the percentage in the population who know the answer.```

Let the proportion of students who get the answer right be $P$, then approximately $P \sim N(p, \frac{p(1-p)}{n})$. From the experiment we get $p = 0.6$ and $n=100$. Therefore $P \sim N(0.6, 0.05^2)$. Let the proportion of the students who know the answer be $P_1$, then $P = P_1 + \frac{1}{4}(1-P_1)$, which gives us $P_1 = \frac{4}{3}(P -\frac{1}{4})$ which gives us $P_1 \sim N(0.47, 0.07^2)$. Therefore the 95% confidence interval would be $0.47 \pm 1.96 * 0.07 = [0.31, 0.59]$

### Question 4.4
```Compare two options for a national opinion survey: (a) a simple random sample of 1000 Americans, or (b) a survey that oversamples Latinos, with 300 randomly sampled Latinos and 700 others randomly sampled from the non-Latino population. One of these options will give more accurate comparisons between Latinos and others; the other will give more accurate estimates for the total population average. Assume that the national population is 15% Latino, that the items of interest are yes/no questions with approximately equal proportions of each response, and (unrealistically) that the surveys have no problems with nonresponse.```

For estimating the difference of response between the 2 groups, the difference proportion follows the distribution of $N(p_1 - p_2, \frac{p_1 (1-p_1)}{n_1} + \frac{p_2 (1-p_2)}{n_2})$. In both cases $p_1=p_2 = 0.5$, then first case becomes $\sqrt{\frac{0.5^2}{300} + \frac{0.5^2}{700}} = 0.34$ and second case becomes $\sqrt{\frac{0.5^2}{150} + \frac{0.5^2}{850}} = 0.44$

For estimating the sample mean for the random sampling we get standard error of $\sqrt{\frac{0.5^2}{1000}} = 0.016$. For estimating the sample mean from the oversampled we get $\hat{\mu} = 0.15p_1 + 0.85p_2$ so the se would be $\sqrt{0.15 ^ 2\frac{0.5^2}{300} + 0.85^2\frac{0.5^2}{700}} = 0.21$

## Chapter 8
Some common paradigms of considering *inference* in regression modeling - that is, estimating the regression coefficients and accessing the uncertainty in those estimates.

### Least Squares
The most common technique. You want to minimize the residual sum of squares (RSS) in the model $y_i = a + b x_i + \epsilon_i$. 
$$\text{RSS} = \sum_i(y_i-\hat{a}-\hat{b} x_i)^2$$

#### Estimating the Residual Standard Deviation $\sigma$
A natural way to estimate $\sigma$ would be to simply take the standard deviation of the residuals, $\sqrt{\frac{1}{n} \sum_ir_i^2}$ - but this would actually underestimate the rse. As our estimation of the standard error already used up 2 degrees of freedom for the intercept and the slope parameter.

What are the possible remedies?

Solution 1 - simply subtract the lost degrees of freedom from the denominator. so your estimate becomes,

$$\hat{\sigma} = \sqrt{\frac{1}{n-k}\sum_i r_i^2}$$

Solution 2 - perform a **Leave-one-out** cross-validation, where a model is fitted $n$ times for the same data set, using $X_{-i}, y_{-i}$ (denoting the dataset excluding the i the observation), and compute the residual for that model fit as $r_i = y_i - \hat{a} - \hat{b} x_i$.

### Maximum Likelihood Estimation (MLE)
If the errors from the linear model are independent and normally distributed, so that $y_i \sim N(a+b x_i, \sigma^2)$, then the OLS estimates of $(a, b)$ are also MLE estimates. A small caveat is that the MLE estimate of standard error $\hat{\sigma} = \sqrt{\frac{1}{n}\sum_i(y_i - a - b x_i)^2}$ without the degree of freedom adjustment.

### Bayesian Inference
A good way to incorporate priors into the estimation process, plus the bonus of getting back an entire distribution instead of a point estimate of the fitted parameters.

### Influence of individual points in a fitted regression
The change in $\hat{b}$ when an incremental change of $1$ is applied to $x_i$.

$$ \Delta \hat{b} = \frac{x_i - \bar{x}}{\sum_i (x_i-\bar{x})^2}$$

A related concept is called the **leverage** of a regression point. Recall the hat matrix in regression $\hat{y} = Hy = X(X^TX)^{-1}X^Ty$, it being called the hat matrix because it literally takes the $y$ into the estimated space (hat). Expanding the matrix multiplication, we get

$$\hat{y_i} = h_{i1}y_1 + h_{i2}y_2 + ... + h_{ii}y_i + ... + h_{in}y_n$$

The leverage $h_{ii}$ denotes the influence the ith observation $y_i$ has on $\hat{y_i}$. It is 
* a value between 0 and 1
* proportional to the difference between the distance between $x_i$ and $\bar{x}$
* sums up to $p$, the number of parameters in the model

One can thus identify an extreme value in $x$ as 

$$h_{ii} \ge 3 \bar{h} = 3 \frac{p}{n}$$

## Chapter 9 
### Bayesian Regression with `bambi` and `pymc`

After performing a bayesian regression with `bambi` (underlying pymc) there are a couple values of interest

```python
model = bmb.Model(“weight ~ height“, data=earnings)
idata = model.fit(draws=2000, chainis=4) # inference data by arviz
```

- the *point prediction* $\hat{a} + \hat{b} x^{new}$
    - just use the mean params + new data point
        
- the *linear predictor with uncertainty* $a + bx^{new}$ propagating the inferential uncertainty in (a, b) - this represents the expected / average value of y for new data points with predictors $x^{new}$
    - sample from `model.predict(idata=idata, kind=”response_params”, inplace=False)`
    - data in `new_predictions.posterior.mu`
    - as $n \to \infty$ , the standard deviation approaches $0$.
        
- the *predictive distribution for a new observation* $a + bx^{new} + \epsilon$ - this represents uncertainty about a new observation $y$ with predictors $x^{new}$
    
    - sample from `model.predict(idata=idata, kind=”response”, inplace=False)`
    - data in `new_predictions.posterior_predictive`
    - as $n \to \infty$, the standard deviation approaches $\hat{\sigma}$


### Uniform, weakly informative and informative priors in regression
When the available data are strong relative to other readily available information, we typically don't concern ourselves with choosing a prior explicitly. 

* Uniform / flat often called *noninformative*. With a flat prior the maximum likelihood estimate is also the mode of the posterior distribution. You can do 

```python
import bambi as bmb

bmb.Model(..., prior=bmb.Prior("flat"))
```

* default prior distribution - weakly informative family of prior distribution. For a model of the form $y = a + b_1x_1 + b_2x_2 + .. + \epsilon$, each coefficient $b_i \sim N(0, 2.5\frac{\sigma_y}{\sigma_x})$ and intercept $a \sim N(\bar{y}, \sigma_y)$ and $\sigma \sim \text{HalfNormal}(\sigma_y)$.

## Chapter 10

### Ways of Interpreting Multiple Linear Regression
* *predictive interpretation* considers how the outcome variable differs on average, when comparing two groups of items that differ by 1 in the relevant predictor while being identical in all the other predictors. Under the linear model, the coefficient is the expected difference in y between these two items.
* *counterfactual interpretation* is expressed in terms of changes within individuals, rather than comparisons between individuals. Here, the coefficient is the expected change in y caused by adding 1 to the relevant predictor, while leaving all others the same. (this interpretation is useful for casual inferences)

### Interaction Terms
For example, a regression regressing child's IQ on mother's IQ and high school graduation status can have an interaction term that captures the difference in slope when a mother finishes and not finishes high school. One key indicator of potential existance of interaction terms if large coefficients for uninteracted terms.

### Indicator Variables
The usual spiele with perfect multicolinearity, i.e. when you have an intercept variable and $k$ binary dummy variables for each of the $k$ categories, the model can't assign weight to the baseline model (there are infinite ways to arrive at the same baseline result). So either drop the intercept or do $k-1$ indicators. The interpretation for the latter is that whichever indicator is left out would be the baseline model. 

### Some Notations
For a classical linear regression model

$$y_i = \beta_1X_{i1} + \beta_2X_{i2} + ... + \beta_{k}X_{ik} + \epsilon_i$$

where the errors $\epsilon_i$ have **independent normal distributions with mean 0 and standard deviation** $\sigma$. We can express this using a compact notation

$$\mathbb{y} \sim N(X\mathbb{\beta}, I\sigma^2)$$

Where $\mathbb{y} \in \mathbb{R}^{n \times 1}, X \in \mathbb{R}^{n \times k}, \beta \in \mathbb{R}^{k \times 1}$. We are well familiar with the OLS solution $\hat{\beta} = (X^TX)^{-1}X^Ty$. The estimate of the residual standard deviation is 

$$\hat{\sigma} = \sqrt{\frac{\text{RSS}}{n-k}}$$

Then from this we can also get variance-covariance matrix of each individual estimator

$$Var(\hat{\beta}) = \hat{\sigma}^2(X^TX)^{-1}$$

Where the square root of the diagonal elements are the standard errors for each predictor estimation (what is shown in the summary stats of a statsmodel fitted result). Now when  predicting for a new observation $\mathbb{x_i}$ using the fitted model, note now there's 2 source of variance for a new observation, i.e.:

* the estimation error, errors associated with the uncertainty around the regression coefficients, $\hat{\sigma}\sqrt{x_i^T(X^TX)^{-1}x_i}$ - in `get_prediction`, this corresponds to `se_mean`
* residual error, errors that are inherent with the data generating process, $\hat{\sigma}$ - corresponds to square root of `model.scale`

And so the standard error of the predicted value would be $\hat{\sigma}\sqrt{1 + x_i^T(X^TX)^{-1}x_i}$, accessible via `se_obs` from `get_prediction`.

### Nonidentifiable Parameters
In maximum likelihood, parameters are nonidentified if they can be changed without altering the likelihood. A model is said to be nonidentifiable if it contains parameters that can cannot be estimated uniquely. (or standard errors of near infinity). A typical cause if colinearity or near colinearity.

### Weighted Regressions

$$\hat{\beta} = (X^TW^{-1}X)^{-1}X^TW^{-1}y$$

Where $W$ is the matrix of weights. It follows that the standard error of the estimated coefficients are

$$Var(\hat{\beta}) = \hat{\sigma}^2(X^TW^{-1}X)^{-1}$$

In the case of heteroskescidicity where we are weighting by the inverse of the residual covariance matrix $\Omega_{\epsilon}^{-1}$, $\hat{\sigma}^2$ is just 1 since we standardized the inputs already.

Weighted regressions can be derived from a variety of models,

* using observed data to represent a larger population. For example, suppose out data come from a survey that oversamples older white women, and we are interested in estimating the population regression. Then we would assign to survery respondent a weight that is proprotional to the number of people of that type in the population.
* duplicate observations. 
* unequal variances - heteroskedescity.