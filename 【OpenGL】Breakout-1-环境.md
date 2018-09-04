---
title: 【OpenGL】Breakout-1-环境
date: 2018-03-11 13:20:15
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;终于开始用所学的东西去写点可以真正使用的内容了啊。由于之前的学习都是在 Mac 环境下，而这次的实战项目决定在 Windows 下使用 VS 2017 实现（VS的安装请去网上随便找个教程）。所以这篇文章主要为环境配置。

&emsp;&emsp;首先，这个练手实战项目中 OpenGL， GLFW , GLEW 三个库是必不可少的，其次我们使用 Assimp （可能会使用）导入模型，用 GLM 计算矩阵等，这篇中就是为了配置这些东西。

<!--more-->

&emsp;&emsp;GLFW ， GLEW 下载比较简单，和 MAC 种差不多。下载 [GLEW](https://sourceforge.net/projects/glew/) ，[GLFW](http://www.glfw.org/) ，在这里我们依旧下载源文件，然后使用 CMake GUI 程序创建 VS2017 项目，之后将其编译为静态库（.lib）文件即可。

&emsp;&emsp;我们在 C 盘根目录创建 OpenGL 文件夹，并在其中创建子文件夹 libs 和 headers 。

![文件夹](https://image.ibb.co/mK4h2n/image.png)

&emsp;&emsp;之后，我们将生成的 lib 文件拖入 libs 文件夹，将源文件里的 headers 目录拖入 headers 目录，最好分类放，如下：

![文件夹1](https://image.ibb.co/dU00oS/image.png)

&emsp;&emsp; Assimp 的话也差不多，点此 [Assimp下载](https://github.com/assimp/assimp/releases/tag/v3.3.1/) 源文件，同样生成为 .lib 文件， 放入 libs ， headers 文件夹拖入 headers 。

&emsp;&emsp;我们还需要 GLM 和 stbi 库（这两个只需要下载头文件）就可以了，[GLM](https://github.com/g-truc/glm/tags) ，[STBI](https://github.com/nothings/stb) 。将其放入 headers 文件夹。

&emsp;&emsp;做完以上步骤后，我们打开 Visual Studio ，创建新项目。

![new item](https://image.ibb.co/eJezhn/image.png)

&emsp;&emsp;选择 Visual C++ 和 Empty Project。

&emsp;&emsp;项目创建完成后，我们右键项目打开属性 Properties 。点击 VC++ Directories ， 修改 Include Directories 将我们刚才的 headers 文件夹添加进去，修改 Library Directories 将 libs 文件夹添加进去。

![add include](https://image.ibb.co/dUW1v7/image.png)

&emsp;&emsp;除此之外，点击 Linker - Input ，在 Additional Dependencies 中添加我们刚才的那几个 lib 文件，比如我是

opengl32.lib ，glfw3.lib，glew32.lib，assimp.lib（opengl32.lib是我们安装 VS 就会有的，所以不用单独下载，当然你单独下载也可以）。

![dependencies](https://image.ibb.co/dJBvNn/image.png)

&emsp;&emsp;配置了这些之后，按理说已经没有问题了。我们创建 main.cpp 文件，输入以下代码做实验。

```c++
#include <iostream>

#define GLEW_STATIC
#include <GL/glew.h>
#include <GLFW/glfw3.h>

#define WIDTH 1920
#define HEIGHT 1080

void InitGlfw() {
	/*初始化glfw*/
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3); //设置最大版本
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3); //设置最小版本
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE); //设置core-profile
	glfwWindowHint(GLFW_RESIZABLE, false); //设置不可改变大小
#ifdef __APPLE__
	glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif
}



void DisPlayColor() {
	static int r = 0;
	static int g = 128;
	static int b = 255;
	static int dr = 1;
	static int dg = 1;
	static int db = 1;

	glClearColor(r / 255.0f, g / 255.0f, b / 255.0f, 1.0f);

	r += dr;
	g += dg;
	b += db;

	if (r > 255 || r < 0) dr = -dr;
	if (g > 255 || g < 0) dg = -dg;
	if (b > 255 || b < 0) db = -db;

	glClear(GL_COLOR_BUFFER_BIT);

}



void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode) {
	if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS) {
		glfwSetWindowShouldClose(window, GL_TRUE);	}

}



int main()
{
	InitGlfw();
	//创建一个窗口
	GLFWwindow *window = glfwCreateWindow(WIDTH, HEIGHT, "Empty Window", nullptr, nullptr);

	//设置窗口环境
	glfwMakeContextCurrent(window);

	if (window == nullptr) {
		std::cout << "Faild to create glfw window" << std::endl;
		glfwTerminate();
	}

	//设置glew
	glewExperimental = GL_TRUE;
	if (glewInit() != GLEW_OK) {
		std::cout << "Faild to init glew" << std::endl;
		return -1;
	}

	//设置位置
	glViewport(0, 0, WIDTH, HEIGHT);
	glfwSetKeyCallback(window, key_callback);

	while (!glfwWindowShouldClose(window)) {
		glfwPollEvents();
		DisPlayColor();
		glfwSwapBuffers(window);

	}

	return 0;

}
```

&emsp;&emsp;如果代码成功运行的话应该是一个不断变换背景颜色的空窗口，如果没有，那么请仔细检查下配置问题。

![result](https://image.ibb.co/mvPqNn/image.png)

&emsp;&emsp;项目 Github 地址 : [https://github.com/KsGin/OpenGL-BreakOut](https://github.com/KsGin/OpenGL-BreakOut)



