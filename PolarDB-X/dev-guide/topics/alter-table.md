ALTER TABLE 
================================

您可以通过ALTER TABLE语法改变表的结构，如增加列、增加索引、修改数据定义等。

注意事项 
-------------------------

不支持通过ALTER TABLE语法修改拆分字段。

语法 
-----------------------

**说明** ALTER TABLE用于改变表的结构，如增加列、增加索引和修改数据定义。详细语法请参见[MySQL修改表语法](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)

```sql
ALTER [ONLINE|OFFLINE] [IGNORE] TABLE tbl_name
[alter_specification [, alter_specification] ...]
[partition_options]
```



**示例** 

* 增加列在"user_log"表中增加一列"idcard"，示例如下：

  ```sql
  ALTER TABLE user_log
  ADD COLUMN idcard varchar(30);
  ```

  

* 增加局部索引在"user_log"表中为"idcard"列增加一个名为"idcard_idx"的索引，示例如下：

  ```sql
  ALTER TABLE user_log
  ADD INDEX idcard_idx (idcard);
  ```

  

* 重命名局部索引将"user_log"表中"idcard_idx"索引命修改为"idcard_idx_new"，示例如下：

  ```sql
  ALTER TABLE user_log
  RENAME INDEX `idcard_idx` TO `idcard_idx_new`;
  ```

  

* 删除局部索引删除"user_log"表中的"idcard_idx"索引，示例如下：

  ```sql
  ALTER TABLE user_log
  DROP INDEX idcard_idx;
  ```

  

* 修改字段将"user_log"表中"idcard"列（字段类型为varchar）的长度由30改为40，语法示例如下：

  ```sql
  ALTER TABLE user_log
  MODIFY COLUMN idcard varchar(40);
  ```

  




全局二级索引 
---------------------------

PolarDB-X支持全局二级索引 (Global Secondary Index, GSI)，基本原理请参见[全局二级索引](../../features/topics/gsi.md)。

**列变更** 

使用全局二级索引的表，对列的修改，语法和普通表的一致。
**说明** 当修改的表包含全局二级索引时，对列的修改有额外的限制，关于GSI的限制与约定，详情请参见[如何使用全局二级索引](gsi-faq.md)。

**索引变更** 

语法

```sql
ALTER TABLE tbl_name
    alter_specification # 全局二级索引相关变更仅支持一条alter_specification
alter_specification:
  | ADD GLOBAL {INDEX|KEY} index_name # 全局二级索引必须显式指定索引名
      [index_type] (index_sharding_col_name,...)
      global_secondary_index_option
      [index_option] ...
  | ADD [CONSTRAINT [symbol]] UNIQUE GLOBAL
      [INDEX|KEY] index_name # 全局二级索引必须显式指定索引名
      [index_type] (index_sharding_col_name,...)
      global_secondary_index_option
      [index_option] ...
  | DROP {INDEX|KEY} index_name
  | RENAME {INDEX|KEY} old_index_name TO new_index_name
global_secondary_index_option:
    [COVERING (col_name,...)] # Covering Index
    drds_partition_options # 包含且仅包含index_sharding_col_name中指定的列
# 指定索引表拆分方式
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
    col_name [(length)] [ASC | DESC]
index_option:
    KEY_BLOCK_SIZE [=] value
  | index_type
  | WITH PARSER parser_name
  | COMMENT 'string'
index_type:
    USING {BTREE | HASH}
```



`ALTER TABLE ADD GLOBAL INDEX`系列语法用于在建表后添加GSI，该系列语法在MySQL语法上新引入了GLOBAL关键字，用于指定添加的索引类型为GSI。

`ALTER TABLE { DROP | RENAME } INDEX`语法同样可以对GSI进行修改，目前建表后创建GSI存在一定限制。关于GSI的限制与约定，详情请参见[如何使用全局二级索引](gsi-faq.md)。

全局二级索引定义子句详细说明请参见[CREATE TABLE](create-table.md)。

