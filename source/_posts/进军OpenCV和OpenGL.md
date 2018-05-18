---
title: 进军OpenCV和OpenGL
date: 2018-05-16 10:54:49
tags: [OpenCV, OpenGL]
categories: 计算机图形学
---
近期开始做OpenCV和OpenGL方向的研究与开发工作了，感谢my的书卡，感谢GitHub和机械工业出版社，带我入门，带我编程，能够从一个门外汉进入这个领域。

---
## OpenCV简介
OpenCV是一个跨平台计算机视觉库，可以运行在Linux、Windows、Android和Mac OS操作系统上运行。在没有做这方面工作之前不太明白OpenCV与OpenGL的区别。现在让我来讲的话，大概可以这么理解：**OpenCV是一个算法库**，它实现了很多图形处理和计算机视觉的通用算法，而**OpenGL是一个图形库**，它实现了一些基本的计算机图形图像，可以方便的开发二维或者三维计算机应用程序。
## 1. Windows下配置OpenCV与VS2015集成环境
**为了配置这个环境，中间遇到了无数的坑，一个一个解决，将这个过程中遇到的问题总结一下，写在这里。**
在做图像处理的时候，频繁使用到OpenCV中的算法，于是想系统的学习一下这方面的知识，买了一本《OpenCV By Example》照着里面的例子学习……
### 1.1 **环境：** 
*Windows10 x64, VS2015, OpenCV3.3.0, Cmake3.11.1*
首先下载OpenCV: [https://opencv.org/releases.html](https://opencv.org/releases.html)
![opencv](./opencv0.png)
Documentation是OpenCV的使用手册，source是源码，Win pack是编译后的opencv自解压包，我们这里可以选择下载源码或者下载编译后的自解压包。二者最基本的区别就是：前者需要我们本地编译，后者可以直接使用。我们将两个文件都下载下来:
![opencv3.3](./opencv1.png)
### 1.2 方法一：自解压包的安装与配置
下载完成后的exe文件直接运行，就是自解压过程
![exe](./opencv2.png)
选择好目标文件夹（**这里注意路径中不要有中文**），解压完成后得到如下文件：
![opencvfile](./opencv3.png)
**环境变量：** 在windows的环境变量中的Path中添加opencv中的bin路径：例如我的是：C:\opencv\build\x64\vc14\bin
**vs配置：** 在配置好环境变量后在VS2015中创建一个C++的win32控制台程序，然后开始配置这个项目的属性，首先在视图->其他窗口 中打开当前项目的属性管理器窗口:
![config](./opencv4.png)
因为我们前面配置的都是x64的环境，所以这里选择x64的Debug属性文件夹
![debug](./opencv5.png)
下面需要修改相关配置：
1. VC++目录：
将下面的内容依次加入到VC++的包含目录(include path)
C:\opencv\build\include
C:\opencv\build\include\opencv
C:\opencv\build\include\opencv2
将下面内容依次加入到VC++库目录(lib path)
C:\opencv\build\x64\vc14\lib
2. 连接器目录：
链接器->输入->附加依赖项
将下面的lib加入到其中:
opencv_world330d.lib
![worldd.lib](./opencv6.png)
这里注意的是，有些地方可能会用到opencv_ts***.lib但是自解压包内是不包含这个lib的，需要我们使用自己编译的程序。
至此，OpenCV的自解压版已经配置完毕，可以直接在工程中调用OpenCV中的函数：
```C++
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

#include "opencv2/core.hpp"
#include "opencv2/highgui.hpp"
using namespace cv;
int mainOne()
{
	Mat color = imread("D:\\WorkSpace\\VSworkspace\\OpenCV\\OpencvBooks\\img\\lena.jpg");
	//图片必须添加到工程目下
	//也就是和test.cpp文件放在一个文件夹下！！！
	imshow("测试程序", color);
	Mat gray = imread("D:\\WorkSpace\\VSworkspace\\OpenCV\\OpencvBooks\\img\\lena.jpg",0);
	//写图像
	imwrite("D:\\WorkSpace\\VSworkspace\\OpenCV\\OpencvBooks\\img\\lena.jpg", gray);
	// 通过opencv函数获取相同像素
	int myrow = color.cols - 1;
	int myCol = color.rows - 1;
	Vec3b pixel = color.at<Vec3b>(myrow,myCol);
	cout << "Pixel value (B,G,R): (" << (int)pixel[0] << "," << (int)pixel[1] << "," << (int)pixel[2] << ")" << endl;
	cout << color.size << endl; 
	// 初始化矩阵 
	Mat m = Mat::eye(5, 5, CV_32F);
	Mat n = Mat::ones(5,5,CV_32F);
	Mat result = m*n;
	cout << m << endl;
	FileStorage fs("test.yml", FileStorage::WRITE);
	fs << "result" << result;
	fs.release();
	imshow("Lena BGR", color);
	imshow("Lena Gray", gray);
	waitKey(0);
	return 0;
}
```
## 1.3 方法二：本地编译再配置
找到前面下载的源码版的opencv，同时下载并安装cmake工具到本机。
然后查看这个文件夹，可以看到有cmakelists文件，这个就是cmake需要用的构建工程的文件
![opencv7](./opencv7.png)
然后选择source code和要build到的文件夹：
![opencv8](./opencv8.png)
点击configure
![opencv9](./opencv9.png)
选择本机的 VS2015 Win64,然后Finish，这个过程是生成VS2015的opencv环境，并检验本机的vs环境对于opencv的必要依赖是否完整，这个过程需要一段时间,完成后，选择你需要build的文件，然后再点击Configure开始构建：
![10](./opencv10.png)
然后点击Generate开始生成链接库，完成后就会发现build文件夹内已经存在了opencv的工程sln。
![11](./opencv11.png)
然后打开这个工程，找到CMakeTargets文件夹内的ALL_BUILD，进行编译：右键->生成，这个过程持续一段时间。编译完成后，再找到CMakeTargets文件夹内的INSTALL工程，再次执行生成，这样整个OpenCV的编译工作就完成了，生成了对应的动态链接库和可执行文件。可以看到在build文件夹内的的install文件夹为编译生成的文件：
![opencv16](./opencv16.png)
我们对比一下编译完成的OpenCV 和官方的自解压版本中的内容：
![opencv17](./opencv17.png)
![opencv18](./opencv18.png)
可以看到编译版本比自解压版多一些链接库和lib文件。但是同时缺少 opencv_world***.dll/opencv_world***.lib
后面的操作就和自解压版相同，如果需要使用编译版本的OpenCV，就需要把环境变量配置成刚刚编译好的内容，同时设置VS中的属性页即可。


---
**PS:** 我在前面的执行cmake的过程中，多勾选了BUILD_openc_world 和 WITH_QT 导致在VS中编译OpenCV源码的过程中出现了很多错误：
![opencv13](./opencv13.png)
![opencv14](./opencv14.png)
![opencv15](./opencv15.png)
至今不知道如何解决，感觉是本机的QT配置有问题。
