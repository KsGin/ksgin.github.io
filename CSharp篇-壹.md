---
title: CSharp篇-壹
date: 2017-05-01 15:35:40
tags: .net
---

# .NET Core Web Api 快速入手 —— C#篇 [ 壹 ]

### 本篇内容
*数据类型*<br>
*var关键字*<br>
*数组，循环，输入与输出*<br>
<br>

1. c#数据类型

    &emsp;&emsp;在 C# 中，变量分为以下几种类型： 值类型（Value types）,引用类型（Reference types）,指针类型（Pointer types）

    &emsp;&emsp;值类型变量可以直接分配给一个值。它们是从类 System.ValueType 中派生的。

    &emsp;&emsp;值类型直接包含数据。比如 int、char、float，它们分别存储数字、字母、浮点数。当您声明一个 int 类型时，系统分配内存来存储值。

    &emsp;&emsp;引用类型及不包含存储在变量中的实际数据，但它们包含对变量的引用。它们指的是一个内存位置。使用多个变量时，引用类型可以指向一个内存位置。如果内存位置的数据是由一个变量改变的，其他变量会自动反映这种值的变化。

    &emsp;&emsp;指针类型变量存储另一种类型的内存地址。C# 中的指针与 C 或 C++ 中的指针有相同的功能。[ 不做具体讲解 ]

    &emsp;&emsp;数据类型无需熟背，在长久的使用中自然会了解。目前只需要知道C#中存在这些类型就可以了

    <!--more-->

2. c#var关键字

    &emsp;&emsp;var关键字是伴随这.NET 3.5以后，伴随着匿名函数、LINQ而来,由编译器帮我们推断具体的类型。总体来说，当一个变量是局部变量(不包括类级别的变量)，并且在声明的时候初始化，是使用var关键字的前提。

    &emsp;&emsp;比如在声明并且初始化局部变量的时候可使用var

    ```CSharp    
    var a = 10;
    var b = "123";
    ```

    &emsp;&emsp;在foreach循环中使用

    ```CSharp
    for(var it : mList)
    {
        //do somethong
    }
    ```
    &emsp;&emsp;var关键字可以大幅度减少代码冗余量，弱化数据类型的概念。配合foreach语句 ， linq语句 ， lambda表达式使用有很好的效果。具体用或者不用看个人喜好。总之我是很喜欢这个特性的。

3. c#循环，数组，输入，输出

    - 循环
        - while 循环

            ```CSharp
            while(condition)
            {
                statement(s);
            }
            ```

        - for 循环

            ```CSharp
            for ( init; condition; increment )
            {
                statement(s);
            }
            ```

        - do-while 

            ```CSharp
            do
            {
                statement(s);
            }
            while(condition)
            ```

        - foreach 循环
            &emsp;&emsp;c#中支持几乎所有的数组，容器进行foreach循环，并且我个人极力倡导如此使用，具体之后再讲

            ```CSharp      
            int[] fibarray = new int[] { 0, 1, 1, 2, 3, 5, 8, 13 };
            foreach (var element in fibarray)
            {
                System.Console.WriteLine(element);
            }
            System.Console.WriteLine(); 
            ```

    - 数组
             
        &emsp;&emsp;c#语言支持普通数组（如下），也支持容器，平时建议使用容器。

        ```CSharp  
        //定义并且初始化int一维数组
        int[] fibarray = new int[] { 0, 1, 1, 2, 3, 5, 8, 13 };
        ```

    - 输出输入

        - 控制台输出

            &emsp;&emsp;C# 控制台程序一般使用 .NET Framework Console 类提供的输入/输出服务。
            ```csharp
            Console.WriteLine("Hello World!");
            ```
            &emsp;&emsp;语句使用 WriteLine 方法。它在命令行窗口中显示其字符串参数并换行。其他 Console 方法用于不同的输入和输出操作。Console 类是 System 命名空间的成员。如果程序开头没有包含using System; 语句，则必须指定System 类，如下所示：
            ```CSharp
            System.Console.WriteLine("Hello World!");
            ```
            &emsp;&emsp;WriteLine 方法十分有用，在编写控制台应用程序时会经常用到它。<br>
            &emsp;&emsp;WriteLine 可显示字符串：
            ``` CSharp
            Console.WriteLine("Hello World!");
            ```
            &emsp;&emsp;WriteLine 也可显示数字：
            ``` CSharp
            int x = 42; 
            Console.WriteLine(x); 
            ```
            &emsp;&emsp;如果需要显示若干个项，则用 {0} 表示第一项，{1} 表示第二项，依此类推，如下所示：
            ```CSharp 
            int year = 2008; 
            string str = "今年是"; 
            Console.WriteLine(" {0} {1}年.", str, year);
            ```
            &emsp;&emsp;输出应如下:
            ```CSharp
            今年是2008年.
            ```
            &emsp;&emsp;Console.WriteLine()方法是将要输出的字符串与换行控制字符一起出,当次语句执行完毕时,光标会移到目前输出字符串的下一行.<br> 
            &emsp;&emsp;至于Console.Write()方法,光标会停在输出字符串的最后一个字符后,不会移动到下一行，其余的用法与Console.WriteLine()一样。
        - 控制台输入<br>
            &emsp;&emsp;在C#控制台程序中提供了两种方法让用户输入所需数据，它们是有Console类提供的静态方法。static int Read() 和 static string ReadLine()。<br>
            &emsp;&emsp;要读取单个字符，则使用Read()方法，它等待用户输入一个键，然后返回结果。字符作为int类型的值返回，所以要显示字符就必须转换为char类型。<br>
            &emsp;&emsp;要读取一串字符，则使用ReadLine()方法。该方法一直读取字符，直到用户按下ENTER键，然后将它们返回到string 类型的对象中。
            ```CSharp
            using System;
            //Console.Read() 示例
            class KbIn 
            {
                public static void Main()
                {   
                    char ch;
                    Console.Write("Press a key followed by ENTER: ");
                    // 读取一个字符
                    ch = (char) Console.Read(); 
                    Console.WriteLine("Your key is: " + ch);
                }

            }
            using System;
            //Console.ReadLine() 示例
            class ReadString 
            {
                public static void Main() 
                {
                    string str;
                    Console.WriteLine("Enter some characters.");
                    str = Console.ReadLine();
                    Console.WriteLine("You entered: " + str);
                }
            }
            ```

