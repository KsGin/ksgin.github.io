---
title: 【DirectX】28-透明混合
date: 2018-04-12 10:56:09
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 26: Transparency](http://www.rastertek.com/dx11tut26.html)

[【D3D11游戏编程】学习笔记十五：混合（Blending）](https://blog.csdn.net/bonchoix/article/details/8453943)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将简单的介绍在 DX 中实现透明度混合的方法。开启混合时，当像素片段通过深度测试后，不会直接覆盖掉原缓冲中的颜色，而是使用透明度因子进行插值，结果作为当前点的颜色，这就是混合。

<!--more-->

&emsp;&emsp;要使用混合的话，我们需要设置 `Blend State` ，和其他的一样我们需要创建相应指针对象然后进行设置，首先声明一个 `ID3D11BlendState` 指针：

```c++
ID3D11BlendState	*pD3DBlendState = nullptr;
```

&emsp;&emsp;在 `InitWindowAndD3D` 方法里我们新加入一个需要初始化的指针：

```c++
inline HRESULT InitWindowAndD3D(
	const HINSTANCE hInstance,
	const char * windowName,
	HWND * hWnd,
	IDXGISwapChain **pSwapChain,
	ID3D11BlendState **pD3DBlendState,	// 这里
	ID3D11RenderTargetView **pRenderTargetView,
	ID3D11Device **pDevice,
	ID3D11DeviceContext **pImmediateContext) {
    ..............
}
```

&emsp;&emsp;然后进行初始化：

```c++
D3D11_BLEND_DESC dd;
ZeroMemory(&dd, sizeof(dd));
dd.RenderTarget[0].BlendEnable = true;
dd.RenderTarget[0].SrcBlend = D3D11_BLEND_SRC_ALPHA;
dd.RenderTarget[0].DestBlend = D3D11_BLEND_INV_SRC_ALPHA;
dd.RenderTarget[0].BlendOp = D3D11_BLEND_OP_ADD;
dd.RenderTarget[0].SrcBlendAlpha = D3D11_BLEND_ONE;
dd.RenderTarget[0].DestBlendAlpha = D3D11_BLEND_ZERO;
dd.RenderTarget[0].BlendOpAlpha = D3D11_BLEND_OP_ADD;
dd.RenderTarget[0].RenderTargetWriteMask = 0x0f;

hr = (*pDevice)->CreateBlendState(&dd, pD3DBlendState);
if (FAILED(hr)) {
	MessageBox(nullptr, "ERROR::CreateBlendState_Error", "Error", MB_OK);
	return hr;
}
```

&emsp;&emsp;那些赋值的含义如下：

> ​	D3D11_BLEND_ZERO：此外针对颜色混合，F为（0，0，0），针对alpha值，F为0。
>
> ​	D3D11_BLEND_ONE：针对颜色混合为（1，1，1），针对alpha值为1；
>
> ​	D3D11_BLEND_SRC_COLOR：针对颜色混合为（Rs，Gs, Bs），针对alpha值为As；
>
> ​	D3D11_BLEND_INV_SRC_COLOR：针对颜色混合为（1-Rs，1-Gs，1-Bs），针对alpha值为1-As；
>
> ​	D3D11_BLEND_SRC_ALPHA：针对颜色混合为（As,As,As），针对alpha值为As；
>
> ​	D3D11_BLEND_INV_SRC_ALPHA：针对颜色混合为（1-As,1-As,1-As），针对alpha值为1-As；
>
> ​	D3D11_BLEND_DEST_COLOR：针对颜色混合为（Rd,Gd,Bd），针对alpha值为Ad；
>
> ​	D3D11_BLEND_INV_DESC_COLOR：针对颜色混合为（1-Rd,1-Gd,1-Bd），针对alpha值为1-Ad；
>
> ​	D3D11_BLEND_DEST_ALPHA：针对颜色混合为（Ad,Ad,Ad），针对alpha值为Ad；
>
> ​	D3D11_BLEND_INV_DEST_ALPHA：针对颜色混合为(1-Rd,1-Rd,1-Rd），针对alpha值为1-Ad；
>
> ​	D3D11_BLEND_BLEND_FACTOR：此时的混合因子为程序员指定的颜色值（R,G,B,A)，针对颜色混合为(R,G,B)，针对alpha值为A。该颜色值通过函数ID3D11DeviceContext::OMSetBlendState来指定；
>
> ​	D3D11_BLEND_INV_BLEND_FACTOR：同上，为程序员指定颜色值，针对颜色混合为(1-R,1-G,1-B)，针对alpha值为1-A。

&emsp;&emsp;之后，我们可以开始渲染了。这次会渲染两个变形后的 cube ，一个在前开启了混合，一个在后未开启混合，并且两个有一定区域的重叠。

&emsp;&emsp;我们定义两个 cube 的世界坐标矩阵：

```c++
XMMATRIX worldMatrix[2] = {
	XMMatrixIdentity() * XMMatrixScaling(1.5 , 1.5 , 0.2),
	XMMatrixIdentity() * XMMatrixScaling(1.5 , 1.5 , 0.2) * XMMatrixTranslation(1.0 , 1.0 , 1.0)
};
```

&emsp;&emsp;**请注意，当我们使用混合渲染的时候，渲染顺序必须是从后往前，即深度从大到小，否则无法实现混合效果。因为我们如果先绘制深度小的部分，后绘制的部分在深度测试的时候就被丢弃了。**

&emsp;&emsp;现在，渲染两个 cube ：

```c++
matrix3d.world  = worldMatrix[1];
Update3DModelWorld(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
DrawModelIndex(
	cubeVertexNum,
	pCubeVertexBufferObject,
	pCubeIndexBufferObject,
	pMatrixDBuffer3D,
	pCubeSamplerState,
	nNumCubeTex,
	pCubeShaderResourceView,
	pCubeVertexShader,
	pCubePixelShader,
	pCubeInputLayout,
	pDepthStencilView,
	pEnableDepthStencilState,
	&pRenderTargetViewT,
	&pD3DImmediateContext);

pD3DImmediateContext->OMSetBlendState(pD3DBlendState, bkgc, D3D11_DEFAULT_SAMPLE_MASK);
matrix3d.world  = worldMatrix[0];
Update3DModelWorld(matrix3d, pMatrixDBuffer3D, &pD3DImmediateContext);
DrawModelIndex(
	cubeVertexNum,
	pCubeVertexBufferObject,
	pCubeIndexBufferObject,
	pMatrixDBuffer3D,
	pCubeSamplerState,
	nNumCubeTex,
	pCubeShaderResourceView,
	pCubeVertexShader,
	pCubePixelShader,
	pCubeInputLayout,
	pDepthStencilView,
	pEnableDepthStencilState,
	&pRenderTargetViewT,
	&pD3DImmediateContext);
pD3DImmediateContext->OMSetBlendState(0, bkgc, D3D11_DEFAULT_SAMPLE_MASK);
```

&emsp;&emsp;可以看到，在渲染第二个 cube 之前，我们开启了 blend ，渲染完毕后又直接关闭掉了它。

&emsp;&emsp;最后，我们在片段着色器里直接设置 cube 颜色的透明度为 0.5 ：

```c++
float4 main(pixelInputType input) : SV_TARGET {
	float4 color = tex[0].Sample(samp, input.texcoord);
	color.a = 0.5f;
	return color;
} 
```

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/njVzjx/image.png)

&emsp;&emsp;当渲染靠后的 cube 的时候，我们并未开启 blend ，所以它是完全可见的，而较近的那个则变成了透明状。

&emsp;&emsp;源代码：[DX11Tutorial-Transparency](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Transparency)

