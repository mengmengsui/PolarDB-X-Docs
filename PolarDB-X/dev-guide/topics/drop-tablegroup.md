DROP TABLEGROUP 
====================================

DROP TABLEGROUP语句用于删除表组，仅当表组为空时（也就是表组不包括任何的表）才允许删除表组。

语法 
-----------------------

```sql
drop_tablegroup_stmt:
DROP tablegroup [IF EXISTS] tablegroup_name
```



示例 
-----------------------

删除一个名为test_tg的表组。

```sql
drop tablegroup test_tg;
```



