---
title: 【DirectX】23-使用贴图实现特殊效果
date: 2018-04-06 12:06:22
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 18: Light Maps](http://www.rastertek.com/dx11tut18.html)

[Tutorial 19: Alpha Mapping](http://www.rastertek.com/dx11tut19.html)

#### 学习记录

&emsp;&emsp;这篇文章主要会介绍使用多重贴图来实现的一些有趣的效果，包括使用光照纹理和透明纹理。

<!--more-->

&emsp;&emsp;第一个需要介绍的是 Light Maps ，它使得我们可以在某些情况下使用及其少的资源实现光照效果。其实介绍起来也很简单，就是使用一张渐变的黑白纹理来和我们的基础纹理混合，达到类似于光照的效果，渐变纹理如下：

![1](https://image.ibb.co/iWRePx/texture2.gif)

&emsp;&emsp;我们使用它来取代上一篇中完全不搭的那张纹理，同时修改像素着色器。

```c++
Texture2D tex[2];
SamplerState samp;

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
};

float4 main(pixelInputType input) : SV_TARGET{
	return tex[0].Sample(samp, input.texcoord) * (tex[1].Sample(samp, input.texcoord) + 0.1f);
} 
```

&emsp;&emsp;简单的对其相乘，可以达到光照的效果，如下：

![2](https://image.ibb.co/cvJ4rc/image.png)

&emsp;&emsp;第二个要介绍的是 Alpha Mapping。这个则是使用一张透明贴图，来对另外两张图片进行混合。首先列出我们的贴图：

![3](https://image.ibb.co/k0QTHH/texture3.gif)

![4](https://image.ibb.co/fducBc/texture4.gif)

![5](https://image.ibb.co/gazYjx/texture5.gif)

&emsp;&emsp;分别将他们命名为 `texture3.gif` , `texture4.gif` , `texture5.gif` ，同时扩充我们的纹理数组：

```c++
const char * cubeTexs[] = {
		"texture1.jpg",
		"texture2.gif",
		"texture3.gif",
		"texture4.gif",
		"texture5.gif"
	};
const int nNumCubeTex = 5;
```

&emsp;&emsp;最后，修改像素着色器：

```c++
Texture2D tex[5];
SamplerState samp;

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
};


float4 main(pixelInputType input) : SV_TARGET{
	//return tex[0].Sample(samp, input.texcoord) * (tex[1].Sample(samp, input.texcoord) + 0.1f);

	float4 alpha = tex[2].Sample(samp, input.texcoord);
	return tex[3].Sample(samp, input.texcoord) * alpha + tex[4].Sample(samp, input.texcoord) * (1-alpha);
} 
```

&emsp;&emsp;效果如下：

![6](https://image.ibb.co/gUnvgc/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-TextureMaps](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-TextureMaps)