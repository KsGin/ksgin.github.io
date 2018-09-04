---
title: 【DirectX】40-投影纹理映射
date: 2018-04-22 21:52:56
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 43: Projective Texturing](http://www.rastertek.com/dx11tut43.html)

《GPU Programming And Cg Language Primer 1rd Edition》

#### 学习记录

&emsp;&emsp;这篇文章中我们介绍投影纹理映射（Projective Texture Mapping），并且给出一个简单的示例。

<!--more-->

> &emsp;&emsp;投影纹理映射（Projective Texture Mapping）初由Segal在文章“Fast shadows and lighting effects using texture maaping”中提出，用于映射一个纹理到物体上，就像将幻灯片投影到墙上一样。该方法不需要在应用程序中指定顶点纹理坐标，实际上，投影纹理映射中使用的纹理坐标是在顶点着色程序中通过视点矩阵和投 影矩阵计算得到的，通常也被称作投影纹理坐标(coordinates in projective space)。 而我们常用的纹理坐标是在建模软件中通过手工调整纹理和3D模型的对应关系 而产生的。 
> &emsp;&emsp;投影纹理映射的目的是将纹理和三维空间顶点进行对应，这种对应的方法好 比“将纹理当作一张幻灯片，投影到墙上一样”。
>
> &emsp;&emsp;投影纹理映射真正的流程是“根据投影机（视点相机）的位置、投影角度，物体的坐标，求出每个顶点所对应的纹理坐标，然后依据纹理坐标去查询纹理 值”，也就是说，不是将纹理投影到墙上，而是把墙投影到纹理上。投影纹理坐 标的求得，也与纹理本身没有关系，而是由投影机的位置、角度，以及3D模型 的顶点坐标所决定。

&emsp;&emsp;这篇文章中我们使用一个 `ProjectionShaderClass` 类来实现渲染，这个类和 `TextureShaderClass` 不同的是它类似 `LightShaderClass` ，同时在 `MatrixBufferType` 里多了一个 `view` 和 `projection` 矩阵，其声明如下：

```c++
class ProjectionShaderClass
{
private:
	struct MatrixBufferType
	{
		DirectX::XMMATRIX world;
		DirectX::XMMATRIX view;
		DirectX::XMMATRIX projection;
		DirectX::XMMATRIX view2;
		DirectX::XMMATRIX projection2;
	};

	struct LightBufferType
	{
		DirectX::XMFLOAT4 ambientColor;
		DirectX::XMFLOAT4 diffuseColor;
		DirectX::XMFLOAT3 lightDirection;
		float padding;
	};

public:
	ProjectionShaderClass();
	ProjectionShaderClass(const ProjectionShaderClass&);
	~ProjectionShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*, 
		DirectX::XMFLOAT4, DirectX::XMFLOAT4, DirectX::XMFLOAT3, 
		    DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*);

private:
	bool InitializeShader(ID3D11Device*, HWND, CHAR*, CHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, CHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*, 
		DirectX::XMFLOAT4, DirectX::XMFLOAT4, DirectX::XMFLOAT3,
				 DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView*);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11SamplerState* m_sampleState;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11Buffer* m_lightBuffer;
};
```

&emsp;&emsp;其实现并没有什么多少特殊的地方，所以暂且不提。新增的 `ViewPoint` 类也只是简单的进行了数据封装，主要来看我们的着色器。

&emsp;&emsp;在 `VertexShader` 中，我们使用新增的 `viewMatrix2` 和 `projectionMatrix2` 矩阵变换 `viewPosition` 并且传递给像素着色器，如下：

```C++
/////////////
// GLOBALS //
/////////////
cbuffer MatrixBuffer
{
    matrix worldMatrix;
    matrix viewMatrix;
    matrix projectionMatrix;
    matrix viewMatrix2;
    matrix projectionMatrix2;
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
    float4 viewPosition : TEXCOORD1;
};


////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType ProjectionVertexShader(VertexInputType input)
{
    PixelInputType output;
    

    // Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

    // Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);

    // Store the position of the vertice as viewed by the projection view point in a separate variable.
    output.viewPosition = mul(input.position, worldMatrix);
    output.viewPosition = mul(output.viewPosition, viewMatrix2);
    output.viewPosition = mul(output.viewPosition, projectionMatrix2);

    // Store the texture coordinates for the pixel shader.
    output.tex = input.tex;
    
    // Calculate the normal vector against the world matrix only.
    output.normal = mul(input.normal, (float3x3)worldMatrix);
	
    // Normalize the normal vector.
    output.normal = normalize(output.normal);

    return output;
}
```

&emsp;&emsp;在 `PixelShader` 中，我们除了计算光照以外，我们将 `ViewPosition` 变换为 0 到 1 之内，然后使用它作为纹理坐标来渲染纹理：

```c++
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
    float4 color;
    float3 lightDir;
    float lightIntensity;
    float4 textureColor;
    float2 projectTexCoord;
    float4 projectionColor;
    // Set the default output color to the ambient light value for all pixels.
    color = ambientColor;

    // Invert the light direction for calculations.
    lightDir = -lightDirection;

    // Calculate the amount of light on this pixel.
    lightIntensity = saturate(dot(input.normal, lightDir));

    if(lightIntensity > 0.0f)
    {
        // Determine the light color based on the diffuse color and the amount of light intensity.
        color += (diffuseColor * lightIntensity);
    }

    // Saturate the light color.
    color = saturate(color);

    // Sample the pixel color from the texture using the sampler at this texture coordinate location.
    textureColor = shaderTexture.Sample(SampleType, input.tex);

    // Combine the light color and the texture color.
    color = color * textureColor;
    // Calculate the projected texture coordinates.
    projectTexCoord.x =  input.viewPosition.x / input.viewPosition.w / 2.0f + 0.5f;
    projectTexCoord.y = -input.viewPosition.y / input.viewPosition.w / 2.0f + 0.5f;
    // Determine if the projected coordinates are in the 0 to 1 range.  If it is then this pixel is inside the projected view port.
    if((saturate(projectTexCoord.x) == projectTexCoord.x) && (saturate(projectTexCoord.y) == projectTexCoord.y))
    {
        // Sample the color value from the projection texture using the sampler at the projected texture coordinate location.
        projectionColor = projectionTexture.Sample(SampleType, projectTexCoord);

        // Set the output color of this pixel to the projection texture overriding the regular color value.
        color = projectionColor;
    }
    return color;
}
```

&emsp;&emsp;在 `GraphicsClass` 中我们初始化相应类的对象，设定观察点以及对应的变换矩阵，部分代码如下：

```c++
m_ViewPoint = new ViewPointClass;
if (!m_ViewPoint) {
	return false;
}

m_ViewPoint->SetPosition(2.0f, 5.0f, -2.0f);
m_ViewPoint->SetLookAt(0.0f, 0.0f, 0.0f);
m_ViewPoint->SetProjectionParameters((float)(XM_PI / 2.0f), 1.0f, 0.1f, 100.0f);
m_ViewPoint->GenerateViewMatrix();
m_ViewPoint->GenerateProjectionMatrix();
```

&emsp;&emsp;最后，在 `Render` 里我们使用 `ProjectionShaderClass` 的对象来进行渲染，它接受多个矩阵参数：

```c++
bool GraphicsClass::Render()
{
	bool result;

	XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	XMMATRIX viewMatrix2, projectionMatrix2;

	m_D3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);
	m_Camera->Render();

	m_D3D->GetProjectionMatrix(projectionMatrix);
	m_Camera->GetViewMatrix(viewMatrix);
	m_D3D->GetWorldMatrix(worldMatrix);

	m_ViewPoint->GetViewMatrix(viewMatrix2);
	m_ViewPoint->GetProjectionMatrix(projectionMatrix2);

	m_Floor->Render(m_D3D->GetDeviceContext());
	result = m_ProjectionShader->Render(m_D3D->GetDeviceContext(), m_Floor->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix, 
					    m_Floor->GetTexture(), m_Light->GetAmbientColor(), m_Light->GetDiffuseColor(), m_Light->GetDirection(), 
					    viewMatrix2, projectionMatrix2, m_ProjectionTexture->GetTexture());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix and setup the translation for the cube model.
	m_D3D->GetWorldMatrix(worldMatrix);
	worldMatrix *= XMMatrixTranslation(0.0f, 2.0f, 0.0f);

	// Render the cube model using the projection shader.
	m_Model->Render(m_D3D->GetDeviceContext());
	result = m_ProjectionShader->Render(m_D3D->GetDeviceContext(), m_Model->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix, 
					    m_Model->GetTexture(), m_Light->GetAmbientColor(), m_Light->GetDiffuseColor(), m_Light->GetDirection(), 
					    viewMatrix2, projectionMatrix2, m_ProjectionTexture->GetTexture());
	if(!result)
	{
		return false;
	}

	m_D3D->EndScene();

	return true;
}
```

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/eo4e4H/image.png)

&emsp;&emsp;源代码：[**DX11Tutorial-ProjectiveTexturing**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-ProjectiveTexturing)

