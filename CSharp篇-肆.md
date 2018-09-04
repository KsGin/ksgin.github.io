---
title: CSharp篇-肆
date: 2017-05-05 18:12:40
tags: .net
---

# .NET Core Web Api 快速入手 —— C#篇 [ 肆 ]

### 本篇内容
*using关键字*<br>
*Attribute*<br>
<br>

1. using关键字

    using一般有着以下几种用法：
    + ***using System***。<br>&emsp;&emsp;这个是最常用的，就是using+命名空间，这样就可以直接使用命名空间中的类型，而免去了使用详细的命名空间

    + ***using别名***。<br>&emsp;&emsp;using + 别名 = 包括详细命名空间信息的具体的类型。例如我们用以下语句引入*System.IO.Compression*命名空间：
      *using Zip=System.IO.Compression;*这时我们就可以用Zip表示*System.IO.Compression*命名空间，使用Zip.GZipStream就是使用*System.IO.Compression.GZipStream*。给程序书写带来方便。<!--more-->

    + ***using语句，定义一个范围，在范围结束时处理对象。***<br>&emsp;&emsp;当在某个代码段中使用了类的实例，而希望无论因为什么原因，只要离开了这个代码段就自动调用这个类实例的Dispose。要达到这样的目的，用try...catch来捕捉异常也是可以的，但用using也很方便。<br>&emsp;&emsp;*在我们的项目代码中，大量使用了using语句来处理与数据库有关的操作，这样好处是能及时关闭数据库连接对象。*<br>&emsp;&emsp;例如:

        ```csharp
        using (var con = new SqlConnection(Server.SqlConString))
        	{
        		con.Open();

        		var sqlCom = new SqlCommand("sp_login", con)
        		{
        			CommandType = CommandType.StoredProcedure
        		};

        		sqlCom.Parameters.Add(new SqlParameter
        		{
        			ParameterName = "@account",
        			Direction = ParameterDirection.Input,
        			SqlDbType = SqlDbType.NVarChar,
        			Size = 15,
        			Value = account
        		});
        		sqlCom.Parameters.Add(new SqlParameter
        		{
        			ParameterName = "@password",
        			Direction = ParameterDirection.Input,
        			SqlDbType = SqlDbType.NVarChar,
        			Size = 15,
        			Value = password
        		});
        		sqlCom.Parameters.Add(new SqlParameter
        		{
        			ParameterName = "@return",
        			Direction = ParameterDirection.ReturnValue,
        			SqlDbType = SqlDbType.Int
        		});

        		sqlCom.ExecuteNonQuery();

        		return sqlCom.Parameters["@return"].Value;
        	 }
        ```
        &emsp;&emsp;以上代码中，便是使用了using(/\* new object \*/){/\*do something\*/}来包裹代码。

2. Attribute 特性
    >&emsp;&emsp;特性（Attribute）是用于在运行时传递程序中各种元素（比如类、方法、结构、枚举、组件等）的行为信息的声明性标签。您可以通过使用特性向程序添加声明性信息。一个声明性标签是通过放置在它所应用的元素前面的方括号（[ ]）来描述的。
    >&emsp;&emsp;特性（Attribute）用于添加元数据，如编译器指令和注释、描述、方法、类等其他信息。.Net 框架提供了两种类型的特性：预定义特性和自定义特性。

    **预定义特性（Attribute）**

    .Net 框架提供了三种预定义特性：
    * AttributeUsage:<br>
      &emsp;&emsp;预定义特性 AttributeUsage 描述了如何使用一个自定义特性类。它规定了特性可应用到的项目的类型。
      规定该特性的语法如下：
    * Conditional:<br>
      &emsp;&emsp;这个预定义特性标记了一个条件方法，其执行依赖于它顶的预处理标识符。
    * Obsolete:<br>
      &emsp;&emsp;这个预定义特性标记了不应被使用的程序实体。

    [MSDN具体介绍](https://msdn.microsoft.com/en-us/library/aa288454.aspx)

    **创建自定义特性（Attribute）[ 不使用 ]**

    .Net 框架允许创建自定义特性，用于存储声明性的信息，且可在运行时被检索。该信息根据设计标准和应用程序需要，可与任何目标元素相关。

    &emsp;&emsp;创建并使用自定义特性包含四个步骤：

    * 声明自定义特性
    * 构建自定义特性
    * 在目标程序元素上应用自定义特性
    * 通过反射访问特性

    最后一个步骤包含编写一个简单的程序来读取元数据以便查找各种符号。元数据是用于描述其他数据的数据和信息。该程序应使用反射来在运行时访问特性。

    [MSDN具体介绍](https://msdn.microsoft.com/en-us/library/aa288454.aspx#vcwlkattributestutorialanchor1)

