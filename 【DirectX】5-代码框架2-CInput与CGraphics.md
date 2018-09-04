---
title: 【DirectX】5-代码框架2-CInput与CGraphics
date: 2018-03-20 08:26:01
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 2: Creating a Framework and Window](http://www.rastertek.com/dx11tut02.html)

#### 学习记录

&emsp;&emsp;上一篇文章中，我们构建了一个用于窗口创建的类 `CSystem` ，并成功使用它启动了窗口。可以看到，在我们封装之后，它的使用及其的方便。而这篇文章中，我们将对其进行扩展，在我们创建窗口后，我们将在主循环中写入程序逻辑。包括我们的鼠标键盘输入以及对应的程序输出，因此，我们将实现 `CInput` 和 `CGraphics` 类（之所以不以 `COutput` 是因为我们的程序不仅仅包含图形一种输出，但我们现在只实现这一种）。在完成今天的两个类后，我们的框架结果将会变为下图的样式 。

![dx framework 1](https://image.ibb.co/cAdBaH/image.png)

<!--more-->

&emsp;&emsp;我们的 `CInput` 类将包含鼠标键盘的处理，鼠标与键盘将成为子类来设计，所以这里的 `CInput` 类简直简单的可怕。

```c++
class CInput {
public:
	CInput();
	CInput(CInput& input);
	~CInput();
	bool Initialize();
	void Shutdown();
private:
	bool InitializeInput();
	void ShutdownInput();
};
```

&emsp;&emsp;这将是我们的 `CInput` 类声明，它仅仅包含了 `Initialize()`  ， 以及` Shutdown()`函数，以及构造和析构函数。我们将在之后为这个类添加键盘处理 `CKeyboard` 和鼠标处理 `CMouse` 类（下一篇文章再谈）。

&emsp;&emsp;`CInput` 类的构造方法中我们调用 `Initialize()` ，而在 `Initialize()` 中调用 `InitializeInput()` ，在 `Shutdown()` 方法里调用 `ShutdownInput()` 方法。

&emsp;&emsp;现在我们已经有了 `CInput` 类，则准备在 `CSystem` 里添加对应的元素 。在 `CSystem` 的声明里为它添加一个 `CInput` 的指针对象：

```c++
CInput *mInput_;
```

&emsp;&emsp;在 `CSystem` 的 `Initialize()` 里我们将初始化 `CInput` 指针对象，最后在析构函数里释放掉它。

```c++
bool CSystem::Initialize() {
	... 
	this->mInput_ = new CInput;
	if (!mInput_) {
		return false;
	}
	...
	return true;
}

void CSystem::Shutdown() {
	delete mInput_;
	delete mGraphics_;
}
```

&emsp;&emsp;&emsp;现在，再来看看我们的 `CGraphics` 类：

```c++
class CGraphics {
public:
	CGraphics();
	CGraphics(CGraphics& cGraphics);
	~CGraphics();
	bool Initialize();
	void Shutdown();
	bool Render();
private:
	bool InitializeGraphics();
	void ShutdownGraphics();
	bool RenderGraphics();
};
```

&emsp;&emsp;我们的 `CGraphics` 类中的 `Initialize()` 与 `Shutdown()` 和 `CInput` 中的一样，暂且不表。`Render()` 暂时也是简单的调用了 `RenderGraphics()` 。而 `RenderGraphics()` 则是负责了主要的东西，因为我们并没有后续内容，所以在此也暂时不提。

&emsp;&emsp;综上所述，我们的 `CGraphics` 类也是相当简单，现在和 `CInput` 一样将他加入到 `CSystem` 的初始化与结束中去：

```c++
bool CSystem::Initialize() {
	int screenWidth = 1920;
	int screenHeight = 1080;
	InitializeWindows(screenWidth, screenHeight);
	this->mInput_ = new CInput;
	if (!mInput_) {
		return false;
	}
	this->mGraphics_ = new CGraphics;
	if (!mGraphics_) {
		return false;
	}
	return true;
}

void CSystem::Shutdown() {
	if (mInput_) {
		mInput_->Shutdown();
		delete mInput_;
		mInput_ = 0;
	}
	if (mGraphics_) {
		delete mGraphics_;
		mGraphics_ = 0;
	}
	ShutdownWindows();
}
```

&emsp;&emsp;以上是我们增加了 `CInput` 和 `CGraphics` 类后的初始化。但是虽然进行了初始化，但是在代码中却并没有真的调用它，接下来看看在 `CSystem` 中的调用。

&emsp;&emsp;`CInput` 我们因为没有具体实现的原因，没什么可以动用的。而 `CGraphics` 则需要在 `Run` 方法里进行修改。

```c++
void CSystem::Run() const {
	MSG msg;
	ZeroMemory(&msg, sizeof(MSG));
	while (msg.message != WM_QUIT)
	{
		if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
		{
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
		if (mGraphics_) {
			mGraphics_->Render();
		}
	}
}
```

&emsp;&emsp;我们在 `CSystem::Run()` 里调用了 `CGraphics::Render()` 方法。

&emsp;&emsp;现在，你点击运行，如果没有错误的话，那么那个黑色窗口应该还会出现。

![DX Framework2](https://image.ibb.co/ieKWQH/image.png)

&emsp;&emsp;下一篇我们将会实现 `CInput` 下的类 `CKeyboard` 和 `CMouse` 。