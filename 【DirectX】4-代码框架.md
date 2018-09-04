---
title: 【DirectX】4-代码框架
date: 2018-03-19 21:39:58
tags:	
	- DirectX
	- Computer graphics
---

&emsp;&emsp;在之前的代码中，我们使用了大量的代码去绘制了一个三角形。对于我们来说实在是有点麻烦了，如果每次都需要写这么多的代码的话无疑是很容易让人写崩溃的。在此和之后的几篇文章，我们将会对之前使用的基本功能进行封装。

&emsp;&emsp;此篇所看教程来自：[**DirectX 11 Tutorials**](http://www.rastertek.com/tutdx11.html) 。

<!--more-->

&emsp;&emsp;首先，我们以一个 CSystem 类为总类（我们的代码中类以 C 开头，Class 的意思）。CSystem 类将直接用在我们的 main 方法里，它会封装一些最基本的功能，包括我们所必须的 windows 窗口的创建。来看看我们的 CSystem 类声明：

```c++
class CSystem
{
public:
	CSystem();
	CSystem(const CSystem&);
	~CSystem();

	bool Initialize();
	void Shutdown();
	void Run() const;
	LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM) const;
private:
	void InitializeWindows(int&, int&);
	void ShutdownWindows();
private:
	LPCSTR mApplicationName_;
	HINSTANCE mHinstance_ = nullptr;
	HWND mHwnd_;
};
```

&emsp;&emsp;在这个类中封装了不少的变量和函数，我们会依次的解释他们，首先在构造方法中我们初始化那几个 `private` 变量（稍后我们会介绍每一个变量的作用）。

```c++
CSystem::CSystem(): mApplicationName_(nullptr), mHinstance_(nullptr), mHwnd_(nullptr),mInput_(0),mGraphics_(0) {
    this->Initialize();
}
CSystem::CSystem(const CSystem& other) {
	*this = other;
    this->Initialize();
}
```

&emsp;&emsp;在这个类中，我们封装了 `Initialize` 方法，这个方法会调用 `InitializeWindows` 为我们创建一个窗口，并使用 `private` 属性 `mHwnd` 来作为句柄。来看看这两个方法具体的实现：

```c++
bool CSystem::Initialize() {
	int screenWidth = 1920;
	int screenHeight = 1080;
	InitializeWindows(screenWidth, screenHeight);
	return true;
}
```

&emsp;&emsp;在 `CSystem::Initialize()` 中，我们只是简单的调用了 `InitializeWindows();` （事实上并不止这些，但是现在我们只有这一个类，功能也极其简单）。

&emsp;&emsp;接下来看 `InitializeWindows();` 方法：

```c++
void CSystem::InitializeWindows(int& screenWidth, int& screenHeight)
{
	mApplicationName_ = "DirectX";
	mHinstance_ = GetModuleHandle(NULL);
    
	WNDCLASS wc;
	wc.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
	wc.lpfnWndProc = MessageHandler;
	wc.cbClsExtra = 0;
	wc.cbWndExtra = 0;
	wc.hInstance = mHinstance_;
	wc.hIcon = LoadIcon(NULL, IDI_WINLOGO);
	wc.hCursor = LoadCursor(NULL, IDC_ARROW);
	wc.hbrBackground = static_cast<HBRUSH>(GetStockObject(BLACK_BRUSH));
	wc.lpszMenuName = NULL;
	wc.lpszClassName = mApplicationName_;
    
	RegisterClass(&wc);

	mHwnd_ = CreateWindow(mApplicationName_, mApplicationName_,WS_OVERLAPPEDWINDOW,
		0 , 0, screenWidth, screenHeight, NULL, NULL, mHinstance_, NULL);

	ShowWindow(mHwnd_, SW_SHOW);
    UpdateWindow(mHwnd_);
}
```

&emsp;&emsp;请注意我们在 `InitializeWindows` 方法里初始化了窗口，但是在设置窗口类的 `lpfnWndProc` 属性的时候，我们虽然在 CSystem 类里定义了一个 `MessageHandler` ，但是却无法将他直接用在这里（），所以我们还需要创建一个额外的（不在类里边的 `WndProc` 方法）。

```c++
// 在 CSystem.h 中定义一个 CSystem 对象和一个 WndProc 方法
static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
static CSystem* applicationHandle = 0;

// 我们定义的全部 WndProc 方法，并在里边调用了 CSystem 类对象的 MessageHandler 方法。
LRESULT CALLBACK WndProc(const HWND hwnd, const UINT umessage, const WPARAM wparam, const LPARAM lparam)
{
	switch (umessage)
	{
	case WM_DESTROY:
		PostQuitMessage(0);
		return 0;
	case WM_CLOSE:
		PostQuitMessage(0);
		return 0;
	default:
		return applicationHandle->MessageHandler(hwnd, umessage, wparam, lparam);
	}
}
```

&emsp;&emsp;而在 `MessageHandler` 方法里，我们只是简单的返回了 Windows 默认的消息函数。

```c++
LRESULT CSystem::MessageHandler(const HWND hwnd, const UINT umsg, const WPARAM wparam, const LPARAM lparam) {
	return DefWindowProc(hwnd, umsg, wparam, lparam);
}
```

&emsp;&emsp;初始化方法结束后，我们使用 `CSystem::Run();` 来启动窗口：

```c++
void CSystem::Run() const {
	MSG msg;
	ZeroMemory(&msg, sizeof(MSG));
	while (!msg.message != WM_QUIT)
	{
		// Handle the windows messages.
		if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
		{
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
	}
}
```

&emsp;&emsp;`Run();` 方法里我们则是简单的封装了消息循环。当然，这里将是之后添加内容的主要地方。

&emsp;&emsp;最后，我们还有 `Shutdown();` 和 `ShutdownWindows();` 尚未介绍，和初始化一样，`public` 的 `Shutdown();` 也只是简单的调用了 `ShutdownWindows();` 。

```c++
void CSystem::Shutdown() {
    ShutdownWindows();
}
```

&emsp;&emsp;在 `CSystem::ShutdownWindows()` 中则是什么都没有做（暂时不需要）。

```c++
void CSystem::ShutdownWindows() {
}
```

&emsp;&emsp;这样，我们只是简单的将窗口的初始化封装到了类中，可以现在来试试效果：

```C++
int WINAPI WinMain(HINSTANCE hInstance , HINSTANCE hPrevInstance , LPSTR lpCmdLine , INT nCmdShow) {
	CSystem *system = new CSystem;
	system->Run();
	return 0;
}
```

&emsp;&emsp;点击启动，成功应该是一个黑色的窗口：

![Empty Window](https://image.ibb.co/i3ut5H/image.png)