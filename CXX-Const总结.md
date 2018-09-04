---
title: CXX-Const总结
date: 2018-03-15 22:20:58
tags: 
	- C/C++
---

&emsp;&emsp;在 C/C++ 中，const 是一个比较常见的关键字，其用法也特别灵活，很容易就搞混。这几天在复习基础知识正巧看了这个，加上今天时间有点晚没事干刚好做个总结。

<!--more-->

#### C/C++ const 作用

+ 定义 const 常量，和宏常量的区别是 const 常量有类型检查（#define 定义的宏常量只做简单的替换）
+ 修饰函数参数可以保护参数不被修改，一般用法是修改变量引用`fun(const int& i){...}` 
+ 将一些必要的常量值（比如普通一年365天）定义为 const 常量可以减少代码中出现的 Magic number ，增强代码可读性而且修改较为方便。
+ const 定义的常量相比较宏定义而言更加节省空间，在程序运行的过程中只有一次拷贝（宏定义的调用多次即有多个）。

#### C/C++ const 用法

+ 定义常量

&emsp;&emsp;`const int i = 10` 和 `int const i = 10 ` 两者作用相同，都是定义不可改变的常量值。 

+ 定义指针

  + 修饰指针，指针不可变其指向的值可变

    ```c++
    int* const cpInt;
    ```

  + 修饰值，指针可变而指向的值不可变

    ```c++
    const int* pcInt;
    ```

  + 两者皆不可变

    ```c++
    const int* const cpcInt;
    ```

+ 修饰函数

  + 修饰函数参数

     ```c++
     void f(const int i){...}	//修饰形参
     void f(const int& i){...}	//修饰引用(引用对象不可进行更改)
     void f(const int* i){...}	//修饰指针(指针值不可更改)
     void f(int const* i){...}	//修饰指针(指针指向不可更改)
     ```

  + 修饰函数返回值

     ```c++
     const int f(){...}	//修饰方法返回值
     const int* f(){...}	//相当于修饰变量 const int* f
     int* const f(){...}	//同上
     ```

+ 类内修饰

  + 类常量

     ```c++
     class A{
         const int aId = 10; 	//不可更改
         ...
     }
     ```

  + 常成员函数

     ```c++
     class A{
         void f() const {...}	//方法不可以改变类内成员信息也不可以调用非常函成员数
     }
     ```

  + 修饰类对象 / 对象引用 / 对象指针

     &emsp;&emsp;除了和变量一样无法更改对象内值这种以外，也不允许调用任何非 const 方法。

#### const 与非 const 的转换

&emsp;&emsp;C++ 提供了 const_cast&lt;Type> 来对 const 进行转换。

```c++
int i = 3;
const int& cref_i = i;
const_case<int&>(cref_i) = 4;
```

