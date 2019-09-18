---
title: robust optimization in ML
date: 2018-04-25 13:25:41
tags: machine learning
categories: artificial intelligence
---
In real large application where the training data may have noise and outliers, general ML algorithm may be very sensitive to noise and outliers. algorithm with robust can overcome this situation. 

<!--more-->

### 1. Robust Factorization Machines for User Response Prediction

####contributions

the work aims at characterizing the environment-induced un- certainty in the user signals and reformulating the FM and FFM objective functions to be immune against data fluctuations. 
main contributions:

* employ robust optimization principles to model the noise arising in online advertising signals as boundes box-type interval uncertainty sets.
* propose two novel algorithms: robust factorization machine (RFM) and robust field-aware factorization machine(RFFM).
* using parallel stochastic gradient decent.
* evaluate on three publicly available response prediction datasets from criteo and avazu. for classification robust version takes a slight performance hit in the standard setting but significantly outperforms the non-robust counterparts when subjected to noise.
* provide guidelines for selection of uncertainly sets.
* a general model


#### model

we associate uncertainty vector $\mu\in \mathbb{R}^d\space s.t. \space |\mu_j|\leq\eta_j,\forall j\in \{1,…,d\}$ for characterizing noise in linear interactions and matrix $\sum\in\mathbb{R}^{d\times d} \space s.t. \space \sum_{j,k}=\sigma_j\sigma_k,|\sigma_j|\le \rho_j,\forall j\in\{1,…,d\}$ for capturing noise induced by pairwise interaction terms. Define robust factorization machines as $\phi _{RFM}$：
$$
\phi_{RFM}(\mathbf{x},\mathbf{w},\mathbf{V},\mathbf{x},\mathbf{\Sigma})=w_0+\sum_{j=1}^dw_j(x_j+\mu_j)+\frac{1}{2}\sum_{j=1}^{d}\sum_{k=j}^{d}\langle \mathbf{v}_j,\mathbf{v}_k\rangle(x_jx_k+\Sigma_{j,k})
$$
after expanding:
$$
\phi_{RFM}(\mathbf{x},\mathbf{w},\mathbf{V},\mathbf{x},\mathbf{\Sigma})=w_0+\sum_{j=1}^{d}w_jx_j+\sum_{j=1}^{d}w_j\mu_j+\frac{1}{2}\sum_{f=1}^{p}(\sum_{j=1}^{d}v_{j,f}x_j)^2+
\\
\frac{1}{2}\sum_{f=1}^{p}(\sum_{j=1}^{d}v_{j,f}\sigma_j)^2+
+\frac{1}{2}\sum_{f=1}^{p}\sum_{j=1}^{d}v_{j,f}^2x_j^2+\frac{1}{2}\sum_{f=1}^{p}\sum_{j=1}^{d}v_{j,f}^2\sigma_j^2
$$

#### how to robust

change the model, use uncertainty sets.

### 2. Robust Top-k Multiclass SVM for Visual Category Recognition

#### contributions

The major limitation of the existing top-k extension is its sensitivity to abnormal observations, i.e., outliers, after all its loss is still convex. to overcome this problem, we proposed a robust multiclass SVM formulation that directly aims at minimizing a weighted and truncated combination of the ordered prediction score. the algorithm includes many previous multiclass SVMs as special cases.

#### model

Given a training sample $(x_i,y_i),i=1,…,n$, where $x_i\in\mathcal{X}\subseteq \mathbb{R}^d$ and $y_i \in \mathcal{Y}:=\{1,…,c\}$, loss function for the multiclass SVM:
$$
\ell_i=\ell_i(W):=\max_{j\in\mathcal{Y}} \{\mathbb{I}_{j\neq y_i}+\langle\mathbf{w}_j,\mathbf{x}_i\rangle-\langle\mathbf{w}_{y_i},\mathbf{x}_i\rangle\}
$$


We use $a_{[k]}$ to denote the k-th largest entry of the vector a, and the matrix $W_{\backslash y}\in \mathbb{R}^{(c-1)\times d}$ remove the y-th row of $W$. then loss function for top-k multiclass SVM:
$$
\ell_i(W):=\max \{0,(W_{\backslash y_i}\mathbf{x}_i)_{[k]}+1-\langle\mathbf{w}_{y_i},\mathbf{x}_i\rangle\}
\\
=\max \{0, \left ( (W-\mathbf{1}\mathbf{w}_{y_i}^{T})\mathbf{x}_i+1-\mathbf{e}_{y_i}\right )_{[k]}\}
$$


where $\mathbf{e}_j$ is the j-th canonical basis vector, and $\mathbf{1}$ is the all ones vector.

the major limitation of the original nonconvex top-k loss equation is their sensitivity to abnormal observations, i.e. outliers. This is mostly due to the unboundedness of the corresponding losses. To overcome this limitation and make it more robust, change the loss function:
$$
\ell_i(W):=\min \{ \max \{0,\sum_{j=1}^c \alpha_j s_{[j]}^i\},\tau \}
$$
where we use the following abbreviations for the scores
$$
s^i=(W-\mathbf{1}\mathbf{w}_{y_i}^T)\mathbf{x}_i+1-\mathbf{e}_{y_i}
$$
and $\alpha \in \mathbb{R}^k$ is an arbitrary weight that choose to combine the ordered scores. $\tau > 0$ is a hyperparameter that we use to cap the loss for any training pair.

#### how to robust

change the loss function by use capped form, make algorithm can "give up" focusing any training pair that incurs an excessively large loss, namely, outliers. 

