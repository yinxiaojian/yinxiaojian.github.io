---
title: Neural Collaborative Filtering
date: 2018-08-23 16:11:54
tags: machine learning
categories: paper notes
---

>[Neural Collaborative Filtering](https://arxiv.org/pdf/1708.05031.pdf)

<!--- more --->

### Background

Matrix factorization is known to be the most simple yet effective model for the collaborative filtering task. Much research effort has been devoted to enhancing MF. But the **inner product** function can **limit the expressiveness** of MF. This paper's solution is exploring the use of **deep neural networks** for learning the interaction function from data. We focus on implicit feedback which is more challenging to utilize than explicit feedback.

### Contributions

1. present a neural network architecture to model latent features of users and items and devise **a general framework NCF** for collaborative filtering based on neural networks. 
2. show that MF can be interpreted as a specialization of NCF and utilize a multi-layer perceptron to endow NCF modelling with a high level of non-linearities. 
3. perform extensive experiments on two real-world datasets.

### Algorithm

NCF is the base framework, GMF and MLP are two kinds of detail implementation of neural fm. NeuFM is the combine of GFM and MLP.

#### General Framework (NCF)

NCF uses a multi-layer model to learn the user-item interaction function 

* Input: sparse **feature vector** for user u and item i 
* Output: predicted score $\hat{y}_{ui}$ 

Interaction function: 
$$
\hat{y}_{ui} = f(\mathbf{p}_u, \mathbf{q}_i)
$$
Structure figure:

![](http://oygov02sc.bkt.clouddn.com/Neural%20collaborative%20filtering%20framework.png)

####Generalized Matrix factorization (GMF)

NCF can express and generalize MF:
$$
\hat{y}_{ui} = f(\mathbf{p}_u, \mathbf{q}_i) = 
\sigma(h^T\phi_{out}(\phi_{X}(...\phi_2(\phi_1(\mathbf{p}_u,\mathbf{q}_i))...)))
$$
With define layer 1 as an **element-wise product**, output layer as fully connected layer without bias, then we have GMF:
$$
\hat{y}_{ui} = a_{out}(\mathbf{h}^T(\mathbf{p}_u\odot \mathbf{q}_i))
$$

####Multi-Layer Perceptron (MLP)

NCF can endow more nonlinearities to learn the interaction function: 
$$
\phi_1(\mathbf{p}_u, \mathbf{q}_i) = 
\left[                
  \begin{array}{ccc}   
    \mathbf{p}_u\\
    \mathbf{q}_i\\
  \end{array}
\right]                
$$
Remaining Layers:
$$
\phi_2(\mathbf{z}_1) = a_2(\mathbf{W}_2^T \mathbf{z}_1 + \mathbf{b}_2) \\
...
\\
\phi_2(\mathbf{z}_{L-1}) = a_L(\mathbf{W}_L^T \mathbf{z}_{L-1} + \mathbf{b}_L)
\\
\hat{y}_{ui} = \sigma(\mathbf{h}^T\phi_L(\mathbf{z}_{L-1}))
$$

#### Fusion of GMF and MLP (NeuFM)

Fusing GMF and MLP by allowing them learn different sets of embeddings.

structure figure:

![](https://ws1.sinaimg.cn/large/005PPQ5Ily1g0yn1irgeyj30wk0jndkk.jpg)

#### Learning Model

**Explicit Feedback**: regression with squared loss
$$
L_{sqr} = \sum_{(u,i) \in \mathcal{Y}} (y_{ui} - \hat{y}_{ui})^2
$$
**Implicit Feedback**: log loss
$$
L = \sum_{(u,i) \in {\mathcal{Y} \cup \mathcal{Y^-}}}
y_{ui}log\hat{y}_{ui} + (1-y_{ui})log(1-\hat{y}_{ui})
$$
Optimization: SGD (adaptive learning rate variants)

