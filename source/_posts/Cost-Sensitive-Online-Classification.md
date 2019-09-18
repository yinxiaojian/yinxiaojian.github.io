---
title: Cost Sensitive Online Classification
date: 2018-06-14 14:38:47
tags: machine learning
categories: artificial intelligence 
---

paper list:

A Framework of Online Learning with Imbalanced Streaming Data

Online Asymmetric Active Learning with Imbalanced Data 

<!--more-->

### A Framework of Online Learning with Imbalanced Streaming Data 

####Background

Many previous works have considered cost-sensitive approaches in an online setting for streaming data, where **fixed costs** are assigned to different classes, or ad-hoc costs are adapted based on the distribution of data received so far. However, it is not necessary for them to achieve optimal performance in terms of the measures suited for imbalanced data, such as F-measure, area under ROC curve (AUROC), area under precision and recall curve (AUPRC).  

#### Contribution

**Motivation:** if multiple classifiers with a number of **cost** are learned simultaneously, there must exist one setting that is most appropriate to the data.  

**Algorithm:**

* construct a pool of multiple values of **cost**->**classifiers**.
* choose the classifier with best (according to distribution) performance (F-measure, ROC, AUROC) on historical data to predict
* update classifiers and performance

#### Theoretical analysis

When K is sufficiently large (the number of classifiers), there exists **a sequence of classifiers** among the K sequences that will even tually converge to a classifier that has a close-to-optimal F-measure.

Maximizing F-measure is equivalent to minimizing a cost- sensitive error. 

### Online Asymmetric Active Learning with Imbalanced Data 

#### Background

This paper considers online learning with **imbalanced** streaming data under a **query budget**: the act of querying for labels is constrained to a budget limit.

#### Contribution

**Algorithm**

* Make decision about the binary label
* Query stage: decide whether to query the label and update the model
  * if reaches the budget limit, use the last updated model or averaged model
  * else, use the output of **query function**{0, 1}, if output equal 1, update model

the key concern is the **query model**.

**Query model**

* Compute $p_t = w_{t-1}x_t$
* if $p_t \ge 0$, the probability (output = 1) is $\frac{c_+}{|p_t|+c_+}$。else the probability is $\frac{c_-}{|p_t|+c_-}$.