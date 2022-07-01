INSERT 
===========================

您可以使用INSERT语句往表中插入数据。

语法 
-----------------------

```sql
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
[INTO] [schema_name.]tbl_name
[(col_name [, col_name] ...)]
{VALUES | VALUE} (value_list) [, (value_list)]
[ON DUPLICATE KEY UPDATE assignment_list]
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
[INTO] [schema_name.]tbl_name
SET assignment_list
[ON DUPLICATE KEY UPDATE assignment_list]
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
[INTO] [schema_name.]tbl_name
[(col_name [, col_name] ...)]
SELECT ...
[ON DUPLICATE KEY UPDATE assignment_list]
value_list:
value [, value] ...
value:
{expr | DEFAULT}
assignment_list:
assignment [, assignment] ...
assignment:
col_name = value
```



语法限制 
-------------------------

不支持使用以下语法。

* INSERT IGNORE ON DUPLICATE KEY UPDATE语法，例如：

  ```sql
  INSERT IGNORE INTO tb (id) VALUES(7) ON DUPLICATE KEY UPDATE id = id + 1;
  ```

  

* PARTITION语法，例如：

  ```sql
  INSERT INTO tb PARTITION (p0) (id) VALUES(7);
  ```

  

* 嵌套NEXTVAL的语法，例如：

  ```sql
  INSERT INTO tb(id) VALUES(SEQ1.NEXTVAL + 1);
  ```

  

* 包含列名的语法，例如：

  ```sql
  INSERT INTO tb(id1, id2) VALUES(1, id1 + 1);
  ```

  




相关文献 
-------------------------

MySQL [INSERT](https://dev.mysql.com/doc/refman/5.7/en/insert.html) 语法。
