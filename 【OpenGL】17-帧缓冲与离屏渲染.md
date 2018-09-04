---
title: 【OpenGL】17-帧缓冲与离屏渲染
date: 2018-03-08 10:33:55
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[帧缓冲](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/05%20Framebuffers/)

【英文版】[Framebuffers](https://learnopengl.com/Advanced-OpenGL/Framebuffers)

#### 学习记录

&emsp;&emsp;[帧缓冲（Framebuffer Object, FBO）](http://blog.csdn.net/jxw167/article/details/54985183)，是 OpenGL 中实现**后处理**效果的核心技术，在我们之前的代码中，所有的不论是颜色缓冲，深度缓冲等等都是使用的默认帧缓冲，即在主循环中实现渲染并且反映到窗口中。现在我们在默认缓冲之外创建一个自己的帧缓冲对象，在当前帧缓冲之外渲染，以实现一些炫酷的效果。请注意我们使用我们自定义的缓冲对象进行渲染的时候，他是不会显示在主窗口的，所以又称为**离屏渲染**。

<!--more-->

&emsp;&emsp;帧缓冲的创建和其他缓冲类似：

```c++
	GLuint FBO;
    glGenFramebuffers(1, &FBO);
    glBindFramebuffer(GL_FRAMEBUFFER, FBO);
```

&emsp;&emsp;注意，当我们使用了 `glBindFramebuffer(GL_FRAMEBUFFER, FBO);`  绑定帧缓冲之后，我们之后所做的渲染都是在 FBO 缓冲上，如果我们要返回默认缓冲，则使用：

```c++
	glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

> &emsp;&emsp;我们也可以使用GL_READ_FRAMEBUFFER或GL_DRAW_FRAMEBUFFER，将一个帧缓冲分别绑定到读取目标或写入目标。绑定到GL_READ_FRAMEBUFFER的帧缓冲将会使用在所有像是glReadPixels的读取操作中，而绑定到GL_DRAW_FRAMEBUFFER的帧缓冲将会被用作渲染、清除等写入操作的目标。大部分情况你都不需要区分它们，通常都会使用GL_FRAMEBUFFER，绑定到两个上。
>
> &emsp;&emsp;创建了缓冲之后，我们需要为缓冲添加内容，现在的它还不完整(Complete)，一个完整的帧缓冲需要满足以下的条件：
>
> - 附加至少一个缓冲（颜色、深度或模板缓冲）。
> - 至少有一个颜色附件(Attachment)。
> - 所有的附件都必须是完整的（保留了内存）。
> - 每个缓冲都应该有相同的样本数。
>
> &emsp;&emsp;如果你不知道什么是样本，不要担心，我们将在[之后的](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/11%20Anti%20Aliasing/)教程中讲到。

&emsp;&emsp;我们可以主动判断帧缓冲是否完整：

```c++
    if (glCheckFramebufferStatus(FBO) != GL_FRAMEBUFFER_COMPLETE) {
        std::cout << "FRAME::BUFFER::ERROR" << std::endl;
    }
```

&emsp;&emsp;现在，为帧缓冲添加一个纹理贴图附件，首先创建纹理：

```c++
	GLuint texture;
    glGenTextures(1, &texture);
    glBindTexture(GL_TEXTURE_2D, texture);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, WIDTH, HEIGHT, 0, GL_RGB, GL_UNSIGNED_BYTE, nullptr);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glBindTexture(GL_TEXTURE_2D, 0);
```

&emsp;&emsp;请注意我们 glTexImage2D 方法中 data 的参数设置了 nullptr ，是因为我们要将纹理绑定为帧缓冲的附件，使得我们所渲染的画面成为一个纹理（然后可以将它作为贴图贴到其他地方）。接下来绑定纹理：

```c++
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```

> &emsp;&emsp;glFrameBufferTexture2D有以下的参数：
>
> - `target`：帧缓冲的目标（绘制、读取或者两者皆有）
> - `attachment`：我们想要附加的附件类型。当前我们正在附加一个颜色附件。注意最后的`0`意味着我们可以附加多个颜色附件。我们将在之后的教程中提到。
> - `textarget`：你希望附加的纹理类型
> - `texture`：要附加的纹理本身
> - `level`：多级渐远纹理的级别。我们将它保留为0。

&emsp;&emsp;在绑定纹理后，我们还需要为其添加深度缓冲和模板缓冲，使用：

```c++
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_DEPTH_COMPONENT, texture, 0);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_STENCIL_ATTACHMENT, GL_STENCIL_INDEX, texture, 0);
```

&emsp;&emsp;或者将其放在一起：

```c++
	glTexImage2D(
  		GL_TEXTURE_2D, 0, GL_DEPTH24_STENCIL8, WIDTH, HEIGHT, 0, 
  		GL_DEPTH_STENCIL, GL_UNSIGNED_INT_24_8, NULL
	);
	glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, texture, 0);
