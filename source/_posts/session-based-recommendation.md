---
layout: article
title: session-based recommendation
date: 2018-11-02 00:06:57
tags: [machine learning, deep learning]
categories: [paper notes]
---

### Definition

session is a mechanism used by website to identify users. A typical scenario is shopping cat of e-commerce website. It creates a specific session for the user and track the user's click behavior. In short, session is a an ordered collection of past items the user has interacted with.

<!--- more --->

### Traditional Methods

#### item-to-item recommendation approach

The next recommendation can be computed based on simliar matrix. The main **issue** is only taking into account the last click of the user, in effect ignoring the information of the past clicks.

#### Markov decision Processes(MDPs)

The next recommendation can be computed on the basis of the transition probability between items. the main **issue** is that the state space quickly becomes unmanageable when trying to include all possible sequences of user selections.

### Deep Neural Network

Here introuce two main directions: RNN and CNN. 

> Paper for RNN: SESSION-BASED RECOMMENDATIONS WITH RECURRENT NEURAL NETWORKS
>
> Paper for CNN: Personalized Top-N Sequential Recommendation via Convolutional Sequence Embedding

#### RNN-based Method

##### Contribution

The first article proposed that RNNs can be applied to session-based recommendation with remarkable results.

##### Network Structure

**Input**: the actual state of the session. The state of the session can either be the item of the actual event or the events in the session so far.  One hot encoding is used with adding an
additional embedding layer to reduce dimension.

**Output**: a probability distribution over the next element in the session.

![General architecture of the network](http://oygov02sc.bkt.clouddn.com/SESSION-BASED%20RECOMMENDATIONS-structure)

##### Trianing Strategy

**Session-parallel mini-batches**: To generate batches for training, this paper create an order for the sessions and then use the first event of the first $X$(batch size) sessions to form the input of the first mini-batch. The second mini-batch is formed from the second events and so on. If any of the sessions end, the next available session is put in its place. Due to  independent of  sessions,  hidden state need to be reset when this switch occurs.

**sampling  on the output**: Calculating a score for each item in each step would make the algorithm scale with the product of the number of items and the number of events. This paper choose next clicked item in this mini-batch as positive example and the items from the other mini-batch as negative examples to compute. Meanwhile, only corresponding weights will be updated. 

> the negative sampling strategy is based on popularity

##### RANKING LOSS

This paper adopt pair-wise ranking loss which is more stable than point-wise ranking loss in this network. Because there are no absolute positive and negative samples.

This paper use the following two.

**BPR**:  Bayesian Personalized Ranking is a matrix factorization method.
$$
L_s = -\frac{1}{N_S} \cdot \sum_{j=1}^{N_S} \log (\sigma (\hat{r}_{s,i} - \hat{r}_{s,j})).
$$
**TOP1**: regularized approximation of the relative rank of the relevant item.
$$
L_s = \frac{1}{N_S} \cdot \sum_{j=1}^{N_S} \sigma (\hat{r}_{s,j} - \hat{r}_{s,i}) + \sigma (\hat{r}_{s,j}^2),
$$
where $N_S$ is the sample size, $\hat{r}_{s, k}$ is the score on item $k$ at the given point of the session, $i$ is the desired item (positive) and $j$ are the negative samples.

#### CNN-based Method

##### Contribution

This paper proposed a ConvolutionAl Sequence Embedding Recommendation Model, or
Caser for short, as a solution to top-N sequential (session-based) recommendation.

> - RNN depend on a hidden state of the entire past that cannot fully utilize parallel computation within a sequence. Thus their speed is limited in both training and
>   evaluation. (Ref: A Simple but Hard-to-Beat Baseline for Session-based Recommendations.)
> - In sequential (session-based) recommendation problem, not all adjacent actions have dependency relationships (e.g. a user bought i2 after i1 only because she loves i2). Therefore, sequentially connected network structure (RNN) may not work well.

##### Network Structure

The network architecture is as follows

![](https://wx2.sinaimg.cn/large/005PPQ5Ily1g0yn0xevcij31nc0ua7ei.jpg)

**Embedding Look-up**: Caser captures sequence features in the latent space by feeding
the embeddings of previous $L$ items into the neural network. After embedding look-up, we get a matrix $\mathbf{E}^{(u,t)} \in \mathbb{R}^{L \times d}$ for user $u$ at time step $t$.we also have an embedding $\mathbf{P}_u \in \mathbb{R}^d$.
Rd for a user $u$, representing user features in latent space. 

**Convolutional Layers**: *Horizontal filters* are $h \times d$ matrix with different height $h$, which pick up signals for sequential patterns by sliding over the rows of $\mathbf{E}$. *Vertical filter* is a $L \times 1$ matrix, which capture point-level sequential patterns by sliding over the columns of $\mathbf{E}$.

**Fully-connected Layers**: The outputs of the two convolutional layers are fed into a fully-connected neural network layer to get *convolutional sequence embedding*  $\mathbf{z} \in \mathbb{R}^d$, which has more high-level and abstract features. To capture user’s general preferences, This network look-up $\mathbf{P}_u$, then concatenate  $\mathbf{P}_u$ and $\mathbf{z}$ together and project them to an output layer. valuey $\mathbf{y}^{(u, t)}$in the output layer is associated with the probability over the next element in the session.

##### Trianing Strategy

Transform $\mathbf{y}^{(u, t)}$ to probabilities by sigmoid function, then the likelihood of all sequences in the dataset is:
$$
p(\mathcal{S}|\Theta) = \prod_u \prod_{t\in \mathcal{C}^u} \sigma(\mathbf{y}_{\mathcal{S}_t^u}^{(u,t)})  \prod_{j \neq \mathcal{S}_t^u} (1 - \sigma(\mathbf{y}_{j}^{(u,t)})),
$$
where $\mathcal{C}^u = \{L+1, L+2, ..., |\mathcal{S}^{u}|\}$. Taking the negative logarithm of likelihood, the objective function, also kown as *binary cross-entropy* loss:
$$
\ell = \sum_{u} \sum_{t\in \mathcal{C}_u} \sum_{i \in \mathcal{D}_t^u} -\log (\sigma(\mathbf{y}_i^{(u,t)})) + \sum_{j \neq i} -\log (1 - \sigma(\mathbf{y}_j^{(u,t)})),
$$
where $\mathcal{D}_t^u = \{ \mathcal{S}_t^u, \mathcal{S}_{t+1}^u,..., \mathcal{S}_{t+T}^u  \}$. For each target item $i$,  negative instances $j$  in the second term are randomly sampling from item set.

