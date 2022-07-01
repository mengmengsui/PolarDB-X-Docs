规则和拓扑查询语句 
==============================

本文介绍了规则和拓扑类查询语句。

* SHOW RULE \[FROM tablename\]

* SHOW FULL RULE \[FROM tablename\]

* SHOW TOPOLOGY FROM tablename

* SHOW PARTITIONS FROM tablename

* SHOW BROADCASTS

* SHOW DATASOURCES

* SHOW NODE




SHOW RULE \[FROM tablename\] 
-------------------------------------------------

使用说明：

* `show rule`：查看数据库下每一个逻辑表的拆分情况；

* `show rule from tablename`：查看数据库下指定逻辑表的拆分情况。




```sql
mysql> show rule;
+----+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
| ID | TABLE_NAME | BROADCAST | DB_PARTITION_KEY | DB_PARTITION_POLICY | DB_PARTITION_COUNT | TB_PARTITION_KEY | TB_PARTITION_POLICY | TB_PARTITION_COUNT |
+----+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
|  0 | k_1        |         0 | k                | hash                | 40                 | k                | hash                | 2                  |
|  1 | k_2        |         0 | k                | hash                | 40                 | k                | hash                | 2                  |
|  2 | sbtest1    |         0 | id               | hash                | 40                 | id               | hash                | 2                  |
|  3 | t1         |         0 | id               | hash                | 40                 | id               | hash                | 4                  |
+----+------------+-----------+------------------+---------------------+--------------------+------------------+---------------------+--------------------+
4 rows in set (0.05 sec)
```



重要列详解：

* **BROADCAST** ：是否为广播表（0：否，1：是）；

* **DB_PARTITION_KEY** ：分库的拆分键，没有分库的话，值为空；

* **DB_PARTITION_POLICY** ：分库的拆分策略，取值包括哈希或YYYYMM、YYYYDD、YYYYWEEK等日期策略；

* **DB_PARTITION_COUNT** ：分库数；

* **TB_PARTITION_KEY** ：分表的拆分键，没有分表的话，值为空；

* **TB_PARTITION_POLICY** ：分表的拆分策略，取值包括哈希或MM、DD、MMDD、WEEK等日期策略；

* **TB_PARTITION_COUNT** ：分表数。




SHOW FULL RULE \[FROM tablename\] 
------------------------------------------------------

查看数据库下逻辑表的拆分规则，比SHOW RULE指令展示的信息更加详细。

```sql
mysql> show full rule;
+----+------------+-----------+------------+-----------------------+----------------------+--------------------------------------------------+-------------------+----------------------------------------+----------------+--------------------+
| ID | TABLE_NAME | BROADCAST | JOIN_GROUP | ALLOW_FULL_TABLE_SCAN | DB_NAME_PATTERN      | DB_RULES_STR                                     | TB_NAME_PATTERN   | TB_RULES_STR                           | PARTITION_KEYS | DEFAULT_DB_INDEX   |
+----+------------+-----------+------------+-----------------------+----------------------+--------------------------------------------------+-------------------+----------------------------------------+----------------+--------------------+
|  0 | k_1        |         0 | NULL       |                     1 | TEST1_{000000}_GROUP | ((#k,1,80#).longValue().abs() % 80).intdiv(2)    | k_1_cewR_{00}     | ((#k,1,80#).longValue().abs() % 80)    | k              | TEST1_SINGLE_GROUP |
|  1 | k_2        |         0 | NULL       |                     1 | TEST1_{000000}_GROUP | ((#k,1,80#).longValue().abs() % 80).intdiv(2)    | k_2_1xsQ_{00}     | ((#k,1,80#).longValue().abs() % 80)    | k              | TEST1_SINGLE_GROUP |
|  2 | sbtest1    |         0 | NULL       |                     1 | TEST1_{000000}_GROUP | ((#id,1,80#).longValue().abs() % 80).intdiv(2)   | sbtest1_wO5k_{00} | ((#id,1,80#).longValue().abs() % 80)   | id             | TEST1_SINGLE_GROUP |
|  3 | t1         |         0 | NULL       |                     1 | TEST1_{000000}_GROUP | ((#id,1,160#).longValue().abs() % 160).intdiv(4) | t1_EMrC_{000}     | ((#id,1,160#).longValue().abs() % 160) | id             | TEST1_SINGLE_GROUP |
+----+------------+-----------+------------+-----------------------+----------------------+--------------------------------------------------+-------------------+----------------------------------------+----------------+--------------------+
```



重要列详解：

