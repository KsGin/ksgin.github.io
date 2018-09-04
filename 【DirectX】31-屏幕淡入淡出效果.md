---
title: 【DirectX】31-屏幕淡入淡出效果
date: 2018-04-14 14:08:16
tags:	
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 28: Screen Fades](http://www.rastertek.com/dx11tut28.html)

#### 学习记录

&emsp;&emsp;这一篇文章中我们将基于之前的代码内容来实现场景中创建淡入淡出的效果。框架应当如下：

![1](https://image.ibb.co/jpWsRn/image.png)

&emsp;&emsp;新加的 `BitmapClass` , `FadeShaderClass` 两个类我们将在下边详细介绍。

<!--more--> 

&emsp;&emsp;淡入淡出的代码说起来很简单，我们先将需要渲染的内容渲染至纹理，然后根据在短时间内增加或者减少颜色以达到淡入淡出效果。首先我们需要渲染一张至屏幕的纹理，那么就需要一张屏幕贴图，这个由我们的 `bitmapclass` 类来做。这个类可以创建两个三角形覆盖屏幕并且贴图，也可以只是创建两个三角形覆盖屏幕，使用此缓冲区和 `RenderTextureClass` 所生成的纹理来渲染。

&emsp;&emsp;`BitmapClass` 类的声明如下：

```c++
#pragma once
#include <D3D11.h>
#include <DirectXMath.h>
#include "textureclass.h"

class BitmapClass
{
private:
	struct VertexType
	{
		DirectX::XMFLOAT3 position;
		DirectX::XMFLOAT2 texture;
	};

public:
	BitmapClass();
	BitmapClass(const BitmapClass&);
	~BitmapClass();

	bool Initialize(ID3D11Device*, int, int, int, int);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, int);

	int GetIndexCount() const;

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	bool UpdateBuffers(ID3D11DeviceContext*, int, int);
	void RenderBuffers(ID3D11DeviceContext*);

	bool LoadTexture(ID3D11Device*, CHAR*);
	void ReleaseTexture();

private:
	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
	TextureClass *m_Texture;
	int m_vertexCount, m_indexCount;
	int m_screenWidth, m_screenHeight;
	int m_bitmapWidth, m_bitmapHeight;
	int m_previousPosX, m_previousPosY;
};
```

&emsp;&emsp;这个类将根据给定的 w 和 h 生成一张覆盖屏幕的位图，在声明中我们有读取纹理和释放纹理的操作，不过在这篇文章中并未使用。

&emsp;&emsp;`FadeShaderClass` 则是由 `TextureShaderClass` 修改而来，它多了一个 `FadeAmount` 属性来控制纹理显示颜色。它的声明如下：

```c++
class FadeShaderClass
{
private:
	struct MatrixBufferType
	{
		DirectX::XMMATRIX world;
		DirectX::XMMATRIX view;
		DirectX::XMMATRIX projection;
	};

	struct FadeBufferType
	{
		float fadeAmount;
		DirectX::XMFLOAT3 padding;
	};

public:
	FadeShaderClass();
	FadeShaderClass(const FadeShaderClass&);
	~FadeShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*, float);

private:
	bool InitializeShader(ID3D11Device*, HWND, CHAR*, CHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, CHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*, float);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;
	struct ID3D11Buffer* m_fadeBuffer;
};
```

&emsp;&emsp;可以看到我们的多了一个 `FadeBufferType` 类型，这个类型只有一个 `float` 变量（另外那个是用来对齐的）。实现也是比较简单，在 `InitializeShader` 方法里，我们初始化了 `fadeBuffer` ：

```c++
........
    // Setup the description of the fade dynamic constant buffer that is in the vertex shader.
	fadeBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
	fadeBufferDesc.ByteWidth = sizeof(FadeBufferType);
	fadeBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
	fadeBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
	fadeBufferDesc.MiscFlags = 0;
	fadeBufferDesc.StructureByteStride = 0;

	// Create the constant buffer pointer so we can access the pixel shader constant buffer from within this class.
	result = pDevice->CreateBuffer(&fadeBufferDesc, NULL, &m_fadeBuffer);
	if(FAILED(result))
	{
		return false;
	}
........
```

&emsp;&emsp;这两个类实现后我们就可以简单的来实现这个效果了。为了节省资源，我们在渲染的时候首先会判断当前淡入的动画是否完毕，如果没有完毕。我们会先将内容渲染至纹理，这个交以 `FadeShaderClass` 来处理。然后在将纹理贴图到屏幕位图中，如果已经完毕，则直接渲染至后台缓冲。

&emsp;&emsp;`GraphicsClass` 更新后的定义如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: GraphicsClass
////////////////////////////////////////////////////////////////////////////////
class GraphicsClass
{
public:
	GraphicsClass();
	GraphicsClass(const GraphicsClass&);
	~GraphicsClass();

	bool Initialize(int, int, HWND);
	void Shutdown();
	bool Frame(float frameTime);
	bool Render();

private:
	bool RenderToTexture(float rotation);
	bool RenderFadingScene();
	bool RenderNormalScene(float);

	D3DClass* m_D3D;
	CameraClass* m_Camera;
	ModelClass* m_Model;
	TextureShaderClass* m_TextureShader;
	RenderTextureClass* m_RenderTexture;

	BitmapClass* m_Bitmap;

	float m_fadeInTime, m_accumulatedTime, m_fadePercentage;
	bool m_fadeDone;
	FadeShaderClass* m_FadeShader;
};

```

&emsp;&emsp;我们增加了几个成员变量，用来控制淡入淡出的动画效果。`m_fadeDone` 判断动画效果是否结束，`m_fadeInTime` 则是动画效果的总时间，`m_accumulatedTime` 则是已累积时间，`m_fadePercentage` 代表我们将会渲染的颜色亮度的百分比。当累积时间超过总时间的时候，动画结束。

&emsp;&emsp;在初始化方法里初始化这几个值：

```c++
m_fadeInTime = 3000.0f;
m_accumulatedTime = 0;
m_fadePercentage = 0;
m_fadeDone = false;

m_FadeShader = new FadeShaderClass;
if (!m_FadeShader) {
	return false;
}
result = m_FadeShader->Initialize(m_D3D->GetDevice(), hwnd);
if (!result) {
	MessageBox(hwnd, "Could not initialize the fade shader object", "Error", MB_OK);
	return false;
}
```

&emsp;&emsp;`Frame` 方法里，我们每帧更新累积值以及淡入百分比，直到动画结束。

```c++
bool GraphicsClass::Frame(float frameTime)
{
	// Set the position of the camera.
	if (!m_fadeDone) {
		m_accumulatedTime += frameTime;
		if (m_accumulatedTime < m_fadeInTime) {
			m_fadePercentage = m_accumulatedTime / m_fadeInTime;
		} else {
			m_fadeDone = true;
			m_fadePercentage = 1.0f;
		}
	}
	m_Camera->SetPosition(0.0f, 0.0f, -10.0f);
	return true;
}
```

&emsp;&emsp;最后，在 `Render` 方法里，我们根据当前是否在动画状态选择不一样的渲染方式：

```c++
bool GraphicsClass::Render()
{
	bool result;
	static float rotation = 0.0f;

	rotation += (float)XM_PI * 0.005f;
	if (rotation > 360.0f) {
		rotation -= 360.0f;
	}

	if (m_fadeDone) {
		RenderNormalScene(rotation);
	} else {
		// Render the entire scene as a reflection to the texture first.
		result = RenderToTexture(rotation);
		if (!result)
		{
			return false;
		}

		// Render the scene as normal to the back buffer.
		result = RenderFadingScene();
		if (!result)
		{
			return false;
		}
	}

	return true;
}
```

&emsp;&emsp;`RenderToTexture` 与 `RenderFadingScene` 方法定义如下：

```c++
bool GraphicsClass::RenderToTexture(float rotation)
{
	XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	bool result;


	// Set the render target to be the render to texture.
	m_RenderTexture->SetRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView());

	// Clear the render to texture.
	m_RenderTexture->ClearRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView(), 0.0f, 0.0f, 0.0f, 1.0f);

	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	m_D3D->GetWorldMatrix(worldMatrix);
	m_Camera->GetViewMatrix(viewMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);

	// Multiply the world matrix by the rotation.
	worldMatrix = worldMatrix * XMMatrixRotationY(rotation);

	// Put the model vertex and index buffers on the graphics pipeline to prepare them for drawing.
	m_Model->Render(m_D3D->GetDeviceContext());

	// Render the model with the texture shader.
	result = m_TextureShader->Render(m_D3D->GetDeviceContext(), m_Model->GetIndexCount(), worldMatrix, viewMatrix,
					 projectionMatrix, m_Model->GetTexture());
	if(!result)
	{
		return false;
	}

	// Reset the render target back to the original back buffer and not the render to texture anymore.
	m_D3D->SetBackBufferRenderTarget();

	return true;
}


