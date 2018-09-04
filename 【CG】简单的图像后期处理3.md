---
title: 【CG】简单的图像后期处理3
date: 2018-04-02 22:32:21
tags: Computer graphics
---

&emsp;&emsp;这篇文章中主要介绍锐化（sharpening）效果。如果说模糊是一种去中心化，使得像素边缘及灰度跳变的部分趋于平滑，而锐化则是恰恰相反。

&emsp;&emsp;在百度百科，对锐化的描述如下：

> &emsp;&emsp;图像**锐化**(image sharpening)是补偿图像的轮廓，增强图像的边缘及灰度跳变的部分，使图像变得清晰，分为空域处理和频域处理两类。 图像**锐化**是为了突出图像上地物的边缘、轮廓，或某些线性目标要素的特征。 这种滤波方法提高了地物边缘与周围像元之间的反差，因此也被称为边缘增强。

<!--more-->

&emsp;&emsp;我们的实现也是颇为简单，在上一篇中我们事先高斯模糊的时候，定义了一个滤波器数组以及一个用来做权重的平滑矩阵，如下：

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

&emsp;&emsp;在 `main` 方法里定义的 `fil` 使得我们单个像素的颜色与周围颜色权值叠加，给一种模糊的效果，而锐化则是将 `fil` 改为：

```c++
float3x3 fil = float3x3(
	-1.0f, -2.0f, -1.0f, 
	-2.0f, 16.0f, -2.0f, 
	-1.0f, -2.0f, -1.0f
);
```

&emsp;&emsp;效果如下：

![346CBD858E06B57CC480B12AD38BC0](https://image.ibb.co/iFZmVS/image.png)



&emsp;&emsp;源代码：[Post Processing](https://github.com/KsGin/PostProcessing)

