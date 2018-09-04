---
title: 【OpenGL】8-变换（试用矩阵运算实现平移，旋转，缩放）
date: 2018-02-19 22:04:17
tags: 
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[变换](https://learnopengl-cn.github.io/01%20Getting%20started/07%20Transformations/#_18)

【英文版】[Transformations](http://learnopengl.com/#!Getting-started/Transformations)

#### 学习目录

1. 使用GLM数学库进行模型矩阵的平移
2. 使用GLM数学库进行模型矩阵的缩放
3. 使用GLM数学库进行模型矩阵的旋转

<!--more-->

#### 学习心得

&emsp;&emsp;这篇文章中关于矩阵的计算（加减乘除等等）参考：[矩阵运算](http://www2.edu-edu.com.cn/lesson_crs78/self/j_0022/soft/ch0605.html)

##### &emsp;平移

&emsp;&emsp;首先看平移，在此我们使用上一篇文章中的项目作为基础来进行立方体的平移。我们的立方体如下：

![ProjectionWX20180219-221106@2x](https://ws1.sinaimg.cn/large/006tNc79ly1fom63n6nvjj31ks16igoy.jpg)

&emsp;&emsp;在此我们注意到之前代码中的MVP变量（Model,View,Projection），三个矩阵分别代表模型，视图，投影，而今天的操作平移是针对模型矩阵。首先修正我们摄像机的位置，将其改为在Z轴上方，即`0,0,5`位置

```c++
    // Camera matrix
    glm::mat4 View = glm::lookAt(
            glm::vec3(0, 0, 5), //camera location
            glm::vec3(0, 0, 0), // and looks at the origin
            glm::vec3(0, -1, 0)  // Head is up (set to 0,-1,0 to look upside-down)
    );
```

&emsp;&emsp;其次修改投影矩阵，将视角放大一些（将`glm::radians(45.0f)`修改为`glm::radians(90.0f)`），即可见区域变大，这样我们在进行平移的时候可以平移更多距离不至于离开视线。

```c++
glm::mat4 Projection = glm::perspective(glm::radians(90.0f), 4.0f / 3.0f, 0.1f, 100.0f);
```

&emsp;&emsp;这时我们的视角将变成从Z轴往下看，之后的平移旋转操作将围绕X和Y轴进行，这是我们转换摄像机后的运行截图：

![WX20180219-223829@2x](https://ws2.sinaimg.cn/large/006tNc79ly1fom63mmnxtj31jy158abj.jpg)

&emsp;&emsp;由于我们摄像机位置的X，Y都是0，这时候我们看这个立方体的时候只能看到一个面，看起来像是一个平面正方形。

&emsp;&emsp;glm库中给我们提供了创建平移矩阵的方法 [glm::translate](https://glm.g-truc.net/0.9.2/api/a00245.html#ga4683c446c8432476750ade56f2537397)，这个api的介绍为

> Builds a translation 4 * 4 matrix created from a vector of 3 components.

&emsp;&emsp;即使用一个vec3来创建平移矩阵（vec3自然是X , Y , Z三轴的值了），举个例子现在要向X轴正方向平移1，则使用`glm::translate`方法应该如下：

```c++
glm::mat4 trans = glm::translate(glm::mat4(), glm::vec3(1.0f, 0.0f, 0.0f));
```

&emsp;&emsp;创建好平移矩阵，将其与Model相乘

```c++
Model = Model * trans;
```

&emsp;&emsp;之后再进行投影，视图，模型的矩阵相乘就可以了。我们将这一段放在我们的主循环里，但是编译运行后发现会直接看不到立方体了，这种情况是因为计算机处理的速度太快，在我们没看到的情况下已经平移出了摄像机可见区域。将vec3里的赋值改为0.001或者0.01即可。

##### &emsp;缩放

&emsp;&emsp;缩放对我们来说也是很简单的，glm也提供了构造缩放矩阵的方法[glm::scale](https://glm.g-truc.net/0.9.2/api/a00245.html#ga6da77ee2c33d0d33de557a37ff35b197)

> Builds a scale 4 * 4 matrix created from 3 scalars.

&emsp;&emsp;使用方法和平移差不多，三个参数即是X，Y，Z，当然我们要是放到主循环的话肯定要将数字设置的接近1一些，这样无论是放大还是缩小都能给我们看到的时间。

```c++
Model = Model * glm::scale(glm::mat4() , glm::vec3(0.999f,0.999f,0.999f));
```

##### &emsp;旋转

&emsp;&emsp;旋转矩阵的构建是比较麻烦的，但是glm已经提供给了我们可用的api，即[glm::rotate](https://glm.g-truc.net/0.9.2/api/a00245.html#ga48168ff70412019857ceb28b3b2b1f5e)

> Builds a rotation 4 * 4 matrix created from an axis vector and an angle expressed in degrees.

&emsp;&emsp;由这句介绍可以看到，我们所需要给出的参数是X，Y，Z的坐标和角度，即绕着哪一个轴旋转，假如现在要绕着X轴旋转，即：

```c++
Model = Model * glm::rotate(glm::mat4() , glm::radians(90.0f)/1000, glm::vec3(-1.0f, 0.0f, 0.0f));
```

&emsp;&emsp;这里我们使用了[glm::radians](https://glm.g-truc.net/0.9.4/api/a00136.html#ga4fb76e28851c9ff6653532566084e091)，是一个转换函数，将角度转换为弧度，之所以给转换后的弧度值除以1000也是因为要让他旋转的慢一些，可以看得更清楚。