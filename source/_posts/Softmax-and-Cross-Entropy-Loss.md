---
layout: article
title: Softmax and Cross Entropy Loss
date: 2018-04-11 11:01:08
tags: softmax
categories: [artificial intelligence]
---

introduction and derivative of Softmax and Cross Entropy Loss.

<!--more-->

### Softmax function

the softmax function takes N-dimensional vector of real numbers and transforms it into a vector of real number in range of (0,1) which means the probability of each classification.
$$
p_i=\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}
$$
it is easy to know  $\sum_{k=1}^Np_i=1$ .

the code of softmax function in python:

```python
def softmax(X):
    exps = np.exp(X)
    return exps / np.sum(exps)
```

sometimes the exp operation can overshot the limit of the floating point numbers in python, so we need to normalize the value by multiply a constant C.
$$
p_i=\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}
=\frac{Ce^{a_i}}{C\sum_{k=1}^Ne^{a_k}}
=\frac{e^{a_i+log(C)}}{\sum_{k=1}^Ne^{a_k+log(C)}}
$$
Generally, we max $log(C)=-max(a)$, so we need make a little change of our code:

```python
def softmax(X):
    exps = np.exp(X-np.max(X))
    return exps / np.sum(exps)
```

###Cross Entropy Loss

the defination of cross entropy loss is:
$$
L=-\sum_iy_i log(p_i)
$$
note: 

$p_i$ means the softmax value of  i-th neuron  in fully connected layer;

if the true label of input sample is $j$,  then $y_{i=j}=1;y_{i\neq j}=0$.

the code of loss function in python:

```python
def cross_entropy(X, y):
    """
    X is the output of fully connected layer(N-dimensional vector)
    y is true label(integer)
    """
    p = softmax(X)
    loss = -np.log(p[y])
    return loss
```

### Backward propagation

we need to compute the derivative of cross entropy loss:
$$
\frac{\partial L}{\partial a_i}=\frac{\partial L}{\partial p_j}\frac{\partial p_j}{\partial a_i}
$$
it is easy to compute $\frac{\partial L}{\partial p_i}$
$$
\frac{\partial L}{\partial p_i}=-\sum_iy_i\frac{1}{p_i}\tag1
$$
then compute $\frac{\partial p_j}{\partial a_i}$, we need to divide it into two situations：

1. if $i=j$

$$
\begin{align*}
\frac{\partial p_j}{\partial a_i}&=\frac{\partial p_i}{\partial a_i}=\frac{\partial{\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}}}{\partial a_i}=\frac{\sum_{k=1}^{N}e^{a_i}e^{a_k}-(e^{a_i})^2}{(\sum_{k=1}^Ne^{a_k})^2}\\
&=\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}\left(1-\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}\right)\\
&=p_i(1-p_i)
\end{align*}
$$



2. If $i\neq j$

$$
\frac{\partial p_j}{\partial a_i}=\frac{\partial{\frac{e^{a_j}}{\sum_{k=1}^Ne^{a_k}}}}{\partial a_i}=
-\frac{e^{a_j}}{\sum_{k=1}^Ne^{a_k}}\frac{e^{a_i}}{\sum_{k=1}^Ne^{a_k}}=-p_jp_i\tag2
$$

combine the result of (1) and (2)
$$
\begin{align*}
\frac{\partial L}{\partial a_i}& = \left(-\sum_iy_i\frac{1}{p_i}\right)\frac{\partial p_i}{\partial a_i}\\
&=-\frac{y_i}{p_i}p_i(1-p_i)+\sum_{i\neq j}\frac{y_i}{p_i}p_ip_j\\
&=-y_i+y_ip_i+\sum_{i\neq j}y_ip_i\\
&=-y_i+p_i\sum_iy_i\\
&=p_i-y_i
\end{align*}
$$
code in python

```python
def delta_entropy(X,y):
    """
    X is the output of fully connected layer(N-dimensional vector)
    y is true labels(integer)
    """
    grad = softmax(X)
    grad[y] -= 1
    return grad
```









