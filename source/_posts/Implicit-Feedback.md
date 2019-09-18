---
layout: title
title: Implicit Feedback
date: 2018-08-21 16:29:59
tags: machine learning
categories: paper notes
---

> References
>
> [Collaborative Filtering for Implicit Feedback Datasets](http://yifanhu.net/PUB/cf.pdf)
>
> [TriRank: Review-aware Explainable Recommendation by Modeling Aspects](http://yifanhu.net/PUB/cf.pdf)

<!--- more --->

###Concept of explicit/implicit feedback

**Explicit feedback**:  explicit input by users regarding their interest in products

* Netflix/douban collects star rating for movies
* Music application collects star rating for music or singers

**Implicit feedback:** in many scenes, explicit rating are not always available. Users interact with items through implicit feedback. Input can't clearly reflect user preferences.

* The logs of the user watching the video
* Purchase history, browsing history or search patterns

###Why implicit feedback

The vast majority of the literature in the field is focused on processing explicit feedback; probably thanks to the convenience of using this kind of pure information. **However, explicit feedback is not always available**.

**In many practical situations** recommender systems need to **be centered on implicit feedback**. This may reflect reluctance of users to rate products, or *limitations of the system* that is unable to collect explicit feedback. In an implicit model, once the user gives approval to collect usage data, no additional explicit feedback (e.g. ratings) is required on the user’s part.

###The prime characteristics of implicit feedback

1. No negative feedback. By observing the users behavior, we can infer which items they **probably like** and thus chose to consume. However, it is hard to reliably infer which items a user did not like.  
2. Implicit feedback is inherently noisy.  While we passively track the users behavior, we can only guess their preferences and true motives. like point one, we can only guess these users **probably like **these items.
3. The numerical value of explicit feedback indicates *preference*, whereas the numerical value of implicit feedback indicates *confidence*.
4. Evaluation of implicit-feedback recommender requires appropriate measures.

###Evaluation methodology

**percentile-ranking**: See section 6 in Collaborative Filtering for Implicit Feedback Datasets.

**Hit Ratio at N (HR@N)**: a way of calculating how many “hits” you have in an N-sized list of ranked items. A “hit” could be defined as something that the user has clicked on, purchased, or saved/favourited ("1" in datas). Lager HR indicates better performance.

the size of testing data: M, for the i-th example in testing data (user: $i$, item: $j$), Use model recommend $N$-size ranked list $L$ of items for user $i$. if item $j$ appears in the recommended list, it is deemed a hit. HR is calculated as:
$$
HR@N=\frac{Number\ of \ Hits@N}{M}
$$
**Normalized Discounted Cumulative Gain (NDCG)**: assigns higher importance to results at top ranks, scoring successively lower ranks with marginal fractional utility.
$$
NDCG@N = Z_N \sum_{i=1}^N \frac{2^{r_i}-1}{log_2 (i+1)}
$$
where $Z_K$ is the normalizer to ensure the ideal ranking has a value of 1; $r_i$ is the graded relevance of item at position $i$. In implicit feedback, $r_i$ are binary value. if the user interact with item: $r_i = 1$, and 0 otherwise. Lager NDCG indicates better performance.

Here is simple implementation of NDCG:

```python
# ref: https://gist.github.com/bwhite/3726239
def dcg_at_k(r, k, method=0):
    """Score is discounted cumulative gain (dcg)
    Relevance is positive real values.  Can use binary
    as the previous methods.
    Args:
        r: Relevance scores (list or numpy) in rank order
            (first element is the first item)
        k: Number of results to consider
        method: If 0 then weights are [1.0, 1.0, 0.6309, 0.5, 0.4307, ...]
                If 1 then weights are [1.0, 0.6309, 0.5, 0.4307, ...]
    Returns:
        Discounted cumulative gain
    """
    r = np.asfarray(r)[:k]
    if r.size:
        if method == 0:
            return r[0] + np.sum(r[1:] / np.log2(np.arange(2, r.size + 1)))
        elif method == 1:
            return np.sum(r / np.log2(np.arange(2, r.size + 2)))
        else:
            raise ValueError('method must be 0 or 1.')
    return 0.


def ndcg_at_k(r, k, method=0):
    """Score is normalized discounted cumulative gain (ndcg)
    Relevance is positive real values.  Can use binary
    as the previous methods.
    Args:
        r: Relevance scores (list or numpy) in rank order
            (first element is the first item)
        k: Number of results to consider
        method: If 0 then weights are [1.0, 1.0, 0.6309, 0.5, 0.4307, ...]
                If 1 then weights are [1.0, 0.6309, 0.5, 0.4307, ...]
    Returns:
        Normalized discounted cumulative gain
    """
    dcg_max = dcg_at_k(sorted(r, reverse=True), k, method)
    if not dcg_max:
        return 0.
    return dcg_at_k(r, k, method) / dcg_max
```

