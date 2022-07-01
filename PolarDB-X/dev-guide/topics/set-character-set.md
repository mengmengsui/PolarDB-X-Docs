SET CHARACTER SET 
======================================

您可以使用SET CHARACTER SET语句设置当前客户端与服务端通信使用的字符集。该语句会将`character_set_client`和`character_set_results`设置为给定值，将`character_set_connection`设置为`character_set_database`的值。

语法 
-----------------------

```sql
SET {CHARACTER SET | CHARSET}
{'charset_name' | DEFAULT}
```

其中，`charset_name`可不加引号。

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
7 rows in set (0.01 sec)
mysql> SET CHARSET 'big5';
Query OK, 0 rows affected (0.06 sec)
mysql> SHOW SESSION VARIABLES LIKE 'character\_set\_%';
+--------------------------+--------+
| Variable_name            | Value  |
+--------------------------+--------+
| character_set_client     | big5   |
| character_set_connection | utf8   |
| character_set_database   | utf8   |
| character_set_filesystem | binary |
| character_set_results    | big5   |
| character_set_server     | utf8   |
| character_set_system     | utf8   |
+--------------------------+--------+
7 rows in set (0.21 sec)
```


