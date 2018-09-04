---
title: 【DirectX】20-数据更新
date: 2018-04-04 18:19:34
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;这篇文章中，我们将介绍一些比较简单的内容，主要是对于之前代码的一些修改。在之前我们的代码中，例如 [【directx】18-fps与cpu使用率](https://blog.ksgin.com/2018/04/02/【directx】18-fps与cpu使用率/) 这一篇中。由于帧数和 cpu 使用率的数值是不断变化的，所以我们需要实时的生成不一样的顶点数据，这样程序会在每一帧运行的时候去申请一段空间，然后送给 GPU 。虽然我们也自己处理了一部分的内存释放，例如在申请 vertex 数组前先判断数组是否存在，存在的话先去释放它。但是也还是有很大的改进空间，这篇文章主要就是对这个进行更改。

<!--more-->

&emsp;&emsp;在此之前，我们对代码结构进行了部分重构处理。虽然没有去和很早之前一样去设计多个类，但是也将我们的部分代码进行了封装。等会你可以在文章下边提供的源代码网址里看到这些。

&emsp;&emsp;然后，在此对于上边提到的改进，这次我们将不再重新去生成顶点缓冲，而是使用 DX 中提供的缓冲更新方法，我们首先初始化我们用来显示文字的顶点，使用的是 `InitSentenceVertexAndIndexBuffer` 方法：

```c++
inline HRESULT InitSentenceVertexAndIndexBuffer(
	const char * sentence,
	const float drawX,
	const float drawY,
	Font * fonts,
	ID3D11Device * const pDevice,
	ID3D11Buffer **pVertexBufferObject,
	ID3D11Buffer **pIndexBufferObject,
	UINT &nNumVertex,
	Vertex *&vertices,
	UINT *&indices) {

	if (vertices) {
		delete[] vertices;
		vertices = nullptr;
	}

	if (indices) {
		delete[] indices;
		indices = nullptr;
	}

	HRESULT hr;

	nNumVertex = strlen(sentence) * 6;
	vertices = new Vertex[nNumVertex];

	int idx = 0;
	float posX = drawX;
	const float posY = drawY;
	for (auto i = 0; i < nNumVertex / 6; ++i) {
		const Font letter = fonts[static_cast<int>(sentence[i]) - 32];
		if (sentence[i] - 32 == 0) {
			posX += 3;
		}
		else {
			vertices[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			vertices[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			vertices[idx++].tex = XMFLOAT2(letter.right, 1.0f);
			vertices[idx].pos = XMFLOAT3(posX, posY - 16, 0.0f);	// 左下
			vertices[idx++].tex = XMFLOAT2(letter.left, 1.0f);
			vertices[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			vertices[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY, 0.0f);	// 右上
			vertices[idx++].tex = XMFLOAT2(letter.right, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			vertices[idx++].tex = XMFLOAT2(letter.right, 1.0f);
			posX += letter.size + 1.0f;
		}
	}

	D3D11_BUFFER_DESC vertexBufferDesc;
	ZeroMemory(&vertexBufferDesc, sizeof(vertexBufferDesc));
	vertexBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
	vertexBufferDesc.ByteWidth = sizeof(Vertex) * nNumVertex;
	vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	vertexBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

	D3D11_SUBRESOURCE_DATA verticesSourceData;
	ZeroMemory(&verticesSourceData, sizeof(D3D11_SUBRESOURCE_DATA));
	verticesSourceData.pSysMem = vertices;

	hr = pDevice->CreateBuffer(&vertexBufferDesc, &verticesSourceData, pVertexBufferObject);
	if (FAILED(hr))
	{
		return hr;
	}

	indices = new UINT[nNumVertex];
	for (auto i = 0; i < nNumVertex; ++i) indices[i] = i;

	D3D11_BUFFER_DESC indexDesc;
	ZeroMemory(&indexDesc, sizeof(indexDesc));
	indexDesc.Usage = D3D11_USAGE_DYNAMIC;
	indexDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
	indexDesc.ByteWidth = sizeof(UINT) * nNumVertex;
    vertexBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;

	D3D11_SUBRESOURCE_DATA indexData;
	ZeroMemory(&indexData, sizeof(indexData));
	indexData.pSysMem = indices;

	hr = pDevice->CreateBuffer(&indexDesc, &indexData, pIndexBufferObject);
	if (FAILED(hr))
	{
		return hr;
	}
	return S_OK;
}
```

&emsp;&emsp;还可以看到的是我们在这个方法里没有去定义顶点缓冲和索引缓冲对象。那是因为我们将它转移到了主函数里，使用二级指针传入进来为其赋值，这样可以使得我们的函数尽量的 "干净" 一些，也可以在主函数里统一的声明和释放指针。

&emsp;&emsp;之前我们是频繁的使用这个方法来生成顶点数据，生成顶点缓冲，然后绑定到立即上下文设备对象中，现在我们设计了一个新方法：

```c++
inline HRESULT UpdateSentenceVertexAndIndexBuffer(
	const char * sentence,
	const float drawX,
	const float drawY,
	Font * fonts,
	ID3D11Buffer **pVertexBufferObject,
	ID3D11Buffer **pIndexBufferObject,
	ID3D11DeviceContext **pImmediateContext,
	UINT &nNumVertex,
	Vertex *&vertices,
	UINT *&indices) {
	if (vertices) {
		delete[] vertices;
		vertices = nullptr;
	}

	if (indices) {
		delete[] indices;
		indices = nullptr;
	}

	HRESULT hr;

	nNumVertex = strlen(sentence) * 6;
	vertices = new Vertex[nNumVertex];

	int idx = 0;
	float posX = drawX;
	const float posY = drawY;
	for (auto i = 0; i < nNumVertex / 6; ++i) {
		const Font letter = fonts[static_cast<int>(sentence[i]) - 32];
		if (sentence[i] - 32 == 0) {
			posX += 3;
		}
		else {
			vertices[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			vertices[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			vertices[idx++].tex = XMFLOAT2(letter.right, 1.0f);
			vertices[idx].pos = XMFLOAT3(posX, posY - 16, 0.0f);	// 左下
			vertices[idx++].tex = XMFLOAT2(letter.left, 1.0f);
			vertices[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			vertices[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY, 0.0f);	// 右上
			vertices[idx++].tex = XMFLOAT2(letter.right, 0.0f);
			vertices[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			vertices[idx++].tex = XMFLOAT2(letter.right, 1.0f);
			posX += letter.size + 1.0f;
		}
	}

	indices = new UINT[nNumVertex];
	for (auto i = 0; i < nNumVertex; ++i) indices[i] = i;

	D3D11_MAPPED_SUBRESOURCE dms;
	hr = (*pImmediateContext)->Map(*pVertexBufferObject, 0, D3D11_MAP_WRITE_DISCARD, 0, &dms);
	if (FAILED(hr)) {
		return hr;
	}
	auto *vertexPtr = static_cast<Vertex*>(dms.pData);
	memcpy(vertexPtr, vertices, sizeof(Vertex) * nNumVertex);
	(*pImmediateContext)->Unmap(*pVertexBufferObject, 0);
	hr = (*pImmediateContext)->Map(*pIndexBufferObject, 0, D3D11_MAP_WRITE_DISCARD, 0, &dms);
	if (FAILED(hr)) {
		return hr;
	}
	auto *indexPtr = static_cast<UINT*>(dms.pData);
	memcpy(indexPtr, indices, sizeof(UINT) * nNumVertex);
	(*pImmediateContext)->Unmap(*pIndexBufferObject, 0);

	return S_OK;
}
```

&emsp;&emsp;事实上，这两个方法虽然一个为初始化一个为更新，但是内部含有大量的重复代码，这仍然是我们可以优化的一部分，在此我们暂且不对它进行修改，因为这个并不是重点。

&emsp;&emsp;可以看到，下边的方法中我们没有去使用顶点创建缓冲，而是使用上下文对象的方法来将缓冲中数据映射到一个指针上，然后通过拷贝将数据拷贝到缓冲里。通过这样我们可以去实时的修改顶点数据。同时还需要注意的地方是，如果我们需要更新数据，那么在初始化顶点缓冲的时候就需要一定的配置更改了，请注意一下我们上边初始化缓冲的代码：

```c++
vertexBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
vertexBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
```

&emsp;&emsp;我们修改了顶点缓冲描述的 `Usage` 和 `CPUAccessFlags` 两个值。

&emsp;&emsp;同时正如上边所说，我们将几乎所有的指针对象的声明和释放都放到了主函数里，这样可以避免很多的函数中内存泄漏。如下：

```c++
//////////////////////////////////////// declare //////////////////////////////////////
ID3D11RenderTargetView			*pD3DRenderTargetView = nullptr;
ID3D11Device					*pD3DDevice = nullptr;
ID3D11DeviceContext				*pD3DImmediateContext = nullptr;
IDXGISwapChain					*pD3DSwapChain = nullptr;
ID3D11DepthStencilView			*pDepthStencilView = nullptr;
ID3D11DepthStencilState			*pDisableDepthStencilState = nullptr;
ID3D11DepthStencilState			*pEnableDepthStencilState = nullptr;

const UINT cubeVertexNum = 36;
ID3D11SamplerState				*pCubeSamplerState = nullptr;
ID3D11ShaderResourceView		*pCubeShaderResourceView = nullptr;
ID3D11Buffer					*pCubeVertexBufferObject = nullptr;
ID3D11Buffer					*pCubeIndexBufferObject = nullptr;
ID3D11Buffer					*pMatrixDBuffer3D = nullptr;
ID3D11Buffer					*pMatrixDBuffer2D = nullptr;
Vertex							*cubeVertice = nullptr;
UINT							*cubeIndices = nullptr;
ID3D11VertexShader				*pCubeVertexShader = nullptr;
ID3D11PixelShader				*pCubePixelShader = nullptr;
ID3D11InputLayout				*pCubeInputLayout = nullptr;

UINT sentenceVertexNum = 0;
Font							*fonts = nullptr;
Vertex							*fontVertice = nullptr;
UINT							*fontIndices = nullptr;
ID3D11Buffer					*pFontVertexBufferObject = nullptr;
ID3D11Buffer					*pFontIndexBufferObject = nullptr;
ID3D11VertexShader				*pFontVertexShader = nullptr;
ID3D11PixelShader				*pFontPixelShader = nullptr;
ID3D11InputLayout				*pFontInputLayout = nullptr;
ID3D11SamplerState				*pFontSamplerState = nullptr;
ID3D11ShaderResourceView		*pFontShaderResourceView = nullptr;

//////////////////////////////////////// use //////////////////////////////////////////////

//////////////////////////////////////// release /////////////////////////////////////////
if (pD3DRenderTargetView) pD3DRenderTargetView->Release();
if (pD3DDevice) pD3DDevice->Release();
if (pD3DImmediateContext) pD3DImmediateContext->Release();
if (pD3DSwapChain) pD3DSwapChain->Release();
if (pDepthStencilView) pDepthStencilView->Release();
if (pDisableDepthStencilState) pDisableDepthStencilState->Release();
if (pEnableDepthStencilState) pEnableDepthStencilState->Release();
if (pCubeSamplerState) pCubeSamplerState->Release();
if (pCubeShaderResourceView) pCubeShaderResourceView->Release();
if (pCubeVertexBufferObject) pCubeVertexBufferObject->Release();
if (pCubeIndexBufferObject) pCubeIndexBufferObject->Release();
if (pMatrixDBuffer3D) pMatrixDBuffer3D->Release();
if (pMatrixDBuffer2D) pMatrixDBuffer2D->Release();
if (pCubeVertexShader) pCubeVertexShader->Release();
if (pCubePixelShader) pCubePixelShader->Release();
if (pCubeInputLayout) pCubeInputLayout->Release();
if (pFontVertexBufferObject) pFontVertexBufferObject->Release();
if (pFontIndexBufferObject) pFontIndexBufferObject->Release();
if (pFontVertexShader) pFontVertexShader->Release();
if (pFontPixelShader) pFontPixelShader->Release();
if (pFontInputLayout) pFontInputLayout->Release();
if (pFontSamplerState) pFontSamplerState->Release();
if (pFontShaderResourceView) pFontShaderResourceView->Release();
delete fonts;
delete[] fontVertice;
delete[] fontIndices;
delete[] cubeVertice;
delete[] cubeIndices;
PdhCloseQuery(hQuery);
```

&emsp;&emsp;如此，在程序运行起来后，我们的程序内存始终可以保持在一个可观的地步。下一篇中我们将去绘制及其多的 3d 模型并介绍对于 3d 场景下的视锥裁剪问题。这篇文章的代码也是和它在一起的。

&emsp;&emsp;源代码：[DX11Tutorial-FrustumCulling](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-FrustumCulling)

