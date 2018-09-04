---
title: 【DirectX】24-法线贴图
date: 2018-04-08 13:15:09
tags: 
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 20: Bump Mapping](http://www.rastertek.com/dx11tut20.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将介绍使用纹理的法线贴图来实现感官上的凹凸效果。在之前我们的文章中我们使用一张纹理图片实现墙壁的纹理效果，但是在光照下因为纹理在一个平面的原因，法线总是向着一个方向。所以总是没有真实感，而这篇文章中我们用一张和纹理对应的法线图片进行贴图来增强真实感。

<!--more-->

&emsp;&emsp;我们的源纹理如下：

![1](https://image.ibb.co/er5LHH/1.png)

&emsp;&emsp;法线纹理如下：

![2](https://image.ibb.co/inay4x/2.png)

&emsp;&emsp;法线贴图其实就是将纹理上位置对应的法线存入纹理中，我们可以直接在 pixel 着色器中计算出像素点对应的法线从而避免了使用插值法得到的所有法线共向的问题。

&emsp;&emsp;在实现之前，或许需要了解以下内容：切线空间，切线副切线的计算（这些内容可能会找个时间去专门介绍也可能不会=。=）

&emsp;&emsp;首先我们给顶点结构体中添加了三个属性：

```c++
XMFLOAT3 nor;
XMFLOAT3 tgt;
XMFLOAT3 bit;
```

&emsp;&emsp;分别为法线，切线，副切线。这三个属性可以通过计算得到也可以直接给定（一般的模型会有这些属性），在这里我们试着使用顶点数据进行计算。

```c++
inline HRESULT CalculateTangentAndBiTangent(Vertex *&vertices, UINT *&indices, const UINT nNumIndices) {
	const UINT faceCount = nNumIndices / 3;
	FLOAT edge1[3], edge2[3];
	FLOAT tuVector[2], tvVector[2];
	XMFLOAT3 tangent, bitangent, normal;
	FLOAT f, l;
	for (UINT i = 0; i < faceCount; ++i) {
		Vertex& v1 = vertices[indices[3 * i]];
		Vertex& v2 = vertices[indices[3 * i + 1]];
		Vertex& v3 = vertices[indices[3 * i + 2]];
		/////////////////////////////////////////////// Edge
		edge1[0] = v2.pos.x - v1.pos.x;
		edge1[1] = v2.pos.y - v1.pos.y;
		edge1[2] = v2.pos.z - v1.pos.z;
		edge2[0] = v3.pos.x - v1.pos.x;
		edge2[1] = v3.pos.y - v1.pos.y;
		edge2[2] = v3.pos.z - v1.pos.z;

		/////////////////////////////////////////////// UV
		tuVector[0] = v2.tex.x - v1.tex.x;
		tvVector[0] = v2.tex.y - v1.tex.y;
		tuVector[1] = v3.tex.x - v1.tex.x;
		tvVector[1] = v3.tex.y - v1.tex.y;

		/////////////////////////////////////////////// f
		f = 1.0f / (tuVector[0] * tvVector[1] - tuVector[1] * tvVector[0]);
		/////////////////////////////////////////////// tangent
		tangent.x = (tvVector[1] * edge1[0] - tvVector[0] * edge2[0]) * f;
		tangent.y = (tvVector[1] * edge1[1] - tvVector[0] * edge2[1]) * f;
		tangent.z = (tvVector[1] * edge1[2] - tvVector[0] * edge2[2]) * f;
		l = sqrt(tangent.x * tangent.x + tangent.y * tangent.y + tangent.z * tangent.z);
		tangent.x /= l;
		tangent.y /= l;
		tangent.z /= l;
		v1.tgt = v2.tgt = v3.tgt = XMFLOAT3(tangent.x, tangent.y, tangent.z);
		///////////////////////////////////////////////// bitangent
		bitangent.x = (tuVector[0] * edge2[0] - tuVector[1] * edge1[0]) * f;
		bitangent.y = (tuVector[0] * edge2[1] - tuVector[1] * edge1[1]) * f;
		bitangent.z = (tuVector[0] * edge2[2] - tuVector[1] * edge1[2]) * f;
		l = sqrt(bitangent.x * bitangent.x + bitangent.y * bitangent.y + bitangent.z * bitangent.z);
		bitangent.x /= l;
		bitangent.y /= l;
		bitangent.z /= l;
		v1.bit = v2.bit = v3.bit = XMFLOAT3(bitangent.x, bitangent.y, bitangent.z);
		
        ////////////////////////////////////////////////// normal
		normal.x = tangent.y * bitangent.z - tangent.z * bitangent.y;
		normal.y = tangent.z * bitangent.x - tangent.x * bitangent.z;
		normal.z = tangent.x * bitangent.y - tangent.y * bitangent.x;
		l = sqrt(normal.x * normal.x + normal.y * normal.y + normal.z * normal.z);
		normal.x /= l;
		normal.y /= l;
		normal.z /= l;
		v1.nor = v2.nor = v3.nor = XMFLOAT3(normal.x, normal.y, normal.z);
	}
	return S_OK;
}
```

&emsp;&emsp;我们在 `InitCubeVertexAndIndexBuffer` 里写入定点数据和索引数据后调用这个方法。

&emsp;&emsp;同时，我们的顶点格式变了，所以 InputLayout 也要改变：

```c++
D3D11_INPUT_ELEMENT_DESC layout[5];
ZeroMemory(&layout[0], sizeof(layout[0]));
layout[0].SemanticName = "POSITION";
layout[0].Format = DXGI_FORMAT_R32G32B32_FLOAT;
layout[0].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
ZeroMemory(&layout[1], sizeof(layout[1]));
layout[1].SemanticName = "TEXCOORD";
layout[1].Format = DXGI_FORMAT_R32G32B32_FLOAT;
layout[1].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
layout[1].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
ZeroMemory(&layout[2], sizeof(layout[2]));
layout[2].SemanticName = "NORMAL";
layout[2].Format = DXGI_FORMAT_R32G32B32_FLOAT;
layout[2].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
layout[2].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
ZeroMemory(&layout[3], sizeof(layout[3]));
layout[3].SemanticName = "TANGENT";
layout[3].Format = DXGI_FORMAT_R32G32B32_FLOAT;
layout[3].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
layout[3].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
ZeroMemory(&layout[4], sizeof(layout[4]));
layout[4].SemanticName = "BITANGENT";
layout[4].Format = DXGI_FORMAT_R32G32B32_FLOAT;
layout[4].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
layout[4].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
```

&emsp;&emsp;我们现在有五个属性，在着色器里也需要变化：

```c++
struct vertexInputType {
	float4 pos : POSITION;
	float2 texcoord : TEXCOORD;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
};

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
};
```

&emsp;&emsp;在顶点着色器里我们对法线这些数据进行处理：

```c++
pixelInputType main( vertexInputType input ){
	pixelInputType output;

	output.pos = input.pos;	
	output.pos = mul(output.pos, world);
	output.pos = mul(output.pos, view);
	output.pos = mul(output.pos, projection);

	output.texcoord = input.texcoord;

	output.normal = mul(normalize(input.normal), (float)world);
	output.normal = normalize(output.normal);

	output.tangent = mul(normalize(input.tangent), (float)world);
	output.tangent = normalize(output.tangent);

	output.bitangent = mul(normalize(input.bitangent), (float)world);
	output.bitangent = normalize(output.bitangent);

		
	return output;
}
```

&emsp;&emsp;片段着色器里我们对法线贴图进行计算 `bumpNormal = (bumpMap.x * input.tangent) + (bumpMap.y * input.binormal) + (bumpMap.z * input.normal);` ，这将是我们计算凹凸法线的公式。然后我们会用这个法线去和光照做计算。

```c++
Texture2D tex[2];
SamplerState samp;

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD;
	float3 normal : NORMAL;
	float3 tangent : TANGENT;
	float3 bitangent : BITANGENT;
};


float4 main(pixelInputType input) : SV_TARGET{
	float3 lightDir;
	float4 bumpMap;
	float4 texColor;
	float3 bumpNormal;
	float color;
    
	lightDir = float3(-1.0f, -1.0f, 1.0f);
	bumpMap = tex[1].Sample(samp, input.texcoord);	
	bumpMap = bumpMap * 2.0f - 1.0f;
	bumpNormal = (bumpMap.x * input.tangent) + (bumpMap.y * input.bitangent) + (bumpMap.z * input.normal);	
	bumpNormal = normalize(bumpNormal);
	color = saturate(dot(bumpNormal , -lightDir));
	texColor = tex[0].Sample(samp, input.texcoord);

	return color * float4(1.0f,1.0f,1.0f,1.0f) * texColor;
} 
```

&emsp;&emsp;`lightDir` 是我们的光照方向，`bumpNormal` 则是我们计算出来的凹凸法线，使用它来进行光照计算后，效果如下：

![3](https://image.ibb.co/fOmKcH/image.png)

&emsp;&emsp;可以看到，比起直接使用插值法得到的数据法线计算光照，使用法线贴图后我们的纹理多了一些细节，更加具有真实感。

&emsp;&emsp;源代码：[DX11Tutorial-BumpMapping](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-BumpMapping)