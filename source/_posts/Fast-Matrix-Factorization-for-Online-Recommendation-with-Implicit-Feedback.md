---
layout: title
title: Fast Matrix Factorization for Online Recommendation with Implicit Feedback
date: 2018-08-15 19:17:16
tags: machine learning
categories: paper notes
---

> [Fast Matrix Factorization for Online Recommendation with Implicit Feedback](https://www.comp.nus.edu.sg/~xiangnan/papers/sigir16-eals-cm.pdf)

<!--- more --->

### Background

User Personalization has become prevalent in modern recommender system. Among its various methods, MF is important. Early work on MF have largely focused explicit feedback. (introduction of explict feedback and MF methods).

However, explicit fimplictit ratings are not always avaliable in many applications: **implicit feedback**. (introduction of implicit feedback).

(introduction of algorithm to solve implicit feedback). two challenging problem: **implicit feedback** and **online  learning**.

introduction of our algorithm, summarize key contributions.

### Contribution

- propose an **item popularity-aware weighting** scheme on the full missing data that effectively tailors the MF model for learning from implicit feedback.  
- develop a new algorithm for learning model parameters efficiently and devise an incremental update strategy to support real-time **online learning**.
- conduct extensive experiments with both offline and online protocols on two real-world datasets, showing that our method consistently **outperforms** state-of-the-art implicit MF methods

### Algorithm

#### 1 Item-Oriented Weighting on Missing Data

**Intution**: a popular item is more likely to be known by users, thus a missing on it is more probably that the user is not interested with it. 

Old design : Uniform weight
$$
L = \sum_{(u,i) \in R} w(r_{ui} - \hat{r}_{ui})^2
+ \sum_{(u,i)\notin R}\hat{r}_{ui}^2
+ \lambda(\sum_{u=1}^{M}\|\mathbb{p}_u\|^2 + \sum_{i=1}^N \|\mathbb{q}_i\|^2)
$$
New design: Non-uniform weighing 
$$
L = \sum_{(u,i) \in R} w(r_{ui} - \hat{r}_{ui})^2
+ \sum_{u=1}^M \sum_{i\notin R_u}c_i \hat{r}_{ui}^2
+ \lambda(\sum_{u=1}^{M}\|\mathbb{p}_u\|^2 + \sum_{i=1}^N \|\mathbb{q}_i\|^2)
$$
 where $c_i = c_0 \frac{f_i^a}{\sum_{j=1}^N f_j^a}$,  $f_i$ denotes the popularity of item $i$, given by its frequency in the implicit feedback data: $|R_i|/\sum_{j=1}^N|R_j|$.

#### 2 Fast eALS Learning Algorithm

develop a Coordinate Descent learner to optimize the whole-data based MF 

- Element-wise Alternating Least Squares Learner (eALS)
- Optimize one latent factor with others fixed (greedy exact optimization) 

#### 3 Online Incremental Learning

new interaction should changel the **local features**, while the global reamain largely unchanged.



