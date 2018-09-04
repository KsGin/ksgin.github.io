---
title: 【DirectX】47-延迟渲染
date: 2018-04-27 22:26:25
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 50: Deferred Shading](http://www.rastertek.com/dx11tut50.html)

《Real-Time Rendering 3rd》

《GPU-Programming-AndCgLanguage-Primer》

#### 学习记录

&emsp;&emsp;在计算机图形中，延迟渲染（Deferred Rendering），即延迟着色（Deferred Shading）是将着色计算延迟到深度测试之后进行处理的一种渲染方法。延迟着色技术最大的优势就是将光源的数目和场景中物体的数目在复杂度层面上完全分开，能够在渲染拥有成百上千光源的场景的同时依然保持很高的帧率，给我们渲染拥有大量光源的场景提供了很多可能性。

&emsp;&emsp;这篇文章，我们简单介绍延迟渲染的实现。

<!--more-->

&emsp;&emsp;延迟渲染的实现我们主要分为两步，第一步将所有物体绘制到屏幕缓冲（G-Buffer），第二部在深度测试后进行光照计算，对应于正向渲染（Forward Rendering）O（m*n）的复杂度场景，延迟渲染的复杂度仅仅为 O（m+n），因为它避免了对在深度测试中被丢弃的片元的渲染。

> &emsp;&emsp;Deferred shading is the process of splitting up traditional rendering into a two stage system that is designed to improve efficiency in shaders that require complex lighting.
>
> &emsp;&emsp;In the first stage we render as usual by using a vertex and pixel shader.However the purpose of the pixel shader is entirely different.We no longer use the pixel shader to do lighting calculations and then output a color to the back buffer.We instead output information about that pixel (such as normals, texture color, and so forth) to a render to texture buffer.So now our pixel shader output from the first stage has become a 2D texture full of scene information that can be used as a texture input to the second stage.
>
> &emsp;&emsp;In the second stage we re-write all our traditional 3D shaders to now do 2D post processing using the render to texture outputs from the first stage.And because we are doing 2D post processing we have just a set number of pixels to run our lighting equations on instead of a massive scene full of thousands of complex 3D objects.Therefore it no longer matters if we have thousands of lights in our scene or how many polygons are in each object.We only perform lighting equations on the very final 2D output pixels.
>
> &emsp;&emsp;So deferred shading eliminates all sorts of calculations that would be required in the vertex and pixel shader for every single model in the scene.And all those complex calculations create output data that is usually discarded anyhow due to culling.So all those inefficiencies are now eliminated and our shading equations are now a fixed amount of processing regardless of scene size, number of lights, and so forth.This really opens the door so we can do more complex lighting as well as simplify and combine our shaders that already required 2D post processing.

&emsp;&emsp;首先我们有一个 `DeferredBufferClass` 类来定义我们的 `G-Buffer` ，在这里我们 `Buffer` 中存储颜色和法线的信息，这个类由 `RenderToTexture` 修改而来，他如今有多个缓冲以用于将不同的属性（颜色，法线）来存储在不同位置，其声明如下：

```c++
/////////////
// DEFINES //
/////////////
const int BUFFER_COUNT = 2;

//////////////
// INCLUDES //
//////////////
#include <D3D11.h>
#include <D3DX10math.h>

////////////////////////////////////////////////////////////////////////////////
// Class name: DeferredBuffersClass
////////////////////////////////////////////////////////////////////////////////
class DeferredBuffersClass
{
public:
	DeferredBuffersClass();
	DeferredBuffersClass(const DeferredBuffersClass&);
	~DeferredBuffersClass();

	bool Initialize(ID3D11Device*, int, int, float, float);
	void Shutdown();

	void SetRenderTargets(ID3D11DeviceContext*);
	void ClearRenderTargets(ID3D11DeviceContext*, float, float, float, float);

	ID3D11ShaderResourceView* GetShaderResourceView(int);

private:
	int m_textureWidth, m_textureHeight;
	ID3D11Texture2D* m_renderTargetTextureArray[BUFFER_COUNT];
	ID3D11RenderTargetView* m_renderTargetViewArray[BUFFER_COUNT];
	ID3D11ShaderResourceView* m_shaderResourceViewArray[BUFFER_COUNT];
	ID3D11Texture2D* m_depthStencilBuffer;
	ID3D11DepthStencilView* m_depthStencilView;
	D3D11_VIEWPORT m_viewport;
};
```

&emsp;&emsp;之后我们将使用 `DeferredShaderClass` 类来将场景渲染至我们的 `G-Buffer` ，首先来看我们的着色器，如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////

//////////////////////
// CONSTANT BUFFERS //
//////////////////////
cbuffer MatrixBuffer
{
	matrix worldMatrix;
	matrix viewMatrix;
	matrix projectionMatrix;
};

//////////////
// TYPEDEFS //
//////////////
struct VertexInputType
{
    float4 position : POSITION;
    float2 tex : TEXCOORD0;
	float3 normal : NORMAL;
};

struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
	float3 normal : NORMAL;
};

