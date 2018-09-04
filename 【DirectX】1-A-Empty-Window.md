---
title: 【DirectX】1-A-Empty-Window
date: 2018-03-14 19:28:58
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;哈哈哈哈哈哈是不是超熟悉的。继 OpenGL 基础学完后，在继续钻研 OpenGL 比较深入的内容的同时，也顺便学学 DirectX 吧，毕竟在游戏界，DirectX 还是主流。在很久之前，写者也是接触了一些 Windows 编程的，当然仅仅只是一点。也就写出来了几个窗口，还是跟着教程的。现在在学习了 OpenGL 之后，那些东西也有了一些理解，是时候继续学习了。

<!--more-->

&emsp;&emsp;在学习 DirectX 之前要先介绍下 Windows 的窗口编程，以及 Windows 的消息机制。

##### Windows 窗口

![Window](https://image.ibb.co/gTCMZc/image.png)

&emsp;&emsp;由图可见，Windows （WS_EX_TOPMOST | WS_OVERLAPPEDWINDOW）窗口共分为**标题栏**，**可调按钮（最大化，最小化和关闭）**，**客户区**几个区域。&emsp;我们的渲染一般都是在客户区里边，当然 Windows 也会给我们提供修改其他部位的 API。在 Windows 编程中，我们使用 HWND（句柄） 来标记窗口。

##### Windows 消息机制

&emsp;&emsp;Windows 使用消息队列来进行维护窗口消息，比如我们鼠标光标在窗口里按动了左键。此时操作系统会感知到这个事件，并且将一个左键消息发送到窗口的**消息队列**中，而我们的程序从消息队列中取出来消息并进行相应的处理。这个处理则是操作系统调用程序中处理消息的函数 WndProc 。

&emsp;&emsp;消息队列，则是 Windows 在程序启动的时候创建的一个消息容器，这个容器用来存放所有的该程序窗口的消息。在我们触发了事件之后，操作系统将消息放入消息队列，而我们的程序会不断的从消息队列中读取消息并且处理他，这就是 Windows 窗口的消息机制。

##### 创建一个 Windows 窗口

+ ###### Windows 程序入口

  &emsp;&emsp;我们在学习 C/C++ 的时候已经写过不少的程序了，他们在 Windows 中都输入控制台应用，甚至在我们的 OpenGL 学习中，使用 Windows 开发的时候，也都是 Win32 控制台应用。在 Win32 控制台应用里，我们的程序入口时 main 方法：

  ```c++
  int main(int argc , char** argv){
      ...
  }
  ```

  &emsp;&emsp;而在 Windows 程序中，我们则是使用了 WinMain 的入口方法：

  ```c++
  int WINAPI WinMain(
      HINSTANCE hInstance, 		//当前运行的实例的句柄
      HINSTANCE hPreInstince, 	//上一个运行的实例的句柄
      LPSTR lpCmdLine, 			//程序运行的命令行参数
      int nCmdShow){				//程序窗口的显示方式（不显示=0，正常显示=1，最小化=2，最大化=3）
      ...
  }
  ```

  &emsp;&emsp;注释上我们说明了各个参数的意义，有些参数我们在之后的代码中要用到。

+ ###### 设计 Windows 类

  &emsp;&emsp;在 OpenGL 中，我们在创建窗口之前要进行 glfw 的配置，glew 的配置等，而在 Windows 程序窗口中，Windows 系统提供给了我们一个 Window 类：[WNDCLASS](https://msdn.microsoft.com/en-us/library/ms633576%28v=vs.85%29.aspx)，我们可以通过填充这个类的属性来配置我们的窗口类。

  &emsp;&emsp;窗口类的定义如下：

  ```c++
  typedef struct tagWNDCLASS {
    UINT      style;				//窗口显示样式，枚举值
    WNDPROC   lpfnWndProc;		//消息处理回调函数
    int       cbClsExtra;			//为窗口类额外分配的字节数
    int       cbWndExtra;			//为窗口实例额外分配的字节数
    HINSTANCE hInstance;			//窗口过程的实例句柄
    HICON     hIcon;				//窗口图标
    HCURSOR   hCursor;			//窗口光标
    HBRUSH    hbrBackground;		 //窗口背景画刷
    LPCTSTR   lpszMenuName;		//窗口菜单
    LPCTSTR   lpszClassName;		//窗口类的名字（！和CreateWindow里的参数必须一样！）
  } WNDCLASS, *PWNDCLASS;
  ```

  &emsp;&emsp;*style* 参数的具体枚举值我们可以看 MSDN 文档（在学习 Windows 开发的时候，MSDN 是我们永远的老师）：[Window Class Styles](https://msdn.microsoft.com/en-us/library/ff729176%28v=vs.85%29.aspx)

  &emsp;&emsp;我们创建 WNDCLASS 的实例，并为他赋值，即设计好了我们的窗口类。如下：

  ```c++
  WNDCLASS winClass;
  winClass.style = CS_VREDRAW | CS_HREDRAW;
  winClass.hInstance = hInstance;
  winClass.cbClsExtra = 0;
  winClass.cbWndExtra = 0;
  winClass.hCursor = LoadCursor(nullptr, IDC_ARROW);
  winClass.hIcon = LoadIcon(nullptr, IDC_ICON);
  winClass.hbrBackground = static_cast<HBRUSH>(GetStockObject(DKGRAY_BRUSH));
  winClass.lpfnWndProc = static_cast<WNDPROC>(WndProc);
  winClass.lpszClassName = "windowClass";
  winClass.lpszMenuName = nullptr;
  ```

+ ###### 注册窗口类

  &emsp;&emsp;在设计好窗口类之后，我们需要向 Windows 注册这个类（请注意我们只是用给定的类模板设计了我们的窗口类，在程序中我们的 windowClass 只是一个 WNDCLASS 的实例对象而已，我们需要注册他成为一个类而让我们可以使用它来创建窗口）。

  &emsp;&emsp;我们使用 RegisterClass 方法来注册我们的窗口类，传入 WNDCLASS 的实例对象，他将为我们创建名为winClass.lpszClassName 的窗口类。

  ```c++
  if (!RegisterClass(&winClass)) {
  	MessageBox(nullptr,"ERROR::WINDOWS_CLASS::REGISTER_ERROR", "REGISTER REEOR" , 0);
  	return 0;
  }
  ```

+ ###### 创建窗口

  &emsp;&emsp;注册完了窗口类之后，我们便可以使用它来创建窗口，使用 CreateWindow 方法：

  ```c++
  HWND WINAPI CreateWindow(
    _In_opt_ LPCTSTR   lpClassName,	//窗口类名称
    _In_opt_ LPCTSTR   lpWindowName,	//窗口名称
    _In_     DWORD     dwStyle,		//窗口样式，枚举值
    _In_     int       x,				//窗口左上角坐标
    _In_     int       y,				//窗口左上角坐标
    _In_     int       nWidth,		//窗口宽度
    _In_     int       nHeight,		//窗口高度
    _In_opt_ HWND      hWndParent,	//父窗口（创建子窗口时有用）
    _In_opt_ HMENU     hMenu,			//窗口菜单
    _In_opt_ HINSTANCE hInstance,		//实例句柄
    _In_opt_ LPVOID    lpParam		//附加参数
  );
  ```

  &emsp;&emsp;各个参数的用途可以看注释，关于窗口样式的详细信息看 MSDN 文档：[Window Styles](https://msdn.microsoft.com/en-us/library/ms632600%28v=vs.85%29.aspx)，当窗口创建成功时会返回他的窗口句柄 hWnd。

  &emsp;&emsp;现在我们使用它来创建一个窗口：

  ```c++
  const auto windowHandle = CreateWindow(
      "windowClass", 
      "Empty Window", 
      WS_EX_TOPMOST | WS_OVERLAPPEDWINDOW, 
      0, 
      0, 
      1920, 
      1080, 
      nullptr, 
      nullptr, 
      hInstance, 
      nullptr);
  if (!windowHandle) {
  	MessageBox(nullptr,"ERROR::WINDOWS_CREATE::CREATE_ERROR", "CREATE REEOR" , 0);
  	return 0;
  }
  ```

+ ###### 窗口更新

  &emsp;&emsp;创建好窗口后，我们可以使用 ShowWindow 来显示窗口，而在窗口更新之后，则可以使用 UpdateWindow 来更新窗口显示，在调用 ShowWindow 之后也需要调用 UpdateWindow 。

  ```c++
  ShowWindow(windowHandle, SW_SHOWNORMAL);
  UpdateWindow(windowHandle);
  ```

  &emsp;&emsp;ShowWindow 有两个参数，窗口句柄和窗口的显示方式，这也是一个枚举值，具体值属性我们可以看这里：[Show Controls](https://msdn.microsoft.com/en-us/library/ms633548%28v=vs.85%29.aspx)

+ ###### 消息处理

  &emsp;&emsp;文章前边介绍了 Windows 的消息处理方式，而现在需要写一个消息循环来不断的从消息队列读取消息并且处理消息，这里我们需要介绍 Windows 封装好的消息结构体和消息处理函数。

  ```c++
  //消息结构体
  typedef struct tagMSG {     
      HWND hwnd;  	//窗口句柄
      UINT message;  	//消息标识
      WPARAM wParam;  //消息附加信息
      LPARAM lParam;  //消息附加信息
      DWORD time;  	//消息到达消息队列的时间
      POINT pt;  		//鼠标当前位置
  } MSG;  

  //从消息队列中得到消息	成功返回非零
  BOOL WINAPI PeekMessage(
    _Out_    LPMSG lpMsg,			//msg指针
    _In_opt_ HWND  hWnd,			//窗口句柄
    _In_     UINT  wMsgFilterMin,	//指定被检查的消息范围第一个消息，通常为0
    _In_     UINT  wMsgFilterMax,	//指定被检查的消息范围最后一个消息，通常为0
    _In_     UINT  wRemoveMsg		//是否将消息从消息队列中删除（PM_NOREMOVE 保留，PM_REMOVE 删除）
  );
  ```

  &emsp;&emsp;我们通过使用 PeekMessage（还有个 GetMessage也可以得到消息，但是不会 Remove） 来接收消息，然后使用 TranslateMessage 转化消息（将其转换为字符消息），最后通过 DispatchMessage 来将消息传递给窗口处理函数。

  ```c++
  MSG uMsg;
  ZeroMemory(&uMsg, sizeof(MSG));
  uMsg.message = WM_ACTIVATE;
  while (uMsg.message != WM_QUIT) {
  	if (PeekMessage(&uMsg, nullptr, 0, 0, PM_REMOVE)) {	//使用 PeekMessage
  		TranslateMessage(&uMsg);		//使用 TranslateMessage
  		DispatchMessage(&uMsg);			//使用 DispatchMessage
  	}
  }
  ```

  &emsp;&emsp;这两个方法都比较简单，传入消息结构体就可以了。之后，我们在窗口内部产生的消息将会被翻译成字符消息后送往 WindowProc（窗口处理消息），这是一个回调（CALLBACK）函数。

  ```C++
  LRESULT CALLBACK WndProc(
      const HWND hwnd, 	//窗口句柄
      const UINT uMsg, 	//消息结构体
      const WPARAM wParam, //消息附加参数
      LPARAM lParam)		//消息附加参数
  {
      ...
  }
  ```

  &emsp;&emsp;这个窗口过程函数是一个 CALLBACK 函数，我们在设置窗口类的时候就将他绑定到了窗口类的过程函数上`winClass.lpfnWndProc = static_cast<WNDPROC>(WndProc);`，他会在收到消息后自动被调用。我们可以在他内部根据消息的不同来做不一样的处理。

  &emsp;&emsp;Windows 将各种消息封装为常量，所以我们可以直接根据消息的值来判断：

  ```c++
  LRESULT CALLBACK WndProc(const HWND hwnd, const UINT uMsg, const WPARAM wParam, LPARAM lParam) {
  	switch (uMsg) {
  	case WM_LBUTTONDOWN:
  		MessageBox(nullptr, "鼠标按下", "消息详情", 0);
  		break;
  	default: break;
  	}
  	return DefWindowProc(hwnd, uMsg, wParam, lParam);
  }
  ```

  &emsp;&emsp;Windows 窗口的消息常量便是以 `WM_` 开头的常量值。

  &emsp;&emsp;我们使用了 switch ，当 uMsg 为 `WM_LBUTTONDOWN` （鼠标左键按下）时弹出一个提示框，来看看效果吧：

  ![Window](https://image.ibb.co/kJsGHx/image.png)

  &emsp;&emsp;成功！

  &emsp;&emsp;完整的窗口代码在 Github ，点这里前往：[A Win32 Window](https://github.com/KsGin/LearnDirectX/blob/master/A-Win32-Window/main.cpp)