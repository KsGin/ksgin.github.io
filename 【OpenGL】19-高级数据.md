---
title: 【OpenGL】19-高级数据
date: 2018-03-10 18:07:00
tags:
	- OpenGL
	- Computer graphics
---

#### 参考教程

【中文版】[高级数据](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/07%20Advanced%20Data/)

【英文版】[Advanced Data](http://learnopengl.com/#!Advanced-OpenGL/Advanced-Data)

#### 学习记录

&emsp;&emsp;这篇文章中我们将介绍几个特殊的方法，`glBufferSubData` 和 `glCopyBufferSubData` 。看到这两个方法，我想你能够想到 `glBufferData` 吧。glBufferData 是我们在之前学习中填充数据的方法，我们使用它将顶点，法线等数据统一填充到 BO(Buffer Object) 里。而这篇文章中主要介绍了部分填充方法 glBufferSubData ，它允许我们分别填充顶点或者纹理坐标或者法线等等。

<!--more-->

&emsp;&emsp;首先看一下 [glBufferSubData](http://docs.gl/gl4/glBufferSubData) 的官方介绍：

``` c++
void glBufferSubData(	
    GLenum target,
 	GLintptr offset,
 	GLsizeiptr size,
 	const GLvoid * data
 	);
```

*target*

Specifies the target to which the buffer object is bound for `glBufferSubData`, which must be one of the buffer binding targets in the following table:

| **Buffer Binding Target**      | **Purpose**                        |
| ------------------------------ | ---------------------------------- |
| `GL_ARRAY_BUFFER`              | Vertex attributes                  |
| `GL_ATOMIC_COUNTER_BUFFER`     | Atomic counter storage             |
| `GL_COPY_READ_BUFFER`          | Buffer copy source                 |
| `GL_COPY_WRITE_BUFFER`         | Buffer copy destination            |
| `GL_DISPATCH_INDIRECT_BUFFER`  | Indirect compute dispatch commands |
| `GL_DRAW_INDIRECT_BUFFER`      | Indirect command arguments         |
| `GL_ELEMENT_ARRAY_BUFFER`      | Vertex array indices               |
| `GL_PIXEL_PACK_BUFFER`         | Pixel read target                  |
| `GL_PIXEL_UNPACK_BUFFER`       | Texture data source                |
| `GL_QUERY_BUFFER`              | Query result buffer                |
| `GL_SHADER_STORAGE_BUFFER`     | Read-write storage for shaders     |
| `GL_TEXTURE_BUFFER`            | Texture data buffer                |
| `GL_TRANSFORM_FEEDBACK_BUFFER` | Transform feedback buffer          |
| `GL_UNIFORM_BUFFER`            | Uniform block storage              |

*offset*

Specifies the offset into the buffer object's data store where data replacement will begin, measured in bytes.

*size*

Specifies the size in bytes of the data store region being replaced.

*data*

Specifies a pointer to the new data that will be copied into the data store.

&emsp;&emsp;这个方法给定了偏移量参数，允许我们在 buffer 的固定位置写入数据，第一个参数和 glBufferData 相同，第二个数据是偏移量，也就是你要开始写入的地址，第三个参数则是你要写入的大小，第四个则是 data 。也就是说你写入的数据应位于 BO 的 [ offset , offset + size ] 位置内。

&emsp;&emsp;那么这样做有什么意义呢，想想我们之前的顶点坐标数组，我们所有的数据信息必须是交叉存在的，也就是说数组排列必须如下

```c++
float vertices[] = {
    pos1.x , pos1.y , pos1.z , nor1.x ,nor1.y , nor1.z ,
    pos2.x , pos2.y , pos2.z , nor2.x ,nor2.y , nor2.z ,
    ...
}
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

 &emsp;&emsp;如上，我们的信息以每一个顶点为一个单元，其中顶点坐标，法线坐标，贴图UV，等等交叉混合在一个数组里。我们设置步长，每步采用个数等进行填充。而有了部分填充后，我们可以简单的将法线，UV坐标等分开放在多个数组里。

```c++
float vertices[] = {
    pos1.x , pos1.y , pos1.z , 
    pos2.x , pos2.y , pos2.z , 
    ...
}
glBufferSubData(GL_ARRAY_BUFFER, 0 , sizeof(vertices), vertices);
float normals[] = {
    nor1.x ,nor1.y , nor1.z ,
	nor2.x ,nor2.y , nor2.z ,
    ...
}
glBufferSubData(GL_ARRAY_BUFFER, 0 , sizeof(normals), normals);
```

&emsp;&emsp;当然，如果你真的像上边这样简单的写了之后，那么你的程序基本上会崩掉。那是因为 glBufferSubData 方法和 glBufferData 最根本的不同在于 glBufferSubData 是不分配内存的。所以你需要在调用 glBufferSubData 之前调用 glBufferData 并将其 data 设置为 null （随便你设置什么，反正你后边使用 glBufferSubData 会将本来里边的数据冲走）。

&emsp;&emsp;glBufferSubData 这个方法提供给了我们部分更新缓冲的能力，也使得我们实现某个功能时可以有更完美的选择。

>  &emsp;&emsp;OpenGL中的缓冲只是一个管理特定内存块的对象，没有其它更多的功能了。在我们将它绑定到一个缓冲目标(Buffer Target)时，我们才赋予了其意义。当我们绑定一个缓冲到GL_ARRAY_BUFFER时，它就是一个顶点数组缓冲，但我们也可以很容易地将其绑定到GL_ELEMENT_ARRAY_BUFFER。OpenGL内部会为每个目标储存一个缓冲，并且会根据目标的不同，以不同的方式处理缓冲。
>
> &emsp;&emsp;将数据导入缓冲的另外一种方法是，请求缓冲内存的指针，直接将数据复制到缓冲当中。通过调用glMapBuffer函数，OpenGL会返回当前绑定缓冲的内存指针。之后可以使用例如 memset 等 c 中常用方法进行填充，拷贝等。

&emsp;&emsp;除了这个之外，这篇中还需要介绍的一个方法是 [glCopyBufferSubData](http://docs.gl/gl4/glCopyBufferSubData) ：

```c++
void glCopyBufferSubData(	
    GLenum readTarget,
    GLenum writeTarget,
    GLintptr readOffset,
    GLintptr writeOffset,
    GLsizeiptr size
    );
```

- *readTarget*

  Specifies the target to which the source buffer object is bound for `glCopyBufferSubData`

- *writeTarget*

  Specifies the target to which the destination buffer object is bound for `glCopyBufferSubData`.

- *readOffset*

  Specifies the offset, in basic machine units, within the data store of the source buffer object at which data will be read.

- *writeOffset*

  Specifies the offset, in basic machine units, within the data store of the destination buffer object at which data will be written.

- *size*

  Specifies the size, in basic machine units, of the data to be copied from the source buffer object to the destination buffer object.

## Description

&emsp;&emsp;`glCopyBufferSubData` and `glCopyNamedBufferSubData` copy part of the data store attached to a source buffer object to the data store attached to a destination buffer object. The number of basic machine units indicated by *size* is copied from the source at offset *readOffset* to the destination at *writeOffset*. *readOffset*, *writeOffset* and *size* are in terms of basic machine units.

&emsp;&emsp;For `glCopyBufferSubData`, *readTarget* and *writeTarget* specify the targets to which the source and destination buffer objects are bound, and must each be one of the buffer binding targets in the following table:

| **Buffer Binding Target**      | **Purpose**                        |
| ------------------------------ | ---------------------------------- |
| `GL_ARRAY_BUFFER`              | Vertex attributes                  |
| `GL_ATOMIC_COUNTER_BUFFER`     | Atomic counter storage             |
| `GL_COPY_READ_BUFFER`          | Buffer copy source                 |
| `GL_COPY_WRITE_BUFFER`         | Buffer copy destination            |
| `GL_DISPATCH_INDIRECT_BUFFER`  | Indirect compute dispatch commands |
| `GL_DRAW_INDIRECT_BUFFER`      | Indirect command arguments         |
| `GL_ELEMENT_ARRAY_BUFFER`      | Vertex array indices               |
| `GL_PIXEL_PACK_BUFFER`         | Pixel read target                  |
| `GL_PIXEL_UNPACK_BUFFER`       | Texture data source                |
| `GL_QUERY_BUFFER`              | Query result buffer                |
| `GL_SHADER_STORAGE_BUFFER`     | Read-write storage for shaders     |
| `GL_TEXTURE_BUFFER`            | Texture data buffer                |
| `GL_TRANSFORM_FEEDBACK_BUFFER` | Transform feedback buffer          |
| `GL_UNIFORM_BUFFER`            | Uniform block storage              |

&emsp;&emsp;Any of these targets may be used, but the targets `GL_COPY_READ_BUFFER` and `GL_COPY_WRITE_BUFFER` are provided specifically to allow copies between buffers without disturbing other GL state.

&emsp;&emsp;*readOffset*, *writeOffset* and *size* must all be greater than or equal to zero. Furthermore, $readOffset+size$ must not exceeed the size of the source buffer object, and $writeOffset+size$ must not exceeed the size of the buffer bound to *writeTarget*. If the source and destination are the same buffer object, then the source and destination ranges must not overlap.

&emsp;&emsp;这个方法提供给了我们将缓冲中的一部分复制给另一部分的功能，我们可以自由的复制缓冲中的一部分数据。`readtarget`和`writetarget`参数需要填入复制源和复制目标的缓冲目标。比如说，我们可以将VERTEX_ARRAY_BUFFER缓冲复制到VERTEX_ELEMENT_ARRAY_BUFFER缓冲，分别将这些缓冲目标设置为读和写的目标。当前绑定到这些缓冲目标的缓冲将会被影响到。

&emsp;&emsp;但如果我们想读写数据的两个不同缓冲都为顶点数组缓冲该怎么办呢？我们不能同时将两个缓冲绑定到同一个缓冲目标上。正是出于这个原因，OpenGL提供给我们另外两个缓冲目标，叫做GL_COPY_READ_BUFFER和GL_COPY_WRITE_BUFFER。我们接下来就可以将需要的缓冲绑定到这两个缓冲目标上，并将这两个目标作为`readtarget`和`writetarget`参数。

&emsp;&emsp;接下来glCopyBufferSubData会从`readtarget`中读取`size`大小的数据，并将其写入`writetarget`缓冲的`writeoffset`偏移量处。下面这个例子展示了如何复制两个顶点数组缓冲：

```c++
	float vertexData[] = { ... };
	glBindBuffer(GL_COPY_READ_BUFFER, vbo1);
	glBindBuffer(GL_COPY_WRITE_BUFFER, vbo2);
	glCopyBufferSubData(GL_COPY_READ_BUFFER, GL_COPY_WRITE_BUFFER, 0, 0, sizeof(vertexData));
```

&emsp;&emsp;这两个方法在一些场景是非常有用的。