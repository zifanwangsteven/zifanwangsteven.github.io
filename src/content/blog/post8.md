---
title: "Entropy, Cross-Entropy & KL Divergence"
description: "Some closely related topics I constantly confuse up in language modeling"
pubDate: "Sep 28 2025"
heroImage: "/kl_div.webp"
tags: ["ML"]
---
### Basics
**Entropy** is a simple measurement of **uncertainty** in a probability distribution. For a probablity distribution A it takes the form of 
$$P(A) = -\sum_i P(A_i) \log P(A_i)$$
As is evidant from the form, the value is always greater than or equal to zero. As a sanity check, if a distribution is deterministic where $A=a$ with probability $1$, then we get an entropy of 1. Suppose we have a uniform distribution where each event has a probability of $\frac{1}{n}$ of happening, then $P(A) = \log {n}$ is monotonoically increasing in $n$ which aligns with our expectations - as $n$ increases, the distribution becomes more dispersed, so entropy as a chaos measure naturally increases.

**Cross-Entropy** is a related concept where instead of measuring the uncertainty of one probability distribution $A$ - it measures the relative difference between one distribution $A$ and another distribution $B$.
$$P(A, B) = -\sum_i P(A_i) \log{P(B_i)}$$
From the expression we can infer that cross-entropy is also a non-negative value. We also have the property that $P(A, B) \geq P(A)$. 
$$
\begin{align*}
-\sum_i p_i\log q_i + \sum_i p_i \log p_i &= \sum_i p_i \log \frac{p_i}{q_i} \\
& \geq \sum_i p_i (1 - \frac{q_i}{p_i}) \\
& =\sum_i p_i - \sum_i q_i \\
& = 0
\end{align*}
$$
Notice the last step follows from the fact that $p_i$ and $q_i$ are probability distributions that sums up to 1. In cases where $A = B$, cross entropy $P(A, B)$ attains its minimal value which is the base entropy $P(A)$.

This non-negative difference $P(A, B) - P(A)$ is what we refer to as the **KL-Divergence** of $P || Q$, which measures the **extra** surprise one gets from distribution $Q$ compared to the optimal model / true distribution $P$. In cases where we have a perfect model $Q = P$ then we have a KL-Divergence of 0.

One natural question that arises is why would we create 2 separate metrics for seemingly the same question - how different a distribution is from another? Well consider the following 2 tasks
1. The next word distribution in a legal document (P)
2. The next word distribution in creative poetry (Q)

Suppose we build 2 separate models, one for each task, with respective cross-entropy scores 2.5 and 4.5. Which model is better? It would be ambivalent because the base-entropy, or the relative uncertainty of the task (or can be understood as the difficulty of the task are different) are not clear. Suppose we are then given the information that the base entropy for $P, Q$ is respectively $2, 4.2$, then subtracting these from the cross-entropies, we get KL-divergences of $0.5, 0.3$, indicating model 2 is a better approximation of the best possible model. In machine learning, we are typically fixing a task $P$ as constant and optimizing model $Q$ only, in these cases, a loss function built with cross-entropy or KL-Divergence are often equivalent.
### A Note on Distance
One caveat on cross-entropy and KL-Divergence is that there are **NOT** distance metrics i.e. not symmetric. $\text{KL}(P || Q) \neq \text{KL}(Q||P)$ - the choice of which metric to use depends on **whose probability distribution** we want to weight the loss by. A concrete example would be the loss function for RLHF (Reinforcement Learning from Human Feedback) policy model,
$$
L = -\mathbb{E}[r(x,y)] + \beta D_\text{KL}(\pi_{\theta} || \pi_{\text{ref}})
$$
In human terms, we would like to maximize reward obtained from policy $\pi_{\theta}$, while deviating as little as possible from a reference policy $\pi_{\text{ref}}$. Why do we do $D_\text{KL}(\pi_{\theta} || \pi_{\text{ref}})$ instead of $D_\text{KL}(\pi_{\text{ref}} || \pi_{\theta})$? Since we want to penalize more frequent actions from $\pi_{\theta}$ that deviate from the reference distribution and not the other way around.
### In Language Modeling
In language modeling, we typically model the loss of a model as the cross-entropy between the true next word distribution and model next word distribution, since the actual next word distribution is **deterministic** (we know ahead of time exactly what each word in a sentence is), its distribution will just be a delta function with prob of 1 at the actual word and 0 everywhere else, so the cross-entropy loss at word $i$ simply becomes
$$
-\log (p_i | p_{<i}) = -\log(\text{softmax}(o_i)[x_{i+1}]) = -\log(\frac{\exp(o_i[x_{i+1}])}{\sum_{a=1}^{\text{vocab}}\exp (o_i[a])})
$$
For a loss function we can simply take the average over the entire length of the sentence and batch. We reporting model evaluation, we typically report another number **perplexity**, which is nothing but the exponential of the cross entropy loss.
$$
\text{perplexity} = \exp (-\frac{1}{N}\sum_i^N(\log(p_i|p_{<i})))
$$
This intuitively introduces a concept of "effective vocab size", which tells us, on average when making a next-word prediction, what is the equivalent vocab size if the modern were to uniformly sample from it?