---
title: "Portfolio Optimization with Mosek"
description: "If you missed your convex optimization class like I did"
pubDate: "Sep 15 2025"
heroImage: "/itemPreview.webp"
tags: ["portfolio optimization"]
---
[Mosek](https://docs.mosek.com/portfolio-cookbook/preface.html) is the defacto software for large scale optimization. It includes on its website a banger of a tutorial for portfolio optimization and covex (conic) optimiztion in general, and this is a close study of its materials.

### Conic Optimization
The general form of conic optimization goes as follows,
$$
\begin{align*}
\max \quad c^Tx \\
\text{subject to} \quad Ax + b \in \mathcal{K}
\end{align*}
$$
where $\mathcal{K}$ is a one of the following basic type of cones:
* Linear cones, mostly for Linear Optimization problems (LO) $\mathbb{R}^{+}, \mathbb{R}^{-}, \{0\}$ 
* Quadratic Cones $\mathcal{Q}^n = \{(x_1, x_2, ..., x_n)|x_1 \geq \sqrt{x_2^2 + x_3^2 + ... + x_n^2}\}$
* Rotated Quadratic Cones $\mathcal{Q}_r^n = \{(x_1, x_2, ..., x_n) | 2x_1x_2 \geq x_3^2 + ... + x_n^2, x_1, x_2 \geq 0\}$

### Basic Constraints
#### Maximum Function
We want to model the maximum constraint that $\max(x_1, x_2, ..., x_n) \leq c$. We can introduce an auxiliry variable $t$ which is a standin for the maximum of the vector and set up the following constraints:

$$
\begin{align*}
x_1 \leq t \\
x_2 \leq t \\
... \\
x_n \leq t \\
t \leq c
\end{align*}
$$

#### Positive and Negative Part of a Variable
Sometimes we wish to decompose the positive and negative part of a variable. Where $x^+ = \max(x, 0)$ and $x^- = \max(-x, 0)$ 
Where we model its behavior as 
$$
\begin{align*}
x = x^+-x^-\\
x^= \geq 0 \\
x^- \geq 0
\end{align*}
$$

#### Absolute Value
$|x| \leq c \to -c \leq x \leq c$ Or we can formulate it as a quadratic cone, i.e. $(c, x) \in \mathcal{Q}^2$.

#### 1-Norm (Manhattan Norm)
$ ||\mathbb{x}||_1 = c \to -\mathbb{z} \leq \mathbb{x} \leq \mathbb{z}, \mathbb{1}^T\mathbb{z} = c$
#### 2-Norm (Euclidean Norm)
$||\mathbb{x}||_2 \leq c \to (c, \mathbb{x}) \in \mathcal{Q}^{n+1} \quad $
#### Squared Euclidean Norm
$||\mathbb{x}||_2^2 \leq c \to (c, \frac{1}{2}, \mathbb{x}) \in \mathcal{Q}^{n+2}_r$

#### Quadratic Form
$\mathbb{x}^TQ\mathbb{x} \leq c$ where $Q$ is a SPD matrix. (Your covariance matrix). Now go with your [favorite decomposition](https://zifanwangsteven.github.io/blog/all-bout-that-decomp/) to decompose $Q = GG^T$. Then your problem becomes,
$$
\begin{align*}
\mathbb{x}^TGG^Tx \leq c \\
||G^T\mathbb{x}||^2 \leq c
\end{align*}
$$
Which can be modeled using either a 
* Quadratic Cone as $(\sqrt{c}, G^T\mathbb{x}) \in \mathcal{Q}^{n+1}$
* Rotated Quadratic Cone as $(c, \frac{1}{2}, G^T\mathbb{x}) \in \mathcal{Q}^{n+2}$