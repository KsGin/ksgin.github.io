---
title: 【DirectX】7-代码框架4-CDirect3D类
date: 2018-03-21 17:40:42
tags:	
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 3: Initializing DirectX 11](http://www.rastertek.com/dx11tut03.html)

#### 学习记录

&emsp;&emsp;在之前的文章中，我们简单的将我们的框架下的 `CInput` 填充了以下，实现了 `CKeyboard` 和 `CMouse` 两个基础类。输入貌似就可以暂时这样了，现在我们来看输出。也就是我们 `CSystem` 类下的另一个 `CGraphics` 。这篇文章中要实现的是对于 D3D 系统函数的封装，将其命名为 `CDirect3D` 。

&emsp;&emsp;更新 `CDirect3D` 之后我们的框架如下：

![CDirect3D1](https://image.ibb.co/cYudPc/image.png)

<!--more-->

&emsp;&emsp;之前我们实现了很多个类，但是都算是很简单的。而这一篇就比较复杂了，我们之前写过 dx 的初始化代码，也是大致清楚都需要做什么，先来看看这个类的声明。

```c++
class CDirect3D {
public:
	CDirect3D();
	CDirect3D(CDirect3D& cd);
	~CDirect3D();

	bool Initialize(int, int, bool, HWND, bool, float, float);
	void Shutdown();
	
	void BeginScene(float, float, float, float) const;
	void EndScene() const;

	ID3D11Device* GetDevice() const;
	ID3D11DeviceContext* GetDeviceContext() const;

	void GetProjectionMatrix(D3DXMATRIX&) const;
	void GetWorldMatrix(D3DXMATRIX&) const;
	void GetOrthoMatrix(D3DXMATRIX&) const;
	void GetVideoCardInfo(char*, int&) const;

private:
	bool mVsyncEnabled_{};
	int mVideoCardMemory_{};
	char mVideoCardDescription_[128]{};
	IDXGISwapChain* mSwapChain_{};
	ID3D11Device* mDevice_{};
	ID3D11DeviceContext* mDeviceContext_{};
	ID3D11RenderTargetView* mRenderTargetView_{};
	ID3D11Texture2D* mDepthStencilBuffer_{};
	ID3D11DepthStencilState* mDepthStencilState_{};
	ID3D11DepthStencilView* mDepthStencilView_{};
	ID3D11RasterizerState* mRasterState_{};
	D3DXMATRIX mProjectionMatrix_;
	D3DXMATRIX mWorldMatrix_;
	D3DXMATRIX mOrthoMatrix_;
};
```

&emsp;&emsp;嗯，看着确实很绝望了，这个类包括了大量的变量定义与函数声明。不过这里边的大部分的变量我们都见过（`IDXGISWapChain` ，`ID3D11Device` 等等），剩下的几个例如深度模板的变量我们在 OpenGL 中也是见过的（如果你没有学过，请忽略这个或者翻一下 OpenGL 的学习笔记）。

&emsp;&emsp;那么来看看这个类的实现吧，首先在构造函数里我们初始化这几个指针变量为空。

```c++
CDirect3D::CDirect3D() {
	mSwapChain_ = 0;
	mDevice_ = 0;
	mDeviceContext_ = 0;
	mRenderTargetView_ = 0;
	mDepthStencilBuffer_ = 0;
	mDepthStencilState_ = 0;
	mDepthStencilView_ = 0;
	mRasterState_ = 0;
}
```

&emsp;&emsp;现在来看看我们的 `Initialize()` 函数，这将是我们这个类中最重要的函数。

> The screenWidth and screenHeight variables that are given to this function are the width and height of the window we created in the SystemClass.
> Direct3D will use these to initialize and use the same window dimensions.
> The hwnd variable is a handle to the window.  Direct3D will need this handle to access the window previously created.
> The fullscreen variable is whether we are running in windowed mode or fullscreen.  Direct3D needs this as well for creating the window with the correct settings.
> The screenDepth and screenNear variables are the depth settings for our 3D environment that will be rendered in the window.  
> The vsync variable indicates if we want Direct3D to render according to the users monitor refresh rate or to just go as fast as possible.
>
> ​									——（教程原文中对于 initialize 方法的参数的描述）

&emsp;&emsp;在 `Initialize()` 中，我们首先声明了初始化我们成员变量所需要的一些东西。在我们初始化 D3D 之前，我们从显卡 / 显示器上获得刷新率，这样我们可以根据不同的计算机屏幕刷新率不同实现自适应。

```c++
IDXGIFactory* factory;
IDXGIAdapter* adapter;
IDXGIOutput* adapterOutput;
unsigned int numModes, i, numerator = 0, denominator = 0, stringLength;
DXGI_MODE_DESC* displayModeList;
DXGI_ADAPTER_DESC adapterDesc;
int error;

// 创建一个 DXGI 工厂类实例
HRESULT result = CreateDXGIFactory(__uuidof(IDXGIFactory), reinterpret_cast<void**>(&factory));
if (FAILED(result))
{
	return false;
}

// 使用工厂对象来创建一个显卡适配器
result = factory->EnumAdapters(0, &adapter);
if (FAILED(result))
{
	return false;
}

// 枚举出一个输出适配器
result = adapter->EnumOutputs(0, &adapterOutput);
if (FAILED(result))
{
	return false;
}

// 获取适合输出的显示格式的数量
result = adapterOutput->GetDisplayModeList(DXGI_FORMAT_R8G8B8A8_UNORM, DXGI_ENUM_MODES_INTERLACED, &numModes, NULL);
if (FAILED(result))
{
	return false;
}

// 创建 List 用来保存显示模式
displayModeList = new DXGI_MODE_DESC[numModes];
if (!displayModeList)
{
	return false;
}

// 填充显示模式数组
result = adapterOutput->GetDisplayModeList(DXGI_FORMAT_R8G8B8A8_UNORM, DXGI_ENUM_MODES_INTERLACED, &numModes, displayModeList);
if (FAILED(result))
{
	return false;
}

for (i = 0; i < numModes; i++)
{
	if (displayModeList[i].Width == static_cast<unsigned int>(screenWidth))
	{
		if (displayModeList[i].Height == static_cast<unsigned int>(screenHeight))
		{
			numerator = displayModeList[i].RefreshRate.Numerator;
			denominator = displayModeList[i].RefreshRate.Denominator;
		}
	}
}
```

&emsp;&emsp;可以看到我们调用了两次 `GetDisplayModeList` ，第一次为 `numModes` 赋值，之后我们有了显示模式的个数之后可以创建数组，之后才会将数组也传入进去用来保存显示模式列表。最后，我们遍历数组并且匹配我们所传入进来的 `width` 和 `height` 。最终得到合适的 `numerator` 和 `denominator` 值，用于我们最后在创建交换链的时候赋值。

```c++
// Get the adapter (video card) description.
result = adapter->GetDesc(&adapterDesc);
if (FAILED(result))
{
	return false;
}

// Store the dedicated video card memory in megabytes.
mVideoCardMemory_ = static_cast<int>(adapterDesc.DedicatedVideoMemory / 1024 / 1024);

// Convert the name of the video card to a character array and store it.
error = wcstombs_s(&stringLength, mVideoCardDescription_, 128, adapterDesc.Description, 128);
if (error != 0)
{
	return false;
}
```

&emsp;&emsp;做完这些后，我们再来获得显卡的内存，保存在 `mVideoCardMemory_` 里。当我们得到了这些数据后，就可以开始创建交换链那些东西了（我们之前做过的）。

&emsp;&emsp;先释放掉刚才为了获得显卡显示器数据创建的指针对象。

```c++
// Release the display mode list.
delete[] displayModeList;
displayModeList = 0;

// Release the adapter output.
adapterOutput->Release();
adapterOutput = 0;

// Release the adapter.
adapter->Release();
adapter = 0;

// Release the factory.
factory->Release();
factory = 0;
```

&emsp;&emsp;之后，初始化交换链描述：

```c++
DXGI_SWAP_CHAIN_DESC swapChainDesc;
// Initialize the swap chain description.
ZeroMemory(&swapChainDesc, sizeof(swapChainDesc));
swapChainDesc.BufferCount = 1;
swapChainDesc.BufferDesc.Width = screenWidth;
swapChainDesc.BufferDesc.Height = screenHeight;
swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
if (mVsyncEnabled_)
{
	swapChainDesc.BufferDesc.RefreshRate.Numerator = numerator;
	swapChainDesc.BufferDesc.RefreshRate.Denominator = denominator;
}
else
{
	swapChainDesc.BufferDesc.RefreshRate.Numerator = 0;
	swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;
}
swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
swapChainDesc.OutputWindow = hwnd;
swapChainDesc.SampleDesc.Count = 1;
swapChainDesc.SampleDesc.Quality = 0;
if (fullscreen) swapChainDesc.Windowed = false;
else swapChainDesc.Windowed = true;
swapChainDesc.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
swapChainDesc.BufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED;
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
swapChainDesc.Flags = 0;
```

&emsp;&emsp;初始化完了之后就可以直接创建了，然后是创建 `RenderTargetView` 这些。

```c++
// Set the feature level to DirectX 11.
D3D_FEATURE_LEVEL featureLevel = D3D_FEATURE_LEVEL_11_0;

// Create the swap chain, Direct3D device, and Direct3D device context.
result = D3D11CreateDeviceAndSwapChain(NULL, D3D_DRIVER_TYPE_HARDWARE, NULL, 0, &featureLevel, 1,D3D11_SDK_VERSION, &swapChainDesc, &mSwapChain_, &mDevice_, NULL, &mDeviceContext_);
if (FAILED(result))
{
	return false;
}

// Get the pointer to the back buffer.
result = mSwapChain_->GetBuffer(0, __uuidof(ID3D11Texture2D), reinterpret_cast<LPVOID*>(&backBufferPtr));
if (FAILED(result))
{
	return false;
}

// Create the render target view with the back buffer pointer.
result = mDevice_->CreateRenderTargetView(backBufferPtr, NULL, &mRenderTargetView_);
if (FAILED(result))
{
	return false;
}

// Release pointer to the back buffer as we no longer need it.
backBufferPtr->Release();
backBufferPtr = 0;
```

&emsp;&emsp;现在我们来创建深度缓冲描述，用它来创建[深度缓冲](https://zh.wikipedia.org/wiki/深度缓冲)，并且附加一个[深度模板](http://blog.csdn.net/huhaoxuan2010/article/details/78111171)。

```c++
// Initialize the description of the depth buffer.
ZeroMemory(&depthBufferDesc, sizeof(depthBufferDesc));

// Set up the description of the depth buffer.
depthBufferDesc.Width = screenWidth;
depthBufferDesc.Height = screenHeight;
depthBufferDesc.MipLevels = 1;
depthBufferDesc.ArraySize = 1;
depthBufferDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
depthBufferDesc.SampleDesc.Count = 1;
depthBufferDesc.SampleDesc.Quality = 0;
depthBufferDesc.Usage = D3D11_USAGE_DEFAULT;
depthBufferDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
depthBufferDesc.CPUAccessFlags = 0;
depthBufferDesc.MiscFlags = 0;

// Create the texture for the depth buffer using the filled out description.
result = mDevice_->CreateTexture2D(&depthBufferDesc, NULL, &mDepthStencilBuffer_);
if (FAILED(result))
{
	return false;
}

// Initialize the description of the stencil state.
ZeroMemory(&depthStencilDesc, sizeof(depthStencilDesc));

// Set up the description of the stencil state.
depthStencilDesc.DepthEnable = true;
depthStencilDesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK_ALL;
depthStencilDesc.DepthFunc = D3D11_COMPARISON_LESS;

depthStencilDesc.StencilEnable = true;
depthStencilDesc.StencilReadMask = 0xFF;
depthStencilDesc.StencilWriteMask = 0xFF;

// Stencil operations if pixel is front-facing.
depthStencilDesc.FrontFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
depthStencilDesc.FrontFace.StencilDepthFailOp = D3D11_STENCIL_OP_INCR;
depthStencilDesc.FrontFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
depthStencilDesc.FrontFace.StencilFunc = D3D11_COMPARISON_ALWAYS;

// Stencil operations if pixel is back-facing.
depthStencilDesc.BackFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
depthStencilDesc.BackFace.StencilDepthFailOp = D3D11_STENCIL_OP_DECR;
depthStencilDesc.BackFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
depthStencilDesc.BackFace.StencilFunc = D3D11_COMPARISON_ALWAYS;

// Create the depth stencil state.
result = mDevice_->CreateDepthStencilState(&depthStencilDesc, &mDepthStencilState_);
if (FAILED(result))
{
	return false;
}

// Set the depth stencil state.
mDeviceContext_->OMSetDepthStencilState(mDepthStencilState_, 1);

// Initailze the depth stencil view.
ZeroMemory(&depthStencilViewDesc, sizeof(depthStencilViewDesc));

// Set up the depth stencil view description.
depthStencilViewDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
depthStencilViewDesc.ViewDimension = D3D11_DSV_DIMENSION_TEXTURE2D;
depthStencilViewDesc.Texture2D.MipSlice = 0;

// Create the depth stencil view.
result = mDevice_->CreateDepthStencilView(mDepthStencilBuffer_, &depthStencilViewDesc, &mDepthStencilView_);
if (FAILED(result))
{
	return false;
}

// Bind the render target view and depth stencil buffer to the output render pipeline.
mDeviceContext_->OMSetRenderTargets(1, &mRenderTargetView_, mDepthStencilView_);
```

&emsp;&emsp;完成这个，我们来创建一个  Raster ：

```c++
rasterDesc.AntialiasedLineEnable = false;
rasterDesc.CullMode = D3D11_CULL_BACK;
rasterDesc.DepthBias = 0;
rasterDesc.DepthBiasClamp = 0.0f;
rasterDesc.DepthClipEnable = true;
rasterDesc.FillMode = D3D11_FILL_SOLID;
rasterDesc.FrontCounterClockwise = false;
rasterDesc.MultisampleEnable = false;
rasterDesc.ScissorEnable = false;
rasterDesc.SlopeScaledDepthBias = 0.0f;

// Create the rasterizer state from the description we just filled out.
result = mDevice_->CreateRasterizerState(&rasterDesc, &mRasterState_);
if (FAILED(result))
{
	return false;
}

// Now set the rasterizer state.
mDeviceContext_->RSSetState(mRasterState_);
```

> &emsp;&emsp;Now that the render targets are setup we can continue on to some extra functions that will give us more control over our scenes for future tutorials.
> First thing is we'll create is a rasterizer state.  This will give us control over how polygons are rendered.  
> &emsp;&emsp;We can do things like make our scenes render in wireframe mode or have DirectX draw both the front and back faces of polygons.
> &emsp;&emsp;By default DirectX already has a rasterizer state set up and working the exact same as the one below but you have no control to change it unless you set up one yourself.

&emsp;&emsp;我们自定义光栅化可以使得我们实现更多的效果。

&emsp;&emsp;最后，创建 `ViewPort` 以及初始化那几个矩阵（可以看到我们并没有 `ViewMatrix`，这个准备封装在摄像机类里 `CCamera`）。

```c++
// Setup the viewport for rendering.
viewport.Width = static_cast<float>(screenWidth);
viewport.Height = static_cast<float>(screenHeight);
viewport.MinDepth = 0.0f;
viewport.MaxDepth = 1.0f;
viewport.TopLeftX = 0.0f;
viewport.TopLeftY = 0.0f;

// Create the viewport.
mDeviceContext_->RSSetViewports(1, &viewport);

// Setup the projection matrix.
const float fieldOfView = static_cast<float>(D3DX_PI) / 4.0f;
const float screenAspect = static_cast<float>(screenWidth) / static_cast<float>(screenHeight);

// Create the projection matrix for 3D rendering.
D3DXMatrixPerspectiveFovLH(&mProjectionMatrix_, fieldOfView, screenAspect, screenNear, screenDepth);

// Initialize the world matrix to the identity matrix.
D3DXMatrixIdentity(&mWorldMatrix_);

// Create an orthographic projection matrix for 2D rendering.
D3DXMatrixOrthoLH(&mOrthoMatrix_, static_cast<float>(screenWidth), static_cast<float>(screenHeight), screenNear, screenDepth);
```

&emsp;&emsp;终于，我们的初始化方法结束了（好长），可以看到是非常麻烦的。幸好我们实际编程中是不会去专注于这里。

&emsp;&emsp;解决了初始化，剩下的就简单的多了。`Shutdown` 方法中我们将 `delete` 掉成员变量指针。

```c++
void CDirect3D::Shutdown() {
	if (mSwapChain_) {
		mSwapChain_->Release();
		mSwapChain_ = 0;
	}
	if (mRasterState_) {
		mRasterState_->Release();
		mRasterState_ = 0;
	}
	if (mDevice_) {
		mDevice_->Release();
		mDevice_ = 0;
	}
	if (mDeviceContext_) {
		mDeviceContext_->Release();
		mDeviceContext_ = 0;
	}
	if (mRenderTargetView_) {
		mRenderTargetView_->Release();
		mRenderTargetView_ = 0;
	}
	if (mDepthStencilBuffer_) {
		mDepthStencilBuffer_->Release();
		mDepthStencilBuffer_ = 0;
	}
	if (mDepthStencilState_) {
		mDepthStencilState_->Release();
		mDepthStencilState_ = 0;
	}
	if (mDepthStencilView_) {
		mDepthStencilView_->Release();
		mDepthStencilView_ = 0;
	}
}
```

&emsp;&emsp;在 `BeginScene` 和 `EndScene` 中我们实现缓冲清空（颜色和深度）和交换。

```c++
void CDirect3D::BeginScene(const float r, const float g, const float b , const float a) const {
	float color[] = { r , g , b , a };
	mDeviceContext_->ClearRenderTargetView(mRenderTargetView_, color);
	mDeviceContext_->ClearDepthStencilView(mDepthStencilView_, D3D11_CLEAR_DEPTH, 1.0f, 0);
}

void CDirect3D::EndScene() const {
	if (mVsyncEnabled_) {
		mSwapChain_->Present(1, 0);
	} else {
		mSwapChain_->Present(0, 0);
	}
}
```

&emsp;&emsp;剩下的方法则是简单的赋值了：

```c++
ID3D11Device* CDirect3D::GetDevice() const {
	return mDevice_;
}

ID3D11DeviceContext* CDirect3D::GetDeviceContext() const {
	return mDeviceContext_;
}

void CDirect3D::GetProjectionMatrix(D3DXMATRIX& projectionMatrix) const {
	projectionMatrix = mProjectionMatrix_;
}

void CDirect3D::GetWorldMatrix(D3DXMATRIX& worldMatrix) const {
	worldMatrix = mWorldMatrix_;
}

void CDirect3D::GetOrthoMatrix(D3DXMATRIX& orthoMatrix) const {
	orthoMatrix = mOrthoMatrix_;
}

void CDirect3D::GetVideoCardInfo(char* cardName, int& memory) const {
	strcpy(cardName, mVideoCardDescription_);
	memory = mVideoCardMemory_;
}
```

&emsp;&emsp;终于，我们实现了 `CDirect3D` 类。现在来修改 `CSystem` 使得它调用 `CDirect3D` 。比如初始化或者删除，还有在 `Render()` 方法的修改：

```c++
bool CGraphics::Initialize(const int screenWidth, const int screenHeight, const HWND hwnd) {

	mDirect3D_ = new CDirect3D;
	if (!mDirect3D_) {
		return false;
	}
	const HRESULT hr = mDirect3D_->Initialize(screenWidth, screenHeight, VSYNC_ENABLED, hwnd, FULL_SCREEN, SCREEN_DEPTH, SCREEN_NEAR);
	if (FAILED(hr)) {
		return false;
	}

	return this->InitializeGraphics();
}

void CGraphics::Shutdown() {
	if (mDirect3D_) {
		mDirect3D_->Shutdown();
		delete mDirect3D_;
		mDirect3D_ = 0;
	}

	this->ShutdownGraphics();
}

bool CGraphics::RenderGraphics(const float r , const float g , const float b , const float a) {

	mDirect3D_->BeginScene(r, g, b, a);

	mDirect3D_->EndScene();

	return true;
}
```

&emsp;&emsp;这里，我们的 `Initialize` 需要增加 `rgba` 的参数。

&emsp;&emsp;这一篇很长，写的时候也比较乱，之后若是有时间的话可能会进行修改。实现框架并不是我们最终的目的，在这个过程中我们介绍它的实现的过程才是最终需要学习的内容。