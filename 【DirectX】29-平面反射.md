---
title: 【DirectX】29-平面反射
date: 2018-04-13 20:43:57
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 27: Reflection](http://www.rastertek.com/dx11tut27.html)

#### 学习记录

&emsp;&emsp;这一篇文章中，我们主要介绍在 dx11 中实现平面反射。与之前不同，这篇文章和之后的源代码我们将使用已有框架来进行实验（事实上是因为代码逐渐变多之后看的好乱），相信在之前基础知识完成学习后对于这个框架已经没有多少入手难度了。源码将在文末下载。

<!--more-->

&emsp;&emsp; 创建反射我们首先会将要渲染的物体的反射图案渲染到纹理上，然后用纹理与镜面混合，最终达到反射效果。将 cube 的反射图案渲染至纹理如下：

```c++
bool GraphicsClass::RenderToTexture()
{
	D3DXMATRIX worldMatrix, reflectionViewMatrix, projectionMatrix;
	static float rotation = 0.0f;


	// Set the render target to be the render to texture.
	m_RenderTexture->SetRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView());

	// Clear the render to texture.
	m_RenderTexture->ClearRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView(), 0.0f, 0.0f, 0.0f, 1.0f);

	// Use the camera to calculate the reflection matrix.
	m_Camera->RenderReflection(-1.5f);

	// Get the camera reflection view matrix instead of the normal view matrix.
	reflectionViewMatrix = m_Camera->GetReflectionViewMatrix();

	// Get the world and projection matrices.
	m_D3D->GetWorldMatrix(worldMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);

	// Update the rotation variable each frame.
	rotation += (float)D3DX_PI * 0.005f;
	if(rotation > 360.0f)
	{
		rotation -= 360.0f;
	}
	D3DXMatrixRotationY(&worldMatrix, rotation);

	// Put the model vertex and index buffers on the graphics pipeline to prepare them for drawing.
	m_Model->Render(m_D3D->GetDeviceContext());

	// Render the model using the texture shader and the reflection view matrix.
	m_TextureShader->Render(m_D3D->GetDeviceContext(), m_Model->GetIndexCount(), worldMatrix, reflectionViewMatrix, 
							projectionMatrix, m_Model->GetTexture());

	// Reset the render target back to the original back buffer and not the render to texture anymore.
	m_D3D->SetBackBufferRenderTarget();

	return true;
}
```

&emsp;&emsp;与普通的渲染至纹理不同，这次我们采用了 `reflectionViewMatrix = m_Camera->GetReflectionViewMatrix();` 来代替普通的 view 矩阵，`ReflectionViewMatrix` 定义如下：

```c++
void CameraClass::RenderReflection(float height)
{
	D3DXVECTOR3 up, position, lookAt;
	float radians;


	// Setup the vector that points upwards.
	up.x = 0.0f;
	up.y = 1.0f;
	up.z = 0.0f;

	// Setup the position of the camera in the world.
	// For planar reflection invert the Y position of the camera.
	position.x = m_positionX;
	position.y = -m_positionY + (height * 2.0f);
	position.z = m_positionZ;

	// Calculate the rotation in radians.
	radians = m_rotationY * 0.0174532925f;

	// Setup where the camera is looking.
	lookAt.x = sinf(radians) + m_positionX;
	lookAt.y = position.y;
	lookAt.z = cosf(radians) + m_positionZ;

	// Create the view matrix from the three vectors.
	D3DXMatrixLookAtLH(&m_reflectionViewMatrix, &position, &lookAt, &up);

	return;
}
```



&emsp;&emsp;可以看到，我们使用一个高度作为参数来设置一个对于 x , z 平面的 Reflection 矩阵。在设置中也是简单的对 y 轴做以修改。

&emsp;&emsp;之后，我们使用类 `ReflectionShaderClass` 来渲染反射纹理：

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: reflectionshaderclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _REFLECTIONSHADERCLASS_H_
#define _REFLECTIONSHADERCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <D3D11.h>
#include <D3DX10math.h>
#include <fstream>
using namespace std;


////////////////////////////////////////////////////////////////////////////////
// Class name: ReflectionShaderClass
////////////////////////////////////////////////////////////////////////////////
class ReflectionShaderClass
{
private:
	struct MatrixBufferType
	{
		D3DXMATRIX world;
		D3DXMATRIX view;
		D3DXMATRIX projection;
	};

	struct ReflectionBufferType
	{
		D3DXMATRIX reflectionMatrix;
	};

public:
	ReflectionShaderClass();
	ReflectionShaderClass(const ReflectionShaderClass&);
	~ReflectionShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX, ID3D11ShaderResourceView*, 
				ID3D11ShaderResourceView*, D3DXMATRIX);

private:
	bool InitializeShader(ID3D11Device*, HWND, char*, char*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, char*);

	bool SetShaderParameters(ID3D11DeviceContext*, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX, ID3D11ShaderResourceView*, 
							 ID3D11ShaderResourceView*, D3DXMATRIX);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;
	ID3D11Buffer* m_reflectionBuffer;
};

#endif
```

&emsp;&emsp;这个和我们直接的 shader 类差不多，除了多出了一个新的矩阵变量，这个变量我们将传入着色器来为我们渲染正确的目标。

&emsp;&emsp;在我们的顶点着色器里，我们在它对于像素着色器的输出结构体里新增了一个属性，它用来保存纹理的输入的位置：

```c++
struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
    float4 reflectionPosition : TEXCOORD1;
};
```

&emsp;&emsp;同时，使用反射矩阵来设置新增的变量：

```c++
// Create the reflection projection world matrix.
reflectProjectWorld = mul(reflectionMatrix, projectionMatrix);
reflectProjectWorld = mul(worldMatrix, reflectProjectWorld);

// Calculate the input position against the reflectProjectWorld matrix.
output.reflectionPosition = mul(input.position, reflectProjectWorld);
```

&emsp;&emsp;像素着色器中，我们将反射位置映射到 0 - 1 之间作为纹理坐标使用，然后读取我们之前渲染的纹理，最后于平面纹理进行混合：

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: reflection.ps
////////////////////////////////////////////////////////////////////////////////


/////////////
// GLOBALS //
/////////////
Texture2D shaderTexture;
SamplerState SampleType;
Texture2D reflectionTexture;


//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
	float4 reflectionPosition : TEXCOORD1;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 ReflectionPixelShader(PixelInputType input) : SV_TARGET
{
	float4 textureColor;
    float2 reflectTexCoord;
    float4 reflectionColor;
    float4 color;


    // Sample the texture pixel at this location.
    textureColor = shaderTexture.Sample(SampleType, input.tex);
    
    // Calculate the projected reflection texture coordinates.
    reflectTexCoord.x = input.reflectionPosition.x / input.reflectionPosition.w / 2.0f + 0.5f;
    reflectTexCoord.y = -input.reflectionPosition.y / input.reflectionPosition.w / 2.0f + 0.5f;

	// Sample the texture pixel from the reflection texture using the projected texture coordinates.
    reflectionColor = reflectionTexture.Sample(SampleType, reflectTexCoord);

	// Do a linear interpolation between the two textures for a blend effect.
    color = lerp(textureColor, reflectionColor, 0.15f);

    return color;
}
```

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/gGLTxS/image.png)

&emsp;&emsp;（这篇文章开始使用近似于框架的方式来组织代码，可能显得比较突兀，稍后会专门再介绍一下这个框架的构造）

&emsp;&emsp;源代码：[DX11Tutorial-Reflection](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Reflection)