---
title: 【OpenGL】20-高级GLSL-内建变量
date: 2018-03-11 13:17:54
tags:
	- OpenGL
	- Computer graphics
---

#### 教程地址

【中文版】[高级GLSL](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/08%20Advanced%20GLSL/)

【英文版】[Advanced GLSL](http://learnopengl.com/#!Advanced-OpenGL/Advanced-GLSL)

#### 学习记录

&emsp;&emsp;[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language) ，OpenGL Shading Language 。我们之前的代码中用了不少，但都是很短的代码，不管是顶点着色器或者是片段着色器都是简单的几行，我们对其了解也很少，这篇文章中就专门介绍 GLSL 。我们将会讨论一些有趣的内建变量(Built-in Variable)，管理着色器输入和输出的新方式以及一个叫做Uniform缓冲对象(Uniform Buffer Object)的有用工具。

<!--more-->

&emsp;&emsp;所谓内建变量，其实就是着色器已经给我们定义好的变量，比如 gl_Position ，这个我们所有顶点着色器都使用过的变量，他定义输出了顶点的位置。

&emsp;&emsp;除了 gl_Position 外，GLSL 内建了很多其他变量，比如 gl_PointSize ，他允许我们修改每一个点的大小，当我们使用 GL_POINTS 为图元进行绘制时可以使用。

&emsp;&emsp;gl_PointSize 的使用比较简单，当在 OpenGL 中启用 gl_PontSize 之后，我们直接在顶点着色器里调用就是了。（此时我们需要将 glDrawArrays() 的绘制参数改为 GL_POINTS）

```c++
#version 420 core
layout (location = 0) in vec3 aPos;

uniform mat4 model;
uniform mat4 projection;
uniform mat4 view;

void main(){
	gl_Position = projection * view * model * vec4(aPos , 1.0f);
	gl_PointSize = aPos.z * 10;
}
```

&emsp;&emsp;在这里我们将其设置为顶点 z 轴（深度）的十倍，这样有一种越靠近我们顶点越大的感觉。

![points](https://image.ibb.co/m0nNk7/image.png)

&emsp;&emsp;顶点着色器中还有一个内建变量 gl_VertexID ，用来访问我们当前所绘制的顶点的索引，当使用索引绘制时，即 glDrawElements 时他会存储当前索引，当使用 glDrawArrays 绘制时会存储当前已渲染的图元个数。这个变量可以被我们使用已做出一些有趣的效果，用刚才绘制的点的代码来示例，我们可以使每一个的颜色不同。

&emsp;&emsp;在顶点着色器中，我们使用 gl_VertexID 去设置一个到片段着色器的 vec4 变量，并在片段着色器中进行渲染。

```c++
#version 420 core
layout (location = 0) in vec3 aPos;

uniform mat4 model;
uniform mat4 projection;
uniform mat4 view;

out vec4 pColor;

void main(){
	gl_Position = projection * view * model * vec4(aPos , 1.0f);
	gl_PointSize = abs(aPos.z) * 10;
	pColor = vec4(gl_VertexID / 12.0f + 0.1f , gl_VertexID / 3.0f + 0.1f , gl_VertexID / 8.0f + 0.1f, 1.0f);
}
```

```c++
#version 420 core

out vec4 FragColor;
in vec4 pColor;

void main(){
	FragColor = pColor;
}
```

&emsp;&emsp;由于顶点 ID 不同，所以亮度不同，效果如下：

![color](https://image.ibb.co/iJsBdS/image.png)

&emsp;&emsp;前两个是顶点着色器的内建变量，现在我们来看下片段着色器中的变量 gl_FragCoord ，gl_FragCoord 的 x 和 y 分量分别是屏幕的 x , y 坐标，z 则是深度变量，相当于 z 轴。以此，我们可以通过这个变量去对模型在屏幕的各个部位进行不同的着色，或者去实现其他有趣的效果。

&emsp;&emsp;我们将顶点分别放在屏幕中心两侧（x 轴0点两侧），并且略微调大了 gl_PointSize 使得等下效果更加显著。现在，我们在片段着色器里进行修改。

```c++
#version 420 core

out vec4 FragColor;
in vec4 pColor;

void main(){
	if(gl_FragCoord.x < 960){
		FragColor = vec4(1.0f,1.0f,1.0f,0.0f);
	} else {
		FragColor = pColor;
	}
}
```

&emsp;&emsp;由于我们的屏幕 width 值为1920，所以960正好是一半，这样我们将凡是在屏幕 x 轴左边的全部渲染为白色，右边则是刚才的颜色。效果如下：

![fragcoord](https://image.ibb.co/b436TS/image.png)

&emsp;&emsp;可以看到，即便是一个点，只要依旧可以进行不同的着色。

&emsp;&emsp;接下来一个是 gl_FrontFacing 变量，这是一个布尔值变量，他告诉了我们当前的片段属于正面还是背面，以此我们也可以进行着色，例如对箱子外部和内部渲染不同的纹理等。（由于我这里代码只有点的，这个就不演示了）用法也极为简单：

```c++
#version 420 core

out vec4 FragColor;
in vec4 pColor;

void main(){
	if(gl_FrontFacing){
		//first color
	} else {
		//second color
	}
}
```

&emsp;&emsp;另外一个变量 gl_FragDepth 允许我们在片段着色器里修改深度值，然而，由我们自己设置深度值有一个很大的缺点，只要我们在片段着色器中对gl_FragDepth进行写入，OpenGL就会（像[深度测试](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/01%20Depth%20testing/)小节中讨论的那样）禁用所有的提前深度测试(Early Depth Testing)。它被禁用的原因是，OpenGL无法在片段着色器运行**之前**得知片段将拥有的深度值，因为片段着色器可能会完全修改这个深度值。在写入gl_FragDepth时，你就需要考虑到它所带来的性能影响。