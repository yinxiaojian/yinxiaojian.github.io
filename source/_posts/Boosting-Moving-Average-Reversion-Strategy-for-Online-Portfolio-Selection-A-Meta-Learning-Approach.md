---
title: >-
  Boosting Moving Average Reversion Strategy for Online Portfolio Selection: A
  Meta-Learning Approach
date: 2018-07-15 20:30:17
tags: machine learning
categories: paper notes
---

> [Boosting Moving Average Reversion Strategy for Online Portfolio Selection: A Meta-Learning Approach](https://link.springer.com/chapter/10.1007/978-3-319-55699-4_30)

<!--- more --->

#### Background

- All existing mean reversion strategies do not fully consider the noisy data and outliers 
- the assumption of single-period prediction leads to estimation error
- RMR (Robust Mean Reversion) and OLMAR uses multi-period prediction, but the algorithm sees each period equally, which ignores the temporal heterogeneity of historical price relatives and causes inaccuracy of predictions. 

#### Contribution

We utilize meta learning to exploit the benefit of multi-period prediction and the periods are assigned with weights according to their performances. Moreover, this alleviates the impact of noisy data and outliers. 

**first phase**: predict the price relative at next period based on historical data.

**second phase: **compute the optimal portfolio given the prediction of price relative. 

use Passive-Aggressive algorithm.

### Algorithm

1. generate a set of experts and each expert is a predictor of the price relative following a mean reversion policy.  
2. In each period, each expert first makes its predictions on the price relatives in next period.
3. compute the cumulated losses induced by each expert from their historical predictions and the true past price relatives.  
4. With the cumulated losses, we compute the weights assigned to each expert.
5. make prediction.
6. use **Online Passive-Aggressive learning method **to compute an optimized portfolio with the final prediction of price relatives in next period. 

