---
title: 【DirectX】21-视锥裁剪
date: 2018-04-06 9:52:07
tags:
	- DirectX
	- Computer graphics
---

#### 教程地址

[Tutorial 16: Frustum Culling](http://www.rastertek.com/dx11tut16.html)

#### 学习记录

&emsp;&emsp;在透视投影（3D投影）中，我们所能看到的区域根本上来看很像是一个锥形。如下：

![1](https://image.ibb.co/iVJFwc/image.png)

&emsp;&emsp;如图，E 是我们的摄像机位置，摄像机所能观察到的角度则是我们的 FOV ，近截面和远截面则是我们最近和最远所能看到的平面。我们的透视投影矩阵便是拿这些数据来构造：

```c++
XMMatrixPerspectiveFovLH( fov  , width / height , near , far);
```

&emsp;&emsp;所以我们的可视区域则是这个锥形体的内部部分。在这个视锥之外的物体则不会被渲染，一般情况下我们将所有要渲染的顶点传入 GPU ，GPU将会检查是否可见，如果不可见将会被剔除。但是将这个任务交给 GPU 的话其效率确实很低。假设我们有渲染数千个物体，而只有十几个是可见的，那么我们需要将数千个物体的顶点信息传递给 GPU 并在渲染时判断，这无疑的十分浪费资源的。因此我们需要在传递数据之前进行判断这个模型是否在我们的视锥体之内，如果没有那么干脆不去发送它。

<!--more-->

&emsp;&emsp;首先我们在随机位置，绘制了 5000 个立方体：

```c++
MatrixXD matrix3d = {
	XMMatrixIdentity() ,
	XMMatrixLookAtLH(
		XMVectorSet(0.0f, 3.0f , -100.0f , 0.0f),
		XMVectorSet(0.0f, 0.0f ,  0.0f , 0.0f),
		XMVectorSet(0.0f, 1.0f ,  0.0f , 0.0f)
	),
	XMMatrixPerspectiveFovLH(90 ,static_cast<float>(width) / static_cast<float>(height) , 0.01f , 100.0f)
};

const UINT nNumWorldList = 5000;
auto *worldList = new XMMATRIX[nNumWorldList];
for (int i = 0 ; i < nNumWorldList ; i++) {
	worldList[i] = matrix3d.world;
	worldList[i] *= XMMatrixTranslation(
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldList / 10, 
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldList / 10, 
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldList / 10);
}
```

&emsp;&emsp;这五千个坐标由 `rand` 来生成，我们只是保证了它在一个限定的范围内 `-nNumWorldList / 10 ->nNumWorldList / 10 ` 。然后在循环里绘制它们：

```c++
for (int i = 0 ; i < nNumWorldList ; ++i) {
	matrix3d.world = worldList[i];
	Update3DModelPos(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
	
    DrawModelIndex(	// 绘制
	cubeVertexNum,
	pCubeVertexBufferObject,
	pCubeIndexBufferObject,
	pMatrixDBuffer3D,
	pCubeSamplerState,
	pCubeShaderResourceView,
	pCubeVertexShader,
	pCubePixelShader,
	pCubeInputLayout,
	pDepthStencilView,
	pEnableDepthStencilState,
	&pD3DRenderTargetView,
	&pD3DImmediateContext);
}
```

&emsp;&emsp;效果如下：

![1](https://image.ibb.co/cNDC9x/image.png)

&emsp;&emsp;屏幕中随机出现了大量的带纹理立方体，这些立方体是在我们视锥之内的，所以可以看到。GPU 替我们剔除了那些我们不会看到的内容。但是我们也可以看到，左上角的 FPS 已经只剩下了三十左右。仅仅是绘制五千个模型已经让我们的帧数接近了崩溃的边缘。如果是五万呢？

![2](https://image.ibb.co/eGyjNH/image.png)

&emsp;&emsp;由于模型的位置分布区域与我们的模型个数有关，而我们的可见区域一直那么大，所以现在在我们的可见区域的模型已经没几个了。然而我们的帧数已经变成个位数，这已经没法正常使用了，这还仅仅只是静态物体的绘制。所以在绘制大量物体的时候，对其进行裁剪就是必须的了。

&emsp;&emsp;关于视锥裁剪这是一个很大的话题，它可以只做最简单的判断（视锥裁剪的过程同样耗费资源），也可以去进行优化。在这里我们只做最简单的部分，相当于做个介绍。如果需要知道更多的视锥裁剪内容，请打开谷歌。

&emsp;&emsp;我们对立方体判断视锥的时候，是判断立方体的每一个点是否在视锥体之内，判断点的方法则是判断这个点是否与视锥体的六个面同向。（可以参考我们判断点在三角形内部的方法）

&emsp;&emsp;所以可以先计算视锥体的六个面：

```c++
inline HRESULT InitFrustumPlane(const FLOAT depth, const MatrixXD &matrix3D , XMVECTOR *&viewPlane) {
	const XMMATRIX vip = matrix3D.view * matrix3D.projection;
	if (viewPlane) {
		delete[] viewPlane;
		viewPlane = 0;
	}
	viewPlane = new XMVECTOR[6];
	//near
	viewPlane[0].m128_f32[0] = vip.r[0].m128_f32[3] + vip.r[0].m128_f32[2];
	viewPlane[0].m128_f32[1] = vip.r[1].m128_f32[3] + vip.r[1].m128_f32[2];
	viewPlane[0].m128_f32[2] = vip.r[2].m128_f32[3] + vip.r[2].m128_f32[2];
	viewPlane[0].m128_f32[3] = vip.r[3].m128_f32[3] + vip.r[3].m128_f32[2];
	viewPlane[0] = XMPlaneNormalize(viewPlane[0]);
	//far
	viewPlane[1].m128_f32[0] = vip.r[0].m128_f32[3] - vip.r[0].m128_f32[2];
	viewPlane[1].m128_f32[1] = vip.r[1].m128_f32[3] - vip.r[1].m128_f32[2];
	viewPlane[1].m128_f32[2] = vip.r[2].m128_f32[3] - vip.r[2].m128_f32[2];
	viewPlane[1].m128_f32[3] = vip.r[3].m128_f32[3] - vip.r[3].m128_f32[2];
	viewPlane[1] = XMPlaneNormalize(viewPlane[1]);
	//left
	viewPlane[2].m128_f32[0] = vip.r[0].m128_f32[3] + vip.r[0].m128_f32[0];
	viewPlane[2].m128_f32[1] = vip.r[1].m128_f32[3] + vip.r[1].m128_f32[0];
	viewPlane[2].m128_f32[2] = vip.r[2].m128_f32[3] + vip.r[2].m128_f32[0];
	viewPlane[2].m128_f32[3] = vip.r[3].m128_f32[3] + vip.r[3].m128_f32[0];
	viewPlane[2] = XMPlaneNormalize(viewPlane[2]);
	//right																   
	viewPlane[3].m128_f32[0] = vip.r[0].m128_f32[3] - vip.r[0].m128_f32[0];
	viewPlane[3].m128_f32[1] = vip.r[1].m128_f32[3] - vip.r[1].m128_f32[0];
	viewPlane[3].m128_f32[2] = vip.r[2].m128_f32[3] - vip.r[2].m128_f32[0];
	viewPlane[3].m128_f32[3] = vip.r[3].m128_f32[3] - vip.r[3].m128_f32[0];
	viewPlane[3] = XMPlaneNormalize(viewPlane[3]);
	//top
	viewPlane[4].m128_f32[0] = vip.r[0].m128_f32[3] - vip.r[0].m128_f32[1];
	viewPlane[4].m128_f32[1] = vip.r[1].m128_f32[3] - vip.r[1].m128_f32[1];
	viewPlane[4].m128_f32[2] = vip.r[2].m128_f32[3] - vip.r[2].m128_f32[1];
	viewPlane[4].m128_f32[3] = vip.r[3].m128_f32[3] - vip.r[3].m128_f32[1];
	viewPlane[4] = XMPlaneNormalize(viewPlane[4]);
	//bottom
	viewPlane[5].m128_f32[0] = vip.r[0].m128_f32[3] + vip.r[0].m128_f32[1];
	viewPlane[5].m128_f32[1] = vip.r[1].m128_f32[3] + vip.r[1].m128_f32[1];
	viewPlane[5].m128_f32[2] = vip.r[2].m128_f32[3] + vip.r[2].m128_f32[1];
	viewPlane[5].m128_f32[3] = vip.r[3].m128_f32[3] + vip.r[3].m128_f32[1];
	viewPlane[5] = XMPlaneNormalize(viewPlane[5]);

	return S_OK;
}
```

&emsp;&emsp;当我们需要在 `world space` 进行裁剪的时候，我们需要的是 `vp（view * projection）` 矩阵（请注意这个顺序不能反）。这里有关于计算面的一个问题回答：[知乎传送门](https://www.zhihu.com/question/46377273)

&emsp;&emsp;有了视锥体的六个面之后，可以进行判断了。

```c++
inline bool CheckCube(XMVECTOR *&viewPlane , const FLOAT centerX , const FLOAT centerY , const FLOAT centerZ , const FLOAT R) {
	for (int i = 0 ; i < 6 ; ++i) {
		float dot;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX + R , centerY + R , centerZ + R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX + R , centerY + R , centerZ - R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX + R , centerY - R , centerZ + R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX + R , centerY - R , centerZ - R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX - R , centerY + R , centerZ + R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX - R , centerY + R , centerZ - R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX - R , centerY - R , centerZ + R , 0.0f)));if (dot > 0.0f) continue;
		XMStoreFloat(&dot,XMPlaneDotCoord(viewPlane[i],XMVectorSet(centerX - R , centerY - R , centerZ - R , 0.0f)));if (dot > 0.0f) continue;
		return false;
	}
	return true;
}
```

&emsp;&emsp;在这里我们并没有直接传入立方体的六个点，而是传入了中心点和它的半径（如果是长方体则需要中心点和长，宽，高）。

&emsp;&emsp;有了这两个方法，我们便可以在主函数里调用以在 `Draw` 之前判断物体是否在视锥体之内来对物体进行相应剔除。同时我们增加了一个计数显示，使得当前视锥体之内立方体数量可以在屏幕上显示出来。如下：

```c++
const FLOAT snear = 0.01f;
	const FLOAT sfar = 100.0f;
	MatrixXD matrix3d = {
	XMMatrixIdentity() ,
	XMMatrixLookAtLH(
		XMVectorSet(0.0f, 0.0f , -100.0f , 0.0f),
		XMVectorSet(0.0f, 0.0f ,  0.0f , 0.0f),
		XMVectorSet(0.0f, 1.0f ,  0.0f , 0.0f)
	),
	XMMatrixPerspectiveFovLH(90 ,static_cast<float>(width) / static_cast<float>(height) , snear , sfar)
};

XMVECTOR *viewPlanes = nullptr;
InitFrustumPlane(sfar , matrix3d, viewPlanes);

const UINT nNumWorldTranslateList = 5000;
auto *worldTranslateList = new XMFLOAT3[nNumWorldTranslateList];
auto *worldList = new XMMATRIX[nNumWorldTranslateList];
for (int i = 0 ; i < nNumWorldTranslateList ; i++) {
	worldTranslateList[i] = XMFLOAT3(
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldTranslateList / 10, 
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldTranslateList / 10, 
		(static_cast<float>(rand()) - static_cast<float>(rand())) / RAND_MAX * nNumWorldTranslateList / 10);	
	worldList[i] = XMMatrixIdentity() * XMMatrixTranslation(worldTranslateList[i].x, worldTranslateList[i].y, worldTranslateList[i].z);
}
......
cube = 0;
for (int i = 0; i < nNumWorldTranslateList; ++i) {
	worldTranslateList[i] = XMFLOAT3(worldTranslateList[i].x , worldTranslateList[i].y , worldTranslateList[i].z + 0.03f);
	worldList[i] *= XMMatrixTranslation(0.0f , 0.0f , 0.03f);
	if (CheckCube(viewPlanes, worldTranslateList[i].x, worldTranslateList[i].y, worldTranslateList[i].z, 0.5f)) {				
		matrix3d.world = worldList[i];
		Update3DModelWorld(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
		DrawModelIndex(
			cubeVertexNum,
			pCubeVertexBufferObject,
			pCubeIndexBufferObject,
			pMatrixDBuffer3D,
			pCubeSamplerState,
			pCubeShaderResourceView,
			pCubeVertexShader,
			pCubePixelShader,
			pCubeInputLayout,
			pDepthStencilView,
			pEnableDepthStencilState,
			&pD3DRenderTargetView,
			&pD3DImmediateContext);
		++cube;
	}
}
```

&emsp;&emsp;效果如下：

![1](https://image.ibb.co/kztZPx/image.png)

&emsp;&emsp;可以看到，左上的计数那里显示我们有一百多个 cube 被渲染，而屏幕上呈现的部分貌似差不多。。。如果你闲的无聊可以去数数。

&emsp;&emsp;源代码：[**DX11Tutorial-FrustumCulling**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-FrustumCulling)







