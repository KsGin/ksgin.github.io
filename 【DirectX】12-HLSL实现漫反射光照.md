---
title: 【DirectX】12-HLSL实现漫反射光照
date: 2018-03-27 14:47:18
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;这篇文章中，我们将实现在 HLSL 下的漫反射光照绘制。漫反射的原理我们在 OpenGL 学习笔记中已经详细提及，这里就省下篇幅了。

<!--more-->

&emsp;&emsp;首先我们修改之前的立方体代码来给漫反射光照做准备。我们将绘制一个缓慢转动的立方体模型并为他贴图，然后我们需要一个光源，使用一个比较小的白色立方体代替。

&emsp;&emsp;重新定义顶点数据结构：

```c++
struct Vertex
{
	XMFLOAT3 pos;
	XMFLOAT2 texcoord;
	XMFLOAT3 normal;	// 计算漫反射光需要用到法线
};
```

&emsp;&emsp;之后，我们更改立方体的顶点（由于法线的原因，导致我们会将顶点的个数扩展到24个），并且更新索引：

```c++
	Vertex vertices[] = {
		// 正面
		{ XMFLOAT3(-0.5f, -0.5f, -0.5f), XMFLOAT2(0, 1) , XMFLOAT3(0.0f , 0.0f , -1.0f)},
		{ XMFLOAT3(-0.5f, +0.5f, -0.5f), XMFLOAT2(0, 0) , XMFLOAT3(0.0f , 0.0f , -1.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, -0.5f), XMFLOAT2(1, 0) , XMFLOAT3(0.0f , 0.0f , -1.0f)},
		{ XMFLOAT3(+0.5f, -0.5f, -0.5f), XMFLOAT2(1, 1) , XMFLOAT3(0.0f , 0.0f , -1.0f)},

		// 背面
		{ XMFLOAT3(-0.5f, -0.5f, +0.5f), XMFLOAT2(1, 1) , XMFLOAT3(0.0f , 0.0f , +1.0f)},
		{ XMFLOAT3(-0.5f, +0.5f, +0.5f), XMFLOAT2(1, 0) , XMFLOAT3(0.0f , 0.0f , +1.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, +0.5f), XMFLOAT2(0, 0) , XMFLOAT3(0.0f , 0.0f , +1.0f)},
		{ XMFLOAT3(+0.5f, -0.5f, +0.5f), XMFLOAT2(0, 1) , XMFLOAT3(0.0f , 0.0f , +1.0f)},

		// 上面
		{ XMFLOAT3(-0.5f, +0.5f, -0.5f), XMFLOAT2(0, 1) , XMFLOAT3(0.0f , +1.0f , 0.0f)},
		{ XMFLOAT3(-0.5f, +0.5f, +0.5f), XMFLOAT2(0, 0) , XMFLOAT3(0.0f , +1.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, +0.5f), XMFLOAT2(1, 0) , XMFLOAT3(0.0f , +1.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, -0.5f), XMFLOAT2(1, 1) , XMFLOAT3(0.0f , +1.0f , 0.0f)},

		// 下面
		{ XMFLOAT3(-0.5f, -0.5f, -0.5f), XMFLOAT2(1, 1) , XMFLOAT3(0.0f , -1.0f , 0.0f)},
		{ XMFLOAT3(-0.5f, -0.5f, +0.5f), XMFLOAT2(1, 0) , XMFLOAT3(0.0f , -1.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, -0.5f, +0.5f), XMFLOAT2(0, 0) , XMFLOAT3(0.0f , -1.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, -0.5f, -0.5f), XMFLOAT2(0, 1) , XMFLOAT3(0.0f , -1.0f , 0.0f)},

		// 左侧
		{ XMFLOAT3(-0.5f, -0.5f, +0.5f), XMFLOAT2(0, 1) , XMFLOAT3(-1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(-0.5f, +0.5f, +0.5f), XMFLOAT2(0, 0) , XMFLOAT3(-1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(-0.5f, +0.5f, -0.5f), XMFLOAT2(1, 0) , XMFLOAT3(-1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(-0.5f, -0.5f, -0.5f), XMFLOAT2(1, 1) , XMFLOAT3(-1.0f , 0.0f , 0.0f)},

		// 右侧
		{ XMFLOAT3(+0.5f, -0.5f, -0.5f), XMFLOAT2(0, 1) , XMFLOAT3(+1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, -0.5f), XMFLOAT2(0, 0) , XMFLOAT3(+1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, +0.5f, +0.5f), XMFLOAT2(1, 0) , XMFLOAT3(+1.0f , 0.0f , 0.0f)},
		{ XMFLOAT3(+0.5f, -0.5f, +0.5f), XMFLOAT2(1, 1) , XMFLOAT3(+1.0f , 0.0f , 0.0f)},
	};

UINT indices[] = {
		// 前
		0, 1, 2,
		0, 2, 3,
		// 后
		4, 6, 5,
		4, 7, 6,
		// 左
		16, 17, 18,
		16, 18, 19,
		// 右
		20, 21, 22,
		20, 22, 23,
		// 上
		8, 9, 10,
		8, 10, 11,
		// 下
		12, 14, 13,
		12, 15, 14
	};
```

