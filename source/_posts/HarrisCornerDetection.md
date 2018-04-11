---
title: HarrisCornerDetection
date: 2018-04-10 16:50:40
tags: [特征提取, 图形, 图像]
categories: 图像处理
mathjax: true
---
## 上一篇文章介绍了什么是特征，本篇介绍一下如何提取特征（角点特征）
上一篇文章当中，我们学习到了对于一个图像，图像的角落是比较适合作为特征的区域，是各个方向上强度变化较大的区域。早在一九八八年（1988年老子还特么没出生呢！人家就开始研究这些东西了。。。这个领域到底跟美国有多大差距啊！！），Chris Harris＆Mike Stephens早期在其论文《A Combined Corner and Edge Detector》中就尝试去发现了这些角落，并把它称为Harris Corner Detector。他把这个简单的想法变成了一种数学形式。而且它基本上找到了（u，v）在各个方向上位移后的亮度差。这个表达式如下：

$$E(u,v) = \sum_{x,y}\underbrace{w(x,y)}_{window function}[\underbrace{I(x+u,y+v)}_{shifted intensity} - \underbrace{I(x,y)}_{intensity}]^2$$

Window function是给对应的 $x,y$ 像素加权的矩形窗口或高斯窗口。对于角点检测，我们必须使得$E(u,v)$取得最大值。这样就意味着，对于上面的公式，我们必须使得等号后的第二项最大化（中括号内的项），运用泰勒展开式并使用一些数学过程，我们能够最终得到下列方程：

$$
E(u,v) \approx [u,v] M
\left[
 \begin{matrix}
   u\\
   v\\
  \end{matrix}
\right]
$$
同时：

$$
M = \sum_{x,y}(x,y)
\left[
 \begin{matrix}
   I_{x}I_{x}&I_{x}I_{y}\\
   I_{x}I_{y}&I_{y}I_{y}\\
  \end{matrix}
\right]
$$

这里$I_{x}$ 和 $I_{y}$ 是图像在 $x$ 和 $y$ 方向上的导数，（能够简单的通过``cv.Sobel()``方法得到）。

然后是主要部分，这帮人又创建了一个分数，它将决定一个窗口是否可以包含角点。

$$
R = det(M) - k(trace(M))^2
$$
and
* $det(M) = \lambda_{1}\lambda_{2}$
* $trace(M) = \lambda_{1} + \lambda_{2}$
* $\lambda_{1}$ and $\lambda_{2}$ 是M的特征值