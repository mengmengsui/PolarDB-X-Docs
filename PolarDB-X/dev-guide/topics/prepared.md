Prepared语句 
===============================

本文介绍了Prepare协议的概念、用途及在Java中的开启方法。

Prepare协议介绍 
--------------------------------

Prepare协议分为两种模式：

* 二进制模式：该模式基于高效的客户端/服务器二进制协议，是程序开发中最常用的模式；

* 文本模式：该模式基于SQL语法来实现，包括PREPARE/EXECUTE/DEALLOCATE PREPARE三种语句。




PolarDB-X对这两种模式均提供支持。使用预处理语句和占位符来获取参数值具有以下优势：

* 每次执行时解析语句的开销都较小。通常情况下，数据库应用程序处理大量几乎相同的语句，只改变Prepare语句中的变量值，这样可以大幅度提升SQL执行效率。

* 防止SQL注入攻击。




二进制模式 
--------------------------

二进制Prepare协议支持使用JDBC及其他各种语言，MySQL支持范围可参见[Prepared Statements](https://dev.mysql.com/doc/internals/en/prepared-statements.html) 。PolarDB-X的支持情况如下：

* Prepare协议支持范围：
  * [COM_STMT_PREPARE](https://dev.mysql.com/doc/internals/en/com-stmt-prepare.html#packet-COM_STMT_PREPARE)
  
  * [COM_STMT_EXECUTE](https://dev.mysql.com/doc/internals/en/com-stmt-execute.html#packet-COM_STMT_EXECUTE)
  
  * [COM_STMT_CLOSE](https://dev.mysql.com/doc/internals/en/com-stmt-close.html#packet-COM_STMT_CLOSE)
  
  * [COM_STMT_RESET](https://dev.mysql.com/doc/internals/en/com-stmt-reset.html#packet-COM_STMT_RESET)
  

  

* Prepare协议SQL支持范围：支持所有DML语句，例如：SELECT、UPDATE、DELETE、INSERT等。

* Prepare协议SQL不支持范围：不支持DML以外其他SQL语句，例如：SHOW、SET等。




在Java中开启Prepare协议

* 在JAVA客户端中，如果需要使用Prepare协议，需要显式在URL连接串中增加useServerPrepStmts=true参数，如果不指定此参数，则PreparedStatement默认会走普通查询。

* 如：jdbc:mysql://xxxxxx:3306/xxxxxx? **useServerPrepStmts=true** 




Java使用示例：

```sql
Class.forName("com.mysql.jdbc.Driver");
Connection connection =  DriverManager.getConnection("jdbc:mysql://xxxxxx:3306/xxxxxx?useServerPrepStmts=true", "xxxxx", "xxxxx");
String sql = "insert into batch values(?,?)";
PreparedStatement preparedStatement = connection.prepareStatement(sql); 
preparedStatement.setInt(1, 0);
preparedStatement.setString(2, "polardb-x");
preparedStatement.executeUpdate();
```



文本模式 
-------------------------

首先通过PREPARE语句给预处理语句preparable_stmt指定名称stmt_name，其中stmt_name不区分大小写，并且preparable_stmt只能为单语句。

```sql
PREPARE stmt_name FROM preparable_stmt;
```



接着通过EXECUTE语句执行指定的预处理语句，如果预处理语句包含参数占位符的话，必须用USING子句指定用户定义变量作为参数。

```sql
EXECUTE stmt_name [USING @var_name [, @var_name] ...];
```



最后通过DEALLOCATE PREPARE语句来释放清理预处理语句。

```sql
DEALLOCATE PREPARE stmt_name;
```

示例如下：

```sql
mysql> PREPARE stmt2 FROM 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
mysql> SET @a = 6;
mysql> SET @b = 8;
mysql> EXECUTE stmt2 USING @a, @b;
+------------+
| hypotenuse |
+------------+
|         10 |
+------------+
mysql> DEALLOCATE PREPARE stmt2;
```



