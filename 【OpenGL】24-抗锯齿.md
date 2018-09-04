---
title: 【OpenGL】24-抗锯齿
date: 2018-03-13 14:16:29
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[抗锯齿](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/11%20Anti%20Aliasing/)

【英文版】[Anti Aliasing](http://learnopengl.com/#!Advanced-OpenGL/Anti-Aliasing)

#### 学习记录

&emsp;&emsp;在讨论抗锯齿之前，我们需要知道什么是锯齿。我们的屏幕显示效果其实是由一个个像素点显示而成的，就是相当于一个个小方格堆砌而成，而当我们在屏幕上显示一条线的时候，他可能是比较平滑的，因为可以沿着小方格的边缘着色，但是若是斜线呢。

<!--more-->

&emsp;&emsp;试想，在几何中，用边长为1的单位正方形去构造一条与平面呈 45 ° 的斜线，我们最终堆砌而成的是很明显的锯齿状，见下图：

![anti_aliasing_rasterization_filled](https://ws4.sinaimg.cn/large/006tKfTcly1fpbayx4czaj308c08h3yx.jpg)

&emsp;&emsp;可以看到，在与正方形边缘平行的那一条线是比较平滑的，而其他两条就成了那种台阶状，我们又称之为锯齿。而在计算机图形中，所有的图形也是由一个个小方格组成，方格的大小和屏幕分辨率有关，当屏幕分辨率较低的时候，我们可以看到画面上几乎都是一个个的小方格。所以提高分辨率是解决锯齿的最好的办法。

&emsp;&emsp;不过，提高分辨率是硬件的技术，在超高分辨率的屏幕尚未发明出来时，我们也有各种技术来对图像边缘进行处理，以提高画面感。这篇文章就是用来介绍这几门技术。

###### 多重采样技术（MSAA）

&emsp;&emsp;多重采样技术，即在每一个像素点里放置多个采样点，根据最后图像边缘覆盖的采样点多少来觉得色度的深浅。默认一般是一个的，首先我们看看一个的情况：

![anti_aliasing_rasterization](https://ws4.sinaimg.cn/large/006tKfTcly1fpbayzz88pj308c08hmxx.jpg)

&emsp;&emsp;这种情况下，当我们的边缘覆盖到采样点的时候，整个像素都会被绘制成片段的颜色，当没有覆盖的时候即不进行染色。这样下来就会出现边缘的锯齿，如最上边的图那样。而多重采样技术，则是在一个像素点内有多个采样点，覆盖每个采样点则增加点颜色深度，当一个像素内所有的采样点都覆盖时则为片段颜色。例如我们使用四个采样点，某一个像素区域内片段覆盖了三个采样点，则这个像素的颜色是 `fragColor * 0.75` 。

![anti_aliasing_rasterization_samples_filled](https://ws4.sinaimg.cn/large/006tKfTcly1fpbaz1uhswj30b40ba0tx.jpg)



&emsp;&emsp;其实这种技术我们一直在用，还记得我们 OpenGL 代码中的 `glfwWindowHint(GLFW_SAMPLES, 4);` 。这即是开启了赋值了每个像素点内的采样点数量，我们将其设置为 4 。同时在设置采样点数量的时候，我们还需要使用 glEnable(GL_MULTISAMPLE); 来开启（默认开启）多重采样的功能。这样下来 OpenGL 将会按照四个采样点来计算，而不用我们自己去抗锯齿。

&emsp;&emsp;我们将之前的 A Colored Cube 代码拿出来，将其颜色染为纯色，然后注释掉设置采样点的代码，来看看效果：

![WX20180313-165901@2x](https://ws4.sinaimg.cn/large/006tKfTcly1fpbayz6a2pj31380qqt9q.jpg)

&emsp;&emsp;能够看到边缘不是很平滑，但是也没有看到具体的锯齿，接下来我们将图片放大：

![WX20180313-170134@2x](https://ws2.sinaimg.cn/large/006tKfTcly1fpbayy3uvdj311o0mkt99.jpg)

&emsp;&emsp;是不是很明显的看到了。这就是锯齿。现在，我们设置采样点为8，再看看效果：

```c++
glfwWindowHint(GLFW_SAMPLES, 8);
glEnable(GL_MULTISAMPLE); 
```

![WX20180313-170439@2x](https://ws2.sinaimg.cn/large/006tKfTcly1fpbaz0v9hhj31ao0ti759.jpg)

&emsp;&emsp;这样处理之后，我们在稍微近的地方看下来显示效果会好很多。当然提高了显示效果，那么资源也是耗费更多，我们在每一个像素点上做8个采样点，最终的耗费便是以前的8倍。所以除了一些对画面要求比较苛刻的程序，一般情况下不会设置这么多的。

#### 题外话

&emsp;&emsp;事实上，OpenGL 的基础学习到这里也就差不多结束了，Learn OpenGL 剩下的教程都是高级光照和一些其他的东西。我也不会去保持一天一更这种速度去写这些内容的博客，这一系列文章算上这个总共二十四篇，就算是结束了。后边的内容，可能更多是总结性的内容了。目前虽然已经看完了 OpenGL 教程中的内容，但是却没有真正写过太多的代码，以后会零零散散更新一些知识的细节。

