---
title: 【OpenGL】5-彩色立方体
date: 2018-01-21 19:51:36
tags: 
    - OpenGL
    - Computer graphics
---

#### 参考教程

【中文版】[第四课：彩色立方体](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-4-a-colored-cube/)

【英文版】[Tutorial 4 : A Colored Cube](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-4-a-colored-cube/)

&emsp;&emsp;在之前的几篇笔记中，最后的结果仅仅只是画出来一个三角形，但是真实的计算机图形怎么可能是一个三角形可以概括的。在仅支持Draw Triangle的OpenGL里，我们将学着使用三角形去构造出复杂的图形等等。在这篇笔记中，主要学会使用12个三角形去构造一个正立方体。

<!--more-->

&emsp;&emsp;我们都知道，正立方体有6个正方形面，每个面可以用两个三角形拼出来（事实上我们画立方体外部摄像视角的静态实体时是用不到那么多三角形的，因为有一部分属于摄像机盲区），总共需要12个三角形。在这里，我们给出12个三角形的顶点数据。

```c++
// Our vertices. Tree consecutive floats give a 3D vertex; Three consecutive vertices give a triangle.
// A cube has 6 faces with 2 triangles each, so this makes 6*2=12 triangles, and 12*3 vertices
    static const GLfloat g_vertex_buffer_data[] = {
            -1.0f,-1.0f,-1.0f,
            -1.0f,-1.0f, 1.0f,
            -1.0f, 1.0f, 1.0f,
            1.0f, 1.0f,-1.0f,
            -1.0f,-1.0f,-1.0f,
            -1.0f, 1.0f,-1.0f,
            1.0f,-1.0f, 1.0f,
            -1.0f,-1.0f,-1.0f,
            1.0f,-1.0f,-1.0f,
            1.0f, 1.0f,-1.0f,
            1.0f,-1.0f,-1.0f,
            -1.0f,-1.0f,-1.0f,
            -1.0f,-1.0f,-1.0f,
            -1.0f, 1.0f, 1.0f,
            -1.0f, 1.0f,-1.0f,
            1.0f,-1.0f, 1.0f,
            -1.0f,-1.0f, 1.0f,
            -1.0f,-1.0f,-1.0f,
            -1.0f, 1.0f, 1.0f,
            -1.0f,-1.0f, 1.0f,
            1.0f,-1.0f, 1.0f,
            1.0f, 1.0f, 1.0f,
            1.0f,-1.0f,-1.0f,
            1.0f, 1.0f,-1.0f,
            1.0f,-1.0f,-1.0f,
            1.0f, 1.0f, 1.0f,
            1.0f,-1.0f, 1.0f,
            1.0f, 1.0f, 1.0f,
            1.0f, 1.0f,-1.0f,
            -1.0f, 1.0f,-1.0f,
            1.0f, 1.0f, 1.0f,
            -1.0f, 1.0f,-1.0f,
            -1.0f, 1.0f, 1.0f,
            1.0f, 1.0f, 1.0f,
            -1.0f, 1.0f, 1.0f,
            1.0f,-1.0f, 1.0f
    };
```

&emsp;&emsp;而具体的立方体画图过程，则是和画三角形差不多，给予每个三角形不同的颜色，就可以完成立方体的构造。

&emsp;&emsp;而在这里，我们需要引入一个新的概念观察视角（摄像机视角），在我所学习的教程的前一章，详细的介绍了这些概念以及矩阵的一些基本运算和矩阵运算在OpenGL中和GLSL的操作 [﻿第三课：矩阵](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-3-matrices/)

&emsp;&emsp;对于OpenGL中的Buffer缓冲，在之前的学习中也都学过，这里不再描述。唯一与之前不同的是我们这次画的是12个三角形，所以在`glDrawArrays(GL_TRIANGLES, 0, 12*3);`的时候需要将之前的3改为12*3。此时应该有如下效果：（黑色立方体）

![WX20180121-201214@2x](https://ws2.sinaimg.cn/large/006tNc79ly1fnohq10odvj31km168wfw.jpg)

&emsp;&emsp;下一步则是需要对这一坨黑色的东西进行着色。在这里我们使用在OpenGL代码里定义顶点数组，然后使用顶点着色器程序进行读取，并传入到片段着色器中进行着色的方式。首先定义顶点数组。

```c++
    // One color for each vertex. They were generated randomly.
    static const GLfloat g_color_buffer_data[] = {
            0.583f,  0.771f,  0.014f,
            0.609f,  0.115f,  0.436f,
            0.327f,  0.483f,  0.844f,
            0.822f,  0.569f,  0.201f,
            0.435f,  0.602f,  0.223f,
            0.310f,  0.747f,  0.185f,
            0.597f,  0.770f,  0.761f,
            0.559f,  0.436f,  0.730f,
            0.359f,  0.583f,  0.152f,
            0.483f,  0.596f,  0.789f,
            0.559f,  0.861f,  0.639f,
            0.195f,  0.548f,  0.859f,
            0.014f,  0.184f,  0.576f,
            0.771f,  0.328f,  0.970f,
            0.406f,  0.615f,  0.116f,
            0.676f,  0.977f,  0.133f,
            0.971f,  0.572f,  0.833f,
            0.140f,  0.616f,  0.489f,
            0.997f,  0.513f,  0.064f,
            0.945f,  0.719f,  0.592f,
            0.543f,  0.021f,  0.978f,
            0.279f,  0.317f,  0.505f,
            0.167f,  0.620f,  0.077f,
            0.347f,  0.857f,  0.137f,
            0.055f,  0.953f,  0.042f,
            0.714f,  0.505f,  0.345f,
            0.783f,  0.290f,  0.734f,
            0.722f,  0.645f,  0.174f,
            0.302f,  0.455f,  0.848f,
            0.225f,  0.587f,  0.040f,
            0.517f,  0.713f,  0.338f,
            0.053f,  0.959f,  0.120f,
            0.393f,  0.621f,  0.362f,
            0.673f,  0.211f,  0.457f,
            0.820f,  0.883f,  0.371f,
            0.982f,  0.099f,  0.879f
    };
```

&emsp;&emsp;这个和顶点数组是一样的，具体数字都是随便写进去的，你愿意的话全是0.0f也可以（不过还是黑色）。然后缓冲的绑定也是和之前一样。

```c++
GLuint colorbuffer;
glGenBuffers(1, &colorbuffer);
glBindBuffer(GL_ARRAY_BUFFER, colorbuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(g_color_buffer_data), g_color_buffer_data, GL_STATIC_DRAW);
glVertexAttribPointer(
                1,                                // attribute. No particular reason for 1, but must match the layout in the shader.
                3,                                // size
                GL_FLOAT,                         // type
                GL_FALSE,                         // normalized?
                0,                                // stride
                (void*)0                          // array buffer offset
        );
```

&emsp;&emsp;这时候你应该可以看到这样一个结果：

![WX20180121-203226@2x](https://ws2.sinaimg.cn/large/006tNc79ly1fnoias5wnwj31ks10k0v9.jpg)

&emsp;&emsp;搞定！

&emsp;&emsp;[Github地址](https://github.com/KsGin/LearnOpenGL/tree/master/AColoredCube)

