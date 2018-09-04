---
title: 【DirectX】9-代码框架-CModel和CCamera
date: 2018-03-25 12:36:52
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[教程 4: 缓冲区、着色器和 HLSL](http://www.rastertek.com/dx11s2tut04.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将更新我们框架代码中的 `CModel` 和 `CCamera` 。前者用来描绘我们世界坐标系空间中的某个物体，而后者则描述我们的摄像机，则我们的观察者。

&emsp;&emsp;之后我们的框架将会变为这样（是很丑，但我懒得再去画了！）：

![1](https://image.ibb.co/c6LWZn/image.png)

<!--more-->

&emsp;&emsp;先来看看 `CModel` 类，这个类描述了我们的一个实体模型，并提供了渲染接口，他的声明如下：

```c++
class CModel {
private:
	struct Vertex {
		DirectX::XMFLOAT3 position;
		DirectX::XMFLOAT3 color;
	};

public:
	CModel();
	CModel(CModel& cm);
	~CModel();

	bool Initialize(ID3D11Device*);
	void Shutdown();
	void Render(ID3D11DeviceContext*);
	int GetIndexCount();

private:
	bool InitializeModel(ID3D11Device*);
	void ShutdownModel();
	void RenderModel(ID3D11DeviceContext*);

	ID3D11Buffer *mVertexBuffer_{}, *mIndexBuffer_{};
	int mVertexCount{}, mIndexCount{};
};
```

&emsp;&emsp;这些都是比较简单能理解的，在初始化和销毁的函数里，我们对几个成员变量进行初始化和销毁操作。而公开接口里都是直接调用私有函数，如下：

```c++
CModel::CModel() : mVertexBuffer_(nullptr), mIndexBuffer_(nullptr), mVertexCount(0), mIndexCount(0) {

}

CModel::CModel(CModel& cm) {
	*this = cm;
}

CModel::~CModel() = default;

bool CModel::Initialize(ID3D11Device* device) {
	return InitializeModel(device);
}

void CModel::Shutdown() {
	ShutdownModel();
}

void CModel::Render(ID3D11DeviceContext* mDeviceContext) {
	RenderModel(mDeviceContext);
}

int CModel::GetIndexCount() const {
	return mIndexCount;
}
```

&emsp;&emsp;之后，在 `InitializeModel` 函数里，我们对我们的模型进行初始化，在我们之前的三角形那一篇里我们进行过这些操作（[【directx】3-d3d三角形](https://blog.ksgin.com/2018/03/18/【directx】3-d3d三角形/)），这里也基本一样（不同的是这里我们将使用索引缓冲）：

```c++
bool CModel::InitializeModel(ID3D11Device* device) {
	Vertex *vertices;
	unsigned long *indices;

	D3D11_BUFFER_DESC vertexBufferDesc, indexBufferDesc;
	D3D11_SUBRESOURCE_DATA vertexData, indexData;
	HRESULT hr;

	// 定义数组保存顶点和索引
	mVertexCount = 3;
	mIndexCount = 3;
	vertices = new Vertex[mVertexCount]{
		{DirectX::XMFLOAT3(-1.0f , -1.0f , 0.0f) , DirectX::XMFLOAT4(0.0f , 0.0f , 1.0f , 1.0f)} ,
		{DirectX::XMFLOAT3(0.0f , 1.0f , 0.0f) , DirectX::XMFLOAT4(0.0f , 0.0f , 1.0f , 1.0f)} ,
		{DirectX::XMFLOAT3(1.0f , -1.0f , 0.0f) , DirectX::XMFLOAT4(0.0f , 0.0f , 1.0f , 1.0f)} ,
	};
	if (!vertices) {
		return false;
	}
	indices = new unsigned long[mIndexCount] {
		0, 1, 2
	};
	if (!indices) {
		return false;
	}
	
	// 填写缓冲区描述并且创建缓冲
	ZeroMemory(&vertexBufferDesc, sizeof(vertexBufferDesc));
	vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	vertexBufferDesc.ByteWidth = sizeof(Vertex) * mVertexCount;
	vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	ZeroMemory(&indexBufferDesc, sizeof(indexBufferDesc));
	vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	vertexBufferDesc.ByteWidth = sizeof(unsigned long) * mIndexCount;
	vertexBufferDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;

	hr = device->CreateBuffer(&vertexBufferDesc, &vertexData, &mVertexBuffer_);
	if (FAILED(hr)) {
		return hr;
	}

	hr = device->CreateBuffer(&indexBufferDesc, &indexData, &mIndexBuffer_);
	if (FAILED(hr)) {
		return hr;
	}

	// 删除无用数据
	delete[] vertices;
	vertices = 0;
	delete[] indices;
	indices = 0;

	return true;
}
```

&emsp;&emsp;最后在 `Render` 方法里，我们对设备上下文对象进行配置。

```c++
void CModel::RenderModel(ID3D11DeviceContext* deviceContext) const {
	unsigned int stride , offset;
	stride = sizeof(Vertex);
	offset = 0;
	deviceContext->IASetVertexBuffers(0, 1, &mVertexBuffer_, &stride, &offset);
	deviceContext->IASetIndexBuffer(mIndexBuffer_, DXGI_FORMAT_R32_UINT, 0);
	deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
}
```

&emsp;&emsp;这样，我们就实现了 `CModel` 类，接下来看 `CCamera` 。

&emsp;&emsp;首先来看声明：

```c++
class CCamera {
public:
	CCamera();
	CCamera(CCamera& cc);

	void SetPosition(float, float, float);	// x y z
	void SetRotation(float, float, float);  // pitch yaw roll

	DirectX::XMFLOAT3 GetPosition() const;
	DirectX::XMFLOAT3 GetRotation() const;

	void Render();
	void GetViewMatrix(DirectX::XMMATRIX&) const;

private:
	DirectX::XMFLOAT3 mPosition{};
	DirectX::XMFLOAT3 mRotation{};
	DirectX::XMMATRIX mViewMatrix{};
};

```

&emsp;&emsp;在这个类里我们保存着摄像机的坐标和朝向以及一个观察矩阵。那几个 `Set` `Get` 函数都比较简单，只是赋值性操作而已。如下：

```c++
CCamera::CCamera() {
	mPosition = DirectX::XMFLOAT3();
	mRotation = DirectX::XMFLOAT3();
	mViewMatrix = DirectX::XMMATRIX();
}

CCamera::CCamera(CCamera& cc) {
	*this = cc;
}

void CCamera::SetPosition(const float x , const float y , const float z) {
	mPosition.x = x;
	mPosition.y = y;
	mPosition.z = z;
}

void CCamera::SetRotation(const float pitch, const float yaw, const float roll) {
	mRotation.x = pitch;
	mRotation.y = yaw;
	mRotation.z = roll;
}

DirectX::XMFLOAT3 CCamera::GetPosition() const {
	return mPosition;
}

DirectX::XMFLOAT3 CCamera::GetRotation() const {
	return mRotation;
}

void CCamera::GetViewMatrix(DirectX::XMMATRIX& view) const {
	view = mViewMatrix;
}
```

&emsp;&emsp;最后是 `Render` 函数，在这个函数里我们将使用 `mPosition` 变量和 `mRotation` 变量来更新观察矩阵，即用坐标和朝向来更新观察矩阵：

```c++
void CCamera::Render() {
	DirectX::XMVECTOR positionVec, lookAtVec, upVec;	
	DirectX::XMFLOAT3 positionFo3, lookAtFo3, upFo3;
	DirectX::XMMATRIX rotationMatrix;

	positionFo3 = mPosition;
	upFo3 = DirectX::XMFLOAT3(0.0f, 1.0f, 0.0f);
	lookAtFo3 = DirectX::XMFLOAT3(0.0f, 0.0f,1.0f);

	positionVec = DirectX::XMLoadFloat3(&positionFo3);
	upVec = DirectX::XMLoadFloat3(&upFo3);
	lookAtVec = DirectX::XMLoadFloat3(&lookAtFo3);

	// 使用 rotation 来创建 rotationMatrix 
	rotationMatrix = DirectX::XMMatrixRotationRollPitchYaw(mRotation.x * 0.0174532925f, mRotation.y * 0.0174532925f, mRotation.z * 0.0174532925f);

	// 使用 rotation 矩阵更新 lookAt 和 up 向量
	upVec = DirectX::XMVector2TransformCoord(upVec, rotationMatrix);
	lookAtVec = DirectX::XMVector2TransformCoord(lookAtVec, rotationMatrix);
	lookAtVec = DirectX::XMVectorAdd(positionVec, lookAtVec);

	// 使用 pos , lookat , up 来创建观察矩阵
	mViewMatrix = DirectX::XMMatrixLookAtLH(positionVec, lookAtVec, upVec);

}
```

&emsp;&emsp;这样，我们的摄像机类也结束了。现在可以往上一层的 `CGraphics` 类来添加内容了。首先在声明里增加这两个变量：

```c++
CModel *mModel_;
CCamera *mCamera_;
```

&emsp;&emsp;初始化什么的和其他一样：

```c++
bool CGraphics::Initialize(const int screenWidth, const int screenHeight, const HWND hwnd) {

	HRESULT hr;	
	char vsFileName[] = "vs.vs";
	char psFileName[] = "ps.ps";

	char* vsFile = vsFileName;
	char* psFile = psFileName;

	mDirect3D_ = new CDirect3D;
	if (!mDirect3D_) {
		return false;
	}
	hr = mDirect3D_->Initialize(screenWidth, screenHeight, VSYNC_ENABLED, hwnd, FULL_SCREEN, SCREEN_DEPTH, SCREEN_NEAR);
	if (FAILED(hr)) {
		return false;
	}
	
	mCamera_ = new CCamera;
	if (!mCamera_) {
		return false;
	}
	mCamera_->SetPosition(0.0f, 0.0f, 3.0f);

	mModel_ = new CModel;
	if (!mModel_) {
		return false;
	}
	mModel_->Initialize(mDirect3D_->GetDevice());

	mShader_ = new CShader;
	if (!mShader_) {
		return false;
	}
	hr = mShader_->Initialize(mDirect3D_->GetDevice(), hwnd, vsFile , psFile);
	if (FAILED(hr)) {
		return false;
	}

	return this->InitializeGraphics();
}
```

&emsp;&emsp;最后，是增加内容后的 `RenderGraphics` 函数。

```c++
bool CGraphics::RenderGraphics(const float r , const float g , const float b , const float a) {

	DirectX::XMMATRIX world, view, projection;

	mDirect3D_->BeginScene(r, g, b, a);
	mCamera_->Render();

	mDirect3D_->GetWorldMatrix(world);
	mCamera_->GetViewMatrix(view);
	mDirect3D_->GetProjectionMatrix(projection);

	mModel_->Render(mDirect3D_->GetDeviceContext());

	const bool result = mShader_->Render(mDirect3D_->GetDeviceContext() , mModel_->GetIndexCount() , world , view, projection);
	if (!result) {
		return false;
	}

	mDirect3D_->EndScene();

	return true;
}
```

&emsp;&emsp;需要注意的是，我们更新了数学库，使用 `DirectXMath` 中的 `XMFloat` 和 `XMMatrix` 等，所以之前的代码也需要去修改。