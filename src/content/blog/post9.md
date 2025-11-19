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

## Chapter 11
### Assumptions of Linear Regression
* Linearity and Additivity: $y = \beta_0 + \beta_1 x_1 +... + \beta_k x_k$
* Independence of errors: observations should be independent of each other $\text{Cov}(\epsilon_i, \epsilon_j) = 0$ if $i \neq j$. This assumption is often violated in time series, spatial, and multi-level settings. One can use the [Durbin-Watson Statistic](https://en.wikipedia.org/wiki/Durbin%E2%80%93Watson_statistic) to check for autocorrelation in the residuals.
* Equal variance of errors (Homoskedasticity). $\text{Var}(\epsilon_i) = \sigma^2$ for all $i$. One can use a residual plot (residuals plotted against predicted values), and if you see a funnel shape, then it is problematic. This does not impact the betas, but only there standard errors. Possible remedies, use robust covariance estimators (HC3s) or WLS (like introduced in previous chapter). Most of the time not a huge issue tho.
* No perfect multicollinearity. Use VIF test (variance inflation factors). You should remove the highly correlated predictors or use regularized regression (Lasso / Ridge).
* Exogeneity $X$ should be *exogenous* to $Y$, meaning the predictor should be decided outside the system, and is not influenced by $Y$ (reverse causality) or anything that also affects $Y$ (omitted variable bias). $\mathbb{E}(\epsilon | X) = 0$. OLS can be inconsistent and biased and no true causal effect.
* Normality of errors. Typically examine with a Q-Q plot. - inaccurate inference.

### R Squared
$R^2 = 1 - \frac{RSS}{TSS} \in [0, 1]$ measures relatively predictive power from just predicting the main. Or "explained variance", how much of the variance in the observed variable is captured by the model. This is defined for a typical least-squared based point estimate. But for a bayesian model with a strong prior, the above formula may go out of bounds. We can however, define an alternative metric:

$$R^2* = \frac{\text{var}_{\text{fit}}}{\text{var}_{\text{fit}} + \text{var}_{\text{res}}}$$

Where $\text{var}_{\text{fit}}$ is the variance of the predictored values $Var(\hat{y_i})$, and $\text{var}_{\text{res}}$ is the residual variance $\sigma^2$.

### A note on AICs
$\text{AIC} = 2k-2\log L(y|X,\beta)$ Penalized no of parameters to prevent overfitting, genenerally prefer a model with a smaller AIC all else equal.

### A note on coverage
To evaluate the effectiveness of a confidence interval estimate with a simulation-based approach. We can repeat the experiment (the data generating process + regression fit) for $n$ times and observe the amount of times the true parameters fall within the confidence interval. For example for +- 1 se, we expect the true value to fall within the confidence interval ~68% of the times. If normality of errors asumption is not met, we'll see an deterioration of the confidence interval, as we are undersestimating the heavy tails of the standard errors - thus obtaining a narrower than usual confidence interval.

## Chapter 12

### Linear Transformations
Linear scaling / transformation of predictors / regression coefficients (unsurprisingly) don't change the predictive power of the regression model, but might lead to more interpretable regression results. For example the intercept of a raw regression for children's IQ value of mother's IQ may not be super useful, as it is the predicted IQ had the mother had an IQ of 0. If however, we center the mother IQ predictor at its mean, the intercept gives a much more intuitive interpretation of the average IQ of children when their mother is at the average IQ level. We can take this one step further and use zscores as predictors, standardizing further by normalizing the predictor's standard deviation. We should use an **externally** determined distribution when the sample size is small.

### Correlation as the Regression Parameter
If both the target and the predictors are standardized to have mean 0 and std 1, then the regression parameter $b$ for $y \sim a + bx$ would simply be,

$$b = \frac{\sigma_y}{\sigma_x}\rho_{xy}$$

In this particular case since both $\sigma_y$ and $\sigma_x$ are 1, the regression parameter is simply the correlation between the 2. This also tells us that in general, if the regression parameter has an absolute value of greater than 1, the variation in $y$ must be greater than that in $x$.

### Logarithmic Transformations
The logarithmic transformations takes a linear model into a multiplicative model. Log-log transformations define percentage change relationships. (Also called elasticity). Which goes like, for an $w\%$ change in $x$, what is the average $\%$ change in $y$? For a continuous function $f$ this is defined as

$$Q = \frac{d\log f}{d\log x} = \frac{x}{f}\frac{df}{dx}$$

### General Principles
* include all input variables, that for subtantive reasons, might be expected to be important in predicting the outcome.
* can combine multiple predictors to get a total score
* for inputs that have large effects, consider interaction terms.

## Chapter 13

### Logistic Functions and Sigmoid
The logistic function is 

$$\text{logit}(x) =\log\frac{x}{1-x}$$

where $x \in (0, 1)$, $\text{logit}(x) \in (-\infty, \infty)$. Here $x$ is understood as the probability of an event, then $\frac{x}{1-x}$ is the odds of such event, and the log odds is called the logit.

The inverse of this function would be the famed sigmoid function:

$$\text{logit}^{-1}(x) = \text{sigmoid}(x) = \frac{e^x}{1+e^x}$$

Here $x\in(-\infty, \infty)$, $\text{sigmoid(x)} \in (0, 1)$. Now sigmoid function is monotonically increasing and bounded between $(0, 1)$, therefore it is also a CDF function for what is called the **logistic distribution**. Taking the simple derivative of the above PDF of the standard logistic disitribution:

$$p(x) = \frac{e^{-x}}{(1+e^{-x})^2}$$

More generally, we allow for a location $\mu$ and a scale $s$ parameter, where 

$$\text{sigmoid}(x; \mu, s) = \frac{1}{1 + e^{-\frac{x-\mu}{s}}}$$

### Logistic Regression
In logistic regression, we are predicting for the probabity of a binary outcome,

$$P(y_i = 1) = \text{logit}^{-1}(X_i\beta)$$

The logistic curve is the steepest at its center. For a simple regression of $\text{logit}^{-1}(x) = \alpha + \beta x$ at $x=0$ the slope becomes $\frac{\beta}{4}$ which is the maximum difference in $P(y=1)$ for a unit change in $\beta$. When the predicted probabilities are not far from $0.5$, $\beta/4$ is a good top line estimate.

### Interpretations of Logistic Regression
We can interpret logistic regression as a nonlinear modeling of latent (unobserved) continuous variables variables.

$$
y_i = \begin{cases} 
1 & \text{if} & z_i > 0 \\
0 & \text{if} & z_i < 0
\end{cases} \\
z_i = X_i\beta + \epsilon_i
$$

Where $\epsilon_i \sim \text{Logistic}(0, 1)$. The logistic regression is also Bell-shaped and can be approximated by a Normal with $N(0, 1.6^2)$. Why not let $\sigma$ be a free variable and estimate it jointly with $\beta$? Well the problem is, the binary data we have on the response variable **does not** identify a scale parameter **uniquely**.

$$
z_1 = X\beta + \epsilon\\
z_2 = X(10 \beta) + (10 \epsilon)
$$

The sign of $z_1$ and $z_2$ are the same, and thesefore have the same implications for the set of binary observations. We solve this by fixing the scale variable to 1 and solving only for $\beta$. This corresponds to roughly $\sigma = 1.6$ for a normal approximation.

### MLE
For logistic regressions, we have no close form solutions available, so typically start with MLE, and add regularization / bayesian prior as needed. The likelihood is 

$$
L(y; \beta, X) = \prod_i (\text{logit}^{-1}(X_i\beta)^{y_i}(1-\text{logit}^{-1}(X_i\beta))^{1-y_i})
$$

* Running a logstic regression with a single intercept is equivalent to computing the proportion of an event.
* Running a logistic regression with a single predictor (without intercept) is equivalent to comparing proportions in two populations.

## Chapter 14

Graphing residual plots for binary responses are particularly helpful most of the time, one remedy is to graph average residuals bucketed by predictors.

Average predictive comparisons on the probability scale are more useful than the direct comparisons of parameters, it is important to use the actual expected difference in probability than the difference in probability evaluated at the mid-point of all other predictors.

$$\delta = \frac{\mathbb{E}[y|u^{hi}, v] - \mathbb{E}[y|u^{lo}, v]}{u^{hi}-u^{lo}}$$

When there is **separation** - i.e. the targets are perfectly separated by a set of predictors, the regression coefficients will approach positive / negative infinity - such that only the sign of the predictor matters, and not the value. We can mitigate this by using a stronger prior in our inference.

## Chapter 15

### GLM: Definition and Framework
A framework for statistical analysis that includes linear and logistic regression and special cases. It involves:
* a vector of outcome data $y = (y_1, ..., y_n)$
* a matrix of predictors $X$ and a vector of coefficients $\beta$, forming a linear predictor vector $X\beta$.
* A *link function* $g$, yielding a vector of transformed data $\hat{y}$ = $g^{-1}(X\beta)$ that are used to model the data
* a data distribution $p(y|\hat{y})$
* possibly other parameters, such as variances, overdispersions and cutpoints, involved in the predictors, link function and data distribution.

The options in a generalized linear model are the transformations $g$ and the distribution $p$.

* for linear regression $g(u) = u$, $p(y|\hat{y}) = N(X\beta, \sigma^2)$
* for logistic regression $g^{-1}(u) = \text{logit}^{-1}(u)$, $p(y|\hat{y}) = \hat{y}$ (since here $\hat{y}$ is simply the probabilty of a positive sample)
* Poisson and negative binomial models - each data point $y_i$ can equal $0, 1, 2 ...$, here $g(u) = \exp(u)$
* logistic-binomial model - same as logistic, but each instance contains multiple trails and is therefore a binomial data distribution.
* probit models - same as logistic regression but with the logit function replaced by the normal cumulative distribution, or equivalently with the normal distribution instead of logistic in the latent-data errors.
* multinomial logit and probit models - extensions of logistic and probit regressions for categorical data with more than two options, for example survery responses such as Strongly Agree.... (ordered) or Straberry, Vanilla (unordered).
* robust regression models - replace the usual normal or logistic models by other distributions.

### Poisson and negative binomial regression
In count-data regressions, each unit $i$ corresponds to a setting (spatial location or time period) in which $y_i$ events are observed. For example $y_i$ could be the number of traffic accidents at intersection $i$ in a given year.

As with linear and logistic regression, the variation in $y$ can be explained with linear predictors $X$. The simplest model imaginable is the **Poisson Model**, 

$$y_i \sim \text{Poisson}(e^{X_i\beta})$$

The possion distribution has its own internal scale of distribution $\sqrt{E[y]}$, this may lead to the problem of **overdispersion / underdispersion**, i.e. the data show data show more or less variation than the fitted probability model.

To generalize Poisson model to allow for overdispersion, we make use of the negative binomial model, which includes an additional "reciprical dispersion" parameter $phi$ so that $sd(x) = \sqrt{E[y] + E[y]^2/\phi}$ when $\phi \to \infty$, negative binomial becomes a poisson. Both binomial and poisson models have the natural applications in counting models, their key differences lie in for binomial models there is a natural limit -> can be interpreted as the number of successes out of $n_i$ trails. Whereas if each data point $y_i$ does not have a natural limit, then Poisson / Neg Bin should be preferred. When the limit is far from the mean tho, Poisson can be used to approximate binomial.

Offsets - for counting models there is naturally a concept of *exposure*, the length of the measurement period, the length of the treatment applied extra. It wouldn't make sense for us to regress directly on the raw counts from these varying units of measurement, we instead measure the unit rate by dividing by the exposure $u$. The log of that exposure is called the offset of this regression. It is really not that different from the intercept, but only that instead of estimating an extra regression coef on it, we fix it to a constant coef of 1.

$$
\log\frac{\lambda_i}{u_i} = X_i\beta \\
\lambda_i = u_i \exp(X_i\beta)
$$

### Logistic-Binomial Model
Essentially an aggregation of simple logistic model, instead of modeling each binary outcome one by one, we group by predictor values and get $y_i$ successes out of $n_i$ trials in each step and repeat the experiment for $N$ times. We can easily model this with `statsmodels` as 

```python
import statsmodels.formula.api as smf
import statsmodels.api as sm

smf.glm("success + fail ~ X", data, family=sm.Binomial())
```

Like with Poisson distribution, the $\sigma$ in logistic / binomial distribution is preset to an arbitrary value of 1, and therefore we easily get a problem of overdispersion. We can verify this by computing the theoretical distribution of the residuals.

$$
z_i = \frac{z_i - n_i\hat{p_i}}{\sqrt{n_i \hat{p_i}(1-\hat{p_i})}}
$$

Where $\hat{p_i} = \text{logit}^{-1}({X_i\hat{\beta}})$. We can then formally compute the overdispersion parameter $\frac{1}{N-k}\sum_{i=1}^Nz_i^2$ and compare it with the $\chi^2_{N-k}$ distribution.

### Probit Regression
Similar to logistic regression, only the inverse logit function is replaced with a normal CDF $\Phi$. In the latent formulation,

$$
y_i=\begin{cases}
1, \text{if} \quad z_i > 0\\
0, \text{if} \quad z_i < 0
\end{cases} \\
z_i = X_i\beta + \epsilon_i\\
\epsilon_i \sim N(0, 1)
$$

Here we fix the std param $\sigma=1$ following a similar rationale of nonidentifiability as in logistic regression.

### Ordered and Unordered Categorial Regression
Consider a categorial outcome $y$ that can take on values $1, 2, ... K$, we can express the outcome as a series of logistic regressions:

$$
P(y>1) = \text{logit}^{-1}(X\beta) \\ 
P(y>2) = \text{logit}^{-1}(X\beta-c_2) \\ 
... \\
P(y>K-1) = \text{logit}^{-1}(X\beta-c_{K-1})
$$

These series of *cut points* $0 = c_1 < c_2 < c_3 < ... < c_{K-1}$. are constrained to increase as the probabilities are strictly decreasing. For a regression with $K$ categories, we need $K-2$ extra parameters in $c_K$ in addition to $\beta$.

