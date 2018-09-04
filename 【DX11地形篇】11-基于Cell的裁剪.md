---
title: 【DX11地形篇】11-基于Cell的裁剪
date: 2018-05-09 13:02:15
tags:	
	- DirectX
	- Terrain
	- GameDev
	- Computer graphics
---

#### 参考教程

[Tutorial 10: Terrain Cell Culling ](http://www.rastertek.com/dx11ter10.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们介绍基于 Cell 的裁剪，本篇会使用到我们之前介绍的裁剪技术：[【DirectX】21-视锥裁剪](https://blog.ksgin.com/2018/04/06/【directx】21-视锥裁剪/) 。代码修改幅度并不大。

<!--more-->

&emsp;&emsp;首先来看我们的裁剪类，我们既然使用立方体 Cell ，那么自然使用包围盒裁剪，裁剪类声明如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: FrustumClass
////////////////////////////////////////////////////////////////////////////////
class FrustumClass
{
public:
	FrustumClass();
	FrustumClass(const FrustumClass&);
	~FrustumClass();

	void Initialize(float);

	void ConstructFrustum(XMMATRIX, XMMATRIX);

	bool CheckPoint(float, float, float);
	bool CheckCube(float, float, float, float);
	bool CheckSphere(float, float, float, float);
	bool CheckRectangle(float, float, float, float, float, float);
	bool CheckRectangle2(float, float, float, float, float, float);

private:
	float m_screenDepth;
	float m_planes[6][4];
};
```

&emsp;&emsp;在 `TerrainClass` 中我们新增多个变量以及方法，主要是增加了对裁剪/渲染的数量进行封装以及裁剪的执行，如下：

```c++
////////////////////////////////////////////////////////////////////////////////
// Class name: TerrainClass
////////////////////////////////////////////////////////////////////////////////
class TerrainClass
{
private:
	struct VertexType
	{
		XMFLOAT3 position;
		XMFLOAT2 texcoord;
		XMFLOAT3 normal;
		XMFLOAT3 tangent;
		XMFLOAT3 binormal;
		XMFLOAT3 color;
	};

	struct HeightMapType {
		float x, y, z;
		float nx, ny, nz;
		float r, g, b;
	};

	struct ModelType {
		float x, y, z;
		float tu, tv;
		float nx, ny, nz;
		float tx, ty, tz;
		float bx, by, bz;
		float r, g, b;
	};

	struct VectorType {
		float x, y, z;
	};

	struct TempVertexType
	{
		float x, y, z;
		float tu, tv;
		float nx, ny, nz;
	};

public:
	TerrainClass();
	TerrainClass(const TerrainClass&);
	~TerrainClass();

	bool Initialize(ID3D11Device* , char*);
	void Shutdown();
	bool Render(ID3D11DeviceContext*);

	int GetIndexCount();

	void Frame();
	bool RenderCell(ID3D11DeviceContext*, int, FrustumClass*);
	void RenderCellLines(ID3D11DeviceContext*, int);

	int GetCellIndexCount(int);
	int GetCellLinesIndexCount(int);
	int GetCellCount();

	int GetRenderCount();
	int GetCellsDrawn();
	int GetCellsCulled();

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	void RenderBuffers(ID3D11DeviceContext*);

	void CalculateTerrainVectors();
	void CalculateTangentBinormal(TempVertexType, TempVertexType, TempVertexType, VectorType&, VectorType&);

	bool LoadColorMap();
	bool CalculateNormals();
	bool LoadSetupFile(CHAR*);
	bool LoadBitmapHeightMap();
	bool LoadRawHeightMap();
	void ShutdownHeightMap();
	void SetTerrainCoordinates();
	bool BuildTerrainModel();
	void ShutdownTerrainModel();

	bool LoadTerrainCells(ID3D11Device*);
	void ShutdownTerrainCells();

private:
	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
	int m_vertexCount, m_indexCount;

	int m_terrainHeight , m_terrainWidth;
	float m_heightScale;
	LPSTR m_terrainFilename;
	LPSTR m_colorMapFilename;
	HeightMapType* m_heightMap;
	ModelType* m_terrainModel;

	TerrainCellClass* m_TerrainCells;
	int m_cellCount, m_renderCount, m_cellsDrawn, m_cellsCulled;

};
```

&emsp;&emsp;在 `RenderCell` 函数中，调用裁剪类的方法进行裁剪计算，我们将只渲染能看到的部分 Cell：

```c++
bool TerrainClass::RenderCell(ID3D11DeviceContext* deviceContext, int cellId, FrustumClass* Frustum)
{
	float maxWidth, maxHeight, maxDepth, minWidth, minHeight, minDepth;
	bool result;


	// Get the dimensions of the terrain cell.
	m_TerrainCells[cellId].GetCellDimensions(maxWidth, maxHeight, maxDepth, minWidth, minHeight, minDepth);

	// Check if the cell is visible.  If it is not visible then just return and don't render it.
	result = Frustum->CheckRectangle2(maxWidth, maxHeight, maxDepth, minWidth, minHeight, minDepth);
	if(!result)
	{
		// Increment the number of cells that were culled.
		m_cellsCulled++;

		return false;
	}

	// If it is visible then render it.
	m_TerrainCells[cellId].Render(deviceContext);

	// Add the polygons in the cell to the render count.
	m_renderCount += (m_TerrainCells[cellId].GetVertexCount() / 3);

	// Increment the number of cells that were actually drawn.
	m_cellsDrawn++;

	return true;
}
```

&emsp;&emsp;在 `ZoneClass` 中我们正常的渲染即可：

```c++
bool ZoneClass::Render(D3DClass* Direct3D, ShaderManagerClass* ShaderManager , TextureManagerClass* TextureManager)
{
	XMMATRIX worldMatrix, viewMatrix, projectionMatrix, baseViewMatrix, orthoMatrix;
	bool result;
	XMFLOAT3 cameraPosition;
	
	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	Direct3D->GetWorldMatrix(worldMatrix);
	m_Camera->GetViewMatrix(viewMatrix);
	Direct3D->GetProjectionMatrix(projectionMatrix);
	m_Camera->GetBaseViewMatrix(baseViewMatrix);
	Direct3D->GetOrthoMatrix(orthoMatrix);
	
	// Get the position of the camera.
	cameraPosition = m_Camera->GetPosition();

	// Construct the frustum.
	m_Frustum->ConstructFrustum(projectionMatrix, viewMatrix);

	// Clear the buffers to begin the scene.
	Direct3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);

	// Turn off back face culling and turn off the Z buffer.
	Direct3D->TurnOffCulling();
	Direct3D->TurnZBufferOff();

	// Translate the sky dome to be centered around the camera position.
	worldMatrix = XMMatrixTranslation(cameraPosition.x, cameraPosition.y, cameraPosition.z);

	// Render the sky dome using the sky dome shader.
	m_SkyDome->Render(Direct3D->GetDeviceContext());
	result = ShaderManager->RenderSkydomeShader(Direct3D->GetDeviceContext(), m_SkyDome->GetIndexCount(), worldMatrix, viewMatrix, 
						    projectionMatrix, m_SkyDome->GetApexColor(), m_SkyDome->GetCenterColor());
	if(!result)
	{
		return false;
	}

	// Reset the world matrix.
	Direct3D->GetWorldMatrix(worldMatrix);

	// Turn the Z buffer back and back face culling on.
	Direct3D->TurnZBufferOn();
	Direct3D->TurnOnCulling();

	if (m_wireFrame) {
		Direct3D->EnableWireframe();
	}

	// Render the terrain cells (and cell lines if needed).
	for(auto i=0; i<m_Terrain->GetCellCount(); i++)
	{
		// Render each terrain cell if it is visible only.
		result = m_Terrain->RenderCell(Direct3D->GetDeviceContext(), i, m_Frustum);
		if(result)
		{
			// Render the cell buffers using the terrain shader.
			result = ShaderManager->RenderTerrainShader(Direct3D->GetDeviceContext(), m_Terrain->GetCellIndexCount(i), worldMatrix, viewMatrix,
								    projectionMatrix, TextureManager->GetTexture(0), TextureManager->GetTexture(1),
								    m_Light->GetDirection(), m_Light->GetDiffuseColor());
			if(!result)
			{
				return false;
			}

			// If needed then render the bounding box around this terrain cell using the color shader. 
			if(m_cellLines)
			{
				m_Terrain->RenderCellLines(Direct3D->GetDeviceContext(), i);
				ShaderManager->RenderColorShader(Direct3D->GetDeviceContext(), m_Terrain->GetCellLinesIndexCount(i), worldMatrix, 
								 viewMatrix, projectionMatrix);
				if(!result)
				{
					return false;
				}
			}
		}
	}

	if (m_wireFrame) {
		Direct3D->DisableWireframe();
	}

	// Update the render counts in the UI.
	result = m_UserInterface->UpdateRenderCounts(Direct3D->GetDeviceContext(), m_Terrain->GetRenderCount(), m_Terrain->GetCellsDrawn(), 
						     m_Terrain->GetCellsCulled());
	if(!result)
	{
		return false;
	}


	// Render the user interface.
	if(m_displayUI)
	{
		result = m_UserInterface->Render(Direct3D, ShaderManager, worldMatrix, baseViewMatrix, orthoMatrix);
		if(!result)
		{
			return false;
		}
	}

	// Present the rendered scene to the screen.
	Direct3D->EndScene();

	return true;
}
```

&emsp;&emsp;最终效果：

![2](https://image.ibb.co/c7K6W7/image.png)

&emsp;&emsp;*可以看我们新增的 UI 显示来了解裁剪情况*

&emsp;&emsp;源代码：[**DX11TerrainTutorial-TerrainCellCulling**](https://github.com/KsGin/DX11TerrainTutorial/tree/master/DX11TerrainTutorial-TerrainCellCulling)

