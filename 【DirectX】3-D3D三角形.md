---
title: 【DirectX】3-D3D三角形
date: 2018-03-18 13:28:14
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;上一篇文章中我们介绍了 **交换链**，**设备**，**设备上下文** 等等名词，并成功创建了他们，这篇中我们将和学习 OpenGL 时一样去使用三个顶点来绘制一个三角形。

&emsp;&emsp;这篇中将要介绍的则是 `ID3D11Buffer（缓冲）` ，`ID3D11VertexShader(顶点着色器)` 和 `ID3D11PixelShader(像素着色器)` 这几个东西，我们将顶点数组构图，使用着色器填色。（事实上和我们的 OpenGL 基本一样）

<!--more-->

&emsp;&emsp;在 OpenGL 中我们使用 glm 数学库（[OpenGL Mathematics](https://glm.g-truc.net/)），而在 DirectX 中则是使用 XM 库（[DirectXMath]()）。首先定义一个三角形的顶点：

```c++
struct Vertex {				// 定义一个顶点结构体，暂时就一个坐标属性
	DirectX::XMFLOAT3 pos;
};

Vertex vertices[] =			// 顶点数组
{
    DirectX::XMFLOAT3(0.0f, 0.3f, 0.3f),
    DirectX::XMFLOAT3(0.3f, -0.3f, 0.3f),
    DirectX::XMFLOAT3(-0.3f, -0.3f, 0.3f),
};
```

&emsp;&emsp;现在使用 `D3D11_BUFFER_DESC` 创建定点缓冲描述，

```c++
// 缓存信息描述
D3D11_BUFFER_DESC vertexBufferDesc;
ZeroMemory(&vertexBufferDesc, sizeof(vertexBufferDesc));
vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;		// 默认使用
vertexBufferDesc.ByteWidth = sizeof(Vertex) * 3;	// 大小（我们有三个顶点）
vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;	// Bind
```

&emsp;&emsp;创建 `D3D11_SUBRESOURCE_DATA` ：

```c++
D3D11_SUBRESOURCE_DATA verticesSourceData;
ZeroMemory(&verticesSourceData, sizeof(D3D11_SUBRESOURCE_DATA));
verticesSourceData.pSysMem = vertices;
```

&emsp;&emsp;最后定义 `ID3D11Buffer *pVertexBufferObject = nullptr;` 将他初始化，接下来使用我们的设备对象来创建一个缓冲：

```c++
pDevice->CreateBuffer(&vertexBufferDesc, &verticesSourceData, &pVertexBufferObject);
```

&emsp;&emsp;这几段代码就是为了创建缓冲而来。

&emsp;&emsp;在 OpenGL 中，我们现在应该去使用一个 VertexArraysObject（VAO）来描述这个缓冲，但是在 DirectX 中右上下文设备的存在，我们直接使用上下文：

```c++
UINT stride = sizeof(Vertex);
UINT offset = 0;
pImmediateContext->IASetVertexBuffers(0, 1, &pVertexBufferObject, &stride, &offset);
pImmediateContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
```

&emsp;&emsp;我们使用立即上下文对象 `pImmediateContext` 来接手这一段顶点数据缓冲。现在你如果直接在主循环里绘制的话，会得到一个消息：`A Vertex Shader is always required when drawing, but none is currently bound.`

因此我们在绘制的时候必须为其配置着色器程序。

&emsp;&emsp;与 DirectX 配套的着色器程序语言叫做 [HLSL（High Level  Shading Language）](https://msdn.microsoft.com/en-us/library/windows/desktop/bb509561%28v=vs.85%29.aspx)，它也是使用文本文件来写，然后在 DirectX 代码中编译。首先我们来放两个最简单的着色器代码，顶点着色器和像素着色器（对应 OpenGL 中的片段着色器）：

```c++
//triangle.vs 顶点着色器
float4 Main( float4 Pos : POSITION ) : SV_POSITION
{
    return Pos;
}
//triangle.ps 像素着色器
float4 Main( float4 Pos : SV_POSITION ) : SV_TARGET
{
    return float4( 1.0f, 1.0f, 0.0f, 1.0f );    
}
```

&emsp;&emsp;相比而言，HLSL 的代码看起来更简单一点。我们直接定义了 Main 方法（**需要注意的是 hlsl 的入口方法并非固定，当你启动编译的时候需要指定入口方法，也就是说是由你决定命名的**）。在这里需要看一下 `SV_POSITION` 和 `POSITION` 以及 `SV_TARGET` 这三个变量。`SV_POSITION ` 和 `POSITION` 无疑都是用来描述位置坐标的标识，而区别是 `SV_POSITION` 用来代表经过顶点着色器变换后的坐标。SV是Systems Value的简写，在`SV_POSITION`的情况下，如果它是绑定在一个从顶点着色器输出的数据结构上的话，意味着这个输出的数据结构包含了最终的转换过的，将用于光栅器的顶点坐标。

&emsp;&emsp;也就是说，顶点着色器接收未经处理的原始坐标，在顶点着色器中处理后送给像素着色器（虽然这个例子上我们并未处理）。然后 `SV_TARGET` 标识则是这个函数返回一个传递给下一阶段的值（也就是最终的OutPut Merger 的颜色值）。

&emsp;&emsp;如果需要了解更多的 HLSL 知识，可以去 MSDN 看一下（MSDN 是所有 Windows 程序员要翻烂的地方）。

&emsp;&emsp;接下来，我们对这两个着色器进行编译：

```c++
ID3D10Blob* pErrorMessage = nullptr;
ID3D10Blob* pVertexShaderBlob = nullptr;
ID3D10Blob* pPixelShaderBlob = nullptr;
```

&emsp;&emsp;首先定义顶点着色器 Blob 对象和像素着色器 Blob 对象以及用来查错的对象。之后调用 `D3DX11CompileFromFile` 来进行编译，先来看看这个方法的参数：

```c++
HRESULT D3DX11CompileFromFile(
  _In_        LPCTSTR            pSrcFile,	// 源文件
  _In_  const D3D10_SHADER_MACRO *pDefines,  // 可选项 为 NULL 
  _In_        LPD3D10INCLUDE     pInclude,  // NULL
  _In_        LPCSTR             pFunctionName, // 入口函数名称
  _In_        LPCSTR             pProfile,  // 配置信息 （ps_x_x 像素 ,vs_x_x 顶点）
  _In_        UINT               Flags1,  // 参数配置
  _In_        UINT               Flags2,  // 参数配置
  _In_        ID3DX11ThreadPump  *pPump,  // NULL 
  _Out_       ID3D10Blob         **ppShader,  // shader 对象
  _Out_       ID3D10Blob         **ppErrorMsgs, // 错误信息对象
  _Out_       HRESULT            *pHResult  //NULL
);
```

&emsp;&emsp;具体使用如下：

```c++
// 编译顶点着色器
hr = D3DX11CompileFromFile("./triangle.vs", nullptr, nullptr, "Main", "vs_5_0", D3DCOMPILER_STRIP_DEBUG_INFO , 0, nullptr, &pVertexShaderBlob, &pErrorMessage, nullptr);
if (FAILED(hr)) {
	if (pErrorMessage) MessageBox(NULL, static_cast<CHAR*>(pErrorMessage->GetBufferPointer()), "Error", MB_OK);
	else MessageBox(NULL, "Triangle.vs File Not Found", "Error", MB_OK);
	return hr;
}
// 编译像素着色器
hr = D3DX11CompileFromFile("./triangle.ps", nullptr, nullptr, "Main", "ps_5_0", D3DCOMPILER_STRIP_DEBUG_INFO , 0, nullptr, &pPixelShaderBlob, &pErrorMessage, nullptr);
if (FAILED(hr)) {
	if (pErrorMessage) MessageBox(NULL, static_cast<CHAR*>(pErrorMessage->GetBufferPointer()), "Error", MB_OK);
	else MessageBox(NULL, "Triangle.vs File Not Found", "Error", MB_OK);
	return hr;
}
```

&emsp;&emsp;在编译成功后，我们使用设备对象来创建顶点着色器和像素着色器：

```c++
hr = pDevice->CreateVertexShader(pVertexShaderBlob->GetBufferPointer(), pVertexShaderBlob->GetBufferSize(), nullptr, &pVertexShader);
if (FAILED(hr)) {
	MessageBox(NULL, "ERROR::CreateVertexShader", "Error", MB_OK);
	return hr;
}
hr = pDevice->CreatePixelShader(pPixelShaderBlob->GetBufferPointer(), pPixelShaderBlob->GetBufferSize(), nullptr, &pPixelShader);
if (FAILED(hr)) {
	MessageBox(NULL, "ERROR::CreateVertexShader", "Error", MB_OK);
	return hr;
}	
```

&emsp;&emsp;在顶点着色器中，我们使用了 `POSITION` ，在 DirectX 代码中创建一个 `InputLayout` 来描述 ` input-assembler` 阶段的数据。

```c++
D3D11_INPUT_ELEMENT_DESC layout[] = { { "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0 } };
const UINT numElements = ARRAYSIZE( layout );
hr = pDevice->CreateInputLayout(layout, numElements, pVertexShaderBlob->GetBufferPointer(), pVertexShaderBlob->GetBufferSize(), &pInputLayout);
if (FAILED(hr)) {
	MessageBox(NULL, "ERROR::CreateInputLayout", "Error", MB_OK);
	return hr;
}
pImmediateContext->IASetInputLayout(pInputLayout);	// 设置
```

&emsp;&emsp;DX 中多数 API 都有着及其繁杂的参数，限于篇幅在这里写者并没有全部列出来，最详细的信息还是首推 MSDN （话说我推荐过多少次了）。

&emsp;&emsp;这些都结束后，我们在主循环里绘制它：

```c++
MSG msg;
ZeroMemory(&msg, sizeof(MSG));
while (msg.message != WM_QUIT) {
	float color[] = { 0.1f , 0.2 , 0.5f , 1.0f };
	pImmediateContext->ClearRenderTargetView(pRenderTargetView, color);
	pImmediateContext->VSSetShader(pVertexShader, nullptr, 0);	//set vs
	pImmediateContext->PSSetShader(pPixelShader, nullptr, 0);	//set ps
	pImmediateContext->Draw(3, 0);		//draw
	pSwapChain->Present(0, 0);
	if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE)) {
		TranslateMessage(&msg);
		DispatchMessage(&msg);
	}
}
```

&emsp;&emsp;如果代码没有错的话应该可以看到这样一个效果：

![Triangle](https://image.ibb.co/mv9wqH/image.png)