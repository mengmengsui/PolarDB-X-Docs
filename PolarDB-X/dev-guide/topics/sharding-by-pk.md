主键拆分 
=========================

PolarDB-X新增按主键类型自动拆分表功能，简易地将分布式技术引入到普通DDL语法，您只需要执行简单的修改，系统将根据主键和索引键自动选择拆分键和拆分方式，完成从单机数据库到分布式数据库的切换。

前提条件 
-------------------------

PolarDB-X内核小版本需为5.4.9或以上。

注意事项 
-------------------------

* 主键拆分表仅支持在建表时指定主键，不支持对已有的表添加或删除主键。

* 主键拆分表的非LOCAL索引必须指定索引名。

* 拆分规则变更后，主键拆分表将变成普通表（即不再适用原主键拆分表中的自动拆分规则或索引转换规则）。如何变更拆分规则，请参见[变更表类型及拆分规则](alter-sharding-rule.md)。




语法 
-----------------------

在CREATE TABLE语法中新增了PARTITION关键字，同时，在创建索引的子句中新增了LOCAL、GLOBAL和CLUSTERED关键字，以适应主键拆分表。

```sql
CREATE PARTITION TABLE [IF NOT EXISTS] tbl_name
(create_definition, ...)
[table_options]
[drds_partition_options]
create_definition:
col_name column_definition
| mysql_create_definition
| [UNIQUE] [LOCAL | GLOBAL | CLUSTERED] INDEX index_name [index_type] (index_col_name,...)
[global_secondary_index_option]
[index_option] ...
```



* LOCAL：强制指定为本地索引。

* GLOBAL：[全局二级索引](gsi-faq.md)。

* CLUSTERED：[聚簇索引](clustered-index.md)。



**说明** 关于CREATE TABLE的详细语法介绍，请参见[CREATE TABLE](create-table2.md)。

如果不想修改DDL，想使用普通单库单表的DDL直接创建主键拆分表，您可以通过在SQL命令行中设置用户变量的方式开启主键拆分，方法如下：

1. 执行set @auto_partition=1;命令，开启自动主键拆分。

2. 执行CREATE TABLE语句创建表（无需附加PARTITION关键字），该动作将被视为创建主键拆分表。

3. 执行set @auto_partition=0;命令，关闭自动主键拆分。




自动拆分规则 
---------------------------

* 如果目标表没有指定主键，PolarDB-X会启用隐式主键并将其作为拆分键，该主键为BIGINT类型的自增主键，且对用户不可见。

* 如果目标表指定了主键，PolarDB-X会使用该主键作为拆分键。如果为复合主键，则选择复合主键的第一列作为拆分键。

* 自动拆分仅拆分数据库，不拆分数据表，且拆分算法根据主键类型自动选择：

  |                                 主键类型                                  |  拆分算法  |
  |-----------------------------------------------------------------------|--------|
  | TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT、CHAR、VARCHAR                    | HASH   |
  | DATE、DATETIME、TIMESTAMP                                               | YYYYDD |
  | BIT、FLOAT、DOUBLE、TIME、YEAR、BLOB、ENUM、DECIMAL、BINARY、TEXT、SET、GEOMETRY | 不支持拆分  |

  




索引转换规则 
---------------------------

* 如果指定了LOCAL关键字，即强制指定索引为本地索引。

* 对主键拆分表执行创建索引操作时，如果未指定LOCAL关键字，该操作将被自动地转变为创建无覆盖列（covering）的全局二级索引，并且按索引列的第一列进行自动拆分。如果需要建立普通的局部索引，您需要指定LOCAL关键字。

* 创建全局二级索引和聚簇索引时，会创建一个带`_local_`前缀的本地索引。如果删除全局二级索引，PolarDB-X会自动同步删除对应的本地索引。

* 主键拆分表可以不指定全局二级索引、聚簇索引的拆分方式，PolarDB-X会根据自动拆分原则对索引键的第一列执行拆分。




下述语句及其注释为您展示了索引的转换规则。

```sql
CREATE PARTITION TABLE `t_order` (
  `x` int,
  `order_id` varchar(20) DEFAULT NULL,
  `seller_id` varchar(20) DEFAULT NULL,
  LOCAL INDEX `l_seller` using btree (`seller_id`), -- 强制指定为本地索引
  UNIQUE LOCAL INDEX `l_order` using btree (`order_id`), -- 强制指定为本地唯一索引
  INDEX `i_seller` using btree (`seller_id`), -- 会被替换为GSI，自动拆分
  UNIQUE INDEX `i_order` using btree (`order_id`), -- 会被替换为UGSI，自动拆分
  GLOBAL INDEX `g_seller` using btree (`seller_id`), -- 自动拆分
  UNIQUE GLOBAL INDEX `g_order` using btree (`order_id`), -- 自动拆分
  CLUSTERED INDEX `c_seller` using btree (`seller_id`), -- 自动拆分聚簇
  UNIQUE CLUSTERED INDEX `c_order` using btree (`order_id`) -- 自动拆分聚簇
);
```



执行show create table t_order;命令，查看表结构信息。

