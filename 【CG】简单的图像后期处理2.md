---
title: 【CG】简单的图像后期处理2
date: 2018-03-30 18:04:54
mathjax: true
tags:
	- Computer graphics
---

&emsp;&emsp;这篇文章中，我们将介绍[高斯模糊（Gaussian Blur）](https://zh.wikipedia.org/wiki/%E9%AB%98%E6%96%AF%E6%A8%A1%E7%B3%8A)以及在 HLSL 中实现这个效果。这个东西虽然听起来比较高端，但其实并不复杂，首先介绍一下他的概念：

> &emsp;&emsp;**高斯模糊**（英语：Gaussian Blur），也叫高斯平滑，是在[Adobe Photoshop](https://zh.wikipedia.org/wiki/Adobe_Photoshop)、[GIMP](https://zh.wikipedia.org/wiki/GIMP)以及[Paint.NET](https://zh.wikipedia.org/wiki/Paint.NET)等图像处理软件中广泛使用的处理效果，通常用它来减少图像噪声以及降低细节层次。这种模糊技术生成的图像，其视觉效果就像是经过一个半透明屏幕在观察图像，这与镜头焦外成像效果[散景](https://zh.wikipedia.org/wiki/%E6%95%A3%E6%99%AF)以及普通照明阴影中的效果都明显不同。高斯平滑也用于[计算机视觉](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%A7%86%E8%A7%89)算法中的预先处理阶段，以增强图像在不同比例大小下的图像效果（参见[尺度空间](https://zh.wikipedia.org/w/index.php?title=%E5%B0%BA%E5%BA%A6%E7%A9%BA%E9%97%B4&action=edit&redlink=1)表示以及[尺度空间](https://zh.wikipedia.org/w/index.php?title=%E5%B0%BA%E5%BA%A6%E7%A9%BA%E9%97%B4&action=edit&redlink=1)实现）。 从数学的角度来看，图像的高斯模糊过程就是图像与[正态分布](https://zh.wikipedia.org/wiki/%E6%AD%A3%E6%80%81%E5%88%86%E5%B8%83)做卷积。由于正态分布又叫作*高斯分布*，所以这项技术就叫作高斯模糊。图像与圆形[方框模糊](https://zh.wikipedia.org/w/index.php?title=%E6%96%B9%E6%A1%86%E6%A8%A1%E7%B3%8A&action=edit&redlink=1)做卷积将会生成更加精确的焦外成像效果。由于高斯函数的[傅立叶变换](https://zh.wikipedia.org/wiki/%E5%82%85%E7%AB%8B%E5%8F%B6%E5%8F%98%E6%8D%A2)是另外一个高斯函数，所以高斯模糊对于图像来说就是一个[低通滤波器](https://zh.wikipedia.org/wiki/%E4%BD%8E%E9%80%9A%E6%BB%A4%E6%B3%A2%E5%99%A8)。

<!--more-->

&emsp;&emsp;他的实现原理是对进行模糊的像素进行去中心化，每个像素的值都是周围相邻像素的加权平均，而他本身的值有着最大的高斯分布值，相邻元素随着和要进行模糊的像素的距离变远而权值变低。在二维空间内的高斯分布方程如下：
$$
G(x,y)=\frac{1}{\sqrt{2\pi\sigma^2}}e^{-(x^2+y^2)/(2\sigma^2)}
$$
&emsp;&emsp;这些东西如果有兴趣的话可以去看 [正态分布](https://zh.wikipedia.org/wiki/%E6%AD%A3%E6%80%81%E5%88%86%E5%B8%83) 。再次不多介绍，主要来看看我们将要去实现的内容，在 HLSL 中实现模糊。

&emsp;&emsp;前边已经说过，要实现这个效果是取这个像素和周围像素的颜色值进行加权平均，而我们可以取周围纹理坐标的值来进行计算。

&emsp;&emsp;首先我们定义一个权值矩阵：

```c++
float3x3 fil = float3x3(
		1.0f, 2.0f, 1.0f, 
		2.0f, 4.0f, 2.0f, 
		1.0f, 2.0f, 1.0f
	) / 16;
```

&emsp;&emsp;`fil[1][1]` 则是我们要求的像素的值，可以看到确实是距离目标像素距离越近权值越大。由于最终权值是要为1，所以除以16。

&emsp;&emsp;现在，我们来定义偏移坐标：

```c++
float2 posDelta[3][3] = {		// 3x3偏移UV坐标
	{float2(-1.0f , -1.0f) , float2(0.0f , -1.0f) , float2(1.0f , -1.0f)} ,
	{float2(-1.0f , 0.0f) , float2(0.0f , 0.0f) , float2(1.0f , 0.0f)},
	{float2(-1.0f , 1.0f) , float2(0.0f , 1.0f) , float2(1.0f , 1.0f)}
};
```

&emsp;&emsp;取的值也是在我们要求的像素的周围的位置。然后根据偏移坐标求出偏移后坐标的值，并使用采样器采取颜色，乘以权值，最终叠加到最终的颜色上：

```c++
float4 color = float4(0.0f, 0.0f, 0.0f, 1.0f);
for (int i = 0; i < 3; i++) {
	for (int j = 0; j < 3; j++) {
		float2 tUV = float2(uv.x + posDelta[i][j].x , uv.y + posDelta[i][j].y);
		// 加到最终颜色上
		color += tex.Sample(samp, tUV / texSize) * fil[i][j];
	};
}
```

&emsp;&emsp;对于我们的透明度，并不会去做计算，所以我们修正透明度，且返回颜色：

```c++
color.w = 1.0f;	// 透明度矫正
return color;
```

&emsp;&emsp;完整代码如下：

```c++
float4 doFilter(float3x3 fil , float2 texSize , float2 uv) {
	float2 posDelta[3][3] = {		// 3x3偏移坐标
		{float2(-1.0f , -1.0f) , float2(0.0f , -1.0f) , float2(1.0f , -1.0f)} ,
		{float2(-1.0f , 0.0f) , float2(0.0f , 0.0f) , float2(1.0f , 0.0f)},
		{float2(-1.0f , 1.0f) , float2(0.0f , 1.0f) , float2(1.0f , 1.0f)}
	};

	float4 color = float4(0.0f, 0.0f, 0.0f, 1.0f);

	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			float2 tUV = float2(uv.x + posDelta[i][j].x , uv.y + posDelta[i][j].y);
			// 加到最终颜色上
			color += tex.Sample(samp, tUV / texSize) * fil[i][j];
		};
	}
	color.w = 1.0f;	// 透明度矫正
	return color;
}

float4 main(pixelInputType input) : SV_TARGET{
	float2 texSize = float2(1920 , 1200);		// 图片的大小
	float3x3 fil = float3x3(
		1.0f, 2.0f, 1.0f, 
		2.0f, 4.0f, 2.0f, 
		1.0f, 2.0f, 1.0f
	) / 16;
	float2 uv = input.texcoord * texSize;
	return doFilter(fil , texSize , uv);	
} 
```

&emsp;&emsp;**这里需要注意的是 `texSize` 属性，我们的 UV 坐标取值是 0 到 1 的，所以我们会将纹理的位置映射到 UV 坐标上。**

&emsp;&emsp;效果如下：

![1](https://image.ibb.co/n3KCFS/image.png)

&emsp;&emsp;和我们之前的图片比较：

![2](https://image.ibb.co/bUneC7/image.png)

&emsp;&emsp;还是很明显的，在这里我们只是使用了周围一圈的像素来做加权平均，如果可以使用更多的话效果会更好。

