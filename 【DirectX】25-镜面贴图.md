---
title: 【DirectX】25-镜面贴图
date: 2018-04-08 18:29:08
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 21: Specular Mapping](http://www.rastertek.com/dx11tut21.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将基于上一篇的内容加一点扩展，为凹凸贴图实现的效果再加上镜面反射。关于镜面反射在 OpenGL 学习笔记中有介绍过：[【OpenGL】11-光照](https://blog.ksgin.com/2018/02/21/【opengl】11-光照/) 。这一篇文章则是简单的介绍在使用贴图的情况下实现。

<!--more-->

&emsp;&emsp;这一篇内容比较少，因为我们只需要更改着色器代码就可以了。首先我们更新一下我们的贴图。这一次将使用新的图片（在文末的源代码链接可以找到）。

&emsp;&emsp;在这里我们只需要对着色器代码进行修改，首先在顶点着色器中我们需要一个摄像机坐标，这里我们直接在着色器里写入我们当前的摄像机位置：

```c++
float3 cameraPos = float3(1.0f, 1.0f, -1.0f);
```

&emsp;&emsp;同时我们需要得到与世界变换矩阵变换后的坐标：

```c++
float4 worldPosition;
worldPosition = mul(input.pos, world);
```

&emsp;&emsp;我们在 `pixelInputType` 结构体里也新增了视野方向变量 ，并且在顶点着色器里进行计算，传给像素着色器：

```c++
worldPosition = mul(input.pos, world);
output.viewDir = cameraPos - output.pos.xyz;
output.viewDir = normalize(output.viewDir);
```

&emsp;&emsp;顶点着色器的完整代码如下：

```c++
cbuffer ConstantBuffer : register(b0) {
	matrix world;
	matrix view;
	matrix projection;
}

struct vertexInputType {
	float4 pos : POSITION;
	float2 texcoord : TEXCOORD;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
};

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
	float3 viewDir : TEXCOORD1;
};

pixelInputType main( vertexInputType input ){
	pixelInputType output;
	float4 worldPosition;
	float3 cameraPos = float3(1.0f, 1.0f, -1.0f);

	output.pos = input.pos;	
	output.pos = mul(output.pos, world);
	output.pos = mul(output.pos, view);
	output.pos = mul(output.pos, projection);

	output.texcoord = input.texcoord;

	output.normal = mul(input.normal, (float3x3)world);
	output.normal = normalize(output.normal);

	output.tangent = mul(input.tangent, (float3x3)world);
	output.tangent = normalize(output.tangent);

	output.bitangent = mul(input.bitangent, (float3x3)world);
	output.bitangent = normalize(output.bitangent);

	worldPosition = mul(worldPosition, world);
	output.viewDir = cameraPos - output.pos.xyz;
	output.viewDir = normalize(output.viewDir);

	return output;
}
```

&emsp;&emsp;在片段着色器里，我们为了更好的显示效果，将光源方向修改了一下，变成了正对 z 轴，它的反方向就成了正对 z 负轴。

```c++
lightDir = normalize(float3(0.0f, 0.0f, 1.0f));
lightDir = -lightDir;
```

&emsp;&emsp;片段着色前前边的代码与上一篇一样，而当法线与光照反向量的点乘大于 0 的时候，即光能照到的时候，我们开始计算镜面反射。

```c++
if (lightIntensity > 0.0f) {		
	reflection = normalize(2 * lightIntensity * bumpNormal - float4(lightDir, 1.0f));
	specular = pow(saturate(dot(reflection, input.viewDir)), specularPower);
	specular = specularIntensity * specular;
}
```

&emsp;&emsp;最后将颜色混合。

```c++
return saturate(color + specular);
```

&emsp;&emsp;像素着色器的完整代码如下：

```c++
Texture2D tex[3];
SamplerState samp;

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
	float3 viewDir : TEXCOORD1;
};


float4 main(pixelInputType input) : SV_TARGET {
	float4 texColor;
	float4 bumpMap;
	float4 specularIntensity;
	
	float3 lightDir;
	float3 bumpNormal;
	float lightIntensity;
	float4 color;
	
	float3 reflection;
	float4 specularPower = 16;
	float4 specular;

	texColor = tex[0].Sample(samp, input.texcoord);
	bumpMap = tex[1].Sample(samp, input.texcoord);	
	specularIntensity = tex[2].Sample(samp, input.texcoord);
	
	lightDir = normalize(float3(0.0f, 0.0f, 1.0f));
	lightDir = -lightDir;

	bumpMap = bumpMap * 2.0f - 1.0f;
	bumpNormal = (bumpMap.x * input.tangent) + (bumpMap.y * input.bitangent) + (bumpMap.z * input.normal);	
	bumpNormal = normalize(bumpNormal);

	lightIntensity = saturate(dot(bumpNormal , lightDir));
	color = lightIntensity * float4(1.0f, 1.0f, 1.0f, 1.0f);
	color = color * texColor;

	if (lightIntensity > 0.0f) {		
		reflection = normalize(2 * lightIntensity * bumpNormal - float4(lightDir, 1.0f));
		specular = pow(saturate(dot(reflection, input.viewDir)), specularPower);
		specular = specularIntensity * specular;
	}

	return saturate(color + specular);
} 
```

&emsp;&emsp;最终效果如下：

![img5](https://image.ibb.co/d9McWc/image.png)

&emsp;&emsp;这个效果很明显了吧。

&emsp;&emsp;源代码：[**DX11Tutorial-SpecularMapping**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-SpecularMapping)

