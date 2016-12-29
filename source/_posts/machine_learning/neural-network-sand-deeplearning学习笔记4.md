---
title: Neural network sand deeplearning学习笔记4
date: 2016-12-29 20:05:00
tags:
- Machine Learning
- Deep Learning
- Notes
---

在[第一章](/2016/12/15/machine_learning/neural-network-sand-deeplearning学习笔记1/)中，说明了神经网络可以**近似**任何函数。在[chap4](http://neuralnetworksanddeeplearning.com/chap4.html)直观的解释了为什么神经网络能够**近似**任何函数。

<!-- more -->

另外插一句，神经网络可以近似任何**函数**，也就是说一个输入可以训练处任何输出。而`Recurrent Neural Network`可以近似任何**程序**，可以生成任意序列，因此更为强大。

在[chap5](http://neuralnetworksanddeeplearning.com/chap5.html)中，解释了为什么全连接的深度神经网络不容易训练。

从宏观上讲，全连接的深度神经网络参数太多。参数越多，模型越容易过拟合。

从微观上讲，全连接的深度神经网络越是底层（远离输出层），学习速度越慢。

由梯度方程可以看出，底层的梯度包含了很多$w_i\sigma'(z_i)$连乘项。对于`sigmoid`激活函数而言，这些项往往是小于1的，因此这就带来了`vanishing gradient problem`；当然也有可能大于1，这样就是`explding gradient problem`。总之，梯度是不稳定的（unstable）。这个问题给深度学习带来了很大的麻烦。

除此之外，作者还提到了一些其他的问题：

1. 使用`sigmoid`激活函数，最后一层隐含层容易在`0`附近饱和，导致学习速度变慢。
2. 参数初始化，以及momentum规划都对深度学习有着实质性的影响。

还有其它很多，总之，通往深度学习的道路是坎坷的。我们需要重新审视之前的很多决策，比如激活函数的选取、权重参数的初始化、网络模型等等。
