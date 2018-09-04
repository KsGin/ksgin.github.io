---
title: CSharp篇-伍
date: 2017-05-06 17:01:40
tags: .net
---

# .NET Core Web Api 快速入手 —— C#篇 [ 伍 ]

### 本篇内容
*C# 数据库使用*<br>
<br>

>&emsp;&emsp;C#使用数据库的方式大体上有两种，ado 和 ef 模型，在我们的项目中使用了传统的 ado ，所以对于 ef 模型暂且不提。

>&emsp;&emsp;ado.net提供了丰富的数据库操作，这些操作可以分为三个步骤：

>- 第一，使用SqlConnection对象连接数据库；
>- 第二，建立SqlCommand对象，负责SQL语句的执行和存储过程的调用；
>- 第三，对SQL或存储过程执行后返回的“结果”进行操作。

>&emsp;&emsp;对返回“结果”的操作可以分为两类：

>- 一是用SqlDataReader直接一行一行的读取数据集；
>- 二是DataSet联合SqlDataAdapter来操作数据库。

>&emsp;&emsp;两者比较：
>
><!--more-->

>&emsp;&emsp;SqlDataReader时刻与远程数据库服务器保持连接，将远程的数据通过“流”的形式单向传输给客户端，它是“只读”的。由于是直接访问数据库，所以效率较高，但使用起来不方便。

>&emsp;&emsp;DataSet一次性从数据源获取数据到本地，并在本地建立一个微型数据库（包含表、行、列、规则、表之间的关系等），期间可以断开与服务器的连接，使用SqlDataAdapter对象操作“本地微型数据库”，结束后通过SqlDataAdapter一次性更新到远程数据库服务器。这种方式使用起来更方，便简单。但性能较第一种稍微差一点。

>&emsp;&emsp;ado图
>![图](http://images.cnitblog.com/blog/33509/201305/18002906-bd0650c935e04d8caebbd029908ecbb6.jpg)


1. 连接字符串的写法

    ```csharp
    string connectString = "Data Source=.;Initial Catalog=Student;Integrated Security=True";
    ```
2. System.Data.SqlClient.SqlConnection;类

    &emsp;&emsp;其返回数据库连接对象，参数字符串。实例化“连接对象”，并打开连接

    ```csharp
    SqlConnection con = new SqlConnection(connectString);
    con.Open();
    ```
    &emsp;&emsp;在使用完成后要关闭对象。(*我们项目中使用using语句来包裹这个对象，在using语句块结束后会释放里边所有的对象，所以没有手动关闭，一般还是养成关闭的习惯比较好*)

3. SqlCommand对象

    >命名空间：System.Data.SqlClient.SqlCommand;  
    >SqlCommand对象用于执行数据库操作，操作方式有三种：

    > + SQL语句:command.CommandType = CommandType.Text;
    > + 存储过程:command.CommandType=CommandType.StoredProcedure;
    > + 整张表:command.CommandType = CommandType.TableDirect;

    > 实例化一个SqlCommand对象

    >```csharp
    >SqlCommand command = new SqlCommand();
    >command.Connection = con;            // 绑定SqlConnection对象或直接从SqlConnection创建
    >SqlCommand command = con.CreateCommand();
    >```
    ```

    >常用方法：

    >+ command.ExecuteNonQuery(): 返回受影响函数，如增、删、改操作；

    >+ command.ExecuteScalar()：执行查询，返回首行首列的结果；
       
    >+ command.ExecuteReader()：返回一个数据流（SqlDataReader对象）。

    >常用操作:

    >① 执行SQL

    >```csharp
    SqlCommand cmd = conn.CreateCommand();              //创建SqlCommand对象
    cmd.CommandType = CommandType.Text;
    cmd.CommandText = "select * from products = @ID";   //sql语句
    cmd.Parameters.Add("@ID", SqlDbType.Int);
    cmd.Parameters["@ID"].Value = 1;                    //给参数sql语句的参数赋值
    ```

    >② 调用存储过程

    >```csharp
    >SqlCommand cmd = conn.CreateCommand();                      
    >cmd.CommandType = System.Data.CommandType.StoredProcedure;
    >cmd.CommandText = "存储过程名";
    >```
    ```

    >③ 整张表

    >```csharp
    SqlCommand cmd = conn.CreateCommand();    
    cmd.CommandType = System.Data.CommandType.TableDirect;
    cmd.CommandText = "表名"
    ```

4. SqlDataReader对象

    >&emsp;&emsp;命名空间：System.Data.SqlClient.SqlDataReader;

    >&emsp;&emsp;SqlDataReader对象提供只读单向数据的功能，单向：只能依次读取下一条数据；只读：DataReader中的数据是只读的，不能修改；相对地DataSet中的数据可以任意读取和修改.

    >&emsp;&emsp;它有一个很重要的方法，是Read()，返回值是个布尔值，作用是前进到下一条数据，一条条的返回数据，当布尔值为真时执行，为假时跳出。如

    ```csharp
    SqlCommand command = new SqlCommand();
    command.Connection = con;
    command.CommandType = CommandType.Text;
    command.CommandText = "Select * from Users";
    SqlDataReader reader = command.ExecuteReader();		//执行SQL，返回一个“流”
    while (reader.Read())
    {
        Console.Write(reader["username"]);// 打印出每个用户的用户名
    }
    ```

5. DataSet对象[未使用]

6. 关闭资源

    &emsp;&emsp;资源使用完毕后应及时关闭连接和释放，具体方法如下：

    ```csharp
    myDataSet.Dispose();        // 释放DataSet对象
    myDataAdapter.Dispose();    // 释放SqlDataAdapter对象
    myDataReader.Dispose();     // 释放SqlDataReader对象
    con.Close();             // 关闭数据库连接
    con.Dispose();           // 释放数据库连接对象
    ```

