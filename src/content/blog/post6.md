---
title: "All 'bout that Decomp"
description: "Your Healthy Dose of Matrix Decompositions"
pubDate: "Aug 18 2025"
heroImage: "/matrix_decomp.jpg"
tags: ["linear algebra"]
---

Matrix decompositions are useful - either they are more efficient than solving a vanilla version of the problem at hand by utilizing particular properties of certain matrices or they can provide numerically stable solutions to otherwise unsolvable systems. In this post I enumerate some common matrix decompositions that underpin my everyday work.


### QR Decomposition

QR decomposition allows us to express an arbitrary matrix $A$ as a product of (1) a matrix with orthonormal columns $Q$ and (2) and upper triangular matrix $R$. Notice that it exists for any arbitrary matrix $A$, even when it's rank-deficient (in which case we'd have to allow for 0s on the diagonal terms in $R$, but no biggie). 

The most well-known procedure to compute a QR decomposition is through the *Gram-Schmidt Process*. Which turns a set of independent vectors into a set of orthonormal vectors. We accomplish this via repeatedly projecting new vectors on to existing set of orthogonalized / normalized vectors.

```python
import numpy as np
from np.linalg import norm

def gram_schmidt(vecs: list[np.ndarray]):
    outputs = []
    for v in vecs:
        u = v.copy().astype(float)
        for basis in outputs:
            u -= np.dot(u, basis) * basis
        norm_u = np.linalg.norm(u)
        if norm_u > 1e-12:
            outputs.append(u / norm_u)
    return outputs

def QR(A: np.ndarray):
    outputs = gram_schmidt([A[:, i] for i in range(A.shape[1])])
    Q = np.column_stack(outputs)
    R = Q.T @ A
    return Q, R
```

In reality, easier to simply use `np.linalg.qr`. 

QR decomposition can usually be used to solve linear systems $Ax=b$ without having to compute the inverse of $A$ (improved numerical stability). 
$$(QR)x=b \to Rx=Q^Tb$$
Since $R$ is upper triangular, we can now solve for the system easily via back substitution.

And yes, as you were thinking, it is also helpful for computing least squares solutions. Since $Q$ form a orthonormal basis for the column space of $A$, the normal equation becomes $Q^T(QRx-b)=0 \to Rx=Q^Tb$. Assuming $m > n$, we have square matrix $R$ of shape $n \times n$, an exact solution by back-substitution. 

### LU Decomposition (for Square Matrices)
An unappreciated pillar of matrix decompositions, an LU decomposition is easily produceable with every Gaussian elimination performed. Recall that we perform gaussian elimination (assuming no permutations needed) by repeatedly left-multiplying the target matrix $A$ with fundamental matrices $E_1....E_n$ until we reach a upper triangular matrix $U$ - at which stage we are ready to perform a back substitution. So, to spell this out a little bit, we have $E_nE_{n-1}...E_1A = U \to A = E_1^{-1}...E_n^{-1}U = LU$. And voila! As simple as that. Again this decomposition helps us solve for linear systems easily via
* $Ly = b$ forward substitution, fast
* $Ux = y$ backward substitution, fast

We compute the decomposition once and can use the same matrices for calculations of many different systems. Win!

### Eigendecomposition (for diagonalizable square matrices)
Probably the most studied decompostion in any intro linear algebra class $A = V\Lambda V^{-1}$. But interestingly a murky set of matrices that it applies to - square matrices, that's easy to understand, but what about diagonalization? To come up with a matrix $\Lambda$ - we need the eigenvectors of $A$ to span $\mathbb{R}^n$. Okay - but realistically, we often care about some special matrices where we **know for a fact** that are diagonalizable.

* Real symmetric matrices - diagonalizable with orthonormal eigenvectors. $A = Q\Lambda Q^T$
* Normal matrices (AA^T = A^TA) are unitarily diagonalizable.

### Cholesky Decomposition (for real, symmetric, positive definite matrices)

I know, seems like quite a few limiting conditions, but useful for one thing really - covariance matrices, which (un)fortunutely is at the center of every quant system. So starting a SPD matrix $A$, cholesky decomposes it into a lower triangular and upper trianguler matrix $A=LL^T$. I omit the proof here but note that SPD and Cholesky are an iff condition.