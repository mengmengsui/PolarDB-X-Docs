如何使用Blob对象 
===============================

本文介绍了如何使用Blob对象。

Java的MySQL各版本驱动（包括5.1.x和8.0.x），在实现PreparedStatement.setBlob方法时都有一些缺陷（无论连接的是mysql server还是PolarDB-X，都存在缺陷），对于一些用于存储二进制格式（例如图片、视频等）的Blob对象，会有概率性的报出语法错误，本文列出一些需要注意的地方。

场景一：PolarDB-X版本\<5.4.9 
----------------------------------------

对于低版本的PolarDB-X，其自身不支持SQL中直接携带与连接串字符集不符的二进制信息，需要在客户端转义成如下列所示的十六进制字符串。

```sql
insert into t1 values (0xaabbccdd,x'aabbccdd');
```



方法1：对于Java用户，可以使用PreparedStatement.setBytes方法替换setBlob方法，mysql驱动会将byte\[\]转义成十六进制字符串。

方法2：对于Java用户，如果不方便使用方法1（例如使用Hibernate等框架，无法控制框架使用哪个set方法），请联系阿里云技术支持，我们会提供一个定制的mysql驱动包，该驱动包内会完成转义。

方法3：对于其他语言用户，请在应用程序中自行完成转义。

场景二：5.4.9\<=PolarDB-X版本\<5.4.13 
-------------------------------------------------

该版本的PolarDB-X，额外支持了_binary前缀。因此，除了场景一中的方法继续适用之外，还有以下方法可以使用：

方法4：对于Java用户，可以修改SQL语句，例如原语句：

```sql
insert into t1 values (?)
```



修改为：

```sql
insert into t1 values (_binary?)
```



方法5：对于Java用户，可以使用8.0.26版本的mysql驱动，该驱动在setBlob方法内会自动加上_binary前缀。

场景三：PolarDB-X版本\>=5.4.13 
------------------------------------------

该版本比较彻底的兼容了mysql对于二进制信息的处理，上述场景一和场景二的方法继续适用，还有以下方法可以使用：

方法6：使用utf8/utf8mb4连接数据库，例如对于Java用户，jdbcurl中加入参数：

```sql
useUnicode=true&characterEncoding=utf8
```



对于其他语言用户，可以在建完连接后，执行：

```sql
set names utf8mb4;
```



实际上，mysql驱动对于setBlob的实现是有问题的，即使是使用官方mysql server，也必须在连接使用utf8/utf8mb4编码的情况下，才能很好的支持setBlob这种使用方法；对于gbk等编码，都有一定的概率报语法错误。