* **BROADCAST** ：是否为广播表（0：否，1：是）；

* **JOIN_GROUP** ：保留字段，暂时无意义。

* **ALLOW_FULL_TABLE_SCAN** ：分库分表在没有指定分表键值的情况下是否允许查询数据，如果配置为true，此时需要扫描每一个物理表来查找出符合条件的数据，简称为全表扫描；

* **DB_NAME_PATTERN** ：DB_NAME_PATTERN中{}之间的0为占位符，执行SQL时会被DB_RULES_STR计算出的值替代，并保持位数。比如，DB_NAME_PATTERN的值为SEQ_{0000}_RDS，DB_RULES_STR的值为\[1,2,3,4\]，则会产生4个DB_NAME，分别为SEQ_0001_RDS、SEQ_0002_RDS、SEQ_0003_RDS、SEQ_0004_RDS；

* **DB_RULES_STR** ：具体的分库规则；

* **TB_NAME_PATTERN** ：TB_NAME_PATTERN中{}之间的0为占位符，执行SQL时会被TB_RULES_STR计算出的值替代，并保持位数。比如，TB_NAME_PATTERN的值为table_{00}，TB_RULES_STR的值为\[1,2,3,4,5,6,7,8\]，则会产生8张表，分别为table_01、table_02、table_03、table_04、table_05、table_06、table_07、table_08；

* **TB_RULES_STR** ：分表规则；

* **PARTITION_KEYS** ：分库和分表键集合，对于既分库又分表的情形，分库键在前，分表键在后；

* **DEFAULT_DB_INDEX** ：单库单表存放的分库。




SHOW TOPOLOGY FROM tablename 
-------------------------------------------------

查看指定逻辑表的拓扑分布，展示该逻辑表保存在哪些分库中，每个分库下包含哪些分表。

```sql
mysql> show topology from emp;   
+------+--------------------------------------------------+------------+  
| ID   | GROUP_NAME                                       | TABLE_NAME |  
+------+--------------------------------------------------+------------+  
|    0 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0000_RDS | emp_0      |   
|    1 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0000_RDS | emp_1      |   
|    2 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0001_RDS | emp_0      |    
|    3 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0001_RDS | emp_1      |    
|    4 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0002_RDS | emp_0      |    
|    5 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0002_RDS | emp_1      |    
|    6 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0003_RDS | emp_0      |    
|    7 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0003_RDS | emp_1      |    
|    8 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0004_RDS | emp_0      |    
|    9 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0004_RDS | emp_1      |    
|   10 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0005_RDS | emp_0      |    
|   11 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0005_RDS | emp_1      |    
|   12 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0006_RDS | emp_0      |    
|   13 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0006_RDS | emp_1      |   
|   14 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0007_RDS | emp_0      |   
|   15 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0007_RDS | emp_1      |    
+------+--------------------------------------------------+------------+
```



SHOW PARTITIONS FROM tablename 
---------------------------------------------------

查看分库分表键集合，分库键和分表键之间用逗号分割。如果最终结果有两个值，说明是既分库又分表的情形，第一个是分库键，第二个是分表键。如果结果只有一个值，说明是分库不分表的情形，该值是分库键。

```sql
mysql> show partitions from emp;
+-----------+
| KEYS      |
+-----------+
| emp_no,id |
+-----------+
1 row in set (0.00 sec)
```



SHOW BROADCASTS 
------------------------------------

查看广播表列表。

```sql
mysql> show broadcasts;
+------+------------+
| ID   | TABLE_NAME |
+------+------------+
|    0 | brd2       |
|    1 | brd_tbl    |
+------+------------+
2 rows in set (0.01 sec)
```



SHOW DATASOURCES 
-------------------------------------

查看底层存储信息，包含数据库名、数据库分组名、连接信息、用户名、底层存储类型、读写权重、连接池信息等。

