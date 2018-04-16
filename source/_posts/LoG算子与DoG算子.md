---
title: LoG算子与DoG算子
date: 2018-04-16 14:07:01
tags: [边缘检测算法,算子,滤波]
categories: 特征提取
mathjax: true
---
## 简介
图像中灰度变化较大的非连续像素可以看做是边缘，边缘是最为重要的图像特征之一，在目标检测、追踪、识别中都必不可少的使用到了边缘，人类视觉系统也对边缘信息非常敏感。
边缘检测的一般步骤如下：
1. 滤波（去噪）
2. 增强（一般是通过计算梯度幅值）
3. 检测（在图像中有许多点的梯度幅值会比较大，而这些点并不都是边缘，所以应该用某种方法来确定边缘点，比如最简单的边缘检测判据：梯度幅值阈值）
4. 定位（有些应用场合要求确定边缘位置，可以在子像素水平上来估计，指出边缘的位置和方向）

边缘检测方法比较常用的有基于各种算子的方法，有基于一阶导数的各种算子（Roberts、Sobel、Canny、Prewitt等），还有基于二阶导数的Laplacian算子（LoG）等。

LoG边缘检测算子是David Courtnay Marr和Ellen Hildreth（1980）共同提出的。因此，也称为边缘检测算法或Marr & Hildreth算子。该算法首先对图像做高斯滤波，然后再求其拉普拉斯（Laplacian）二阶导数。即图像与 Laplacian of the Gaussian function 进行滤波运算。最后，通过检测滤波结果的零交叉（Zero crossings）可以获得图像或物体的边缘。因而，也被业界简称为Laplacian-of-Gaussian (LoG)算子。

## Marr和Hildreth证明了以下两个观点：
1. 灰度变化与图像尺寸没有关系，因此检测需要不同尺度的算子
2. 灰度的突然变化会在一阶导数中引起波峰和波谷，或者二阶导数中一起零交叉

其中一阶导数一般找梯度极值。二阶导数找过零点（需要忽略无意义的过零点（即均匀零区））。

## LoG边缘检测算法步骤：

1. 平滑：高斯滤波器
2. 增强：Laplacian算子计算二阶导
3. 检测：二阶导零交叉点并对应于一阶导数的较大峰值
4. 定位：线性内插

根据卷积的求导法则，先卷积后求导和先求导后卷积是相等的，所以可以把第1、2步合并为一步，先对高斯滤波器做拉普拉斯变换，得到LoG算子，然后再用这个算子与图像做卷积。

### 探测器
灰度的突然变化会在一阶导数中引起波峰或者波谷，或者在二阶导数中等效的引起零交叉。
* 一阶微分探测器
从数学上讲，像素的灰度值变化速率，可以用一阶微分来检测，对于二维离散函数，其微分可以表示为：
$$\frac{\partial f(x,y)}{\partial x} = f(x+1,y) - f(x,y)\qquad$$
$$\frac{\partial f(x,y)}{\partial y} = f(x,y + 1) - f(x,y)\qquad$$
* 二阶微分检测器
二阶微分同样可以如一阶微分一样，检测到函数的变化。二阶微分其实就是拉普拉斯算子，一般使用下面的方法表示：（两个方向）

$$\nabla^{2}f = \frac{\partial^{2}f}{\partial{x^{2}}} + \frac{\partial^{2}f}{\partial{y^{2}}}$$
为了适应图像处理，我们将拉普拉斯算子写成离散形式，其中
$$\frac{\partial^{2}f}{\partial{x^{2}}} = f(x+1,y)+f(x-1,y)-2f(x,y)$$
$$\frac{\partial^{2}f}{\partial{y^{2}}} = f(x,y+1)+f(x,y-1)-2f(x,y)$$

最终结果为:(两个方向作和)
\begin{equation}
\begin{aligned}
\frac{\partial^{2}f}{\partial{x^{2}}} + \frac{\partial^{2}f}{\partial{x^{2}}}&= f(x+1,y)+f(x-1,y)-2f(x,y) + f(x+1,y)+f(x-1,y)-2f(x,y)\\&= f(x+1,y)+f(x-1,y) + f(x+1,y)+f(x-1,y) - 4f(x,y)
\nonumber
\end{aligned}
\end{equation}

所以拉普拉斯算子在图像处理中的模板为：
$$
 \left[
 \begin{matrix}
   0 & 1 & 0 \\
   1 & -4 & 1 \\
   0 & 1 & 0
  \end{matrix}
  \right]
$$
拉普拉斯算子对噪声和离散点极为敏感，所以在利用其进行边缘检测的时候，需要首先对图像进行平滑，除去噪声的影响。因为高斯运算和拉普拉斯运算可以叠加，我们可以将其的最终形式写为：
![LoG公式推导](./LoG.png)
最终简化为:
![最终](./LoG公式最终形式.png)
MatLab中的样子
```Matlab
laplace_gaussian_filter = fspecial('log',[40 40],4.5);
subplot(121)
surf(laplace_gaussian_filter);
subplot(122)
surf(-laplace_gaussian_filter);
```
![LoG](./LoG算子.png)
## 参考：
* [边缘检测滤波器](https://www.jianshu.com/p/2ac784fd22fc)