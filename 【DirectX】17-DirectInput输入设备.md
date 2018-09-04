---
title: 【DirectX】17-DirectInput输入设备
date: 2018-04-01 20:26:34
tags: 
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 13: Direct Input](http://www.rastertek.com/dx11tut13.html)

#### 学习记录

&emsp;&emsp;之前的内容几乎一直在学习 D3D ，今天这一篇中我们将学习 DirectInput ，DX 中的输入轮子。这一篇中将实现在屏幕上实时显示屏幕的坐标，以及使用 DirectInput 中的键盘控制来设置 ESC 键退出窗口。

<!--more-->

&emsp;&emsp;DirectInput 的使用和 D3D 异曲同工，都是需要先初始化一个设备接口，然后使用这个设备接口来创建具体的设备。不过 DirectInput 常用的内容不多，所以就不会像是 D3D 一样需要分好多篇文章来写。

&emsp;&emsp;使用 DirectInput ，我们需要导入头文件 `#include <dinput.h>` 和引用 `dinput8.lib` 和 `dxguid.lib` 静态库。来看看初始化一个 DirectInput 对象（关于每个方法具体参数参考 MSDN）：

```c++
IDirectInput8 *pInputDevice = nullptr;
hr = DirectInput8Create(hInstance, DIRECTINPUT8_VERSION, IID_IDirectInput8, reinterpret_cast<LPVOID*>(&pInputDevice), nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateInputDevice", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;有了这个对象之后，我们用它来创建键盘：

```c++
IDirectInputDevice8 *pKeyboard = nullptr;
IDirectInputDevice8 *pMouse = nullptr;
hr = pInputDevice->CreateDevice(GUID_SysKeyboard, &pKeyboard, nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateKeyBoard", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;设置键盘对象属性：

```c++
hr = pKeyboard->SetDataFormat(&c_dfDIKeyboard);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SetKeyBoardDataFormat", "ERROR", MB_OK);
	return hr;
}
hr = pKeyboard->SetCooperativeLevel(hWnd, DISCL_FOREGROUND | DISCL_EXCLUSIVE);	//设置为独占
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SetKeyBoardCooperativeLevel", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;Acquire 它：

```c++
hr = pKeyboard->Acquire();
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::KeyBoardAcquire", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;鼠标的创建以及设置和键盘一致：

```c++
hr = pInputDevice->CreateDevice(GUID_SysMouse, &pMouse, nullptr);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateMouse", "ERROR", MB_OK);
	return hr;
}
hr = pMouse->SetDataFormat(&c_dfDIMouse);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SetMouseDataFormat", "ERROR", MB_OK);
	return hr;
}
hr = pMouse->SetCooperativeLevel(hWnd, DISCL_FOREGROUND | DISCL_NONEXCLUSIVE);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::SetMouseCooperativeLevel", "ERROR", MB_OK);
	return hr;
}
hr = pMouse->Acquire();
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::MouseAcquire", "ERROR", MB_OK);
	return hr;
}
```

&emsp;&emsp;现在我们有了键盘和鼠标对象，可以使用它了，首先创建我们需要的数据（键盘为一个 `char` 数组保存键盘状态，鼠标为 `DIMOUSESTATE` 和 鼠标位置）：

```c++
char pKeyBoardState[256];
DIMOUSESTATE mouseState;
int mouseX = 0, mouseY = 0;	
```

&emsp;&emsp;在主循环中，我们通过鼠标和键盘对象的 `GetDeviceState` 函数来获得他们的状态，比如键盘如下：

```c++
hr = pKeyboard->GetDeviceState(sizeof(pKeyBoardState), static_cast<LPVOID>(&pKeyBoardState));
if (FAILED(hr)) {
	if (hr == DIERR_INPUTLOST || hr == DIERR_NOTACQUIRED) {
		pKeyboard->Acquire();
	}
	else {
		MessageBox(nullptr, "ERROR::GetKeyBoardState", "ERROR", MB_OK);
		return hr;
	}
}
```

&emsp;&emsp;这里有个地方需要注意，我们在 `GetDeviceState` 返回异常后，并没有直接返回，而是先判断了一下异常内容，当它因为未获得或者失去焦点的时候，我们尝试重新取 `Acquire` 它，如果不是这两个问题的话，自然就直接返回了。

&emsp;&emsp;鼠标和键盘基本一样，他的坐标则是用我们声明的坐标去加设备对象返回给我们的偏移量：

```c++
hr = pMouse->GetDeviceState(sizeof(DIMOUSESTATE), static_cast<LPVOID>(&mouseState));
if (FAILED(hr)) {
	if (hr == DIERR_INPUTLOST || hr == DIERR_NOTACQUIRED) {
		pMouse->Acquire();
	}
	else {
		MessageBox(nullptr, "ERROR::GetMouseState", "ERROR", MB_OK);
		return hr;
	}
}
mouseX += mouseState.lX;
mouseY += mouseState.lY;
```

&emsp;&emsp;现在，我们已经完整的获得了鼠标和键盘的对象，可以分别根据它的内容做一些其他的事情了，当键盘的 ESC 键按下时，发送一个 `QUIT` 的消息：

```c++
if (pKeyBoardState[DIK_ESCAPE] & 0x80) {
	PostQuitMessage(0);
}
```

&emsp;&emsp;同时我们可以在每帧里绘制当前的坐标位置，我们设计了一个方法去在屏幕上打印这个坐标：

```c++
inline HRESULT PrintMousePos(
	const int mouseX ,		// 鼠标坐标 x
	const int mouseY , 		// 鼠标坐标 y
	Font *fonts,
	ID3D11VertexShader *pVertexShader , 
	ID3D11PixelShader *pPixelShader , 
	ID3D11InputLayout *pInputLayout,
	ID3D11Device *pDevice , 
	ID3D11DeviceContext **pImmediateContext) {

	string xyStr = "Mouse X = " + to_string(mouseX) + " Y = " + to_string(mouseY);
	Vertex *vertices = nullptr;
	int nNumVertices = 0;

	InitVertex(pDevice , xyStr.c_str() , -800 , 400 , fonts , pImmediateContext , &nNumVertices , &vertices);
	(*pImmediateContext)->VSSetShader(pVertexShader, 0, 0);
	(*pImmediateContext)->PSSetShader(pPixelShader, 0, 0);
	(*pImmediateContext)->IASetInputLayout(pInputLayout);
	(*pImmediateContext)->Draw(nNumVertices * 6, 0);

	return S_OK;
}
```

&emsp;&emsp;我们可以在循环里去绘制，这样每帧都会生成不同的顶点数组并且去渲染它。当然这样是非常非常不理智的，如果要实际运用，一般会使用动态的更新顶点缓冲里的数据而非每一次循环单独生成。我这样的话你会发现运行一会后就炸了，因为一直在申请内存来存储顶点。。。。。。。。。。。。。。。。。。。

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/c0Akpn/image.png)