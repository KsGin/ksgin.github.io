---
title: 【DirectX】26-渲染到纹理
date: 2018-04-09 21:18:48
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[**Tutorial 22: Render to Texture**](http://www.rastertek.com/dx11tut22.html)

#### 学习记录

&emsp;&emsp;在 OpenGL 中我们介绍过帧缓冲：[【OpenGL】17-帧缓冲与离屏渲染](https://blog.ksgin.com/2018/03/08/%E3%80%90opengl%E3%80%9117-%E5%B8%A7%E7%BC%93%E5%86%B2%E4%B8%8E%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93/) ，这篇文章中我们主要介绍在 DX11 中实现离屏渲染。

<!--more-->

&emsp;&emsp;这篇文章的内容比较简单，我们会首先渲染一张长宽为 200 的纹理，然后使用这张纹理来为我们的立方体贴图。

&emsp;&emsp;首先我们需要创建新的贴图，以及相应视图：

```c++
HRESULT InitTextureResourceView(
	const INT width , 
	const INT height ,
	ID3D11Device *pDevice,
	ID3D11Texture2D **pTexure2D , 
	ID3D11RenderTargetView **pRenderTargetView , 
	ID3D11ShaderResourceView ** pShaderResourceView) {
	
	HRESULT hr;
	D3D11_TEXTURE2D_DESC td;
	ZeroMemory(&td, sizeof(td));
	td.Width = width;
	td.Height = height;
	td.MipLevels = 1;
	td.ArraySize = 1;
	td.Format = DXGI_FORMAT_R32G32B32A32_FLOAT;
	td.SampleDesc.Count = 1;
	td.Usage = D3D11_USAGE_DEFAULT;
	td.BindFlags = D3D11_BIND_RENDER_TARGET | D3D11_BIND_SHADER_RESOURCE;

	hr = pDevice->CreateTexture2D(&td, nullptr, pTexure2D);
	if (FAILED(hr)) {
		return hr;
	}

	D3D11_RENDER_TARGET_VIEW_DESC rd;
	ZeroMemory(&rd, sizeof(rd));
	rd.Format = td.Format;
	rd.ViewDimension = D3D11_RTV_DIMENSION_TEXTURE2D;
	rd.Texture2D.MipSlice = 0;

	hr = pDevice->CreateRenderTargetView(*pTexure2D, &rd, pRenderTargetView);
	if (FAILED(hr)) {
		return hr;
	}

	D3D11_SHADER_RESOURCE_VIEW_DESC sd;
	ZeroMemory(&sd, sizeof(sd));
	sd.Format = td.Format;
	sd.ViewDimension = D3D11_SRV_DIMENSION_TEXTURE2D;
	sd.Texture2D.MipLevels = 1;

	hr = pDevice->CreateShaderResourceView(*pTexure2D, &sd, &pShaderResourceView[0]);
	if (FAILED(hr)) {
		return hr;
	}
	return S_OK;
}
```

&emsp;&emsp;之后，我们在 `main` 方法里创建不同的 `viewport` ,分别用于渲染纹理和渲染屏幕：

```c++
D3D11_VIEWPORT dv1;
dv1.Width = width;
dv1.Height = height;
dv1.TopLeftX = 0;
dv1.TopLeftY = 0;
dv1.MinDepth = 0;
dv1.MaxDepth = 1;

D3D11_VIEWPORT dv2;
dv2.Width = 200;
dv2.Height = 200;
dv2.TopLeftX = 0;
dv2.TopLeftY = 0;
dv2.MinDepth = 0;
dv2.MaxDepth = 1;
```

&emsp;&emsp;声明并初始化我们的纹理及其资源等（调用刚才的方法）：

```c++
ID3D11ShaderResourceView *pShaderResourceViewT[nNumCubeTex];
ID3D11RenderTargetView *pRenderTargetViewT = nullptr;
ID3D11Texture2D *pTexure2DT = nullptr;

hr = InitTextureResourceView(
	200,
	200,
	pD3DDevice,
	&pTexure2DT,
	&pRenderTargetViewT,
	pShaderResourceViewT);
if (FAILED(hr)) {
	return hr;
}
```

&emsp;&emsp;此时，我们为要渲染的纹理创建一个新的投影矩阵（因为直接用之前的有点丑。。。），如下：

```c++
const MatrixXD matrixTex = {
XMMatrixIdentity() * XMMatrixScaling(3.0f , 3.0f , 3.0f) ,
XMMatrixLookAtLH(
	XMVectorSet(0.0f, 0.0f , -4.0f , 0.0f),
	XMVectorSet(0.0f, 0.0f ,  0.0f , 0.0f),
	XMVectorSet(0.0f, 1.0f ,  0.0f , 0.0f)
),
XMMatrixPerspectiveFovLH(90 , 1.0f , snear , sfar)
};
```

&emsp;&emsp;如果你要渲染静态的二维目标，可以使用正交矩阵并且在帧循环前就可以绘制完毕，如果你的纹理是动态的，那么就要在每一帧里运行了。

&emsp;&emsp;我们在主循环里加入每一帧的清空以及对应渲染：

```c++
pD3DImmediateContext->ClearRenderTargetView(pD3DRenderTargetView, backgroundColor1); // black
pD3DImmediateContext->ClearRenderTargetView(pRenderTargetViewT, backgroundColor2);	// green
pD3DImmediateContext->ClearDepthStencilView(pDepthStencilView, D3D11_CLEAR_DEPTH, 1.0f, 0);

pD3DImmediateContext->RSSetViewports(1, &dv2);
Update3DModelWorld(matrixTex, pMatrixDBuffer3D, &pD3DImmediateContext);
DrawModelIndex(
	cubeVertexNum,
	pCubeVertexBufferObject,
	pCubeIndexBufferObject,
	pMatrixDBuffer3D,
	pCubeSamplerState,
	nNumCubeTex ,
	pCubeShaderResourceView,	// 使用 cube 渲染的视图
	pCubeVertexShader,
	pCubePixelShader,
	pCubeInputLayout,
	pDepthStencilView,
	pEnableDepthStencilState,
	&pRenderTargetViewT,		// 渲染到纹理的 target
	&pD3DImmediateContext);

pD3DImmediateContext->RSSetViewports(1, &dv1);
matrix3d.world *= XMMatrixRotationY(0.0001);
Update3DModelWorld(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
DrawModelIndex(
	cubeVertexNum,
	pCubeVertexBufferObject,
	pCubeIndexBufferObject,
	pMatrixDBuffer3D,
	pCubeSamplerState,
	nNumCubeTex ,
	pShaderResourceViewT,		// 使用我们选然后的纹理视图
	pCubeVertexShader,
	pCubePixelShader,
	pCubeInputLayout,
	pDepthStencilView,
	pEnableDepthStencilState,
	&pD3DRenderTargetView,		// 渲染到屏幕
	&pD3DImmediateContext);

UpdateSentenceVertexAndIndexBuffer(
	(fpsText + to_string(GetFPS(startTime, fps, tCount)) + cpuText + to_string(GetCPUUsage(canSample, cpuUsage, lastSampleTime, hQuery, hCounter))).c_str(),
	-width / 2 + offsetX, height / 2 - offsetY, fonts,
	&pFontVertexBufferObject, &pFontIndexBufferObject, &pD3DImmediateContext,
	sentenceVertexNum, fontVertice, fontIndices);

DrawModelIndex(
	sentenceVertexNum,
	pFontVertexBufferObject,
	pFontIndexBufferObject,
	pMatrixDBuffer2D,
	pFontSamplerState,
	nNumFontTex ,
	pFontShaderResourceView,
	pFontVertexShader,
	pFontPixelShader,
	pFontInputLayout,
	pDepthStencilView,
	pDisableDepthStencilState,
	&pD3DRenderTargetView,
	&pD3DImmediateContext);

pD3DSwapChain->Present(0, 0);
```

&emsp;&emsp;这一大段代码需要注意的地方也就是我注释出来的部分，可以看到我们首先为纹理渲染，然后用这张纹理去贴图我们的立方体，同时我们为两个渲染目标使用了不同的背景颜色来重置。

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/jGN1Rc/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-RenderToTexture](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-RenderToTexture)

