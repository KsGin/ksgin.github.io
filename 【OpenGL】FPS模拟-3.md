---
title: 【OpenGL】FPS模拟-3
date: 2018-03-03 16:44:46
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;之前两篇种我们简略的实现了基本地图的构造和摄像机的移动，这篇文章主要着重于摄像机的视角移动，和视角移动带来的摄像机移动的几个问题。

&emsp;&emsp;我们之前摄像机的移动是直接对cameraMat（即view矩阵）进行 translate 操作，是直接参照坐标轴实现的，即向前就是想z轴负方向移动，左就是x轴负方向，然而我们需要实现的是向前是向摄像机方向的正前方移动。所以需要修改。

&emsp;&emsp;我们假设的游戏角色是在xz平面上移动的，如果以摄像机前方作为正方向，如何保证当摄像机面向正上方时W键不会使得摄像机飞到天上...等等需要注意的问题。

<!--more-->

&emsp;&emsp;在开始写代码之前，我们需要了解一个概念：[欧拉角(Euler Angle)](https://zh.wikipedia.org/wiki/欧拉角)

> &emsp;&emsp;欧拉角(Euler Angle)是表示3D空间中可以表示任何旋转的三个值，由莱昂哈德·欧拉在18世纪提出。有三种欧拉角：俯仰角(Pitch)、偏航角(Yaw)和滚转角(Roll)，下面的图片展示了它们的含义：
>
> ![img](http://www.learnopengl.com/img/getting-started/camera_pitch_yaw_roll.png)
>
> &emsp;&emsp;**俯仰角**是描述我们如何往上和往下看的角，它在第一张图中表示。第二张图显示了**偏航角**，偏航角表示我们往左和往右看的大小。**滚转角**代表我们如何翻滚摄像机。每个欧拉角都有一个值来表示，把三个角结合起来我们就能够计算3D空间中任何的旋转了。

&emsp;&emsp;在这次的代码中我们将进行类FPS摄像机视角的移动，即对 Pitch 和 Yaw 角的转化。我们需要将这两个角度转换成三维空间里具体的方向向量。

> &emsp;&emsp;用一个给定的俯仰角和偏航角，我们可以把它们转换为一个代表新的方向向量的3D向量。俯仰角和偏航角转换为方向向量的处理需要一些三角学知识，我们以最基本的情况开始：
>
> ![img](http://www.learnopengl.com/img/getting-started/camera_triangle.png)
>
> &emsp;&emsp;如果我们把斜边边长定义为1，我们就能知道邻边的长度是cos x/h=cos x/1=cos xcos⁡ x/h=cos⁡ x/1=cos⁡ x，它的对边是sin y/h=sin y/1=sin ysin⁡ y/h=sin⁡ y/1=sin⁡ y。这样我们获得了能够得到x和y方向的长度的公式，它们取决于所给的角度。我们使用它来计算方向向量的元素：
>
> ![img](http://www.learnopengl.com/img/getting-started/camera_pitch.png)
>
> &emsp;&emsp;这个三角形看起来和前面的三角形很像，所以如果我们想象自己在xz平面上，正望向y轴，我们可以基于第一个三角形计算长度/y方向的强度(我们往上或往下看多少)
>
> &emsp;&emsp;看看我们是否能够为偏航角找到需要的元素：
>
> ![img](http://www.learnopengl.com/img/getting-started/camera_yaw.png)
>
> &emsp;&emsp;就像俯仰角一样我们可以看到x元素取决于cos(偏航角)的值，z值同样取决于偏航角的正弦值。把这个加到前面的值中。

&emsp;&emsp;由上边的两个图可得，我们的方向向量的 X 轴坐标取决于 cos(yaw) 和 cos(pitch) ，Y 轴取决于 sin(pitch) ，Z轴取决于 cos(pitch) 和 sin(yaw) 。

&emsp;&emsp;此时我们将之前的代码修改，重构 View 矩阵：

```c++
	vec3 cameraPos(0.0f, 5.0f, 20.0f);
	vec3 cameraFront(0.0f, 0.0f, -20.0f);
	vec3 cameraUp(0.0f, 1.0f, 0.0f);
```

&emsp;&emsp;定义摄像机坐标 cameraPos ，摄像机朝向 cameraFront 和看的方向 cameraUp 三个向量，对 cameraFront 修改以实现视角的旋转。

&emsp;&emsp;我们首先定义 pitch，yaw 两个变量：

```c++
	double pitch = 0.0, yaw = 0.0;
```

&emsp;&emsp;pitch 为绕 X 轴旋转的角，即视角的上下旋转，应该由屏幕坐标系 Y 轴决定， yaw 为绕 Y 轴旋转的角，即视角左右旋转，由屏幕坐标系的  X 轴决定。我们将鼠标在 Y 轴的偏移量作为 pitch 增量， X 轴偏移量作为 yaw 增量。

```c++
	pitch += (preY - curPosY) * 0.3; //乘以0.3的原因是降低点旋转速度
	yaw += (curPosX - preX) * 0.3;
```

&emsp;&emsp;得到了 pitch 和 yaw 值之后，我们便可以直接赋值了，由之前计算的

> &emsp;&emsp;我们的方向向量的 X 轴坐标取决于 cos(yaw) 和 cos(pitch) ，Y 轴取决于 sin(pitch) ，Z轴取决于 cos(pitch) 和 sin(yaw) 。

```c++
	cameraFront = normalize(
        	vec3(cos(radians(pitch))  * cos(radians(yaw)),  //x
            sin(radians(pitch)), 							//y
            cos(radians(pitch)) * sin(radians(yaw)))		//z
    );
```

&emsp;&emsp;得到了摄像机方向之后，直接使用 lookAt 方法：

```c++
	mat4 View = lookAt(cameraPos, cameraFront + cameraPos, cameraUp);
```

&emsp;&emsp;视角旋转搞定，以下是效果（点击图片进入播放页面）：

[![图片](https://image.ibb.co/kAQGHn/Screenshot_1.png)](https://youtu.be/ve9mjJlAauc)

&emsp;&emsp;视角的旋转成功了，但是带来的问题还没解决，即移动问题，我们需要让前后左右的位移与我们的面向有关而不是固定在坐标轴上。前后的处理比较简单，直接使用 cameraFront （摄像机方向）乘以速度就可以了。

```c++
		if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
		{
			cameraPos += speed * vec3(cameraFront.x , 0.0f , cameraFront.z);
		}

		if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
		{
			cameraPos += -speed * vec3(cameraFront.x, 0.0f, cameraFront.z);
		}
```

&emsp;&emsp;请注意看，我们没有直接使用 cameraFront 这个向量，而是使用它的 x 值和 z 值重新构造了一个向量，这个又是为何呢？

&emsp;&emsp;在前边我们提出了当摄像机方向向上偏或者向下偏的时候如何处理使得我们的摄像机不会向上或者向下位移的问题，就是在这里解决的，我们忽略了摄像机的方向向量种的 Y 分量。

&emsp;&emsp;左右移动（A ，D）则是向摄像机方向与 Y  轴方向形成的面的法线两边移动，使用 glm::cross 方法求法线方向向量，然后乘以速度即可，同样忽略高度。

```c++
		if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
		{
			cameraPos += -speed * cross(vec3(cameraFront.x, 0.0f, cameraFront.z), cameraUp);
		}

		if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
		{
			cameraPos += speed * cross(vec3(cameraFront.x, 0.0f, cameraFront.z), cameraUp);
		}
```

&emsp;&emsp;这时，我们基本已经可以实现自由移动了。不过还有一些细节问题。

&emsp;&emsp;首先是旋转，在进行 pitch 角旋转的时候，由于我们向上或者向下看不会看到背后（当 pitch > 90 或者 pitch < -90），所以需要对 pitch 的值进行限制。

```c++
	pitch = max(-89.0 , min(89.0, pitch)); //pitch 在-89到89范围内
```

&emsp;&emsp;还有是鼠标范围，我们将鼠标设置为在窗口循环出现，即可以无限的向一个方向转。

```c++
	//鼠标在窗口内循环
	if (curPosX < 0 || curPosX > 1024 || curPosY < 0 || curPosY > 768) {
		if (curPosX <= 0) curPosX = preX = 1023;
		if (curPosX >= 1024) curPosX = preX = 1;
		if (curPosY <= 0) curPosY = preY = 767;
		if (curPosY >= 768) curPosY = preY = 1;
		glfwSetCursorPos(window, curPosX, curPosY);
    }
```

&emsp;&emsp;这样我们可以看到，当鼠标光标从窗口最右边消失后会从最左边再进来，我们依旧可以向右旋转视角。但是这样难免有些不美观，在 FPS 视角的游戏中也没有显示鼠标位置的，所以我们调用 windows api 隐藏鼠标。

```c++
	//隐藏鼠标显示
	ShowCursor(false);
```

&emsp;&emsp;最终效果如下（点击图片进入Youtube查看视频）：

[![img](https://image.ibb.co/kAQGHn/Screenshot_1.png)](https://youtu.be/sEubooUPXzg)