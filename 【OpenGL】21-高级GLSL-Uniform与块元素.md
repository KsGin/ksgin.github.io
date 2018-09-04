---
title: 【OpenGL】21-高级GLSL-Uniform与块元素
date: 2018-03-11 19:19:09
tags:
	- OpenGL
	- Computer graphics
---

#### 教程地址

【中文版】[高级GLSL](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/08%20Advanced%20GLSL/)

【英文版】[Advanced GLSL](http://learnopengl.com/#!Advanced-OpenGL/Advanced-GLSL)

#### 学习记录

&emsp;&emsp;上篇文章中我们介绍了 GLSL 的内建变量，并简单的实现了几个例子。这篇文章中我们将介绍**接口块（Interface Block）**与 **Uniform 缓冲对象（Uniform Buffer Object）**。

<!--more-->

&emsp;&emsp;接口块实际上就是 GLSL 中的结构体，我们使用它将多个同类变量放在一个结构中进行传递，定义如下：

```c++
Block_Name {
    ...
} Intance_Name;
```

&emsp;&emsp;Block_Name 是接口块在传递过程中使用的名字，如果我们需要将数据从顶点着色器发送到片段着色器，那么必须保证 Block_Name 相同，这和普通的变量是相同的。举例如下：

```c++
//顶点着色器中
out Vertex_To_Fragment_Object {
    ...
    vec4 pColor;
    ...
} VOO;

VOO.pColor = ...;
```

```c++
//片段着色器中
in Vertex_To_Fragment_Object {
    ...
    vec4 pColor;
    ...
} FII;

... = FII.pColor;
```

&emsp;&emsp;接口块其实就是个结构体嘛！

&emsp;&emsp;下来介绍 Uniform 缓冲对象：

> &emsp;&emsp;OpenGL 为我们提供了一个叫做 Uniform 缓冲对象(Uniform Buffer Object)的工具，它允许我们定义一系列在多个着色器中相同的**全局**Uniform变量。当使用Uniform缓冲对象的时候，我们只需要设置相关的uniform**一次**。当然，我们仍需要手动设置每个着色器中不同的uniform。并且创建和配置Uniform缓冲对象会有一点繁琐。
>
> &emsp;&emsp;因为Uniform缓冲对象仍是一个缓冲，我们可以使用glGenBuffers来创建它，将它绑定到GL_UNIFORM_BUFFER缓冲目标，并将所有相关的uniform数据存入缓冲。

&emsp;&emsp;如果说接口块是着色器之间传递数据的结构体，那么 Uniform Buffer Object 则是 OpenGL 代码与 着色器代码之间的结构体了。在我们之前的很多代码中，我们大量使用了 Uniform 变量，比如：

```c++
uniform mat4 model;
uniform mat4 projection;
uniform mat4 view;
```

&emsp;&emsp;而在这些代码中，一般情况下我们需要为每一个模型的着色器设置一次这个，然后再绘制的时候再大量的使用 glUniformMatrix4fv 方法为其赋值，所以经常看到这样的代码（我们使用了 Shader 类）：

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

&emsp;&emsp;可以看到，仅仅是 model 不同，我们就需要为每一个着色器赋值一遍，相对于而言 projection 和 view 几乎是不会因模型不同而异的。如果有几十个变量，几千个模型，那么我们就需要 几十*几千 行这样的冗余代码。而现在有了 UBO （Uniform Buffer Object），我们可以将其归类，统一绑定。

&emsp;&emsp;在着色器代码中我们定义 UBO ：

```c++
layout (std140) uniform VP{
	mat4 projection;
	mat4 view;
};
```

&emsp;&emsp;我们将多个模型都相同的数据绑定在一起（关于 std140 等下解释），在 OpenGL 代码中，需要创建相应的 UBO 缓冲：

```c++
unsigned int uboBlock;
glGenBuffers(1, &uboBlock);
glBindBuffer(GL_UNIFORM_BUFFER, uboBlock);
glBufferData(GL_UNIFORM_BUFFER, sizeof(glm::mat4) * 2, nullptr, GL_STATIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

&emsp;&emsp;Uniform 缓冲的构建与其他缓冲相同，不过我们并没有进行初始化数据而是设为 nullptr ，size 我们设置为两个 mat4 的大小（我们 uniform 中有两个 mat4 变量）。

> &emsp;&emsp;现在，每当我们需要对缓冲更新或者插入数据，我们都会绑定到uboExampleBlock，并使用glBufferSubData来更新它的内存。我们只需要更新这个Uniform缓冲一次，所有使用这个缓冲的着色器就都使用的是更新后的数据了。但是，如何才能让OpenGL知道哪个Uniform缓冲对应的是哪个Uniform块呢？
>
> &emsp;&emsp;在OpenGL上下文中，定义了一些绑定点(Binding Point)，我们可以将一个Uniform缓冲链接至它。在创建Uniform缓冲之后，我们将它绑定到其中一个绑定点上，并将着色器中的Uniform块绑定到相同的绑定点，把它们连接到一起。下面的这个图示展示了这个：
>
> ![img](https://learnopengl-cn.github.io/img/04/08/advanced_glsl_binding_points.png)
>
> &emsp;&emsp;你可以看到，我们可以绑定多个Uniform缓冲到不同的绑定点上。因为着色器A和着色器B都有一个链接到绑定点0的Uniform块，它们的Uniform块将会共享相同的uniform数据，uboMatrices，前提条件是两个着色器都定义了相同的Matrices Uniform块。
>
> &emsp;&emsp;为了将Uniform块绑定到一个特定的绑定点中，我们需要调用glUniformBlockBinding函数，它的第一个参数是一个程序对象，之后是一个Uniform块索引和链接到的绑定点。Uniform块索引(Uniform Block Index)是着色器中已定义Uniform块的位置值索引。这可以通过调用glGetUniformBlockIndex来获取，它接受一个程序对象和Uniform块的名称。

&emsp;&emsp;我们将我们的着色器中的 VP 缓冲绑定到绑定点 0 上（我们可以将多个着色器的 VP 缓冲对象绑定到一个绑定点，只要他们 VP 的内容相同，这正是我们的目的）：

```c++
const auto vpIndex = glGetUniformBlockIndex(glslShader.ID, "VP");   //获取着色器中的 uniform 缓冲对象位置索引
glUniformBlockBinding(glslShader.ID, vpIndex, 0); //将其绑定到 0 绑定点
```

&emsp;&emsp;在 GLSL 420 （即 OpenGL 4.2版本）之后，支持直接在定义 Uniform 缓冲对象的时候绑定：

```c++
layout(std140, binding = 2) uniform VP { 
	mat4 projection;
	mat4 view;
};
```

&emsp;&emsp;现在将 OpenGL 代码中的 uboBlock 缓冲对象也绑定到 0 绑定点：

```c++
glBindBufferBase(GL_UNIFORM_BUFFER, 0, uboBlock); //将 uboBlock 对象（OpenGL中的缓冲对象）绑定到 0 绑定点
```

&emsp;&emsp;这样绑定之后，我们可以使用 uboBlock 对象来更新着色器中的 Uniform 缓冲对象。我们使用之前介绍过的缓冲部分填充函数为其添加数据：

```c++
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), &projection);
glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4) , sizeof(glm::mat4), &camera.GetViewMatrix());
```

&emsp;&emsp;正常使用。我们现在创建四个模型与四个着色器程序，每个着色器程序里仅仅是最后给的颜色不同，之后实现绑定，最后渲染：

```c++
const auto projection = glm::perspective(glm::radians(90.0f), static_cast<float>(WIDTH) / static_cast<float>(HEIGHT), 0.1f, 100.0f);
auto camera = Camera(glm::vec3(0.0f, 0.0f, 10.0f));
const auto model1 = glm::translate(glm::mat4(1.0f), glm::vec3(5.0f, 5.0f, 0.0f));
const auto model2 = glm::translate(glm::mat4(1.0f), glm::vec3(-5.0f, 5.0f, 0.0f));
const auto model3 = glm::translate(glm::mat4(1.0f), glm::vec3(5.0f, -5.0f, 0.0f));
const auto model4 = glm::translate(glm::mat4(1.0f), glm::vec3(-5.0f, -5.0f, 0.0f));
...
//填充数据
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), &projection);
glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4), sizeof(glm::mat4), &camera.GetViewMatrix());
do {
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
}
```

&emsp;&emsp;效果如下：

![uniform buffer object](https://image.ibb.co/c8xGTS/image.png)

&emsp;最后，我们来解释下我们着色器代码中 stb140 的含义：std140 代表着我们使用140布局，这是 uniform 内存布局方式的一种。

> &emsp;&emsp;Uniform块的内容是储存在一个缓冲对象中的，它实际上只是一块预留内存。因为这块内存并不会保存它具体保存的是什么类型的数据，我们还需要告诉OpenGL内存的哪一部分对应着着色器中的哪一个uniform变量。
>
> &emsp;&emsp;假设着色器中有以下的这个Uniform块：
>

```c++
layout (std140) uniform ExampleBlock
{
    float value;
    vec3  vector;
    mat4  matrix;
    float values[3];
    bool  boolean;
    int   integer;
};
```

> &emsp;&emsp;我们需要知道的是每个变量的大小（字节）和（从块起始位置的）偏移量，来让我们能够按顺序将它们放进缓冲中。每个元素的大小都是在OpenGL中有清楚地声明的，而且直接对应C++数据类型，其中向量和矩阵都是大的float数组。OpenGL没有声明的是这些变量间的间距(Spacing)。这允许硬件能够在它认为合适的位置放置变量。比如说，一些硬件可能会将一个vec3放置在float边上。不是所有的硬件都能这样处理，可能会在附加这个float之前，先将vec3填充(Pad)为一个4个float的数组。这个特性本身很棒，但是会对我们造成麻烦。
>
> &emsp;&emsp;默认情况下，GLSL会使用一个叫做共享(Shared)布局的Uniform内存布局，共享是因为一旦硬件定义了偏移量，它们在多个程序中是**共享**并一致的。使用共享布局时，GLSL是可以为了优化而对uniform变量的位置进行变动的，只要变量的顺序保持不变。因为我们无法知道每个uniform变量的偏移量，我们也就不知道如何准确地填充我们的Uniform缓冲了。我们能够使用像是glGetUniformIndices这样的函数来查询这个信息，但这超出本节的范围了。
>
> &emsp;&emsp;虽然共享布局给了我们很多节省空间的优化，但是我们需要查询每个uniform变量的偏移量，这会产生非常多的工作量。通常的做法是，不使用共享布局，而是使用std140布局。std140布局声明了每个变量的偏移量都是由一系列规则所决定的，这**显式地**声明了每个变量类型的内存布局。由于这是显式提及的，我们可以手动计算出每个变量的偏移量。
>
> &emsp;&emsp;每个变量都有一个基准对齐量(Base Alignment)，它等于一个变量在Uniform块中所占据的空间（包括填充量(Padding)），这个基准对齐量是使用std140布局的规则计算出来的。接下来，对每个变量，我们再计算它的对齐偏移量(Aligned Offset)，它是一个变量从块起始位置的字节偏移量。一个变量的对齐字节偏移量**必须**等于基准对齐量的倍数。
>
> &emsp;&emsp;布局规则的原文可以在OpenGL的Uniform缓冲规范[这里](http://www.opengl.org/registry/specs/ARB/uniform_buffer_object.txt)找到，但我们将会在下面列出最常见的规则。GLSL中的每个变量，比如说int、float和bool，都被定义为4字节量。每4个字节将会用一个`N`来表示。
>
> | 类型                | 布局规则                                                     |
> | ------------------- | ------------------------------------------------------------ |
> | 标量，比如int和bool | 每个标量的基准对齐量为N。                                    |
> | 向量                | 2N或者4N。这意味着vec3的基准对齐量为4N。                     |
> | 标量或向量的数组    | 每个元素的基准对齐量与vec4的相同。                           |
> | 矩阵                | 储存为列向量的数组，每个向量的基准对齐量与vec4的相同。       |
> | 结构体              | 等于所有元素根据规则计算后的大小，但会填充到vec4大小的倍数。 |
>
> &emsp;&emsp;和OpenGL大多数的规范一样，使用例子就能更容易地理解。我们会使用之前引入的那个叫做ExampleBlock的Uniform块，并使用std140布局计算出每个成员的对齐偏移量：

```c++
layout (std140) uniform ExampleBlock
{
                     // 基准对齐量       // 对齐偏移量
    float value;     // 4               // 0 
    vec3 vector;     // 16              // 16  (必须是16的倍数，所以 4->16)
    mat4 matrix;     // 16              // 32  (列 0)
                     // 16              // 48  (列 1)
                     // 16              // 64  (列 2)
                     // 16              // 80  (列 3)
    float values[3]; // 16              // 96  (values[0])
                     // 16              // 112 (values[1])
                     // 16              // 128 (values[2])
    bool boolean;    // 4               // 144
    int integer;     // 4               // 148
}; 
```

> &emsp;&emsp;通过在Uniform块定义之前添加`layout (std140)`语句，我们告诉OpenGL这个Uniform块使用的是std140布局。除此之外还可以选择两个布局，但它们都需要我们在填充缓冲之前先查询每个偏移量。我们已经见过`shared`布局了，剩下的一个布局是`packed`。当使用紧凑(Packed)布局时，是不能保证这个布局在每个程序中保持不变的（即非共享），因为它允许编译器去将uniform变量从Uniform块中优化掉，这在每个着色器中都可能是不同的。