---
title: 【DirectX】8-代码框架5-Shader
date: 2018-03-22 17:37:54
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 4: Buffers, Shaders, and HLSL](http://www.rastertek.com/dx11tut04.html)

#### 学习记录

&emsp;&emsp;这一篇文章，我们将继续完善 `CGraphics` 的框架类，我们将为框架加上 `CShader` 类。这个类的目的是封装着色器的部分内容，我们在之前的三角形教程中已经用过了 `HLSL` ，所以这一篇不会很困难。

&emsp;&emsp;这个类完善后，我们的框架将会更新为：

![Shader](https://image.ibb.co/gPQ5Pc/image.png)

<!--more-->

&emsp;&emsp;首先我们来看看这个类的声明：

```c++
class CShader {
	struct MatrixBufferType {
		D3DXMATRIX world;
		D3DXMATRIX view;
		D3DXMATRIX projection;
	};

public:
	CShader();
	CShader(CShader& cs);
	~CShader();

	bool Initialize(ID3D11Device*, HWND , PCHAR , PCHAR);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX);

private:
	bool InitializeShader(ID3D11Device*, HWND, PCHAR, PCHAR);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, WCHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, D3DXMATRIX, D3DXMATRIX, D3DXMATRIX);
	void RenderShader(ID3D11DeviceContext*, int);

	ID3D11VertexShader* mVertexShader_;
	ID3D11PixelShader* mPixelShader_;
	ID3D11InputLayout* mLayout_;
	ID3D11Buffer* mMatrixBuffer_;
};
```

&emsp;&emsp;构造方法和析构方法就是初始化和最后的删除：

```c++
CShader::CShader() {
	mVertexShader_ = 0;
	mMatrixBuffer_ = 0;
	mLayout_ = 0;
	mPixelShader_ = 0;
}

void CShader::ShutdownShader() {
	if (mVertexShader_) {
		mVertexShader_->Release();
		mVertexShader_ = 0;
	}
	if (mMatrixBuffer_) {
		mMatrixBuffer_->Release();
		mMatrixBuffer_ = 0;
	}
	if (mPixelShader_) {
		mPixelShader_->Release();
		mPixelShader_ = 0;
	}
	if (mLayout_) {
		mLayout_->Release();
		mLayout_ = 0;
	}
}
```

&emsp;&emsp;在 `Initialize()` 方法中我们直接调用 `InitializeShader()` 方法，所以直接来看这个：

```c++
bool CShader::InitializeShader(ID3D11Device* device, HWND hwnd, PCHAR vsFilePath, PCHAR psFilePath) {
	HRESULT hr;
	ID3D10Blob* errorMessage;
	ID3D10Blob* vertexShaderBuffer;
	ID3D10Blob* pixelShaderBuffer;

	unsigned int numElements;

	hr = D3DX11CompileFromFile(vsFilePath, NULL, NULL, "Main", "vs_5_0", 
		D3D10_SHADER_ENABLE_STRICTNESS, 0, NULL,&vertexShaderBuffer , &errorMessage , nullptr);
	if (FAILED(hr)) {
		if (errorMessage) {
			OutputShaderErrorMessage(errorMessage, hwnd, vsFilePath);
		} else {
			return hr;
		}
	}
	hr = D3DX11CompileFromFile(psFilePath, NULL, NULL, "Main", "ps_5_0",
		D3D10_SHADER_ENABLE_STRICTNESS, 0, NULL, &pixelShaderBuffer, &errorMessage, nullptr);
	if (FAILED(hr)) {
		if (errorMessage) {
			OutputShaderErrorMessage(errorMessage, hwnd, vsFilePath);
		} else {
			return hr;
		}
	}

	hr = device->CreateVertexShader(vertexShaderBuffer->GetBufferPointer(), vertexShaderBuffer->GetBufferSize(), NULL, &mVertexShader_);
	if (FAILED(hr)) {
		return hr;
	}

	hr = device->CreatePixelShader(pixelShaderBuffer->GetBufferPointer(), pixelShaderBuffer->GetBufferSize(), NULL, &mPixelShader_);
	if (FAILED(hr)) {
		return hr;
	}

	D3D11_INPUT_ELEMENT_DESC polygonLayout[2] = {
		{"POSITION" , 0 , DXGI_FORMAT_R32G32B32A32_FLOAT , 0 , 0 , D3D11_INPUT_PER_VERTEX_DATA , 0} ,
		{"COLOR" , 0 , DXGI_FORMAT_R32G32B32A32_FLOAT , 0 , D3D11_APPEND_ALIGNED_ELEMENT , D3D11_INPUT_PER_VERTEX_DATA , 0}
	};

	numElements = sizeof(polygonLayout) / sizeof(polygonLayout[0]);

	hr = device->CreateInputLayout(polygonLayout, numElements, vertexShaderBuffer->GetBufferPointer(), vertexShaderBuffer->GetBufferSize(), &mLayout_);
	if (FAILED(hr)) {
		return false;
	}

	vertexShaderBuffer->Release();
	vertexShaderBuffer = 0;
	pixelShaderBuffer->Release();
	pixelShaderBuffer = 0;

	D3D11_BUFFER_DESC matrixBufferDesc;
	matrixBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
	matrixBufferDesc.ByteWidth = sizeof(MatrixBufferType);
	matrixBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
	matrixBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
	matrixBufferDesc.MiscFlags = 0;
	matrixBufferDesc.StructureByteStride = 0;

	hr = device->CreateBuffer(&matrixBufferDesc, NULL, &mMatrixBuffer_);
	if (FAILED(hr)) {
		return hr;
	}

	return true;
}
```

***&emsp;&emsp;&emsp;&emsp;（这段代码我们之前见过的，所以就不仔细讲了）***

&emsp;&emsp;`OutShaderErrorMessage()` 如下：

```c++
void CShader::OutputShaderErrorMessage(ID3D10Blob* errorMessage, HWND hwnd, PCHAR fileName) const {
	char* error;
	unsigned long bufferSize;
	std::ofstream out;

	error = static_cast<char*>(errorMessage->GetBufferPointer());
	bufferSize = errorMessage->GetBufferSize();
	out.open("log.txt");
	for (auto i = 0 ; i < bufferSize ; ++i) {
		out << error[i];
	}
	out.close();
	errorMessage->Release();
	errorMessage = 0;
	delete error;
}
```

&emsp;&emsp;在这里我们将编译出错的出错信息打印到 `log.txt` 里。

&emsp;&emsp;之后，我们使用 `CShader::SetShaderParameters` 方法来填充 `mMatrixBuffer` 。

```c++
bool CShader::SetShaderParameters(ID3D11DeviceContext* deviceContext, D3DXMATRIX world, D3DXMATRIX view, D3DXMATRIX projection) const {
	HRESULT hr;
	D3D11_MAPPED_SUBRESOURCE mappedSubResource;
	MatrixBufferType* dataPtr;

	D3DXMatrixTranspose(&world, &world);
	D3DXMatrixTranspose(&view, &view);
	D3DXMatrixTranspose(&projection, &projection);

	// 填充矩阵缓冲
	hr = deviceContext->Map(mMatrixBuffer_, 0, D3D11_MAP_WRITE_DISCARD, 0, &mappedSubResource);
	if(FAILED(hr)) {
		return hr;
	}
	dataPtr = static_cast<MatrixBufferType*>(mappedSubResource.pData);
	dataPtr->world = world;
	dataPtr->projection = projection;
	dataPtr->view = view;
	deviceContext->Unmap(mMatrixBuffer_, 0);

	deviceContext->VSSetConstantBuffers(0, 1, &mMatrixBuffer_);

	return true;
}
```

&emsp;&emsp;最后在 `RenderShader()` 里绘制：

```c++
void CShader::RenderShader(ID3D11DeviceContext* deviceContext, const int indexCount) const {
	deviceContext->IASetInputLayout(mLayout_);
	deviceContext->VSSetShader(mVertexShader_, nullptr, 0);
	deviceContext->PSSetShader(mPixelShader_, nullptr, 0);
	deviceContext->DrawIndexed(indexCount, 0, 0);
}
```

&emsp;&emsp;在我们对外的 `Render()` 接口里调用 `SetShaderParameters` 和 `RenderShader` ：

```c++
bool CShader::Render(ID3D11DeviceContext* deviceContext, const int index, const D3DXMATRIX world, const D3DXMATRIX view, const D3DXMATRIX projection) const {
	HRESULT hr;
	hr = SetShaderParameters(deviceContext, world, view, projection);
	if (FAILED(hr)) {
		return false;
	}
	RenderShader(deviceContext, index);
	return true;
}
```

&emsp;&emsp;这样我们的 `CShaper` 类就搞定了，这个类的内容我们之前的代码里都说过，所以这里比较简单。现在可以去更新上层的 `CGraphics` 类了。

&emsp;&emsp;仍然是在声明里创建指针对象，在初始化方法里初始化：

```c++
CShader *mShader_{};

bool CGraphics::Initialize(const int screenWidth, const int screenHeight, const HWND hwnd) {

	HRESULT hr;

	mShader_ = new CShader;
	if (!mShader_) {
		return false;
	}
	hr = mShader_->Initialize(mDirect3D_->GetDevice(), hwnd, "vs.vsh", "ps.psh");
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
}
```

&emsp;&emsp;在 `CGraphics::RenderGraphics` 方法里，我们调用这个渲染：

```c++
bool CGraphics::RenderGraphics(const float r , const float g , const float b , const float a) {
	mDirect3D_->BeginScene(r, g, b, a);
	mShader_->Render(mDirect3D_->GetDeviceContext() , 0 , NULL, NULL, NULL);
	mDirect3D_->EndScene();

	return true;
}
```

&emsp;&emsp;由于我们还没有完成 `CCamera` 和 `CModel` ，所以我们在 `CShader` 对象的接口里的矩阵参数暂时置空。