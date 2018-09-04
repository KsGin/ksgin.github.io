---
title: 【OpenGL】12-模型与Assimp库
date: 2018-03-04 15:17:53
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[Assimp](https://learnopengl-cn.github.io/03%20Model%20Loading/01%20Assimp/) [网格](https://learnopengl-cn.github.io/03%20Model%20Loading/02%20Mesh/)  [模型](https://learnopengl-cn.github.io/03%20Model%20Loading/03%20Model/)

【英文版】[Assimp](https://learnopengl.com/Model-Loading/Assimp) [Mesh](https://learnopengl.com/Model-Loading/Mesh) [Model](https://learnopengl.com/Model-Loading/Model)

#### 学习心得

&emsp;&emsp;在之前我们一直使用正方体作为模型，现在终于可以接触别的了。在我们所玩的游戏，或者我们现实中所接触的东西，大都是不规则形状的，很难去简单的用几个三角形构成，这也是我们这一篇介绍的重点。

<!--more-->

&emsp;&emsp;首先需要了解一下 [Assimp](https://en.wikipedia.org/wiki/Open_Asset_Import_Library) ，以下是维基百科介绍（并没有找到中文的）：

> # Open Asset Import Library
>
> From Wikipedia, the free encyclopedia
>
> | [Developer(s)](https://en.wikipedia.org/wiki/Software_developer) | Alexander GesslerThomas SchulzeKim Kulling, et al.           |
> | ------------------------------------------------------------ | ------------------------------------------------------------ |
> | [Stable release](https://en.wikipedia.org/wiki/Software_release_life_cycle) | 4.1.0 / December 11, 2017; 2 months ago                      |
> | [Repository](https://en.wikipedia.org/wiki/Repository_(version_control)) | <https://github.com/assimp/assimp>[![Edit this at Wikidata](https://upload.wikimedia.org/wikipedia/commons/thumb/7/73/Blue_pencil.svg/10px-Blue_pencil.svg.png)](https://www.wikidata.org/wiki/Q6363636#P1324) |
> | [Operating system](https://en.wikipedia.org/wiki/Operating_system) | [Cross-platform](https://en.wikipedia.org/wiki/Cross-platform) |
> | [Type](https://en.wikipedia.org/wiki/Software_categories#Broad_categories) | [3D model](https://en.wikipedia.org/wiki/3D_model) import [library](https://en.wikipedia.org/wiki/Library_(computing)) |
> | [License](https://en.wikipedia.org/wiki/Software_license)    | [BSD](https://en.wikipedia.org/wiki/BSD_licenses)            |
> | Website                                                      | [www.assimp.org](http://www.assimp.org/)                     |
>
> **Open Asset Import Library** (**Assimp**) is a [cross-platform](https://en.wikipedia.org/wiki/Cross-platform) [3D model](https://en.wikipedia.org/wiki/3D_model) import library which aims to provide a common [application programming interface](https://en.wikipedia.org/wiki/Application_programming_interface) (API) for different [3D asset file formats](https://en.wikipedia.org/wiki/List_of_file_formats#3D_graphics). Written in [C++](https://en.wikipedia.org/wiki/C%2B%2B), it offers interfaces for both [C](https://en.wikipedia.org/wiki/C_(programming_language)) and C++. Bindings to other languages (e.g., [BlitzMax](https://en.wikipedia.org/wiki/Blitz_BASIC), [C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)), [Python](https://en.wikipedia.org/wiki/Python_(programming_language))) are developed as part of the project or are available elsewhere.[[1\]](https://en.wikipedia.org/wiki/Open_Asset_Import_Library#cite_note-1) Given the importance and the benefits of Assimp, a pure Java (/Kotlin) port is being developed [here](https://github.com/java-graphics/assimp).
>
> The imported data is provided in a straightforward, hierarchical data structure. Configurable post processing steps (i.e., normal and tangent generation, various optimizations) augment the feature set.[[2\]](https://en.wikipedia.org/wiki/Open_Asset_Import_Library#cite_note-2)
>
> Assimp currently supports 57 different file formats for reading, including [COLLADA](https://en.wikipedia.org/wiki/COLLADA) (.dae), [3DS](https://en.wikipedia.org/wiki/.3ds), [DirectX](https://en.wikipedia.org/wiki/DirectX) [X](https://en.wikipedia.org/wiki/.x), [Wavefront OBJ](https://en.wikipedia.org/wiki/Wavefront_.obj_file) and [Blender 3D](https://en.wikipedia.org/wiki/Blender_(software)) (.blend).[[3\]](https://en.wikipedia.org/wiki/Open_Asset_Import_Library#cite_note-3) As of Version 3.0 Assimp also provides export functionality for some file formats.[[4\]](https://en.wikipedia.org/wiki/Open_Asset_Import_Library#cite_note-4)
>
> ## See also
>
> - [Free software portal](https://en.wikipedia.org/wiki/Portal:Free_software)
>
>
> - [OpenCTM](https://en.wikipedia.org/wiki/OpenCTM)
> - [MeshLab](https://en.wikipedia.org/wiki/MeshLab)
>
> ## References
>
> 1. http://assimp.sourceforge.net/main_doc.html>
> 2. http://assimp.sourceforge.net/main_features_pp.html>
> 3. http://assimp.sourceforge.net/main_features_formats.html>
> 4. http://assimp.sourceforge.net/main_features_export.html>

&emsp;&emsp;简单来说，这就是一个开源的模型加载库，一套通用的 SDK 。主要用于导入模型 （OBJ），以及将模型处理成我们 OpenGL 中所需要的数据。

> &emsp;&emsp;Assimp能够导入很多种不同的模型文件格式（并也能够导出部分的格式），它会将所有的模型数据加载至Assimp的通用数据结构中。当Assimp加载完模型之后，我们就能够从Assimp的数据结构中提取我们所需的所有数据了。由于Assimp的数据结构保持不变，不论导入的是什么种类的文件格式，它都能够将我们从这些不同的文件格式中抽象出来，用同一种方式访问我们需要的数据。
>
> &emsp;&emsp;当使用Assimp导入一个模型的时候，它通常会将整个模型加载进一个**场景**(Scene)对象，它会包含导入的模型/场景中的所有数据。Assimp会将场景载入为一系列的节点(Node)，每个节点包含了场景对象中所储存数据的索引，每个节点都可以有任意数量的子节点。Assimp数据结构的（简化）模型如下：
>
> ![img](https://learnopengl-cn.github.io/img/03/01/assimp_structure.png)
>
> - 和材质和网格(Mesh)一样，所有的场景/模型数据都包含在Scene对象中。Scene对象也包含了场景根节点的引用。
> - 场景的Root node（根节点）可能包含子节点（和其它的节点一样），它会有一系列指向场景对象中mMeshes数组中储存的网格数据的索引。Scene下的mMeshes数组储存了真正的Mesh对象，节点中的mMeshes数组保存的只是场景中网格数组的索引。
> - 一个Mesh对象本身包含了渲染所需要的所有相关数据，像是顶点位置、法向量、纹理坐标、面(Face)和物体的材质。
> - 一个网格包含了多个面。Face代表的是物体的渲染图元(Primitive)（三角形、方形、点）。一个面包含了组成图元的顶点的索引。由于顶点和索引是分开的，使用一个索引缓冲来渲染是非常简单的（见[你好，三角形](https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/)）。
> - 最后，一个网格也包含了一个Material对象，它包含了一些函数能让我们获取物体的材质属性，比如说颜色和纹理贴图（比如漫反射和镜面光贴图）。
>
> &emsp;&emsp;所以，我们需要做的第一件事是将一个物体加载到Scene对象中，遍历节点，获取对应的Mesh对象（我们需要递归搜索每个节点的子节点），并处理每个Mesh对象来获取顶点数据、索引以及它的材质属性。最终的结果是一系列的网格数据，我们会将它们包含在一个`Model`对象中。
>
> **网格**
>
> &emsp;&emsp;当使用建模工具对物体建模的时候，艺术家通常不会用单个形状创建出整个模型。通常每个模型都由几个子模型/形状组合而成。组合模型的每个单独的形状就叫做一个网格(Mesh)。比如说有一个人形的角色：艺术家通常会将头部、四肢、衣服、武器建模为分开的组件，并将这些网格组合而成的结果表现为最终的模型。一个网格是我们在OpenGL中绘制物体所需的最小单位（顶点数据、索引和材质属性）。一个模型（通常）会包括多个网格。

&emsp;&emsp;在 mac os 下 ，Assimp 的安装比较简单，我们直接使用 `brew install assimp` 命令即可安装，安装好后可以在 `/usr/local/lib` 目录下可以找到 `libassimp.dylib libassimp.x.dylib libassimp.x.x.x.dylib` 三个不同版本的 lib 文件（xxx是版本数字，写者安装的时候是4.0.1）。

&emsp;&emsp;在 windows 下，在 [assimp 下载地址](https://github.com/assimp/assimp/releases/tag/v3.3.1/) 下载源代码，然后使用 CMake 构建对应的 VS 工程，生成 assimp 那个项目得到 .dll 或者 .lib 文件，之后将 include 文件夹和 .lib 文件移到对应的位置就可以了。我们之前配置 glwf 和 glew 时做过类似的事。所以这次简单说明下，按照之前的操作方法就可以了。 