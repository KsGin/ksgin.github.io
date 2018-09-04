---
title: 【DirectX】11-3DRender
date: 2018-03-26 13:07:40
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 7: 3D Model Rendering](http://www.rastertek.com/dx11tut07.html)

#### 学习记录

&emsp;&emsp;渲染 3D 图形的话和 2D 视角不同，我们必须使用 3D 摄像机视角，如果对摄像机以及矩阵作用不太懂的话可以看这里：[【OpenGL】10-摄像机](https://blog.ksgin.com/2018/02/20/%E3%80%90opengl%E3%80%9110-%E6%91%84%E5%83%8F%E6%9C%BA/) 。

<!--more-->

&emsp;&emsp;首先我们先渲染出来一个三角形，修改顶点以及使用索引绘制：

```c++
struct Vertex
{
	XMFLOAT3 pos;
	XMFLOAT4 color;
};
///////////// Vertex ////////////////////////////
Vertex vertices[] =
{
	{ XMFLOAT3(-0.5f, -0.5f, -0.5f), XMFLOAT4(0, 0, 1, 1) },
	{ XMFLOAT3(-0.5f, +0.5f, -0.5f), XMFLOAT4(0, 0, 0, 1) },
	{ XMFLOAT3(+0.5f, +0.5f, -0.5f), XMFLOAT4(1, 0, 0, 1) },
	{ XMFLOAT3(+0.5f, -0.5f, -0.5f), XMFLOAT4(0, 1, 0, 1) },
	{ XMFLOAT3(-0.5f, -0.5f, +0.5f), XMFLOAT4(0, 0, 1, 1) },
	{ XMFLOAT3(-0.5f, +0.5f, +0.5f), XMFLOAT4(1, 1, 0, 1) },
	{ XMFLOAT3(+0.5f, +0.5f, +0.5f), XMFLOAT4(0, 1, 1, 1) },
	{ XMFLOAT3(+0.5f, -0.5f, +0.5f), XMFLOAT4(1, 0, 1, 1) }
};

D3D11_BUFFER_DESC vertexBufferDesc;
ZeroMemory(&vertexBufferDesc, sizeof(vertexBufferDesc));
vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
vertexBufferDesc.ByteWidth = sizeof(Vertex) * 8;
vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;

D3D11_SUBRESOURCE_DATA verticesSourceData;
ZeroMemory(&verticesSourceData, sizeof(D3D11_SUBRESOURCE_DATA));

verticesSourceData.pSysMem = vertices;
ID3D11Buffer *pVertexBufferObject = nullptr;
pDevice->CreateBuffer(&vertexBufferDesc, &verticesSourceData, &pVertexBufferObject);

UINT stride = sizeof(Vertex);
UINT offset = 0;
pImmediateContext->IASetVertexBuffers(0, 1, &pVertexBufferObject, &stride, &offset);
pImmediateContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

///////////////////////// index /////////////////////////////
UINT indices[] = {
	0, 1, 2,
	0, 2, 3,
	4, 6, 5,
	4, 7, 6,
	4, 5, 1,
	4, 1, 0,
	3, 2, 6,
	3, 6, 7,
	1, 5, 6,
	1, 6, 2,
	4, 0, 3,
	4, 3, 7
};

D3D11_BUFFER_DESC indexDesc;
ZeroMemory(&indexDesc, sizeof(indexDesc));
indexDesc.Usage = D3D11_USAGE_IMMUTABLE;
indexDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
indexDesc.ByteWidth = sizeof(UINT)* 36;

D3D11_SUBRESOURCE_DATA indexData;
ZeroMemory(&indexData, sizeof(indexData));
indexData.pSysMem = indices;

ID3D11Buffer *pIndexBufferObject = nullptr;
hr = pDevice->CreateBuffer(&indexDesc, &indexData, &pIndexBufferObject);
if (FAILED(hr))
{
    return false;
}

pImmediateContext->IASetIndexBuffer(pIndexBufferObject, DXGI_FORMAT_R32_UINT , 0);
```

&emsp;&emsp;然后修改消息循环里的绘制方法为：`pImmediateContext->DrawIndexed(36, 0 , 0);` 。

&emsp;&emsp;现在创建我们的 `world` `view` `projection` 三个矩阵，首先定义结构体：

```c++
struct Constant {
	XMMATRIX world;
	XMMATRIX view;
	XMMATRIX projection;
};
```

&emsp;&emsp;我们使用 DX 中提供的几个方法来创建这个三个矩阵：

```c++
Constant cb = {
	XMMatrixTranspose(XMMatrixIdentity()) ,
	XMMatrixTranspose(XMMatrixLookAtLH(
		XMVectorSet(0.0f, 1.0f, -1.0f, 0.0f),
		XMVectorSet(0.0f, 0.0f, 0.0f, 0.0f),
		XMVectorSet(0.0f, 1.0f, 0.0f, 0.0f)
	)),
	XMMatrixTranspose(XMMatrixPerspectiveFovLH(90, static_cast<FLOAT>(width) / height, 0.1f, 100.0f))
};
```

&emsp;&emsp;这里我直接将这些放在了一块创建了结构体，分开写是这样：

```c++
Constant cb;

auto world = DirectX::XMMatrixIdentity();
auto View = DirectX::XMMatrixLookAtLH(
		DirectX::XMVectorSet(0.0f, 1.0f, -1.0f, 0.0f),	// Eye
		DirectX::XMVectorSet(0.0f, 0.0f, 0.0f, 0.0f),	// Focus
		DirectX::XMVectorSet(0.0f, 1.0f, 0.0f, 0.0f)	// Up
	);
auto projection = DirectX::XMMatrixPerspectiveFovLH(90, static_cast<FLOAT>(width) / height, 0.1f, 100.0f);

cb.world = world;
cb.view = view;
cb.projection = projection;

cb.world = DirectX::XMMatrixTranspose(cb.world);
cb.view = DirectX::XMMatrixTranspose(cb.view);
cb.projection = DirectX::XMMatrixTranspose(cb.projection);
```

&emsp;&emsp;**（是不是和 opengl 很像）**

&emsp;&emsp;之后，我们创建 Constant 的缓冲对象并且创建好缓冲：

```c++
D3D11_BUFFER_DESC constantBufferDesc;
ZeroMemory(&constantBufferDesc, sizeof(constantBufferDesc));
constantBufferDesc.Usage = D3D11_USAGE_DEFAULT;
constantBufferDesc.BindFlags = D3D10_BIND_CONSTANT_BUFFER;
constantBufferDesc.ByteWidth = sizeof(Constant);

D3D11_SUBRESOURCE_DATA constantSourceData;
ZeroMemory(&constantSourceData, sizeof(D3D11_SUBRESOURCE_DATA));
constantSourceData.pSysMem = &cb;

ID3D11Buffer *pConstantBuffer = nullptr;
hr = pDevice->CreateBuffer(&constantBufferDesc, &constantSourceData, &pConstantBuffer);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateConstantBuffer", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;设置设备上下文：

```c++
pImmediateContext->VSSetConstantBuffers(0, 1, &pConstantBuffer);
```

&emsp;&emsp;在 DX 代码里需要修改的内容差不多就这些了，接下来看看 HLSL 代码，在顶点着色器里我们需要对 POSITION 来乘这三个矩阵，首先定义一个全局变量：

```c++
cbuffer ConstantBuffer : register(b0) {
	matrix world;
	matrix view;
	matrix projection;
}
```

&emsp;&emsp;在 main 函数里相乘：

```c++
pixelInputType main( vertexInputType input )
{
	pixelInputType output;
	output.pos = input.pos;
	output.pos = mul(output.pos , world);
	output.pos = mul(output.pos , view);
	output.pos = mul(output.pos , projection);
	output.color = input.color;
	return output;
}
```

&emsp;&emsp;接下来运行程序，应该可以看到这个：

![1](https://image.ibb.co/fiTuN7/image.png)

&emsp;&emsp;源代码在这里：[DX11Tutorial-3DRender](https://github.com/KsGin/DX11Tutorial/blob/master/DX11Tutorial-3DRender/main.cpp)

