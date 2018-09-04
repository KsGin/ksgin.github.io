---
title: 【DirectX】13-Assimp库
date: 2018-03-27 16:54:08
tags:
	- DirectX
	- Computer graphics
---

&emsp;&emsp;这篇文章中，我们将使用 Assimp 库导入一个 3D 模型。

&emsp;&emsp;在以往的代码中，我们使用硬编码来定义顶点坐标，可以很简单的输出一些顶点较少的模型，但是当某个模型比较复杂的时候，这样做就显得不太现实了。所以一般情况下我们是在 3D 建模工具下建模，然后导出为模型文件，通过代码导入。这篇文章便来介绍 Assimp 在 DX 下的使用。

<!--more-->

&emsp;&emsp;关于 Mesh 的介绍可以看这里：[杂杂碎碎的常识第二弹-网格与模型](https://blog.ksgin.com/2018/03/23/%E6%9D%82%E6%9D%82%E7%A2%8E%E7%A2%8E%E7%9A%84%E5%B8%B8%E8%AF%86%E7%AC%AC%E4%BA%8C%E5%BC%B9-%E7%BD%91%E6%A0%BC%E4%B8%8E%E6%A8%A1%E5%9E%8B/)

&emsp;&emsp;关于 Assimp 的介绍可以看这里：[【OpenGL】12-模型与Assimp库](https://blog.ksgin.com/2018/03/04/%E3%80%90opengl%E3%80%9112-%E6%A8%A1%E5%9E%8B%E4%B8%8Eassimp%E5%BA%93/)

&emsp;&emsp;在这次的代码中，我们并不会去和 OpenGL 中一样建立类来对网格等数据结构进行组织，以网格为单元来渲染。我们简单的使用 Assimp 读取模型文件后，将其顶点和索引提取出来放在数组里，使用我们之前比较熟悉的代码来渲染。

&emsp;&emsp;首先定义一个 Vertex 数组（和我们之前的一样）。

```c++
struct Vertex
{
	XMFLOAT3 pos;
	XMFLOAT2 texcoord;
	XMFLOAT3 normal;
};
```

&emsp;&emsp;现在，我们使用 Assimp 库来读取模型：

```c++
Importer imp;
const aiScene *scene = imp.ReadFile("model.x", aiProcess_GenNormals | aiProcess_Triangulate | aiProcess_FixInfacingNormals | aiProcess_FlipWindingOrder );
if (!scene) {
	MessageBox(nullptr, "ERROR::ReadOBJ", "ERROR", MB_OK);
	return -1;
}
```

&emsp;&emsp;如果你不太清楚这个 aiScene 对象的结构的话，可以去看上边关于 Assimp 介绍的链接。

&emsp;&emsp;接下来，我们定义两个 `int` 数据来表示顶点和索引的个数：

```c++
int vertexCount = 0;
int indexCount = 0;
```

&emsp;&emsp;下来是对 scene 对象的第一次遍历，用来获取 scene 中所有 Mesh 对象的顶点和索引之和：

```c++
for (unsigned i = 0; i < scene->mNumMeshes; ++i) {
	vertexCount += scene->mMeshes[i]->mNumVertices;
	indexCount += scene->mMeshes[i]->mNumFaces * scene->mMeshes[i]->mFaces->mNumIndices;
}
```

&emsp;&emsp;现在我们可以创建顶点和索引数组了：

```c++
auto *vertices = new Vertex[vertexCount];
auto *indices = new UINT[indexCount];
```

&emsp;&emsp;接下来，我们第二次遍历 scene 对象，为我们的顶点数组和索引数组填充数据：

```c++
UINT idx1 = 0, idx2 = 0;
for (unsigned i = 0; i < scene->mNumMeshes; ++i) {
	const auto mesh = scene->mMeshes[i];
	for (unsigned j = 0; j < mesh->mNumVertices; ++j) {
		vertices[idx1].pos = XMFLOAT3(mesh->mVertices[j].x, mesh->mVertices[j].y, mesh->mVertices[j].z);
		if (mesh->HasNormals()) {
			vertices[idx1].normal = XMFLOAT3(mesh->mNormals[j].x, mesh->mNormals[j].y, mesh->mNormals[j].z);
		} else {
			vertices[idx1].normal = XMFLOAT3(0.0f, 0.0f, 1.0f);
		}
		if (mesh->HasTextureCoords(j)) {
			vertices[idx1].texcoord = XMFLOAT2(mesh->mTextureCoords[j]->x, mesh->mTextureCoords[j]->y);
		} else {
			vertices[idx1].texcoord = XMFLOAT2(0.0f, 0.0f);
		}
		++idx1;
	}

	for (unsigned j = 0; j < mesh->mNumFaces; ++j) {
		assert(mesh->mFaces[j].mNumIndices == 3);
		indices[idx2++] = mesh->mFaces[j].mIndices[0];
		indices[idx2++] = mesh->mFaces[j].mIndices[1];
		indices[idx2++] = mesh->mFaces[j].mIndices[2];
	}
}
```

&emsp;&emsp;上边这一段中，我们首先遍历所有 Mesh ，然后对每个 Mesh 执行以下操作：

+ 获取 Mesh 的顶点数组，并将其保存到我们的顶点数组中。
+ 获取 Mesh 的 Face 数组，并将构成其的索引保存到我们的索引数组中。

&emsp;&emsp;之后的创建缓冲等操作和之前就是完全一样了。我在我的代码中导入了一个蜘蛛的模型，下来看看效果：

![2](https://image.ibb.co/b75mpn/image.png)

&emsp;&emsp;貌似还不错吧。

&emsp;&emsp;程序源码地址：[DX11Tutorial-UsingAssimp](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-UsingAssimp)

