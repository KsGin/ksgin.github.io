---
title: 【OpenGL】6-纹理1
date: 2018-01-26 20:13:23
tags: 
     - OpenGL
     - Computer graphics
---

#### 参考教程

【中文版】[第五课：带纹理的立方体](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-5-a-textured-cube/)

【英文版】[Tutorial 5 : A Textured Cube](http://www.opengl-tutorial.org/beginners-tutorials/tutorial-5-a-textured-cube/)

&emsp;&emsp;感觉自己学的好慢啊。

&emsp;&emsp;纹理（Texture），在计算机图形学中是把存储在内存里的位图包裹到3D渲染物体的表面。 **纹理**贴图给物体提供了丰富的细节，用简单的方式模拟出了复杂的外观。 一个图像（**纹理**）被贴(映射)到场景中的一个简单形体上，就像印花贴到一个平面上一样。

​	<!--more-->

&emsp;&emsp;在OpenGL中，纹理的初始化及使用与顶点缓冲，颜色缓冲基本相同（本身纹理就相当于特殊的颜色）。主要的方法也就那几个，加载`glGenTextures`，   绑定`glBindTexture`，配置`glTexImage2D`（*在glTexImage2D函数中，GL_RGB表示颜色由三个分量构成，GL_BGR则说明了颜色在内存中的存储格式。实际上，BMP存储的并不是RGB，而是BGR，因此得把这个告诉OpenGL*）。

&emsp;&emsp;在着色器中，使用Sampler2D来获知要加载哪一个纹理（同一个着色器中可以访问多个纹理）。顶点着色器中则是将**UV坐标**以UV数对的方式读取并传递给片段着色器。 

> # 关于UV坐标
>
> 给模型贴纹理时，我们需要通过UV坐标来告诉OpenGL用哪块图像填充三角形。
>
> 每个顶点除了位置坐标外还有两个浮点数坐标：U和V。这两个坐标用于访问纹理，如下图所示：
>
> ![img](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/UVintro.png)
>
> 注意观察纹理是怎样在三角形上扭曲的。

&emsp;&emsp;而读取UV数对的方式就如同我们读取颜色缓冲数组或者顶点缓冲数组一样，使用`layout(location = 1) in vec2 vertexUV;`，读取到之后使用texture方法获得纹理并将他的rgb赋值给color（纹理是一种特殊的color），使用方法`color = texture( myTextureSampler, UV ).rgb;`。

&emsp;&emsp;在最终的代码中，我们使用了过滤，下面是过滤和mipmap的介绍：

> # 什么是过滤和mipmap？怎样使用？
>
> 正如在上面截图中看到的，纹理质量不是很好。这是因为在loadBMP_custom函数中，有如下两行代码：
>
> ```
> glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
> glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
>
> ```
>
> 这意味着在片段着色器中，texture()将直接提取位于(U,V)坐标的纹素（texel）。
>
> ![img](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/nearest.png)
>
> 有几种方法可以改善这一状况。
>
> ## 线性过滤（Linear filtering）
>
> 若采用线性过滤。texture()会查看周围的纹素，然后根据UV坐标距离各纹素中心的距离来混合颜色。这就避免了前面看到的锯齿状边缘。
>
> ![img](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/linear1.png)
>
> 线性过滤可以显著改善纹理质量，应用的也很多。但若想获得更高质量的纹理，可以采用各向异性过滤，不过速度有些慢。
>
> ## 各向异性过滤（Anisotropic filtering）
>
> 这种方法逼近了真正片断中的纹素区块。例如下图中稍稍旋转了的纹理，各向异性过滤将沿蓝色矩形框的主方向，作一定数量的采样（即所谓的”各向异性层级”），计算出其内的颜色。
>
> ![img](http://www.opengl-tutorial.org/assets/images/tuto-5-textured-cube/aniso.png)
>
> ## Mipmaps
>
> 线性过滤和各向异性过滤都存在一个共同的问题。那就是如果从远处观察纹理，只对4个纹素作混合显得不够。实际上，如果3D模型位于很远的地方，屏幕上只看得见一个片断（像素），那计算平均值得出最终颜色值时，图像所有的纹素都应该考虑在内。很显然，这种做法没有考虑性能问题。撇开两种过滤方法不谈，这里要介绍的是mipmap技术：
>
> ![img](http://upload.wikimedia.org/wikipedia/commons/5/5c/MipMap_Example_STS101.jpg)
>
> - 一开始，把图像缩小到原来的1/2，然后依次缩小，直到图像只有1x1大小（应该是图像所有纹素的平均值）
> - 绘制模型时，根据纹素大小选择合适的mipmap。
> - 可以选用nearest、linear、anisotropic等任意一种滤波方式来对mipmap采样。
> - 要想效果更好，可以对两个mipmap采样然后混合，得出结果。
>
> 好在这个比较简单，OpenGL都帮我们做好了，只需一个简单的调用：
>
> ```
> // When MAGnifying the image (no bigger mipmap available), use LINEAR filtering
> glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
> // When MINifying the image, use a LINEAR blend of two mipmaps, each filtered LINEARLY too
> glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
> // Generate mipmaps, by the way.
> glGenerateMipmap(GL_TEXTURE_2D);
> ```

&emsp;&emsp;在我们的代码中使用了自己写的bmp读取方法，实际上在OpenGL中提供了这个方法，调用如下

```c++
GLuint loadTGA_glfw(const char * imagepath){

    // Create one OpenGL texture
    GLuint textureID;
    glGenTextures(1, &textureID);

    // "Bind" the newly created texture : all future texture functions will modify this texture
    glBindTexture(GL_TEXTURE_2D, textureID);

    // Read the file, call glTexImage2D with the right parameters
    glfwLoadTexture2D(imagepath, 0);

    // Nice trilinear filtering.
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
    glGenerateMipmap(GL_TEXTURE_2D);

    // Return the ID of the texture we just created
    return textureID;
}
```

&emsp;&emsp;即`glfwLoadTexture2D`方法。

&emsp;&emsp;[源代码](https://github.com/KsGin/LearnOpenGL/tree/master/Textures)