////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType DeferredVertexShader(VertexInputType input)
{
    PixelInputType output;
    
    
	// Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

	// Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);
    
	// Store the texture coordinates for the pixel shader.
    output.tex = input.tex;
    
	// Calculate the normal vector against the world matrix only.
    output.normal = mul(input.normal, (float3x3)worldMatrix);
	
    // Normalize the normal vector.
    output.normal = normalize(output.normal);

	return output;
}

////////////////////////////////////////////////////////////////////////////////
// Filename: deferred.ps
////////////////////////////////////////////////////////////////////////////////

//////////////
// TEXTURES //
//////////////
Texture2D shaderTexture : register(t0);


///////////////////
// SAMPLE STATES //
///////////////////
SamplerState SampleTypeWrap : register(s0);


//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
	float3 normal : NORMAL;
};

struct PixelOutputType
{
    float4 color : SV_Target0;
    float4 normal : SV_Target1;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
PixelOutputType DeferredPixelShader(PixelInputType input) : SV_TARGET
{
	PixelOutputType output;


	// Sample the color from the texture and store it for output to the render target.
	output.color = shaderTexture.Sample(SampleTypeWrap, input.tex);
	
	// Store the normal for output to the render target.
	output.normal = float4(input.normal, 1.0f);

    return output;
}
```

&emsp;&emsp;顶点着色器和以往一样，像素着色器中，我们将渲染物体的颜色（color）和法线（normal）分别输出给不同的 `TARGET` 。

&emsp;&emsp;他们的 `ShaderClass` 相比较 `TextureShaderClass` 几乎没有做修改，如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: DeferredShaderClass
////////////////////////////////////////////////////////////////////////////////
class DeferredShaderClass
{
private:
	struct MatrixBufferType
	{
		D3DXMATRIX world;
		D3DXMATRIX view;
		D3DXMATRIX projection;
	};

public:
	DeferredShaderClass();
	DeferredShaderClass(const DeferredShaderClass&);
	~DeferredShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX, ID3D11ShaderResourceView*);

private:
	bool InitializeShader(ID3D11Device*, HWND, CHAR*, CHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, CHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX, ID3D11ShaderResourceView*);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11SamplerState* m_sampleStateWrap;
	ID3D11Buffer* m_matrixBuffer;
};
```

&emsp;&emsp;这样，在像素着色器阶段，我们并未做太多的运算，当渲染正常进行深度测试后，我们使用 `G-Buffer` 中的颜色纹理和法线纹理计算光照。这样我们计算的光照只有最终渲染的屏幕大小（width * height），当场景中存在大量物体的时候，延迟渲染显然比正向渲染快速的多。当然，由于需要存储 `G-Buffer` ，占用空间自然也多了一些。

&emsp;&emsp;最终渲染结果和普通渲染一个，我们所渲染的 cube 如下：

![1](https://image.ibb.co/krvtOc/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-DeferredShading](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-DeferredShading)



