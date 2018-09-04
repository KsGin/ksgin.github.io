---
title: 【OpenGL】Breakout-4-精灵
date: 2018-03-14 18:05:47
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;到现在为止，我们基本实现了我们的游戏框架代码，而现在就是正式开始渲染游戏和检验前边成果的时候了。这篇文章我们主要介绍精灵（Sprite），以及创建一个精灵类以实现精灵的渲染。

<!--more-->

&emsp;&emsp;[精灵（Sprite） ](https://zh.wikipedia.org/wiki/精灵图) 是计算机图形学中的一个概念（不是精灵族的精灵），他的定义是包含于场景中的一个二维贴图或者图像，也就是说一个二维的贴图，我们今天想要做的就是将他渲染出来。为此我们创建一个 SpriteRender 类：

```c++
class SpriteRenderer
{
    public:
	explicit SpriteRenderer(Shader &&shader);
        ~SpriteRenderer();

        void DrawSprite(Texture2D &texture, glm::vec2 position, 
            glm::vec2 size = glm::vec2(10, 10), GLfloat rotate = 0.0f, 
            glm::vec3 color = glm::vec3(1.0f));
    private:
        Shader shader_; 
        GLuint quadVao_;

        void InitRenderData();
};
```

&emsp;&emsp;在这个类中我们有除了构造和析构外只有一个公开方法，就是 DrawSprite ，我们可以使用它来进行绘制。他的实现的源代码在这里：[SpriteRender.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/SpriteRender.cpp)。代码也比较简单，首先在构造方法里，我们接受了一个 Shader 对象作为参数，并赋值给 shader_ 成员，同时调用 InitRenderData() 方法。

```c++
SpriteRenderer::SpriteRenderer(Shader &&shader) : quadVao_(0) { 
    this->shader_ = shader; 
    this->InitRenderData(); 
}
```

&emsp;&emsp;在 InitRenderData 方法里，我们创建顶点数组，并绑定 VBO ，VAO 等，这些操作我们在之前的代码中都写过很多，也就是换了个地方而已。

```c++
GLuint vbo;
GLfloat vertices[] = {
	0.0f, 1.0f, 0.0f, 1.0f,		
	1.0f, 0.0f, 1.0f, 0.0f,
	0.0f, 0.0f, 0.0f, 0.0f,

	0.0f, 1.0f, 0.0f, 1.0f,
	1.0f, 1.0f, 1.0f, 1.0f,
	1.0f, 0.0f, 1.0f, 0.0f
};

glGenVertexArrays(1, &this->quadVao_);
glGenBuffers(1, &vbo);

glBindBuffer(GL_ARRAY_BUFFER, vbo);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

glBindVertexArray(this->quadVao_);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 4 * sizeof(GLfloat), static_cast<GLvoid*>(0));
glBindBuffer(GL_ARRAY_BUFFER, 0);
glBindVertexArray(0);
```

&emsp;&emsp;**请注意我们 vertices 数组以四个数值为一组，前两个是顶点坐标（我们是二维的游戏，所以没有 z 轴，在顶点着色器里默认赋值为 0.0f），后两个是纹理坐标。**

&emsp;&emsp;DrawSprite 方法则是封装了给着色器传值和 Draw 等函数调用，这里不再介绍。下来主要看我们的着色器操作，我们的小游戏是二维平面游戏，所以不需要 z 轴。同时也需要将观察空间坐标转换为二维投影坐标。

> &emsp;&emsp;从这个[坐标系统](https://learnopengl-cn.github.io/01%20Getting%20started/08%20Coordinate%20Systems/)教程中，我们明白了投影矩阵的作用是把观察空间坐标转化为标准化设备坐标。通过生成合适的投影矩阵，我们就可以在不同的坐标系下计算，这可能比把所有的坐标都指定为标准化设备坐标（再计算）要更容易处理。
>
> &emsp;&emsp;我们不需要对坐标系应用透视，因为这个游戏完全是2D的，所以一个正射投影矩阵(Orthographic Projection Matrix)就可以了。由于正射投影矩阵几乎直接变换所有的坐标至裁剪空间，我们可以定义如下的投影矩阵指定世界坐标为屏幕坐标：`glm::mat4 projection = glm::ortho(0.0f, 800.0f, 600.0f, 0.0f, -1.0f, 1.0f);` 。
>
> &emsp;&emsp;前面的四个参数依次指定了投影平截头体的左、右、下、上边界。这个投影矩阵把所有在0到800之间的x坐标变换到-1到1之间，并把所有在0到600之间的y坐标变换到-1到1之间。这里我们指定了平截头体顶部的y坐标值为0，底部的y坐标值为600。所以，这个场景的左上角坐标为(0,0)，右下角坐标为(800,600)，就像屏幕坐标那样。观察空间坐标直接对应最终像素的坐标。
>
> ![img](https://learnopengl-cn.github.io/img/06/Breakout/03/projection.png)
>
> &emsp;&emsp;这样我们就可以指定所有的顶点坐标为屏幕上的像素坐标了，这对2D游戏来说相当直观。

&emsp;&emsp;在顶点着色器中，我们将传入顶点着色器的数组分为两部分，前两个为 x , y 坐标，后两个为纹理坐标，传值给片段着色器。

```c++
#version 420 core
layout (location = 0) in vec4 aPos;

out vec2 TexCoords;

uniform mat4 model;
uniform mat4 projection;

void main()
{
    TexCoords = aPos.zw;
    gl_Position = projection * model * vec4(aPos.xy, 0.0f, 1.0f);
}
```

&emsp;&emsp;片段着色器：

```c++
#version 420 core

in vec2 TexCoords;
out vec4 color;

uniform sampler2D image;
uniform vec3 spriteColor;

void main()
{
    color = vec4(spriteColor, 1.0f) + texture(image, TexCoords);
}
```

&emsp;&emsp;我们通过传入的纹理和颜色向量来控制纹理颜色。

&emsp;&emsp;精灵类算是完成了，我们将在 Game 类的方法里去加载并且渲染他：

&emsp;&emsp;在 Game::Init 里加载：

```c++
// 加载着色器
ResourceManager::LoadShader("resources/shaders/sprite.vertexShader", "resources/shaders/sprite.fragmentShader", nullptr, "sprite");
// 配置着色器
const auto projection = glm::ortho(0.0f, static_cast<GLfloat>(this->width),
	static_cast<GLfloat>(this->height), 0.0f, -1.0f, 1.0f);
ResourceManager::GetShader("sprite").Use().SetInteger("image", 0);
ResourceManager::GetShader("sprite").SetMatrix4("projection", projection);
// 设置专用于渲染的控制
renderer = new SpriteRenderer(ResourceManager::GetShader("sprite"));
// 加载纹理
ResourceManager::LoadTexture2D("resources/textures/awesomeface.png", GL_TRUE, "face");
```

&emsp;&emsp;在 Game::Render 里调用渲染：

```c++
auto spriteTexture = ResourceManager::GetTexture2D("face");
renderer->DrawSprite(spriteTexture, glm::vec2(100.0f, 100.0f), glm::vec2(200, 300), 45.0f,
	glm::vec3(0.1f, 0.2f, 0.1f));
```

&emsp;&emsp;现在我们看看效果吧：

![Sprite](https://image.ibb.co/f80B0H/image.png)

