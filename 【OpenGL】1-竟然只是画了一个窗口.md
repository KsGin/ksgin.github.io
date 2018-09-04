---
title: 【OpenGL】1-竟然只是画了一个窗口
date: 2017-12-11 13:05:33
tags:
    - OpenGL
    - Computer graphics
---



【OPENGL菜鸟历程】

&emsp;&emsp;O.O从第一次知道OPENGL这个名字到现在已经有三年了，但是还是第一次真正去学这个东西。虽然在之前也大概看过DX，对这种东西的复杂度略知一二，并且做好了去长期啃这个的打算。但是，还是难啊_(:зゝ∠)_

&emsp;&emsp;我看的是[LearnOpenGL](https://learnopengl-cn.github.io/)这个教程，光是配置GLEW,GLFW就费了好久的功夫。唉，距离我的引擎梦，还差多久啊。

&emsp;&emsp;言归正传，本文中的代码可用性建立在你正确的配置了GLEW和GLFW的条件下，接下来有时间的话我会写一篇文章来说明如何在MAC OS下配置GLEW , GLFW，并使用Clion进行代码编写。

<!--more-->

【CMakeList.txt文件】

```cmake
cmake_minimum_required(VERSION 3.8)
project(EmptyWindow)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework OPENGL")
set(SOURCE_FILES EmptyWindow.cpp)
add_executable(EmptyWindow ${SOURCE_FILES})

target_link_libraries(EmptyWindow "/usr/local/lib/libglfw.dylib")

target_link_libraries(EmptyWindow "/usr/local/lib/libGLEW.dylib")
```

&emsp;&emsp;其中`set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework OPENGL")`,`target_link_libraries(EmptyWindow "/usr/local/lib/libglfw.dylib")`和`target_link_libraries(EmptyWindow "/usr/local/lib/libGLEW.dylib")`是对openGL的配置。

&emsp;&emsp;根据这几项的属性名就可以大概猜出来是要配置什么，第一个是链接OPENGL框架，第二个是链接glfw动态链接库，第三个自然就是glew的动态链接库了。

【EmptyWindow.cpp】

```c++
#include <iostream>

#define GLEW_STATIC
#include <GL/glew.h>
#include <GLFW/glfw3.h>

void InitGlfw(){
    /*初始化glfw*/
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR , 3); //设置最大版本
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR , 3); //设置最小版本
    glfwWindowHint(GLFW_OPENGL_PROFILE , GLFW_OPENGL_CORE_PROFILE); //设置core-profile
    glfwWindowHint(GLFW_RESIZABLE , false); //设置不可改变大小
#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT , GL_TRUE);
#endif
}


void DisPlayColor(){

    static int r = 0;
    static int g = 128;
    static int b = 255;
    static int dr = 1;
    static int dg = 1;
    static int db = 1;


    glClearColor(r / 255.0f  , g / 255.0f , b / 255.0f  , 1.0f);

    r += dr;
    g += dg;
    b += db;
    if (r > 255 || r < 0) dr = -dr;
    if (g > 255 || g < 0) dg = -dg;
    if (b > 255 || b < 0) db = -db;

    glClear(GL_COLOR_BUFFER_BIT);
}

void key_callback(GLFWwindow* window , int key , int scancode , int action , int mode){
    if(key == GLFW_KEY_ESCAPE && action == GLFW_PRESS){
        glfwSetWindowShouldClose(window , GL_TRUE);
    }
}

int main()
{

    InitGlfw();

    //创建一个窗口
    GLFWwindow *window = glfwCreateWindow(800 , 600 , "Empty Window" , NULL , NULL);

    //设置窗口环境
    glfwMakeContextCurrent(window);

    if(window == NULL){
        std::cout << "Faild to create glfw window" << std::endl;
        glfwTerminate();
    }

    //设置glew
    glewExperimental = GL_TRUE;
    if(glewInit() != GLEW_OK) {
        std::cout << "Faild to init glew" << std::endl;
        return -1;
    }

    //设置位置
    glViewport(100 , 100 , 800 , 600);

    glfwSetKeyCallback(window , key_callback);

    while (!glfwWindowShouldClose(window)){

        glfwPollEvents();

        DisPlayColor();

        glfwSwapBuffers(window);
    }

    return 0;
}

```

&emsp;&emsp;这是我的文件代码，最后的输出是一个根据时间动态变色的窗口，看起来还是很酷炫的。如果不实现这些花样仅仅是输出一个窗口的话，则可以去掉`DisPlayColor`这个函数，并且将main方法里的while循环中的方法调用移除就可以了。

&emsp;&emsp;之所以使用while，是因为程序顺序执行，到了return 0退出，但是我们不想看见一个一闪而过的窗口，所以用一个while循环来让它持续显示。这个循环的循环条件`!glfwWindowShouldClose(window)`顾名思义，是当窗口不应该退出的时候就继续循环。可是什么时候应该退出呢？这个时候就需要我们的事件处理了，这个以后再提。

&emsp;&emsp;当然，输出这样一个黑窗口确实有点丑，虽然刚开始学并不为了美观，但是强迫症还是想看的舒服一些，所以在循环里边添加这么两行代码。

```c++
glClearColor(0  , 110 , 110 , 0.2f); //参数为 red , green , blue 最后一个参数暂时不管
glClear(GL_COLOR_BUFFER_BIT);
```

&emsp;&emsp;这样，一个窗口就写好啦。