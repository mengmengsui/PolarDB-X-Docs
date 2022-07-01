SHOW TABLEGROUP 
====================================

用于查看当前数据库实例的表组信息，包括每个表组的分区情况，每个表组里都包括哪些表等信息。

语法 
-----------------------

```sql
show_tablegroup_stmt:
SHOW tablegroup [WHERE where_condition]
```



示例 
-----------------------

查找库名为`test_db`下的所有`tablegroup`信息。

```sql
show tablegroup where schema_name='test_db';
```


