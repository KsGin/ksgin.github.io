---
title: 【DirectX】39-粒子发生器
date: 2018-04-21 23:36:09
tags:
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 39: Particle Systems](http://www.rastertek.com/dx11tut39.html)

#### 学习记录

&emsp;&emsp;这篇文章中我们简单的介绍粒子发生器的实现，我们将使用一个 `ParticleSystemClass ` 类来控制粒子，一个 `ParticleShaderClass` 类来渲染粒子。

<!--more-->

&emsp;&emsp;首先来看看我们的 `ParticleSystemClass` 类，声明如下：

```c++
class ParticleSystemClass {
private:
	struct ParticleType {
		float positionX, positionY, positionZ;
		float red, green, blue;
		float velocity;
		bool active;
	};

	struct VertexType {
		DirectX::XMFLOAT3 position;
		DirectX::XMFLOAT2 texture;
		DirectX::XMFLOAT4 color;
	};

public:
	
	ParticleSystemClass();
	ParticleSystemClass(const ParticleSystemClass&);
	~ParticleSystemClass();

	bool Initialize(ID3D11Device*, CHAR*);
	void Shutdown();
	bool Frame(float, ID3D11DeviceContext*);
	void Render(ID3D11DeviceContext*);

	ID3D11ShaderResourceView* GetTexture();
	int GetIndexCount();

private:
	bool LoadTexture(ID3D11Device*, CHAR*);
	void ReleaseTexture();

	bool InitializeParticleSystem();
	void ShutdownParticleSystem();

	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();

	void EmitParticles(float);
	void UpdateParticles(float);
	void KillParticles();

	bool UpdateBuffers(ID3D11DeviceContext*);
	void RenderBuffers(ID3D11DeviceContext*);
private:
	float m_particleDeviationX, m_particleDeviationY, m_particleDeviationZ;
	float m_particleVelocity, m_particleVelocityVariation;
	float m_particleSize, m_particlesPerSecond;
	int m_maxParticles;
	int m_currentParticleCount;
	float m_accumulatedTime;
	TextureClass* m_Texture;
	ParticleType* m_particleList;
	int m_vertexCount, m_indexCount;
	VertexType* m_vertices;
	ID3D11Buffer *m_vertexBuffer, *m_indexBuffer;
};
```

&emsp;&emsp;这个类可能比较复杂，因为我们的粒子发生器的每一个粒子有自己的坐标，速度，颜色等等。我们使用 `void EmitParticles(float);` ， `void UpdateParticles(float);`，`void KillParticles();` 来控制粒子的生命周期。我们不断的在上方生成粒子，并且在下方结束他们，以达到一个粒子瀑布的效果。

&emsp;&emsp;`EmitParticles` 方法如下：

```c++
void ParticleSystemClass::EmitParticles(float frameTime) {
	bool emitParticle, found;
	float positionX, positionY, positionZ, velocity, red, green, blue;
	int index, i, j;
	// Increment the frame time.
	m_accumulatedTime += frameTime;

	// Set emit particle to false for now.
	emitParticle = false;
	
	// Check if it is time to emit a new particle or not.
	if(m_accumulatedTime > (1000.0f / m_particlesPerSecond))
	{
		m_accumulatedTime = 0.0f;
		emitParticle = true;
	}

	// If there are particles to emit then emit one per frame.
	if((emitParticle == true) && (m_currentParticleCount < (m_maxParticles - 1)))
	{
		m_currentParticleCount++;

		// Now generate the randomized particle properties.
		positionX = (((float)rand()-(float)rand())/RAND_MAX) * m_particleDeviationX;
		positionY = (((float)rand()-(float)rand())/RAND_MAX) * m_particleDeviationY;
		positionZ = (((float)rand()-(float)rand())/RAND_MAX) * m_particleDeviationZ;

		velocity = m_particleVelocity + (((float)rand()-(float)rand())/RAND_MAX) * m_particleVelocityVariation;

		red   = (((float)rand()-(float)rand())/RAND_MAX) + 0.5f;
		green = (((float)rand()-(float)rand())/RAND_MAX) + 0.5f;
		blue  = (((float)rand()-(float)rand())/RAND_MAX) + 0.5f;

		// Now since the particles need to be rendered from back to front for blending we have to sort the particle array.
		// We will sort using Z depth so we need to find where in the list the particle should be inserted.
		index = 0;
		found = false;
		while(!found)
		{
			if((m_particleList[index].active == false) || (m_particleList[index].positionZ < positionZ))
			{
				found = true;
			}
			else
			{
				index++;
			}
		}

		// Now that we know the location to insert into we need to copy the array over by one position from the index to make room for the new particle.
		i = m_currentParticleCount;
		j = i - 1;

		while(i != index)
		{
			m_particleList[i].positionX = m_particleList[j].positionX;
			m_particleList[i].positionY = m_particleList[j].positionY;
			m_particleList[i].positionZ = m_particleList[j].positionZ;
			m_particleList[i].red       = m_particleList[j].red;
			m_particleList[i].green     = m_particleList[j].green;
			m_particleList[i].blue      = m_particleList[j].blue;
			m_particleList[i].velocity  = m_particleList[j].velocity;
			m_particleList[i].active    = m_particleList[j].active;
			i--;
			j--;
		}

		// Now insert it into the particle array in the correct depth order.
		m_particleList[index].positionX = positionX;
		m_particleList[index].positionY = positionY;
		m_particleList[index].positionZ = positionZ;
		m_particleList[index].red       = red;
		m_particleList[index].green     = green;
		m_particleList[index].blue      = blue;
		m_particleList[index].velocity  = velocity;
		m_particleList[index].active    = true;
	}
	return;
}
```

&emsp;&emsp;我们在这里生成粒子信息，在 `UpdateParticles` 方法中修改他们的位置，使得其不断向下：

```c++
void ParticleSystemClass::UpdateParticles(float frameTime) {
	int i;
	// Each frame we update all the particles by making them move downwards using their position, velocity, and the frame time.
	for(i=0; i<m_currentParticleCount; i++)
	{
		m_particleList[i].positionY = m_particleList[i].positionY - (m_particleList[i].velocity * frameTime * 0.001f);
	}
	return;
}
```

&emsp;&emsp;最后当粒子移动到下方时，我们使用 `KillParticles` 清除粒子：

```c++
void ParticleSystemClass::KillParticles() {

	int i, j;

	// Kill all the particles that have gone below a certain height range.
	for(i=0; i<m_maxParticles; i++)
	{
		if((m_particleList[i].active == true) && (m_particleList[i].positionY < -3.0f))
		{
			m_particleList[i].active = false;
			m_currentParticleCount--;

			// Now shift all the live particles back up the array to erase the destroyed particle and keep the array sorted correctly.
			for(j=i; j<m_maxParticles-1; j++)
			{
				m_particleList[j].positionX = m_particleList[j+1].positionX;
				m_particleList[j].positionY = m_particleList[j+1].positionY;
				m_particleList[j].positionZ = m_particleList[j+1].positionZ;
				m_particleList[j].red       = m_particleList[j+1].red;
				m_particleList[j].green     = m_particleList[j+1].green;
				m_particleList[j].blue      = m_particleList[j+1].blue;
				m_particleList[j].velocity  = m_particleList[j+1].velocity;
				m_particleList[j].active    = m_particleList[j+1].active;
			}
		}
	}

	return;
}
```

&emsp;&emsp;我们在每一帧中调用这三个方法，他会不断地生成新粒子，修改粒子瀑布中的粒子位置并且删除已经到达底部的粒子。

&emsp;&emsp;`ParticleShaderClass` 类则是和其它的渲染类似，所以在此暂且不表，我们的主要内容便是上边的三个方法。最终效果如下：

![1](https://image.ibb.co/nJgAbx/image.png)

&emsp;&emsp;源代码：[**DX11Tutorial-ParticleSystems**](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-ParticleSystems)