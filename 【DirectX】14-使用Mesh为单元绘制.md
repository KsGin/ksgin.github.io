---
title: 【DirectX】14-使用Mesh为单元绘制
date: 2018-03-28 19:34:12
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;上一篇文章中，我们使用了 Assimp 导入了一个模型，然后将其的顶点和索引全部提取出来并成功在屏幕上渲染。但是这样做有个很明显的问题，我们没法贴图。因为大部分模型的贴图都是针对单个 Mesh 来创建的，我们将所有顶点提出来一起渲染，自然是没有办法做这个了，所以最后我们得到的是一个纯色的蜘蛛。 

&emsp;&emsp;而这一篇中我们将针对 Mesh 进行渲染，并且对其贴图。

<!--more-->

&emsp;&emsp;首先我们会创建一个 ` Mesh` 类 ，这个类将实现对单个网格的渲染，来看看这个类的声明。

```c++
class Mesh {

	ID3D11Buffer *pIndexBufferObject;
	ID3D11Buffer *pVertexBufferObject;
	ID3D11VertexShader *pVertexShaderObject;
	ID3D11PixelShader *pPixelShaderObject;
	ID3D11InputLayout *pInputLayout;
	ID3D11SamplerState *pSamplerState;
	ID3D11ShaderResourceView *pShaderResourceView;

public:

	Vertex * vertices;
	UINT *indices;
	vector<std::string> texturePaths;

	UINT vertexCount;
	UINT indexCount;

	Mesh();

	Mesh(const UINT vertexCount, const UINT indexCount);

	HRESULT Init(ID3D11Device **pDevice , const char* vertexShaderPath , const char* pixelShaderPath);
	void Draw(ID3D11DeviceContext **pImmediateContext);
	void Release();
};
```

&emsp;&emsp;我们这个类设定的比较简单，并没有太多的功能，一个初始化，一个绘制，一个释放指针。来看看实现：

