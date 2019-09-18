---
title: CNN反向传播算法
date: 2018-03-21 11:19:00
tags: [deep learning]
categories: [artificial intelligence]
---

> 全文翻译自 Notes on Convolutional Neural Networks 
>
> [论文链接](http://web.mit.edu/jvb/www/papers/cnn_tutorial.pdf)
>
> 释部分为自己的补充说明，可能存在错误

<!--- more --->

###1 介绍

本文档讨论CNN的推导和实现，并加上一些简单的扩展。相比权值，卷积神经网络涉及到更多的连接。其架构本身实现了正则化。此外，CNN在某种程度上实现了平移不变性。这种特殊的神经网络假设我们希望通过数据驱动的方式学习过滤器，用来提取输入特征。推导是在二维的数据和卷积的基础上展开的，但可以很容易的推广到任意维度。

我们先对经典的BP算法做一个描述，然后推导2D卷积神经网络中卷积层和子采样层（池化层）的BP权值更新方法。在整个过程中，我们强调了视线的效率，给出MATLAB代码片段配合方程讲解。然后讨论如何自动组合前一层的feature map，最后是feature map的稀疏组合。

免责声明：这个笔记可能有错误。

> feature map 就是特征矩阵，一张图片的每个像素点的RGB值可分别组成矩阵（R矩阵，G矩阵，B矩阵），这些就是feature map

###2 全连接的反向传播算法

一个典型的卷积神经网络，开始是卷积层和子采样层交替，最后几层（最接近输出）是全连接的一维网络（全连接层）。当你准备将2D feature map作为一维全连接网络的输入时，直接将所有特征连接为一个一维向量，然后就可以利用BP算法。因此在讨论CNN之前，这里先对BP算法进行解释。

####2.1 前向传播 Feedforward Pass

在接下来的推导中，我们考虑平方和误差函数，对于一个有c个类别和N个训练样本的多分类问题，计算公式如下：
$$
E^N=\frac{1}{2}\sum_{n=1}^{N}\sum_{k=1}^{c}(t_k^n-y_k^n)^2
$$
$t_k^n$是第n个样本对应标签的第k个维度的值，$y_k^n$表示第n个样本对应的网络输出的第k个值。对于多分类问题，输出一般为“one-of-c”形式。如果样本$x^n$属于第k类，那么$t_k^n$为正，$t^n$的其他维度即$t_{k\neq1}^n$为0或负数。这取决于输出激活函数的选择，此处不做讨论。

因为整个训练集的误差为每个训练样本的误差和，所以我们先讨论一个样本。第n个样本的误差如下：
$$
E^N=\frac{1}{2}\sum_{n=1}^{N}\sum_{k=1}^c(t_k^n-y_k^n)^2=\frac{1}{2}||t^n-y^n||_2^2\tag{1}
$$
在全连接层，我们可以根据反向传播规则计算E对网络权值的偏导数。用$l$代表当前层，输出层被指定为$L$层，输入层为第1层，那么当前层的输出为
$$
x^l=f(u^l), with \space u^l=W^lx^{l-1}+b^l\tag{2}
$$


输出激活函数$f(\cdot)$一般选择logistic(sigmoid)函数$f(x)=(1+e^{-\beta x})^{-1}$或者hyperbolic tangent函数$f(x)=a\space tanh(bx)$。logistic函数将$[-\infty,\infty]$映射到$[0,1]$，hyperbolic tangent函数将$[-\infty,\infty]$映射到$[-a,a]$，因此hyperbolic tangent函数的输出平均值一般趋于0，sigmoid函数的输出平均为非零。但是如果将训练数据归一化到均值为0方差为1，可以在梯度下降时增加收敛。对于归一化的数据集来说，hyperbolic tangent函数是较好的选择。LeCun推荐$a=1.7159,b=2/3$，所以最大非线性点在$f(\pm1)=\pm1$，因此对预期训练目标进行正则化将会避免训练饱和。

####2.2 反向传播 Backpropagation Pass

通过网络反向传播的误差可以看作每个神经元关于偏差扰动的敏感度（sensitivities），通过链式法则可得：

> 敏感度就是误差对目标的偏导数

$$
\begin{equation}
\frac{\partial{E}}{\partial{b}}=\frac{\partial{E}}{\partial{u}}\frac{\partial{u}}{\partial{b}}=\delta
\end{equation}\tag{3}
$$

因为$\frac{\partial{u}}{\partial{b}}=1$，所以bias的敏感度和误差对一个神经元的输入的偏导数是相等的。利用这个关系将高层误差反向传播到底层，使用下面的递推关系式：
$$
\delta^t=(W^{l+1})^T\delta^{l+1}\circ f'(u^t)\tag{4}
$$
其中$\circ$表示对应位置元素相乘，从公式（1）可以看到，输出层神经元的敏感度会有一些不同：
$$
\delta^L=f'(u^L)\circ(y^n-t^n)
$$
最后，对每个神经元运用delta规则进行更行，即对神经元的输入使用delta进行缩放。用向量的形式表述就是：输入向量（前一层的输出）和灵敏度向量之间的一个外积：
$$
\frac{\partial{E}}{\partial{W^l}}=x^{l-1}(\delta^l)^T\tag{5}
$$

$$
\bigtriangleup W^l=-\eta\frac{\partial{E}}{\partial{W^l}}\tag{6}
$$

更新偏置值（bias）也是相同的原理，在实际中通常为每个权值赋予一个特定的学习率$\eta_{ij}$。

### 3 卷积神经网络 Convolutional Neural Networks

>公式符号释意
>
>在本文中卷积核有多个，每个卷积核和feature map有多个维度，例如一张图片有RGB值，那么就有三个维度
>
>$i$----卷积核或者feature的第几维度
>$j$----第几个卷积核
>$up()$----上采样函数
>$down()$----下采样函数
>$l$----当前层



通常卷积层和子采样层交替以减少计算时间并逐步建立空间和结构不变性。为了同时保持特异性，需要小的子采样因子。这并不是什么新鲜的概念，但简洁有效。这种模型在哺乳动物的视觉皮层非常常见，在过去十年，听觉神经科学发现，这些相同的设计范例可以在多种不同动物皮层的主要和带状听觉区域中找到。 分级分析和学习架构可能仍然是听觉领域的关键。

####3.1 卷积层 Convolution Layers

我们现在推导卷积层的BP更行。在一个卷积层，上一层的feature map与一个卷积核进行卷积运算，然后通过一个激活函数获得输出feature map。每个输出map可能是多个卷积结果的结合，公式表述：
$$
x_j^l=f\left(\sum_{i\in M_j}x_i^{l-1}*k_{ij}^{l}+b_j^l\right)
$$


$M_j$表示输入maps的集合，在MATLAB中卷积的运算需采用"valid"模式。通常情况下，输入maps选择一对或者三个。后面我们会讨论如何自动选择需要组合的特征map。每一个输出map都会加上一个偏执项$b$，但是对于一个特定的输出map，输入maps会和相同的卷积核进行卷积运算，也就是说，如果输出map $j$ 和 $k$ 都是由map $i$通过卷积运算的结果，那么对应的卷积核是不同的。

>一张图片有RGB三个颜色通道，则对应的卷积核也是三维的，输出的feature map就是三个卷积结果的结合
>
>![CNN_02](https://ws3.sinaimg.cn/large/005PPQ5Ily1g0ymwrjv8jg30m50iqaob.gif)

3.1.1 Computing the Gradients

我们假设每个卷积层$l$后面都接一个子采样层 $l+1$。在使用BP算法计算 $l$ 层每个节点的敏感度时，我们需要先对下一层（$l+1$层）连接当前层（$l$层）的节点的敏感度求和，再乘以这些连接对应的权值（连接$l$层和$l+1$层的权值），最后乘以当前层$l$的该神经元节点的输入$u$的激活函数$f$的导数值。因为下采样的存在，采样层的一个像素（神经元）对应的灵敏度$\delta$对应于卷积层（上一层）的输出map的一块像素（采样窗口大小）。因此，$l$层中的map的每个节点只与$l+1$层中相应map的一个节点连接。为了有效计算$l$层的敏感度，我们可以对下采样层的敏感，度map进行“上采样(upsample)”操作，这样可以使下采样层的敏感度map的大小和卷积层的map大小相同，然后将$l$层的激活值偏导map和$l+1$层的上采样敏感度map对应元素相乘。因为下采样层map的权值均是一个常数$\beta$（见3.2节），所以我们只需要将上一步得到的结果乘以 $ \beta$ 就可以得到$l$层的敏感度map $\delta^l$。我们可以在卷积层的每个feature map进行相同的计算过程：
$$
\delta_j^l=\beta_j^{l+1}\left(f'(u_j^l)\circ up(\delta_j^{l+1})\right)
$$
其中的 $up(\cdot)$表示上采样操作，如果下采样过程的采样因子为$n$，只需将每个像素点在水平方向和垂直方向复制$n$次就可以实现，一种有效的实现方式是利用 Kronecker product:
$$
up(x)=x\otimes1_{n\times n}
$$


>Ref:[克罗内克积](https://zh.wikipedia.org/zh/克罗内克积)

现在对于一个给定的map，我们已经可以计算其敏感度map，然后我们可以简单的对敏感度map的所有节点求和计算出其bias（偏置项）梯度:
$$
\frac{\partial{E}}{\partial{b_j}}=\sum_{u,v}(\delta_j^l)_{uv}
$$
最后，使用BP算法计算权重（卷积核）的梯度，很多权重在连接中是共享的，因此我们需要对所有与该权值有联系（权值共享的连接）的连接对该点求梯度，然后对这些梯度进行求和，就像计算bias梯度一样：
$$
\frac{\partial{E}}{\partial{k_{ij}^l}}=\sum_{u,v}(\delta_j^l)_{uv}(p_i^{l-1})_{uv}\tag{7}
$$
其中$(p_i^{l-1})_{uv}$是$x_i^{l-1}$在卷积运算时和$k_{ij}^l$按元素相乘得到输出map的$(u,v)$位置值的patch。看上去好像我们我刻意追踪输入map中哪些patch与输出map中的哪些像素（及其对应的敏感map）相对应，但是公式(7)可以通过卷积运算获得:
$$
\frac{\partial{E}}{\partial{k_{ij}^l}}=rot180(conv2(x_i^{l-1},rot180(\delta_j^l),'valid'))
$$
这里我们对$\delta$敏感度map做旋转操作是为了这样就可以进行互相关计算而不是卷积，之后将输出旋转回来，以便当我们在前馈阶段执行卷积时，卷积核将具有正确的方向。

####3.2 子采样层 Sub-sampling Layers

子采样层对输入maps进行downsample操作（池化），输入有N个maps，输出也会有N个maps，只是每个maps变小了，公式表述如下：
$$
x_j^l=f\left(\beta_j^ldown(x_j^{l-1})+b_j^l\right)
$$


其中的$down(\cdot)$是下采样函数。典型的操作一般是对输入图像的不同$n\times n$的块的像素进行求和。这样输出图像在两个维度上都缩小了n倍。每个输出map都有一个属于自己的乘性偏置β和一个加性偏置b。

3.2.1 Computing the Gradients

这里的困难在于如何计算敏感度maps。只要我们能得到敏感度maps，需要更新的只有bias参数 $\beta$ 和 $b$ 。如果子采样层的下一层是全连接层，那么子采样层的敏感度maps只需要利用BP算法就可以容易获得。所以我们假设子采样层的上一层和下一层都是卷积层。

当我们在3.1.1节计算卷积核梯度时，我们需要找到输入map中patch和输出map的像素的对应关系。这里就是必须找到当前层的敏感度map中那个patch与下一层的敏感度矩阵的的给定像素对应，这样就可以像公式 (4)那样应用$\delta$递推。 当然，将输入patch和输出像素之间连接相乘的权重恰好是（旋转的）卷积核的权重。利用卷积可以实现该计算：
$$
\delta_j^l=f'(u_j^l)\circ conv2(\delta_j^{l+1},rot180(k_j^{l+1}),full)
$$
如前所述，我们旋转内核是为了是卷积函数实施互相关计算。同时，我们需要对卷积边界进行“full”处理，Matlab的卷积函数会自动执行这个过程，对缺少的输入像素进行补零操作。

现在我们可以很容易的计算 $b$ 和 $\beta$ 的梯度，加性基 $b$ 的处理和之前一样，将敏感度map中所有元素相加即可 ：
$$
\frac{\partial E}{\partial b_j}=\sum_{u,v}(\delta_j^l)_{uv}
$$
对于乘性偏置$\beta$ ，参与了前向传播时下采样map的运算，所以在前向计算时将这些maps保存，这样避免了在反向传播时重复计算。定义：
$$
d_j^l=down(x_j^{l-1})
$$


$\beta$ 梯度计算公式如下：
$$
\frac{\partial E}{\partial \beta_j}=\sum_{u,v}(\delta_j^l)
$$

> 补充说明：对rot180的解释
>
> 假设第$l-1$层输出$x^l$是一个3x3矩阵，第$l$层卷积核$k^l$是一个2x2的矩阵，步幅为1，输出一个2x2的矩阵$z^l$，则有（简化$b^l=0$，单卷积核)
> $$
> x^{l-1}*k^{l}=z^{l}
> $$
> 矩阵形式表达：
> $$
> \left( \begin{array}{}
> x_{11}&x_{12}&x_{13}\\
> x_{21}&x_{22}&x_{23}\\
> x_{31}&x_{32}&x_{33}
> \end{array}
> \right)
> *
> \left( \begin{array}{}
> k_{11}&k_{12}\\
> k_{21}&k_{22}
> \end{array}
> \right)
> =
> \left( \begin{array}{}
> z_{11}&z_{12}\\
> z_{21}&z_{22}
> \end{array}
> \right)
> $$
> 根据卷积的定义可得：
> $$
> z_{11}=x_{11}k_{11}+x_{12}k_{12}+x_{21}k_{21}+x_{22}k_{22}\\
> z_{12}=x_{12}k_{11}+x_{13}k_{12}+x_{22}k_{21}+x_{23}k_{22}\\
> z_{21}=x_{21}k_{11}+x_{22}k_{12}+x_{31}k_{21}+x_{32}k_{22}\\
> z_{22}=x_{22}k_{11}+x_{23}k_{12}+x_{32}k_{21}+x_{33}k_{22}
> $$
> 第$l$层的梯度误差为$\delta^l$，反向传导：
> $$
> \frac{\partial{E}}{\partial{k^l}}=\frac{\partial{E}}{\partial{z^l}}\frac{\partial{z^l}}{\partial{k^l}}=\delta^l\frac{z^l}{k^l}
> $$
> 因此对于卷积核$k^l$的梯度为第$l$层的敏感度（梯度）乘上$\frac{\partial{z^l}}{\partial{k^l}}$，分别计算可得
> $$
> \bigtriangledown{k_{11}}=\delta_{11}x_{11}+\delta_{12}x_{22}+\delta_{21}x_{21}+\delta_{22}x_{22}\\
> \bigtriangledown{k_{12}}=\delta_{11}x_{12}+\delta_{12}x_{13}+\delta_{21}x_{22}+\delta_{22}x_{23}\\
> \bigtriangledown{k_{21}}=\delta_{11}x_{21}+\delta_{12}x_{22}+\delta_{21}x_{31}+\delta_{22}x_{32}\\
> \bigtriangledown{k_{22}}=\delta_{11}x_{22}+\delta_{12}x_{23}+\delta_{21}x_{32}+\delta_{22}x_{33}
> $$
> 上面4个式子用矩阵卷积形式表示：
> $$
> \left( \begin{array}{}
> x_{11}&x_{12}&x_{13}\\
> x_{21}&x_{22}&x_{23}\\
> x_{31}&x_{32}&x_{33}
> \end{array}
> \right)
> *
> \left( \begin{array}{}
> \delta_{11}&\delta_{12}\\
> \delta_{21}&\delta_{22}
> \end{array}
> \right)
> =
> \left( \begin{array}{}
> \bigtriangledown k_{11}&\bigtriangledown k_{12}\\
> \bigtriangledown k_{21}&\bigtriangledown k_{22}
> \end{array}
> \right)
> $$
> 公式化表达为：
> $$
> \frac{\partial{E}}{\partial{k^l}}=x^{l-1}*\delta^l
> $$
>
>
> 这里需要注意，在论文中进行了两次旋转，这是因为在MATLAB中conv2函数在计算卷积时除了会对矩阵进行“0”扩展，还会将卷积核进行旋转，然后再计算。例如：
>
> ```matlab
> a =
>      1     1     1
>      1     1     1
>      1     1     1
> k =
>      1     2     3
>      4     5     6
>      7     8     9
>
> >> convn(a,k,'full')
> ans =
>
>      1     3     6     5     3
>      5    12    21    16     9
>     12    27    45    33    18
>     11    24    39    28    15
>      7    15    24    17     9
> ```
>
> 因此在编写代码时需要注意保持一致（要么旋转，要么不旋转）。
>
> 在3.2节计算 $\delta^l_j$ 也是同理
>
> 利用反向传播：
> $$
> \frac{\partial{E}}{\partial{x^{l-1}}}=\frac{\partial{E}}{\partial{z^l}}\frac{\partial{z^l}}{\partial{x^{l-1}}}=\delta^l\frac{z^l}{x^{l-1}}
> $$
> 利用上式，可得：
> $$
> \bigtriangledown{x_{11}}=\delta_{11}k_{11}\\
> \bigtriangledown{x_{12}}=\delta_{11}k_{12}+\delta_{12}k_{11}\\
> \bigtriangledown{x_{13}}=\delta_{12}k_{12}\\
> \bigtriangledown{x_{21}}=\delta_{11}k_{21}+\delta_{21}k_{11}\\
> \bigtriangledown{x_{22}}=\delta_{11}k_{22}+\delta_{12}k_{21}+\delta_{21}k_{12}+\delta_{22}k_{11}\\
> \bigtriangledown{x_{23}}=\delta_{12}k_{22}+\delta_{22}k_{12}\\
> \bigtriangledown{x_{31}}=\delta_{21}k_{21}\\
> \bigtriangledown{x_{32}}=\delta_{21}k_{22}+\delta_{22}k_{21}\\
> \bigtriangledown{x_{33}}=\delta_{22}k_{22}
> $$
> 上面九个式子用矩阵卷积形式表示：
> $$
> \left( \begin{array}{}
> 0&0&0&0\\
> 0&\delta_{11}&\delta_{12}&0\\
> 0&\delta_{21}&\delta_{22}&0\\
> 0&0&0&0
> \end{array}
> \right)
> *
> \left( \begin{array}{}
> k_{22}&k_{21}\\
> k_{12}&k_{11}
> \end{array}
> \right)
> =
> \left( \begin{array}{}
> \bigtriangledown x_{11}&\bigtriangledown x_{12}&\bigtriangledown x_{13}\\
> \bigtriangledown x_{21}&\bigtriangledown x_{22}&\bigtriangledown x_{23}\\
> \bigtriangledown x_{31}&\bigtriangledown x_{32}&\bigtriangledown x_{33}
> \end{array}
> \right)
> $$
> 公式化表达为：
> $$
> \frac{\partial{E}}{\partial{x^{l-1}}}=\delta^l * rot180(k^l)
> $$
>

####3.3 学习特征maps组合 Learning Combinations of Feature Maps

通常，对不同的maps进行卷积并对结果求和获得一个输出map，往往能取得不错的效果。在一些文献中，通过人工选择输入maps进行组合。但是我们可以尝试通过训练获得这个组合。让 $a_{ij}$ 表示得到第 $j$ 个输出map中第 $i$ 个输入map的权重，那么第 $j$ 个输出map的定义如下：
$$
x_j^l=f\left( \sum_{i=1}^{N_{in}}a_{ij}(x_i^{l-1}*k_i^l)+b_j^l\right)
$$
同时需满足以下约束：
$$
\sum_ia_ij=1,and \space 0\leq a_{ij}\leq1
$$
这些约束可以通过将变量$a_{ij}$表示为softmax形式来加强：
$$
a_{ij}=\frac{exp(c_{ij})}{\sum_kexp(c_{kj})}
$$


因为对于固定的 $j$ 来说，每组权值 $c_{ij}$ 都和其他组权值相独立，所以为了方便描述，我们把下标 $j$ 去掉，只考虑单个map的更新，其他map的更新方式是相同的过程，只是索引 $j$ 不同。

softmax函数的导数：
$$
\frac{\partial a_k}{\partial c_i}=\delta_{ki}a_i-a_ia_k\tag{8}
$$
这里的 $\delta$ 是 kronecker delta，参照公式 (1) 我们可以得到在 $l$ 层误差对 $a_i$ 的偏导：
$$
\frac{\partial E}{\partial a_i}=\frac{\partial E}{\partial u^l}\frac{\partial u^l}{\partial a_i}=\sum_{u,v}(\delta^l \circ (x_i^{l-1}*k_i^l))_{uv}
$$
这里的$\delta^l$对应具有输入 $u$ 的输出map的敏感度map。和前面一样，这里的卷积运算也是“valid”类型，目的是使结果和sensitivity map大小匹配。最后使用链式法则计算损失函数对权值 $c_i$ 的偏导数：
$$
\begin{align}
\frac{\partial E}{\partial c_i}&=\sum_k\frac{\partial E}{\partial a_k}\frac{\partial a_k}{\partial c_i}\tag{9}\\
&=a_i\left( \frac{\partial E}{\partial a_i}-\sum_k\frac{\partial E}{\partial a_k}a_k\right)\tag{10}
\end{align}
$$
 3.3.1 Enforcing Sparse Combinations

为了给 $a_i$ 增加稀疏约束（限制一个输出map只与某些而不是全部输入map相连接），我们在代价函数中添加正则项惩罚 $\Omega(a)$ 。这样就可以使某些权值趋于0，最后只有部分输入maps参与输出map相连接，代价函数为：
$$
\widetilde{E}^n=E^n + \lambda\sum{i,j}|(a){ij}|\tag{11}
$$
然后求这个正则项对 $c_i$梯度的影响：
$$
\frac{\partial\Omega}{\partial a_i}=\lambda sign(a_i)\tag{12}
$$




结合公式 (8) 的结果：
$$
\begin{align}
\frac{\partial\Omega}{\partial c_i}& = \sum_k\frac{\partial\Omega}{\partial a_k}\frac{\partial a_k}{\partial c_i} \\
&=
\lambda\left(|a_i|-a_i\sum_k|a_k|\right)
\end{align}
$$
最后结合公式 (13) 和公式 (9) ，可以求的权重 $c_i$ 的梯度：
$$
\frac{\partial\widetilde{E}^n}{\partial c_i} = \frac{\partial E^n}{\partial c_i} +\frac{\partial \Omega}{\partial c_i}
$$


#### 3.4 加快MATLAB训练速度 Making it Fast with MATLAB

> 与CNN关系不大，不做翻译，可看原文

### 4 实际训练问题 Practical Training Issues (Incomplete)

> 与CNN关系不大，不做翻译，可看原文


$$

$$
