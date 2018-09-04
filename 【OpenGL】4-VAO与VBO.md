---
title: 【OpenGL】4-VAO与VBO
date: 2017-12-14 15:10:55
tags:     
    - OpenGL
    - Computer graphics
---

#### VAO与VBO来历

&emsp;&emsp;首先给出参考资料：[VAO与VBO的前世今生](http://www.photoneray.com/opengl-vao-vbo/)

&emsp;&emsp;VAO（Vertex Arrays Object）：顶点数组对象

&emsp;&emsp;VBO（Vertex Buffer Object）：顶点缓冲对象

&emsp;&emsp;首先由其名称就可以知道，这两个是用来操作顶点的，我们的显示器就是一个大的网格，而我们所看到的界面便是顶点和顶点周围所对应的颜色组成的，在上篇文章中我们使用了两个着色器，顶点着色器与片段着色器，就是为了着色。

<!--more-->![GLSL中VAO和VBO的使用](http://img.blog.csdn.net/20130731232253265?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2lsYW5ncXVhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;[GLSL中VAO和VBO的使用](http://blog.csdn.net/silangquan/article/details/9674371)

&emsp;&emsp;看看我们画三角形的时候使用VAO和VBO的代码。

```c++
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

	while (!glfwWindowShouldClose(window)) {
      	//do something
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES , 0 , 3);
        glBindVertexArray(0);
        //do something
    }

	glDeleteVertexArrays(1,&VAO);
    glDeleteBuffers(1,&VBO);
```

&emsp;&emsp;在这份代码中，我们使用了VAO,VBO，传输三角形顶点信息。

&emsp;&emsp;VAO/VBO实质是为了解决**传输效率**而做的优化手段。VBO是为了均衡数据的传输效率与灵活修改性；VAO的本质是储存绘制状态，简化绘制代码（VAO是状态对象，不存储顶点数组信息）。