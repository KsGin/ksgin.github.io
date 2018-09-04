---
title: 【OpenGL】2-这是一个三角形1
date: 2017-12-12 12:53:20
tags:
    - OpenGL
    - Computer graphics
---

【OpenGL菜鸟历程】

&emsp;&emsp;上次的一堆代码就显示了个窗口，虽然感觉很麻烦但是还不足以让我放弃。继续继续，跟着教程上的下一篇文章，我开始接触了Shader，而这一次就是要用Shader画一个三角。

<!--more-->

【代码O.O】

&emsp;&emsp;这次的项目起名为HelloTriangle，在画三角之前，我们自然是要先把窗口写出来。为了使得代码看起来更清晰，这次将代码封装成函数。

```c++
//我们的glfw的初始化方法
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
```

&emsp;&emsp;之后就是主函数

```c++
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

    while (!glfwWindowShouldClose(window)) {

        glClearColor(0.0f , 0.3f , 0.2f , 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
      
      	//do something
      	
        glfwPollEvents();
        glfwSwapBuffers(window);
    }

    return 0;
```

&emsp;&emsp;这样我们首先得到了一个简单的窗口，接下来才是重点。我们首先给出一个三角形的顶点坐标

```c++
	/*顶点坐标*/
    GLfloat vertices[] = {
            -0.5f, -0.5f, 0.0f,
            0.5f, -0.5f, 0.0f,
            0.0f, 0.5f, 0.0f
    };
```

&emsp;&emsp;这时，需要了解两个概念 VAO (Vertex Array Object)，VBO(Vertex Buffer Object)，不知道这两个东西的话可以看[VAO与VBO的前世今生](http://www.photoneray.com/opengl-vao-vbo/)。

&emsp;&emsp;我们创建这两个对象并进行初始化绑定等操作。

```c++
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
```

&emsp;&emsp;请先不要追究这些方法的具体含义，我们将在未来一篇文章中解释并且对VAO，VBO进行详细的介绍和分析。现在我们写了这一大堆代码，然后呢？

```c++
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES , 0 , 3);
    glBindVertexArray(0);
```

&emsp;&emsp;我们在窗口的代码的循环里，加入这几行，将VAO进行绑定，然后画出来这个顶点数组，再解绑（绑定0就是解绑）。OK，现在代码应该已经好了，可是点击运行后，却没有按照我们的意愿出现一个三角形。

&emsp;&emsp;当然出现了，只是我们没有看到，因为他的颜色并没有和窗口的主色调分开，所以我们看不到而已。下一篇文章将会讲解另一个东西，Shader（着色器）。