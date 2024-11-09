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
Therefore we can conclude the asymptotic standard error of the Sharpe Ratio estimator $SE(\hat{\text{SR}}) = \sqrt{\frac{1 + \text{SR}^2/2}{n}}$.