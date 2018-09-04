---
title: 【图形学基础算法】贰-DDA法生成直线
date: 2018-04-23 20:52:42
mathjax: true
tags:
	- OpenGL
	- Computer graphics
	- Graphics Algorithm
	- Algorithm
---

&emsp;&emsp;这篇文章中我们简单介绍 DDA （Digital differential analyzer ：数值微分法），它根据直线的微分方程来绘制直线，是最简单的一种画线方法。

<!--more-->

&emsp;&emsp;设直线的起点坐标为 $$P_s(x_s,y_s)$$ ，终点坐标为$$P_e(x_e,y_e)$$ ，令 $$\Delta{x} = x_e-x_s$$，$$\Delta{y} = y_e-y_s$$ ，则要绘制的直线微分方程为 
$$
\frac{dx}{dt} = \Delta{x} , \frac{dy}{dt} = \Delta{y} 
$$
&emsp;&emsp;将上式中的微分用差商来代替，即得：
$$
\frac{x_{i+1}-x_{i}}{t_{i+1}-t_i} = \Delta{x} , \frac{y_{i+1}-y_{i}}{t_{i+1}-t_i} = \Delta{y}
$$
&emsp;&emsp;令 $$\Delta{m}=max(|\Delta{x}|,|\Delta{y}|)$$ ，取时间步长为 $$t2-t1=1/\Delta{m}$$ ，则每走一步，可得微分方程数值解得递推公式为 
$$
x_{i+1}=x_i+\Delta{x}/\Delta{m} , y_{i+1}=y_i+\Delta{y}/\Delta{m}
$$
&emsp;&emsp;根据此公式，即可简单的推出我们我们下一步的像素坐标，转换为代码如下：

```c++
void DrawLineDDA(int x1 , int y1 , int x2 , int y2 ,
                 unsigned char r , unsigned char g , unsigned char b ,
                 Texture& tex){
    int dm = 0;
    if (abs(x2 - x1) >= abs(y2 - y1)){
        dm = abs(x2 - x1);
    } else {
        dm = abs(y2 - y1);
    }
    float dx = (float)(x2 - x1) / dm;
    float dy = (float)(y2 - y1) / dm;
    float x = static_cast<float>(x1 + 0.5);
    float y = static_cast<float>(y1 + 0.5);
    for (int i = 0; i < dm; ++i) {
        tex.SetPixel((int)x , (int)y , r , g , b);
        x += dx;
        y += dy;
    }
}
```

&emsp;&emsp;在我们的主循环中调用这个方法绘制两条斜线，如下：

```c++
while (!glfwWindowShouldClose(pWindow)) {
    glfwPollEvents();

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);

    glslShader.use();
    glslTex.Use();
    screen.Use();

    glslTex.ClearPixel();
    DrawLineDDA(100 , 100 , 700 , 400 , 0 , 255 , 0 , glslTex);
    DrawLineDDA(100 , 400 , 700 , 100 , 255 , 0 , 0 , glslTex);

    glDrawElements(GL_TRIANGLES , screen.IndexCount() , GL_UNSIGNED_INT , 0);

    glfwSwapBuffers(pWindow);
}
```

&emsp;&emsp;最终效果：

![1](https://image.ibb.co/jYoW3c/image.png)

&emsp;&emsp;*参考资料 ：《计算机图形学基础（OpenGL版）》*