DROP VIEW 
==============================

本文将介绍如何使用DROP VIEW语句删除PolarDB-X的视图。

语法 
-----------------------

```sql
DROP VIEW [IF EXISTS] view_name
```



示例 
-----------------------

```sql
# 创建视图
create view v as select 1;
# 删除视图
drop view v;
```


