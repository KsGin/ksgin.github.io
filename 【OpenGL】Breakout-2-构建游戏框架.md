---
title: 【OpenGL】Breakout-2-构建游戏框架
date: 2018-03-12 21:23:18
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;一个真正的程序往往不是一个 main 方法就可以完成的，显然，对于初学 OpenGL 的我们（尽管我们可能有很长时间的 C/C++ 编程经验）来说，在初次实现一个小项目的时候，需要在开发之前理清项目结构。这样在正式编码的时候才能不会手忙脚乱。

<!--more-->

&emsp;&emsp;首先定义顶级类游戏类（Game Class）控制游戏状态，资源管理器类控制游戏资源（ResourceManager Class），所以我们需要这两个类：

```c++
// Game 类
class Game {
	...
};

// ResourceManager 类
class ResourceManager {
  	...  
};
```

&emsp;&emsp;除此之外，我们之前在代码中使用的 Shader 类和纹理操作也是需要封装的，在之前的 Shader 类中，我们用 Shader 类直接从读取着色器文件到返回 Program 全部在类内实现，这显然不是一个好的实现，更何况我们现在又有了全局的资源管理器类，所以将 Shader 类重写并添加相应的 Texture 类（Shader 和 Texture 统属于 ResourceManager 类进行管理，**请注意我们的资源管理类是为了直接调用其他类而存在的，所以属性和方法等都用 static 修饰**）。所以我们为 ResourceManager 类添加 Shader 和 Texture 的管理：

```c++
// 定义着色器与纹理存储
static std::map<std::string, Shader> shaders;
static std::map<std::string, Texture2D> textures;

// 定义 Shader 交互方法
static Shader LoadShader(const GLchar *vertexShaderFile, const GLchar *fragmentShaderFile, const GLchar *geometryShaderFile, std::string name);
static Shader GetShader(std::string name);

// 定义 Texture2D 交互方法
static Texture2D LoadTexture2D(const GLchar *file, GLboolean alpha, std::string name);
static Texture2D GetTexture2D(std::string name);
```

&emsp;&emsp;我们定义了用来存储 shader 对象和 texture 对象的变量，然后给定了从文件中读取源文件编译处理成 shader 或者 texture 并写入 map 的方法，并给定从 map 中返回 shader 或 texture 的方法。这样，我们在调用着色器或者纹理的时候可以这样操作。

```c++
Shader shader = ResourceManager::GetShader("shader_name"); //调用已有着色器
Shader shader = ResourceManager::LoadShader("vFile.vs" , "fFile.fs" , "gFile.gs" , "shader_name"); //使用新编译着色器
```

&emsp;&emsp;纹理操作类似。

&emsp;&emsp;在暂时不考虑细节实现的情况下，我们使用 ResourceManager 进行资源对象管理暂时就这样子了。接下来看另一个类 Game 。Game 类的主要作用是（简单）管理游戏代码，并与此同时将所有的窗口代码从游戏中解耦。这样子的话，我们就可以把相同的类迁移到完全不同的窗口库（比如SDL或SFML）而不需要做太多的工作。在我们的 Game 类中，我们需要一个游戏状态的属性（GameState），为此我们先创建一个枚举值代表游戏的不同状态：

```c++
enum GameState {
	GAME_ACTIVE,	
	GAME_MENU,
	GAME_WIN
};
```

&emsp;&emsp;除此外我们再封装一个初始化函数（Init），一个输入处理的函数（ProcessInput），一个画面的更新函数（Update），一个渲染函数（Render）。

```c++
// 游戏状态
enum GameState {
	GAME_ACTIVE,
	GAME_MENU,
	GAME_WIN
};

// 游戏类
class Game {
public:
	// 游戏状态
	GameState state;
	GLboolean keys[1024];
	GLuint width, height;
	// 构造方法与析构方法
	Game(GLuint width , GLuint height);
	~Game();
	// 游戏初始化
	void Init();
    // 游戏输入处理
	void ProcessInput(GLuint dt);
    // 游戏更新
	void Update(GLfloat dt);
    // 渲染
	void Render();
};
```

&emsp;&emsp;暂不考虑具体的实现，我们一个小游戏的框架首先要搭建起来，在这个游戏框架里， Game 类处理游戏代码， ResourceManager 管理游戏的资源。