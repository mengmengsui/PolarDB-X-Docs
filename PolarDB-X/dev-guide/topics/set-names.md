SET NAMES 
==============================

您可以使用SET NAMES语句设置字符集。该语句会将`character_set_client`，`character_set_connection`和`character_set_results`设置为给定的字符集。

语法 
-----------------------

```sql
SET NAMES {'charset_name'
[COLLATE 'collation_name'] | DEFAULT}
```

其中，`charset_name`和`collation_name`可不加引号，当用户给出`collation_name`时，字符序也将被设置为给定值。

示例 
-----------------------

```sql
mysql> SHOW SESSION VARIABLES LIKE 'character\_set\_%';
+--------------------------+---------+
| Variable_name            | Value   |
+--------------------------+---------+
| character_set_client     | utf8mb4 |
| character_set_connection | utf8mb4 |
| character_set_database   | utf8    |
| character_set_filesystem | binary  |
| character_set_results    | utf8mb4 |
| character_set_server     | utf8    |
| character_set_system     | utf8    |
+--------------------------+---------+
7 rows in set (0.03 sec)
mysql> SET NAMES gb18030 COLLATE gb18030_chinese_ci;
Query OK, 0 rows affected (0.02 sec)
mysql> SHOW SESSION VARIABLES LIKE 'character\_set\_%';
+--------------------------+---------+
| Variable_name            | Value   |
+--------------------------+---------+
| character_set_client     | gb18030 |
| character_set_connection | gb18030 |
| character_set_database   | utf8    |
| character_set_filesystem | binary  |
| character_set_results    | gb18030 |
| character_set_server     | utf8    |
| character_set_system     | utf8    |
+--------------------------+---------+
7 rows in set (0.02 sec)
```


