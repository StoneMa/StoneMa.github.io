---
title: HarrisCornerDetection
date: 2018-04-10 16:50:40
tags: [特征提取, 图形, 图像]
categories: 图像处理
---
## 上一篇文章介绍了什么是特征，本篇介绍一下如何提取特征（角点特征）
上一篇文章当中，我们学习到了对于一个图像，图像的角落是比较适合作为特征的区域，是各个方向上强度变化较大的区域。早在一九八八年（1988年老子还特么没出生呢！人家就开始研究这些东西了。。。这个领域到底跟美国有多大差距啊！！），Chris Harris＆Mike Stephens就在其论文《A Combined Corner and Edge Detector》中发现了这些角落的一个早期尝试，现在它被称为Harris Corner Detector。他把这个简单的想法变成了一种数学形式。而且它基本上找到了（u，v）在各个方向上位移后的亮度差。这个表达式如下：

$$
E(u,v) = \sum_{x,y}\underbrace{w(x,y)}_{window function}[\underbrace{I(x+u,y+v)}_{shifted intensity} - \underbrace{I(x,y)}_{intensity}]^2
$$

Window function是给对应的 $x,y$ 像素加权的矩形窗口或高斯窗口