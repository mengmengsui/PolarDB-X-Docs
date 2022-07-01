CREATE INDEX 
=================================

PolarDB-X支持创建局部索引和全局二级索引 (Global Secondary Index, GSI) ，同时支持删除这两种索引。

局部索引 
-------------------------

关于局部索引，详情清参见[CREATE INDEX Statement](https://dev.mysql.com/doc/refman/8.0/en/create-index.html?spm=a2c4g.11186623.2.5.c4f27cd2WfPxT7) 。

全局二级索引 
---------------------------

关于全局二级索引基本原理，请参见[全局二级索引](../../features/topics/gsi.md)。

**语法** 

```sql
CREATE [UNIQUE]
    GLOBAL INDEX index_name [index_type]
    ON tbl_name (index_sharding_col_name,...)
    global_secondary_index_option
    [index_option]
    [algorithm_option | lock_option] ...
# 全局二级索引特有语法，具体说明请参见CREATE TABLE文档
global_secondary_index_option:
    [COVERING (col_name,...)]
    drds_partition_options
# 分库分表子句，具体说明请参见CREATE TABLE文档
drds_partition_options:
    DBPARTITION BY db_sharding_algorithm
    [TBPARTITION BY {table_sharding_algorithm} [TBPARTITIONS num]]
db_sharding_algorithm:
    HASH([col_name])
  | {YYYYMM|YYYYWEEK|YYYYDD|YYYYMM_OPT|YYYYWEEK_OPT|YYYYDD_OPT}(col_name)
  | UNI_HASH(col_name)
  | RIGHT_SHIFT(col_name, n)
  | RANGE_HASH(col_name, col_name, n)
table_sharding_algorithm:
    HASH(col_name)
  | {MM|DD|WEEK|MMDD|YYYYMM|YYYYWEEK|YYYYDD|YYYYMM_OPT|YYYYWEEK_OPT|YYYYDD_OPT}(col_name)
  | UNI_HASH(col_name)
  | RIGHT_SHIFT(col_name, n)
  | RANGE_HASH(col_name, col_name, n)
 # 以下为MySQL DDL语法
index_sharding_col_name:
    col_name [(length)] [ASC | DESC] # length参数仅用于在索引表拆分键上创建局部索引
index_option:
    KEY_BLOCK_SIZE [=] value
  | index_type
  | WITH PARSER parser_name
  | COMMENT 'string'
index_type:
    USING {BTREE | HASH}
algorithm_option:
    ALGORITHM [=] {DEFAULT|INPLACE|COPY}
lock_option:
    LOCK [=] {DEFAULT|NONE|SHARED|EXCLUSIVE}
```



`CREATE GLOBAL INDEX`系列语法用于在建表后添加GSI，该系列语法在MySQL语法上新引入了GLOBAL关键字，用于指定添加的索引类型为GSI。目前建表后创建GSI存在一定限制，关于GSI的限制与约定，详情请参见[如何使用全局二级索引](gsi-faq.md)。

关于全局二级索引定义子句详细说明，请参见[CREATE TABLE](create-table.md)。

**示例** 

下面以建立普通全局二级索引为例，介绍在建表后创建GSI。

```sql
# 创建表
CREATE TABLE t_order (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `order_id` varchar(20) DEFAULT NULL,
  `buyer_id` varchar(20) DEFAULT NULL,
  `seller_id` varchar(20) DEFAULT NULL,
  `order_snapshot` longtext DEFAULT NULL,
  `order_detail` longtext DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `l_i_order` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
# 创建全局二级索引
ALTER TABLE t_order ADD UNIQUE GLOBAL INDEX `g_i_buyer` (`buyer_id`) COVERING (`order_snapshot`) dbpartition by hash(`buyer_id`);
```



* 主表："t_order"只分库不分表，分库的拆分方式为按照"order_id"列进行哈希。

* 索引表："g_i_buyer"只分库不分表，分库的拆分方式为按照"buyer_id"列进行哈希，指定覆盖列为"order_snapshot"。

* 索引定义子句：``GLOBAL INDEX `g_i_seller` ON t_order (`seller_id`) dbpartition by hash(`seller_id`)``。




通过`SHOW INDEX`查看索引信息，包含拆分键order_id上的局部索引，和seller_id、id和order_id上的GSI，其中seller_id为索引表的拆分键，id和order_id为默认的覆盖列（主键和主表的拆分键）。
**说明** 关于GSI的限制与约定，详情请参见[如何使用全局二级索引](gsi-faq.md)，SHOW INDEX详细说明，请参见[SHOW INDEX](show-index.md)。

```sql
mysql> show index from t_order;  
+---------+------------+------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+   
| TABLE   | NON_UNIQUE | KEY_NAME   | SEQ_IN_INDEX | COLUMN_NAME | COLLATION | CARDINALITY | SUB_PART | PACKED | NULL | INDEX_TYPE | COMMENT  | INDEX_COMMENT |    
+---------+------------+------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+   
| t_order |          0 | PRIMARY    |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |          |               |  
| t_order |          1 | l_i_order  |            1 | order_id    | A         |           0 |     NULL | NULL   | YES  | BTREE      |          |               |   
| t_order |          1 | g_i_seller |            1 | seller_id   | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | INDEX    |               | 
| t_order |          1 | g_i_seller |            2 | id          | NULL      |           0 |     NULL | NULL   |      | GLOBAL     | COVERING |               |   
| t_order |          1 | g_i_seller |            3 | order_id    | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |   
+---------+------------+------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
```



通过`SHOW GLOBAL INDEX`可以单独查看GSI信息，详情请参见[SHOW GLOBAL INDEX](show-global-index.md)。

```sql
mysql> show global index from t_order;    
+---------------------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+    
| SCHEMA              | TABLE   | NON_UNIQUE | KEY_NAME   | INDEX_NAMES | COVERING_NAMES | INDEX_TYPE | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT | STATUS |
+---------------------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+    
| ZZY3_DRDS_LOCAL_APP | t_order | 1          | g_i_seller | seller_id   | id, order_id   | NULL       | seller_id        | HASH                | 4                  |                  | NULL                | NULL               | PUBLIC |   
+---------------------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
```



查看索引表的结构，索引表包含主表的主键、分库分表键、默认的覆盖列和自定义覆盖列，主键列去除了`AUTO_INCREMENT`属性，并且去除了主表中的局部索引。

```sql
mysql> show create table g_i_seller;   
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+   
| Table      | Create Table                                                                                                                                                                                                                                                                                  |   
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+  
| g_i_seller | CREATE TABLE `g_i_seller` (  
  `id` bigint(11) NOT NULL,  
  `order_id` varchar(20) DEFAULT NULL,  
  `seller_id` varchar(20) DEFAULT NULL,   
  PRIMARY KEY (`id`),  
  KEY `auto_shard_key_seller_id` (`seller_id`) USING BTREE  
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`seller_id`) | 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   
```



