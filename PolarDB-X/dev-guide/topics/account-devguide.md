账号权限管理 
===========================

本文介绍了账号权限管理的相关操作。

PolarDB-X账号和权限系统的用法与MySQL 5.7一致，支持GRANT、REVOKE、SHOW GRANTS、CREATE USER、DROP USER、SET PASSWORD等语句，目前支持库级和表级权限的授予，全局级别和列级别权限暂时不支持。

创建账号 
-------------------------

语法：

```sql
CREATE USER [IF NOT EXISTS] user IDENTIFIED BY 'password';
```



其中，user通过用户名和主机名的组合`'username'@'host'`确定了一个账号，账号规则如下：

* username为创建的用户名，用户名遵循以下规则；
  * 大小写敏感；
  
  * 长度必须大于等于4个字符，小于等于20个字符；
  
  * 必须以字母开头；
  
  * 字符可以包括大写字母、小写字母、数字。
  

  

* host指定了创建用户可以在哪台主机上登录，用户名一样但是主机名不一样也代表不同的账号，需满足以下规则：
  * HOST必须是纯IP地址，可以包含_和%通配符（_代表一个字符，%代表0个或多个字符）。含有通配符的HOST需要加上单引号，例如lily@'0.9.%.%'，david@'%'；
  
  * 假设系统中有两个用户都符合当前准备登录的用户，则以最长前缀匹配（不包含通配符的最长IP段）的那个用户为准。例如系统有两个用户david@'30.9.12_.234'和david@'30.9.1%.234'，在主机30.9.127.234上面登录david，则使用的是david@'30.9.12_.234'这个用户；
  
  * 开启VPC时，主机的IP地址会发生变化。为避免账号和权限系统中的配置无效，请将HOST配置为'%'来匹配任意IP。
  

  

* password为用户密码，需满足以下规则：
  * 长度必须大于等于6个字符，小于等于20个字符；
  
  * 字符可以包括大写字母、小写字母、数字、特殊字符（@#$%\^\&+=）。
  

  




示例：

```sql
mysql> CREATE USER 'user1'@'127.0.0.1' IDENTIFIED BY '123456';
mysql> CREATE USER IF NOT EXISTS 'user2'@'%' identified by '123456';
```



修改账号密码 
---------------------------

语法：

```sql
SET PASSWORD FOR user = PASSWORD('auth_string')
```



示例：

```sql
mysql> SET PASSWORD FOR 'user1'@'127.0.0.1' = PASSWORD('654321');
```



删除账号 
-------------------------

语法：

```sql
DROP USER user;
```



示例：

```sql
mysql> DROP USER 'user2'@'%';
```



授予账号权限 
---------------------------

语法：

```sql
GRANT privileges ON database.table TO user;
```



其中，privileges为具体权限类型，数据库权限级别从高到低依次是：全局级别权限（暂不支持）、数据库级别权限、表级别权限、列级别权限。PolarDB-X目前支持和表相关联的8个基本权限项：CREATE、DROP、ALTER、INDEX、INSERT、DELETE、UPDATE、SELECT。

* TRUNCATE操作需要有表上的DROP权限；

* REPLACE操作需要有表上的INSERT和DELETE权限；

* CREATE INDEX 和 DROP INDEX操作需要有表上的INDEX权限；

* CREATE SEQUENCE需要有数据库级的创建表（CREATE）权限；

* DROP SEQUENCE需要有数据库级的删除表（DROP）权限；

* ALTER SEQUENCE需要有数据库级的更改表（ALTER）权限；

* INSERT ON DUPLICATE UPDATE语句需要有表上的INSERT和UPDATE权限。




示例：

```sql
mysql> GRANT SELECT,UPDATE ON `db1`.* TO 'user1'@'127.0.0.1';
```



查看账号权限 
---------------------------

语法：

```sql
SHOW GRANTS [FOR user];
```



可以使用current_user()来获取当前用户。

示例：

```sql
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT SELECT, UPDATE ON db1.* TO 'user1'@'127.0.0.1' |
+------------------------------------------------------+
mysql> SHOW GRANTS FOR current_user();
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT SELECT, UPDATE ON db1.* TO 'user1'@'127.0.0.1' |
+------------------------------------------------------+
```



回收账号权限 
---------------------------

语法：

```sql
REVOKE privileges ON database.table TO user;
```



示例：

```sql
mysql> REVOKE UPDATE ON db1.* FROM 'user1'@'127.0.0.1';
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'               |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'    |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1' |
+----------------------------------------------+
```


