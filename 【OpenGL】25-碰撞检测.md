---
title: 【OpenGL】25-碰撞检测
date: 2018-03-17 10:16:02
tags:
	- OpenGL	
	- Computer graphics
---

&emsp;&emsp;这篇文章中，我们将介绍 [2D碰撞检测（2D Collision detection）](https://zh.wikipedia.org/wiki/碰撞偵測) 。在我们的 Breakout 游戏中，我们将使用碰撞检测来实现弹性球销毁可破坏方块功能。

<!--more-->

&emsp;&emsp;首先介绍我们可能用到的几个碰撞检测算法。我们在处理碰撞检测时，并不会用物体本身的顶点数据，而是使用它的近似几何来判断，比如我们的方块的近似几何自然就是长方形的，我们的弹性球近似几何就是圆形。

&emsp;&emsp;AABB（Axis-aligned Bounding Box）即**轴对齐碰撞箱**，它使用正方形的轴来判断是否发生碰撞。AABB-AABB则是两个矩形的碰撞检测。当两个矩形的 X ，Y 轴都发生重合时，则这两个矩形发生了碰撞。而判断轴重合则可以根据两个矩形的中点轴距离（在某一个轴上的距离）和中点到各自的变距离之差来判断

![](https://image.ibb.co/koRLfH/collisions_overlap.png)

&emsp;&emsp;我们用代码来计算：

```c++
bool CollisionDetection(Rectangle ra , Rectangle rb){
    bool xc = (abs(ra.pos.x - rb.pos.x) - (ra.size.x + rb.size.x)) > 0;	//判断 x 轴是否重合
    bool yc = (abs(ra.pos.y - rb.pos.y) - (ra.size.y + rb.size.y)) > 0; //判断 y 轴是否重合
    return xc && yc; //x y 轴都重合时发生碰撞
}
```

&emsp;&emsp;**注意：AABB-AABB 只能检测到没有旋转过的矩形之间的碰撞。**

&emsp;&emsp;除了 AABB-AABB ，我们来再看下**矩形**和**圆**之间的的碰撞检测。矩形和圆的碰撞检测则是根据矩形到圆的最近点到圆的距离和圆半径长度比较的，到圆的距离小于圆的半径则发生了碰撞。这个算法较为麻烦的就是求矩形到圆的最近的点，看下图：

![c1](https://image.ibb.co/keEbnx/collisions_aabb_circle.png)

&emsp;&emsp;我们使用圆和矩形的中点的矢量（圆中点坐标 - 矩形中点坐标），然后使用矩形的半边长来限制这个矢量使得它与矩形中心相加的点不会超出矩形的范围。最后我们用矩形的中心点加上这个限制后的向量。如下：

```c++
bool CollisionDetection(Rectangle ra , Circle cb){
    //求矢量
    vec2 v2;
    v2.x = cb.pos.x - ra.pos.x;
    v2.y = cb.pos.y - ra.pos.y;
    //限制其范围
    v2.x = min(ra.size.x , max(-ra.size.x , v2.x));
    v2.y = min(ra.size.y , max(-ra.size.y , v2.y));
    //求最近点的坐标
    vec2 pos = v2 + ra.pos;
    //返回碰撞检测结果 （距离小于半径时）
    return sqrt(pow(pos.x - cb.pos.x,2) , pow(pos.y - cb.pos,y , 2)) < cb.radius;
}
```

&emsp;&emsp;最后来看看两个圆之间的碰撞吧，这个比较简单，直接判断两中点坐标的距离和半径之和的差值就可以了：

```c++
bool CollisionDetection(Circle ca , Circle cb){
    //返回碰撞检测结果 （距离小于半径时）
    return sqrt(pow(ca.pos.x - cb.pos.x,2) , pow(ca.pos.y - cb.pos,y , 2)) < ca.radius + cb.radius;
}
```