---
title: 【OpenCV_初识】环境配置
date: 2018-04-08 16:04:08
tags:
	- OpenCV
	- Computer vision
---

&emsp;&emsp;心里没有 AC 数的我，又要开始开新坑了。这一系列文章的坑是为了学习 OpenCV 而开。

&emsp;&emsp;本篇文章介绍了在 Mac 环境下 OpenCV + CLion 的配置。

<!--more-->

&emsp;&emsp;其实很简单，因为有 brew 的缘故，只需要简单的一条命令就可以了。

```shell
brew install opencv
```

&emsp;&emsp;之后，你可以喝杯茶冷静一下，等待它安装结束。安装结束后 opencv 的 include 和 lib 文件夹会分别在 `/usr/local/Cellar/opencv/x.x.x_x/include` 和 

`/usr/local/Cellar/opencv/x.x.x_x/lib`，可以在 Clion 项目里添加头文件和库的依赖，如下：

```c++
cmake_minimum_required(VERSION 3.10)
project(OpenCVDemo1_OpenImage)

set(CMAKE_CXX_STANDARD 11)

include_directories(/usr/local/Cellar/opencv/3.4.1_2/include)

add_executable(OpenCVDemo1_OpenImage main.cpp)

target_link_libraries(OpenCVDemo1_OpenImage /usr/local/Cellar/opencv/3.4.1_2/lib/libopencv_core.dylib)
target_link_libraries(OpenCVDemo1_OpenImage /usr/local/Cellar/opencv/3.4.1_2/lib/libopencv_imgproc.dylib)
target_link_libraries(OpenCVDemo1_OpenImage /usr/local/Cellar/opencv/3.4.1_2/lib/libopencv_highgui.dylib)
target_link_libraries(OpenCVDemo1_OpenImage /usr/local/Cellar/opencv/3.4.1_2/lib/libopencv_imgcodecs.dylib)

```

&emsp;&emsp;之后，就可以写代码了。下边有一份测试代码：

```c++
#include <iostream>
#include <opencv2/opencv.hpp>
#include <opencv2/highgui.hpp>

using namespace std;
using namespace cv;

int main(int argc, char *argv[]) {

    Mat im(300 , 300 , CV_8UC1);

    if (im.empty()) {
        cout << "Can not create the image" << endl;
        return -1;
    }
    imshow("CV", im);
    waitKey(0);
}
```

&emsp;&emsp;简单介绍下这几行代码：

&emsp;&emsp;`Mat im(300 , 300 , CV_8UC1)` 创建了一个长宽 300，单通道分量的图片。（关于图片可能会在下篇介绍，暂时知道是创建图片就好）

&emsp;&emsp;`imshow("CV" , im)` 则是创建了一个窗口来显示图片，由于我们的图片比较简单，所以运行后就是一个简单的黑色窗口。

![1](https://image.ibb.co/bT7ngc/image.png)

