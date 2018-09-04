---
title: 【DirectX】34-玻璃和冰效果
date: 2018-04-17 21:04:12
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 32: Glass and Ice](http://www.rastertek.com/dx11tut32.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们简单介绍玻璃和冰效果。此次使用框架文件如下：

![1](https://image.ibb.co/fkKjcS/image.png)

<!--more-->

&emsp;&emsp;玻璃和冰的主要实现与我们之前介绍过的 [水](https://blog.ksgin.com/2018/04/15/【directx】32-水/) 实现类似，我们将它背后所能看见的物体渲染至纹理，然后通过法线贴图对纹理进行一定程度上的移位，达成效果。类似如下：

![2](https://image.ibb.co/nmvMrn/image.png)

&emsp;&emsp;在渲染这个的时候，我们需要以下几张纹理（使用冰来介绍）：

&emsp;&emsp;物体纹理：

![3](https://image.ibb.co/j6O7Wn/image.png)

&emsp;&emsp;冰的表面纹理：

![4](https://image.ibb.co/e4g5HS/image.png)

&emsp;&emsp;冰的法线纹理：

![5](https://image.ibb.co/cs9xWn/image.png)

&emsp;&emsp;在原教程中，对其渲染描述如下：

> Step 1: Render the scene that is behind the glass to a texture, this is called the refraction.
>
> Step 2: Project the refraction texture onto the glass surface.
>
> Step 3: Perturb the texture coordinates of the refraction texture using a normal map to simulate light traveling through glass.
>
> Step 4: Combine the perturbed refraction texture with a glass color texture for the final result.
>
> We will now examine how to implement each step for both glass and ice.

&emsp;&emsp;我们来看代码实现吧。首先来看这次新增的 `GlassShaderClass` 类，这个类也是从 `TextureShaderClass` 类修改而成，它相比于普通的纹理渲染类多了一个 `GlassBufferType` ，如下：

```c++
struct GlassBufferType 
{
	float refractionScale;
	DirectX::XMFLOAT3 padding;
};
```

&emsp;&emsp;`refractionScale` 决定了我们最后纹理的移位程度。

&emsp;&emsp;除多了一个缓冲之外，它的渲染使用三张纹理资源，如下：

```c++
bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , float);
```

&emsp;&emsp;其最终的声明如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: GlassShaderClass
////////////////////////////////////////////////////////////////////////////////
class GlassShaderClass
{
	struct MatrixBufferType
	{
		DirectX::XMMATRIX world;
		DirectX::XMMATRIX view;
		DirectX::XMMATRIX projection;
	};

	struct GlassBufferType 
	{
		float refractionScale;
		DirectX::XMFLOAT3 padding;
	};

public:
	GlassShaderClass();
	GlassShaderClass(const GlassShaderClass&);
	~GlassShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , float);

private:
	bool InitializeShader(ID3D11Device*, HWND, char*, char*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, char*);

	bool SetShaderParameters(ID3D11DeviceContext*, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , ID3D11ShaderResourceView* , float);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;
	ID3D11Buffer* m_glassBuffer;
};
```

&emsp;&emsp;当使用多张纹理的时候，我们再设置的时候需要一一设置，在 `SetShaderParameters` 方法中，我们对其进行配置：

```c++
// Set shader texture resource in the pixel shader.
deviceContext->PSSetShaderResources(0, 1, &texture);
deviceContext->PSSetShaderResources(1, 1, &normalTexture);
deviceContext->PSSetShaderResources(2, 1, &refractionTexture);
```

 &emsp;&emsp;限于篇幅，其它的实现基本与 `TextureShaderClass` 相同，使用多缓冲的操作我们也介绍过很多次了，所以在此暂且不表。

&emsp;&emsp;在其对应的顶点着色器中，我们向像素着色器里新传入一个变量 `refractionPosition` （折射顶点坐标）：

```c++
cbuffer MatrixBuffer
{
	matrix worldMatrix;
	matrix viewMatrix;
	matrix projectionMatrix;
};

struct VertexInputType 
{
	float4 position : POSITION;
	float2 texcoord : TEXCOORD0;
};

struct PixelInputType 
{
	float4 position : SV_POSITION;
	float2 texcoord : TEXCOORD0;
	float4 refractionPosition : TEXCOORD1;
};

PixelInputType GlassVertexShader(VertexInputType input)
{
	PixelInputType output;
	matrix viewProjectWorld;

	input.position.w = 1.0f;

	output.position = mul(input.position, worldMatrix);
	output.position = mul(output.position, viewMatrix);
	output.position = mul(output.position, projectionMatrix);

	output.texcoord = input.texcoord;

	viewProjectWorld = mul(viewMatrix, projectionMatrix);
	viewProjectWorld = mul(worldMatrix, viewProjectWorld);

	output.refractionPosition = mul(input.position, viewProjectWorld);

	return output;
}
```

&emsp;&emsp;像素着色器中，我们使用法线纹理读出的法线对折射纹理进行移位，然后使用最终使用折射纹理和冰本身纹理进行混合：

```c++
SamplerState sampleType;

Texture2D colorTex : register(t0);
Texture2D normalTex : register(t1);
Texture2D refractionTex : register(t2);

cbuffer GlassBuffer
{
	float refractionScale;
	float3 padding;
};

struct PixelInputType 
{
	float4 position : SV_POSITION;
	float2 texcoord : TEXCOORD0;
	float4 refractionPosition : TEXCOORD1;
};

float4 GlassPixelShader(PixelInputType input) : SV_TARGET
{
	float2 refractTexcoord;
	float4 normalMap;
	float3 normal;
	float4 refractionColor;
	float4 textureColor;
	float4 color;
 
	refractTexcoord.x = input.refractionPosition.x / input.refractionPosition.w / 2.0f + 0.5f;
	refractTexcoord.y = input.refractionPosition.y / input.refractionPosition.w / 2.0f + 0.5f;

	normalMap = normalTex.Sample(sampleType, input.texcoord);
	normal = (normalMap.xyz * 2.0f) - 1.0f;

	refractTexcoord = refractTexcoord + (normal.xy * refractionScale);

	refractionColor = refractionTex.Sample(sampleType, refractTexcoord);

	textureColor = colorTex.Sample(sampleType, input.texcoord);

	color = lerp(refractionColor, textureColor, 0.5f);

	return color;
}
```

&emsp;&emsp;由于我们的玻璃模型需要两个纹理贴图（颜色纹理和法线纹理），所以我们修改 `ModelClass` ，使其可以适应两张纹理，修改后如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: ModelClass
////////////////////////////////////////////////////////////////////////////////
class ModelClass
{
private:
	struct VertexType
	{
		DirectX::XMFLOAT3 position;
	    DirectX::XMFLOAT2 texture;
		DirectX::XMFLOAT3 normal;
	};

	struct ModelType
	{
		float x, y, z;
		float tu, tv;
		float nx, ny, nz;
	};

public:
	ModelClass();
	ModelClass(const ModelClass&);
	~ModelClass();

	bool Initialize(ID3D11Device*, char*, char* , char* = nullptr);
	void Shutdown();
	void Render(ID3D11DeviceContext*);

	int GetIndexCount();
	ID3D11ShaderResourceView* GetTexture();
	ID3D11ShaderResourceView* GetNormalTexture();

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	void RenderBuffers(ID3D11DeviceContext*);

	bool LoadTexture(ID3D11Device*, char*);
	void ReleaseTexture();

	bool LoadNormalTexture(ID3D11Device*, char*);
	void ReleaseNormalTexture();

	bool LoadModel(char*);
	void ReleaseModel();

private:
	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
	int m_vertexCount, m_indexCount;
	TextureClass* m_Texture;
	TextureClass* m_NormalTexture;
	ModelType* m_model;
};

```

&emsp;&emsp;我们新增了 `m_NormalTexture` 成员变量以及它的读取和释放函数，并且在构造函数中使用了默认为空的第三个参数，这样我们可以根据自己需求选择是否使用模型的法线纹理。

&emsp;&emsp;最后来看 `GraphicsClass` ，我们新增了 `m_WindowModel` ，`m_GlassShader` ，`m_RenderTexture`成员变量以及他们的初始化等操作。

&emsp;&emsp;最终渲染时，我们首先渲染我们的 cube 至纹理，然后就可以使用纹理来渲染冰效果。部分代码如下：

```c++
bool GraphicsClass::Frame()
{
	static float rotation = 0.0f;
	bool result;

	rotation += (float)XM_PI * 0.005f;
	if (rotation > 360.0f) {
		rotation -= 360.0f;
	}
	// Set the position and rotation of the camera.
	m_Camera->SetPosition(0.0f, 0.0f, -10.0f);

	result = RenderToTexture(rotation);
	if (!result) {
		return false;
	}

	result = Render(rotation);
	if (!result) {
		return false;
	}
	return true;
}

bool GraphicsClass::RenderToTexture(float rotation) {
	bool result;
	XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	m_RenderTexture->SetRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView());
	m_RenderTexture->ClearRenderTarget(m_D3D->GetDeviceContext(), m_D3D->GetDepthStencilView(),
		0.0f, 0.0f, 0.0f, 1.0f);
	m_Camera->Render();
	m_D3D->GetWorldMatrix(worldMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);
	m_Camera->GetViewMatrix(viewMatrix);

	worldMatrix *= XMMatrixRotationY(rotation);

	m_Model->Render(m_D3D->GetDeviceContext());

	result = m_TextureShader->Render(m_D3D->GetDeviceContext(), 
		m_Model->GetIndexCount(), 
		worldMatrix, viewMatrix, projectionMatrix, 
		m_Model->GetTexture());
	if (!result) {
		return false;
	}

	m_D3D->SetBackBufferRenderTarget();
	return true;
}

bool GraphicsClass::Render(float rotation)
{
	bool result;
	XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	float refracionScale = 0.1f;

	m_D3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);
	m_Camera->Render();

	m_D3D->GetWorldMatrix(worldMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);
	m_Camera->GetViewMatrix(viewMatrix);

	worldMatrix *= XMMatrixRotationY(rotation);

	m_Model->Render(m_D3D->GetDeviceContext());
	result = m_TextureShader->Render(
		m_D3D->GetDeviceContext(), 
		m_Model->GetIndexCount(), 
		worldMatrix, 
		viewMatrix, 
		projectionMatrix, 
		m_Model->GetTexture());
	if (!result) {
		return false;
	}

	m_D3D->GetWorldMatrix(worldMatrix);
	worldMatrix *= XMMatrixTranslation(0.0f, 0.0f, -1.5f);

	m_WindowModel->Render(m_D3D->GetDeviceContext());
	result = m_GlassShader->Render(
		m_D3D->GetDeviceContext(),
		m_WindowModel->GetIndexCount(),
		worldMatrix,
		viewMatrix,
		projectionMatrix,
		m_WindowModel->GetTexture() , 
		m_WindowModel->GetNormalTexture(),
		m_RenderTexture->GetShaderResourceView(),
		refracionScale
	);
	if (!result) {
		return false;
	}


	m_D3D->EndScene();

	return true;
}
```

&emsp;&emsp;我们在这里渲染了两次 cube ，第一次渲染至纹理其纹理供我们渲染冰所用，第二次渲染至后台呈现所用。最终效果如下：

![6](https://image.ibb.co/nmvMrn/image.png)

&emsp;&emsp;（渲染玻璃只是更换颜色法线贴图就可以）

&emsp;&emsp;源代码：[**DX11Tutorial-GlassAndIce**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-GlassAndIce)



