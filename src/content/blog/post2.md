---
title: "Statistical Testing Not In Practice"
description: "The theoretical stuff you really need to memorize"
pubDate: "Oct 29 2024"
heroImage: "/itemPreview.webp"
tags: ["stats"]
---
The concept behind statistical testing should be intuitive to anyone, but for some reason the details for the particular setups for certain tests and jargons constantly elude me. This is an attempt to compile together such concepts in a way that made most sense to me. 

The basic idea is: you have some sort of an estimator for a value you are interested in, and, having studied basic statistics, you are not only interested in finding out the value of the estimator itself, but how convicted can you be in that value, in other words, you want to find out the **statistical significance** of that value. You then propose a hypothesis, often stylized as $H_0$, that says something about the the estimator, where as $H_a$ the alternative hypothesis would the exact complement. You then come up with a predefined procedure called a test $\delta$, that determines whether there is enough statistical evidence to reject your null hypothesis $H_0$ or otherwise using samples drawn. Formally:

### The Setup of a Test
#### Hypotheses
You draw $n$ samples $\mathbf{X} = (X_1, X_2, ..., X_n)$ i.i.d $P_{\theta}$ from a model parametrized $\theta$. Where the parameter value $\theta$ is unknown but should be in some known range $\Omega$. We consider the problem:
$$
H_0: \theta \isin \Omega_0 \quad \text{vesus} \quad H_a: \theta \isin \Omega_a
$$
where we have $\Omega_1 \cup \Omega_a = \Omega$ and $\Omega_1 \cap \Omega_a = \emptyset$. We typically call $H_0$ the **null hypothesis** and $H_1$ the **alternative hypothesis**. 
#### Testing and Test Statistic
Let $\mathbf{X}$ be a sample of size $n$ drawn from a distribution parametrized $\theta$, then a test statistic is a function of the sample only: $T = \varphi(\mathbf{X})$. Let $S$ denote all the n-dimensional distributions of $\mathbf{X}$, we can then partition the space into 2:

* $S_1$: the **critical region** that contains all $\mathbf{X}$ for which we reject $H_0$
* $S_0$: otherwise, fail to reject $H_0$  

We can then formalize a test $\delta$ as: reject $H_0$ if $T \isin R$ where $R$ is called the rejection region. Then we can also formalize the critical region $S_1$ as:
$$
S_1 =\{\mathbf{X} \ | \ \varphi(\mathbf{X})\isin R\}
$$
#### Testing Errors
Associated with the testing procedure $\delta$ are 2 types of errors:
* Type I error: rejecting $H_0$ when $H_0$ is True
* Type II error: failing to reject $H_0$ when $H_0$ is False
#### Power Functions
The power function for test using a test statistic $T$ is defined as
$$\pi(\theta | \delta) = P_\theta(T \isin R) \quad \text{for} \quad \theta \isin \Omega$$
That is, for any given parameter value $\theta$, what is the probability of the test rejecting the null hypothesis $H_0$? Given this expression, we can formalize the probability of committing errors as follows:
* $P(\text{Type I Error}) = \pi(\theta|\delta), \ \theta \isin \Omega_0$
* $P(\text{Type II Error}) = 1 - \pi(\theta|\delta), \ \theta \isin \Omega_1$

Therefore, for a test to be considered to be good, its power function should have the following properties. **Low** values for $\pi(\theta|\delta)$ when $\theta \isin \Omega_0$ and **high** values when $\theta \isin \Omega_1$. If the null hypothesis is a simple hypothesis, the ideal power function should just be 1 - delta function.

