---
title: Neural network sand deeplearning学习笔记3
date: 2016-12-19 14:57:29
tags:
- Machine Learning
- Deep Learning
- Notes
---

这一章主要讲了一些神经网络模型的改进技术。

<!-- more -->

## Cost function
选择一个合适的代价函数（Cost function），能够加快learning的速度。本章中作者介绍了几种代价函数。

### Cross-entropy cost function
在之前使用的代价函数（Cost function）是MSE。但MSE用在对Sigmoid neuron的评价中有个问题，就是初始误差比较大时学习的很慢。

这个直观的看，就是因为$z$较大时，$\sigma(z)$对$z$不敏感。数学上分析，是因为在偏导数中，存在$\sigma'(z)$这一项。

为了解决这个问题，就要想办法消去$\sigma'(z)$。尝试令偏导数，
$$
{% raw %}
\begin{eqnarray} 
  \frac{\partial C}{\partial w_j} & = & x_j(a-y) \\
  \frac{\partial C}{\partial b } & = & (a-y)
\end{eqnarray}
{% endraw %}
$$
然后积分，得到代价函数
$$
C = -[y \ln a + (1-y) \ln (1-a)]+ {\rm constant}
$$
在考虑有多个样本的情况，得到cross-entropy cost function
$$
C = -\frac{1}{n} \sum_x [y \ln a + (1-y) \ln (1-a)]+ {\rm constant}
$$

值得一提的是，在输出neuron是linear neuron的时候，代价函数应该使用MSE，因为这时偏导数中不包含$\sigma'(z)$这一项，不存在输出饱和时学习的很慢的问题。

### Softmax and Log-likelihood cost function
Softmax方法定义输出层为，
$$
a^L_j = e^{z^L_j} / \sum_k e^{z^L_k}
$$

Softmax输出有个好处，那就是可以认为它的输出是一个概率分布函数。

对Softmax输出，应该使用Log-likelihood cost function
$$
C=-\ln a^L_y
$$
这时，依然可以使用BP算法对其进行求解。

## Overfitting
在机器学习中，过拟合是一个很常见的问题。也就是说算法学习到的是噪声带来的特征（peculiarity），因而不能很好地归纳（generalize）本质的规律。

解决过拟合有几个思路，包括人为增加训练样本、正则化（regularization）等。

### 正则化
正则化rescaling weight before learning，在代价函数中增加weight。正则化有L1,L2,Dropout等方式。

L1,L2原理是尽可能让weight小，让参数少。Dropout原理是随机“删除”某些neurons，然后训练，然后多次重复这个过程，这样训练出来的neuron更robust。

### 人为增加训练样本
可以发现，随着样本数量的增多，过拟合问题会逐步得到解决，因此可以通过人为增加样本的方式解决过拟合。比如MNIST中，就可以对数字进行膨胀，旋转等操作。

## 权重初始化
权重初始化也是一个问题，之前一直使用standard normal distribution初始化权重。但是这样很容易在hidden layer出现饱和，因为隐含层输出将是服从于$N(0,n\_{in})$（假设输入全为1）。

修改权重为$w/\sqrt{n\_{in}}$，这样是的隐含层输入$z$也满足**标准**正态分布，方差变小，初始的输出不容易饱和。

## 高阶参数的选择
高阶参数，包括$\lambda$，隐藏层数，隐藏层神经元数目等。一般高阶参数人工指定，没有太好的统一解决方案。

作者给出了几点建议：
1. 先拿小规模的学习测试，时间快
2. 先看到小成果（trivial）
...等待继续总结

## SGD的替代方法
Hessian方法、Momentum-based etc