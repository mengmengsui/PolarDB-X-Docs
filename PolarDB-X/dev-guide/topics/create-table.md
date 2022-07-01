CREATE TABLE 
=================================

本文主要介绍使用DDL语句进行建表的语法、子句、参数和基本方式。

语法 
-----------------------

```sql
CREATE [SHADOW] TABLE [IF NOT EXISTS] tbl_name
    (create_definition, ...)
    [table_options]
    [drds_partition_options]
create_definition:
    col_name column_definition
  | mysql_create_definition
  | [UNIQUE] GLOBAL INDEX index_name [index_type] (index_sharding_col_name,...)
      [global_secondary_index_option]
      [index_option] ...
# 全局二级索引相关
global_secondary_index_option:
    [COVERING (col_name,...)]
    [drds_partition_options]
# 分库分表子句
drds_partition_options:
    DBPARTITION BY db_partition_algorithm
    [TBPARTITION BY table_partition_algorithm [TBPARTITIONS num]]
    [LOCALITY=locality_option]
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
# 可以在创建单表时指定该表的存储位置
locality_option:
    'dn=storage_inst_id_list'
storage_inst_id_list:
    storage_inst_id[,storage_inst_id_list]
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

**说明** PolarDB-X DDL语法基于MySQL语法，以上主要列出了差异部分，详细语法请参见[MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/create-table.html) 。

分库分表子句和参数 
------------------------------

* `DBPARTITION BY hash(partition_key)`：指定分库键和分库算法。

* `TBPARTITION BY { HASH(column) | {MM|DD|WEEK|MMDD|YYYYMM|YYYYWEEK|YYYYDD|YYYYMM_OPT|YYYYWEEK_OPT|YYYYDD_OPT}(column)`（可选）：默认与`DBPARTITION BY`相同，指定物理表使用什么方式映射数据。

* `TBPARTITIONS num`（可选）：每个库上的物理表数目（默认为1），如不分表，就不需要指定该字段。

* 拆分函数的详细介绍，请参见[拆分函数](sharding-functions.md) 。




全局二级索引定义子句 
-------------------------------

* `[UNIQUE] GLOBAL`：定义全局二级索引，UNIQUE GLOBAL代表全局唯一索引。

* `index_name`：索引名，也是索引表的名称。

* `index_type`：索引表中分库分表键上局部索引的类型，支持范围请参见[MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/create-table.html#create-table-indexes-keys) 。

* `index_sharding_col_name,...`：索引列，包含且仅包含索引表的全部分库分表键。详情请参见[全局二级索引](gsi-faq.md)。

* `global_secondary_index_option`：全局二级索引的扩展语法。
  * `COVERING (col_name,...)`：覆盖列，索引表中除索引列以外的其他列，默认包含主键和主表的分库分表键。详情请参见[全局二级索引](gsi-faq.md)。
  
  * `drds_partition_options`：索引表的分库分表子句，详情请参见本小节中的分库分表子句和参数。
  
  
  
* `index_option`：索引表中分库分表键上局部索引的属性，详情请参见[MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/create-table.html#create-table-indexes-keys) 。




全链路压测影子表子句 
-------------------------------

`SHADOW`：创建全链路压测影子表，表名必须以`_test_`为前缀，前缀后的表名部分必须与关联的正式表名一致，且正式表必须先于影子表创建。

LOCALITY 
-----------------------------

通过LOCALITY指定单表的存储位置。
**说明**

* 通过LOCALITY语法指定了数据库或单表的位置之后，不支持再修改该库或该表的存储位置。

* 数据库中单表的存储位置不受该库存储位置的影响。创建单表时若未指定存储位置，则会被随机放置在一个存储节点上，且后续该PolarDB-X实例上创建的所有未指定存储位置的单表，均会被放置在该存储节点上。




单库单表 
-------------------------

建一张单库单表，不做任何拆分。

```sql
CREATE TABLE single_tbl(
 id bigint not null auto_increment, 
 name varchar(30), 
 primary key(id)
);
```



查看逻辑表的节点拓扑，可以看出只在0库创建了一张单库单表的逻辑表。

```sql
mysql> show topology from single_tbl;
+------+------------------------------------------------------------------+------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME |
+------+------------------------------------------------------------------+------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | single_tbl |
+------+------------------------------------------------------------------+------------+
1 row in set (0.01 sec)
```



假设已有一个PolarDB-X实例，该实例上已有 *polardbx-storage-0-master* 和 *polardbx-storage-1-master* 两个存储节点，其中 *polardbx-storage-1-master* 上已创建了一个数据库db1。现需要在db1库中创建一张表，并指定其的存储位置为 *polardbx-storage-0-master* ，语法如下：

```sql
mysql>CREATE TABLE tb1 (id int) LOCALITY='dn=polardbx-storage-0-master';
```

创建成功后，您可以通过如下语句查看该表的拓扑结构：

```sql
mysql>SHOW TOPOLOGY FROM tb1;
```

返回结果如下：

```sql
+----+------------------+------------+
| ID | GROUP_NAME       | TABLE_NAME | 
+----+------------------+------------+
| 0  | DB1_000000_GROUP | tb1        |
+----+------------------+------------+
1 row in set
```


**说明** 以上返回结果表示tb1表位于DB1_000000_GROUP分库。

分库不分表 
--------------------------

假设已经建好的分库数为8，建一张表，只分库不分表，分库方式为根据ID列哈希。

```sql
CREATE TABLE multi_db_single_tbl(
  id bigint not null auto_increment, 
  name varchar(30), 
  primary key(id)
) dbpartition by hash(id);
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了1张分表，既只做了分库。

```sql
mysql> show topology from multi_db_single_tbl;
+------+------------------------------------------------------------------+---------------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME          |
+------+------------------------------------------------------------------+---------------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | multi_db_single_tbl |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | multi_db_single_tbl |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0002_RDS | multi_db_single_tbl |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0003_RDS | multi_db_single_tbl |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0004_RDS | multi_db_single_tbl |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0005_RDS | multi_db_single_tbl |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0006_RDS | multi_db_single_tbl |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | multi_db_single_tbl |
+------+------------------------------------------------------------------+---------------------+
8 rows in set (0.01 sec)
```



分库分表 
-------------------------

您可以使用如下拆分方式进行分库分表：

* 使用哈希函数做拆分

* 使用双字段哈希函数做拆分

* 使用日期做拆分



**说明** 以下示例均假设已经建好的分库数为8。

使用哈希函数做拆分 
------------------------------

建一张表，既分库又分表，每个库含有3张物理表，分库拆分方式为按照ID列进行哈希，分表拆分方式为按照bid列进行哈希。您可以先根据ID列的值进行哈希运算，将表中数据分布在多个子库中，每个子库中的数据再根据bid列值的哈希运算结果分布在3个物理表中。

```sql
CREATE TABLE multi_db_multi_tbl(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30), 
 primary key(id)
) dbpartition by hash(id) tbpartition by hash(bid) tbpartitions 3;
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了3张分表。

```sql
mysql> show topology from multi_db_multi_tbl;
+------+------------------------------------------------------------------+-----------------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME            |
+------+------------------------------------------------------------------+-----------------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | multi_db_multi_tbl_00 |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | multi_db_multi_tbl_01 |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | multi_db_multi_tbl_02 |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | multi_db_multi_tbl_03 |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | multi_db_multi_tbl_04 |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | multi_db_multi_tbl_05 |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0002_RDS | multi_db_multi_tbl_06 |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0002_RDS | multi_db_multi_tbl_07 |
|    8 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0002_RDS | multi_db_multi_tbl_08 |
|    9 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0003_RDS | multi_db_multi_tbl_09 |
|   10 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0003_RDS | multi_db_multi_tbl_10 |
|   11 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0003_RDS | multi_db_multi_tbl_11 |
|   12 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0004_RDS | multi_db_multi_tbl_12 |
|   13 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0004_RDS | multi_db_multi_tbl_13 |
|   14 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0004_RDS | multi_db_multi_tbl_14 |
|   15 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0005_RDS | multi_db_multi_tbl_15 |
|   16 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0005_RDS | multi_db_multi_tbl_16 |
|   17 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0005_RDS | multi_db_multi_tbl_17 |
|   18 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0006_RDS | multi_db_multi_tbl_18 |
|   19 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0006_RDS | multi_db_multi_tbl_19 |
|   20 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0006_RDS | multi_db_multi_tbl_20 |
|   21 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | multi_db_multi_tbl_21 |
|   22 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | multi_db_multi_tbl_22 |
|   23 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | multi_db_multi_tbl_23 |
+------+------------------------------------------------------------------+-----------------------+
24 rows in set (0.01 sec)
```



查看该逻辑表的拆分规则，可以看出分库分表的拆分方式均为哈希，分库的拆分键为ID，分表的拆分键为bid。

```sql
mysql> show rule from multi_db_multi_tbl;
+------+--------------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME         | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+--------------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | multi_db_multi_tbl |         0 | id               | hash                | 8                  | bid              | hash                | 3                  |
+------+--------------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.01 sec)
```



使用双字段哈希函数做拆分 
---------------------------------

* 使用要求拆分键的类型必须是字符类型或数字类型。

* 路由方式根据任一拆分键后N位计算哈希值，以哈希方式完成路由计算。N为函数第三个参数。例如`RANGE_HASH(COL1, COL2, N)`，计算时会优先选择COL1，截取其后N位进行计算。COL1不存在时按COL2计算。

* 适用场景适合于需要有两个拆分键，并且仅使用其中一个拆分键值进行查询时的场景。假设用户的里已经分了8个物理库， 现业务有如下的场景：
  * 一个业务想按买家ID和订单ID对订单表进行分库。
  
  * 查询时条件仅有买家ID或订单ID。
  
  
  




此时可使用以下DDL对订单表进行构建：

```sql
create table test_order_tb (
 id bigint not null auto_increment,
 seller_id varchar(30) DEFAULT NULL,
 order_id varchar(30) DEFAULT NULL,
 buyer_id varchar(30) DEFAULT NULL,
 create_time datetime DEFAULT NULL,
 primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by RANGE_HASH(buyer_id, order_id, 10) tbpartition by RANGE_HASH(buyer_id, order_id, 10) tbpartitions 3;  
```


**说明**

* 两个拆分键皆不能修改。

* 插入数据时如果发现两个拆分键指向不同的分库或分表时，插入会失败。




使用日期做拆分 
----------------------------

除了可以使用哈希函数做拆分算法，您还可以使用日期函数`MM`、`DD`、`WEEK`或`MMDD`来作为分表的拆分算法，具体步骤请参见如下示例。

建一张表，既分库又分表，分库方式为根据`userId`列哈希，分表方式为根据`actionDate`列，按照一周七天来拆分（`WEEK(actionDate)`计算的是`DAY_OF_WEEK`）。

比如`actionDate`列的值是2017-02-27，这天是星期一，`WEEK(actionDate)`算出的值是2，该条记录就会被存储到`2（2 % 7 = 2）`这张分表（位于某个分库，具体的表名是 `user_log_2`）；比如`actionDate`列的值是2017-02-26，这天是星期天，`WEEK(actionDate)`算出的值是1，该条记录就会被存储到`1（1 % 7 = 1）`这张分表（位于某个分库，具体的表名是 `user_log_1`）。

```sql
CREATE TABLE user_log(
 userId int, 
 name varchar(30), 
 operation varchar(30), 
 actionDate DATE
) dbpartition by hash(userId) tbpartition by WEEK(actionDate) tbpartitions 7;
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了7张分表（一周7天）。

```sql
mysql> show topology from user_log;
+------+------------------------------------------------------------------+------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME |
+------+------------------------------------------------------------------+------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_0 |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_1 |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_2 |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_3 |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_4 |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_5 |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log_6 |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_0 |
|    8 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_1 |
|    9 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_2 |
|   10 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_3 |
|   11 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_4 |
|   12 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_5 |
|   13 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log_6 |
...
|   49 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_0 |
|   50 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_1 |
|   51 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_2 |
|   52 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_3 |
|   53 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_4 |
|   54 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_5 |
|   55 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log_6 |
+------+------------------------------------------------------------------+------------+
56 rows in set (0.01 sec)
```


**说明** 由于返回结果较长，这里用...做了省略处理。

查看该逻辑表的拆分规则，可以看出分库的拆分方式为哈希，分库的拆分键为`userId`，分表的拆分方式为按照时间函数`WEEK`进行拆分，分表的拆分键为`actionDate`。

```sql
mysql> show rule from user_log;
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | user_log   |         0 | userId           | hash                | 8                  | actionDate       | week                | 7                  |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.00 sec)
```



查看给定分库键和分表键参数时， SQL被路由到哪个物理分库和该物理分库下的哪张物理表。

建一张表，既分库又分表，分库方式为根据`userId`列哈希，分表方式为根据`actionDate`列，按照一年12个月进行拆分（`MM(actionDate)`计算的是`MONTH_OF_YEAR`）。

比如`actionDate`列的值是2017-02-27，`MM(actionDate)`算出的值是02，该条记录就会被存储到`02（02 % 12 = 02）`这张分表（位于某个分库，具体的表名是 `user_log_02`）。比如`actionDate`列的值是2016-12-27，`MM(actionDate)`算出的值是12，该条记录就会被存储到`00（12 % 12 = 00）`这张分表（位于某个分库，具体的表名是 `user_log_00`）。

```sql
CREATE TABLE user_log2(
 userId int, 
 name varchar(30), 
 operation varchar(30), 
 actionDate DATE
) dbpartition by hash(userId) tbpartition by MM(actionDate) tbpartitions 12; 
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了12张分表（1年有12个月）。

```sql
mysql> show topology from user_log2;
+------+------------------------------------------------------------------+--------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME   |
+------+------------------------------------------------------------------+--------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_00 |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_01 |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_02 |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_03 |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_04 |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_05 |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_06 |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_07 |
|    8 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_08 |
|    9 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_09 |
|   10 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_10 |
|   11 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log2_11 |
|   12 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_00 |
|   13 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_01 |
|   14 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_02 |
|   15 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_03 |
|   16 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_04 |
|   17 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_05 |
|   18 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_06 |
|   19 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_07 |
|   20 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_08 |
|   21 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_09 |
|   22 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_10 |
|   23 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0001_RDS | user_log2_11 |
...
|   84 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_00 |
|   85 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_01 |
|   86 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_02 |
|   87 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_03 |
|   88 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_04 |
|   89 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_05 |
|   90 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_06 |
|   91 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_07 |
|   92 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_08 |
|   93 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_09 |
|   94 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_10 |
|   95 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log2_11 |
+------+------------------------------------------------------------------+--------------+
96 rows in set (0.02 sec)
```


**说明** 由于返回结果较长，这里用...做了省略处理。

查看该逻辑表的拆分规则，可以看出分库的拆分方式为哈希，分库的拆分键为`userId`，分表的拆分方式为按照时间函数`MM`进行拆分，分表的拆分键为`actionDate`。

```sql
mysql> show rule from user_log2;
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | user_log2  |         0 | userId           | hash                | 8                  | actionDate       | mm                  | 12                 |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.00 sec)
```



建一张表，既分库又分表，分库方式为根据`userId`列哈希，分表方式为按照一个月31天进行拆分（函数`DD(actionDate)`计算的是`DAY_OF_MONTH`）。

比如`actionDate`列的值是2017-02-27，`DD(actionDate)`算出的值是27，该条记录就会被存储到`27（27 % 31 = 27）`这张分表（位于某个分库，具体的表名是`user_log_27`）。

```sql
CREATE TABLE user_log3(
 userId int, 
 name varchar(30), 
 operation varchar(30), 
 actionDate DATE
) dbpartition by hash(userId) tbpartition by DD(actionDate) tbpartitions 31;
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了31张分表（按每个月有31天处理）。

