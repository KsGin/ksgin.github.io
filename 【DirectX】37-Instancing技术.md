---
title: 【DirectX】37-Instancing技术
date: 2018-04-20 18:08:14
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 37: Instancing](http://www.rastertek.com/dx11tut37.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们简单的介绍 DX 中 Instancing 绘制，代码依旧很简单。

<!--more-->

&emsp;&emsp;在之前介绍视锥裁剪的时候（[【DirectX】21-视锥裁剪](https://blog.ksgin.com/2018/04/06/【directx】21-视锥裁剪/)），我们在屏幕上绘制了大量的物体，使用一个循环然后调用 N 次 `DrawIndexed` 方法来绘制。即每一帧给 GPU 发送了 N 次基本相同的数据，这显然是一个不太理想的方法，在 OpenGL 系列学习笔记中，我们介绍过在 OpenGL 下的重复物体渲染（[【OpenGL】23-实例化](https://blog.ksgin.com/2018/03/13/【opengl】23-实例化/)） ，在 DX 中自然也有。

&emsp;&emsp;我们这次会使用 Instancing 绘制技术在屏幕上渲染多个 Square 。代码对 `ModelClass` 和 `TextureClass` 作以修改即可实现。

&emsp;&emsp;首先修改我们的 `ModelClass` 类，我们需要绘制多个基本相同物体的场景自然是在他们区别较小的情况下，例如相同的物体在不同的位置，我们使用一个结构体来保存多个实例间不同的属性，在这里只有 position 一个。

```c++
struct InstanceType 
{
	DirectX::XMFLOAT3 position;
};
```

&emsp;&emsp;同时，我们声明一个 Instance 缓冲以及一个 int 型的变量保存实例个数，同时我们暂时的去掉了 Index 相关变量，最终的声明如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: ModelClass
////////////////////////////////////////////////////////////////////////////////
class ModelClass
{
private:
	struct VertexType
	{
		DirectX::XMFLOAT3 position;
		DirectX::XMFLOAT2 texture;
		DirectX::XMFLOAT3 normal;
	};

	struct ModelType
	{
		float x, y, z;
		float tu, tv;
		float nx, ny, nz;
	};

	struct InstanceType 
	{
		DirectX::XMFLOAT3 position;
	};

public:
	ModelClass();
	ModelClass(const ModelClass&);
	~ModelClass();

	bool Initialize(ID3D11Device*, char*, char*);
	void Shutdown();
	void Render(ID3D11DeviceContext*);

	int GetVertexCount();
	int GetInstanceCount();
	ID3D11ShaderResourceView* GetTexture();

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	void RenderBuffers(ID3D11DeviceContext*);

	bool LoadTexture(ID3D11Device*, char*);
	void ReleaseTexture();

	bool LoadModel(char*);
	void ReleaseModel();

private:
	ID3D11Buffer * m_vertexBuffer, *m_instanceBuffer;
	int m_vertexCount, m_instanceCount;
	TextureClass* m_Texture;
	ModelType* m_model;
};
```

&emsp;&emsp;我们声明了两个方法用于获取单个实例的顶点个数以及实例个数，然后创建绑定缓冲的操作和其他一样，所以代码就不贴了。接下来看我们对于 `TextureShaderClass` 的改动，在这里我们需要修改的只有两样，一个是输入布局 `InputLayout` ，一个是绘制方式 `DrawInstanced`。 输入布局中我们新增了一项：

```c++
polygonLayout[2].SemanticName = "TEXCOORD";
polygonLayout[2].SemanticIndex = 1;
polygonLayout[2].Format = DXGI_FORMAT_R32G32_FLOAT;
polygonLayout[2].InputSlot = 1;
polygonLayout[2].AlignedByteOffset = 0;
polygonLayout[2].InputSlotClass = D3D11_INPUT_PER_INSTANCE_DATA;
polygonLayout[2].InstanceDataStepRate = 1;
```

&emsp;&emsp;最后，我们在 `RenderShader` 方法里修改绘制方式：

```c++
// Render the triangle.
deviceContext->DrawInstanced(vertexCount , instanceCount , 0 , 0);
```

&emsp;&emsp;这样，我们可以在屏幕上绘制多个 Square 。如下：

![4](https://image.ibb.co/jy5fP7/image.png)

&emsp;&emsp;这里我在 `ModelClass` 直接硬编码写入了实例的属性。

&emsp;&emsp;源代码：[DX11Tutorial-Instancing](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Instancing)