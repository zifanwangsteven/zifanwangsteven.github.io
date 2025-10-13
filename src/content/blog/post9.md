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