bool GraphicsClass::RenderFadingScene()
{
	XMMATRIX worldMatrix, viewMatrix, orthoMatrix;
	bool result;


	// Clear the buffers to begin the scene.
	m_D3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);

	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	m_D3D->GetWorldMatrix(worldMatrix);
	m_Camera->GetViewMatrix(viewMatrix);
	m_D3D->GetOrthoMatrix(orthoMatrix);

	m_D3D->TurnZBufferOff();
	result = m_Bitmap->Render(m_D3D->GetDeviceContext(), 0, 0);
	if (!result) {
		return false;
	}

	result = m_FadeShader->Render(
		m_D3D->GetDeviceContext(),
		m_Bitmap->GetIndexCount(), worldMatrix, viewMatrix, orthoMatrix,
		m_RenderTexture->GetShaderResourceView(), m_fadePercentage);
	if (!result) {
		return false;
	}

	m_D3D->TurnZBufferOn();
	// Present the rendered scene to the screen.
	m_D3D->EndScene();

	return true;
}
```

&emsp;&emsp;`RenderNormalScene` 则是直接渲染至后台缓冲。

&emsp;&emsp;我们的着色器代码也是比较简单，顶点着色器和普通一样，而像素着色器则是新增了一个缓冲：

```c++
cbuffer FadeBuffer
{
    float fadeAmount;
    float3 padding;		// 占位用的
};
```

&emsp;&emsp;我们将纹理采样的值与 `fadeAmount` 相乘则是最终的颜色了。

```c++
float4 FadePixelShader(PixelInputType input) : SV_TARGET
{
    float4 color;

    // Sample the texture pixel at this location.
    color = shaderTexture.Sample(SampleType, input.tex);

	// Reduce the color brightness to the current fade percentage.
    color = color * fadeAmount;

    return color;
}
```

&emsp;&emsp;最终效果则是一个淡淡变量的立方体。我随便截了一张淡入过程中的图，如下：

![2](https://image.ibb.co/cCYOXS/image.png)

&emsp;&emsp;源代码：[**DX11Tutorial-ScreenFades**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-ScreenFades)

