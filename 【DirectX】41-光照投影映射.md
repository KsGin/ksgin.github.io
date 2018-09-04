---
title: 【DirectX】41-光照投影映射
date: 2018-04-22 22:34:33
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 44: Projected Light Maps](http://www.rastertek.com/dx11tut44.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们基于上篇的代码来实现光照纹理投影。这篇使用上一篇文章的代码，我们仅仅简单的改动了片段着色器。

<!--more-->

&emsp;&emsp;我们使用上一篇的场景：

![1](https://image.ibb.co/bT7z4H/image.png)

&emsp;&emsp;以及一张纹理贴图：

![2](https://image.ibb.co/c1Ke4H/image.png)

&emsp;&emsp;我们使用上一篇中介绍的方法将这张图片投影到我们的场景下，直接投影上去效果如下：

![3](https://image.ibb.co/iODaBx/image.png)

&emsp;&emsp;但我们的目的并非这样，我们将从纹理中读取的数据不直接输出，而是作为光照强度，乘以场景本身纹理颜色，`PixelShader` 代码如下：

```c++
//////////////
// TEXTURES //
//////////////
Texture2D shaderTexture : register(t0);
Texture2D projectionTexture : register(t1);


//////////////
// SAMPLERS //
//////////////
SamplerState SampleType;


//////////////////////
// CONSTANT BUFFERS //
//////////////////////
cbuffer LightBuffer
{
    float4 ambientColor;
    float4 diffuseColor;
    float3 lightDirection;
    float padding;
};


//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
    float3 normal : NORMAL;
    float4 viewPosition : TEXCOORD1;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 ProjectionPixelShader(PixelInputType input) : SV_TARGET
{
    float4 textureColor;
    float2 projectTexCoord;
    float4 projectionColor;
	float4 color;

    // Sample the pixel color from the texture using the sampler at this texture coordinate location.
    textureColor = shaderTexture.Sample(SampleType, input.tex);

    // Calculate the projected texture coordinates.
    projectTexCoord.x =  input.viewPosition.x / input.viewPosition.w / 2.0f + 0.5f;
    projectTexCoord.y = -input.viewPosition.y / input.viewPosition.w / 2.0f + 0.5f;
    // Determine if the projected coordinates are in the 0 to 1 range.  If it is then this pixel is inside the projected view port.
    if((saturate(projectTexCoord.x) == projectTexCoord.x) && (saturate(projectTexCoord.y) == projectTexCoord.y))
    {
        // Sample the color value from the projection texture using the sampler at the projected texture coordinate location.
        projectionColor = projectionTexture.Sample(SampleType, projectTexCoord);
		return projectionColor;
		color = (projectionColor + 0.1f) * textureColor;
	} else {
		color = 0.1f * textureColor;
	}

    return textureColor;
}
```

&emsp;&emsp;设定默认的环境光为 0.1 后，使用光照贴图投影映射，效果如下：

![3](https://image.ibb.co/diCfdc/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-ProjectedLightMaps](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-ProjectedLightMaps)

