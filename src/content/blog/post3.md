---
title: "Sharpe Ratios 123"
description: "The Many Fascinating Facts on the OG Metric"
pubDate: "Nov 9 2024"
heroImage: "/itemPreview.webp"
tags: ["stats", "finance"]
---
### Definition
The Sharpe Ratio (SR) is a measurement of expected (excess) return in units of asset volatility. If we assume iid stationary returns for the series $\{r_t\}$, then the Sharpe Ratio is just a scaled (by a factor of $\sqrt{n}$) t-statistic of the mean.
$$
\begin{split}
\hat{\mu} &= \frac{1}{n}\sum_{i=1}^nr_i \\
\hat{\sigma}^2 &= \frac{1}{n-1}\sum_{i=1}^n(r_i-\hat{\mu})^2 \\
\hat{\text{SR}} &= \frac{\hat{\mu}}{\hat{\sigma}} = \sqrt{n}t_{n-1}
\end{split}
$$
### Asymptotic Distributions
Assuming the same iid stationary returns, we can obtain the asymptotic distribution of the Sharpe Ratio estimator $\hat{\text{SR}}$ using **Central Limit Theorem** and **Delta Method**. First, we find the asymptotic distributions of the estimators $\hat{\mu}$ and $\hat{\sigma}^2$ using **Central Limit Theorem**:
$$
\begin{split}
\sqrt{n}(\hat{\mu} - \mu)&\xrightarrow{d} N(0, \sigma^2) \\
\sqrt{n}(\hat{\sigma}^2-\sigma^2) &\xrightarrow{d} N(0, \sigma^4) \\
\end{split}
$$
Then we recall the **Delta Method** for finding the asymptotic distribution for the function of an estimator(s) for which we already know the asymptotic distribution. Let $\{\hat{\mathbf{\theta}_n}\}$ be a series of $k\times1$ vectors and let $\mathbf{\theta}_0$ be a $k\times1$ vector. Suppose the series stasties the following:
$$
\sqrt{n}(\mathbf{\hat{\theta}}_n - \mathbf{\theta}_0) 
\xrightarrow{d} N(\mathbf{0}, V)
$$
where $V$ is the covariance matrix. Let $g: \mathbb{R}^k \rightarrow \mathbb{R}^l$ be a vector function and assume $g$ have continuous first order partial derivatives w.r.t. $\mathbf{\theta}$. Then we have the following result:
$$
\sqrt{n}(g(\mathbf{\hat{\theta}}_n)-g(\mathbf{\theta}_0)) \xrightarrow{d} N(\mathbf{0}, J_g(\theta_0)VJ_g(\theta_0)^T)
$$
Where $J_g(\theta)$ is the $l\times k$ Jacobian Matrix. Applied specifically to our case, we have the following
$$
\begin{split}
g(\mu, \sigma^2) &= \frac{\mu}{\sigma^2}\\
V &= \begin{bmatrix}\sigma^2 & 0 \\ 0 & \sigma^4\end{bmatrix} \\
J_g(\mu, \sigma^2) &= \begin{bmatrix}\frac{1}{\sigma} & -\frac{\mu}{2\sigma^3}\end{bmatrix} \\ \sqrt{n}(\hat{\text{SR}} - \text{SR})&\xrightarrow{d} N(0, 1+\frac{\mu^2}{2\sigma^2}) \sim N(0, 1+\frac{1}{2}\text{SR}^2)
\end{split} \\
$$
Therefore we can conclude the asymptotic standard error of the Sharpe Ratio estimator $SE(\hat{\text{SR}}) = \sqrt{\frac{1 + \text{SR}^2/2}{n}}$. This yields a somewhat intuitive but often overlooked result, the higher the estimated Sharpe Ratio of a particular strategy, the larger the estimation error.
### Time Aggregation
It's easy to mistake the Sharpe Ratio as unitless, but it is really dependent on the 2 statistics that make up the value: mean and volatility and their scaling properties w.r.t. to time.

For mean, the value scales linearly w.r.t. Suppose we have 2 means $\mu_1,  \mu_2$ for the same distribution corresponding to 2 different time intervals $t_1, t_2$ and we know that $t_1 = nt_2$, then we have
$$
\mu_1 = n\mu_2
$$
For iid series, the same identity also holds for variance, which would imply a square root scale for volatility:
$$
\begin{split}
\sigma_1^2 = n\sigma_2^2 \\
\sigma_1 = \sqrt{n}\sigma_2
\end{split}
$$
Taking the ratio of the two, we get
$$
\text{SR}_1 = \sqrt{n}\text{SR}_2
$$
A concrete example would be conversion from a **daily** Sharpe Ratio to an **annualized** Sharpe ratio, where the scaling factor would be $\sqrt{251}$. (In U.S., stock markets are open 251 days a year as of 2024)
### Application I: Loss Probabilities
Recall Cantelli's Inequality:
$$
P(X < \mu-\lambda) \leq \frac{\sigma^2}{\sigma^2 + \lambda^2}
$$
Let $X$ be the annualized return of a strategy and $\text{SR}$ be the annualized Sharpe Ratio. We want to find out the probability of losses in multiples of the standard deviation $L\sigma$ where $L$ is a scaling factor. We can then get:
$$
\begin{split}
P(X<-L\sigma) &= \frac{\sigma^2}{\sigma^2 + (L\sigma+\mu)^2} \\
&=\frac{1}{1+(L+\text{SR})^2}
\end{split}
$$
This holds for any distribution of returns. For the same level of loss, the higher the Sharpe Ratio, the lower the probability.
### Application II: Optimal Sharpe Allocation
Suppose we have an investible universte of $n$ assets / factors whose returns are a random vector $\mathbf{r} \isin \mathbb{R}^{n}$, and covariance matrix $\Omega_r \isin \mathbb{R}^{n \times n}$. We want to find an allocation $\mathbf{w} \isin \mathbb{R}^n$ that optimizes the portfolio variance subject a certain level of exposure to each asset / factor. Formally,
$$
\begin{split}
\min_{\mathbf{w}} \mathbf{w}^T\Omega_r\mathbf{w}\\
\text{s.t.} \quad \mathbf{b}^T\mathbf{w} = 1
\end{split}
$$
We can formulate this as an unconstraint optimization problem using the Lagrangian
$$
L(\mathbf{w}, \lambda) = \mathbf{w}^T\Omega_r\mathbf{w}-\lambda(1-\mathbf{b}^T\mathbf{w})
$$
Solving for the stationary points, we get 
$$
\mathbf{w}^* = \frac{\Omega_r^{-1}\mathbf{b}}{\mathbf{b}^T\Omega_r^{-1}\mathbf{b}}
$$
To get the optimal 