```

&emsp;&emsp;除了这两种，我们来介绍另一个东西：**渲染缓冲对象附件**

> &emsp;&emsp;渲染缓冲对象(Renderbuffer Object)是在纹理之后引入到OpenGL中，作为一个可用的帧缓冲附件类型的，所以在过去纹理是唯一可用的附件。和纹理图像一样，渲染缓冲对象是一个真正的缓冲，即一系列的字节、整数、像素等。渲染缓冲对象附加的好处是，它会将数据储存为OpenGL原生的渲染格式，它是为离屏渲染到帧缓冲优化过的。
>
> &emsp;&emsp;渲染缓冲对象直接将所有的渲染数据储存到它的缓冲中，不会做任何针对纹理格式的转换，让它变为一个更快的可写储存介质。然而，渲染缓冲对象通常都是只写的，所以你不能读取它们（比如使用纹理访问）。当然你仍然还是能够使用glReadPixels来读取它，这会从当前绑定的帧缓冲，而不是附件本身，中返回特定区域的像素。
>
> &emsp;&emsp;因为它的数据已经是原生的格式了，当写入或者复制它的数据到其它缓冲中时是非常快的。所以，交换缓冲这样的操作在使用渲染缓冲对象时会非常快。我们在每个渲染迭代最后使用的glfwSwapBuffers，也可以通过渲染缓冲对象实现：只需要写入一个渲染缓冲图像，并在最后交换到另外一个渲染缓冲就可以了。渲染缓冲对象对这种操作非常完美。

&emsp;&emsp;既然也是缓冲对象，那么使用类似：

```c++
	GLuint RBO;
    glGenRenderbuffers(1, &RBO);
    glBindRenderbuffer(GL_RENDERBUFFER, RBO);
    glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, WIDTH, HEIGHT);
    glBindRenderbuffer(GL_RENDERBUFFER, 0);
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, RBO);
```

&emsp;&emsp;我们在 glRenderbufferStorage 的参数里设置内部格式为：GL_DEPTH24_STENCIL8 （即 24 位深度 + 8 位模板缓冲）。

&emsp;&emsp;现在我们将颜色缓冲，深度缓冲，模板缓冲都加在了帧缓冲里，现在为其的纹理进行渲染，我们使用之前光照的部分场景，将其渲染到帧缓冲内部：

```c++
  	do {

        glBindFramebuffer(GL_FRAMEBUFFER, FBO);  //以下内容绘制在自定义的帧缓冲里

        glEnable(GL_DEPTH_TEST);

        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

        glUseProgram(cudeProgram);
        glUniformMatrix4fv(glGetUniformLocation(cudeProgram, "view"), 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(glGetUniformLocation(cudeProgram, "model"), 1, GL_FALSE, &cudeModel[0][0]);
        glUniformMatrix4fv(glGetUniformLocation(cudeProgram, "projection"), 1, GL_FALSE, &Projection[0][0]);
        glUniform3f(glGetUniformLocation(cudeProgram, "viewPos"), 4.0f, 5.0f, 3.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightPos"), 0.0f, 1.0f, 0.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "objectColor"), 1.0f, 0.5f, 0.31f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightColor"), 1.0f, 1.0f, 1.0f);
        glBindVertexArray(cudeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);

        glUseProgram(lightProgram);
        glUniformMatrix4fv(glGetUniformLocation(lightProgram, "projection"), 1, GL_FALSE, &Projection[0][0]);
        glUniformMatrix4fv(glGetUniformLocation(lightProgram, "view"), 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(glGetUniformLocation(lightProgram, "model"), 1, GL_FALSE, &lightModel[0][0]);
        glBindVertexArray(lightVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);

        // Swap buffers
        glfwSwapBuffers(window);
        glfwPollEvents();

    } while (glfwGetKey(window, GLFW_KEY_ESCAPE) != GLFW_PRESS &&
             glfwWindowShouldClose(window) == 0);
```

&emsp;&emsp;我们在 do-while 的循环里第一行绑定了缓冲为我们所定义的帧缓冲 FBO，之后的代码里所做的渲染将都渲染到 FBO 里，并变成可以使用的纹理（还记得之前创建的那个 data 为空的纹理吗）。现在如果启动，黑色窗口将不会有任何显示，因为并没有在主窗口的默认缓冲放入任何内容。

&emsp;&emsp;现在我们创建一个窗口大小的长方形，并将纹理贴在上方（这样的显示与我们没有使用帧缓冲是一致的），首先创建长方形数组（代码请见 [Github](https://github.com/KsGin/LearnOpenGL/blob/master/Framebuffers) ），并配置相应的 VAO , VBO 以及着色器程序，在 Github 都可以找到。最后，我们在 do-while 循环里将帧缓冲绑定为默认缓冲，并为长方形附上纹理。

```c++
	do{
        ...
		glBindFramebuffer(GL_FRAMEBUFFER, 0); //以下为默认窗口帧缓冲
        glDisable(GL_DEPTH_TEST);
        glClear(GL_COLOR_BUFFER_BIT);

        glUseProgram(screenProgram);
        glUniform1i(glGetUniformLocation(screenProgram, "screenTexture"), 0);
        glBindTexture(GL_TEXTURE_2D, texture);

        glBindVertexArray(screenVAO);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);
        ...
    }(...)
```

&emsp;&emsp;我们在帧缓冲里渲染的内容作为一个纹理贴图在我们所画的长方形上成功显示。

&emsp;&emsp;当然，帧缓冲的作用并不是让我们如此去多次一举的写代码，而是实现很多很好玩的特效，因为现在我们将之前的缓冲做成纹理，所以我们可以在着色器里对纹理做很多的改动，例如修改颜色为 x = 1 - x （反相）等等。 