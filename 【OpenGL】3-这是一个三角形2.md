---
title: 【OpenGL】3-这是一个三角形2
date: 2017-12-13 11:14:02
tags: 
    - OpenGL
    - Computer graphics
---

#### Shader

&emsp;&emsp;这篇文章我们将简单介绍一下[Shader](https://en.wikipedia.org/wiki/Shader)与对Shader编程，在此将讨论OpenGL的Shader语言[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)。

&emsp;&emsp;首先推荐教程（毕竟我这里只是我自己学的经历，为防止错误的思想误导了别人，还是先摆出比较正确的教程再说）

​		1. [着色器](https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/05%20Shaders/)

​		2. [GLSL](http://www.cnblogs.com/mazhenyu/p/5580946.html)

<!--more-->

> &emsp;&emsp;**着色器**（英语：shader）应用于[计算机图形学](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9B%BE%E5%BD%A2%E5%AD%A6)领域，指一组供计算机图形资源在执行渲染任务时使用的指令，用于计算图像的颜色或明暗。但近来，它也能用于处理一些特殊效果，或者视频后处理。通俗地说，着色器告诉电脑如何用特有的一种方法去绘制物体。
>
> &emsp;&emsp;程序员将着色器应用于[图形处理器](https://zh.wikipedia.org/wiki/%E5%9C%96%E5%BD%A2%E8%99%95%E7%90%86%E5%99%A8)（GPU）的[可编程流水线](https://zh.wikipedia.org/w/index.php?title=%E5%8F%AF%E7%BC%96%E7%A8%8B%E6%B5%81%E6%B0%B4%E7%BA%BF&action=edit&redlink=1)，来实现三维应用程序。这样的图形处理器有别于传统的固定流水线处理器，为GPU编程带来更高的灵活性和适应性。以前固有的流水线只能进行一些几何变换和像素灰度计算。现在[可编程流水线](https://zh.wikipedia.org/w/index.php?title=%E5%8F%AF%E7%BC%96%E7%A8%8B%E6%B5%81%E6%B0%B4%E7%BA%BF&action=edit&redlink=1)还能处理所有像素、顶点、纹理的位置、色调、饱和度、明度、对比度并实时地绘制图像。着色器还能产生如[模糊](https://zh.wikipedia.org/wiki/%E6%A8%A1%E7%B3%8A)、[高光](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%85%89)、[有体积光源](https://zh.wikipedia.org/w/index.php?title=%E6%9C%89%E4%BD%93%E7%A7%AF%E5%85%89%E6%BA%90&action=edit&redlink=1)、失焦、[卡通渲染](https://zh.wikipedia.org/wiki/%E5%8D%A1%E9%80%9A%E6%B8%B2%E6%9F%93)、[色调分离](https://zh.wikipedia.org/wiki/%E8%89%B2%E8%AA%BF%E5%88%86%E9%9B%A2)、[畸变](https://zh.wikipedia.org/wiki/%E7%95%B8%E8%AE%8A)、[凹凸贴图](https://zh.wikipedia.org/wiki/%E5%87%B9%E5%87%B8%E8%B4%B4%E5%9B%BE)、[边缘检测](https://zh.wikipedia.org/wiki/%E8%BE%B9%E7%BC%98%E6%A3%80%E6%B5%8B)、[运动检测](https://zh.wikipedia.org/w/index.php?title=%E8%BF%90%E5%8A%A8%E6%A3%80%E6%B5%8B&action=edit&redlink=1)等效果

> Direct3D和OpenGL都使用了以下三种着色器：
>
> - [顶点着色器](https://zh.wikipedia.org/wiki/%E9%A0%82%E9%BB%9E%E8%91%97%E8%89%B2%E5%99%A8)处理每个顶点，将顶点的空间位置投影在屏幕上，即计算顶点的二维坐标。同时，它也负责顶点的[深度缓冲](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E7%BC%93%E5%86%B2)（Z-Buffer）的计算。顶点着色器可以掌控顶点的位置、颜色和[纹理坐标](https://zh.wikipedia.org/w/index.php?title=%E7%BA%B9%E7%90%86%E5%9D%90%E6%A0%87&action=edit&redlink=1)等属性，但无法生成新的顶点。顶点着色器的输出传递到流水线的下一步。如果有之后定义了[几何着色器](https://zh.wikipedia.org/wiki/%E5%B9%BE%E4%BD%95%E8%91%97%E8%89%B2%E5%99%A8)，则几何着色器会处理顶点着色器的输出数据，否则，[光栅化器](https://zh.wikipedia.org/wiki/%E5%85%89%E6%A0%85%E5%8C%96%E5%99%A8)继续流水线任务。
>
>
> - [几何着色器](https://zh.wikipedia.org/wiki/%E5%B9%BE%E4%BD%95%E8%91%97%E8%89%B2%E5%99%A8)可以从[多边形网格](https://zh.wikipedia.org/wiki/%E5%A4%9A%E8%BE%B9%E5%BD%A2%E7%BD%91%E6%A0%BC)中增删顶点。它能够执行对CPU来说过于繁重的生成几何结构和增加模型细节的工作。Direct3D版本10增加了支持几何着色器的API, 成为Shader Model 4.0的组成部分。OpenGL只可通过它的一个插件来使用几何着色器，但极有可能在3.1版本中该功能将会归并。几何着色器的输出连接光栅化器的输入。
>
>
> - [像素着色器](https://zh.wikipedia.org/wiki/%E5%83%8F%E7%B4%A0%E7%9D%80%E8%89%B2%E5%99%A8)(Direct3D)，常常又称为[片断着色器](https://zh.wikipedia.org/wiki/%E7%89%87%E6%96%AD%E7%9D%80%E8%89%B2%E5%99%A8)(OpenGL)，处理来自光栅化器的数据。光栅化器已经将多边形填满并通过流水线传送至像素着色器，后者逐像素计算颜色。像素着色器常用来处理场景光照和与之相关的效果，如[凸凹纹理映射](https://zh.wikipedia.org/wiki/%E5%87%B8%E5%87%B9%E7%BA%B9%E7%90%86%E6%98%A0%E5%B0%84)和[调色](https://zh.wikipedia.org/w/index.php?title=%E8%B0%83%E8%89%B2&action=edit&redlink=1)。名称片断着色器似乎更为准确，因为对于着色器的调用和屏幕上像素的显示并非一一对应。举个例子，对于一个像素，片断着色器可能会被调用若干次来决定它最终的颜色，那些被[遮挡](https://zh.wikipedia.org/w/index.php?title=%E9%81%AE%E6%8C%A1&action=edit&redlink=1)的物体也会被计算，直到最后的[深度缓冲](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E7%BC%93%E5%86%B2)才将各物体前后排序。
>
> [统一着色器模型](https://zh.wikipedia.org/w/index.php?title=%E7%BB%9F%E4%B8%80%E7%9D%80%E8%89%B2%E5%99%A8%E6%A8%A1%E5%9E%8B&action=edit&redlink=1)将上述三种着色器统一起来，发布于OpenGL和Direct3D 10里面。

&emsp;&emsp;这一大堆来自于维基百科的介绍，我们可以从中得到一堆的名词，暂时需要记住的就【顶点着色器】和【片段着色器】，我们也将使用这两个着色器让我们的三角形显示出来。

&emsp;&emsp;首先介绍一些Shader的基本构造：

```c++
#version version_number				//声明版本

in type in_variable_name;			//输入变量1
in type in_variable_name;			//输入变量2

out type out_variable_name;			//输出变量

uniform type uniform_name;			//uniform变量

int main()							//main方法
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

&emsp;&emsp;GLSL是一种类C语言，in , out , uniform都是他的关键字 ，type则是float , int等等的基本类型。在GLSL中有`int`,`float`,`double`,`uint`,`bool`这几种基本类型，还有高级一些的向量和矩阵(暂且不提)类型。

### 向量

GLSL中的向量是一个可以包含有1、2、3或者4个分量的容器，分量的类型可以是前面默认基础类型的任意一个。它们可以是下面的形式（`n`代表分量的数量）：

| 类型      | 含义                      |
| ------- | ----------------------- |
| `vecn`  | 包含`n`个float分量的默认向量      |
| `bvecn` | 包含`n`个bool分量的向量         |
| `ivecn` | 包含`n`个int分量的向量          |
| `uvecn` | 包含`n`个unsigned int分量的向量 |
| `dvecn` | 包含`n`个double分量的向量       |

&emsp;&emsp;当前我们的目标是给三角形着色，这需要用到顶点着色器和片段着色器。我们首先需要一个顶点着色器的代码:

```c++
#version 330 core			//声明版本
layout(location = 0) in vec3 position;	//position变量的属性位置值为0
out vec4 vertexColor;
void main(){
  gl_Position = vec4(position.x,position.y,position.z,1.0);	 //为gl_Position赋值
  vertexColor = vec4(0.5f, 0.0f, 0.0f, 1.0f);	//为输出变量赋值
}
```

&emsp;&emsp;还有一个片段着色器:

```c++
#version 330 core
in vec4 vertexColor; 	//接收顶点着色器传来的值
out vec4 fragmentColor;	//定义片段着色器输出
void main(){
  fragmentColor = vertexColor;   //为输出赋值
}
```

&emsp;&emsp;我们可以将这两个着色器程序写入文件，然后从文件里读出来编译，也可以像我这样直接写成语句。

```c++
    GLchar const* vertexShaderSource =
            "#version 330 core\n"
                    "layout(location = 0) in vec3 position;\n"
      				"out vec4 vertexColor;\n"
                    "void main(){\n"
                    "   gl_Position = vec4(position.x,position.y,position.z,1.0);\n"
      				"	vertexColor = vec4(0.5f, 0.0f, 0.0f, 1.0f);\n"
                    "}\n";


    GLchar const* fragmentShaderSource =
            "#version 330 core\n"
      				"in vec4 vertexColor;\n"
                    "out vec4 fragmentColor;\n"
                    "void main(){\n"
                    "   fragmentColor = vertexColor;\n"
                    "}\n";
```

&emsp;&emsp;写好了着色器程序，接下来是要编译链接他

```c++
    //创建缓冲器
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);

    //获得缓冲器对象
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glShaderSource(fragmentShader , 1 , &fragmentShaderSource , NULL);

    //编译缓冲器
    glCompileShader(vertexShader);
    glCompileShader(fragmentShader);

    //检查着色器

    GLint Result = GL_FALSE;
    GLint InfoLogLength;

    glGetShaderiv(vertexShader , GL_COMPILE_STATUS , &Result);
    glGetShaderiv(vertexShader , GL_INFO_LOG_LENGTH , &InfoLogLength);
    if(InfoLogLength > 0){
        std::vector<char> ShaderErrorMessage(InfoLogLength+1);
        glGetShaderInfoLog(vertexShader, InfoLogLength, NULL, &ShaderErrorMessage[0]);
        printf("%s\n", &ShaderErrorMessage[0]);
    } else {
        printf("VertexShader complier successed\n");
    }

    glGetShaderiv(fragmentShader , GL_COMPILE_STATUS , &Result);
    glGetShaderiv(fragmentShader , GL_INFO_LOG_LENGTH , &InfoLogLength);
    if(InfoLogLength > 0){
        std::vector<char> ShaderErrorMessage(InfoLogLength+1);
        glGetShaderInfoLog(fragmentShader, InfoLogLength, NULL, &ShaderErrorMessage[0]);
        printf("%s\n", &ShaderErrorMessage[0]);
    } else {
        printf("FragmentShader complier successed\n");
    }

    //连接程序
    GLuint program = glCreateProgram();
    glAttachShader(program , vertexShader);
    glAttachShader(program , fragmentShader);
    glLinkProgram(program);

    //
    glDetachShader(program , vertexShader);
    glDetachShader(program , fragmentShader);


    //检查program

    glGetProgramiv(program, GL_LINK_STATUS, &Result);
    glGetProgramiv(program, GL_INFO_LOG_LENGTH, &InfoLogLength);
    if ( InfoLogLength > 0 ){
        std::vector<char> ProgramErrorMessage(InfoLogLength+1);
        glGetProgramInfoLog(program, InfoLogLength, NULL, &ProgramErrorMessage[0]);
        printf("%s\n", &ProgramErrorMessage[0]);
    } else {
        printf("Link successed!\n");
    }

    //删除着色器
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
```

&emsp;&emsp;这里我们注意的是检查着色器那里，我们用了`glGetShaderiv`获得着色器编译信息`GL_COMPILE_STATUS`和

`GL_INFO_LOG_LENGTH`，当着色器出错，`GL_INFO_LOG_LENGTH > 0`，就要查看着色器编译过程中的错误信息了，使用`glGetShaderInfoLog`获得。

&emsp;&emsp;这些执行完成后，我们的代码基本已经成了，在main的循环里使用这个已经链接着色器的program就可以了。

```c++
glUseProgram(program);
```

&emsp;&emsp;在最后修改的时候我的完整代码经过了一些更改，如下:

```c++
#include <iostream>
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>
#include <vector>

#define GLFW_CONTEXT_VERSION 3
#define WINDOW_WIDTH 800
#define WINDOW_HEIGHT 600

void InitGLFW(){
    /*初始化GLFW*/
    glfwInit();
    glfwWindowHint(GLFW_SAMPLES , 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, GLFW_CONTEXT_VERSION);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, GLFW_CONTEXT_VERSION);
    glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif
}


GLuint getProgram() {

    GLchar const* vertexShaderSource =
            "#version 330 core\n"
                    "layout(location = 0) in vec3 position;\n"
                    "void main(){\n"
                    "   gl_Position = vec4(position.x,position.y,position.z,1.0);\n"
                    "}\n";


    GLchar const* fragmentShaderSource =
            "#version 330 core\n"
                    "out vec3 fragmentColor;\n"
                    "uniform vec3 color;\n"
                    "void main(){\n"
                    "   fragmentColor = color;\n"
                    "}\n";
    //创建缓冲器
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);

    //获得缓冲器对象
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glShaderSource(fragmentShader , 1 , &fragmentShaderSource , NULL);

    //编译缓冲器
    glCompileShader(vertexShader);
    glCompileShader(fragmentShader);

    //检查着色器

    GLint Result = GL_FALSE;
    GLint InfoLogLength;

    glGetShaderiv(vertexShader , GL_COMPILE_STATUS , &Result);
    glGetShaderiv(vertexShader , GL_INFO_LOG_LENGTH , &InfoLogLength);
    if(InfoLogLength > 0){
        std::vector<char> ShaderErrorMessage(InfoLogLength+1);
        glGetShaderInfoLog(vertexShader, InfoLogLength, NULL, &ShaderErrorMessage[0]);
        printf("%s\n", &ShaderErrorMessage[0]);
    } else {
        printf("VertexShader complier successed\n");
    }

    glGetShaderiv(fragmentShader , GL_COMPILE_STATUS , &Result);
    glGetShaderiv(fragmentShader , GL_INFO_LOG_LENGTH , &InfoLogLength);
    if(InfoLogLength > 0){
        std::vector<char> ShaderErrorMessage(InfoLogLength+1);
        glGetShaderInfoLog(fragmentShader, InfoLogLength, NULL, &ShaderErrorMessage[0]);
        printf("%s\n", &ShaderErrorMessage[0]);
    } else {
        printf("FragmentShader complier successed\n");
    }

    //连接程序
    GLuint program = glCreateProgram();
    glAttachShader(program , vertexShader);
    glAttachShader(program , fragmentShader);
    glLinkProgram(program);

    //
    glDetachShader(program , vertexShader);
    glDetachShader(program , fragmentShader);


    //检查program

    glGetProgramiv(program, GL_LINK_STATUS, &Result);
    glGetProgramiv(program, GL_INFO_LOG_LENGTH, &InfoLogLength);
    if ( InfoLogLength > 0 ){
        std::vector<char> ProgramErrorMessage(InfoLogLength+1);
        glGetProgramInfoLog(program, InfoLogLength, NULL, &ProgramErrorMessage[0]);
        printf("%s\n", &ProgramErrorMessage[0]);
    } else {
        printf("Link successed!\n");
    }

    //删除着色器
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    return program;
}


glm::vec3 getRGB(){

    static int r = 0;
    static int g = 128;
    static int b = 255;
    static int dr = 1;
    static int dg = 1;
    static int db = 1;

    r += dr;
    g += dg;
    b += db;
    if (r > 255 || r < 0) dr = -dr;
    if (g > 255 || g < 0) dg = -dg;
    if (b > 255 || b < 0) db = -db;

    return glm::vec3(r , g , b);
}

int main() {

    InitGLFW();

    /*创建一个窗口对象*/
    GLFWwindow *window = glfwCreateWindow(WINDOW_WIDTH, WINDOW_HEIGHT, "Triangle", NULL, NULL);
    glfwMakeContextCurrent(window);
    if (!window) {
        std::cout << "faild create glfw window" << std::endl;
        return -1;
    }

    /*设置GLEW*/
    glewExperimental = GL_TRUE;
    if (glewInit() != GLEW_OK) {
        std::cout << "faild to init glew" << std::endl;
        return -2;
    }

    glfwSetInputMode(window, GLFW_STICKY_KEYS, GL_TRUE);

    GLuint program = getProgram();


    /*顶点坐标*/
    GLfloat vertices[] = {
            -0.5f, -0.5f, 0.0f,
            0.5f, -0.5f, 0.0f,
            0.0f, 0.5f, 0.0f
    };

    GLuint VBO;
    GLuint VAO;
    glGenVertexArrays(1 , &VAO);
    glGenBuffers(1, &VBO);

    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glVertexAttribPointer( 0 , 3 , GL_FLOAT , GL_FALSE , 0 , (void*)0);

    glEnableVertexAttribArray(0);
    glBindBuffer(1,VBO);
    glBindVertexArray(0);
    glDisableVertexAttribArray(0);

    while (!glfwWindowShouldClose(window)) {

        glClearColor(0.0f , 0.3f , 0.2f , 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        glm::vec3 v = getRGB();
        int location = glGetUniformLocation(program , "color");
        glUseProgram(program);
        glUniform3f(location , v.r / 255.0f , v.g / 255.0f , v.b / 255.0f);

        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES , 0 , 3);
        glBindVertexArray(0);


        glfwPollEvents();
        glfwSwapBuffers(window);
    }

    glDeleteVertexArrays(1,&VAO);
    glDeleteBuffers(1,&VBO);
    glDeleteProgram(program);
    glfwTerminate();
    return 0;
}
```

&emsp;&emsp;这个代码中使用了uniform变量

> &emsp;&emsp;uniform变量是外部application程序传递给(vertex和fragment)shader的变量。因此它是application通过函数glUniform()函数赋值的 。**在(vertex和fragment)shader程序内部，uniform变量就像是C语言里面的常量(const)，它不能被shader程序修改。**也就是说shader只能用不能改uniform变量。