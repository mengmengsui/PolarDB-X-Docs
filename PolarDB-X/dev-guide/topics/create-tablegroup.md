CREATE TABLEGROUP 
======================================

CREATE TABLEGROUP语句用于在数据库中创建一个表组。

定义 
-----------------------

表组是表的属性，在PolarDB-X中允许用户创建表组，然后将多个表和表组关联起来，同一个表组里的表必须是采用相同的分区策略，并且同一个表组的所有分区表，在设计分区变更操作时，必须保持同步。例如要对表组的一个表的某个分区进行分裂，那么需要通过分裂整个表组某个分区来实现这个目的。

语法 
-----------------------

```sql
create_tablegroup_stmt:
CREATE tablegroup [IF NOT EXISTS] tablegroup_name
```



示例 
-----------------------

创建一个名为test_tg的表组。

```sql
create tablegroup test_tg;
```



创建好表组后可以通过`show tablegroup`命令查看表组的详细信息。例如，查看名为test_tg的表组信息。

```sql
show tablegroup where TABLE_GROUP_NAME='test_tg'
```



