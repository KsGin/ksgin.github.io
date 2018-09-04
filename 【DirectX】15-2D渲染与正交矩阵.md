---
title: 【DirectX】15-2D渲染与正交矩阵
date: 2018-03-31 11:51:37
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 11: 2D Rendering](http://www.rastertek.com/dx11tut11.html)

#### 学习记录

&emsp;&emsp;这篇文章中主要介绍在 DX 中对于 2D 的渲染，这篇内容其实不多，也没啥营养。本来不想写的，但是因为之后要进行字体渲染什么的，干脆在之前把这个先写了吧。

<!--more-->

&emsp;&emsp;在 2D 坐标系中，我们只有 x,y 概念而没有 z 轴，所以在此之前我们需要关闭 z 轴，即禁用深度缓冲。禁用之后我们的绘制将会以画家算法为准，先绘制的物体会被后绘制的遮挡。首先使用深度模板缓冲，将是否启用设置为 `false `。

```c++
D3D11_DEPTH_STENCIL_DESC ddDisableStencilopDesc;
ZeroMemory(&ddDisableStencilopDesc, sizeof(ddDisableStencilopDesc));
ddDisableStencilopDesc.DepthEnable = false;	//////////////////////////////// 2D中禁用深度 //////////////////////////////////////////////////////////////////////
ddDisableStencilopDesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK_ALL;
ddDisableStencilopDesc.DepthFunc = D3D11_COMPARISON_LESS;
ddDisableStencilopDesc.StencilEnable = true;
ddDisableStencilopDesc.StencilReadMask = 0xFF;
ddDisableStencilopDesc.StencilWriteMask = 0xFF;
ddDisableStencilopDesc.FrontFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
ddDisableStencilopDesc.FrontFace.StencilDepthFailOp = D3D11_STENCIL_OP_INCR;
ddDisableStencilopDesc.FrontFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
ddDisableStencilopDesc.FrontFace.StencilFunc = D3D11_COMPARISON_ALWAYS;
ddDisableStencilopDesc.BackFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
ddDisableStencilopDesc.BackFace.StencilDepthFailOp = D3D11_STENCIL_OP_DECR;
ddDisableStencilopDesc.BackFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
ddDisableStencilopDesc.BackFace.StencilFunc = D3D11_COMPARISON_ALWAYS;

ID3D11DepthStencilState *pDisableDepthStencilState = nullptr;
hr = pDevice->CreateDepthStencilState(&ddDisableStencilopDesc, &pDisableDepthStencilState);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateDepthStencilState", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;这个和 3D 中的是一样的，如果我们需要 3D 的缓冲可以直接将 `false` 改为 `true` 。接着在立即上下文对象中设置它：

```c++
pImmediateContext->OMSetDepthStencilState(pDisableDepthStencilState, 1);
```

&emsp;&emsp;现在来创建深度模板测试视图并启用：

```c++
D3D11_DEPTH_STENCIL_VIEW_DESC depthStencilViewDesc;
ZeroMemory(&depthStencilViewDesc, sizeof(depthStencilViewDesc));
depthStencilViewDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
depthStencilViewDesc.ViewDimension = D3D11_DSV_DIMENSION_TEXTURE2D;
depthStencilViewDesc.Texture2D.MipSlice = 0;
ID3D11DepthStencilView *pDepthStencilView = nullptr;
hr = pDevice->CreateDepthStencilView(pDepthBuffer, &depthStencilViewDesc, &pDepthStencilView);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateDepthStencilView", "ERROR", MB_OK);
	return hr;
}
pImmediateContext->OMSetRenderTargets(1, &pRenderTargetView, pDepthStencilView);
```

&emsp;&emsp;除了对 Z 的禁用之外，我们需要使用正交投影矩阵来代替 3D 的投影矩阵，创建出新的坐标系统矩阵：

```c++
MatrixXD matrix2d = {
	XMMatrixIdentity() ,
	XMMatrixLookAtLH(
		XMVectorSet(0.0f, 0.0f , -100.0f , 0.0f),
		XMVectorSet(0.0f, 0.0f ,  0.0f , 0.0f),
		XMVectorSet(0.0f, 1.0f ,  0.0f , 0.0f)
	),
	XMMatrixOrthographicLH(static_cast<float>(width) , static_cast<float>(height) , 0.01f , 100.0f)
};
```

&emsp;&emsp;在使用了正交投影后，我们对我们的举行坐标进行修改，现在他们的范围与屏幕大小有关，且原点在屏幕中央。现在我们将矩形设置为中心在原点边长为200的一个正方形：

```c++
Vertex vertices[] =
{
	{ DirectX::XMFLOAT3(-100.0f, -100.0f, 0.0f) , DirectX::XMFLOAT2(0.0f,1.0f) },
	{ DirectX::XMFLOAT3(-100.0f, 100.0f, 0.0f) , DirectX::XMFLOAT2(0.0f,0.0f) },
	{ DirectX::XMFLOAT3(100.0f, 100.0f, 0.0f) , DirectX::XMFLOAT2(1.0f,0.0f) },
	{ DirectX::XMFLOAT3(-100.0f, -100.0f, 0.0f) , DirectX::XMFLOAT2(0.0f,1.0f) },
	{ DirectX::XMFLOAT3(100.0f, 100.0f, 0.0f) , DirectX::XMFLOAT2(1.0f,0.0f) },
	{ DirectX::XMFLOAT3(100.0f, -100.0f, 0.0f) , DirectX::XMFLOAT2(1.0f,1.0f) }
};
```

&emsp;&emsp;其他基本一致，现在编译运行程序我们应该可以看到一个 200 * 200 的图片：

![1](https://image.ibb.co/mTxjx7/image.png)

&emsp;&emsp;源代码：[DX11Tutorial：2D Render](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-2DRender)