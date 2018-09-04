---
title: 【OpenGL】Breakout-3-实现我们的基础类
date: 2018-03-13 09:27:46
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;上一篇文章中，我们为游戏搭建了基本框架，创建了几个必要类，但是也仅仅是创建了他们而没有实现。现在我们将开始为他们编码，实现其内容。这将是出现大量代码的一篇文章。

<!--more-->

&emsp;&emsp;首先是 Shader 类，我们之前的代码中也用到了 Shader 类，但是那个类封装的较为粗糙。甚至于将文件读取也封装到了里边，而现在我们对其稍微更改：

```c++
class Shader
{
public:

	// State
	GLuint id;
	// Constructor
	Shader();
	// Sets the current shader as active
	Shader  &Use();
	// Compiles the shader from given source code
	void    Compile(const GLchar *vertexSource, const GLchar *fragmentSource, const GLchar *geometrySource = nullptr); // Note: geometry source code is optional 
	// Utility functions
	void    SetFloat(const GLchar *name, GLfloat value, GLboolean useShader = false);
	void    SetInteger(const GLchar *name, GLint value, GLboolean useShader = false);
	void    SetVector2F(const GLchar *name, GLfloat x, GLfloat y, GLboolean useShader = false);
	void    SetVector2F(const GLchar *name, const glm::vec2 &value, GLboolean useShader = false);
	void    SetVector3F(const GLchar *name, GLfloat x, GLfloat y, GLfloat z, GLboolean useShader = false);
	void    SetVector3F(const GLchar *name, const glm::vec3 &value, GLboolean useShader = false);
	void    SetVector4F(const GLchar *name, GLfloat x, GLfloat y, GLfloat z, GLfloat w, GLboolean useShader = false);
	void    SetVector4F(const GLchar *name, const glm::vec4 &value, GLboolean useShader = false);
	void    SetMatrix4(const GLchar *name, const glm::mat4 &matrix, GLboolean useShader = false);
};
```

&emsp;&emsp;我们为其封装了 id 属性，Compiler 方法以及 Use 方法和为着色器传输数据的 Set 系列方法。具体方法的实现看这里：[Shader.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/Shader.cpp) 。

&emsp;&emsp;除了 Shader 之外，另一个资源类则是 Texture 类，这是我们的 Texture2D 类声明：

```c++
#include <GL/glew.h>

// Texture2D is able to store and configure a texture in OpenGL.
// It also hosts utility functions for easy management.
class Texture2D
{
public:
    // Holds the ID of the texture object, used for all texture operations to reference to this particlar texture
    GLuint id{};
    // Texture image dimensions
    GLuint width, height; // Width and height of loaded image in pixels
    // Texture Format
    GLuint internalFormat; // Format of texture object
    GLuint imageFormat; // Format of loaded image
    // Texture configuration
    GLuint wrapS; // Wrapping mode on S axis
    GLuint wrapT; // Wrapping mode on T axis
    GLuint filterMin; // Filtering mode if texture pixels < screen pixels
    GLuint filterMax; // Filtering mode if texture pixels > screen pixels
    // Constructor (sets default texture modes)
    Texture2D();
    // Generates texture from image data
    void Generate(GLuint width, GLuint height, unsigned char* data);
    // Binds the texture as the current active GL_TEXTURE_2D texture object
    void Bind() const;
};
```

&emsp;&emsp;如 Shader 类一样，我们在类中封装了大量的属性和方法，具体实现也可以直接看源代码 [Texture2D.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/Texture2D.cpp)，毕竟这不是我们这次游戏练习的主要内容。

&emsp;&emsp;游戏类的封装我们在上一篇文章中介绍了，资源管理器类其实也就是我们上一篇文章中写的那几个，毕竟现在只是大概一个框架，没有多少详尽的内容。

&emsp;&emsp;这是 ResourceManager 类的声明：

```c++
class ResourceManager
{
public:
	// 定义着色器与纹理存储
	static std::map<std::string, Shader> shaders;
	static std::map<std::string, Texture2D> textures;

	// 定义 Shader 交互方法
	static Shader LoadShader(const GLchar *vertexShaderFile, const GLchar *fragmentShaderFile, const GLchar *geometryShaderFile, std::string name);
	static Shader GetShader(std::string name);

	// 定义 Texture2D 交互方法
	static Texture2D LoadTexture2D(const GLchar *file, GLboolean alpha, std::string name);
	static Texture2D GetTexture2D(std::string name);

	ResourceManager();
	~ResourceManager();
};
```

&emsp;&emsp;具体实现也直接看源文件吧：[ResourceManager.cpp](https://github.com/KsGin/OpenGL-BreakOut/blob/master/OpenGL-Breakout/ResourceManager.cpp)

&emsp;&emsp;在接下来的学习中，我们将着重于介绍围绕游戏逻辑以及游戏开发技术的方面，基础的 OpenGL 知识将一笔带过。