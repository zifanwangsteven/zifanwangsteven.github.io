---
title: "All 'bout that Decomp"
description: "Your Healthy Dose of Matrix Decompositions"
pubDate: "Aug 18 2025"
heroImage: "/matrix_decomp.jpg"
tags: ["linear algebra"]
---

Matrix decompositions are useful - either they are more efficient than solving a vanilla version of the problem at hand by utilizing particular properties of certain matrices or they can provide numerically stable solutions to otherwise unsolvable systems. Here are some essentials:

### QR Decomposition

QR decomposition allows us to express an arbitrary matrix $A$ as a product of (1) a matrix with orthonormal columns $Q$ and (2) and upper triagular matrix $R$. Notice that it exists for any arbitrary matrix $A$, even when it's rank-deficient (in which case we'd have to allow for 0s on the diagonal terms in $R$, but no biggie). 

The most well-known procedure to compute a QR decomposition is through the *Gram-Schmidt Process*. Which turns a set of independent vectors into a set of orthonormal vectors. 