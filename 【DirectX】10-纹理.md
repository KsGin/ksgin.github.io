---
title: 【DirectX】10-纹理
date: 2018-03-25 17:01:43
tags:
	- DirectX
	- Computer graphics
---

#### 前言

&emsp;&emsp;这篇文章开始不再以教程中的框架代码为示例，因为我发现本身在学习 DirectX 基本语法当中，多写几遍能加深印象是其一，其二是对那些 API 理解并非太深刻的时候实现较为复杂的内容的时候会导致难以去 Debug ，同时也丧失了基本的学习效果。

#### 前置内容

&emsp;&emsp;我们使用我们之前做过的简单的三角形那个例子，将其代码拷贝过来（再写一遍最好，虽然说写重复代码比较愚蠢，但是蠢有蠢的好处），用来实现这篇文章要实现的内容。

&emsp;&emsp;我们这篇文章将学习纹理的用法，之后在我们之前的纯色三角形上贴一张好看图片

<!--more-->

#### 教程地址

[Tutorial 5: Texturing](http://www.rastertek.com/dx11s2tut05.html)

#### 学习记录

&emsp;&emsp;纹理（Texture）的概念我想就不用介绍了，在 OpenGL 中我们使用过很多次。这篇文章主要是介绍 DX 中纹理的加载操作。首先我们修改着色器程序：

```c++
// file : triangleVertexShader.hlsl
struct vertexInputType {
	float4 pos : POSITION;
	float2 texcoord : TEXCOORD0;
};

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
};

pixelInputType main( vertexInputType input )
{
	pixelInputType output;
	output.pos = input.pos;
	output.texcoord = input.texcoord;
	return output;
}
```

&emsp;&emsp;我们在顶点着色器中定义了结构体用于传递数据，结构体包括 `position` 和 `texcoord` 数据（即 `UV` 坐标）。

&emsp;&emsp;在像素着色器中，我们定义了 `Texture2D` 变量和 `SamplerState` 采样器变量，用于从纹理上采样颜色。

```c++
// file: trianglePixelShader.hlsl
Texture2D tex;
SamplerState samplerType;
 
struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
};

float4 main(pixelInputType input) : SV_TARGET
{
	return tex.Sample(samplerType, input.texcoord);
}
```

&emsp;&emsp;而在 DX 代码中，我们首先修改 `Vertex` 结构体和顶点坐标集合，现在 `Vertex` 结构体应当包含一个 UV 坐标：

```c++
struct Vertex {
	DirectX::XMFLOAT3 pos;
	DirectX::XMFLOAT2 tex;
};

Vertex vertices[] =
{
	{ DirectX::XMFLOAT3(0.0f, 0.5f, 0.5f) , DirectX::XMFLOAT2(0.0f,1.0f) },
	{ DirectX::XMFLOAT3(0.5f, -0.5f, 0.5f) , DirectX::XMFLOAT2(1.0f,-1.0f) },
	{ DirectX::XMFLOAT3(-0.5f, -0.5f, 0.5f) , DirectX::XMFLOAT2(-1.0f,1.0f) }
};
```

&emsp;&emsp;现在创建一个采样器描述并为他赋值：

```c++
D3D11_SAMPLER_DESC samplerDesc;
samplerDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;
samplerDesc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;
samplerDesc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;
samplerDesc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;
samplerDesc.MipLODBias = 0.0f;
samplerDesc.MaxAnisotropy = 1;
samplerDesc.ComparisonFunc = D3D11_COMPARISON_ALWAYS;
samplerDesc.BorderColor[0] = 0;
samplerDesc.BorderColor[1] = 0;
samplerDesc.BorderColor[2] = 0;
samplerDesc.BorderColor[3] = 0;
samplerDesc.MinLOD = 0;
samplerDesc.MaxLOD = D3D11_FLOAT32_MAX;
```

&emsp;&emsp;关于这个结构体的各个属性可以看 MSDN 。（我懒）

&emsp;&emsp;使用采样器描述创建一个指针对象：

```c++
ID3D11SamplerState *pSamplerState = nullptr;
hr = pDevice->CreateSamplerState(&samplerDesc, &pSamplerState);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateSampler", "Error", MB_OK);
	return hr;
}
```

&emsp;&emsp;从文件创建一个渲染视图：

```c++
ID3D11ShaderResourceView *pShaderResourceView;
hr = D3DX11CreateShaderResourceViewFromFile(pDevice, "./triangle.jpg", nullptr, nullptr, &pShaderResourceView, nullptr);
	if (FAILED(hr)) {
		MessageBox(nullptr, "ERROR::CreateShaderResourceView", "Error", MB_OK);
		return hr;
	}
```

&emsp;&emsp;为立即上下文对象赋值渲染视图和采样器：

```c++
pImmediateContext->PSSetShaderResources(0, 1, &pShaderResourceView);
pImmediateContext->PSSetSamplers(0, 1, &pSamplerState);
```

&emsp;&emsp;这时，我们还需要修改布局描述：

```c++
D3D11_INPUT_ELEMENT_DESC layout[] = {
		{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0 } ,
		{ "TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 }
	};

```

&emsp;&emsp;在 `POSITION` 的基础上我们需要添加一个 `TEXCOORD` ，其他的代码都不需要修改，现在点击运行，就可以看到我们的三角形了。

![tex](https://image.ibb.co/jMYNQS/image.png)

&emsp;&emsp;完整代码请看：[Github : DX11Tutorial texture triangle](https://github.com/KsGin/DX11Tutorial/blob/master/DX11Tutorial-TextureTriangle/main.cpp)



