---
title: 【DirectX】16-渲染字体
date: 2018-03-31 11:52:02
tags:	
	- DirectX
	- Computer graphics
---

#### 参考教程

[Tutorial 12: Font Engine](http://www.rastertek.com/dx11tut12.html)

#### 学习记录

&emsp;&emsp;这篇文章中，我们将去实现文字的渲染。在渲染文字这方面，一直有很多解决方案，也都是有利有弊吧。这里我们介绍的是使用文字贴图来实现。

&emsp;&emsp;使用文字贴图渲染，也就是说将文字的显示放在一张纹理图片上，我们根据需要渲染的字来选择对应的纹理来渲染。这种方法虽然方便，但是也有很多局限，比如中文数千个字，要保存起来就比较麻烦了。虽然局限性比较大，但是对于简单的文本渲染来说还是可以使用的。

<!--more-->

&emsp;&emsp;首先，我们需要一张文字纹理贴图和一个纹理索引文件，纹理贴图和纹理索引文件一会可以到源代码网址里下载。纹理索引文件格式如下：

```C++
95
33 ! 0.0        0.000976563 1
34 " 0.00195313 0.00488281  3
35 # 0.00585938 0.0136719   8
36 $ 0.0146484  0.0195313   5
37 % 0.0205078  0.0302734   10
38 & 0.03125    0.0390625   8
39 ' 0.0400391  0.0410156   1
40 ( 0.0419922  0.0449219   3
41 ) 0.0458984  0.0488281   3
42 * 0.0498047  0.0546875   5
43 + 0.0556641  0.0625      7
44 , 0.0634766  0.0644531   1
45 - 0.0654297  0.0683594   3
46 . 0.0693359  0.0703125   1
47 / 0.0712891  0.0751953   4
48 0 0.0761719  0.0820313   6
49 1 0.0830078  0.0859375   3
50 2 0.0869141  0.0927734   6
51 3 0.09375    0.0996094   6
52 4 0.100586   0.106445    6
53 5 0.107422   0.113281    6
...
```

&emsp;&emsp;这只是其中的一部分数据，第一行的数字是文件里字符的个数，剩下每一行代表一个字符，每一行的第一个数字是 ascii 值，第二个是显示字符，第三个是纹理的 u 坐标，第四个是纹理的 v 坐标，最后一个是这个字的大小。

&emsp;&emsp;现在，我们来创建一个字体的结构体：

```c++
struct Font {
	float left, right;
	int size;
};
```

&emsp;&emsp;这个结构体包含了每个字的纹理坐标的左右点和大小，事实上我们还可以自定义它的大小来让我们显示的时候能够显示更多样式的字体。

&emsp;&emsp;有了结构体，我们试着将文件里的数字读入一个 `Font` 数组：

```c++
Font *fonts = nullptr;
LoadFontData("./fontdata.txt", &fonts);
```

&emsp;&emsp;这里我们创建了一个方法，来看看这个方法的具体实现：

```c++
bool LoadFontData(const char* path , Font **fontData) {

	ifstream ifs(path);
	if (!ifs.is_open()) return false;
	int fontCount = 0;
	ifs >> fontCount;

	*fontData = new Font[fontCount];
	if (!(*fontData)) return false;

	for (int i = 0 ; i < fontCount ; ++i) {
		int ascii;
		char letter;
		ifs >> ascii >> letter >> (*fontData)[i].left >> (*fontData)[i].right >> (*fontData)[i].size;
	}

	ifs.close();
	return true;
}
```

&emsp;&emsp;这里读入文件里的第一个字符，得到字符的个数，然后为传进来的指针创建数组，并且赋值。

&emsp;&emsp;得到了字符数组之后，我们就可以使用这个数组来实现我们的字符显示了。在我们使用文字贴图这种方法渲染文字时，我们会为需要渲染的每一个文字创建一个小矩形，即两个三角形。并为每个三角形赋值他的坐标和 UV 坐标。

&emsp;&emsp;来看看我们的实现：

```c++
Vertex *vertices = nullptr;
char sentence[] = "hello";
const int n = strlen(sentence);
float drawX = 0.0f , drawY = 0.0f;
GetFontsVertex(fonts, sentence ,n, drawX, drawY, &vertices);
```

&emsp;&emsp;我们创建了 `Vertex` 指针传入到 `GetFontsVertex` 方法中，同时传入的还有我们想要渲染的句子和句子长度以及我们想要渲染文字的起点坐标。这个方法实现如下：

```c++
void GetFontsVertex(const Font *fontData , const char* sentence , const int n , const float x , const float y , Vertex **vertices) {
	(*vertices) = new Vertex[n * 6];
	int idx = 0;
	float posX = x;
	const float posY = y;
	for (auto i = 0 ; i < n; ++i) {
		const int let = static_cast<int>(sentence[i]) - 32;
		const Font letter = fontData[let];
		if (sentence[i] - 32 == 0) {
			posX += 3;
		} else {
			(*vertices)[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			(*vertices)[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			(*vertices)[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			(*vertices)[idx++].tex = XMFLOAT2(letter.right, 1.0f); 
			(*vertices)[idx].pos = XMFLOAT3(posX, posY - 16, 0.0f);	// 左下
			(*vertices)[idx++].tex = XMFLOAT2(letter.left, 1.0f); 
			(*vertices)[idx].pos = XMFLOAT3(posX, posY, 0.0f);	// 左上
			(*vertices)[idx++].tex = XMFLOAT2(letter.left, 0.0f);
			(*vertices)[idx].pos = XMFLOAT3(posX + letter.size, posY, 0.0f);	// 右上
			(*vertices)[idx++].tex = XMFLOAT2(letter.right, 0.0f);
			(*vertices)[idx].pos = XMFLOAT3(posX + letter.size, posY - 16, 0.0f);	// 右下
			(*vertices)[idx++].tex = XMFLOAT2(letter.right, 1.0f); 
			posX += letter.size + 1.0f;
		}
	}
}
```

&emsp;&emsp;在这个方法里，我们通过遍历句子得到要渲染的每一个字符，然后为其创建一个矩形（六个顶点），最后加到我们传进来的顶点数组里。

&emsp;&emsp;有了顶点数组，我们只要将纹理改成文字贴图就可以实现正常的渲染了。

![1](https://image.ibb.co/nqcMAS/image.png)

&emsp;&emsp;源代码地址在这里：[DX11Tutorial Font](https://github.com/KsGin/DX11Tutorial/tree/master/DX11Tutorial-Font)