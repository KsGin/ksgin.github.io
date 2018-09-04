---
title: 【OpenGL】18-三维贴图
date: 2018-03-09 23:52:17
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[立方体贴图](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/06%20Cubemaps/)

【英文版】[Cubemaps](https://learnopengl.com/Advanced-OpenGL/Cubemaps)

#### 学习记录

&emsp;&emsp;之前我们学习的纹理贴图都是为某一面进行贴图，这篇文章中，将会学到一种一般用来构建基本环境的贴图方式：**立方体贴图**。

&emsp;&emsp;立方体贴图很容易理解，就是为一个立方体的六个面进行贴图，不过这次不是在表面。我们以立方体内部中心为原点，为立方体内部面贴图（是不是觉得是在渲染一个房间环境或者一个巨大的场景贴图，没错！）。依照此贴图我们可以去渲染出一些很好看的场景。

<!--more-->

> &emsp;&emsp;简单来说，立方体贴图就是一个包含了6个2D纹理的纹理，每个2D纹理都组成了立方体的一个面：一个有纹理的立方体。你可能会奇怪，这样一个立方体有什么用途呢？为什么要把6张纹理合并到一张纹理中，而不是直接使用6个单独的纹理呢？立方体贴图有一个非常有用的特性，它可以通过一个方向向量来进行索引/采样。假设我们有一个1x1x1的单位立方体，方向向量的原点位于它的中心。使用一个橘黄色的方向向量来从立方体贴图上采样一个纹理值会像是这样：
>
> ![cubemaps_sampling](https://ws4.sinaimg.cn/large/006tNc79ly1fp7qvvhw39j30b409y74h.jpg)
>
> &emsp;&emsp;方向向量的大小并不重要，只要提供了方向，OpenGL就会获取方向向量（最终）所击中的纹素，并返回对应的采样纹理值。
>
> &emsp;&emsp;如果我们假设将这样的立方体贴图应用到一个立方体上，采样立方体贴图所使用的方向向量将和立方体（插值的）顶点位置非常相像。这样子，只要立方体的中心位于原点，我们就能使用立方体的实际位置向量来对立方体贴图进行采样了。接下来，我们可以将所有顶点的纹理坐标当做是立方体的顶点位置。最终得到的结果就是可以访问立方体贴图上正确面(Face)纹理的一个纹理坐标。

&emsp;&emsp;立方体贴图也是纹理贴图，所以创建基本一样：

```c++
	unsigned int textureID;
	glGenTextures(1, &textureID);
	glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);
```

> &emsp;&emsp;因为立方体贴图包含有6个纹理，每个面一个，我们需要调用glTexImage2D函数6次，参数和之前教程中很类似。但这一次我们将纹理目标(**target**)参数设置为立方体贴图的一个特定的面，告诉OpenGL我们在对立方体贴图的哪一个面创建纹理。这就意味着我们需要对立方体贴图的每一个面都调用一次glTexImage2D。
>
> &emsp;&emsp;由于我们有6个面，OpenGL给我们提供了6个特殊的纹理目标，专门对应立方体贴图的一个面。
>
> | 纹理目标                         | 方位 |
> | -------------------------------- | ---- |
> | `GL_TEXTURE_CUBE_MAP_POSITIVE_X` | 右   |
> | `GL_TEXTURE_CUBE_MAP_NEGATIVE_X` | 左   |
> | `GL_TEXTURE_CUBE_MAP_POSITIVE_Y` | 上   |
> | `GL_TEXTURE_CUBE_MAP_NEGATIVE_Y` | 下   |
> | `GL_TEXTURE_CUBE_MAP_POSITIVE_Z` | 后   |
> | `GL_TEXTURE_CUBE_MAP_NEGATIVE_Z` | 前   |

&emsp;&emsp;这六个纹理目标都是枚举值，这代表着他们其实也是从 0 开始的 Int 值，也就是说只要我们按照相应的顺序排放贴图顺序，我们就可以使用 for 循环实现贴图。

```c++
	// 立方体贴图
    std::vector<std::string> faces{
            "../resources/shadowpeak_rt.tga", //right
            "../resources/shadowpeak_lf.tga", //left
            "../resources/shadowpeak_up.tga", //up
            "../resources/shadowpeak_dn.tga", //down
            "../resources/shadowpeak_bk.tga", //back
            "../resources/shadowpeak_ft.tga" //front
    };
```

&emsp;&emsp;之后，我们使用 for 遍历来贴图：

```c++
	GLint face_width, face_height, face_nrChannels;
    unsigned char *data;
    for (int i = 0; i < faces.size(); ++i) {
        data = stbi_load(faces[i].c_str(), &face_width, &face_height, &face_nrChannels, 0);
        glTexImage2D(static_cast<GLenum>(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i), 0, GL_RGB, face_width, face_height, 0,
                     GL_RGB, GL_UNSIGNED_BYTE, data);
    }
```

&emsp;&emsp;同时设定立方体贴图的线性过滤和环绕方式：

```c++
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
```

&emsp;&emsp;而在着色器中，我们使用 uniform samplerCube cubemap; 来创建立方体贴图的 uniform 对象，由于立方体贴图的特性，我们用立方体顶点坐标向量就可以对纹理进行采样。所以在顶点着色器中传入立方体顶点坐标，并将其传入给片段着色器，然后调用：

```GLSL
	FragColor = texture(cubemap, textureDir); //texture方向  即顶点坐标
```

&emsp;&emsp;我们可以使用立方体贴图来创造环境贴图，在[这里](http://www.custommapmakers.org/skyboxes.php)可以下载立方体贴图。本来按理说这里已经结束了，我们应该看到一个完美的环境贴图，但是启动后却发现并非如此：

![WX20180310-150737@2x](https://ws4.sinaimg.cn/large/006tNc79ly1fp7qvy5ku1j31kw17wdso.jpg)

&emsp;&emsp;可以看到，我们的上下贴图与前后左右贴图并没有完美的契合，这其实是一个 OpenGL 的问题，见此 [Convention of faces in OpenGL cubemapping](https://stackoverflow.com/questions/11685608/convention-of-faces-in-opengl-cubemapping) ，我们修改贴图数组的顺序使得其显示正确。

```c++
    // 立方体贴图
    std::vector<std::string> faces{
            "../resources/glacier_ft.tga", //front
            "../resources/glacier_bk.tga", //back
            "../resources/glacier_up.tga", //up
            "../resources/glacier_dn.tga", //down
            "../resources/glacier_rt.tga", //right
            "../resources/glacier_lf.tga", //left
    };
```

&emsp;&emsp;这样，显示成功。

![WX20180310-151142@2x](https://ws4.sinaimg.cn/large/006tNc79ly1fp7qvuzjevj31kw0zjgzt.jpg)

&emsp;&emsp;我们可以试着转动摄像机以查看身后或者其他方向，是不是一切正常了？完整代码在此：[GitHub](https://github.com/KsGin/LearnOpenGL/tree/master/Cubemaps)

