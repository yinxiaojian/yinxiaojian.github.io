---
layout: article
title: meta learning notes
date: 2018-10-21 22:06:17
tags: [machine learning, deep learning]
categories: [paper notes]
---

### Definition

Meta-learning is equal to learning to learn. In practice, very closely related to multi-task
learning.

In the typical machine learning setting, we are interested in a dataset $D$ and usually split $D$ so that we optimize parameters $\theta$ on a training set $D_{train}$ and evaluate its generalization on the test set $D_{test}$. In meta-learning, however, we are dealing with meta-sets $\mathcal{D}$ containing multiple regular datasets, where each $D \in \mathcal{D}$ has a split of $D_{train}$ and $D_{test}$.

<!--- more --->

* Consider a typical learning algorithm
  * Input: training set $D_{train} = \{(X_t,Y_t)\}_{t=1}^{T}$
  * Output: parameters $\theta$ of model $M$
  * Objective: good performance on test set $D_{test} = (X,Y)$
* Desire a meta-learning algorithm
  * Input: meta-training set $\mathcal{D}_{meta\_train} = \{(D_{train}^{(n)},D_{test}^{(n)})\}_{n=1}^{N}$
  * Output: parameters $\Theta$ representing a training algorithm
  * Objective: good performance on meta-test set $\mathcal{D}_{meta\_test} = \{(D_{train}^{(n')},D_{test}^{(n')})\}_{n'=1}^{N}$

![meta learning, ref:Ravi&Larochelle'17](https://ws2.sinaimg.cn/large/005PPQ5Ily1g0yn2gf0muj31he0xe7wh.jpg)

### Paper and Method

####OPTIMIZATION AS A MODEL FOR FEW-SHOT LEARNING

The standard optimization algorithms used to train deep neural networks are some variant of gradient descent, which uses updates of the form
$$
\theta_t = \theta_{t-1} - \alpha_t \nabla_{\theta_{t-1}}\mathcal{L}_t.
$$
Key Observation is that this update resembles the update for the cell state in an LSTM
$$
c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t,
$$
if $f_t = 1$, $c_{t-1} = \theta_{t-1}$, $i_t = \alpha_t$ and $\tilde{c}_t = - \nabla_{\theta_{t-1}} \mathcal{L}_t$.

Thus, training a meta-learner LSTM to learn an update rule for training a neural network is key idea.

![](https://ws4.sinaimg.cn/large/005PPQ5Ily1g0yn09xsp9j313q0mewo9.jpg) 

####Learning to learn by gradient descent by gradient descent

Use LSTM to learn the update strategy of neural network, learn an optimizer by gradient descent method, and then use this optimizer to optimize other network parameters. The task of LSTM optimizer is learning $\nabla_{\theta_{t-1}}\mathcal{L}_t$.

![](https://ws1.sinaimg.cn/large/005PPQ5Ily1g0ymzujzn9j31420g80ut.jpg)

#### Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks

The goal of meta-learning is to train a model that can quickly adapt to a new task using only a few training iterations. To accomplish this, the model or learner is trained during a meta-learning phase on a set of tasks, such that the trained model can quickly adapt to new tasks using only a small number of examples or trials.

