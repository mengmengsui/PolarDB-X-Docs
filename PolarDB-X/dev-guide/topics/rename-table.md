RENAME TABLE 
=================================

本文介绍了对表做重命名的方法。

语法 
-----------------------

```sql
RENAME TABLE
tbl_name TO new_tbl_name
```



注意事项 
-------------------------

* PolarDB-X不支持同时RENAME多张表。

* 考虑到稳定性和性能，目前不支持重命名包含全局二级索引的表。

* 不支持单独重命名索引表。如需重命名索引，请参见[ALTER TABLE](alter-table.md)中RENAME INDEX推荐的方法。