&emsp;&emsp;这是我们绘制立方体的时候使用的新的数据（具体绘制过程有兴趣的可以去看源码），由于我们需要绘制光源，所以需要简单的光源立方体使用的着色器程序，如下：

```c++
// lightVertexShader.hlsl
cbuffer ConstantBuffer : register(b0) {
	matrix world;
	matrix view;
	matrix projection;
}

float4 main( float4 pos : POSITION ) : SV_POSITION{
	pos = mul(pos , world);
	pos = mul(pos , view);
	pos = mul(pos , projection);
	return pos;
}

//lightPixelShader.hlsl
struct pixelInputType {
	float4 pos : SV_POSITION;
};

float4 main(float4 pos : SV_POSITION) : SV_TARGET{
	return float4(1.0f,1.0f,1.0f,1.0f);
}
```

&emsp;&emsp;现在，进行绘制，同时使用 `XMMatrixRotation` 方法使其旋转：

```c++
		cubeWorld = cubeWorld * XMMatrixRotationX(0.0001f) * XMMatrixRotationY(0.0001f) * XMMatrixRotationZ(0.0001f);
		cb.world = XMMatrixTranspose(cubeWorld);
		pImmediateContext->UpdateSubresource(pConstantBuffer, 0, nullptr, &cb, 0, 0);
		pImmediateContext->VSSetConstantBuffers(0, 1, &pConstantBuffer);
		pImmediateContext->PSSetShaderResources(0, 1, &pShaderResourceView);
		pImmediateContext->PSSetSamplers(0, 1, &pSamplerState);
		pImmediateContext->IASetInputLayout(pCubeInputLayout);
		pImmediateContext->VSSetShader(pCubeVertexShader, nullptr, 0);
		pImmediateContext->PSSetShader(pCubePixelShader, nullptr, 0);
		pImmediateContext->DrawIndexed(36, 0, 0);

		cb.world = XMMatrixTranspose(lightWorld * XMMatrixTranslation(0.0f, 3.0f, 0.0f));
		pImmediateContext->UpdateSubresource(pConstantBuffer, 0, nullptr, &cb, 0, 0);
		pImmediateContext->VSSetConstantBuffers(0, 1, &pConstantBuffer);
		pImmediateContext->IASetInputLayout(pLightInputLayout);
		pImmediateContext->VSSetShader(pLightVertexShader, nullptr, 0);
		pImmediateContext->PSSetShader(pLightPixelShader, nullptr, 0);
		pImmediateContext->DrawIndexed(36, 0, 0);

```

&emsp;&emsp;现在你应该看到的是一个旋转的立方体和一个在立方体上方的光源，如下：

![1](https://image.ibb.co/kF2Xc7/image.png)

&emsp;&emsp;接下来就是 HLSL 中的代码了。首先修改顶点着色器的数据结构：

```c++
struct vertexInputType {
	float4 pos : POSITION;
	float2 texcoord : TEXCOORD0;
	float3 normal : NORMAL;
};

struct pixelInputType {
	float4 pos : SV_POSITION;
	float4 posw : POSITION1;	// 使用这个坐标来计算光方向向量
	float2 texcoord : TEXCOORD0;
	float3 normal : NORMAL;
};
```

&emsp;&emsp;同时在对顶点着色器中变换坐标的同时也变换法线坐标：

```c++
pixelInputType main( vertexInputType input ){
	pixelInputType output;
	output.pos = input.pos;
	output.pos = mul(output.pos , world);
	output.pos = mul(output.pos , view);
	output.pos = mul(output.pos , projection);
	output.texcoord = input.texcoord;
	output.normal = mul(input.normal , (float3x3)world); // normal
	return output;
}
```

&emsp;&emsp;有了法线之后，我们可以在像素着色器中根据法线与光照方向的夹角 cos 值来计算颜色了。首先我们使用硬编码写好光源位置和环境光大小：

```c++
float ambient = 0.1f;
float3 lightPos = float3(0.0f, 3.0f, 0.0f);
```

&emsp;&emsp;之后计算光照方向：

```c++
float3 lightDir = - lightPos;
```

&emsp;&emsp;最后，使用光照方向的反向量来和法线向量点乘获得 cos 值：

```c++
float diffuse = saturate(dot(normalize(input.normal), normalize(-lightDir))); // staturate 将其值限制在 0  1 区间内
```

&emsp;&emsp;配合纹理颜色和环境光颜色来计算最终颜色：

```c++
return tex.Sample(samp , input.texcoord) * (ambient + diffuse);
```

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/bxRdjn/image.png)

&emsp;&emsp;源代码地址：[DX11Tutorial DiffuseLighting](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-DiffuseLighting)

