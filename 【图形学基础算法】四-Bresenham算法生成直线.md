---
title: 【图形学基础算法】四-Bresenham算法生成直线
date: 2018-04-26 16:34:49
tags:	
	- OpenGL
	- Computer graphics
	- Graphics Algorithm
	- Algorithm
---

&emsp;&emsp;这篇文章中我们介绍直线生成算法 Bresenham ，Bresenham 是直线生成中最常用的算法，采取加减和移位运算来实现。

<!--more-->

> &emsp;&emsp;Bresenham直线算法是用来描绘由两点所决定的直线的算法，它会算出一条线段在 n 维光栅上最接近的点。这个算法只会用到较为快速的整数加法、减法和位元移位，常用于绘制电脑画面中的直线。是计算机图形学中最先发展出来的算法。

&emsp;&emsp;其具体推算可以看这里：

[The Bresenham Line-Drawing Algorithm](https://www.cs.helsinki.fi/group/goa/mallinnus/lines/bresenh.html)

&emsp;&emsp;实现代码如下：

```c++
void DrawLineBresenham(int x1, int y1, int x2, int y2,
               unsigned char r, unsigned char g, unsigned char b,
               Texture &tex) {
    tex.SetPixel(x1, y1, r, g, b);
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    if (dx == 0 && dy == 0) return;
    int flag = 0;
    if (dx < dy) {
        flag = 1;
        swap(x1, y1);
        swap(x2, y2);
        swap(dx, dy);
    }
    int tx = (x2 - x1) > 0 ? 1 : -1;
    int ty = (y2 - y1) > 0 ? 1 : -1;
    int curx = x1;
    int cury = y1;
    int dS = 2 * dy;
    int dT = 2 * (dy - dx);
    int d = dS - dx;
    while (curx != x2) {
        if (d < 0) d += dS;
        else {
            cury += ty;
            d += dT;
        }
        if (flag) tex.SetPixel(cury, curx, r, g, b);
        else tex.SetPixel(curx, cury, r, g, b);
        curx += tx;
    }
}
```

