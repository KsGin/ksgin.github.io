---
title: 【DirectX】45-渲染透明物体的阴影
date: 2018-04-26 13:25:40
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 49: Shadow Mapping and Transparency](http://www.rastertek.com/dx11tut49.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们介绍对透明物体的阴影渲染，我们将渲染一棵树的阴影（树的叶子通常以带透明度的贴图而非实体模型呈现）。

<!--more-->

&emsp;&emsp;当我们渲染一棵树的时候我们使用部分透明的贴图来绘制树叶，因为如果为单个树叶建模实在是太麻烦了。所以树叶一般情况下都只是一个矩形，如果我们不开启透明度，那么绘制的结果如下：

![1](https://image.ibb.co/eTxSWx/image.png)

&emsp;&emsp;开启了透明之后，渲染结果如下：

![2](https://image.ibb.co/dQTvJc/image.png)

&emsp;&emsp;树的渲染已经正常了，但是其阴影还是错误的样子。这是因为我们绘制深度图的时候是针对于模型，而透明度则是保存在纹理中的。所以要渲染正常的树阴影我们需要修改我们的深度图使其支持带纹理的深度映射。

&emsp;&emsp;首先我们修改 `depth.vs` 和 `depth.ps` ，使其支持纹理并且当纹理的不透明度低于某一个值时候我们不计算它的深度，如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: depth.vs
////////////////////////////////////////////////////////////////////////////////


/////////////
// GLOBALS //
/////////////
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
};

struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 depthPosition : TEXTURE0;
	float2 tex : TEXCOORD0;
};


////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType DepthVertexShader(VertexInputType input)
{
    PixelInputType output;
    
    
	// Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

	// Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);

	// Store the position value in a second input value for depth value calculations.
	output.depthPosition = output.position;

	output.tex = input.tex;
	
	return output;
}

////////////////////////////////////////////////////////////////////////////////
// Filename: depth.ps
////////////////////////////////////////////////////////////////////////////////

SamplerState samplerType : register(s0);
Texture2D shaderTexture : register(t0);

//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 depthPosition : TEXTURE0;
	float2 tex : TEXCOORD0;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 DepthPixelShader(PixelInputType input) : SV_TARGET
{
	float depthValue;
	float4 color;
	
	// Get the alpha value from the texture
	float alpha = shaderTexture.Sample(samplerType , input.tex).a;
	
	// discard render if the alpha less than 0.8
	if(alpha < 0.8) discard;
	
	// Get the depth value of the pixel by dividing the Z pixel depth by the homogeneous W coordinate.
	depthValue = input.depthPosition.z / input.depthPosition.w;

	color = float4(depthValue, depthValue, depthValue, 1.0f);

	return color;
}
```

&emsp;&emsp;主要的改变就在这里，在 `DepthShaderClass` 里它只是修改为支持读取纹理，其声明如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: DepthShaderClass
////////////////////////////////////////////////////////////////////////////////
class DepthShaderClass
{
private:
	struct MatrixBufferType
	{
		DirectX::XMMATRIX world;
		DirectX::XMMATRIX view;
		DirectX::XMMATRIX projection;
	};

public:
	DepthShaderClass();
	DepthShaderClass(const DepthShaderClass&);
	~DepthShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*);

private:
	bool InitializeShader(ID3D11Device*, HWND, CHAR*, CHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, CHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;
};
```

&emsp;&emsp;在渲染深度图时，我们使用模型对应的纹理，部分代码如下：

```c++
bool GraphicsClass::RenderSceneToTexture()
{
	DirectX::XMMATRIX worldMatrix, lightViewMatrix, lightOrthoMatrix;
	float posX, posY, posZ;
	bool result;

	// Turn on alpha blending
	m_D3D->TurnOnAlphaBlending();

	// Set the render target to be the render to texture.
	m_RenderTexture->SetRenderTarget(m_D3D->GetDeviceContext());

	// Clear the render to texture.
	m_RenderTexture->ClearRenderTarget(m_D3D->GetDeviceContext(), 0.0f, 0.0f, 0.0f, 1.0f);

	// Generate the light view matrix based on the light's position.
	m_Light->GenerateViewMatrix();

	// Get the world matrix from the d3d object.
	m_D3D->GetWorldMatrix(worldMatrix);

	// Get the view and orthographic matrices from the light object.
	m_Light->GetViewMatrix(lightViewMatrix);
	m_Light->GetOrthoMatrix(lightOrthoMatrix);

	// Setup the translation matrix for the cube model.
	worldMatrix *= DirectX::XMMatrixScaling(0.05f, 0.05f, 0.05f);

	// Render the cube model with the depth shader.
	m_TrunkModel->Render(m_D3D->GetDeviceContext());
	result = m_DepthShader->Render(m_D3D->GetDeviceContext(), m_TrunkModel->GetIndexCount(), worldMatrix, lightViewMatrix, lightOrthoMatrix , m_TrunkModel->GetTexture());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix.
	m_D3D->GetWorldMatrix(worldMatrix);

	// Setup the translation matrix for the sphere model.
	worldMatrix *= DirectX::XMMatrixScaling(0.05f, 0.05f, 0.05f);

	// Render the sphere model with the depth shader.
	m_LeavesModel->Render(m_D3D->GetDeviceContext());
	result = m_DepthShader->Render(m_D3D->GetDeviceContext(), m_LeavesModel->GetIndexCount(), worldMatrix, lightViewMatrix, lightOrthoMatrix , m_LeavesModel->GetTexture());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix.
	m_D3D->GetWorldMatrix(worldMatrix);

	// Setup the translation matrix for the ground model.
	m_GroundModel->GetPosition(posX, posY, posZ);
	worldMatrix *= DirectX::XMMatrixTranslation(posX, posY, posZ);

	// Render the ground model with the depth shader.
	m_GroundModel->Render(m_D3D->GetDeviceContext());
	result = m_DepthShader->Render(m_D3D->GetDeviceContext(), m_GroundModel->GetIndexCount(), worldMatrix, lightViewMatrix, lightOrthoMatrix , m_GroundModel->GetTexture());
	if(!result)
	{
		return false;
	}

	
	// Turn off alpha blending
	m_D3D->TurnOffAlphaBlending();

	// Reset the render target back to the original back buffer and not the render to texture anymore.
	m_D3D->SetBackBufferRenderTarget();

	// Reset the viewport back to the original.
	m_D3D->ResetViewport();

	return true;
}


bool GraphicsClass::Render()
{
	DirectX::XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	DirectX::XMMATRIX lightViewMatrix, lightOrthoMatrix;
	bool result;

	// First render the scene to a texture.
	result = RenderSceneToTexture();
	if(!result)
	{
		return false;
	}

	// Clear the buffers to begin the scene.
	m_D3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);

	// Turn on alpha blending
	m_D3D->TurnOnAlphaBlending();

	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Generate the light view matrix based on the light's position.
	m_Light->GenerateViewMatrix();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	m_Camera->GetViewMatrix(viewMatrix);
	m_D3D->GetWorldMatrix(worldMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);

	// Get the light's view and projection matrices from the light object.
	m_Light->GetViewMatrix(lightViewMatrix);
	m_Light->GetOrthoMatrix(lightOrthoMatrix);

	worldMatrix *= DirectX::XMMatrixScaling(0.05f, 0.05f, 0.05f);
	// Put the cube model vertex and index buffers on the graphics pipeline to prepare them for drawing.
	m_TrunkModel->Render(m_D3D->GetDeviceContext());

	// Render the model using the shadow shader.
	result = m_ShadowShader->Render(m_D3D->GetDeviceContext(), m_TrunkModel->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix, lightViewMatrix, 
									lightOrthoMatrix, m_TrunkModel->GetTexture(), m_RenderTexture->GetShaderResourceView(), m_Light->GetDirection(),
									m_Light->GetAmbientColor(), m_Light->GetDiffuseColor());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix.
	m_D3D->GetWorldMatrix(worldMatrix);

	worldMatrix *= DirectX::XMMatrixScaling(0.05f, 0.05f, 0.05f);

	// Put the model vertex and index buffers on the graphics pipeline to prepare them for drawing.
	m_LeavesModel->Render(m_D3D->GetDeviceContext());
	result = m_ShadowShader->Render(m_D3D->GetDeviceContext(), m_LeavesModel->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix, lightViewMatrix, 
									lightOrthoMatrix, m_LeavesModel->GetTexture(), m_RenderTexture->GetShaderResourceView(), m_Light->GetDirection(), 
									m_Light->GetAmbientColor(), m_Light->GetDiffuseColor());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix.
	m_D3D->GetWorldMatrix(worldMatrix);

	// Render the ground model using the shadow shader.
	m_GroundModel->Render(m_D3D->GetDeviceContext());
	result = m_ShadowShader->Render(m_D3D->GetDeviceContext(), m_GroundModel->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix, lightViewMatrix,
									lightOrthoMatrix, m_GroundModel->GetTexture(), m_RenderTexture->GetShaderResourceView(), m_Light->GetDirection(), 
									m_Light->GetAmbientColor(), m_Light->GetDiffuseColor());
	if(!result)
	{
		return false;
	}


	// Turn off alpha blending
	m_D3D->TurnOffAlphaBlending();

	// Present the rendered scene to the screen.
	m_D3D->EndScene();

	return true;
}
```

&emsp;&emsp;最终效果：

![3](https://image.ibb.co/jPxKBx/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-ShadowMappingAndTransparency](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-ShadowMappingAndTransparency)

