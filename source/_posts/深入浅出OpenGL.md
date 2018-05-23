---
title: 深入浅出OpenGL
date: 2018-05-21 09:00:36
tags: [OpenGL,图形图像]
categories: OpenGL编程
---
<!-- omit in toc -->
# OpenGL概述
## 什么是OpenGL？
OpenGL是一种应用程序的编程接口(Application Programming Interface, API)，它是一种可以对图形硬件设备特性进行访问的软件库。
《OpenGL编程指南》
## OpenGL的工作流：
OpenGL主要工作就是将二维及三维物体绘制到帧缓存器中
### 主要步骤：
1. 构造几何要素（点，线，多边形），创建对象的数学描述。在三维空间中放置对象。
2. 计算对象的颜色，颜色可以给出，或由光照条件及纹理间接给出。
3. 光栅化，把对象的数字描述和颜色信息转换为屏幕上的像素。
