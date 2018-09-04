---
title: 【OpenGL】22-几何着色器
date: 2018-03-12 14:16:47
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[几何着色器](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/09%20Geometry%20Shader/)

【英文版】[Geometry Shader](http://learnopengl.com/#!Advanced-OpenGL/Geometry-Shader)

#### 学习记录

&emsp;&emsp;我们这么多篇文章以来，着色器是很常用的东西了，在之前还介绍了着色器中一些变量以及接口块等使用技巧。而我们的着色器一直是顶点着色器，片段着色器两种。使用顶点着色器传递定点数据，使用片段着色器着色。而这篇文章中我们将介绍位于顶点着色器与片段着色器之间的可选着色器：[几何着色器（Geometry Shader）](https://www.khronos.org/opengl/wiki/Geometry_Shader)。

<!--more-->

&emsp;&emsp;几何着色器一般结构如下：

```c++
#version 420 core     //版本
layout (input_primitive) in;	//input
layout (output_primitive, max_vertices = ver_count) out; //output

// main function
void main() {
    ...
}
```

&emsp;&emsp;我们在几何着色器中首先需要定义两个变量，使用 layout 关键字，一个输出 out ，一个输入 in ，在 in 中我们接收一个图元作为属性。

> &emsp;&emsp;The input_primitive type *must* match the primitive type for the vertex stream provided to the GS. If [Tessellation](https://www.khronos.org/opengl/wiki/Tessellation) is enabled, then the primitive type is specified by the [Tessellation Evaluation Shader](https://www.khronos.org/opengl/wiki/Tessellation_Evaluation_Shader)'s output qualifiers. If Tessellation is not enabled, then the primitive type is provided by the [drawing command](https://www.khronos.org/opengl/wiki/Vertex_Rendering) that renders with this shader program. The valid values for input_primitive, along with the valid OpenGL [primitive types](https://www.khronos.org/opengl/wiki/Primitive) or tessellation forms, are:
>
> | GS input            | OpenGL primitives                                   | [TES parameter](https://www.khronos.org/opengl/wiki/Tessellation_Evaluation_Shader#Tessellation_options) | vertex count |
> | ------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------ |
> | points              | GL_POINTS                                           | point_mode                                                   | 1            |
> | lines               | GL_LINES, GL_LINE_STRIP, GL_LINE_LIST               | isolines                                                     | 2            |
> | lines_adjacency     | GL_LINES_ADJACENCY, GL_LINE_STRIP_ADJACENCY         | N/A                                                          | 4            |
> | triangles           | GL_TRIANGLES, GL_TRIANGLE_STRIP, GL_TRIANGLE_FAN    | triangles, quads                                             | 3            |
> | triangles_adjacency | GL_TRIANGLES_ADJACENCY, GL_TRIANGLE_STRIP_ADJACENCY | N/A                                                          | 6            |
>
> &emsp;&emsp;The vertex count is the number of vertices that the GS receives per-input primitive.
>
> &emsp;&emsp;The output_primitive must be one of the following:
>
> - points
> - line_strip
> - triangle_strip
>
> &emsp;&emsp;These work exactly the same way their counterpart OpenGL rendering modes do. To output individual triangles or lines, simply use EndPrimitive (see below) after emitting each set of 3 or 2 vertices.

&emsp;&emsp;在 in 变量中我们接受一个图元，之后经过几何着色器的拓展我们可以将其转换为其他图元类型并且会传给片段着色器 max_vertices = ver_count 个顶点（请注意我们在这里定义了最大顶点个数，当其后的代码中超过最大数量后的定点会被抛弃）。

&emsp;&emsp;除了我们需要显性定义的 in 和 out 变量外，我们在集合着色器中也有用于输入的内建（built-in）接口块：

```c++
// input built-in variables
in gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
} gl_in[];
```

&emsp;&emsp;这是一个和顶点着色器中的 gl_Position 相似的内建变量，我们无需定义直接使用即可。使用方法如下：

```c++
#version 420 core

layout(points) in;
layout(line_strip , max_vertices = 4) out;

void main(){
	gl_Position = gl_in[0].gl_Position + vec4(1.0f , 1.0f , 0.0f , 0.0f);	//右上
	EmitVertex();

	gl_Position = gl_in[0].gl_Position + vec4(1.0f , -1.0f , 0.0f , 0.0f);  //右下
	EmitVertex();

	gl_Position = gl_in[0].gl_Position + vec4(-1.0f , -1.0f , 0.0f , 0.0f);	//左下
	EmitVertex();

	gl_Position = gl_in[0].gl_Position + vec4(-1.0f , 1.0f , 0.0f , 0.0f);	//左上
	EmitVertex();

	EndPrimitive();
}
```

&emsp;&emsp;在这段代码中，我们定义了输入进来的图元为 points ，然后输入为 line_strip ，且最大输出的定点数为 4 个。之后，我们使用 gl_in 这个内建接口块，从顶点着色器中读取 gl_Position 并复制给几何着色器中的内建变量。请注意我们在每一个 gl_Position 的赋值后都使用了 EmitVertex 函数，在 main 的结尾调用了 EndPrimitive 函数。

&emsp;&emsp;每次我们调用EmitVertex时，gl_Position中的向量会被添加到图元中来。当EndPrimitive被调用时，所有发射出的(Emitted)顶点都会合成为指定的输出渲染图元。在一个或多个EmitVertex调用之后重复调用EndPrimitive能够生成多个图元。在我们的例子代码中，我们为图元添加了四个顶点，并且将其合成为指定的渲染图元。最后效果如下：

![Geometry Shader](https://image.ibb.co/hAkHL7/image.png)

&emsp;&emsp;我们可以用几何着色器去增加顶点，修改图元等等。