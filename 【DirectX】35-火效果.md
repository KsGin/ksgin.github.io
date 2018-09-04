---
title: 【DirectX】35-火效果
date: 2018-04-19 19:12:16
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 33: Fire](http://www.rastertek.com/dx11tut33.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们简单介绍火焰的一种常用实现，此次使用的框架文件如下：

![1](https://image.ibb.co/iochHS/image.png)

<!--more-->

&emsp;&emsp;火焰的实现也是使用纹理贴图，辅以随时间而进行的纹理坐标扰动而实现。火焰的纹理以及纹理使用如下（原文）：

> One of the most realistic ways to create a fire effect in DirectX is to use a noise texture and to perturb the sampling of that texturein the same way we have for water, glass, and ice.The only major difference is in the manipulation of the noise texture and the specific way we perturb the texture coordinates.
>
> First start with a grey scale noise texture such as follows:
>
> ![img](http://www.rastertek.com/pic0150.gif)
>
> These noise textures can be procedurally generated with several different graphics manipulation programs.The key is to produce one that has noise properties specific to a good looking fire.
>
> The second texture needed for fire effect will be a texture composed of fire colored noise and a flame outline.For example the following texture is composed of a texture that uses perlin noise with fire colors and a texture of a small flame.If you look closely at the middle bottom part of the texture you can see the umbra and penumbra of the flame:
>
> ![img](http://www.rastertek.com/pic0151.gif)
>
> And finally you will need an alpha texture for transparency of the flame so that the final shape is very close to that of a small flame.This can be rough as the perturbation will take care of making it take shape of a good flame:
>
> ![img](http://www.rastertek.com/pic0152.gif)
>
> Now that we have the three textures required for the fire effect we can explain how the shader works.The first thing we do is take the noise texture and create three separate textures from it.The three separate textures are all based on the original noise texture except that they are scaled differently.These scales are called octaves as they are simply repeated tiles of the original noise texture to create numerous more detailed noise textures.The following three noise textures are scaled (tiled) by 1, 2, and 3:
>
> ![img](http://www.rastertek.com/pic0153.gif)
>
> ![img](http://www.rastertek.com/pic0154.gif)
>
> ![img](http://www.rastertek.com/pic0155.gif)
>
> We are also going to scroll all three noise textures upwards to create an upward moving noise which will correlate with a fire that burns upwards.The scroll speed will be set differently for all three so that each moves at a different rate.We will eventually combine the three noise textures so having them move at different rates adds dimension to the fire.
>
> The next step is to convert the three noise textures from the (0, 1) range into the (-1, +1) range.This has the effect of removing some of the noise and creating noise that looks closer to fire.For example the first noise texture now looks like the following:
>
> ![img](http://www.rastertek.com/pic0156.gif)
>
> With all three noise textures in the (-1, +1) range we can now add distortion to each texture along the x and y coordinates.The distortion modifies the noise textures by scaling down the noise similar to how the refraction scale in the water, glass, and ice shaders did so.Once each noise texture is distorted we can then combine all three noise textures to produce a final noise texture.
>
> Note that the distortion here is only applied to the x and y coordinates since we will be using them as a look up table for the x and y texture samplingjust like we do with normal maps.The z coordinate is ignored as it has no use when sampling textures in 2D.
>
> Remember also that each frame the three noise textures are scrolling upwards at different rates so combining them creates an almost organized flowing noisethat looks like fire.
>
> Now the next very important step is to perturb the original fire color texture.In other shaders such as water, glass, and ice we usually use a normal map at this point to perturb the texture sampling coordinates.However in fire we use the noise as our perturbed texture sampling coordinates.But before we do that we want to also perturb the noise itself.  We will use a distortion scale and bias to distort the noise higher at the top of the texture and less at the bottom of the textureto create a solid flame at the base and flame licks at the top.  In other words we perturb the noise along the Y axis using an increased amount of distortion as we go up the noise texture from the bottom.
>
> We now use the perturbed final noise texture as our look up table for texture sampling coordinates that will be used to sample the fire color texture.Note that we use Clamp instead of Wrap for the fire color texture or we will get flame licks wrapping around to the bottom which would ruin the look.The fire color texture sampled using the perturbed noise now looks like the following:
>
> ![img](http://www.rastertek.com/pic0158.gif)
>
> Now that we have an animated burning square that looks fairly realistic we need a way of shaping it into more of a flame.To do so we turn on blending and use the alpha texture we created earlier.The trick is to sample the alpha texture using the same perturbed noise to make the alpha look like a burning flame.We also need to use Clamp instead of Wrap for sampling to prevent the flames from wrapping around to the bottom.When we do so we get the following result from the sampled perturbed alpha: 
>
> ![img](http://www.rastertek.com/pic0159.gif)
>
> To complete the effect we set the perturbed alpha value to be the alpha channel of the perturbed fire color texture and the blending takes care of the rest:
>
> ![img](http://www.rastertek.com/pic0160.gif)
>
> When you see this animated it looks very realistic.

&emsp;&emsp;实现中，我们首先修改 `ModelClass` 使其接受一个模型和三个纹理，声明如下：

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

	bool Initialize(ID3D11Device*, char*, char* , char* , char*);
	void Shutdown();
	void Render(ID3D11DeviceContext*);

	int GetIndexCount();
	ID3D11ShaderResourceView* GetTexture1();
	ID3D11ShaderResourceView* GetTexture2();
	ID3D11ShaderResourceView* GetTexture3();

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	void RenderBuffers(ID3D11DeviceContext*);

	bool LoadTexture(ID3D11Device*, char* , char* , char*);
	void ReleaseTexture();

	bool LoadModel(char*);
	void ReleaseModel();

private:
	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
	int m_vertexCount, m_indexCount;
	TextureClass* m_Texture1;
	TextureClass* m_Texture2;
	TextureClass* m_Texture3;
	ModelType* m_model;
};
```

&emsp;&emsp;我们的 `FireShaderClass` 中，我们除了一般的 `MatrixBufferType` 缓冲结构体外，还需要两个为生成火焰效果而添加的缓冲结构体：

```c++
struct NoiseBufferType
{
	float frameTime;
	DirectX::XMFLOAT3 scrollSpeeds;
	DirectX::XMFLOAT3 scales;
	float padding;
};

struct DistortionBufferType
{
	DirectX::XMFLOAT2 distortion1;
	DirectX::XMFLOAT2 distortion2;
	DirectX::XMFLOAT2 distortion3;
	float distortionScale;
	float distortionBias;
};
```

&emsp;&emsp;为省篇幅，其创建缓冲和填充数据的过程不贴出来了。主要看我们的 HLSL 代码。我们的 `NoiseBufferType` 缓冲是送给顶点着色器的，使用它们生成多个不同的纹理坐标：

```c++
/////////////
// GLOBALS //
/////////////
cbuffer MatrixBuffer
{
	matrix worldMatrix;
	matrix viewMatrix;
	matrix projectionMatrix;
};

cbuffer NoiseBuffer
{
	float frameTime;
	float3 scrollSpeeds;
	float3 scales;
	float padding;
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
    float2 tex : TEXCOORD0;
   	float2 texCoords1 : TEXCOORD1;
	float2 texCoords2 : TEXCOORD2;
	float2 texCoords3 : TEXCOORD3;
};


////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType FireVertexShader(VertexInputType input)
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

    // Compute texture coordinates for first noise texture using the first scale and upward scrolling speed values.
	output.texCoords1 = (input.tex * scales.x);
	output.texCoords1.y = output.texCoords1.y + (frameTime * scrollSpeeds.x);

    // Compute texture coordinates for second noise texture using the second scale and upward scrolling speed values.
	output.texCoords2 = (input.tex * scales.y);
	output.texCoords2.y = output.texCoords2.y + (frameTime * scrollSpeeds.y);

    // Compute texture coordinates for third noise texture using the third scale and upward scrolling speed values.
	output.texCoords3 = (input.tex * scales.z);
	output.texCoords3.y = output.texCoords3.y + (frameTime * scrollSpeeds.z);
	
    return output;
}
```

&emsp;&emsp;像素着色器中，我们使用由 CPU 代码传进来的缓冲和顶点着色器传进来的多个纹理坐标，从 `noiseTexture ` 中混合得到最终的 nosie 纹理坐标值，并且使用它从火焰纹理和透明纹理中取值。

![2](https://image.ibb.co/cNeXj7/image.png)

&emsp;&emsp;源代码：[**DX11Tutorial-Fire**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Fire)

