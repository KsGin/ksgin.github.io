---
title: 【DirectX】6-代码框架3-键盘与鼠标
date: 2018-03-20 18:59:00
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 2: Creating a Framework and Window](http://www.rastertek.com/dx11tut02.html)

#### 学习记录

&emsp;&emsp;[上一篇文章](https://blog.ksgin.com/2018/03/20/【directx】5-代码框架2-cinput与cgraphics/#more) 中我们新增了 `CInput` 和 `CGraphics` ，但是两个里边都没有实际内容。所以只是填充了框架，而这一篇中我们来完善 `CInput` 类的分支：`CKeyboard` 和 `CMouse` 。

&emsp;&emsp;这一篇之后，我们的框架结构如下：

![key2](https://image.ibb.co/mATiNx/image.png)

<!--more-->

&emsp;&emsp;我们的 `CKeyboard` 用来提供基本的键盘操作，我们会使用一个 `keys` 数组来保存每一个键位的值。当某个键位按下时，我们更该这个位置的值。除此之外我们需要给出一个或者某个键值的方法，这就是我们这个 `Keyboard` 类最主要的功能（在原教程中，它的 `InputClass` 类直接提供了这些方法，但我将其提出作为单独的键盘类使用）。来看看这个类的声明：

```c++
class CKeyboard {
public:
	CKeyboard();
	CKeyboard(CKeyboard& keyboard);
	~CKeyboard();
	bool Initialize();
	void Shutdown();
	void KeyDown(unsigned int key);
	void KeyUp(unsigned int key);
	bool IsKeyDown(unsigned int key);
	unsigned int keyCount = 1024;

private:
	bool InitializeKeyboard();
	void ShutdownKeyboard();
	std::vector<bool> keys;
};
```

&emsp;&emsp;每个按键都有其对应的值，所以我们设定了一个 `keys` 数组用以保存这个键位是否被按下。`Initialize()` 这些方法和其他类都一样暂且不提，我们的 `Keydown()` 和 `Keyup()` 方法会更新 `keys` 的值，而 `IsKeyDown()` 则返回一个布尔值。实现如下：

```c++
CKeyboard::CKeyboard() {
	this->Initialize();
}

CKeyboard::CKeyboard(CKeyboard& keyboard) {
	*this = keyboard;
	this->Initialize();
}

CKeyboard::~CKeyboard() {

}

bool CKeyboard::Initialize() {

	return this->InitializeKeyboard();
}

void CKeyboard::Shutdown() {
	
	this->ShutdownKeyboard();
}

void CKeyboard::ShutdownKeyboard() {

}

void CKeyboard::KeyDown(const unsigned key) {
	keys[key] = true;
}

void CKeyboard::KeyUp(const unsigned key) {
	keys[key] = false;
}

bool CKeyboard::IsKeyDown(const unsigned key) {
	return keys[key];
}

bool CKeyboard::InitializeKeyboard() {
	keys = std::vector<bool>(keyCount, false);
	return true;
}
```

&emsp;&emsp;这个也比较简单，然后当我们做完这些后，可以向上给 `CInput` 中添加内容了。在 `CInput` 中我们添加了一个 `CKeyboard` 对象并在 `Initialize()` 中初始化。

```c++
class CInput {
public:
    ...
	CKeyboard *mKeyboard_;
	...
};

bool CInput::Initialize() {
	mKeyboard_ = new CKeyboard;
	if (!mKeyboard_) {
		return false;
	}
	this->InitializeInput();
	return true;
}

void CInput::Shutdown() {
	if (mKeyboard_) {
		mKeyboard_->Shutdown();
		delete mKeyboard_;
		mKeyboard_ = 0;
	}
	this->ShutdownInput();
}
```

&emsp;&emsp;而再往上一层，我们可以在 `CSystem` 类的消息处理函数中更新他们对于键盘的操作了。

```c++
LRESULT CSystem::MessageHandler(const HWND hwnd, const UINT umsg, const WPARAM wparam, const LPARAM lparam) {
	switch (umsg) {
	case WM_KEYDOWN:
		mInput_->mKeyboard_->KeyDown(static_cast<unsigned int>(wparam));
		break;
	case WM_KEYUP:
		mInput_->mKeyboard_->KeyUp(static_cast<unsigned int>(wparam));
		break;
	default: break;
	}
	return DefWindowProc(hwnd, umsg, wparam, lparam);
}
```

&emsp;&emsp;这样，我们的 `CKeyboard` 算是完全的添加了进去，接下来看 `CMouse` 。**事实上，DX 中有专门的输入设备 DXInput ，不过我们还没学到。** `CMouse` 我们暂时只给了获得和设置鼠标位置的方法。

```c++
class CMouse {
	CMouse();
	CMouse(CMouse& mouse);
	~CMouse();
	bool Initialize();
	void Shutdown();

	void SetPos();
	void GetPos();

private:
	bool InitializeMouse();
	void ShutdownMouse();
};
```

&emsp;&emsp;初始化那些方法暂且不表。在 `SetPos()` 中我们设置光标位置，`GetPos()` 自然就是读取了。我们会在这两个方法里使用 *WINAPI* 来获得或者设置数据，等到我们接触了 DXInput 之后会将其代替。

```c++
void CMouse::SetPos(const unsigned int& x, const unsigned int& y) {
	SetCursorPos(x, y);
}

void CMouse::GetPos(unsigned int& x, unsigned int& y) {
	auto*point = new POINT;
	GetCursorPos(point);
	x = point->x;
	y = point->y;
}
```

&emsp;&emsp;*(关于键盘与鼠标这里，只是暂时的这样封装。之后会用 DXInput 来实现 `CInput` 这个类。)*

