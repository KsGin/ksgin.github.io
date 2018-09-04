---
title: 【OpenGL】Breakout-6-弹性球
date: 2018-03-16 12:56:46
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;在上一篇文章之后，我们已经实现了基本的游戏场景绘制，但只是个静态的图像，要让它彻底玩起来还需要很多东西。现在我们来实现这个游戏中最重要的一部分：**弹性球（Elastic ball）** 。

<!--more-->

&emsp;&emsp;首先我们将他渲染出来，弹性球既属于 GameObjcet 又有他自己的属性和方法，比如需要一个是否处于被绑定在挡板上的属性和一个 Move（计算弹性球位置） 的方法。所以我们使用 GameObject 派生出来一个类：

```c++
class BallObject : public GameObject {
public:
	// 球的状态 
	GLfloat   radius;
	GLboolean stuck;


	BallObject();
	BallObject(glm::vec2 pos, GLfloat radius, glm::vec2 velocity, Texture2D sprite);

	glm::vec2 Move(GLfloat dt, GLuint windowWidth);
	void      Reset(glm::vec2 position, glm::vec2 velocity);
};
```

&emsp;&emsp;实现代码看这里：[BallObject.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/BallObject.cpp)

&emsp;&emsp;在 Move 方法中，我们根据球的方向来进行位移，当它触碰到边界的时候触发反弹等等，我们仅处理在上，左和右边界的反弹，如果碰到下边则直接 "死亡 " 。

```c++
glm::vec2 BallObject::Move(const GLfloat dt, const GLuint windowWidth) {
	// 如果没有被固定在挡板上
    if (!this->stuck)
    { 
        // 移动球
        this->position += this->velocity * dt;
        // 检查是否在窗口边界以外，如果是的话反转速度并恢复到正确的位置
        if (this->position.x <= 0.0f)
        {
            this->velocity.x = -this->velocity.x;
            this->position.x = 0.0f;
        }
        else if (this->position.x + this->size.x >= windowWidth)
        {
            this->velocity.x = -this->velocity.x;
            this->position.x = windowWidth - this->size.x;
        }
        if (this->position.y <= 0.0f)
        {
            this->velocity.y = -this->velocity.y;
            this->position.y = 0.0f;
        }

    }
    return this->position;
}
```

&emsp;&emsp;现在我们在 Game 类中添加一个 BallObject 对象：

```c++
BallObject		ball;
```

&emsp;&emsp;在 Game::Init 中初始化变量：

```c++
// 加载弹性球 
const auto ballRadius = 12.5f;
const auto ballPos = paddlePos + glm::vec2(player.size.x / 2 - ballRadius, -player.size.y + ballRadius);		//在挡板上面最中央
ball = BallObject(ballPos, ballRadius, glm::vec2(100.0f, -100.0f), ResourceManager::GetTexture2D("face"));
```

&emsp;&emsp;当游戏未开始的时候，我们的弹性球处于 stuck 状态（stuck 属性为 true），此时我们的弹性球被绑定在挡板上随着挡板移动。所以我们在弹性球 stuck 状态时移动挡板，弹性球也会跟随移动，需要修改 Game::ProcessInput 方法：

```c++
/**
 * 处理输入
 */
void Game::ProcessInput(const GLuint dt) {
	if (this->state == GAME_ACTIVE)
	{
		const auto velocity = 10.0f * dt;
		// 移动挡板
		if (this->keys[GLFW_KEY_A])
		{
			if (player.position.x >= 0) {
				player.position.x -= velocity;
				if (ball.stuck) {
					ball.position.x -= velocity;
				}
			}

		}
		if (this->keys[GLFW_KEY_D])
		{
			if (player.position.x <= this->width - player.size.x) {
				player.position.x += velocity;
				if (ball.stuck) {
					ball.position.x += velocity;
				}
			}
		}
		if (this->keys[GLFW_KEY_SPACE])
		{
			ball.stuck = !ball.stuck;
		}
	}
}
```

&emsp;&emsp;可以看到，我们顺便设置了按空格键修改弹性球的固定属性（stuck）值。

&emsp;&emsp;在 Game::Update方法里，我们调用弹性球的 Move 方法：

```c++
/**
 * 更新游戏状态
 */ 
void Game::Update(const GLfloat dt) {
	if (this->state == GAME_ACTIVE) {
		ball.Move(dt, width);
	}
}
```

&emsp;&emsp;最后，在 Game::Render 里进行渲染：

```c++
// 绘制球
this->ball.Draw(*renderer);
```

&emsp;&emsp;球体的 Update 我们实现了，现在还需要实现挡板的移动。这个比较简单，只需要我们在 Game::ProcessInput 方法里根据按键的不同修改挡板的坐标就可以了（代码上边已经有了）。请注意在这里我们调用了 Game 类的 keys 变量，这个变量是一个 bool 型的数组，里边的每一个元素代表着一个按键的值。每次按键就会更改按键的值，我们在需要判断按键的时候直接判断数组就可以了。

&emsp;&emsp;在渲染循环里更新 keys ：

```c++
for (auto i = 0; i < 1024; ++i) {
	game.keys[i] = glfwGetKey(window, i);
}
```

&emsp;&emsp;之后你应该会看到一个完整的场景：

![ball](https://image.ibb.co/b58saH/image.png)

&emsp;&emsp;运行效果请看这里：<https://youtu.be/5TTyPp8BIJs>