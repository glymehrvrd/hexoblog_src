---
title: Neural network sand deeplearning学习笔记1
date: 2016-12-15 17:19:49
tags:
- Machine Learning
- Deep Learning
- Notes
---

--->[Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/)

这是一位机器学习大牛`Michael Nielsen`写的书，在网络上可以免费的学习。以识别MNIST手写数字为问题，从最基础的神经网络模型开始逐渐深入，一直讲到最前沿的深度学习算法。整个过程没有使用外部库，而是使用Python一点一点的搭建出完整的深度学习框架，非常适合想要自学深度学习的同学。

<!-- more -->

## 概念
首先，什么是神经网络？什么又是深度学习？深度学习和神经网络有着什么样的关系？

神经网络源自于科学家对大脑学习方式的研究，使用神经网络模型可以让计算机获得类似于人类的学习能力。而深度学习算法，则是一系列让神经网络快速、有效的学习的技术。

总体而言，神经网络是一种模型，这种模型十分强大，拥有自我学习的能力。但是这套模型参数太多难以调校，而深度学习算法正是用以调教神经网络模型的工具。因此神经网络配合深度学习算法威力十分强大，也掀起了AI的浪潮。

## Perceptron
神经网络的最基础单元是神经(neuron)，每个神经都做简单的、相同的工作。在最初的神经网络模型中，基础单元叫perceptron，perceptron做的工作是输出二元信息，
$$
\begin{eqnarray}
  \mbox{output} = \left\\{ 
    \begin{array}{ll} 
      0 & \mbox{if } w\cdot x + b \leq 0 \\\\
      1 & \mbox{if } w\cdot x + b > 0
    \end{array}
  \right.
\tag{1}\end{eqnarray}
$$
因此它做的工作类似于AND,OR等逻辑操作。

## Sigmoid neuron
Perceptron是既让人兴奋也让人失望的。让人兴奋的是它和universal gate一样可以实现任何Logic，让人失望的是它和universal gate一样需要人为设计。

而神经网络为什么区别于其他的逻辑门？因为它可以通过**自行学习**来解决问题，而不需要像逻辑电路一样依赖于人为的设计。

怎么样才能够自我学习呢？对比一下人类的学习过程，比如老师教我们识别手写数字。首先给你看一个手写数字`9`，但你可能错误的把它认成了`8`，这时老师会纠正你，然后再次看到这个数字时，我们应该就不会再犯相同的错误了。也就是说人类的学习是一个反馈的过程，我们总是从失败中吸取教训，优化我们的神经网络以期待它下次能给出正确的结果。另一方面，人类的学习是一个相对缓慢的过程，要不也不会有人老犯相同的错误了。

我们期待神经网络也具有类似的特征，**较小的参数调整能够导致较小的输出变化**。Perceptron显然不满足这个要求，因为它是binary输出的，是跳跃性的变化。于是引出sigmoid neuron。

Sigmoid neuron的输出为
$$
\sigma(z)=\frac{1}{1+e^{-z}} \\\\
z=w \cdot x + b
$$
它是一个连续变化的过程，而且$z$很大或者很小时，它和perceptron的表现也类似输出binary。

## Intuitive explaination
神经网络究竟在做啥？有没有直观一点的解释？

神经网络的每个neuron可以认为只处理做任务的一小部分，而每一层可以看做是将任务进行不断地分解。比如在识别数字`0`中，第一层的某些neuron可能专门识别`0`左上角的弧度，某些neuron专门识别`0`右下角的弧度。然后下一层neuron收集这些简单的识别结果进行推论，比如存在四个弧度的，可能就是`0`。

这样看来，层越多越好，每一层neuron越多越好，但实际情况并不是这样的，neuron数量太多会导致参数难以调校，性能可能反而变差。

这些解释都是科学家们猜想的，究竟事实是不是这样还是一个谜。就和我们的大脑的工作原理一样，也没人能够给个准确的结论。

## Gradient descent
梯度下降法是求解神经网络参数的最基本方法。原理就是参数的小变化引起结果误差（Cost）的小变化，那么就计算梯度$\nabla C$，寻找到使结果误差最小的参数。Gradient descent需要选择步长$\lambda$，而且要梯度函数是convex的时候才能找到全局最优解。因此Gradient descent不是万能药，使用它训练出来的神经网络性能不一定好。

对于sigmoid neuron输出，使用cross-entropy作为误差函数；对于linear neuron输出，使用MSE作为误差函数。

然后针对Gradient descent比较慢的问题，使用SGD, Stochastic gradient descent。就是不求所有输出误差的梯度，而选择几个样本求出梯度，更新一次参数，再选择几个样本，再次更新...这样把所有样本用完，叫做完成一个epoch。

## 为什么神经网络好？
`sophisticated algorithm ≤≤ simple learning algorithm + good training data.`

就手写数字识别问题而言，如果不使用机器学习算法，那么这个问题将是相当复杂的。而使用了神经网络模型后，可以使用简单的算法（SGD）和训练数据训练出一个优秀的算法解决这个问题。这就是神经网络的优越性，让计算机自己学习如何去解决问题，而不是人为的去编写算法。