```sql
+---------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                                                                                            |
+---------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t_order | CREATE PARTITION TABLE `t_order` (
  `x` int(11) DEFAULT NULL,
  `order_id` varchar(20) DEFAULT NULL,
  `seller_id` varchar(20) DEFAULT NULL,
  UNIQUE LOCAL KEY `l_order` USING BTREE (`order_id`),
  UNIQUE LOCAL KEY `_local_i_order` USING BTREE (`order_id`),
  UNIQUE LOCAL KEY `_local_g_order` USING BTREE (`order_id`),
  UNIQUE LOCAL KEY `_local_c_order` USING BTREE (`order_id`),
  LOCAL KEY `l_seller` USING BTREE (`seller_id`),
  LOCAL KEY `_local_i_seller` USING BTREE (`seller_id`),
  LOCAL KEY `_local_g_seller` USING BTREE (`seller_id`),
  LOCAL KEY `_local_c_seller` USING BTREE (`seller_id`),
  UNIQUE CLUSTERED KEY `c_order` USING BTREE (`order_id`) DBPARTITION BY HASH(`order_id`),
  CLUSTERED INDEX `c_seller` USING BTREE(`seller_id`) DBPARTITION BY HASH(`seller_id`),
  UNIQUE GLOBAL KEY `g_order` USING BTREE (`order_id`) DBPARTITION BY HASH(`order_id`),
  GLOBAL INDEX `g_seller` USING BTREE(`seller_id`) DBPARTITION BY HASH(`seller_id`),
  UNIQUE GLOBAL KEY `i_order` USING BTREE (`order_id`) DBPARTITION BY HASH(`order_id`),
  GLOBAL INDEX `i_seller` USING BTREE(`seller_id`) DBPARTITION BY HASH(`seller_id`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4   |
+---------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.06 sec)
```



主键拆分表的DDL限制 
--------------------------------

如需对主键拆分表执行下述DDL操作，存在一些限制。

**说明** 关于下述DDL操作的详细语法介绍，请参见[CREATE INDEX](create-index2.md)和[ALTER TABLE](create-table2.md)。


|    DDL类别     |                                               DDL子句                                                |                                                                                                                                                                                                                                                     说明与限制                                                                                                                                                                                                                                                      |
|--------------|----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CREATE INDEX | 无                                                                                                  | 如果使用了`[UNIQUE] LOCAL INDEX`子句，PolarDB-X将创建本地索引，且会自动同步到聚簇索引表中。  * 如果使用了`[UNIQUE] INDEX`子句，PolarDB-X将其转换为对应的全局二级索引，同时，自动创建一个带`_local_`前缀的本地索引并同步到聚簇索引表中。  * 如果使用了[UNIQUE] GLOBAL &#124; CLUSTERED INDEX子句，PolarDB-X会自动创建一个带`_local_`前缀的本地索引并同步到聚簇索引表中。 **说明** 如果未指定拆分方式则会自动拆分。    |
| ALTER TABLE  | ADD {INDEX &#124; KEY} \[index_name\] \[index_type\] (key_part,...) \[index_option\] ...               | 如果使用了`[UNIQUE] LOCAL INDEX`子句，PolarDB-X将创建本地索引，且会自动同步到聚簇索引表中。  * 如果使用了`[UNIQUE] INDEX`子句，PolarDB-X将其转换为对应的全局二级索引，同时，自动创建一个带`_local_`前缀的本地索引并同步到聚簇索引表中。  * 如果使用了[UNIQUE] GLOBAL &#124; CLUSTERED INDEX子句，PolarDB-X会自动创建一个带`_local_`前缀的本地索引并同步到聚簇索引表中。 **说明** 如果未指定拆分方式则会自动拆分。    |
| ALTER TABLE  | ADD \[COLUMN\] (col_name column_definition,...)                                                    | * PolarDB-X将自动添加对应的列到聚簇全局二级索引（CGSI）和带唯一约束的聚簇全局二级索引（UCGSI）。  * 支持回滚操作。                                                                                                                                                                                                                                                                                                                                                     |
| ALTER TABLE  | DROP \[COLUMN\] col_name                                                                           | 不允许删除主键、主表拆分键、索引表拆分键和复合UNIQUE约束中的列。                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ALTER TABLE  | CHANGE \[COLUMN\] old_col_name new_col_name column_definition \[FIRST &#124; AFTER col_name\]          | * 不允许重命名主键、主表拆分键、索引表拆分键。  * 不允许重命名复合UNIQUE约束中的列。 **说明** 其它列的限制可通过执行Hint：/\*+TDDL: cmd_extra(ALLOW_LOOSE_ALTER_COLUMN_WITH_GSI=true)\*/命令解除。更多信息，请参见[Hint简介](hint-faq.md) 。   * 如果仅需修改default信息，推荐使用SET DEFAULT或DROP DEFAULT子句。                                                                                                                       |
| ALTER TABLE  | MODIFY \[COLUMN\] col_name column_definition \[FIRST &#124; AFTER col_name\]                           | * 不允许重命名主键、主表拆分键、索引表拆分键。  * 不允许重命名复合UNIQUE约束中的列。 **说明** 其它列的限制可通过执行Hint：/\*+TDDL: cmd_extra(ALLOW_LOOSE_ALTER_COLUMN_WITH_GSI=true)\*/命令解除。更多信息，请参见[Hint简介](hint-faq.md) 。   * 如果仅需修改default信息，推荐使用SET DEFAULT或DROP DEFAULT子句。                                                                                                                       |
| ALTER TABLE  | ALTER TABLE tbl_name ALTER \[COLUMN\] col_name { SET DEFAULT {literal &#124; (expr)} &#124; DROP DEFAULT } | 支持回滚操作。 **说明** 如果列的默认值为current_timestamp，则不支持回滚操作。                                                                                                                                                                                                                                                                                                                                                                                                                                                             |


