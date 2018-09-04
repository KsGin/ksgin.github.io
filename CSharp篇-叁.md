---
title: CSharp篇-叁
date: 2017-05-03 14:12:40
tags: .net
---

# .NET Core Web Api 快速入手 —— C#篇 [ 叁 ]

### 本篇内容
*静态方法与静态字段*<br>
*命名空间*<br>
<br>

1. 静态方法与静态字段

   &emsp;&emsp;*在[GitHub](https://github.com/KsGin/TTMSWebAPI)上我的TTMS WebApi项目中， Servers文件夹中是具体的数据库操作。所有的方法都使用static静态方法。在Server.cs文件中使用了static存储全局的数据库连接字符串。*
   - 静态方法
        >&emsp;&emsp;静态方法可以被重载但不能被重写，因为它们属于类，不属于类的任何实例。

        ```csharp
        /// <summary>
        /// 登陆
        /// </summary>
        /// <returns>登陆结果</returns>
        /// <param name="account">账号</param>
        /// <param name="password">密码</param>
        public static object Login(string account, string password)
        {
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
        }

        ```
        <!--more-->

        &emsp;&emsp;这便是一个具体的静态方法操作数据库。我们暂时无需关注方法内部的代码，仅仅将方法声明提出来，如下

        ```csharp
        public static object Login(string account , string password)
        {

        }
        ```
        &emsp;&emsp;*public*是访问修饰符，*object*是返回类型，而其中的*static*便是声明这是一个静态方法。这个方法不需要其类创建实例便可以使用类名+方法名的方法来调用。比如:

        ```csharp
        var result = User.Login(account , password);
        ```

    - 静态字段

        >&emsp;&emsp;静态字段有两个常见的用法：一是记录已实例化对象的个数，二是存储必须在所有实例之间共享的值。

        &emsp;&emsp;*在我们的项目中，使用静态字段来存储数据库连接字符串*

        &emsp;&emsp;静态字段的声明:

        ```csharp 

        ```
       public class Server
       {
           /// <summary>
           /// 连接字符串
           /// </summary>
           public static string SqlConString = "";
    	}
        ```
        &emsp;&emsp;在Server类中，创建了一个静态字段SqlConString，并将其初始化为"";
       
        &emsp;&emsp;静态字段的使用就直接是类名+字段名，如下:
       
        ```csharp
        //创建一个数据库连接实例，参数是数据库连接字符串
        var con = new SqlConnection(Server.SqlConString);
        ```
        &emsp;&emsp;在上边的代码中，使用了Server.SqlConString获取这个字段并且作为参数传递给了SqlConnection类的构造方法。
       
        &emsp;&emsp;当然，静态字段也可以作为左值。
        
        ```csharp
        //读取连接字符串
        //Servers是命名空间 [ 见下 ]
       Servers.Server.SqlConString = Configuration.GetConnectionString("DefaultConnection");
        ```
        &emsp;&emsp;如上，为Server.SqlConString赋值。

2. 命名空间 [ 仅仅了解 ]

    >&emsp;&emsp;命名空间的设计目的是提供一种让一组名称与其他名称分隔开的方式。在一个命名空间中声明的类的名称与另一个命名空间中声明的相同的类的名称不冲突。

    &emsp;&emsp;命名空间定义:

    ```csharp
    //TTMS
    namespace TTMS
    {
        //class
    }
    ```
    &emsp;&emsp;调用:
    ```csharp
    //命名空间直接前缀来调用
    Servers.Server.SqlConString = " "; //something
    Servers.Server.Log();
    ```
    ```csharp
    //使用using引入
    using Servers;
    Server.SqlConString = " "; //something
    Server.Log();
    ```


