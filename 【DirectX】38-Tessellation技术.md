---
title: 【DirectX】38-Tessellation技术
date: 2018-04-20 20:06:15
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 38: Hardware Tessellation](http://www.rastertek.com/dx11tut38.html)

[DirectX 11 Tessellation](http://www.nvidia.com.tw/object/tessellation_cn.html)



#### 学习记录

&emsp;&emsp;这篇文章主要介绍曲面细分（Tessellation）技术，本篇代码不多。主要在着色器方面，我们新增了两个着色器 `HullShader` 和 `DomainShader` 。

<!--more-->

&emsp;&emsp;这一篇中，我们介绍在 DX11 中实现曲面细分，我们将使用线框模式画一个正方形：

![1](https://image.ibb.co/fWjGxS/image.png)

&emsp;&emsp;经过细分以后，如下：

![2](https://image.ibb.co/dbaTBn/image.png)

> **&emsp;&emsp;曲面细分**，英文称Tessellation，如果直译的话应该译作“镶嵌化处理技术”。 由ATI开发，微软采纳后将其加入DirectX 11，成为DirectX 11的组成部分之一。 由于这种技术广泛的应用在**曲面**的几何处理上，因此国内翻译时通常译作“**曲面细分**”。 但实际上，这种技术不是只能用在**曲面**的**细分**处理上。

&emsp;&emsp;我们主要的代码增改都在着色器里，所以直接来看着色器代码吧，在 `VertexShader` 中，我们不再处理变换，而是直接将顶点属性传递给我们新增的 `HullShader` ：

```c++
struct VertexInputType
{
    float3 position : POSITION;
    float2 tex : TEXCOORD0;
};

struct HullInputType
{
    float3 position : POSITION;
    float2 tex : TEXCOORD0;
};

HullInputType TextureVertexShader(VertexInputType input)
{
	HullInputType output;
	output.position = input.position;
	output.tex = input.tex;
    return output;
}
```

&emsp;&emsp;在 `HullShader` 阶段，我们从代码中传入一个 `TessellationAmount` 来控制细分级别，对正方形的两个三角形进行细分（** 也可以直接细分四边形 **），我们的 `HullShader` 着色器接受一个 `TessellationBuffer` ：

```c++
cbuffer TessellationBuffer
{
	float tessellationAmount;
	float3 padding;
};
```

&emsp;&emsp;其全部代码如下：

```c++
cbuffer TessellationBuffer
{
	float tessellationAmount;
	float3 padding;
};


// 输入
struct HullInputType
{
	float3 position : POSITION;
    float2 tex : TEXCOORD0;
};

// 输出
struct HullOutputType
{
	float3 position : POSITION;
    float2 tex : TEXCOORD0;
};


// 输出修补程序常量数据。
struct HS_CONSTANT_DATA_OUTPUT
{
	float EdgeTessFactor[3]			: SV_TessFactor; // 例如，对于四象限域，将为 [4]
	float InsideTessFactor			: SV_InsideTessFactor; // 例如，对于四象限域，将为 Inside[2]
};

#define NUM_CONTROL_POINTS 3

// 修补程序常量函数
HS_CONSTANT_DATA_OUTPUT CalcHSPatchConstants(
	InputPatch< HullInputType , NUM_CONTROL_POINTS> ip,
	uint PatchID : SV_PrimitiveID)
{
	HS_CONSTANT_DATA_OUTPUT Output;

    // 设定三条边的细分因子
	Output.EdgeTessFactor[0] = tessellationAmount;	
	Output.EdgeTessFactor[1] = tessellationAmount;
	Output.EdgeTessFactor[2] = tessellationAmount;
    // 设定内部的细分因子
	Output.InsideTessFactor = tessellationAmount; 

	return Output;
}

[domain("tri")]	// 补丁类型为三角形
[partitioning("integer")]	// 细分模式整形
[outputtopology("triangle_cw")]	// 环绕方向设定为顺时针
[outputcontrolpoints(3)]	// 控制点个数
[patchconstantfunc("CalcHSPatchConstants")]	// 函数名
HullOutputType TextureHullShader( 
	InputPatch< HullInputType , NUM_CONTROL_POINTS> ip, 	
	uint i : SV_OutputControlPointID,
	uint PatchID : SV_PrimitiveID )
{
	HullOutputType output;
	// 直接赋值
	output.position = ip[i].position;	
	output.tex = ip[i].tex;

	return output;
}
```

&emsp;&emsp;`HallShader` 阶段我们完成了曲面细分因子的设置和片元控制点的输入，之后会经过固定渲染的细分阶段，这个阶段无需编程。然后才会进入 `DomainShader` 阶段，也是我们要新增的第二个着色器程序。在 `DomainShader` 着色器中，我们对细分后的顶点进行变换，即我们之前在 `VertexShader` 阶段实现的内容在这里实现了：

```c++
cbuffer MatrixBuffer
{
	matrix worldMatrix;
	matrix viewMatrix;
	matrix projectionMatrix;
};

struct HullOutputType
{
	float3 position : POSITION;
    float2 tex : TEXCOORD0;
};

struct PixelInputType
{
	float4 position : SV_POSITION;
    float2 tex : TEXCOORD0;
};


// 输出修补程序常量数据。
struct HS_CONSTANT_DATA_OUTPUT
{
	float EdgeTessFactor[3]			: SV_TessFactor; 
	float InsideTessFactor			: SV_InsideTessFactor; 
};

#define NUM_CONTROL_POINTS 3

[domain("tri")]
PixelInputType TextureDomainShader(
	HS_CONSTANT_DATA_OUTPUT input,
	float3 domain : SV_DomainLocation,
	const OutputPatch< HullOutputType , NUM_CONTROL_POINTS> patch)
{
	PixelInputType output;
	float3 vertexPosition;

	vertexPosition = domain.x * patch[0].position + domain.y * patch[1].position + domain.z * patch[2].position;

	// Calculate the position of the vertex against the world, view, and projection matrices
    output.position = mul(float4(vertexPosition , 1.0f), worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);

	output.tex = domain.x * patch[0].tex + domain.y * patch[1].tex + domain.z * patch[2].tex;

	return output;
}
```

&emsp;&emsp;需要注意的是在这里的时候我们的坐标操作都需要使用 `domain`  和 `patch` 计算得到。最后一个阶段自然是我们的像素着色器了，它和之前一样并没有改变。

&emsp;&emsp;想要观察到我们的曲面细分结果，我们需要使用线框模式绘制，因为直接绘制的画，会是这样：

![3](https://image.ibb.co/nkvLrn/image.png)

&emsp;&emsp;诺。

&emsp;&emsp;在 `D3DClass` 中我们暂且修改光栅化的配置使其使用线框绘制：

```c++
...
   	//rasterDesc.FillMode = D3D11_FILL_SOLID;
	rasterDesc.FillMode = D3D11_FILL_WIREFRAME;	// 设置为线框模式
...
```

&emsp;&emsp;然后在 `TextureShaderClass` 中，新增两个着色器以及对应缓冲：

```c++
class TextureShaderClass
{
private:
	struct MatrixBufferType
	{
		DirectX::XMMATRIX world;
		DirectX::XMMATRIX view;
		DirectX::XMMATRIX projection;
	};

	struct TessellationBufferType 
	{
		float tessellationAmount;
		DirectX::XMFLOAT3 padding;
	};

public:
	TextureShaderClass();
	TextureShaderClass(const TextureShaderClass&);
	~TextureShaderClass();

	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView* , float);

private:
	bool InitializeShader(ID3D11Device*, HWND, CHAR*, CHAR*, CHAR*, CHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, char*);

	bool SetShaderParameters(ID3D11DeviceContext*, DirectX::XMMATRIX, DirectX::XMMATRIX, DirectX::XMMATRIX, ID3D11ShaderResourceView* , float);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11HullShader* m_hullShader;
	ID3D11DomainShader* m_domainShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
	ID3D11SamplerState* m_sampleState;

	ID3D11Buffer* m_tessellationBuffer;
};
```

&emsp;&emsp;他们的创建什么的和顶点着色器这些并无区别，所以代码就不贴了（**请注意我们两个缓冲的位置，`MatrixBuffer` 已经不在顶点着色器了**）。

&emsp;&emsp;最后我们需要修改绘制图元，在 `ModelClass` 里修改为：

```c++
deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_3_CONTROL_POINT_PATCHLIST);
```

&emsp;&emsp;最终我们在 `GraphicsClass` 调用他们的绘制并且传入 `tessellationAmount` 为 3 ，效果如下（**我们使用的是三角形细分**）：

![5](https://image.ibb.co/jeJJcS/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-Tessellation](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Tessellation)