```sql
mysql> show topology from user_log3;
+------+------------------------------------------------------------------+--------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME   |
+------+------------------------------------------------------------------+--------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_00 |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_01 |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_02 |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_03 |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_04 |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_05 |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_06 |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_07 |
|    8 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_08 |
|    9 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_09 |
|   10 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_10 |
|   11 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_11 |
|   12 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_12 |
|   13 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_13 |
|   14 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_14 |
|   15 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_15 |
|   16 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_16 |
|   17 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_17 |
|   18 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_18 |
|   19 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_19 |
|   20 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_20 |
|   21 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_21 |
|   22 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_22 |
|   23 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_23 |
|   24 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_24 |
|   25 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_25 |
|   26 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_26 |
|   27 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_27 |
|   28 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_28 |
|   29 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_29 |
|   30 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log3_30 |
...
|  237 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_20 |
|  238 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_21 |
|  239 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_22 |
|  240 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_23 |
|  241 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_24 |
|  242 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_25 |
|  243 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_26 |
|  244 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_27 |
|  245 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_28 |
|  246 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_29 |
|  247 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log3_30 |
+------+------------------------------------------------------------------+--------------+
248 rows in set (0.01 sec)
```


**说明** 由于返回的结果较长，这里用...做了省略处理。

查看该逻辑表的拆分规则，可以看出分库的拆分方式为哈希，分库的拆分键为`userId`，分表的拆分方式为按照时间函数`DD`进行拆分，分表的拆分键为`actionDate`。

