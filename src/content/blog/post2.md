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
