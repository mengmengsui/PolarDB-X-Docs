DROP DATABASE 
==================================

DROP DATABASE语句用于删除数据库。

语法 
-----------------------

```sql
drop_database_stmt:
DROP DATABASE [IF EXISTS] database_name;
```



参数说明 
-------------------------



|      参数       |        说明         |
|---------------|-------------------|
| IF EXISTS     | 用于防止当数据库不存在时发生错误。 |
| database_name | 指定待删除的数据库名。       |



示例 
-----------------------

删除数据库test。

```sql
mysql>drop database test;
Query OK, 0 rows affected (0.03 sec)
mysql>drop database if exists test;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```



