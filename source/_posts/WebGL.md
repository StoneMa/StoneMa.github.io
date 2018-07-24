---
title: WebGL
date: 2018-07-18 13:46:17
tags: [webgl, Three.js,前端]
categories: [图形学]
---

# WebGL基础与应用

新的技术啊，没人引导，自己啃的话，一开始的时候真的会觉得无从下手，尤其新的领域的内容，有时会感觉力不从  心，但是相信功夫不负有心人，只有肯努力，一定能在这个领域有所建树的，加油！

对于webGL，它实际上是OpenGL的web实现，是HTML5标准之下，新出的一套web呈现技术，能够在浏览器端实现以前只有在桌面上才能实现的复杂图形图像。

通过这篇文章，来介绍学习过程，解决实际问题。
**这次的问题是：如果将三维模型标准化，归一化，正常的显示在显示器上**

## 模型矩阵，视图矩阵和投影矩阵
PS:这部分内容来源于[model_view_projection](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/WebGL_model_view_projection)
通常用于表示3D模型对象的三个核心矩阵就是：模型矩阵，视图矩阵，以及投影矩阵。下面我们详细介绍每个矩阵的用途，以及三者的联系。

空间中点和多边形的基本变换（如平移，缩放和旋转）由各种变换矩阵来处理。这些矩阵可以组合在一起，使它们可用于渲染复杂的3D场景。这些组合矩阵最终将原始模型数据移动到称为 **剪辑空间(Clipspace)** 的特殊坐标空间中。这是一个2单位宽的立方体。中心坐标为（0,0,0），而角点范围为（-1，-1，-1）至（1,1,1）。此剪辑空间被压缩到2d空间并光栅化为图像。
1. 模型矩阵
   模型矩阵定义了如何获取原始模型数据，以及如何在三维世界坐标系中移动模型（ which defines how you take your original model data and move it around in 3d world space.）下一步，为了获取世界坐标系中的坐标，以及将模型移动到剪辑空间中（剪辑空间其实就是我们的可视区域），我们需要投影矩阵，而常用于投影的矩阵是透视矩阵（perspective matrix）它模仿的就是照相机的原理。最后如果需要移动相机，就又需要一个视图矩阵，用来移动相机。
2. 剪辑空间（裁剪空间 Clipspace）
   剪辑空间也就是我们的可视区域，任何数据位于剪辑空间外的话，则会被剪掉，并且不会渲染。在一个WebGL程序中，模型数据通常会以它自己的坐标系上传到GPU，然后顶点着色器会将这些点转换到不同的坐标系系统下进行渲染。
   ![clipspace](./clip-space-graph.svg)
   上图是所有点必须适合的剪辑空间的可视化。它是2个单位宽，由角（-1，-1，-1）到角（1,1,1）的立方体组成。立方体的中间是点（0,0,0）。
   这个空间就是剪辑空间或者叫裁剪空间。裁剪空间的目标是能够方便地对渲染图元进行裁剪：完全位于这块空间内部的图元将会被保留，完全位于这块空间外部的图元将会被剔除，而与这块空间边界相交的图元就会被裁剪。。那么，这块空间是如何决定的呢？答案是由视锥体(view frustum)来决定。
3. 两种投影模式
   视锥体指的是空间中的一块区域，这块区域决定了摄像机可以看到的空间。视锥体由六个平面包围而成，这些平面也被称为 **裁剪平面(clip planes)**。视锥体有两种类型，这涉及两种投影类型：一种是 **正交投影(orthographic projection)**，一种是 **透视投影(perspective projection)**。
   透视投影:
   ![透视投影](./perspective_projection.png)
   
   正交投影:
   ![正交投影](./orthographic_projection.png)

## 世界坐标系（World Coordinate System）
世界坐标系，我们可以看做是一个永恒不变的参考系，所以的其他模型，和各种坐标系都以这个坐标系作为基准。世界坐标系的位置不随模型的变化而改变。

## 模型坐标系 (Model Coordinate System)
模型坐标系是建模时所用到的坐标系，模型的原点为(0,0,0),但是它可以是世界坐标系中的任意一点。
通常在``Three.js``中，我们设置``Object.position.set(0,0,0)``就是将模型的原点设置为世界坐标系的原点。
* 模型中心设置
    ``object.children[i].geometry.center(); //将网格模型的中心移动到世界坐标系的中心`` 这样做，对于单一Mesh构成的三维模型能够轻松将模型移动到世界坐标系中央，但是对于多Mesh构成的模型来讲，每个children的Mesh都会被移动到World坐标系原点。

