DROP INDEX 
===============================

本文介绍了如何删除局部索引和全局二级索引。

局部索引 
-------------------------

删除局部索引的方法和MySQL中一致，请参考[DROP INDEX](https://dev.mysql.com/doc/refman/5.7/en/drop-index.html) 文档。

全局二级索引 
---------------------------

语法

```sql
DROP INDEX index_name ON tbl_name      
```


**说明** index_name为需要删除的全局二级索引名。
