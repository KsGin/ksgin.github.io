---
title: 【DirectX】2-初识DirectX11
date: 2018-03-16 21:40:02
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;本篇文章主要介绍 DirectX 库以及 DirectX 的初始化，在此之前我们需要来配置 DirectX 系统环境。

&emsp;&emsp;DX 的配置还是很方便的，[这里是 DX11 SDK 的下载地址](https://www.microsoft.com/en-us/download/details.aspx?id=23549)，在微软 DC 下载后，直接一路 Next 就可以成功安装了 。之后打开 Visual Studio，建立新项目，在项目属性的 VC++ 目录的 Include 加入 DX 安装路径里的 Include 文件夹， Library 加入 Libs 文件夹（这个和 OpenGL 在 Visual Studio 下的配置基本相同，不多介绍）。

&emsp;&emsp;配置好了之后，我们先建立一个 Win32 窗口，这个我们在第一篇文章中有介绍。建立完毕后现在开始配置 DX11 （希望你不会被这一个配置吓得失去信心）。

<!--more-->

&emsp;&emsp;首先看一下 D3D 程序的基本结构：（如果你对 DX 一点基本概念都没有，可以看这里 [DirectX 维基百科](https://zh.wikipedia.org/wiki/DirectX)）

![DX11 INFO](https://image.ibb.co/eADpPc/DD38_AD30631_CF70_D51_AF845_E21_B5_AEF4.jpg)

&emsp;&emsp;我们今天要做的，就是初始化 D3D 程序这里，当显示了一个窗口后，下面继续创建一个Direct3D 11设备，这个设备用于绘制3D场景。首先必须创建三个对象：一个**设备**、一个**立即执行上下文（immediate context）**和一个**交换链（Swap Chain）**，立即执行上下文对象是Direct3D 11中新添加的。

> &emsp;&emsp;**设备对象**用于将内容绘制在一个缓冲中，设备还包含创建资源的方法。
>
> &emsp;&emsp;**交换链**即表示对缓冲的操作，这些缓冲就是设备绘制的和显示在屏幕上的内容。交换链包含两个或两个以上的缓冲，主要是前缓冲和后备缓冲，它们就是设备绘制形成的纹理，用于显示在屏幕上。**前缓冲（front                     buffer）**就是当前显示在屏幕上的内容，这个缓冲是只读的，无法修改。**后备缓冲（back buffer）**是设备将要绘制的渲染目标，一旦它完成了绘制操作，交换链就会通过交换前缓冲和后备缓冲，将后备缓冲的内容显示在屏幕上，此时后备缓冲就变成了前缓冲。

&emsp;&emsp;交换链在 DX 中有定义好的结构体 [DXGI_SWAP_CHAIN_DESC](https://msdn.microsoft.com/en-us/library/bb173075%28v=vs.85%29.aspx)

```c++
typedef struct DXGI_SWAP_CHAIN_DESC {
  DXGI_MODE_DESC   BufferDesc;		// 描述后台缓冲的结构体
  DXGI_SAMPLE_DESC SampleDesc;		// 描述多重采样的结构体
  DXGI_USAGE       BufferUsage;		// 对于交换链，为DXGI_USAGE_RENDER_TARGET_OUTPUT
  UINT             BufferCount;		// 后台缓冲数量
  HWND             OutputWindow;	// 将要渲染到窗口的句柄
  BOOL             Windowed;		// 以窗口（window）还是全屏（full-screen）模式运行
  DXGI_SWAP_EFFECT SwapEffect;		// 设为DXGI_SWAP_EFFECT_DISCARD，让显卡驱动程序选择最高效的显示模式
  UINT             Flags;			// 通常为 0
} DXGI_SWAP_CHAIN_DESC;
```

&emsp;&emsp;在 `DXGI_SWAP_CHAIN_DESC` 结构体中又嵌套了多个结构体，我们主要看一下第一个：[DXGI_MODE_DESC](https://msdn.microsoft.com/zh-cn/library/bb173064%28v=vs.85%29.aspx)

```c++
typedef struct DXGI_MODE_DESC {
  UINT                     Width;				// 宽度
  UINT                     Height;				// 高度
  DXGI_RATIONAL            RefreshRate;	 		// 刷新率 hz
  DXGI_FORMAT              Format;		 		// 显示格式 RGB RGBA
  DXGI_MODE_SCANLINE_ORDER ScanlineOrdering;	 // scanling mode
  DXGI_MODE_SCALING        Scaling;				// scanling mode
} DXGI_MODE_DESC;
```

&emsp;&emsp;现在，我们去配置一个交换链：

```c++
// 配置交换链
DXGI_SWAP_CHAIN_DESC ds;
ZeroMemory(&ds, sizeof(DXGI_SWAP_CHAIN_DESC));
ds.BufferCount = 1;
ds.BufferDesc.Width = width;
ds.BufferDesc.Height = height;
ds.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
ds.BufferDesc.RefreshRate.Numerator = 60;
ds.BufferDesc.RefreshRate.Denominator = 1;
ds.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
ds.OutputWindow = window;
ds.SampleDesc.Count = 1;
ds.SampleDesc.Quality = 0;
ds.Windowed = TRUE;
```

&emsp;&emsp;配置完成后，我们调用 `D3D11CreateDeviceAndSwapChain` 方法创建交换链：

```c++
IDXGISwapChain* pIdxgiSwapChain = nullptr;
ID3D11RenderTargetView* pD3D11RenderTargetView = nullptr;
ID3D11Device* pD3D11Device = nullptr;
ID3D11DeviceContext*    pImmediateContext = nullptr;
D3D_DRIVER_TYPE         gD3DDriverType = D3D_DRIVER_TYPE_NULL;
D3D_FEATURE_LEVEL       pD3DFeatureLevel = D3D_FEATURE_LEVEL_11_0;

D3D_DRIVER_TYPE driverTypes[] =
{
	D3D_DRIVER_TYPE_HARDWARE,
	D3D_DRIVER_TYPE_WARP,
	D3D_DRIVER_TYPE_REFERENCE,
};
const auto numDriverTypes = ARRAYSIZE(driverTypes);

D3D_FEATURE_LEVEL featureLevels[] =
{
	D3D_FEATURE_LEVEL_11_0,
	D3D_FEATURE_LEVEL_10_1,
	D3D_FEATURE_LEVEL_10_0,
};
const auto numFeatureLevels = ARRAYSIZE(featureLevels);

HRESULT hr = S_OK;
for (auto driverType : driverTypes) {
	hr = D3D11CreateDeviceAndSwapChain(
		nullptr,
		driverType,
		nullptr,
		createDeviceFlags,
		featureLevels,
		numFeatureLevels,
		D3D11_SDK_VERSION,
		&ds,
		&pIdxgiSwapChain,
		&pD3D11Device,
		&pD3DFeatureLevel,
		&pImmediateContext);
	if (SUCCEEDED(hr)) {
		break;
	}
}
if (FAILED(hr)) {
	return hr;
}
```

&emsp;&emsp;请注意在这里我们定义了 `IDXGISwapChain* pIdxgiSwapChain = nullptr;` （交换链），`ID3D11RenderTargetView* pD3D11RenderTargetView = nullptr;` （渲染目标视图） ， `ID3D11Device* pD3D11Device = nullptr;` （设备），`ID3D11DeviceContext*    pImmediateContext = nullptr; ` （设备上下文）等多个指针。这几个指针的全部信息请参考 MSDN （我也是看 MSDN 的）。

&emsp;&emsp;交换链创建成功后，我们创建一个**渲染目标视图（Render Target View）**：

> &emsp;&emsp;渲染目标视图是Direct3D 11中一种**资源视图（resource view）**。资源视图可以让一个资源绑定到图形管线的某个阶段。可以将资源视图看成C中的类型转换，C中的一块 **原始内存（raw memory）** 可以被转换为任意数据结构，我们可以将一块内存转换为整数数组，浮点数数组，结构，结构数组等。如果我们不知道原始内存的类型，那么它对我们来说用处不大。Direct3D11的资源视图工作原理类似，例如一张2D纹理就类似于一块原始内存，就是一种原始基础资源，有了这个原始资源，我们就可以创建不同的资源视图将这个纹理以不同的格式绑定到图形管线的不同阶段，而不同的格式可以是要绘制的渲染目标，接收深度信息的深度模板缓冲，或者也可以是一个纹理资源。C中的类型转换可以以不同方式使用一块内存，而在Direct3D11中是资源视图进行类似的操作。
>
> &emsp;&emsp;因为我们需要将交换链中的后备缓冲绑定为一个渲染目标，所以需要创建一个渲染目标视图，这样Direct3D11就可以在其上进行绘制了。我们首先调用 GetBuffer()  方法获取后备缓冲对象。我们可以使用一个 **D3D11_RENDERTARGETVIEW_DESC** 结构体表示要创建的渲染目标视图，这个结构体通常是 **CreateRenderTargetView** 方法的第二个参数。但是，在本教程中，默认的渲染目标视图就能满足需要，所以第二个参数为NULL表示使用默认的渲染目标视图。创建了渲染目标视图后，我们就可以调用 **OMSetRenderTargets()** 方法将它绑定到图形管线，这样管线的绘制输出被写到了后备缓冲中。

&emsp;&emsp;简单来说，就是创建一个缓冲，代码如下：

```c++
// 创建渲染目标视图
ID3D11Texture2D *pId3D11Texture2D = nullptr;
hr = pIdxgiSwapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), reinterpret_cast<LPVOID*>(&pId3D11Texture2D));
if (FAILED(hr)) {
	return hr;
}
hr = pD3D11Device->CreateRenderTargetView(pId3D11Texture2D, nullptr, &pD3D11RenderTargetView);
if (FAILED(hr)) {
	return hr;
}
pImmediateContext->OMSetRenderTargets(1, &pD3D11RenderTargetView, nullptr);
D3D11_VIEWPORT dv;
dv.Width = width;
dv.Height = height;
dv.MaxDepth = 1;
dv.MinDepth = 0;
dv.TopLeftX = 0;
dv.TopLeftY = 0;
pImmediateContext->RSSetViewports(1, &dv);
```

&emsp;&emsp;（这部分内容实在是没办法，Win 系列的 API 本来就丑，而且还多，即使理解了他的工作机制还是要死记硬背这些内容）

&emsp;&emsp;完成创建视图后，就可以开始渲染了，我们在主循环中使用立即执行上下文对象（pImmediateContext）来进行渲染。

```c++
while (uMsg.message != WM_QUIT) {
	float clearColor[4] = { 0.1f , 0.2f , 0.5f , 0.3f };	// 颜色数组 RGBA
	pImmediateContext->ClearRenderTargetView(pD3D11RenderTargetView, clearColor);
	pIdxgiSwapChain->Present(0, 0);
	if (PeekMessage(&uMsg, nullptr, 0, 0, PM_REMOVE)) {
		TranslateMessage(&uMsg);
		DispatchMessage(&uMsg);
	}		
}
```

&emsp;&emsp;我们定义了一个 Color （包含 RGBA）数组，然后用 `pImmediateContext` 的 `ClearRenderTargetView` 来进行颜色渲染。最终效果如下：

![Windows](https://image.ibb.co/jSMnSx/image.png)

&emsp;&emsp;如果你没有看到这个窗口，那么可以看这里：[GitHub](https://github.com/KsGin/LearnDirectX/blob/master/DirectX-Application/main.cpp)

