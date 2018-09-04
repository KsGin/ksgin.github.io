---
title: CXX中operator-new/delete与new/delete-operator
date: 2018-04-12 21:40:21
tags: C/C++
---

&emsp;&emsp;C++中，我们用 new 和 delete 来创建和销毁对象指针，这是一个内建**操作符**，就像是 sizeof 一样。它为我们申请了一块空间，并且使用我们所要申请的类对象的构造方法为这块空间初始化。

&emsp;&emsp;这篇文章，主要介绍 operator new/delete 与 new/delete operator 。

<!--more-->

&emsp;&emsp;首先给定结论：***new operator 是一个操作符，而 operator 是一个函数。我们无法去改变 new operator 所做的事，但是我们可以重写或者重载 operator new 函数。*** 

&emsp;&emsp;如文章开始所说，new operator 是一个用来申请空间创建对象的操作符，它申请空间则是使用的 operator new 。operator new  的声明通常如下：

```c++
void * operator new(size_t size);
```

&emsp;&emsp;它的返回值为 void * ，一个指向一块原始的、未设初值的内存。如果我们愿意，我们也可以直接使用它来申请空间：

```c++
void * emptyMemory = operator new(10);
```

&emsp;&emsp;这种使用相当于我们调用 malloc ，因为它的唯一任务就是分配内存。而我们调用 `new operator ` 的时候，它会先调用 `operator new ` 申请内存，然后将其转换为一个对象并返回。例如我们如下的调用：

```c++
string *p = new string("M");
```

&emsp;&emsp;它的实现类似于下边的内容：

```c++
void * m = operator new(sizeof(string));
call string::string("M") on *m;
string *p = static_cast<string*>(m);
```

&emsp;&emsp;它的过程就是先申请内存，然后调用对应的构造函数，最后转换为我们所需要的指针对象。

&emsp;&emsp;operator delete 和 delete operator 也是这样的，和他们所对应的 new 类似。delete operator 也是一个内建操作符，operator delete 则是函数。operator delete 的声明如下：

```c++
void operator delete(void* memory);
```

&emsp;&emsp;和 new 相反的是，在执行 delete p 的时候，他首先会调用对应析构函数然后才使用 operator delete ，如下：

```c++
p->~string();
operator delete(p);
```

