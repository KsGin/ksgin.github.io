---
title: 【图形学基础算法】叁-PTP法生成直线
date: 2018-04-24 21:30:09
mathjax: true
tags:
	- OpenGL
	- Computer graphics
	- Graphics Algorithm
	- Algorithm
---

&emsp;&emsp;这篇文章中我们简单介绍 PTP (point to point comparison method ：逐点比较法) 生成直线算法，它是在画线过程中，每画一点就与理想直线比较，已决定下一点的走向，以步步紧逼的方法点亮最接近直线的一组像素。

<!--more-->

&emsp;&emsp;逐点比较发绘制前，先将直线平移，使 y 坐标较小的点位于坐标原点，这样整条直线都可在第一象限和第二象限中去讨论。这里平移的方法是从直线的两端点中找出 y 较小的点，将起点和终点均减去此点坐标。

&emsp;&emsp;平移后，根据另一个点的 x 来判断直线斜率（小于 0 则斜率为负 ，在第二象限，大于 0 相反）。

&emsp;&emsp;算法根据点在理想直线的位置来判断像素移动（每次移动一个像素），如果像素在直线上方则根据象限不同选择向左还是向右移动，如果在直线下方则向上移动。

&emsp;&emsp;偏差公式表示为：$$F_M=y_Mx_A-y_Ax_M$$ 。M 为画笔的当前位置，$$x_M$$ 和 $$y_M$$ 则是当前位置的 $$x$$ ，$$y$$ 坐标绝对值。而 A 则是终点的坐标（起点是原点）绝对值。当 $$F_M > 0$$ 的时候，则代表当前点在理想点上方，我们下一步的点应该是 $$(x + 1 , y)$$ 或者 $$(x-1 , y)$$，相反则是 $$(x , y+1)$$ 。

&emsp;&emsp;这里的公式我们每次需要计算两次乘法，计算量较大。可以进行优化，我们通过前一点的偏差来推算后一点的偏差。

&emsp;&emsp;设画笔的当前位置为 $$M_1(x_1,y_1)$$ ，此时 $$F_1=y_1x_A-y_ax_1<0$$ ，$$y$$ 方向应走 +1 步到 $$M_2$$ ，即
$$
\left\{
\begin{array}\\
x_2=x_1 \\
y_2=y_1+1
\end{array}
\right.
$$
&emsp;&emsp;此时 $$F_2$$ 为：$$F_2=y_2x_A-y_Ax_2=F_1+x_A$$ ，若下一步的时候 ,$$F_2>=0$$ ，则 $$x$$ 方向应走 +1 步到 $$M_3$$ ，即
$$
\left\{
\begin{array}\\
x_3=x_2+1 \\
y_3=y_2
\end{array}
\right.
$$
&emsp;&emsp;$$F_3$$ 为 ：$$F_3=F_2-y_A$$ 。

&emsp;&emsp;递推可得每一步的结果，而对于第二象限，当 $$F_i >= 0$$ ，$$y$$ 走 +1 ，$$x_{i+1} = x_i , y_{i+1}=y_i+1 , F_{i+1}=F_i-(-x_A)$$，当 $$F_i<0$$ ，$$x$$ 走 -1 ，$$x_{i+1}=x_i-1 , y_{i+1}=y_i , F_{i+1}=F_i+y_A$$。

&emsp;&emsp;最终代码如下：

```c++
void DrawLinePTP(int x1, int y1, int x2, int y2,
                 unsigned char r, unsigned char g, unsigned char b,
                 Texture &tex) {
    int x, y, xA, yA;
    if (y1 > y2) {
        yA = y1 - y2;
        xA = x1 - x2;
    } else {
        yA = y2 - y1;
        xA = x2 - x1;
    }
    int f = x = y = 0;
    int n = abs(xA) + abs(yA);

    for (int i = 0; i < n; ++i) {
        if (xA > 0) {
            if (f >= 0) {
                ++x;
                f -= yA;
            } else {
                ++y;
                f += xA;
            }
        } else {
            if (f >= 0) {
                ++y;
                f += xA;
            } else {
                --x;
                f += yA;
            }
        }
        if (y1 > y2) tex.SetPixel(x + x2, y + y2, r, g, b);
        else tex.SetPixel(x + x1, y + y1, r, g, b);
    }
}
```

