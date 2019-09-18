---
title: 'Convexity, Lipschitzness, and Smoothness'
date: 2018-06-28 10:44:50
tags: machine learning
categories: artificial intelligence
---

introduction from "understandingmachine learning theory algorithms"

<!--- more --->

##Convexity

### Convexity set

A set C in a vector space is convex if for any two vector $\mathbf{u},\mathbf{v}$ in $C$, the line segment between $\mathbf{u}$ and $\mathbf{v}$ is contained in $C$. That is, for any $a\in[0,1]$ we have that $\alpha \mathbf{u}+(1-\alpha)\mathbf{v} \in C$.

### Convexity function

Let $C$ be a convex set. A function $f:C \rightarrow \mathbb{R}$ is convex for every $\mathbf{u},\mathbf{v}\in C$ and $\alpha\in[0,1]$,
$$
f(\alpha \mathbf{u} + (1-\alpha)\mathbf{v}) \le \alpha f(\mathbf{u}) + (1-\alpha) f(\mathbf{v}).
$$
**Important property**

* Every local minimum of the function is also a global minimum. 
* For every $\mathbf{w}$ we can construct a tangent to $f$ at $\mathbf{w}$ that lies below $f$ everywhere, that is for convex differentiable function,

$$
\forall \mathbf{u}, f(\mathbf{u}) \ge f(\mathbf{w}) + \langle \nabla f(\mathbf{w}), \mathbf{u} - \mathbf{w} \rangle.
$$

**LEMMA**

Let $f:\mathbb{R} \rightarrow \mathbb{R}$ be a scalar twice diffrential function, and Let $f',f''$ be its first and second derivatives, respectively. Then, the following are equivalent:

1. $f$ is convex 
2. $f'$ is monotonically nondecreasing
3. $f''$ is nonnegative

**CLAIM**

* Assume that $f:\mathbb{R}^d \rightarrow \mathbb{R}$can be written as $f(\mathbf{w}) = g(\langle \mathbf{w}, \mathbf{x}) + y)$, for some $\mathbf{x} \in \mathbb{R}^d, y\in \mathbb{R}$. Then, **convexity of $g$ implies the convexity of $f$**.

  **Meaning**: The composition of a convex scalar function with a linear function yields a convex vector-valued function.

* For $i=1,…,r,let f_i : \mathbb{R}^d \rightarrow \mathbb{R}$ be a convex function. The following functions $g(x)$from $\mathbb{R}$ to $\mathbb{R}$ are also convex.

  $g(x)=max_{i\in[r]} f_i(x)$

  $g(x)=\sum_{i=1}^r w_i f_i(x)$, where for all $i,w_i \ge 0$.

  **Meaning**:The maximum of convex functions is convex and that a weighted sum of convex functions, with nonnegative weights, is also convex.

##Lipschitzness

**DEFINITION**: Let $C \subset \mathbb{R}^d$. A function $f:\mathbb{R}^d\rightarrow\mathbb{R}^k$ is $\rho$-Lipschitz over $C$ if for every $\mathbf{w}_1,\mathbf{w}_2 \in C$ we have $\|f(\mathbb{w_1})-f(\mathbb{w_2}) \| \le \rho\|\mathbf{w_1}-\mathbf{w_2}\|$.

MEANING: if $f$ is everywhere bounded (in absolute value) by $\rho$, then the function is $\rho$-Lipschitz.

**CLAIM**: Let $f(x) = g_1(g_2(\mathbf{x}))$, where $g_1$ is $\rho_1$-Lipschitz and $g_2$ is $\rho_2$-Lipschitz. Then, $f \space is \space(\rho_1\rho_2)-Lipschitz$. In particular, if $g_2$ is the linear function, $g_2(\mathbf{x})=\langle \mathbf{v},\mathbf{x}\rangle+b$, for some $\mathbf{v} \in \mathbb{R}^d, b\in \mathbb{R},then \space f \space (\rho_1\|\mathbf{v}\|)-Lipschitz$.

## Smoothness

**DEFINITION**: A differentiable function $f$: $\mathbb{R}^d \rightarrow \mathbb{R}$ is $\beta$-smooth if its gradient is $\beta$-Lipschitz; namely, for all $\mathbf{v},\mathbf{w}$ we have $\|\nabla f(\mathbf{v}) - \nabla f(\mathbf{w}) \| \le \beta\|\mathbf{v} - \mathbf{w} \|$.

**CLAIM**: Let $f(\mathbf{w}) = g(\langle \mathbf{w} , \mathbf{x} \rangle + b), where \space g:\mathbb{R} \rightarrow \mathbb{R}​$ is a $\beta​$-smooth function, $\mathbf{x} \in \mathbb{R}^d​$, and $b \in \mathbb{R}​$. Then, $f \space is  \space (\beta \|\mathbf{x}\|)​$-smooth.



实际上，凸性要求一个函数在局部增长得比线性函数要快，而强凸性则要求其增长得比一个二次函数（二次项系数为m/2）快。