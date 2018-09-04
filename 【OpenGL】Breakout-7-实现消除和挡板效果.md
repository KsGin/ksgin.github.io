---
title: 【OpenGL】Breakout-7-实现消除和挡板效果
date: 2018-03-17 15:35:43
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;在我们的游戏中，我们需要使用弹性球去破坏所有的可以破坏的东西来得到游戏的胜利。关于碰撞的检测，这篇文章中我们有所提到：[【OpenGL】25-碰撞检测](https://blog.ksgin.com/2018/03/17/【opengl】25-碰撞检测/)  。这篇文章则是实际运用这个来实现游戏逻辑。

<!--more-->

&emsp;&emsp;我们的游戏中，弹性球是圆形，方块和挡板都是矩形，所以我们使用矩形与圆的碰撞检测算法。首先创建一个检测碰撞的方法：

```c++
public Game {
    ...
    bool CollisionDetection(const GameObject &brick,const BallObject &ball) const;
    ...
}
```

&emsp;&emsp;在 Game.cpp 里我们实现这个方法：

```c++
bool Game::CollisionDetection(const GameObject& brick, const BallObject& ball) const {
	// 求矩形和圆中点坐标
	const glm::vec2 brickCenterPos = brick.position + glm::vec2(brick.size.x / 2.0, brick.size.y / 2.0);
	const glm::vec2 ballCenterPos = ball.position + glm::vec2(ball.radius / 2);

	// 求矢量
	const glm::vec2 vecBrickToBall = ballCenterPos - brickCenterPos;

	// 限制矢量并求最近点
	const glm::vec2 pos = brickCenterPos + glm::vec2(glm::max(glm::min(brick.size.x / 2, vecBrickToBall.x), -brick.size.x / 2),
		glm::max(glm::min(brick.size.y / 2, vecBrickToBall.y), -brick.size.y / 2));

	// 判断碰撞
	return glm::distance(pos, ballCenterPos) < ball.radius;
}
```

&emsp;&emsp;现在我们已经可以判断是否碰撞了，实现碰撞后可破坏的方块被破坏也很简单了，在 Update 里我们遍历当前的所有砖块来进行碰撞检测，当某个砖块发生碰撞的时候，如果它可以被破坏，则直接修改他的值。

```c++
/**
 * 更新游戏状态
 */
void Game::Update(const GLfloat dt) {
	if (this->state == GAME_ACTIVE) {
		ball.Move(dt, width);
		for(GameObject& obj : levels[level].bricks) {
			if (CollisionDetection(obj , ball)) {
				if (!obj.isSolid) {
					obj.destroyed = true;
				}
			}
		}
	}

}
```

&emsp;&emsp;当碰撞到的砖块可以破坏的时候我们销毁他，如果不可以破坏呢？我们会进行反弹操作，如果碰撞在它的 X 轴边上（上下两个边）则对弹性球的 Y 速度坐标取负值，碰撞在 Y 轴边上则是 X 取负值。

&emsp;&emsp;我们修改判断碰撞方式给他添加一个引用参数来返回碰撞边的信息，修改后的碰撞检测方法如下：

```c++
bool Game::CollisionDetection(const GameObject& brick, const BallObject& ball , int& xy) const {
	// 求矩形和圆中点坐标
	const glm::vec2 brickCenterPos = brick.position + glm::vec2(brick.size.x / 2.0, brick.size.y / 2.0);
	const glm::vec2 ballCenterPos = ball.position + glm::vec2(ball.radius / 2);

	// 求矢量
	const glm::vec2 vecBrickToBall = ballCenterPos - brickCenterPos;

	// 限制矢量并求最近点
	const glm::vec2 pos = brickCenterPos + glm::vec2(glm::max(glm::min(brick.size.x / 2, vecBrickToBall.x), -brick.size.x / 2),
		glm::max(glm::min(brick.size.y / 2, vecBrickToBall.y), -brick.size.y / 2));

		const glm::vec2 velo = glm::normalize(ball.velocity);
		float f;
		if (dots[0] >= 0 && dots[0] <= 1 && dots[1] >= 0 && dots[1] <= 1) {
			f = glm::dot(compess[1], compess[4]);
			xy = dots[1] >= f ? 0 : 3;
			if (dots[1] == f && glm::dot(compess[4] , velo) == -1) {
				xy = 4;
			}
		}
		if (dots[1] >= 0 && dots[1] <= 1 && dots[2] >= 0 && dots[2] <= 1) {
			f = glm::dot(compess[2], compess[5]);
			xy = (dots[2] >= f ? 1 : 0);
			if (dots[2] == f && glm::dot(compess[5] , velo) == -1) {
				xy = 4;
			}
		}
		if (dots[2] >= 0 && dots[2] <= 1 && dots[3] >= 0 && dots[3] <= 1) {
			f = glm::dot(compess[3], compess[6]);
			xy = (dots[3] >= f ? 2 : 1);
			if (dots[3] == f && glm::dot(compess[6] , velo) == -1) {
				xy = 4;
			}
		}
		if (dots[3] >= 0 && dots[3] <= 1 && dots[0] >= 0 && dots[0] <= 1) {
			f = glm::dot(compess[0], compess[7]);
			xy = (dots[0] >= f ? 3 : 2);
			if (dots[0] == f && glm::dot(compess[7] , velo) == -1) {
				xy = 4;
			}
		}

	// 判断碰撞
	return glm::distance(pos, ballCenterPos) < ball.radius;
}
```

&emsp;&emsp;事实上我们更应该使用枚举来代表1，2，3，代码更清晰一些。比如这样：

```c++
enum Direction {
	RIGHT,
	DOWN,
	LEFT,
	UP,
	CORNER
};
```

&emsp;&emsp;然后我们修改碰撞检测方法中的 1，2，3 为枚举值。之后便可以在 Update 中根据 xy 引用变量带回来的值来判断弹性球碰到不可破坏砖块的反弹情况：

```c++
/**
 * 更新游戏状态
 */
void Game::Update(const GLfloat dt) {
	if (this->state == GAME_ACTIVE) {
		ball.Move(dt, width);
		for(GameObject& obj : levels[level].bricks) {
			int xy = 0;
			if (CollisionDetection(obj , ball , xy)) {
				if (!obj.isSolid) {
					obj.destroyed = true;
				} else {
					if (xy == COLLISTION_X) {	// 碰撞在 X 轴边上
						ball.velocity.y = -ball.velocity.y;
					} else if(xy == COLLISTION_Y) {
						ball.velocity.x = -ball.velocity.x;
					} else {
						ball.velocity.y = -ball.velocity.y;
						ball.velocity.x = -ball.velocity.x;
					}
				}
			}
		}
	}
}
```

&emsp;&emsp;最后，我们实现挡板和弹性球的碰撞检测，当弹性球碰到挡板的上边时反弹，左右两边和下边则直接死亡。在 Update 方法中加入：

```c++
if (CollisionDetection(player, ball, xy) && xy == UP) {
	ball.velocity.y = -ball.velocity.y;
}
```

&emsp;&emsp;当弹性球落入下边界，则死亡重置：

```c++
if (ball.position.y >= height) {
	player.position = glm::vec2(static_cast<GLfloat>(width) / 2 - player.size.x / 2, height - player.size.y / 2);
	ball.Reset(player.position + glm::vec2(player.size.x / 2 - ball.radius, -player.size.y + ball.radius), glm::vec2(50.0f, -50.0f));
	for (auto& obj : levels[level].bricks) {
				obj.destroyed = false;
	}
}
```

&emsp;&emsp;最后添加胜利结果：

```c++
if (levels[level].IsCompleted()) {
	ball.Reset(player.position + glm::vec2(player.size.x / 2 - ball.radius, -player.size.y + ball.radius), glm::vec2(50.0f, -50.0f));
}
```

&emsp;&emsp;最终代码看这里：[Github](https://github.com/KsGin/OpenGL-BreakOut)

