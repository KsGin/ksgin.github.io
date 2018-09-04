---
title: 【OpenGL】9-鼠标与键盘
date: 2018-02-20 10:13:03
tags: 
	- OpenGL
	- Computer graphics
typora-copy-images-to: ipic
---

#### 参考教程

【中文版】[第六课：键盘和鼠标](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-6-keyboard-and-mouse/)

【英文版】[Tutorial 6 : Keyboard and Mouse](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-6-keyboard-and-mouse/)

#### 学习笔记

&emsp;&emsp;上一篇文章中我们使用了矩阵运算让正方体实现了平移旋转等操作，不过都是用我们设定好的值进行变换，这一篇中我们将实现使用鼠标API使得正方体在X，Y平面上随鼠标移动，并且使用W，A，S，D和Up，Down，Left，Right等键位控制关于X，Y轴的旋转。

<!--more-->

&emsp;&emsp;首先贴出我们使用到的鼠标的API [glfwGetCursorPos](http://www.glfw.org/docs/latest/group__input.html#ga01d37b6c40133676b9cea60ca1d7c0cc)，[glfwSetCursorPos](http://www.glfw.org/docs/latest/group__input.html#ga04b03af936d906ca123c8f4ee08b39e7)，这两个API的作用通过名字便可以猜出来一个是Get光标位置，一个是Set光标位置，我们需要让正方体跟随鼠标移动，那么肯定需要调用Get方法。

&emsp;&emsp;首先看一下官方的API描述：

```c++
 void glfwGetCursorPos   (   GLFWwindow *    window,                          
                          	 double *    xpos,                       
                          	 double *    ypos 
 )   
```

> &emsp;&emsp;This function returns the position of the cursor, in screen coordinates, relative to the upper-left corner of the client area of the specified window.
>
> &emsp;&emsp;If the cursor is disabled (with `GLFW_CURSOR_DISABLED`) then the cursor position is unbounded and limited only by the minimum and maximum values of a `double`.
>
> &emsp;&emsp;The coordinate can be converted to their integer equivalents with the `floor` function. Casting directly to an integer type works for positive coordinates, but fails for negative ones.
>
> &emsp;&emsp;Any or all of the position arguments may be `NULL`. If an error occurs, all non-`NULL` position arguments will be set to zero.
>
> - Parameters
>
>   [in]windowThe desired window.[out]xposWhere to store the cursor x-coordinate, relative to the left edge of the client area, or `NULL`.[out]yposWhere to store the cursor y-coordinate, relative to the to top edge of the client area, or `NULL`.
>
>
> - Errors
>
>   Possible errors include [GLFW_NOT_INITIALIZED](http://www.glfw.org/docs/latest/group__errors.html#ga2374ee02c177f12e1fa76ff3ed15e14a) and [GLFW_PLATFORM_ERROR](http://www.glfw.org/docs/latest/group__errors.html#gad44162d78100ea5e87cdd38426b8c7a1).
>
>
> - Thread safety
>
>   This function must only be called from the main thread.
>
>
> - See also
>
>   [Cursor position](http://www.glfw.org/docs/latest/input_guide.html#cursor_pos)
>
>   [glfwSetCursorPos](http://www.glfw.org/docs/latest/group__input.html#ga04b03af936d906ca123c8f4ee08b39e7)

&emsp;&emsp;我们使用`glfwGetCursorPos`可以获得光标的坐标，由于平移操作是相对于上一刻所对应的位置进行计算，所以我们需要保存上一刻的坐标和当前坐标，如下：

```c++
double preX = width / 2, preY = height / 2;
double curX = preX, curY = preY;
//do something
glfwGetCursorPos(window, &curX, &curY);
preX = curX;
preY = curY;
```

&emsp;&emsp;平移的X，Y参数也修改为

```c++
Model = Model * glm::translate(glm::mat4(), glm::vec3(curX-preX, curY-preY, 0.0f));
```

&emsp;&emsp;点击运行，发现打开之后立方体消失了，怎么回事？

&emsp;&emsp;其实是因为打开的那一瞬间，我们的鼠标光标一般不会在显示的正中心，又因为一次性平移的幅度太大，所以超出了可显示区域。这种时候，我们需要用`glfwSetCursorPos`固定光标到最中心，并且修改`glm::translate`中的X，Y将其缩小到百分之一。如下：

```c++
double preX = width / 2, preY = height / 2;
double curX = preX, curY = preY;
glfwSetCursorPos(window , curX , curY);
do {
    glfwGetCursorPos(window, &curX, &curY);
    Model = Model * glm::translate(glm::mat4(), glm::vec3((curX-preX)/100, (curY-preY)/100, 0.0f));
    preX = curX;
    preY = curY;
    ...
} while(...)
```

&emsp;&emsp;Ok，现在启动之后动动鼠标，是不是正方体跟随鼠标移动而移动啦。

&emsp;&emsp;搞定了跟随鼠标平移，那么现在我们需要实现的是使用W，A，S，D或者Up，Down，Left，Right进行旋转。首先需要看一下我们读取键盘的方法[glfwGetKey](http://www.glfw.org/docs/latest/group__input.html#gadd341da06bc8d418b4dc3a3518af9ad2)：

```c++
int glfwGetKey	(	GLFWwindow * 	window,
					int 	key 
)	
```

> &emsp;&emsp;This function returns the last state reported for the specified key to the specified window. The returned state is one of `GLFW_PRESS` or `GLFW_RELEASE`. The higher-level action `GLFW_REPEAT` is only reported to the key callback.
>
> &emsp;&emsp;If the `GLFW_STICKY_KEYS` input mode is enabled, this function returns `GLFW_PRESS` the first time you call it for a key that was pressed, even if that key has already been released.
>
> &emsp;&emsp;The key functions deal with physical keys, with [key tokens](http://www.glfw.org/docs/latest/group__keys.html) named after their use on the standard US keyboard layout. If you want to input text, use the Unicode character callback instead.
>
> &emsp;&emsp;The [modifier key bit masks](http://www.glfw.org/docs/latest/group__mods.html) are not key tokens and cannot be used with this function.
>
> **Do not use this function** to implement [text input](http://www.glfw.org/docs/latest/input_guide.html#input_char).
>
> - Parameters
>
>   [in]windowThe desired window.[in]keyThe desired [keyboard key](http://www.glfw.org/docs/latest/group__keys.html). `GLFW_KEY_UNKNOWN` is not a valid key for this function.
>
>
> - Returns
>
>   One of `GLFW_PRESS` or `GLFW_RELEASE`.
>
>
> - Errors
>
>   Possible errors include [GLFW_NOT_INITIALIZED](http://www.glfw.org/docs/latest/group__errors.html#ga2374ee02c177f12e1fa76ff3ed15e14a) and [GLFW_INVALID_ENUM](http://www.glfw.org/docs/latest/group__errors.html#ga76f6bb9c4eea73db675f096b404593ce).
>
>
> - Thread safety
>
>   This function must only be called from the main thread.
>
>
> - See also
>
>   [Key input](http://www.glfw.org/docs/latest/input_guide.html#input_key)

&emsp;&emsp;这个方法需要我们传一个Window参数和一个键盘标识符，返回是否按键，使用如下：

```c++
if(glfwGetKey(window , GLFW_KEY_UP)){ //当按Up键的时候
            ...
        }
```

&emsp;&emsp;现在我们需要按Up键的时候正方体绕X轴逆时针旋转，自然可以这样写：

```c++
if(glfwGetKey(window , GLFW_KEY_UP)){
            Model = Model * glm::rotate(glm::mat4() , glm::radians(15.0f), glm::vec3(-1.0f, 0.0f, 0.0f));
        }
```

&emsp;&emsp;其他几个也类似：

```c++
        if(glfwGetKey(window , GLFW_KEY_UP) || glfwGetKey(window , GLFW_KEY_W)){
            Model = Model * glm::rotate(glm::mat4() , glm::radians(15.0f), glm::vec3(-1.0f, 0.0f, 0.0f));
        }

        if(glfwGetKey(window , GLFW_KEY_DOWN) || glfwGetKey(window , GLFW_KEY_S)){
            Model = Model * glm::rotate(glm::mat4() , glm::radians(15.0f), glm::vec3(1.0f, 0.0f, 0.0f));
        }

        if(glfwGetKey(window , GLFW_KEY_LEFT) || glfwGetKey(window , GLFW_KEY_A)){
            Model = Model * glm::rotate(glm::mat4() , glm::radians(15.0f), glm::vec3(0.0f, -1.0f, 0.0f));
        }

        if(glfwGetKey(window , GLFW_KEY_RIGHT) || glfwGetKey(window , GLFW_KEY_D)){
            Model = Model * glm::rotate(glm::mat4() , glm::radians(15.0f), glm::vec3(0.0f, 1.0f, 0.0f));
        }
```

&emsp;&emsp;不会Markdown上传短视频，所以具体效果只能自己尝试运行了，如果有什么地方卡壳的话，可以看这里[源代码](https://github.com/KsGin/LearnOpenGL/tree/master/KeyboardAndMouse)。

