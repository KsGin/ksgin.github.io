---
title: 【DirectX】18-FPS与CPU使用率
date: 2018-04-02 21:21:35
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 15: FPS, CPU Usage, and Timers](http://www.rastertek.com/dx11tut15.html)

#### 学习记录

&emsp;&emsp;这篇文章内容不多，主要是趁着前边的 2D 文字渲染印象还好，就顺便学习了关于 3D渲染中 FPS 和 CPU 使用率的显示。

<!--more-->

&emsp;&emsp;FPS 的内容我们自己可以通过计数器计算，每一帧对计数器加1，当一秒后将计数器的值赋值给 FPS 的变量同时清零计数器的值以备下一秒的计算。这里我写成了一个方法：

```c++
inline int GetFPS(unsigned long &startTime , int &fps , int &tc) {
	const auto time = timeGetTime();
	if (time >= (startTime + 1000)) {
		fps = tc;
		tc = 0;
		startTime = timeGetTime();
	} else {
		++tc;
	}
	return fps;
}
```

&emsp;&emsp;这样，我们可以制作出一个并不太精准的 FPS 计数器。这里的 `timeGetTime()` 方法我们需要导入库 `Winmm.lib` 。

&emsp;&emsp;而 CPU 使用率的话我们则是需要使用一个叫做 pdh 的库，在导入了 `pdh.lib` 后 ，我们需要去初始化一些变量：

```c++
HQUERY hQuery = nullptr;
HCOUNTER hCounter = nullptr;
PDH_STATUS status;
```

&emsp;&emsp;`HQUERY` 是我们的查询器句柄，`HCOUNTER` 则是计数器句柄。声明变量之后我们需要调用方法来启用他：

```c++
status = PdhOpenQuery(nullptr, 0, &hQuery);
if (status != ERROR_SUCCESS) {
	canSample = false;
}
status = PdhAddCounter(hQuery, TEXT("\\Processor(_Total)\\% processor time"), 0, &hCounter);
if (status != ERROR_SUCCESS) {
	canSample = false;
}
```

&emsp;&emsp;第一个为打开查询，第二个则是为查询添加一个计数器，其中的`TEXT("\\Processor(_Total)\\% processor time")` 这一段则是我们的计数器路径。现在查询器和计数器都搞定了之后我们依旧将查询的任务写成一个方法：

```c++
inline int GetCPUUsage(const bool canSamleUsage , int &usage, unsigned long &lastTime , const HQUERY hQuery , const HCOUNTER hCounter) {
	const auto time = timeGetTime();
	if (canSamleUsage && time >= (lastTime + 1000)) {
		PDH_FMT_COUNTERVALUE value;
		PdhCollectQueryData(hQuery);
		PdhGetFormattedCounterValue(hCounter, PDH_FMT_LONG, nullptr, &value);
		usage = value.longValue;
		lastTime = timeGetTime();
	}
	return usage;
}
```

&emsp;&emsp;现在在 Main 里调用：

```c++
const string fpsText = "FPS : ";
int tCount = 0;
int fps = 0;
unsigned long startTime = timeGetTime();

const string cpuText = "CUP : ";
unsigned long lastSampleTime = timeGetTime();
int cpuUsage = 0;
bool canSample = true;
HQUERY hQuery = nullptr;
HCOUNTER hCounter = nullptr;
PDH_STATUS status;

status = PdhOpenQuery(nullptr, 0, &hQuery);
if (status != ERROR_SUCCESS) {
	MessageBox(nullptr, "ERROR::OpenQuery", "ERROR", MB_OK);
	canSample = false;
}
status = PdhAddCounter(hQuery, TEXT("\\Processor(_Total)\\% processor time"), 0, &hCounter);
if (status != ERROR_SUCCESS) {
	MessageBox(nullptr, "ERROR::AddCounter", "ERROR", MB_OK);
	canSample = false;
}

MSG msg;
ZeroMemory(&msg, sizeof(MSG));
while (msg.message != WM_QUIT) {
	if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE)) {
		TranslateMessage(&msg);
		DispatchMessage(&msg);
	}
	float color[] = { 0.0f , 0.0f , 0.0f , 1.0f };
	pImmediateContext->ClearRenderTargetView(pRenderTargetView, color);
    // Print
	PrintText(fpsText + to_string(GetFPS(startTime , fps , tCount)) , -620, 300, fonts , pVertexShader, pPixelShader, pInputLayout, pDevice, &vertices, &pImmediateContext);
	PrintText(cpuText + to_string(GetCPUUsage(canSample , cpuUsage , lastSampleTime , hQuery , hCounter)) , -620, 240, fonts , pVertexShader, pPixelShader, pInputLayout, pDevice, &vertices, &pImmediateContext);
    
	pSwapChain->Present(0, 0);
}
```

&emsp;&emsp;显示如下：

![1](https://image.ibb.co/nhA8Ux/image.png)

&emsp;&emsp;源代码地址：[**DX11Tutorial-FpsAndCpuUsage**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-FpsAndCpuUsage)

