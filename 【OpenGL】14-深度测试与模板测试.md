---
title: 【OpenGL】14-深度测试与模板测试
date: 2018-03-06 20:45:59
tags: 
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[深度测试](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/01%20Depth%20testing/) [模板测试](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/02%20Stencil%20testing/)

【英文版】[Depth testing](https://learnopengl.com/Advanced-OpenGL/Depth-testing) [Stencil testing](https://learnopengl.com/Advanced-OpenGL/Stencil-testing)

#### 学习心得

&emsp;&emsp;在我们之前所实现的代码中，总是使用了 3D 图形，包括我们所导入的模型以及之前一直在使用的正立方体。以正方体为例，这是一个对称的结构，总是有一部分内容在一部分之后使得我们不可见。由于计算机屏幕坐标系本身是一个二维的面，我们必须在这上边模拟出 3D 的效果。而遮挡效果也是必须要实现的，在这里需要介绍一个概念：[深度缓冲](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E7%BC%93%E5%86%B2) 。

<!--more-->

> &emsp;&emsp;在[计算机图形学](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9B%BE%E5%BD%A2%E5%AD%A6)中，**深度缓冲**是在三维图形中处理图像深度坐标的过程，这个过程通常在[硬件](https://zh.wikipedia.org/wiki/%E7%A1%AC%E4%BB%B6)中完成，它也可以在[软件](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6)中完成，它是[可见性问题](https://zh.wikipedia.org/w/index.php?title=%E5%8F%AF%E8%A7%81%E6%80%A7%E9%97%AE%E9%A2%98&action=edit&redlink=1)的一个解决方法。可见性问题是确定渲染场景中哪部分可见、哪部分不可见的问题。[画家算法](https://zh.wikipedia.org/wiki/%E7%94%BB%E5%AE%B6%E7%AE%97%E6%B3%95)是另外一种常用的方法，尽管效率较低，但是也可以处理透明场景元素。深度缓冲也称为Z缓冲。
>
> &emsp;&emsp;当[三维图形卡](https://zh.wikipedia.org/wiki/GPU)渲染物体的时候，每一个所生成的[像素](https://zh.wikipedia.org/wiki/%E5%83%8F%E7%B4%A0)的深度（即z坐标）就保存在一个[缓冲区](https://zh.wikipedia.org/wiki/%E7%BC%93%E5%86%B2%E5%8C%BA)中。这个缓冲区叫作**z缓冲区**或者**深度缓冲区**，这个缓冲区通常组织成一个保存每个屏幕像素深度的x-y二维数组。如果场景中的另外一个物体也在同一个像素生成渲染结果，那么图形处理卡就会比较二者的深度，并且保留距离观察者较近的物体。然后这个所保留的物体点深度保存到深度缓冲区中。最后，图形卡就可以根据深度缓冲区正确地生成通常的深度感知效果：较近的物体遮挡较远的物体。这个过程叫作**z消隐**。

&emsp;&emsp;OpenGL 中提供了一个深度测试，用来判断有重合区域的两部分，深度较高的（较远）的那部分将被抛弃不予以渲染。启动深度测试方法如下（默认禁用）：

```c++
	glEnable(GL_DEPTH_TEST);
```

&emsp;&emsp;当它启用的时候，如果一个片段通过了深度测试的话，OpenGL会在深度缓冲中储存该片段的z值；如果没有通过深度缓冲，则会丢弃该片段。如果你启用了深度缓冲，还应该在每个渲染迭代之前使用GL_DEPTH_BUFFER_BIT 来清除深度缓冲，否则你会仍在使用上一次渲染迭代中的写入的深度值：

```c++
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

&emsp;&emsp;可以想象，在某些情况下我们会需要对所有片段都执行深度测试并丢弃相应的片段，但**不**希望更新深度缓冲。基本上来说，在使用一个只读的(Read-only)深度缓冲。OpenGL允许我们禁用深度缓冲的写入，只需要设置它的深度掩码(Depth Mask)设置为`GL_FALSE`就可以了：

```c++
	glDepthMask(GL_FALSE);
```

&emsp;&emsp;在启动深度测试时，我们也可以设置深度测试的模式，使用 glDepthFunc() 方法，参数可以设置如下（默认为 GL_LESS）：

| 函数        | 描述                                         |
| ----------- | -------------------------------------------- |
| GL_ALWAYS   | 永远通过深度测试                             |
| GL_NEVER    | 永远不通过深度测试                           |
| GL_LESS     | 在片段深度值小于缓冲的深度值时通过测试       |
| GL_EQUAL    | 在片段深度值等于缓冲区的深度值时通过测试     |
| GL_LEQUAL   | 在片段深度值小于等于缓冲区的深度值时通过测试 |
| GL_GREATER  | 在片段深度值大于缓冲区的深度值时通过测试     |
| GL_NOTEQUAL | 在片段深度值不等于缓冲区的深度值时通过测试   |
| GL_GEQUAL   | 在片段深度值大于等于缓冲区的深度值时通过测试 |

&emsp;&emsp;除了深度测试，还有一个[模板测试](https://baike.baidu.com/item/%E6%A8%A1%E6%9D%BF%E6%B5%8B%E8%AF%95/1747227) ：

> &emsp;&emsp;[图形卡](https://baike.baidu.com/item/%E5%9B%BE%E5%BD%A2%E5%8D%A1)控制屏幕的方法是在图形卡的显存中分配一个区域，区域中每一个单元中存储的颜色值和屏幕中相对应点的颜色一一对应，也就是说如果程序修改了显存区域中一个单元中存储的颜色值，也就修改了和这个单元相对应的屏幕点的颜色。这个显存区域一般叫屏幕显示区。
>
> &emsp;&emsp;有时，需要为屏幕显示区的每个单元设置一些标记，在渲染时根据这些标记进行不同的处理，记录这些标记的显存单元集合称作模板[缓存](https://baike.baidu.com/item/%E7%BC%93%E5%AD%98)区。例如分辨率为1024×768像素的显示器模式，屏幕显示区的单元数应为1024×768，由于屏幕每个像素点都应有一个标记，所以模板[缓存](https://baike.baidu.com/item/%E7%BC%93%E5%AD%98)区单元数也应为1024×768。模板[缓存](https://baike.baidu.com/item/%E7%BC%93%E5%AD%98)区的一个单元可以是1位、4位或8位二进制数。
>
> &emsp;&emsp;模板缓存区的一个单元和深度缓存区的一个单元总是共用一个32位二进制数，例如，其中24位为深度缓存区单元，8位为模板缓存区单元。显卡不同，模板缓存区单元和深度缓存区单元在32位二进制数中各允许占用的位数也不同。可以用如下方法查看本计算机使用的[显卡](https://baike.baidu.com/item/%E6%98%BE%E5%8D%A1)中，模板缓存区单元和深度缓存区单元各允许占用多少位数，依序选择“开始”|“程序”|“Microsoft DirectX 9.0 SDK Update|DirectX Utilitites|DirectX Caps Viewer”菜单项，然后打开“DirectX Caps Viewer”窗口，即可看到本机显卡所支持的所有深度/模板缓存区单元模式。例如，模式D24S8表示一个32位二进制数中深度缓存区单元24位，模板缓存区单元8位，模式D24X8表示深度缓存区单元24位，其余8位不使用。
>
> &emsp;&emsp;使用渲染函数渲染3D模型的最后一步是，将投影变换得到的且将在屏幕显示的3D模型的每个像素点的颜色值写入屏幕显示区相应的单元。如果允许模版测试，就在每次写入一个像素点颜色值前，使用设定的模版比较函数进行测试，即对该像素点对应的模板缓冲单元的值和模板参考值进行比较，只有在该点模板函数比较成功时，渲染函数才执行写入。当然，最终是否写入，还要取决于深度测试，否则将保留该像素点在屏幕显示区相应单元原来的颜色值。最后还要根据比较结果，对该模板缓冲单元的值做指定的处理。
>
> &emsp;&emsp;例子如下：
>
> ![](https://learnopengl-cn.github.io/img/04/02/stencil_buffer.png)

&emsp;&emsp;模板测试的启动和控制方法和深度测试类似：

```c++
	glEnable(GL_STENCIL_TEST);
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
```

&emsp;&emsp;设置属性也是有具体的方法：

&emsp;&emsp;和深度测试一样，我们对模板缓冲应该通过还是失败，以及它应该如何影响模板缓冲，也是有一定控制的。一共有两个函数能够用来配置模板测试：glStencilFunc 和 glStencilOp。

&emsp;&emsp;glStencilFunc(GLenum func, GLint ref, GLuint mask) 一共包含三个参数：

- `func`：设置模板测试函数(Stencil Test Function)。这个测试函数将会应用到已储存的模板值上和glStencilFunc函数的`ref`值上。可用的选项有：GL_NEVER、GL_LESS、GL_LEQUAL、GL_GREATER、GL_GEQUAL、GL_EQUAL、GL_NOTEQUAL和GL_ALWAYS。它们的语义和深度缓冲的函数类似。
- `ref`：设置了模板测试的参考值(Reference Value)。模板缓冲的内容将会与这个值进行比较。
- `mask`：设置一个掩码，它将会与参考值和储存的模板值在测试比较它们之前进行与(AND)运算。初始情况下所有位都为1。

&emsp;&emsp;glStencilFunc仅仅描述了OpenGL应该对模板缓冲内容做什么，而不是我们应该如何更新缓冲。这就需要glStencilOp这个函数了。

&emsp;&emsp;glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)一共包含三个选项，我们能够设定每个选项应该采取的行为：

- `sfail`：模板测试失败时采取的行为。
- `dpfail`：模板测试通过，但深度测试失败时采取的行为。
- `dppass`：模板测试和深度测试都通过时采取的行为。

&emsp;&emsp;每个选项都可以选用以下的其中一种行为：

| 行为         | 描述                                               |
| ------------ | -------------------------------------------------- |
| GL_KEEP      | 保持当前储存的模板值                               |
| GL_ZERO      | 将模板值设置为0                                    |
| GL_REPLACE   | 将模板值设置为glStencilFunc函数设置的`ref`值       |
| GL_INCR      | 如果模板值小于最大值则将模板值加1                  |
| GL_INCR_WRAP | 与GL_INCR一样，但如果模板值超过了最大值则归零      |
| GL_DECR      | 如果模板值大于最小值则将模板值减1                  |
| GL_DECR_WRAP | 与GL_DECR一样，但如果模板值小于0则将其设置为最大值 |
| GL_INVERT    | 按位翻转当前的模板缓冲值                           |

&emsp;&emsp;默认情况下 glStencilOp 是设置为`(GL_KEEP, GL_KEEP, GL_KEEP)`的，所以不论任何测试的结果是如何，模板缓冲都会保留它的值。默认的行为不会更新模板缓冲，所以如果你想写入模板缓冲的话，你需要至少对其中一个选项设置不同的值。

&emsp;&emsp;所以，通过使用 glStencilFunc 和 glStencilOp ，我们可以精确地指定更新模板缓冲的时机与行为了，我们也可以指定什么时候该让模板缓冲通过，即什么时候片段需要被丢弃。