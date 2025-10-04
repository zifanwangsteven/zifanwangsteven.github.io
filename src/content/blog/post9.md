---
title: "Regression and Other Stories"
description: "An ongoing list of nuggets I find interesting from Andrew Gelman's magnum opus"
pubDate: "Oct 4 2025"
heroImage: "/itemPreview.webp"
tags: ["stats"]
---

### Question 4.3
```A multiple-choice test item has four options. Assume that a student taking this question either knows the answer or does a pure guess. A random sample of 100 students take the item, and 60% get it correct. Give an estimate and 95% confidence interval for the percentage in the population who know the answer.```

Let the proportion of students who get the answer right be $P$, then approximately $P \sim N(p, \frac{p(1-p)}{n})$. From the experiment we get $p = 0.6$ and $n=100$. Therefore $P \sim N(0.6, 0.05^2)$. Let the proportion of the students who know the answer be $P_1$, then $P = P_1 + \frac{1}{4}(1-P_1)$, which gives us $P_1 = \frac{4}{3}(P -\frac{1}{4})$ which gives us $P_1 \sim N(0.47, 0.07^2)$. Therefore the 95% confidence interval would be $0.47 \pm 1.96 * 0.07 = [0.31, 0.59]$

### Question 4.4
```Compare two options for a national opinion survey: (a) a simple random sample of 1000 Americans, or (b) a survey that oversamples Latinos, with 300 randomly sampled Latinos and 700 others randomly sampled from the non-Latino population. One of these options will give more accurate comparisons between Latinos and others; the other will give more accurate estimates for the total population average. Assume that the national population is 15% Latino, that the items of interest are yes/no questions with approximately equal proportions of each response, and (unrealistically) that the surveys have no problems with nonresponse.```

For estimating the difference of response between the 2 groups, the difference proportion follows the distribution of $N(p_1 - p_2, \frac{p_1 (1-p_1)}{n_1} + \frac{p_2 (1-p_2)}{n_2})$. In both cases $p_1=p_2 = 0.5$, then first case becomes $\sqrt{\frac{0.5^2}{300} + \frac{0.5^2}{700}} = 0.34$ and second case becomes $\sqrt{\frac{0.5^2}{150} + \frac{0.5^2}{850}} = 0.44$

For estimating the sample mean for the random sampling we get standard error of $\sqrt{\frac{0.5^2}{1000}} = 0.016$. For estimating the sample mean from the oversampled we get $\hat{\mu} = 0.15p_1 + 0.85p_2$ so the se would be $\sqrt{0.15 ^ 2\frac{0.5^2}{300} + 0.85^2\frac{0.5^2}{700}} = 0.21$