```sql
mysql> show datasources;
+------+----------------------------+------------------------------------------------+--------------------------------------------------+----------------------------------------------------------------------------------+-----------+-------+------+------+------+--------------+----------+--------------+---------------+----------------------------------------------+-------------+--------------+   
| ID   | SCHEMA                     | NAME                                           | GROUP                                            | URL                                                                              | USER      | TYPE  | INIT | MIN  | MAX  | IDLE_TIMEOUT | MAX_WAIT | ACTIVE_COUNT | POOLING_COUNT | ATOM                                         | READ_WEIGHT | WRITE_WEIGHT |
+------+----------------------------+------------------------------------------------+--------------------------------------------------+----------------------------------------------------------------------------------+-----------+-------+------+------+------+--------------+----------+--------------+---------------+----------------------------------------------+-------------+--------------+  
|    0 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0000_iiab_1 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0000_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0000 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0000_iiab | 10          | 10           |  
|    1 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0001_iiab_2 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0001_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0001 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0001_iiab | 10          | 10           |   
|    2 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0002_iiab_3 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0002_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0002 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0002_iiab | 10          | 10           | 
|    3 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0003_iiab_4 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0003_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0003 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0003_iiab | 10          | 10           |   
|    4 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0004_iiab_5 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0004_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0004 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0004_iiab | 10          | 10           |   
|    5 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0005_iiab_6 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0005_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0005 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0005_iiab | 10          | 10           |  
|    6 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0006_iiab_7 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0006_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0006 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0006_iiab | 10          | 10           |  
|    7 | seq_test_1487767780814rgkk | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0007_iiab_8 | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0007_RDS | jdbc:mysql://rds1ur80kcv8g3t6p3ol.mysql.rds.aliyuncs.com:3306/seq_test_wnjg_0007 | jnkinsea0 | mysql | 0    | 24   | 72   | 15           | 5000     | 0            | 1             | rds1ur80kcv8g3t6p3ol_seq_test_wnjg_0007_iiab | 10          | 10           |  
+------+----------------------------+------------------------------------------------+--------------------------------------------------+----------------------------------------------------------------------------------+-----------+-------+------+------+------+--------------+----------+--------------+---------------+----------------------------------------------+-------------+--------------+
```



重要列详解：

* **SCHEMA** ：数据库名；

* **GROUP** ：数据库分组名。分组的目标是管理多组数据完全相同的数据库，比如通过 RDS（MySQL）进行数据复制后的主备数据库。主要用来解决读写分离、主备切换的问题；

* **URL** ：底层RDS（MySQL）的连接信息；

* **TYPE** ：底层存储类型，目前只支持RDS（MySQL)；

* **READ_WEIGHT** ：读权重。在主实例的读压力比较大的时候，可以通过读写分离功能将读流量进行分流，减轻RDS主实例的压力。PolarDB-X会自动识别读写流量，引导写流量进入RDS主实例，读流量则按配置的权重流向所有RDS实例；

* **WRITE_WEIGHT** ：写权重。




SHOW NODE 
------------------------------

查看物理库的读写次数（历史累计数据）、读写权重（历史累计数据）。

```sql
mysql> show node;
+------+--------------------------------------------------+-------------------+------------------+---------------------+--------------------+
| ID   | NAME                                             | MASTER_READ_COUNT | SLAVE_READ_COUNT | MASTER_READ_PERCENT | SLAVE_READ_PERCENT |
+------+--------------------------------------------------+-------------------+------------------+---------------------+--------------------+
| 0    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0000_RDS |                12 |                0 | 100%                | 0%                 |
| 1    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0001_RDS |                 0 |                0 | 0%                  | 0%                 |
| 2    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0002_RDS |                 0 |                0 | 0%                  | 0%                 |
| 3    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0003_RDS |                 0 |                0 | 0%                  | 0%                 |
| 4    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0004_RDS |                 0 |                0 | 0%                  | 0%                 |
| 5    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0005_RDS |                 0 |                0 | 0%                  | 0%                 |
| 6    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0006_RDS |                 0 |                0 | 0%                  | 0%                 |
| 7    | SEQ_TEST_1487767780814RGKKSEQ_TEST_WNJG_0007_RDS |                 0 |                0 | 0%                  | 0%                 |
+------+--------------------------------------------------+-------------------+------------------+---------------------+--------------------+
8 rows in set (0.01 sec)
```



重要列详解：

* **MASTER_COUNT** ：RDS主实例处理的读写查询次数（历史累计数据）；

* **SLAVE_COUNT** ：RDS备实例处理的只读查询次数（历史累计数据）；

* **MASTER_PERCENT** ：RDS主实例处理的读写查询占比（注意该列显示的是累计的实际数据占比，并不是用户配置的百分比）；

* **SLAVE_PERCENT** ：RDS备实例处理的读写查询占比（注意该列显示的是累计的实际数据占比，并不是用户配置的百分比）。



**说明**

* 事务中的只读查询会被发送到RDS主实例；

* 由于`MASTER_PERCENT`，`SLAVE_PERCENT`这两列代表的是历史累计数据，更改读写权重的配比后，这几个数值并不能立即反应最新的读写权重配比，需累计一段比较长的时间才行。



