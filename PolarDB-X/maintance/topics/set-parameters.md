SET语句变量设置 
==============================

您可以通过SET语句设置各类变量，包括用户自定义的变量、session变量和global变量。

语法 
-----------------------

```sql
SET variable = expr [, variable = expr] ...
variable: {
    user_var_name
  | {GLOBAL | @@GLOBAL.} system_var_name
  | [SESSION | @@SESSION. | @@] system_var_name
}
```



注意事项 
-------------------------

使用SET GLOBAL设置全局变量时，PolarDB-X会将其进行持久化，实例重启后仍然生效。此外，SET GLOBAL设置成功后，所有已有的连接都会生效。

示例 
-----------------------

```sql
mysql> SET @foo='bar';
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT @foo;
+------+
| @foo |
+------+
| bar  |
+------+
1 row in set (0.01 sec)
mysql> SET @@time_zone='+09:00';
Query OK, 0 rows affected (0.01 sec)
mysql> SELECT @@time_zone;
+-------------+
| @@time_zone |
+-------------+
| +09:00      |
+-------------+
1 row in set (0.00 sec)
mysql> SET GLOBAL time_zone='+09:00';
Query OK, 0 rows affected (0.04 sec)
mysql> SELECT @@GLOBAL.time_zone;
+--------------------+
| @@global.time_zone |
+--------------------+
| +09:00             |
+--------------------+
1 row in set (0.02 sec)
```