```c++
HRESULT Init(ID3D11Device **pDevice , const char* vertexShaderPath , const char* pixelShaderPath) {
	HRESULT hr;
	D3D11_BUFFER_DESC vertexBufferDesc;
	ZeroMemory(&vertexBufferDesc, sizeof(vertexBufferDesc));
	vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	vertexBufferDesc.ByteWidth = sizeof(Vertex) * vertexCount;
	vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	D3D11_SUBRESOURCE_DATA verticesSourceData;
	ZeroMemory(&verticesSourceData, sizeof(D3D11_SUBRESOURCE_DATA));
	verticesSourceData.pSysMem = vertices;

	D3D11_BUFFER_DESC indexDesc;
	ZeroMemory(&indexDesc, sizeof(indexDesc));
	indexDesc.Usage = D3D11_USAGE_IMMUTABLE;
	indexDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
	indexDesc.ByteWidth = sizeof(UINT) * indexCount;
	D3D11_SUBRESOURCE_DATA indexData;
	ZeroMemory(&indexData, sizeof(indexData));
	indexData.pSysMem = indices;
	hr = (*pDevice)->CreateBuffer(&vertexBufferDesc, &verticesSourceData, &pVertexBufferObject);
	if (FAILED(hr)) { return hr; }
	hr = (*pDevice)->CreateBuffer(&indexDesc, &indexData, &pIndexBufferObject);
	if (FAILED(hr)) { return hr; }

	ID3D10Blob* pErrorMessage = nullptr;
	ID3D10Blob* pVertexShaderBlob = nullptr;
	ID3D10Blob* pPixelShaderBlob = nullptr;

	hr = D3DX11CompileFromFile(vertexShaderPath, nullptr, nullptr, "main", "vs_5_0", D3DCOMPILER_STRIP_DEBUG_INFO, 0, nullptr, &pVertexShaderBlob, &pErrorMessage, nullptr);
	if (FAILED(hr)) {
		if (pErrorMessage) MessageBox(NULL, static_cast<CHAR*>(pErrorMessage->GetBufferPointer()), "Error", MB_OK);
		else MessageBox(NULL, "vertexShader File Not Found", "Error", MB_OK);
		return hr;
	}
	hr = D3DX11CompileFromFile(pixelShaderPath, nullptr, nullptr, "main", "ps_5_0", D3DCOMPILER_STRIP_DEBUG_INFO, 0, nullptr, &pPixelShaderBlob, &pErrorMessage, nullptr);
	if (FAILED(hr)) {
		if (pErrorMessage) MessageBox(NULL, static_cast<CHAR*>(pErrorMessage->GetBufferPointer()), "Error", MB_OK);
		else MessageBox(NULL, "pixelShader File Not Found", "Error", MB_OK);
		return hr;
	}

	hr = (*pDevice)->CreateVertexShader(pVertexShaderBlob->GetBufferPointer(), pVertexShaderBlob->GetBufferSize(), nullptr, &pVertexShaderObject);
	if (FAILED(hr)) {
		MessageBox(NULL, "ERROR::CreateVertexShader", "Error", MB_OK);
		return hr;
	}

	hr = (*pDevice)->CreatePixelShader(pPixelShaderBlob->GetBufferPointer(), pPixelShaderBlob->GetBufferSize(), nullptr, &pPixelShaderObject);
	if (FAILED(hr)) {
		MessageBox(NULL, "ERROR::CreatePixelShader", "Error", MB_OK);
		return hr;
	}

	D3D11_SAMPLER_DESC samplerDesc;
	samplerDesc.Filter = D3D11_FILTER_MIN_MAG_MIP_LINEAR;
	samplerDesc.AddressU = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.AddressV = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.AddressW = D3D11_TEXTURE_ADDRESS_WRAP;
	samplerDesc.MipLODBias = 0.0f;
	samplerDesc.MaxAnisotropy = 1;
	samplerDesc.ComparisonFunc = D3D11_COMPARISON_ALWAYS;
	samplerDesc.BorderColor[0] = 0;
	samplerDesc.BorderColor[1] = 0;
	samplerDesc.BorderColor[2] = 0;
	samplerDesc.BorderColor[3] = 0;
	samplerDesc.MinLOD = 0;
	samplerDesc.MaxLOD = D3D11_FLOAT32_MAX;

	hr = (*pDevice)->CreateSamplerState(&samplerDesc, &pSamplerState);
	if (FAILED(hr)) {
		MessageBox(nullptr, "ERROR::CreateSampler", "Error", MB_OK);
		return hr;
	}

	hr = D3DX11CreateShaderResourceViewFromFile(*pDevice, texturePaths[0].c_str() , nullptr, nullptr, &pShaderResourceView, nullptr);
	if (FAILED(hr)) {
		MessageBox(nullptr, "ERROR::CreateShaderResourceView", "Error", MB_OK);
		return hr;
	}

	D3D11_INPUT_ELEMENT_DESC layout[] = {
		{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D11_INPUT_PER_VERTEX_DATA, 0 } ,
		{ "TEXCOORD", 0, DXGI_FORMAT_R32G32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 } ,
		{ "NORMAL", 0, DXGI_FORMAT_R32G32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 }
	};
	hr = (*pDevice)->CreateInputLayout(layout, ARRAYSIZE(layout), pVertexShaderBlob->GetBufferPointer(), pVertexShaderBlob->GetBufferSize(), &pInputLayout);
	if (FAILED(hr)) {
		MessageBox(NULL, "ERROR::CreateInputLayout", "Error", MB_OK);
		return hr;
	}

	pVertexShaderBlob->Release();
	pPixelShaderBlob->Release();
	if (pErrorMessage) {
		pErrorMessage->Release();
	}
}
```

&emsp;&emsp;其实也就是将我们之前渲染的代码移进来了而已，其他的也差不多：

```c++
void Draw(ID3D11DeviceContext **pImmediateContext) {
	UINT stride = sizeof(Vertex);
	UINT offset = 0;
	(*pImmediateContext)->IASetVertexBuffers(0, 1, &pVertexBufferObject, &stride, &offset);
	(*pImmediateContext)->IASetIndexBuffer(pIndexBufferObject, DXGI_FORMAT_R32_UINT, 0);	
	(*pImmediateContext)->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	(*pImmediateContext)->PSSetShaderResources(0, 1, &pShaderResourceView);
	(*pImmediateContext)->PSSetSamplers(0, 1, &pSamplerState);
	(*pImmediateContext)->VSSetShader(pVertexShaderObject, nullptr, 0);
	(*pImmediateContext)->PSSetShader(pPixelShaderObject, nullptr, 0);
	(*pImmediateContext)->IASetInputLayout(pInputLayout);
	(*pImmediateContext)->DrawIndexed(indexCount, 0, 0);
}

void Release() {
	delete[] vertices;
	vertices = 0;
	delete[] indices;
	indices = 0;
	pIndexBufferObject->Release();
	pVertexBufferObject->Release();
}
```

