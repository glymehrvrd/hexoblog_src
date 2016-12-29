---
title: Neural network sand deeplearning学习笔记2
date: 2016-12-19 10:20:29
tags:
- Machine Learning
- Deep Learning
- Notes
---

[第一章](/2016/12/15/machine_learning/neural-network-sand-deeplearning学习笔记1/)中提出可以使用SGD来训练神经网络，但是其中存在一个关键的问题：如何计算梯度$\nabla C$？直接对所有参数求偏导显然是不现实的，计算量太大。这时就要靠Backpropagation算法了。

<!-- more -->

## Backpropagation算法
只给结论，
$$
{% raw %}
\small
\begin{align}
\delta^L &= \frac{\partial C}{\partial z^L} \tag{BP1} \\[0.6em]
\delta^l &= ((w^{l+1})^T \delta^{l+1}) \odot \sigma'(z^l) \tag{BP2} \\[0.6em]
\frac{\partial C}{\partial b^l_j} &= \delta^l_j \tag{BP3} \\[0.6em]
\frac{\partial C}{\partial w^l_{jk}} &= a^{l-1}_k \delta^l_j \tag{BP4}
\end{align}
{% endraw %}
$$

推导过程，已知，
$$
{% raw %}
\begin{align*}
\delta^l &= \frac{\partial C}{\partial z^l} \\[0.6em]
z^l &= w^l \cdot a^{l-1} + b^l
\end{align*}
{% endraw %}
$$
则BP2
$$
{% raw %}
\begin{align*}
\delta^{l-1}_k &= \frac{\partial C}{\partial z^{l-1}_k} \\
&= \frac{\partial C}{\partial a^{l-1}_k} \cdot \frac{\partial a^{l-1}_k}{\partial z^{l-1}_k} \\
&= (\sum_j \frac{\partial C}{\partial z^l_j} \cdot w^l_{jk}) \frac{\partial a^{l-1}_k}{\partial z^{l-1}_k} \\
&= \sum_j \delta^l_j w^l_{jk} \frac{\partial a^{l-1}_k}{\partial z^{l-1}_k}
\end{align*}
{% endraw %}
$$

使用Backpropagation算法，可以使用一次feedforward和backward后算出全部的偏导数，极大地减轻了运算的复杂度。

## Cost函数
从BP算法可以看到，其实每一次我们都只能计算一个输入$x$的所有参数的偏导数。因此Cost函数必须满足，
$$
C = \frac{1}{n} \sum_x C_x
$$
这样，我们才可以从单输入的偏导数计算中得到多输入的偏导数。

其次，Cost函数也必须是输出$a^L$的函数。

## BP算法的Intuitive Explanation以及来源
前人是怎么想出BP算法的？具体看原文吧，有点长。简而言之就是从参数偏导数的链式法则得到的最原始的形式中慢慢简化得来。