示例

* **建表后添加全局二级索引**

  下面以建立全局唯一索引为例，介绍在建表后如何创建GSI。

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
  
  
  
  通过`SHOW INDEX`查看索引信息，包含拆分键order_id上的局部索引，和buyer_id、id、order_id和order_snapshot上的GSI，其中buyer_id为索引表的拆分键，id和order_id为默认的覆盖列（主键和主表的拆分键），order_snapshot显式指定的覆盖列。
  **说明** 关于GSI的限制与约定，详情请参见[如何使用全局二级索引](gsi-faq.md)，SHOW INDEX详细说明，请参见[SHOW INDEX](show-index.md)。
  
  ```sql
  mysql> show index from t_order;
  +---------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
  | TABLE   | NON_UNIQUE | KEY_NAME  | SEQ_IN_INDEX | COLUMN_NAME    | COLLATION | CARDINALITY | SUB_PART | PACKED | NULL | INDEX_TYPE | COMMENT  | INDEX_COMMENT |
  +---------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
  | t_order |          0 | PRIMARY   |            1 | id             | A         |           0 |     NULL | NULL   |      | BTREE      |          |               |
  | t_order |          1 | l_i_order |            1 | order_id       | A         |           0 |     NULL | NULL   | YES  | BTREE      |          |               |
  | t_order |          0 | g_i_buyer |            1 | buyer_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | INDEX    |               |
  | t_order |          1 | g_i_buyer |            2 | id             | NULL      |           0 |     NULL | NULL   |      | GLOBAL     | COVERING |               |
  | t_order |          1 | g_i_buyer |            3 | order_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
  | t_order |          1 | g_i_buyer |            4 | order_snapshot | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
  +---------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
  ```
  
  
  
  通过`SHOW GLOBAL INDEX`可以单独查看GSI信息，详情请参见[SHOW GLOBAL INDEX](show-global-index.md)。
  
  ```sql
  mysql> show global index from t_order;
  +---------------------+---------+------------+-----------+-------------+------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
  | SCHEMA              | TABLE   | NON_UNIQUE | KEY_NAME  | INDEX_NAMES | COVERING_NAMES               | INDEX_TYPE | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT | STATUS |
  +---------------------+---------+------------+-----------+-------------+------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
  | ZZY3_DRDS_LOCAL_APP | t_order | 0          | g_i_buyer | buyer_id    | id, order_id, order_snapshot | NULL       | buyer_id         | HASH                | 4                  |                  | NULL                | NULL               | PUBLIC |
  +---------------------+---------+------------+-----------+-------------+------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
  ```
  
  
  
  查看索引表的结构，索引表包含主表的主键、分库分表键、默认的覆盖列和自定义覆盖列，主键列去除了AUTO_INCREMENT属性，并且去除了主表中的局部索引，全局唯一索引默认在索引表的所有分库分表键上创建一个唯一索引，以实现全局唯一约束。
  
  ```sql
  mysql> show create table g_i_buyer;
  +-----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Table     | Create Table                                                                                                                                                                                                                                                                                                                 |
  +-----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | g_i_buyer | CREATE TABLE `g_i_buyer` (`id` bigint(11) NOT NULL, `order_id` varchar(20) DEFAULT NULL, `buyer_id` varchar(20) DEFAULT NULL, `order_snapshot` longtext, PRIMARY KEY (`id`), UNIQUE KEY `auto_shard_key_buyer_id` (`buyer_id`) USING BTREE) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`buyer_id`)               |
  +-----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  ```
  
  
  
  
  
* **删除全局二级索引**

  删除名为g_i_seller的GSI，相应的索引表也将被删除。

  ```sql
  # 删除索引
  ALTER TABLE `t_order` DROP INDEX `g_i_seller`;
  ```

  

* **重命名索引**

  默认情况下限制对GSI的重命名。关于GSI的限制与约定，详情请参见[全局二级索引使用](gsi-faq.md)。
  