&emsp;&emsp;现在我们看看对于模型的读取：

```c++
vector<Mesh> LoadModelUsingAssimp(const char* path) {
	Importer imp;
	const aiScene *scene = imp.ReadFile(path,
		aiProcess_GenNormals | aiProcess_Triangulate |
		aiProcess_FixInfacingNormals | aiProcess_FlipWindingOrder |
		aiProcess_GenUVCoords | aiProcess_FlipUVs);
	if (!scene) {
		MessageBox(nullptr, "ERROR::ReadOBJ", "ERROR", MB_OK);
		return vector<Mesh>();
	}

	vector<Mesh> meshes(scene->mNumMeshes);

	for (unsigned i = 0; i < scene->mNumMeshes ; ++i) {
		auto &mesh = meshes[i];
		const auto aiMesh = scene->mMeshes[i];
		mesh = Mesh(aiMesh->mNumVertices, aiMesh->mNumFaces * 3);

		for (size_t j = 0; j < aiMesh->mNumVertices; ++j) {
			mesh.vertices[j].pos = XMFLOAT3(aiMesh->mVertices[j].x, aiMesh->mVertices[j].y, aiMesh->mVertices[j].z);
			if (aiMesh->HasTextureCoords(0)) {
				mesh.vertices[j].texcoord = XMFLOAT2(aiMesh->mTextureCoords[0][j].x, aiMesh->mTextureCoords[0][j].y);
			}
			else {
				mesh.vertices[j].texcoord = XMFLOAT2(0.0f, 0.0f);
			}
			if (aiMesh->HasNormals()) {
				mesh.vertices[j].normal = XMFLOAT3(aiMesh->mNormals[j].x, aiMesh->mNormals[j].y, aiMesh->mNormals[j].z);
			}
			else {
				mesh.vertices[j].normal = XMFLOAT3(0.0f, 0.0f, -1.0f);
			}
		}

		for (size_t j = 0; j < aiMesh->mNumFaces; ++j) {
			for (size_t l = 0; l < aiMesh->mFaces[j].mNumIndices; ++l) {
				mesh.indices[j * aiMesh->mFaces[j].mNumIndices + l] = aiMesh->mFaces[j].mIndices[l];
			}
		}


		const auto material = scene->mMaterials[aiMesh->mMaterialIndex];
		mesh.texturePaths = vector<std::string>(material->GetTextureCount(aiTextureType_DIFFUSE) , std::string("./Resources/Low-Poly\ Spider/textures/"));
		for (auto& str : mesh.texturePaths) {
			aiString aiStr;
			material->GetTexture(aiTextureType_DIFFUSE, 0, &aiStr);
			str += aiStr.C_Str();
		}
	}

	return meshes;
}
```

&emsp;&emsp;在这里，我们并未将顶点和索引等数据扔到总数组里，而是创建了一个 `Mesh` 数组，为单个 `Mesh` 填充数据，包括纹理。（这里的纹理人家不是这么用的，只是我比较懒！！！）

&emsp;&emsp;OK，现在可以调用这些方法了，首先填充 `Mesh` 数组和初始化每个 `Mesh`：

```c++
vector<Mesh> meshes = LoadModelUsingAssimp("./Resources/Low-Poly\ Spider/spider.x");
for (auto& mesh : meshes) mesh.Init(&pDevice , "./vertexShader.hlsl" , "./pixelShader.hlsl");
```

&emsp;&emsp;在消息循环里调用 `Draw` 方法：

```c++
for (auto& mesh : meshes) mesh.Draw(&pImmediateContext);
```

&emsp;&emsp;最终效果如下：

![1](https://image.ibb.co/fLnGqS/image.png)

&emsp;&emsp;源代码地址：[**DX11Tutorial-RenderMesh**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-RenderMesh)

