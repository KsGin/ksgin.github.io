---
title: 【CG】简单的图像后期处理1
date: 2018-03-29 20:45:18
tags:
	- Computer graphics
---

&emsp;&emsp;这两天不知道为什么总是类的不行，累到看个书都看不进去。今天折腾我的破手机，给换了个 rom 和体验各种好玩的应用花去了不少时间，感觉自己又在浪的边缘慢慢试探了。

&emsp;&emsp;这篇文章又是一个新坑的开始，在这个系列里将使用 HLSL 对纹理实现一些常见的比较简单的后期处理效果，像什么反相，模糊这种。这篇主要是开个坑和实现最简单的反相和灰度。

<!--more-->

&emsp;&emsp;我们首先构建一个覆盖整个显示窗口的矩形并为其贴上一张图片，这里简单的贴上顶点的坐标位置和纹理位置：

```c++
struct Vertex {
	DirectX::XMFLOAT3 pos;
	DirectX::XMFLOAT2 tex;
};

Vertex vertices[] =
{
	{ DirectX::XMFLOAT3(-1.0f, -1.0f, 0.5f) , DirectX::XMFLOAT2(0.0f,1.0f) },
	{ DirectX::XMFLOAT3(-1.0f, 1.0f, 0.5f) , DirectX::XMFLOAT2(0.0f,0.0f) },
	{ DirectX::XMFLOAT3(1.0f, 1.0f, 0.5f) , DirectX::XMFLOAT2(1.0f,0.0f) },
	{ DirectX::XMFLOAT3(-1.0f, -1.0f, 0.5f) , DirectX::XMFLOAT2(0.0f,1.0f) },
	{ DirectX::XMFLOAT3(1.0f, 1.0f, 0.5f) , DirectX::XMFLOAT2(1.0f,0.0f) },
	{ DirectX::XMFLOAT3(1.0f, -1.0f, 0.5f) , DirectX::XMFLOAT2(1.0f,1.0f) }
};
```

&emsp;&emsp;我们贴图后应该是这个样子的：

![1](https://image.ibb.co/fDU1QS/image.png)

&emsp;&emsp;请无视掉那个必应的 LOGO ！

&emsp;&emsp;现在，我们来看看**反相(Inversion)**：反相，即颜色取反，我们对其每个像素的颜色 `s` 做 `s = 1-s` 的计算。HLSL 代码中，我们定义一个方法对其反相。

```c++
float4 Inversion(float4 color) {
	return float4(1.0f , 1.0f , 1.0f , 1.0f) - color;
}

float4 main(pixelInputType input) : SV_TARGET{
	float4 color = tex.Sample(samp, input.texcoord);
	return Inversion(color);	
} 
```

![2](https://image.ibb.co/mOQrQS/image.png)

&emsp;&emsp;现在来试试对部分图片反相，我们在 `main` 方法里加个判断，对屏幕右半部分反相处理：

```c++
float4 main(pixelInputType input) : SV_TARGET{
	float4 color = tex.Sample(samp, input.texcoord);
	if (input.pos.x > 960) return Inversion(color);	//窗口大小为1920 
	else return color;	
}
```

&emsp;&emsp;这里只是做个实验，所以直接依照我当前的窗口大小将数字写死了。效果如下：

![3](https://image.ibb.co/k52XX7/image.png)

&emsp;&emsp;现在来看看我们的第二个效果，**灰度化(Grayscale)**：

&emsp;&emsp;灰度化的计算也比较简单，由于是去掉除了黑白灰的其他颜色，所以我们可以简单的将 RGB 做一个平均或者加权平均，这取决于我们的标准和感官，在这里我们简单的平均一下：

```c++
float4 Grayscale(float4 color) {
	float rgbValue = (color.x + color.y + color.z) / 3;
	return float4(rgbValue, rgbValue, rgbValue, 1.0f);
}

float4 main(pixelInputType input) : SV_TARGET{
	float4 color = tex.Sample(samp, input.texcoord);
	return Grayscale(color);
}
```

![4](https://image.ibb.co/mE8QKn/image.png)

&emsp;&emsp;现在，我们对刚才一半反相的那张照片添加下半部分灰度的效果：

```c++
float4 main(pixelInputType input) : SV_TARGET{
	float4 color = tex.Sample(samp, input.texcoord);
	if (input.pos.y > 540) {
		color = Grayscale(color);
	}
	if (input.pos.x > 960) {
		color = Inversion(color);
	} 
	return color;
}
```

&emsp;&emsp;看看效果：

![5](https://image.ibb.co/cSANzn/image.png)

&emsp;&emsp;下一篇将看心情实现高斯模糊（Gaussian Blur）。本系列文章源代码在这里：[PostProcessing](https://github.com/KsGin/PostProcessing)