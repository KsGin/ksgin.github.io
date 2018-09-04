---
title: 【OpenGL】FPS模拟-2
date: 2018-03-02 21:52:25
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;上一篇中我们进行了地图的构造，而这一篇文章中我们将进行任务移动的处理，首先在FPS模拟中，我们所谓的人物其实就是摄像机，即摄像机位移。

<!--more-->

![WechatIMG1993](https://ws2.sinaimg.cn/large/006tNc79ly1foyuuhg35kj31ks15oq4z.jpg)

&emsp;&emsp;图中可以看到，我们的视角是直视前方的，即与地面平行。在大部分FPS游戏中是以鼠标控制镜头方向，以键盘W，A，S，D控制人物位置，即摄像机位置。

&emsp;&emsp;这个比较简单，我们直接使用OpenGL的键盘操作，判断键位并进行摄像机的位移就可以了。`glfwGetKey(window , GLFW_KEY_W) == GLFW_PRESS`判断是否有W键按下，然后进行相应的摄像机像Z轴方向平移。

```c++
        if(glfwGetKey(window , GLFW_KEY_W) == GLFW_PRESS){
            View = View * glm::translate(glm::mat4(1.0f) , vec3(0.0f ,0.0f , 0.1f));
        }
```

&emsp;&emsp;其他同理：

```c++
        if(glfwGetKey(window , GLFW_KEY_W) == GLFW_PRESS){
            View = View * glm::translate(glm::mat4(1.0f) , vec3(0.0f ,0.0f , 0.1f));
        }

        if(glfwGetKey(window , GLFW_KEY_S) == GLFW_PRESS){
            View = View * glm::translate(glm::mat4(1.0f) , vec3(0.0f ,0.0f , -0.1f));
        }

        if(glfwGetKey(window , GLFW_KEY_A) == GLFW_PRESS){
            View = View * glm::translate(glm::mat4(1.0f) , vec3(0.1f ,0.0f , 0.0f));
        }

        if(glfwGetKey(window , GLFW_KEY_D) == GLFW_PRESS){
            View = View * glm::translate(glm::mat4(1.0f) , vec3(-0.1f ,0.0f , 0.0f));
        }
```

&emsp;&emsp;前后左右的位移很简单，但是跳跃就比较麻烦了。跳跃是一个有着明确过程的动作，即跳上来和落下去这两个过程，不是简单将摄像机向上平移。所以当我们点击空格键（大多数FPS游戏中的跳跃键）之后应该进入一个跳跃的状态。而我们的跳跃动作显然不是一帧就能够解决的，所以我们需要将这个动作分散到多个帧去执行。

&emsp;&emsp;首先我们需要两个bool型变量，一个记录当前是否处于跳跃动作状态中，一个记录是否处于上升跳跃状态中（使用上升状态变量是因为我们需要看到明显的从上到下和从下到上的过程）。

&emsp;&emsp;当我们按下空格键时，jumpState置为1，这时候我们进入跳跃状态，判断jumpUp变量，如果为1则摄像机向上平移，当位移达到最高点时将jumpUp变量置0，此时摄像机向下平移。

```c++
bool jumpState = false , jumpUp = true;
double currentHeight = 0.0f;
do{
    ...
		if(glfwGetKey(window , GLFW_KEY_SPACE) == GLFW_PRESS){
            if(!jumpState){
                jumpState = true;
                currentHeight = 0.01f;
            }
        }

        if(jumpState){
            if(currentHeight <= 0.0f){
                jumpState = false;
                jumpUp = true;
            } else {
                if(currentHeight >= 2.0f){
                    jumpUp = false;
                }
                if(jumpUp){
                    currentHeight += 0.1f;
                    View = View * glm::translate(glm::mat4(1.0f) , vec3(0.0f ,-0.1f , 0.0f));
                } else {
                    currentHeight -= 0.1f;
                    View = View * glm::translate(glm::mat4(1.0f) , vec3(0.0f ,0.1f , 0.0f));
                }
            }
        }
    ...
}while(...)
    ...
```

&emsp;&emsp;以下是最终效果（点击图片进入 Youtube 播放界面）：[![FPS](https://ws1.sinaimg.cn/large/006tNc79ly1foyuuiaxbij31ke0w2dhj.jpg)](https://www.youtube.com/watch?v=_xVMvRJnq3E&feature=youtu.be)



