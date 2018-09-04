---
title: 【DirectX】27-雾效果
date: 2018-04-10 19:31:10
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 23: Fog](http://www.rastertek.com/dx11tut23.html)

#### 学习记录

&emsp;&emsp;在这篇开始，我们首先需要纠正上一篇部分的错误之处，在上一篇我们渲染到纹理的时候，渲染了一张 200 x 200 的纹理，然后使用的还是我们之前渲染 1920 x 1080 后缓冲用的深度缓冲，所以会出现错误，错误信息如下：

> &emsp;&emsp;D3D11 ERROR: ID3D11DeviceContext::OMSetRenderTargets: The RenderTargetView at slot 0 is not compatible with the DepthStencilView. DepthStencilViews may only be used with RenderTargetViews if the effective dimensions of the Views are equal, as well as the Resource types, multisample count, and multisample quality. The RenderTargetView at slot 0 has (w:200,h:200,as:1), while the Resource is a Texture2D with (mc:1,mq:0). The DepthStencilView has (w:1920,h:1080,as:1), while the Resource is a Texture2D with (mc:1,mq:0).  [ STATE_SETTING ERROR #388: OMSETRENDERTARGETS_INVALIDVIEW]

&emsp;&emsp;如果我们确实需要渲染不同的视图，那么需要创建不同的深度模板使用，如上。

&emsp;&emsp;这一篇中，我们主要学习在 D3D11 中实现雾效果。

<!--more-->

&emsp;&emsp;首先我们需要模拟雾天气，所以我们修改渲染的背景为淡灰色的雾状，如下：

![1](https://image.ibb.co/kZQzGc/image.png)

&emsp;&emsp;只需要简单的修改清空渲染时候的背景颜色就可以实现了。之后我们为了表现在近处看的清晰远处模糊的情况，我们先渲染出多个立方体。

```c++
vector<XMMATRIX> worldMatrixList(5);
vector<XMFLOAT3> rotateDir(5);
for (int i = 0; i < 5; ++i) {
	worldMatrixList[i] = XMMatrixIdentity() * XMMatrixScaling(0.2f, 0.2f, 0.2f) * XMMatrixTranslation((rand() - rand()) % 2, (rand() - rand()) % 2, (rand() - rand()) % 2);
	rotateDir[i] = XMFLOAT3((rand() - rand()) % 2 * 0.001f, (rand() - rand()) % 2 * 0.001f, (rand() - rand()) % 2 * 0.001f);
}
.......

for (int i = 0; i < 5; ++i) {
	worldMatrixList[i] *= XMMatrixRotationX(rotateDir[i].x) * XMMatrixRotationY(rotateDir[i].y) * XMMatrixRotationZ(rotateDir[i].z);
	matrix3d.world = worldMatrixList[i];
	Update3DModelWorld(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
	DrawModelIndex(
		cubeVertexNum,
		pCubeVertexBufferObject,
		pCubeIndexBufferObject,
		pMatrixDBuffer3D,
		pCubeSamplerState,
		nNumCubeTex,
		pCubeShaderResourceView,
		pCubeVertexShader,
		pCubePixelShader,
		pCubeInputLayout,
		pDepthStencilView,
		pEnableDepthStencilState,
		&pRenderTargetViewT,
		&pD3DImmediateContext);
}  
```

&emsp;&emsp;我们渲染了5个不同位置的立方体，并且他们会随机绕着某一个方向运动，某个状态可能如下：

![2](https://image.ibb.co/degWwc/image.png)

&emsp;&emsp;现在，可以开始实现我们的雾效果了。在全局雾的情况下，我们会清晰的看到距离我们很近的物体，然后随着距离变远，物体会渐渐被雾掩盖，直到看不见。

&emsp;&emsp;首先我们要定义我们从完全看见到完全看不见的范围，写者直接在顶点着色器中进行了定义：

```c++
float fogStart = 1.0f;
float fogEnd = 4.0f;
```

&emsp;&emsp;当距离在 `0 ~ 1`  的范围内我们完全可以看见（小于 0 在摄像机身后），而超过 4 则是完全看不见了。定义了这个之后我们需要物体的坐标和摄像机的坐标：

```c++
float4 modelPosition;
float4 cameraPosition;

modelPosition = mul(input.pos, world);
modelPosition = mul(modelPosition, view);

cameraPosition = float4(1.0f, 1.0f, -1.0f, 0.0f);
```

&emsp;&emsp;摄像机坐标直接写好（与我们实际上的摄像机坐标一致，实际上最好用 `buffer` 传进来），物体坐标则使用 wv 矩阵来计算。有了这个之后，我们可以计算摄像机到物体这一点的距离：

```c++
float distToEye;
distToEye = length(cameraPosition - modelPosition);
```

&emsp;&emsp;现在，我们可以在传入像素着色器的结构体中添加一个元素：

```c++
struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
	float fogFactor : FOG;
};
```

&emsp;&emsp;并且计算他的值：

```c++
output.fogFactor = saturate((distToEye - fogStart) / (fogEnd - fogStart));
```

&emsp;&emsp;这个值我们可以称之为雾权重，这个值越大说明物体距离摄像机越远，那么它的颜色越贴近于雾的颜色（即看起来越模糊）。需要注意的是，这应该是一个在 `0 ~ 1` 之间的数字，我们要用它来插值。所以使用 `saturate` 来限定范围。

&emsp;&emsp;在像素着色器中，则是简单的混合雾颜色和纹理颜色：

```c++
float4 main(pixelInputType input) : SV_TARGET {
	float4 fogColor = float4(0.5f, 0.5f, 0.5f, 1.0f);
	float4 texColor = tex[0].Sample(samp, input.texcoord);
	
	return fogColor * input.fogFactor + texColor * (1 - input.fogFactor);
} 
```

&emsp;&emsp;最终的效果应该如下：

![4](https://image.ibb.co/myJ0px/image.png)

&emsp;&emsp;可以看到，我们的五个 Cube ，有一个已经完全被雾掩盖了，剩下的也都有了或多或少的模糊。距离我们越远的越模糊。

&emsp;&emsp;源代码：[DX11Tutorial-FogEffect](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-FogEffect)