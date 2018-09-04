---
title: 【OpenGL】Breakout-5-环境物体
date: 2018-03-15 15:55:13
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;到现在为止，我们的游戏窗口还只是一个黑乎乎的东西，最多加上一个渲染的精灵。我们需要的可不是这样，现在我们来绘制我们的游戏内容——游戏关卡（Game Level）。首先看看 Breakout 的游戏截图（来自维基百科 [Break_out](https://en.wikipedia.org/wiki/Breakout_%28video_game%29)）：

![Breakout_Wiki](https://image.ibb.co/b6jUNx/Breakout2600_svg.png)

<!--more-->

&emsp;&emsp;我们的游戏自然不会是这个画质（=。=），但是我们可以从这张图片中找到我们需要的东西。首先我们的游戏画面包含以下元素：

1. 可 / 不可被破坏的方块
2. 墙壁边界
3. 一个可以活动的挡板
4. 游戏角色

&emsp;&emsp;在游戏正式界面中，我们需要绘制游戏可能出现的场景，其中包括了游戏场景上半部分的可被破坏或者不可被破坏的砖块，一个可以碰撞弹动的游戏对象，一个在我们控制下移动的挡板（游戏对象碰撞到上，左，右边界和不可破坏砖块，挡板反弹，碰到下边界死亡，碰到可被破坏砖块，则砖块被破坏）。

&emsp;&emsp;游戏的砖块位置情况，我们用一个矩阵来表示，矩阵数字为0代表空，为1代表不可破坏砖块，为2代表可破坏砖块。例如：

```c++
1 1 1 1 1 1
1 1 0 0 1 1
2 2 1 1 2 2
```

&emsp;&emsp;这些砖块将被平铺在游戏场景的上半部分，下半部分底部为一个挡板和我们的游戏对象。现在我们来绘制上半部分的场景。

&emsp;&emsp;关于砖块位置的信息我们将其放在文件里，不同的砖块信息我们将之认为不同的关卡（Game Level）。在此，我们创建一个被称为游戏对象的组件作为一个游戏内物体的基本表示。这样的游戏对象持有一些状态数据，如其位置、大小与速率。它还持有颜色、旋转、是否坚硬(不可被摧毁)、是否被摧毁的属性，除此之外，它还存储了一个Texture2D变量作为其精灵(Sprite)。

&emsp;&emsp;首先看看 GameObject 类：

```c++
// Container object for holding all state relevant for a single
// game object entity. Each object in the game likely needs the
// minimal of state as described within GameObject.
class GameObject
{
public:
    // Object state
    glm::vec2   position, size, velocity;
    glm::vec3   color;
    GLfloat     rotation;
    GLboolean   isSolid;
    GLboolean   destroyed;
    // Render state
    Texture2D   sprite;	
    // Constructor(s)
    GameObject();
    GameObject(glm::vec2 pos, glm::vec2 size, Texture2D sprite, glm::vec3 color = glm::vec3(1.0f), glm::vec2 velocity = glm::vec2(0.0f, 0.0f));
    // Draw sprite
    virtual void Draw(SpriteRenderer &renderer);
};

```

&emsp;&emsp;我们在 GameObject 里定义了一系列的属性，并且有一个 Draw 的方法，这个 Draw 则是简单的判断对象是否存在而是否进行渲染。 GameObject 的实现代码可以看这里：[GameObject.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/GameObject.cpp)。

&emsp;&emsp;除此之外，我们需要一个关卡类，用于关卡的操作（初始化，结束）：

```c++
class GameLevel {
public:
	std::vector<GameObject> bricks;
	GameLevel();
	// 从文件中加载关卡
    void Load(const GLchar *file, GLuint levelWidth, GLuint levelHeight);
    // 渲染关卡
    void Draw(SpriteRenderer &renderer);
    // 检查一个关卡是否已完成 (所有非坚硬的瓷砖均被摧毁)
    GLboolean IsCompleted();
private:
    // 由砖块数据初始化关卡
	void Init(std::vector<std::vector<GLuint>> tileData, GLuint levelWidth, GLuint levelHeight);
};
```

&emsp;&emsp;GameLevel 类的实现看这里：[GameLevel.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/GameLevel.cpp)。

&emsp;&emsp;现在我们有了关卡类，可以正式的绘制关卡场景了。我们现在 Game 类里边加上一个存储关卡的数组以及一个游戏当前关卡的变量：

```c++
std::vector<GameLevel> levels;
GLuint                 level;
```

&emsp;&emsp;在 Init 方法里，我们初始化这两个变量：

```c++
// 加载关卡
GameLevel one;
one.Load("resources/levels/one.lvl", this->width, this->height * 0.5);
this->levels.push_back(one);
this->level = 0;
```

&emsp;&emsp;有了关卡信息后，我们使用 GameLevel 类的 Draw 进行绘制：

```c++
// 加载关卡
GameLevel one;
one.Load("resources/levels/one.lvl", this->width, this->height * 0.5);
GameLevel two; 
two.Load("levels/two.lvl", this->width, this->height * 0.5);
GameLevel three; 
three.Load("levels/three.lvl", this->width, this->height * 0.5);
GameLevel four; 
four.Load("levels/four.lvl", this->width, this->height * 0.5);
this->levels.push_back(one);
this->levels.push_back(two);
this->levels.push_back(three);
this->levels.push_back(four);
this->level = 0;
```

&emsp;&emsp;有了关卡信息后，我们使用 GameLevel 类的 Draw 进行绘制：

```c++
if (this->state == GAME_ACTIVE)
{
	// 绘制关卡
	this->levels[this->level].Draw(*renderer);
}
```

&emsp;&emsp;单一的颜色可能比较丑：

![](https://image.ibb.co/k68Z5H/image.png)

&emsp;&emsp;我们为其加上背景和砖块纹理，首先在 Game::Init 方法里使用 ResourceManager 类读取纹理：

```c++
ResourceManager::LoadTexture2D("resources/textures/background.jpg", GL_FALSE, "background");
ResourceManager::LoadTexture2D("resources/textures/block.png", GL_FALSE, "block");
ResourceManager::LoadTexture2D("resources/textures/block_solid.png", GL_FALSE, "block_solid");
```

&emsp;&emsp;在关卡初始化的时候为可破坏和不可破坏的纹理的加入不同的纹理：

```c++
void GameLevel::Init(std::vector<std::vector<GLuint>> tileData, const GLuint levelWidth, const GLuint levelHeight) {
	const auto height = tileData.size();
	const auto width = tileData[0].size();
	const auto unitWidth = levelWidth / static_cast<GLfloat>(width);
	const auto unitHeight = static_cast<GLfloat>(levelHeight) / height;
	for (GLuint y = 0; y < height; ++y)
	{
		for (GLuint x = 0; x < width; ++x)
		{
			const glm::vec2 pos(unitWidth * x, unitHeight * y);
			const glm::vec2 size(unitWidth, unitHeight);

			if (tileData[y][x] == 1)
			{
				//GameObject obj(pos, size,null,glm::vec3(1.0f, 0.0f, 0.0f));
				GameObject obj(pos, size,
					ResourceManager::GetTexture2D("block_solid"),
					glm::vec3(1.0f, 0.0f, 0.0f)
				);
                
				obj.isSolid = GL_TRUE;
				this->bricks.push_back(obj);
			} else if (tileData[y][x] > 1)
			{
				auto color = glm::vec3(0.5f); 
				if (tileData[y][x] == 2)
					color = glm::vec3(0.3f, 0.2f, 0.0f);
				else if (tileData[y][x] == 3)
					color = glm::vec3(0.1f, 0.3f, 0.1f);
				else if (tileData[y][x] == 4)
					color = glm::vec3(0.4f, 0.6f, 0.1f);
				else if (tileData[y][x] == 5)
					color = glm::vec3(0.3f, 0.1f, 0.9f);
                //this->bricks.emplace_back(pos, size, null , color);
				this->bricks.emplace_back(pos, size, ResourceManager::GetTexture2D("block"), color);
			}
		}
	}
}
```

&emsp;&emsp;（注意注释部分，我们用实际加载的纹理代替了一个空的纹理对象）。最后我们在绘制关卡前先绘制背景纹理：

```c++
void Game::Render() {
	if (this->state == GAME_ACTIVE)
	{
		// 绘制背景
		auto backgroundTexture = ResourceManager::GetTexture2D("background");
		renderer->DrawSprite(backgroundTexture, glm::vec2(0, 0), glm::vec2(this->width, this->height), 0.0f , glm::vec3(0.1f , 0.1f , 0.1f));
		// 绘制关卡
		this->levels[this->level].Draw(*renderer);
	}
}
```

&emsp;&emsp;最终效果如下：

![Level](https://image.ibb.co/jSErkH/image.png)

&emsp;&emsp;貌似不错了，现在我们来绘制挡板，首先在 Game 类中添加 paddle 变量（同样也是 GameObject 对象）：

```c++
GameObject      player;
```

&emsp;&emsp;在 Game::Init 方法里，我们读取纹理并且初始化这个对象：

```c++
ResourceManager::LoadTexture2D("resources/textures/paddle.png", GL_TRUE, "paddle");
const auto paddleSize = glm::vec2(200.0f, 40.0f);
const auto paddlePos = glm::vec2(static_cast<GLfloat>(width) / 2 - paddleSize.x / 2 , height - paddleSize.y / 2);	//在底部
player = GameObject(paddlePos , paddleSize , ResourceManager::GetTexture2D("paddle") , glm::vec3(0.3f) , glm::vec2(500.0f));
```

&emsp;&emsp;最后，在 Game::Render 方法里绘制：

```c++
// 绘制挡板
this->player.Draw(*renderer);
```

&emsp;&emsp;效果如下：

![Result](https://image.ibb.co/faVHsx/image.png)