The easiest way to strike a balance between the 2 types of errors is to specify a priori, a **level of confidence** $\alpha$, s.t.
$$\pi(\theta | \delta) \leq \alpha \quad \text{for all} \quad \theta \in \Omega_0$$
Then among all tests $\delta$ that statisfy the above condition, we search for a test that minimizes the the Type II error.
### A Tour of Tests
#### Z-test
*Testing hypotheses about mean with known variance*   
Given $n$ i.i.d RVs distributed as $N(\mu, \sigma^2)$ with known variance $\sigma^2$, we define the Z-statistic as the following:
$$Z = \sqrt{n}\frac{\bar{X}-\mu}{\sigma} \sim N(0, 1)$$
We reject the null hypothesis when observing large absolute values of $Z$. Suppose we specify a confidence level $\alpha$, then we reject $H_0$ if 
$$\bar{x} \leq \mu_0 - \frac{\sigma}{\sqrt{n}}Z_{\frac{\alpha}{2}} \quad \text{or} \quad \bar{x} \geq \mu_0 + \frac{\sigma}{\sqrt{n}}Z_{\frac{\alpha}{2}}$$
Where $Z_{\frac{\alpha}{2}}$ is the value such that $P(Z\geq Z_{\frac{\alpha}{2}}) = 1-\frac{\alpha}{2}$.
#### t-test
*Testing hypotheses about mean with unkown variance*   
Same assumptions as the Z-test, only this time the variance isnot known ahead of time, so must be approximated with the sample variance
$$s_n^2 = \frac{1}{n-1}\sum_{i}^n(\hat{X}-X_i)^2$$
We define the t-statistic as
$$U_n = \sqrt{n}\frac{\hat{X}-\mu_0}{s_n}$$
Where we similarly reject $H_0$ if 
$$U_n \geq T_{n-1}^{-1}(1-\frac{\alpha}{2})$$
Where $T_{n-1}^{-1}$ is the inverse of the cdf of the t-distribution with $n-1$ degrees of freedom.
#### A note on p-values
Notice in the above examples the testing procedure involves setting an arbitrary confidence level $\alpha$ ahead of time. And for each new significance level we'll have to rerun the test. p-values are an alternative measure for testing which doens't depend on an artifically set significance level but rather computes the largest significance level the current sample would allow. In other words, p-values are the probabilities of observing a test statistic as extreme as the sample under $H_0$. A lower p-value means higher significance observed. To use this method, we simply compute the statistic, and find the probablity of observing a value as extreme as the value under the null hypothesis.
#### 2-sample t-test
*Comparing the mean of 2 normal distributions*   
Assuming the same variance $\sigma^2$ for 2 normal distributions $X$ and $Y$, we want to compare their mean $\mu_1$ and $\mu_2$. We compute the 2 sample t-statistic as:
$$U_{m,n} = \frac{\sqrt{m+n-2}(\bar{X}_m-\bar{Y}_n)}{\sqrt{(\frac{1}{m} + \frac{1}{n})}(S_X^2+S_Y^2)}$$
where 
$$S_X^2=\sum_i^m(\bar{X}-X_i)^2 \quad S_Y^2=\sum_i^m(\bar{Y}-Y_i)^2$$
We reject $H_0$ ($\mu_1 = \mu_2$) if 
$$|U_{m,n}|\geq T^{-1}_{m+n-2}(1-\frac{\alpha}{2})$$
#### F-test
*Comparing the varaince of 2 normal distributions*   
I promise we will move on to something more interesting after this. The F-statistic is simply the ratio between the 2 sample variances we want to test against, i.e.
$$V_{m,n}=\frac{S_m^2 / \sigma_1^2}{S_n^2 / \sigma_2^2}$$
where when $\sigma_1^2 = \sigma_2$, the test statistic is distributed as a F-distribution with $m-1, n-1$ degrees of freedom. Here we reject the null hypothesis $H_0 (\sigma_1 = \sigma_2)$ if 
$$V_{m,n} \geq G_{m-1, n-1}^{-1}(1-\frac{\alpha}{2}) \quad \text{or} \quad V_{m-1,n-1}^{-1}\leq G_{m-1,n-1}^{-1}(\frac{\alpha}{2})$$
Here notice that since F-distribution has support on the only the positive x-axis, we can't use the absolute value notation like above.
