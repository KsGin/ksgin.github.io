---
title: 【OpenGL】23-实例化
date: 2018-03-13 09:28:54
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[实例化](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/10%20Instancing/)

【英文版】[Instancing](http://learnopengl.com/#!Advanced-OpenGL/Instancing)

#### 学习记录

&emsp;&emsp;在这篇文章中，我们来介绍一个用于绘制大量模型物体的技术：**实例化（Instancing）**

<!--more-->

&emsp;&emsp;是否还记得我们在 [高级 GLSL ](https://blog.ksgin.com/2018/03/11/【opengl】21-高级glsl-uniform与块元素/#more) 这一篇文章中，我们绘制的四个正方体模型：

```c++
//绘制第一个模型
someShader1.use();
someShader1.setMat4("projection", projection);
someShader1.setMat4("view", camera.GetViewMatrix());
someShader1.setMat4("model", model1);
glBindVertexArray(vertexArrayObject);
glDrawArrays(GL_TRIANGLES, 0, 36);
glBindVertexArray(0);
//绘制第二个模型
someShader2.use();
someShader2.setMat4("projection", projection);
someShader2.setMat4("view", camera.GetViewMatrix());
someShader2.setMat4("model", model2);
glBindVertexArray(vertexArrayObject);
glDrawArrays(GL_TRIANGLES, 0, 36);
glBindVertexArray(0);
//绘制第三个模型
someShader3.use();
someShader3.setMat4("projection", projection);
someShader3.setMat4("view", camera.GetViewMatrix());
someShader3.setMat4("model", model3);
glBindVertexArray(vertexArrayObject);
glDrawArrays(GL_TRIANGLES, 0, 36);
glBindVertexArray(0);
//绘制第四个模型
someShader4.use();
someShader4.setMat4("projection", projection);
someShader4.setMat4("view", camera.GetViewMatrix());
someShader4.setMat4("model", model4);
glBindVertexArray(vertexArrayObject);
glDrawArrays(GL_TRIANGLES, 0, 36);
glBindVertexArray(0);
```

&emsp;&emsp;刚开始我们需要一个一个的传入 MVP ，然后一个一个的绘制，代码冗余不说效率也不高。之后我们使用 UBO （Uniform Buffer Object）优化了代码：

```c++
glBindVertexArray(vertexArrayObject);

glslShader1.use();
glslShader1.setMat4("model", model1);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader2.use();
glslShader2.setMat4("model", model2);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader3.use();
glslShader3.setMat4("model", model3);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader4.use();
glslShader4.setMat4("model", model4);
glDrawArrays(GL_TRIANGLES, 0, 36);

glBindVertexArray(0);
```

&emsp;&emsp;代码变成了这样，但我们仍然是调用多个绘制函数，如果我们的 Shader 再一样的话，那么应该是这样：

```c++
glBindVertexArray(vertexArrayObject);

glslShader.use();

glslShader.setMat4("model", model1);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader.setMat4("model", model2);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader.setMat4("model", model3);
glDrawArrays(GL_TRIANGLES, 0, 36);

glslShader.setMat4("model", model4);
glDrawArrays(GL_TRIANGLES, 0, 36);

glBindVertexArray(0);
```

&emsp;&emsp;亦或者使用 for 循环：

```c++
glBindVertexArray(vertexArrayObject);
glslShader.use();
for (const auto& model : models) {
	glslShader.setMat4("model", model);
	glDrawArrays(GL_TRIANGLES, 0, 36);
}
glBindVertexArray(0);
```

&emsp;&emsp;这样再多的模型我们也只需要在 models 容器里添加 model 就可以了，代码的冗余似乎解决了，但是效率并没有提高。与绘制顶点本身相比，使用 glDrawArrays 或 glDrawElements 函数告诉 GPU 去绘制顶点数据会消耗更多的性能，因为 OpenGL 在绘制顶点数据之前需要做很多准备工作（比如告诉 GPU 该从哪个缓冲读取数据，从哪寻找顶点属性，而且这些都是在相对缓慢的 CPU 到 GPU 总线(CPU to GPU Bus)上进行的）。所以，即便渲染顶点非常快，命令 GPU 去渲染却未必。

&emsp;&emsp;如果我们能够将数据一次性发送给 GPU ，然后使用一个绘制函数让 OpenGL 利用这些数据绘制多个物体，就会更方便了。这就是**实例化（Instancing）**。通过实例化技术，我们可以快速的绘制出大量的同类模型。

&emsp;&emsp;用了这么大的篇幅去介绍为什么使用实例化技术，那么这个究竟如何使用呢？

&emsp;&emsp;在渲染多个相同的物体时，我们需要提供他的位置（否则在一个位置渲染再多的相同物体都是没有意义的），首先我们在顶点着色器里定义位置数组。

```c++
uniform vec3 positions[40];
```

&emsp;&emsp;我们定义了一个大小为 40 的位置索引，这时候我们需要用提供的当前渲染的物体的索引来选择相应的位置，索性 GLSL 给我们定义了这么一个内建变量：[gl_InstanceID](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/gl_InstanceID.xhtml) 。

> ## Name
>
> gl_InstanceID — contains the index of the current primitive in an instanced draw command 
>
> ## Description
>
> ​     	`gl_InstanceID` is a vertex language input variable that holds the integer index of the current primitive in an instanced draw command such as `glDrawArraysInstanced`. If the current primitive does not originate from an instanced draw command, the value of `gl_InstanceID` is zero.

&emsp;&emsp;gl_InstanceID 给了我们当前所渲染的物体的 ID，从 0 开始，跟随我们渲染的物体数而自增。所以在顶点着色器代码中，使用 gl_InstanceID 索引位置数组，并加到 gl_Position 上。

```c++
gl_Position = projection * view * model * vec4(aPos + positions[gl_InstanceID] , 1.0f);
```

&emsp;&emsp;此时我们的顶点着色器应该是这个样子：

```c++
#version 420 core
layout (location = 0) in vec3 aPos;

uniform mat4 model;
uniform vec3 positions[40];

layout (std140) uniform VP{
	mat4 projection;
	mat4 view;
};

void main(){
	gl_Position = projection * view * model * vec4(aPos + positions[gl_InstanceID] , 1.0f);
	gl_PointSize = abs(aPos.z) * 20;
};
```

&emsp;&emsp;着色器的代码完了，现在我们需要在 OpenGL 代码中为着色器的 positions[] 变量赋值，首先我们创建一个 positions 数组：

```c++
std::vector<glm::vec3> positions(0);

for (float i = 1; i < 10.0; i++) {
	positions.push_back(glm::vec3(3.0f, 3.0f, 0.0f) * i);
	positions.push_back(glm::vec3(-3.0f, -3.0f, 0.0f) * i);
	positions.push_back(glm::vec3(3.0f, -3.0f, 0.0f) * i);
	positions.push_back(glm::vec3(-3.0f, 3.0f, 0.0f) * i);
}
```

&emsp;&emsp;之后，我们在调用 shader.use() 之后，我们为着色器变量写值：

```c++
glslShader.use();
for (auto i = 0; i < positions.size(); ++i) {
	glslShader.setVec3("positions[" + std::to_string(i) + "]", positions[i]);
}
glslShader.setMat4("model", model);
```

&emsp;&emsp;最后，我们调用实例化 Draw 方法：`glDrawArraysInstanced(GL_TRIANGLES, 0, 36, 40);` ，请注意我们最后一个参数是要渲染的物体数量。效果如下：

![intancing](https://image.ibb.co/eGGj07/image.png)

&emsp;&emsp;如果你没有成功，请看这里：[源代码](https://github.com/KsGin/LearnOpenGL/tree/master/Instancing)

