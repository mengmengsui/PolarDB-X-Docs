REPLACE 
============================

您可以使用REPLACE语法往表中插入行或替换表中的行。

语法 
-----------------------

```sql
REPLACE [LOW_PRIORITY | DELAYED]
[INTO] [schema_name.]tbl_name
[(col_name [, col_name] ...)]
{VALUES | VALUE} (value_list) [, (value_list)]
REPLACE [LOW_PRIORITY | DELAYED]
[INTO] [schema_name.]tbl_name
SET assignment_list
REPLACE [LOW_PRIORITY | DELAYED]
[INTO] [schema_name.]tbl_name
[(col_name [, col_name] ...)]
SELECT ...
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

* PARTITION语法，例如：

  ```sql
  REPLACE INTO tb PARTITION (p0) (id) VALUES(7);
  ```

  

* 包含列名的语法，例如：

  ```sql
  REPLACE INTO tb(id1, id2) VALUES(1, id1 + 1);
  ```

  




相关文献 
-------------------------

MySQL [REPLACE](https://dev.mysql.com/doc/refman/5.7/en/replace.html) 语法。
