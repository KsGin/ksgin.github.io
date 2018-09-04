---
title: 【DirectX】46-鼠标拾取
date: 2018-04-26 19:31:15
tags:
	 - DirectX
	 - Computer graphics
---

#### 参考教程

[Tutorial 47: Picking](http://www.rastertek.com/dx11tut47.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们主要介绍鼠标拾取（Picking），即鼠标点击选中物体的过程。本篇代码基于阴影映射代码，主要在 `ApplicationClass` 类中实现。

<!--more-->

&emsp;&emsp;拾取过程一般分为两步，第一步为将鼠标从二维的屏幕坐标系转换为 3D 空间中的一个方向向量，然后使用此向量做与视野内可见 3D 物体的相交性测试，测试成功即可选中。

&emsp;&emsp;我们正常的渲染一个球体，如下：

![1](https://image.ibb.co/dHPpBx/image.png)

&emsp;&emsp;当我们获取到鼠标左键按下时，我们执行我们的变换以及相交测试，部分代码如下：

```c++
// Check if the left mouse button has been pressed.
if(m_Input->IsLeftMouseButtonDown() == true)
{
	// If they have clicked on the screen with the mouse then perform an intersection test.
	if(m_beginCheck == false)
	{
		m_beginCheck = true;
		m_Input->GetMouseLocation(mouseX, mouseY);
		TestIntersection(mouseX, mouseY);
	}
}

// Check if the left mouse button has been released.
if(m_Input->IsLeftMouseButtonDown() == false)
{
	m_beginCheck = false;
}
```

&emsp;&emsp;在 `TestIntersection` 方法中，我们使用与我们之前从 3D 变换相反的操作将鼠标从二维屏幕坐标系变换至三维空间，具体代码如下：

```c++
float pointX, pointY;
D3DXMATRIX projectionMatrix, viewMatrix, inverseViewMatrix, worldMatrix, translateMatrix, inverseWorldMatrix;
D3DXVECTOR3 direction, origin, rayOrigin, rayDirection;
bool intersect, result;


// Move the mouse cursor coordinates into the -1 to +1 range.
pointX = ((2.0f * (float)mouseX) / (float)m_screenWidth) - 1.0f;
pointY = (((2.0f * (float)mouseY) / (float)m_screenHeight) - 1.0f) * -1.0f;
	
// Adjust the points using the projection matrix to account for the aspect ratio of the viewport.
m_D3D->GetProjectionMatrix(projectionMatrix);
pointX = pointX / projectionMatrix._11;
pointY = pointY / projectionMatrix._22;

// Get the inverse of the view matrix.
m_Camera->GetViewMatrix(viewMatrix);
D3DXMatrixInverse(&inverseViewMatrix, NULL, &viewMatrix);

// Calculate the direction of the picking ray in view space.
direction.x = (pointX * inverseViewMatrix._11) + (pointY * inverseViewMatrix._21) + inverseViewMatrix._31;
direction.y = (pointX * inverseViewMatrix._12) + (pointY * inverseViewMatrix._22) + inverseViewMatrix._32;
direction.z = (pointX * inverseViewMatrix._13) + (pointY * inverseViewMatrix._23) + inverseViewMatrix._33;

// Get the origin of the picking ray which is the position of the camera.
origin = m_Camera->GetPosition();

// Get the world matrix and translate to the location of the sphere.
m_D3D->GetWorldMatrix(worldMatrix);
D3DXMatrixTranslation(&translateMatrix, -5.0f, 1.0f, 5.0f);
D3DXMatrixMultiply(&worldMatrix, &worldMatrix, &translateMatrix); 

// Now get the inverse of the translated world matrix.
D3DXMatrixInverse(&inverseWorldMatrix, NULL, &worldMatrix);

// Now transform the ray origin and the ray direction from view space to world space.
D3DXVec3TransformCoord(&rayOrigin, &origin, &inverseWorldMatrix);
D3DXVec3TransformNormal(&rayDirection, &direction, &inverseWorldMatrix);

// Normalize the ray direction.
D3DXVec3Normalize(&rayDirection, &rayDirection);
```

 &emsp;&emsp;最后，执行我们的射线球体相交判断方法：

```c++
RaySphereIntersect(rayOrigin, rayDirection, 1.0f);
```

&emsp;&emsp;方法如下：

```c++
bool ApplicationClass::RaySphereIntersect(D3DXVECTOR3 rayOrigin, D3DXVECTOR3 rayDirection, float radius)
{
	float a, b, c, discriminant;


	// Calculate the a, b, and c coefficients.
	a = (rayDirection.x * rayDirection.x) + (rayDirection.y * rayDirection.y) + (rayDirection.z * rayDirection.z);
	b = ((rayDirection.x * rayOrigin.x) + (rayDirection.y * rayOrigin.y) + (rayDirection.z * rayOrigin.z)) * 2.0f;
	c = ((rayOrigin.x * rayOrigin.x) + (rayOrigin.y * rayOrigin.y) + (rayOrigin.z * rayOrigin.z)) - (radius * radius);

	// Find the discriminant.
	discriminant = (b * b) - (4 * a * c);

	// if discriminant is negative the picking ray missed the sphere, otherwise it intersected the sphere.
	if (discriminant < 0.0f)
	{
		return false;
	}

	return true;
}
```

&emsp;&emsp;最终效果：

![2](https://image.ibb.co/isnHWx/image.png)

&emsp;&emsp;源代码：[DX11Tutorial-Picking](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Picking)