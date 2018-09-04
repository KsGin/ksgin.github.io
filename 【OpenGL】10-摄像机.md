---
title: 【OpenGL】10-摄像机
date: 2018-02-20 21:36:12
tags:
	- OpenGL
	- Computer graphics
typora-copy-images-to: ipic
---

#### 参考教程

【中文版】[摄像机](https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/)

【英文版】[Camera](https://learnopengl.com/Getting-started/Camera)

#### 学习心得

&emsp;&emsp;在开始这一篇之前，我们需要清楚OpenGL中的坐标系规则，OpenGL采用右手坐标系，即伸出右手，让大拇指指向右方，食指向上，剩下手指弯曲90度指向自己，这即是X，Y，Z三轴。

<!--more-->

![right_hand](https://ws3.sinaimg.cn/large/006tNc79ly1fon8v75xooj30ee0ekjxc.jpg)

&emsp;&emsp;关于右手坐标系的具体情况可以参考：[左右手坐标系](http://math001.com/left_hand_right_hand_coordinate/)

&emsp;&emsp;在我们之前的代码中，有这么一段：

```c++
    glm::mat4 View = glm::lookAt(
            glm::vec3(0, 0, 5),
            glm::vec3(0, 0, 0), 
            glm::vec3(0, 1, 0)  
    );
```

&emsp;&emsp;在这段中我们调用了 [glm::lookAt](https://glm.g-truc.net/0.9.2/api/a00245.html#ga2d6b6c381f047ea4d9ca4145fed9edd5) 方法，并给其传入了三个vec3作为参数，那么这个方法到底是怎么操作的呢，首先看看这个方法的介绍：

```c++
detail::tmat4x4<T> glm::gtc::matrix_transform::lookAt	(	
    				detail::tvec3< T > const & 	eye,
					detail::tvec3< T > const & 	center,
					detail::tvec3< T > const & 	up 
)	
```

> Build a look at view matrix.

&emsp;&emsp;这个方法用来构建一个view矩阵，而我们需要给出的三个vec3分别是摄像机的坐标，摄像机的目标坐标，以及一个Up向量，摄像机的位置坐标和摄像机的观察目标自然不用说，那么这个Up向量又是做什么的呢？

&emsp;&emsp;我们需要一个摄像机区域的 Right 向量表示摄像机空间里的X轴正方向，这个X轴正方向我们可以通过摄像机观察的方向（即摄像机->观察点的方向）与一个正向上的向量来进行叉乘来获得。

&emsp;&emsp;当我们提供了摄像机坐标，观察点坐标以及一个向上的向量之后， lookAt 方法将为我们创建一个看着给定的观察点的矩阵（glm省去了我们的计算）。

> &emsp;&emsp;对于想学到更多数学原理的读者，提示一下，在线性代数中这个处理叫做[格拉姆—施密特正交化](http://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process)(Gram-Schmidt Process)。使用这些摄像机向量我们就可以创建一个LookAt矩阵了，它在创建摄像机的时候非常有用。

&emsp;&emsp;现在我们已经大致清楚了摄像机的使用，那么之前我们使用Model进行平移旋转等变换，现在则使用View矩阵的修改实现正方体观察的变换。首先关于平移我们可以通过修改观察点实现，而旋转可以通过修改摄像机的坐标来实现。

> &emsp;&emsp;请记住摄像机的变换和模型的变换是相反的，正如汽车向右走，那么坐在车里的我们可以看到路两旁的树在向左走。

```c++
        glm::mat4 Projection = glm::perspective(glm::radians(90.0f), 4.0f / 3.0f, 0.1f, 10.0f);
        // Camera matrix
        glm::mat4 View       = glm::lookAt(
                glm::vec3(0,0,5), // Camera is at (0,0,5), in World Space
                glm::vec3(0,0,0), // and looks at the origin
                glm::vec3(0,1,0)  // Head is up (set to 0,1,0 to look upside-down)
        );
        // Model matrix : an identity matrix (model will be at the origin)
        glm::mat4 Model      = glm::mat4(1.0f);
        // Our ModelViewProjection : multiplication of our 3 matrices
        glm::mat4 MVP        = Projection * View * Model; // Remember, matrix multiplication is the other way around
```

&emsp;&emsp;首先看看我们之前的M，V，P三个矩阵的定义（在[纹理](https://blog.ksgin.com/2018/02/14/%E3%80%90opengl%E3%80%917-%E7%BA%B9%E7%90%862/)那篇文章时候的代码），现在我们通过修改 LookAt 中的观察点位置来模拟正方体的移动。

```c++
    float X = 0;
    int direct = 0;
    do{
        if(X > 5.0f || X < -5.0f){
            direct = !direct;
        }
        if(direct) X += 0.1f;
        else X -= 0.1f;

        ...
            
        glm::mat4 View       = glm::lookAt(
                glm::vec3(0,0,5), 
                glm::vec3(X,0,0), 
                glm::vec3(0,1,0)  
        );
        
        ...
            
    }while(...)
```

&emsp;&emsp;由上边的代码可以看到，我们修改了 LookAt 方法的观察点参数，并让其在 5.0f 到 -5.0f 间交替，使得正方体在可见区域内来回移动。

[MD不支持视频，我上传了Youtube，点这里看效果](https://youtu.be/q9h24IpLrPw)

&emsp;&emsp;接下来我们通过修改摄像机的位置使得摄像机绕着正方体运动，同时摄像机的观察点一直在圆心（0,0,0），这种情况下我们可以模拟看到正方体的转动（其实是摄像机在转，与移动同理）。

&emsp;&emsp;首先计算绕X轴旋转的Z轴坐标，绕Y轴旋转的话那么需要计算X和Z轴的值，我们使用 glfwGetTime() 获得一个变值，然后在X坐标上对其进行 Cos 计算，在Z坐标上对其进行 Sin 计算（绕Y轴旋转，Y轴不变）。

```C++
       glm::mat4 View       = glm::lookAt(
                glm::vec3(5 * cos(glfwGetTime()),0,5 * sin(glfwGetTime())), 
                glm::vec3(0,0,0), 
                glm::vec3(0,1,0)  
        );
```

&emsp;&emsp;这样，我们可以看到一个以我们观察角度来看是顺时针旋转的正方体（其实是摄像机在逆时针旋转），如果对X 进行 Sin 计算而 Y 进行 Cos 计算的话那么更好相反。

[MD不支持视频，我上传了Youtube，点这里看效果](https://youtu.be/6ICo0PoseTs)

