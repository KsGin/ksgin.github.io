---
title: 【DirectX】22-多重纹理
date: 2018-04-06 11:21:55
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 17: Multitexturing and Texture Arrays](http://www.rastertek.com/dx11tut17.html)

#### 学习记录

&emsp;&emsp;这一篇相比之前的花了两天来写的视锥裁剪就显得简单的多了。我们在很久之前就学会了使用纹理，但每次都是单一的为一个模型贴上一张。而这篇文章中我们将对之前的代码修改以两张纹理贴图一个立方体。

<!--more-->

&emsp;&emsp;首先我们创建我们的纹理文件名数组：

```c++
const char * cubeTexs[] = {
	"texture1.jpg",
	"texture2.jpg"
};
const int nNumCubeTex = 2;
const char * fontTexs[] = {
	"font.dds"
};
const int nNumFontTex = 1;
```

&emsp;&emsp;之后更改我们的 `ID3D11ShaderResourceView` 为数组：

```c++
ID3D11ShaderResourceView		*pCubeShaderResourceView[nNumCubeTex];
ID3D11ShaderResourceView		*pFontShaderResourceView[nNumFontTex];
```

&emsp;&emsp;接下来更改 `InitShader` 方法，将其改为支持多重纹理（其实也就是改成循环创建）：

```c++
inline HRESULT InitShader(
	const char * vertexShaderName,
	const char * pixelShaderName,
	const char * texFileName[],
	const int nNumTex ,
	ID3D11Device *pDevice,
	ID3D11VertexShader **pVertexShader,
	ID3D11PixelShader **pPixelShader,
	ID3D11InputLayout **pInputLayout,
	ID3D11SamplerState **pSamplerState,
	ID3D11ShaderResourceView **pShaderResourceView) {

	.......

	for (int i = 0 ; i < nNumTex ; ++i) {
		hr = D3DX11CreateShaderResourceViewFromFile(pDevice, texFileName[i], nullptr, nullptr, &pShaderResourceView[i], nullptr);
		if (FAILED(hr)) {
			MessageBox(nullptr, "ERROR::CreateShaderResourceView", "Error", MB_OK);
			return hr;
		}
	}

	.......
        
	return S_OK;
}
```

&emsp;&emsp;最后，修改 `DrawModelIndex` 方法：

```c++
inline HRESULT DrawModelIndex(
	const UINT drawIndexCount,
	ID3D11Buffer *pVertexBufferObject,
	ID3D11Buffer *pIndexBufferObject,
	ID3D11Buffer *pMatrixDBuffer3D,
	ID3D11SamplerState *pSamplerState,
	const INT nNumTex ,
	ID3D11ShaderResourceView **pShaderResourceView,
	ID3D11VertexShader *pVertexShader,
	ID3D11PixelShader *pPixelShader,
	ID3D11InputLayout *pInputLayout,
	ID3D11DepthStencilView *pDepthStencilView,
	ID3D11DepthStencilState *pDepthStencilState,
	ID3D11RenderTargetView **pRenderTargetView,
	ID3D11DeviceContext **pImmediateContext) {

	UINT stride = sizeof(Vertex);
	UINT offset = 0;

	(*pImmediateContext)->OMSetDepthStencilState(pDepthStencilState, 1);
	(*pImmediateContext)->OMSetRenderTargets(1, pRenderTargetView, pDepthStencilView);

	(*pImmediateContext)->IASetVertexBuffers(0, 1, &pVertexBufferObject, &stride, &offset);
	(*pImmediateContext)->IASetIndexBuffer(pIndexBufferObject, DXGI_FORMAT_R32_UINT, 0);
	(*pImmediateContext)->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

	(*pImmediateContext)->VSSetConstantBuffers(0, 1, &pMatrixDBuffer3D);

	(*pImmediateContext)->VSSetShader(pVertexShader, 0, 0);
	(*pImmediateContext)->PSSetShader(pPixelShader, 0, 0);
	(*pImmediateContext)->IASetInputLayout(pInputLayout);
	(*pImmediateContext)->PSSetShaderResources(0, nNumTex, pShaderResourceView); //here
	(*pImmediateContext)->PSSetSamplers(0, 1, &pSamplerState);
	(*pImmediateContext)->DrawIndexed(drawIndexCount, 0, 0);
	return S_OK;
}
```

&emsp;&emsp;之前我们使用 `(*pImmediateContext)->PSSetShaderResources(0, nNumTex, pShaderResourceView);` 设置纹理资源，方法的第二个参数就是纹理个数，我们一直设置的 1 ，现在将其改为传进来的纹理个数参数就可以了。

&emsp;&emsp;最后来修改着色器，顶点着色器完全不用动，因为我们要用两张纹理贴图立方体模型，所以修改它的像素着色器就可以了。

```c++
Texture2D tex[2];
SamplerState samp;

struct pixelInputType {
	float4 pos : SV_POSITION;
	float2 texcoord : TEXCOORD0;
};

float4 main(pixelInputType input) : SV_TARGET{
	return tex[0].Sample(samp, input.texcoord) * 0.5f + tex[1].Sample(samp, input.texcoord) * 0.5;
} 
```

&emsp;&emsp;在着色器里，我们更改 tex 为数组变量，然后采样两张纹理并简单的平均混合。

&emsp;&emsp;最后我们去从网上再下载一张照片：

![2](https://image.ibb.co/kDfZrc/texture2.jpg)

&emsp;&emsp;加上我们之前的：

![1](https://image.ibb.co/noa0Wc/texture1.jpg)

&emsp;&emsp;将他们分别改名为 `texture2.jpg` 和 `texture1.jpg` 。就可以渲染了，效果如下：

![3](https://image.ibb.co/jBRXBc/image.png)

&emsp;&emsp;实际上，直接的进行平均后的混合并不是一个很好的办法，在后边我们会提到。也可以去网上了解一下 *伽马校正*（Gamma Correction）。

&emsp;&emsp;源代码：[**DX11Tutorial-MultitexturingAndTextureArrays**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-MultitexturingAndTextureArrays)