```sql
mysql> show rule from user_log3;
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | user_log3  |         0 | userId           | hash                | 8                  | actionDate       | dd                  | 31                 |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.01 sec)
```



建一张表，既分库又分表，分库方式为根据`userId`列哈希，分表方式为按照一年365天进行拆分，路由到365张物理表（`MMDD(actionDate) tbpartitions 365`计算的是`DAY_OF_YEAR % 365`。

比如`actionDate`列的值是2017-02-27，`MMDD(actionDate)`算出的值是58，该条记录就会被存储到58这张分表（位于某个分库，具体的表名是`user_log_58`）。

```sql
CREATE TABLE user_log4(
 userId int, 
 name varchar(30), 
 operation varchar(30), 
 actionDate DATE
) dbpartition by hash(userId) tbpartition by MMDD(actionDate) tbpartitions 365;
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了365张分表（按每年有365天处理）。

```sql
mysql> show topology from user_log4;
+------+------------------------------------------------------------------+---------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME   |
+------+------------------------------------------------------------------+---------------+
...
| 2896 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_341 |
| 2897 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_342 |
| 2898 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_343 |
| 2899 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_344 |
| 2900 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_345 |
| 2901 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_346 |
| 2902 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_347 |
| 2903 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_348 |
| 2904 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_349 |
| 2905 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_350 |
| 2906 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_351 |
| 2907 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_352 |
| 2908 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_353 |
| 2909 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_354 |
| 2910 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_355 |
| 2911 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_356 |
| 2912 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_357 |
| 2913 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_358 |
| 2914 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_359 |
| 2915 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_360 |
| 2916 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_361 |
| 2917 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_362 |
| 2918 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_363 |
| 2919 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log4_364 |
+------+------------------------------------------------------------------+---------------+
2920 rows in set (0.07 sec)
```


**说明** 由于返回的结果较长，这里用...做了省略处理。

查看该逻辑表的拆分规则，可以看出分库的拆分方式为哈希，分库的拆分键为`userId`，分表的拆分方式为按照时间函数`MMDD`进行拆分，分表的拆分键为`actionDate`。

```sql
mysql> show rule from user_log4;
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | user_log4  |         0 | userId           | hash                | 8                  | actionDate       | mmdd                | 365                |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.02 sec)
```



建一张表，既分库又分表，分库方式为根据`userId`列哈希，分表方式为按照一年365天进行拆分，路由到10张物理表（`MMDD(actionDate) tbpartitions 10`计算的是`DAY_OF_YEAR % 10`。

```sql
CREATE TABLE user_log5(
 userId int, 
 name varchar(30), 
 operation varchar(30), 
 actionDate DATE
) dbpartition by hash(userId) tbpartition by MMDD(actionDate) tbpartitions 10;
```



查看该逻辑表的节点拓扑，可以看出在每个分库都创建了10张分表（按照一年365天进行拆分，路由到10张物理表）。

```sql
mysql> show topology from user_log5;
+------+------------------------------------------------------------------+--------------+
| ID   | GROUP_NAME                                                       | TABLE_NAME   |
+------+------------------------------------------------------------------+--------------+
|    0 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_00 |
|    1 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_01 |
|    2 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_02 |
|    3 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_03 |
|    4 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_04 |
|    5 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_05 |
|    6 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_06 |
|    7 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_07 |
|    8 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_08 |
|    9 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0000_RDS | user_log5_09 |
...
|   70 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_00 |
|   71 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_01 |
|   72 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_02 |
|   73 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_03 |
|   74 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_04 |
|   75 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_05 |
|   76 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_06 |
|   77 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_07 |
|   78 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_08 |
|   79 | SANGUAN_TEST_123_1488766060743ACTJSANGUAN_TEST_123_WVVP_0007_RDS | user_log5_09 |
+------+------------------------------------------------------------------+--------------+
80 rows in set (0.02 sec)
```


**说明** 由于返回的结果较长，这里用...做了省略处理。

查看该逻辑表的拆分规则，可以看出分库的拆分方式为哈希，分库的拆分键为`userId`，分表的拆分方式为按照时间函数 `MMDD`进行拆分，路由到10张物理表，分表的拆分键为`actionDate`。

```sql
mysql> show rule from user_log5;
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID   | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|    0 | user_log5  |         0 | userId           | hash                | 8                  | actionDate       | mmdd                | 10                 |
+------+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
1 row in set (0.01 sec)
```



使用主键作为拆分键 
------------------------------

当拆分算法不指定任何拆分字段时，系统默认使用主键作为拆分字段。以下示例将介绍如何使用主键当分库和分表键。

* 使用主键当分库键

  ```sql
  CREATE TABLE prmkey_tbl(
   id bigint not null auto_increment, 
   name varchar(30), 
   primary key(id)
  ) dbpartition by hash();
  ```

  

* 使用主键当分库分表键

  ```sql
  CREATE TABLE prmkey_multi_tbl(
   id bigint not null auto_increment, 
   name varchar(30), 
   primary key(id)
  ) dbpartition by hash() tbpartition by hash() tbpartitions 3;
  ```

  




广播表 
------------------------

子句`BROADCAST`用来指定创建广播表。广播表是指将这个表复制到每个分库上，在分库上通过同步机制实现数据一致，有秒级延迟。这样做的好处是可以将JOIN操作下推到底层的RDS（MySQL），来避免跨库JOIN。关于如何使用广播表来做SQL优化，详情请参见[SQL调优指南](../../sql-tunning/topics/sql-tuning-guide.md)。

```sql
CREATE TABLE brd_tbl(
  id bigint not null auto_increment, 
  name varchar(30), 
  primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 BROADCAST;
```



其他MySQL建表属性 
--------------------------------

您在分库分表的同时还可以指定其他的MySQL建表属性，例如：

```sql
CREATE TABLE multi_db_multi_tbl(
  id bigint not null auto_increment, 
  name varchar(30), 
  primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(id) tbpartition by hash(id) tbpartitions 3;
```



全局二级索引 
---------------------------

本小节介绍如何在建表时定义全局二级索引：

* 定义全局二级索引

* 定义全局唯一索引



**说明** 以下示例均假设已经建好的分库数为8。

定义全局二级索引 
-----------------------------

**示例** 

```sql
CREATE TABLE t_order (
 `id` bigint(11) NOT NULL AUTO_INCREMENT,
 `order_id` varchar(20) DEFAULT NULL,
 `buyer_id` varchar(20) DEFAULT NULL,
 `seller_id` varchar(20) DEFAULT NULL,
 `order_snapshot` longtext DEFAULT NULL,
 `order_detail` longtext DEFAULT NULL,
 PRIMARY KEY (`id`),
 GLOBAL INDEX `g_i_seller`(`seller_id`) dbpartition by hash(`seller_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
```



其中：

* 主表：`t_order`只分库不分表，分库的拆分方式为按照`order_id`列进行哈希。

* 索引表：`g_i_seller`只分库不分表，分库的拆分方式为按照`seller_id`列进行哈希，未指定覆盖列。

* 索引定义子句：``GLOBAL INDEX `g_i_seller`(`seller_id`) dbpartition by hash(`seller_id`)``。



通过`SHOW INDEX`查看索引信息，包含拆分键`order_id`上的局部索引，和`seller_id`、`id`、`order_id`上的GSI，其中`seller_id`为索引表的拆分键，`id`和`order_id`为默认的覆盖列（主键和主表的拆分键）。
**说明** 关于GSI的限制与约定，请参见[使用全局二级索引时的注意事项](gsi-faq.md) ，关于SHOW INDEX详细说明，请参见[SHOW INDEX](show-index.md) 。

```sql
mysql> show index from t_order;
+---------+------------+-------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| TABLE   | NON_UNIQUE | KEY_NAME                | SEQ_IN_INDEX | COLUMN_NAME | COLLATION | CARDINALITY | SUB_PART | PACKED | NULL | INDEX_TYPE | COMMENT  | INDEX_COMMENT |
+---------+------------+-------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| t_order |          0 | PRIMARY                 |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |          |               |
| t_order |          1 | auto_shard_key_order_id |            1 | order_id    | A         |           0 |     NULL | NULL   | YES  | BTREE      |          |               |
| t_order |          1 | g_i_seller              |            1 | seller_id   | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | INDEX    |               |
| t_order |          1 | g_i_seller              |            2 | id          | NULL      |           0 |     NULL | NULL   |      | GLOBAL     | COVERING |               |
| t_order |          1 | g_i_seller              |            3 | order_id    | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
+---------+------------+-------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
```



通过`SHOW GLOBAL INDEX`可以单独查看GSI信息。详细说明请参见[SHOW GLOBAL INDEX](show-global-index.md)。

```sql
mysql> show global index from t_order;
+--------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
| SCHEMA | TABLE   | NON_UNIQUE | KEY_NAME   | INDEX_NAMES | COVERING_NAMES | INDEX_TYPE | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT | STATUS |
+--------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
| d7     | t_order | 1          | g_i_seller | seller_id   | id, order_id   | NULL       | seller_id        | HASH                | 8                  |                  | NULL                | NULL               | PUBLIC |
+--------+---------+------------+------------+-------------+----------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
```



查看索引表的结构，索引表包含主表的主键、分库分表键和默认的覆盖列，主键列去除了`AUTO_INCREMENT`属性，并且去除了主表中的局部索引。

```sql
mysql> show create table g_i_seller;
+------------+-----------------------------------------------------------+
| Table      | Create Table                                              |
+------------+-----------------------------------------------------------+
| g_i_seller | CREATE TABLE `g_i_seller` (
 `id` bigint(11) NOT NULL,
 `order_id` varchar(20) DEFAULT NULL,
 `seller_id` varchar(20) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `auto_shard_key_seller_id` (`seller_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`seller_id`) |
+------------+-----------------------------------------------------------+
```



定义全局唯一索引 
-----------------------------

```sql
CREATE TABLE t_order (
 `id` bigint(11) NOT NULL AUTO_INCREMENT,
 `order_id` varchar(20) DEFAULT NULL,
 `buyer_id` varchar(20) DEFAULT NULL,
 `seller_id` varchar(20) DEFAULT NULL,
 `order_snapshot` longtext DEFAULT NULL,
 `order_detail` longtext DEFAULT NULL,
 PRIMARY KEY (`id`),
 UNIQUE GLOBAL INDEX `g_i_buyer`(`buyer_id`) COVERING(`seller_id`, `order_snapshot`) 
   dbpartition by hash(`buyer_id`) tbpartition by hash(`buyer_id`) tbpartitions 3
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
```



其中：

* 主表：`t_order` 只分库不分表，分库的拆分方式为按照`order_id`列进行哈希。

* 索引表：`g_i_buyer` 只分库且分表，分库和分表的拆分方式均为按照`buyer_id`列进行哈希，覆盖列包含 `seller_id`和`order_snapshot`。

* 索引定义子句：``UNIQUE GLOBAL INDEX `g_i_buyer`(`buyer_id`) COVERING(`seller_id`, `order_snapshot`) dbpartition by hash(`buyer_id`) tbpartition by hash(`buyer_id`) tbpartitions 3``。




通过`SHOW INDEX` 查看索引信息，包含拆分键`order_id`上的局部索引，和`buyer_id`、`id`、`order_id`、`seller_id`和`order_snapshot`上的GSI，其中`buyer_id`为索引表的拆分键，`id`和`order_id `为默认的覆盖列（主键和主表的拆分键），`seller_id`和`order_snapshot`为显示指定的覆盖列。
**说明** 关于GSI的限制与约定，请参见[使用全局二级索引时的注意事项](gsi-faq.md) ，关于SHOW INDEX详细说明，请参见[SHOW INDEX](show-index.md) 。

```sql
mysql> show index from t_order;
+--------------+------------+-------------------------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| TABLE        | NON_UNIQUE | KEY_NAME                | SEQ_IN_INDEX | COLUMN_NAME    | COLLATION | CARDINALITY | SUB_PART | PACKED | NULL | INDEX_TYPE | COMMENT  | INDEX_COMMENT |
+--------------+------------+-------------------------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| t_order_dthb |          0 | PRIMARY                 |            1 | id             | A         |           0 |     NULL | NULL   |      | BTREE      |          |               |
| t_order_dthb |          1 | auto_shard_key_order_id |            1 | order_id       | A         |           0 |     NULL | NULL   | YES  | BTREE      |          |               |
| t_order      |          0 | g_i_buyer               |            1 | buyer_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | INDEX    |               |
| t_order      |          1 | g_i_buyer               |            2 | id             | NULL      |           0 |     NULL | NULL   |      | GLOBAL     | COVERING |               |
| t_order      |          1 | g_i_buyer               |            3 | order_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
| t_order      |          1 | g_i_buyer               |            4 | seller_id      | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
| t_order      |          1 | g_i_buyer               |            5 | order_snapshot | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
+--------------+------------+-------------------------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
```



通过`SHOW GLOBAL INDEX`可以单独查看GSI信息。详细说明请参见[SHOW GLOBAL INDEX](show-global-index.md)。

```sql
mysql> show global index from t_order;
+--------+---------+------------+-----------+-------------+-----------------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
| SCHEMA | TABLE   | NON_UNIQUE | KEY_NAME  | INDEX_NAMES | COVERING_NAMES                          | INDEX_TYPE | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT | STATUS |
+--------+---------+------------+-----------+-------------+-----------------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
| d7     | t_order | 0          | g_i_buyer | buyer_id    | id, order_id, seller_id, order_snapshot | NULL       | buyer_id         | HASH                | 8                  | buyer_id         | HASH                | 3                  | PUBLIC |
+--------+---------+------------+-----------+-------------+-----------------------------------------+------------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+--------+
```



查看索引表的结构，索引表包含主表的主键、分库分表键、默认覆盖列和GSI定义中指定的覆盖列，主键列去除了`AUTO_INCREMENT`属性，并且去除了主表中局部索引，全局唯一索引默认会创建一份数据表来实现全局的唯一性支持。

```sql
mysql> show create table g_i_buyer;
+-----------+--------------------------------------------------------------------------------------------------------+
| Table     | Create Table                                                                                           |
+-----------+--------------------------------------------------------------------------------------------------------+
| g_i_buyer | CREATE TABLE `g_i_buyer` (
  `id` bigint(11) NOT NULL,
  `order_id` varchar(20) DEFAULT NULL,
  `buyer_id` varchar(20) DEFAULT NULL,
  `seller_id` varchar(20) DEFAULT NULL,
  `order_snapshot` longtext,
  PRIMARY KEY (`id`),
  UNIQUE KEY `auto_shard_key_buyer_id` (`buyer_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`buyer_id`) tbpartition by hash(`buyer_id`) tbpartitions 3 |
+-----------+--------------------------------------------------------------------------------------------------------+
```

