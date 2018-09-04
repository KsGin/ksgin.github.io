---
title: 经典光照模型之漫反射与Lambert模型
date: 2018-03-24 17:39:05
mathjax: true
tags:
	- Shader
	- Computer graphics
---

#### 参考资料

1. 《GPU编程与CG语言之阳春白雪与下里巴人》
2. 《Introduction to 3D Game Programming with Directx11》

#### 漫反射

&emsp;&emsp;光的反射我们都是很清楚的，其反射如下：

![1](https://image.ibb.co/bMfKkS/440px_Reflection_angles.png)

&emsp;&emsp;通常情况下，我们计算某个面的颜色可以根据定向光线到这个面上的入射角来计算。

<!--more-->

&emsp;&emsp;但是粗糙的物体表面向各个方向等强度地反射光，这种等同地向各个方向散射的现象称为光的漫反射(diffuse reflection)。产生光的漫反射现象的物体表面称为理想漫反射体，也称为朗伯(Lambert)反射体。

&emsp;&emsp;下面这张图片描述了一个粗糙的表面对于入射光线的发射情况：
![2](https://image.ibb.co/b7XtVS/QQ20180324_214344_2x.png)

&emsp;&emsp;可以看到，在面对一个粗糙的物体时，因为各个小平面的法线不一样，所以我们必须对于各个小平面来计算颜色。

&emsp;&emsp;对于仅暴露在环境光下的朗伯反射体，可以用公式(9-1)表示某点处漫反射的光强:
$$
I_{ambdiff} = k_{d} I_{a}
$$


&emsp;&emsp;其中 $$I_a$$ 表示环境光强度(简称光强)，$$k_d (0< k_d <1)$$为材质对环境光的反射系数，$$I_{ambdiff}$$ 是漫反射体与环境光交互反射的光强。

&emsp;&emsp;即使一个理想的漫反射体在所有方向上具有等量的反射光线，但是表面光强还依赖于光线的入射方向(方向光)。例如，入射光方向垂直的表面与入射光方向成斜角的表面相比，其光强要大的多。这种现象可以用 Lambert 定律进行数学上的量化。

&emsp;&emsp;即，当方向光照射到朗伯反射体上时，漫反射光的光强与入射光的方向和入射点表面法向夹角的余弦成正比，这称之为 Lambert 定律，并由此构造出 Lambert 漫反射模型:
$$
I_{ldiff} = k_d I_l cosθ
$$
&emsp;&emsp;$$I_l$$ 是点光源强度，$θ$ 是入射光方向与顶点法线的夹角，称为入射 $$(0≤θ≤90°)$$，$$I_{ldiff}$$ 是漫反射体与方向光交互反射的光强。入射角为零时，说明光线垂直于物体表面，漫反射光强最大;90°时光线与物体表面平行，物体接收不到任何光线。

&emsp;&emsp;若 N 为顶点单位法向量，L 表示从顶点指向光源的单位向量(注意，是由顶点指向光源，不要弄反了)，则$$cosθ$$ 等价于N 与L的点积。所以公式 (2) 可以表示为公式 (3) :
$$
I_{ldiff} = k_d I_l (N • L)
$$
&emsp;&emsp;综合考虑环境光和方向来，Lambert 光照模型可写为:
$$
I_{diff} =I_{ambdiff} +I_{ldiff} =k_dI_a +k_dI_l(N•L)
$$
&emsp;&emsp;现在我们来使用 HLSL 代码来实现。

```c++
struct VertexIn {
	float4 position : POSITION;
	float4 normal : NORMAL; 
};
struct VertexScreen {
	float4 oPosition : POSITION;
	float4 color : COLOR; 
};
void main_v(
    VertexIn posIn,
	out VertexScreen posOut,
	uniform float4x4 modelViewProj, 
    uniform float4x4 worldMatrix, 
    uniform float4x4 worldMatrix_IT, 
    uniform float3 globalAmbient, 
    uniform float3 lightPosition, 
    uniform float3 lightColor, 
    uniform float3 Kd)
{
	posOut.oPosition = mul(modelViewProj, posIn.position);
	float3 worldPos = mul(worldMatrix, posIn.position).xyz; float3 N = 	mul(worldMatrix_IT, posIn.normal).xyz;
	N = normalize(N);
	//计算入射光方向
	float3 L = lightPosition - worldPos; L = normalize(L);
	//计算方向光漫反射光强
	float3 diffuseColor = Kd*lightColor*max(dot(N, L), 0);
	//计算环境光漫反射光强
	float3 ambientColor = Kd*globalAmbient;
	posOut.color.xyz = diffuseColor+ambientColor;
	posOut.color.w = 1; 
}
```

