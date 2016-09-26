---
title: 让Chrome打印适应Kindle大小的PDF
date: 2016-09-24 11:41:00
categories: technology
tags: Kindle
---

最近发现一本深度学习的好书 [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/)，想保存下载在Kindle上面阅读。

但是书中包含大量由MathJAX渲染的公式，无论是直接保存html或是另存为word，在Kindle上的阅读体验都比较糟糕。只有pdf格式才能完美的显示公式，于是想到使用chrome将网页保存为pdf格式。

首先使用`开发者工具`删掉网页中多余的元素，只保留书本内容，然后使用chrome的`打印pdf`功能将网页保存为pdf格式。

但chrome只能打印A4,A3等几种大小的pdf，这些大小在kindle上阅读体验也不好。

最适合kindle pw1的大小是 `85mmx114mm`。通过在CSS样式表里添加代码控制chrome打印pdf的大小，
```CSS
@media print{
    @page { size:85mm 114mm; margin-left: 2mm; margin-right: 2mm }
}
```
然后打印、保存、上传到Kindle上，效果就很棒啦。

![result_pdf](/images/chromekindlepdf.png)