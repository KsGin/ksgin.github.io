---
title: 【DirectX】36-BillBoard技术
date: 2018-04-20 14:34:38
tags:	
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 34: Billboarding](http://www.rastertek.com/dx11tut34.html)

《计算机游戏程序设计》

#### 学习记录

&emsp;&emsp;这篇文章中，我们简单的介绍下 BillBoard 快速绘制技术。

<!--more-->

&emsp;&emsp;BillBoard 是一种常见的绘制技术，它允许我们使用较小的资源完成对 3D 物体的绘制，在《计算机游戏程序设计》中对 BillBoarding 技术的具体描述如下：

> &emsp;&emsp;“ BillBoard技术采用一个带有纹理的四边形，其纹理图像为该BillBoard所代表的物体的图像，即用带有该物体图像的长方形，代替生成该物体的图形画面。BillBoard放置于所代表物体的位置中心，并随相机的运动而变化，始终面对用户。将BillBoard技术与Alpha纹理和动画技术相结合，可以模拟很多自然现象，如树、烟、火、爆炸、云等。”——《计算机游戏程序设计》

&emsp;&emsp;如书中所述，BillBoard技术并不复杂。其核心内容在于无论何时二维图像都面向于观察者，我们可以用它来绘制树，火等等效果。

&emsp;&emsp;这篇文章的实现代码我们使用最简单的一个二维贴图，同时为了观察二维贴图方向的改变，我们使用带网格的地面作为参考。

&emsp;&emsp;首先我们渲染出我们所需要的地面和贴图正方形：

![1](https://image.ibb.co/kQLo97/image.png)

&emsp;&emsp;此时，我们处于这个 Square 的正前方，在我们看来它貌似可以当作一个正方体（因为后边我们是看不到的），如果我们移动摄像机位置，然后从另一个方向看它，就会变成这样：

![2](https://image.ibb.co/e9kPGn/image.png)

&emsp;&emsp;由于角度问题，我们已经可以很清楚的看出来这是一个二维图像，然而这显示不是我们想要的。如果我们需要使用二维图像来模拟三维物体的话，我们有一种办法是在同一处使用不同角度多渲染几张，这样不管我们从哪个角度来看都能看到物体。例如游戏中的草堆：

![3](https://image.ibb.co/fOTGU7/image.png)

&emsp;&emsp;*截图来自游戏《天涯明月刀》*

&emsp;&emsp;判断是否是这样渲染的方法是拉近之后旋转视角，你会清晰的看到二维图像在不同角度下的形态，当与二维图像一个平面时图像会消失。

&emsp;&emsp;另一种便是这篇文章的主题，BillBoard 绘制技术，它将一张二维图像根据我们摄像机的位置变换而旋转方向，保证永远面向着摄像机，这样不管在哪个角度看来都像是三维模型，我们修改图片面向后再看，如下：

![5](https://image.ibb.co/bE7Dbn/image.png)

&emsp;&emsp;*摄像机依旧在 Square 正前方*

&emsp;&emsp;它的计算也比较简单，我们直接在渲染前根据物体的位置和摄像机的位置，计算出角度，然后再对物体绕 Y 轴旋转，在 `GraphicsClass::Render()` 里直接实现：

```c++
bool GraphicsClass::Render()
{	
	DirectX::XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	bool result;
	XMFLOAT3 cameraPosition, modelPosition;
	double angle;
	float rotation;

	// Clear the buffers to begin the scene.
	m_D3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);

	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	m_Camera->GetViewMatrix(viewMatrix);
	m_D3D->GetProjectionMatrix(projectionMatrix);

	m_D3D->GetWorldMatrix(worldMatrix);
	worldMatrix *= XMMatrixScaling(5.0f, 1.0f, 5.0f);
	m_FloorModel->Render(m_D3D->GetDeviceContext());
	result = m_TextureShader->Render(
		m_D3D->GetDeviceContext(), 
		m_FloorModel->GetIndexCount(), 
		worldMatrix, viewMatrix, projectionMatrix, 
		m_FloorModel->GetTexture());
	if (!result) {
		return false;
	}


	m_Camera->GetPosition(cameraPosition);
	modelPosition = XMFLOAT3(0.0f, 1.5f, 1.0f);

	angle = atan2(modelPosition.x - cameraPosition.x, modelPosition.z - cameraPosition.z) * (180.0f / XM_PI);

	rotation = angle * 0.0174532925f;

	m_D3D->GetWorldMatrix(worldMatrix);
	worldMatrix *= XMMatrixTranslation(modelPosition.x, modelPosition.y, modelPosition.z);
	worldMatrix *= XMMatrixRotationY(rotation);
	m_BillBoardModel->Render(m_D3D->GetDeviceContext());
	result = m_TextureShader->Render(
		m_D3D->GetDeviceContext(), 
		m_BillBoardModel->GetIndexCount(), 
		worldMatrix, viewMatrix, projectionMatrix, 
		m_BillBoardModel->GetTexture());
	if (!result) {
		return false;
	}

	// Present the rendered scene to the screen.
	m_D3D->EndScene();

	return true;
}
```

&emsp;&emsp;源代码：[DX11Tutorial-BillBoarding](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-BillBoarding)

