---
title: 【OpenGL】15-混合
date: 2018-03-07 17:15:40
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[混合](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/03%20Blending/)

【英文版】[Blending](https://learnopengl.com/Advanced-OpenGL/Blending)

#### 学习记录

> &emsp;&emsp;OpenGL中，混合(Blending)通常是实现物体透明度(Transparency)的一种技术。透明就是说一个物体（或者其中的一部分）不是纯色(Solid Color)的，它的颜色是物体本身的颜色和它背后其它物体的颜色的不同强度结合。一个有色玻璃窗是一个透明的物体，玻璃有它自己的颜色，但它最终的颜色还包含了玻璃之后所有物体的颜色。这也是混合这一名字的出处，我们混合(Blend)（不同物体的）多种颜色为一种颜色。所以透明度能让我们看穿物体。

<!--more-->

&emsp;&emsp;在现实生活中，我们有完全透明的玻璃，也有带色彩半透明的玻璃。在透过完全透明玻璃看东西的时候，我们看到的是物体的原本色彩，而在透过有色玻璃看东西的时候，我们看到的就成了玻璃的颜色和物体本身颜色混合后的色彩。

![blending_transparency](https://ws4.sinaimg.cn/large/006tKfTcly1fp4ihxt5arj30go08qab3.jpg)

&emsp;&emsp;如上图所示，当我们透过红色的半透明玻璃看物体时候，看到的便是混合后的颜色。

> &emsp;&emsp;透明的物体可以是完全透明的（让所有的颜色穿过），或者是半透明的（它让颜色通过，同时也会显示自身的颜色）。一个物体的透明度是通过它颜色的aplha值来决定的。Alpha颜色值是颜色向量的第四个分量，你可能已经看到过它很多遍了。在这个教程之前我们都将这个第四个分量设置为1.0，让这个物体的透明度为0.0，而当alpha值为0.0时物体将会是完全透明的。当alpha值为0.5时，物体的颜色有50%是来自物体自身的颜色，50%来自背后物体的颜色。
>
> &emsp;&emsp;我们目前一直使用的纹理有三个颜色分量：红、绿、蓝。但一些材质会有一个内嵌的alpha通道，对每个纹素(Texel)都包含了一个alpha值。这个alpha值精确地告诉我们纹理各个部分的透明度。比如说，下面这个[窗户纹理](https://learnopengl-cn.github.io/img/04/03/blending_transparent_window.png)中的玻璃部分的alpha值为0.25（它在一般情况下是完全的红色，但由于它有75%的透明度，能让很大一部分的网站背景颜色穿过，让它看起来不那么红了），边框的alpha值是0.0。

&emsp;&emsp;首先实现比较简单的要么全透明要么全不透明的纹理透明度，使用小草的纹理图片为例（我们总是需要在地面上绘制很多草丛，然而为了效率我们又不能像是渲染模型一样费大力气去构建他的模型以及渲染他，所以使用一张带有草丛的纹理图片）：

![grass](https://ws3.sinaimg.cn/large/006tKfTcly1fp4ihxbz37j30e80e8wfl.jpg)

&emsp;&emsp;可能你会觉得，既然是一张除了草丛其他部位都是透明的纹理贴图，那么直接渲染上去就可以了吧。然而如果我们不自己进行处理，OpenGL 并不能很好的去完成它。比如我们直接渲染纹理：

```c++
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
```

&emsp;&emsp;请注意和我们之前调用不同的地方在于我们使用了 GL_RGBA 而不是 GL_RGB。

```c++
void main()
{
    FragColor = texture(texture1, TexCoords);
}
```

&emsp;&emsp;由于我们纹理贴图传进来的是 rgba vec4数据，所以倒不用去特意转换为 vec4。在各个地方渲染了几个草丛后如下：

![blending_no_discard](https://ws3.sinaimg.cn/large/006tKfTcly1fp4ihwnnqpj30go0d2gmp.jpg)

&emsp;&emsp;这样我们就可以看到草丛纹理的别扭之处了，我们没有对草丛进行处理，虽然纹理图片本身是透明的，但是默认下并没有进行处理，所以我们看到的是白色底。所以我们需要对纹理进行设置：

```c++
	...
	vec4 texColor = texture(texture1, TexCoords);
    if(texColor.a < 0.1)
        discard;
    FragColor = texColor;
	...
```

&emsp;&emsp;在片段着色器中，我们读取这个纹理渲染单元的 a 属性（透明度），当期超过一个阈值时我们认为他是完全透明的，所以做 discard （丢弃）操作。最终效果如下：

![blending_discard](https://ws1.sinaimg.cn/large/006tKfTcly1fp4ihwx0a8j30go0d2wff.jpg)

> &emsp;&emsp;注意，当采样纹理的边缘的时候，OpenGL会对边缘的值和纹理下一个重复的值进行插值（因为我们将它的环绕方式设置为了GL_REPEAT。这通常是没问题的，但是由于我们使用了透明值，纹理图像的顶部将会与底部边缘的纯色值进行插值。这样的结果是一个半透明的有色边框，你可能会看见它环绕着你的纹理四边形。要想避免这个，每当你alpha纹理的时候，请将纹理的环绕方式设置为GL_CLAMP_TO_EDGE：

```
glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri( GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
```

&emsp;&emsp;是不是看起来问题不大了。这是看起来很简单的要么渲染要么抛弃问题，实现起来也是比较简单，下来我们将看看透过那些半透明的物体（有色玻璃）看东西时候的颜色是如何渲染的。

&emsp;&emsp;首先要启动混合，我们需要调用 glEnable(GL_BLEND); 来启动混合功能，而且需要设置 glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); 方法。以下是参数说明：

> - GL_FUNC_ADD：默认选项，将两个分量相加：result=Src+Dst
> - GL_FUNC_SUBTRACT：将两个分量相减： result=Src−Dst
> - GL_FUNC_REVERSE_SUBTRACT：将两个分量相减，但顺序相反：result=Dst−Src

&emsp;&emsp;OpenGL中的混合规则请看这里：[OpenGL超级宝典笔记——混合](https://my.oschina.net/sweetdark/blog/169668)

&emsp;&emsp;我们在启用且设置混合参数后，效果如下：

```c++
	glEnable(GL_BLEND);
	glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

![blending_incorrect_order](https://ws1.sinaimg.cn/large/006tNc79ly1fp70ml46bvj30go0d2t9w.jpg)

&emsp;&emsp;可以看到我们已经实现了部分混合，可是却出现了遮挡问题，即最前边的窗口遮挡了后边的内容。这是因为我们的渲染顺序不对而造成的。

> &emsp;&emsp;发生这一现象的原因是，深度测试和混合一起使用的话会产生一些麻烦。当写入深度缓冲时，深度缓冲不会检查片段是否是透明的，所以透明的部分会和其它值一样写入到深度缓冲中。结果就是窗户的整个四边形不论透明度都会进行深度测试。即使透明的部分应该显示背后的窗户，深度测试仍然丢弃了它们。
>
> &emsp;&emsp;所以我们不能随意地决定如何渲染窗户，让深度缓冲解决所有的问题了。这也是混合变得有些麻烦的部分。要想保证窗户中能够显示它们背后的窗户，我们需要首先绘制背后的这部分窗户。这也就是说在绘制的时候，我们必须先手动将窗户按照最远到最近来排序，再按照顺序渲染。
>
> &emsp;&emsp;要想让混合在多个物体上工作，我们需要最先绘制最远的物体，最后绘制最近的物体。普通不需要混合的物体仍然可以使用深度缓冲正常绘制，所以它们不需要排序。但我们仍要保证它们在绘制（排序的）透明物体之前已经绘制完毕了。当绘制一个有不透明和透明物体的场景的时候，大体的原则如下：
>
> 1. 先绘制所有不透明的物体。
> 2. 对所有透明的物体排序。
> 3. 按顺序绘制所有透明的物体。
>
> &emsp;&emsp;排序透明物体的一种方法是，从观察者视角获取物体的距离。这可以通过计算摄像机位置向量和物体的位置向量之间的距离所获得。接下来我们把距离和它对应的位置向量存储到一个STL库的map数据结构中。map会自动根据键值(Key)对它的值排序，所以只要我们添加了所有的位置，并以它的距离作为键，它们就会自动根据距离值排序了。