## 如何将加载后处于偏移位置的模型移动到坐标系原点呢？

![原始图](./original.png)
可以看到上图中的模型，虽然模型坐标系和世界坐标系重合，但是模型的原点却位于三维物体的一个角落，看起来不美观。我们起初的办法是通过设置``object.children[i].geometry.center()``将三维物体的中心移动到模型坐标系原点，但是刚好上图中的模型是单个Mesh构成的三维物体，所以没有发现问题，当使用多Mesh三维模型的时候，上面的方法就会把所有的Mesh移动的模型坐标系中心，使得模型的Mesh混乱，无法显示原来的样子。
![Mesh混乱](./building.png)

为了修复这个问题，想到的解决办法是：计算三维模型AABB包围盒两个对角点的位置坐标相应方向和的均值，得到的均值后，将模型的position设置到这个点，从而达到将三维物体中心放置到坐标系原点的目的。
1. 加载三维模型
    ```javascript
        objLoader = new THREE.OBJLoader();
        objLoader.setPath('./obj/');
        objLoader.load('a.obj', function (object) {
        oneObj = object;
        object.traverse(function (child) {
            if (child.type === "Mesh") {
                child.geometry.computeBoundingBox();
                child.geometry.verticesNeedUpdate = true;
                child.material.side = THREE.DoubleSide;
                //child.geometry.center(); //设置模型中心点为几何体的中心
            }
        });
        });
    ```
2. 构造AABB包围盒
    ```javascript
        box = new THREE.BoxHelper(object);
    ```
3. 计算包围盒任意两个对角点的均值，并移动模型
    ```javascript
        var points = box.geometry.attributes.position.array;
        obj_x = (points[0]+points[18])/2;
        obj_y = (points[1]+points[19])/2;
        obj_z = (points[2]+points[20])/2;
        oneObj.position.set(-obj_x,-obj_y,-obj_z); //这是移动模型时要反向移动
    ```
![moving](./moveBuilding.png)

这样，就解决了模型便宜的问题，但是模型的大小适配问题还在，我们的思路是：获取到模型表面的距离最远的两个点，然后保证这个距离是小于``camera``的``far-near``的，同时这个长度大于``far-near`` x 倍，就相应缩小 n*x倍，n是个放大系数，这里我取值是20。
实现过程如下：

```javascript
    // 获取三维模型表面点的最大距离
    var box3 = new THREE.Box3();
    box3.setFromObject(object);
    var maxLength = box3.getSize(new THREE.Vector3()).length();
    var consult = (camera.far - camera.near) / (20*maxLength);
    //写入场景内
    var currentScale = consult;
    object.scale.set(currentScale,currentScale,currentScale);
``` 

## 实现过程
完整实现过程：

```javascript
        //  objLoader
        objLoader = new THREE.OBJLoader();
        objLoader.setPath('./obj/');
        objLoader.load('a.obj', function (object) {
            oneObj = object;
            object.traverse(function (child) {
                if (child.type === "Mesh") {
                    child.geometry.computeBoundingBox();
                    child.geometry.verticesNeedUpdate = true;
                    child.material.side = THREE.DoubleSide;
                    //child.geometry.center(); //设置模型中心点为几何体的中心
                }
            });
            object.name = "zxj";  //设置模型的名称
            //对模型的大小进行调整

            // object.scale.x =0.001;
            // object.scale.y =0.001;
            // object.scale.z =0.001;
            //object.lookAt(new THREE.Vector3(0,0,0));
            // 加入模型的aabb包围盒
            // 获取三维模型表面点的最大距离
            var box3 = new THREE.Box3();
            box3.setFromObject(object);
            var maxLength = box3.getSize(new THREE.Vector3()).length();
            var consult = (camera.far - camera.near) / (20*maxLength);
            //写入场景内
            var currentScale = consult;
            object.scale.set(currentScale,currentScale,currentScale);
            box = new THREE.BoxHelper(object);
            // box.material.transparents = true;
            // box.material.depthTest = false;
            // box.visible = true;
            var points = box.geometry.attributes.position.array;
            obj_x = (points[0]+points[18])/2;
            obj_y = (points[1]+points[19])/2;
            obj_z = (points[2]+points[20])/2;
            oneObj.position.set(-obj_x,-obj_y,-obj_z);
            //scene.add(box);
            scene.add(object);
        });
```

# 裁剪空间(clipspace)
 顶点接下来要从观察空间转换到裁剪空间(clip space，也被称为齐次裁剪空间)中，这个用于转换的矩阵叫做裁剪矩阵(clip matrix)，也被称为投影矩阵(projection matrix)。