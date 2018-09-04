---
title: 【OpenGL】FPS模拟-1
date: 2018-03-01 22:29:38
tags: 
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;在学习了OpenGL最初步的一些内容，包括三维图形，**摄像机**，鼠标键盘操作后，不由得有了拿这些东西做个小练习的想法。之后又是自然而然的想到了小时候就开始玩的FPS游戏，FPS（First Person Shooter第一人称射击游戏）是可以对之前学到的东西进行综合利用的练习项目。

<!--more-->

&emsp;&emsp;在这个小练习中主要实现FPS视角，包括模型移动（摄像机移动），视野移动（摄像机旋转），地图模拟（几个大方块。。）。今天主要实现地图模拟。

&emsp;&emsp;在之前的学习项目中，主要以一个模型进行贴图等等的操作，而今天则需要实现的是一个CS地图的模拟（忘了叫什么名字了，就是四个大方块那个对枪的小地图）。

&emsp;&emsp;首先我们使用光照那一节的项目作为基础，将其拷贝过来，将旋转的方块模型和光照的模型分别做以下处理：

+ 光源升高且固定，作为全地图光源
+ 模型平铺展开（即对X和Z轴进行放大，且对Y轴进行缩小），作为地面
+ 摄像机放在正前方，并且摄像机平视（即将摄像机目标位置的Y轴与摄像机坐标Y轴相等）

```c++
    // Model matrix : an identity matrix (model will be at the origin)
    glm::mat4 floorModel = glm::mat4(1.0f) *
                           glm::scale(glm::mat4(1.0f), vec3(1000.0f, 0.2f, 1000.0f));
	// x * 1000  z * 1000  y * 0.2
    glm::mat4 lightModel = glm::mat4(1.0f) *
                           glm::translate(glm::mat4(1.0f), vec3(0.0f, 100.0f, 0.0f)) *
                           glm::scale(glm::mat4(1.0f), vec3(0.2f, 0.2f, 0.2f));
```

&emsp;&emsp;代码中我们将之前的模型矩阵更名为 floorModel ，并将其大小改为 X * 1000 ，Y * 0.2 ，Z * 1000 。以此作为地图，光源则将其向上平移100。

&emsp;&emsp;修改摄像机参数：

```c++
    // Camera matrix
    glm::mat4 View = glm::lookAt(
            glm::vec3(0, 5, 40),
            glm::vec3(0, 5, 0), 
            glm::vec3(0, 1, 0)  // Head is up (set to 0,-1,0 to look upside-down)
    );
```

&emsp;&emsp;效果如下：

![WechatIMG1992](https://ws4.sinaimg.cn/large/006tNc79ly1foxppvako3j31kq16adhl.jpg)

&emsp;&emsp;做完这些后，我们需要找出四个大方块，在此我们构建四个Model模型：

```c++
    glm::mat4 wall1 = glm::mat4(1.0f) *
                      glm::translate(glm::mat4(1.0f), vec3(-15.0f, 0.0f, 15.0f)) *
                      glm::scale(glm::mat4(1.0f), vec3(20.0f, 40.0f, 20.0f));

    glm::mat4 wall2 = glm::mat4(1.0f) *
                      glm::translate(glm::mat4(1.0f), vec3(15.0f, 0.0f, 15.0f)) *
                      glm::scale(glm::mat4(1.0f), vec3(20.0f, 40.0f, 20.0f));

    glm::mat4 wall3 = glm::mat4(1.0f) *
                      glm::translate(glm::mat4(1.0f), vec3(-15.0f, 0.0f, -15.0f)) *
                      glm::scale(glm::mat4(1.0f), vec3(20.0f, 40.0f, 20.0f));

    glm::mat4 wall4 = glm::mat4(1.0f) *
                      glm::translate(glm::mat4(1.0f), vec3(15.0f, 0.0f, -15.0f)) *
                      glm::scale(glm::mat4(1.0f), vec3(20.0f, 40.0f, 20.0f));
```

&emsp;&emsp;（其实可以用一个模型进行平移所得，我这里是直接复制的。主要为模拟效果，在之后学习了更多东西后会好好写一个FPS视角。）由代码就可以看出我们的四个模型大小相同，但是位置不一样，并且将其放在了我们的地面上。

```c++
        glUseProgram(cudeProgram);
        glUniformMatrix4fv(cudeViewID, 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(cudeModelID, 1, GL_FALSE, &wall1[0][0]);
        glUniformMatrix4fv(cudeProjectionID, 1, GL_FALSE, &Projection[0][0]);
        glUniform3f(glGetUniformLocation(cudeProgram, "viewPos"), 4.0f, 5.0f, 3.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightPos"), 0.0f, 10.0f, 0.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "objectColor"), 1.0f, 0.5f, 0.31f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightColor"), 1.0f, 1.0f, 1.0f);
        glBindVertexArray(cudeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);


        glUseProgram(cudeProgram);
        glUniformMatrix4fv(cudeViewID, 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(cudeModelID, 1, GL_FALSE, &wall2[0][0]);
        glUniformMatrix4fv(cudeProjectionID, 1, GL_FALSE, &Projection[0][0]);
        glUniform3f(glGetUniformLocation(cudeProgram, "viewPos"), 4.0f, 5.0f, 3.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightPos"), 0.0f, 10.0f, 0.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "objectColor"), 1.0f, 0.5f, 0.31f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightColor"), 1.0f, 1.0f, 1.0f);
        glBindVertexArray(cudeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);

        glUseProgram(cudeProgram);
        glUniformMatrix4fv(cudeViewID, 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(cudeModelID, 1, GL_FALSE, &wall3[0][0]);
        glUniformMatrix4fv(cudeProjectionID, 1, GL_FALSE, &Projection[0][0]);
        glUniform3f(glGetUniformLocation(cudeProgram, "viewPos"), 4.0f, 5.0f, 3.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightPos"), 0.0f, 10.0f, 0.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "objectColor"), 1.0f, 0.5f, 0.31f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightColor"), 1.0f, 1.0f, 1.0f);
        glBindVertexArray(cudeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);

        glUseProgram(cudeProgram);
        glUniformMatrix4fv(cudeViewID, 1, GL_FALSE, &View[0][0]);
        glUniformMatrix4fv(cudeModelID, 1, GL_FALSE, &wall4[0][0]);
        glUniformMatrix4fv(cudeProjectionID, 1, GL_FALSE, &Projection[0][0]);
        glUniform3f(glGetUniformLocation(cudeProgram, "viewPos"), 4.0f, 5.0f, 3.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightPos"), 0.0f, 10.0f, 0.0f);
        glUniform3f(glGetUniformLocation(cudeProgram, "objectColor"), 1.0f, 0.5f, 0.31f);
        glUniform3f(glGetUniformLocation(cudeProgram, "lightColor"), 1.0f, 1.0f, 1.0f);
        glBindVertexArray(cudeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
        glBindVertexArray(0);
```

&emsp;&emsp;（不要在意这些相当于复制粘贴的代码）在这里我们将四个方块画了下来。

![WechatIMG1993](https://ws3.sinaimg.cn/large/006tNc79ly1foxppw99mgj31ks15oq4z.jpg)

&emsp;&emsp;如此，我们大概的地图模型算是成功了。

&emsp;&emsp;下一篇中将实现人物的前后左右走动以及跳跃（摄像机的前后左右走动以及跳跃）。