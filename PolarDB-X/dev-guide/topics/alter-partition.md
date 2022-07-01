变更表类型及分区策略
===============================

PolarDB-X支持变更表的类型（即在单表、广播表和分区表三者间进行相互转换），和变更分区表的分区策略（包括分区函数或分区列）。本文介绍相关语法和示例。

前提条件 
-------------------------

* 仅适用于分区模式为 partitioning 的逻辑库（请参见[CREATE DATABASE](../../dev-guide/topics/create-database2.md)）

* 仅内核小版本为5.4.13或以上的PolarDB-X实例支持变更表的类型，和变更表的分区策略。



### 如何查看实例版本
您可以先连接PolarDB-X 2.0数据库，再通过执行如下 `select version()` 命令查看目标实例的当前版本：
```sql
mysql> select version();
+-----------------------------+
| VERSION()                   |
+-----------------------------+
| 5.6.29-TDDL-5.4.8-16079964  |
+-----------------------------+
1 row in set (0.00 sec)         
```


注意事项 
-------------------------

* 表属性变更后，主键分区表将变成普通表（即不再适用原主键分区表中的自动分区策略或索引转换规则）。更多详情，请参见[主键拆分](sharding-by-pk.md)。


* 本文中关于变更分区表、广播表和单表的表类型示例，均在单表`t_order`的基础上进行变更，`t_order`表的创建语句如下：

  ```sql
  CREATE TABLE t_order (
    `id` bigint(11) NOT NULL AUTO_INCREMENT BY GROUP,
    `order_id` varchar(20) DEFAULT NULL,
    `buyer_id` varchar(20) DEFAULT NULL,
    `seller_id` varchar(20) DEFAULT NULL,
    `order_snapshot` longtext DEFAULT NULL,
    `order_detail` longtext DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `l_i_order` (`order_id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  ```

表类型 
------------------------

PolarDB-X实例支持3种类型的表：分区表、广播表和单表。您可以通过ALTER TABLE语句在分区表、广播表和单表间进行转换，同时还能对分区表的分区策略进行变更。


* **分区表**

  使用`partition_options`分区子句进行创建的表。

  `partition_options`可以是如下子句：

  ```sql
  partition_options:
    PARTITION BY
          HASH({column_name | partition_func(column_name)})
        | KEY(column_list)
        | RANGE{({column_name | partition_func(column_name)}) 
        | RANGE COLUMNS(column_list)}
        | LIST{({column_name | partition_func(column_name)}) 
        | LIST COLUMNS(column_list)} }
    partition_list_spec
  ```
  
  **说明** 更多关于分区策略的信息，请参见[分区表](create-table2.md) 。


* **广播表**

  通过`BROADCAST`子句创建的表，系统会将该表复制到每个分库上，并通过分布式事务实现数据一致性。更多详情，请参见[单表与广播表](create-table2.md)。
  
* **单表**

  通过`SINGLE`子句的表。更多详情，请参见[单表与广播表](create-table2.md)。



单表或广播表变为分区表 
--------------------------------

* **语法**

  ```sql
  ALTER TABLE table_name partition_options;
  ```

  
  **说明**
  * 更多关于`partition_options`的信息，请参见[CREATE TABLE](create-table2.md)。
  
  

  
  
* **示例**

  因业务扩展，单表`t_order`无法承载日益增长的数据。此时，您可以使用如下语句将该单表变更为分区表（以`order_id`为分区键，采用KEY分区策略）：

  ```sql
  ALTER TABLE t_order PARTITION BY KEY(`order_id`);
  ```
  
  如需指定分区数量，可以使用如下语句：

  ```sql
  ALTER TABLE t_order PARTITION BY KEY(`order_id`) PARTITIONS 8;
  ```




单表或分区表变为广播表 
--------------------------------

* **语法**

  ```sql
  ALTER TABLE table_name BROADCAST;
  ```
  
* **示例**

  您可以使用如下语句将单表或拆分表`t_order`变更为广播表：

  ```sql
  ALTER TABLE t_order BROADCAST;
  ```



  广播表或分区表变为单表 
--------------------------------

* **语法**

  ```sql
  ALTER TABLE table_name SINGLE;
  ```
  或者
  ```sql
  ALTER TABLE table_name REMOVE PARTITIONING;
  ```
  
* **示例**

  您可以使用如下语句将广播表或拆分表`t_order`变更为单表：

  ```sql
  ALTER TABLE t_order SINGLE;
  ```

  或者
  ```sql
  ALTER TABLE t_order REMOVE PARTITIONING;
  ```



变更分区表的分区策略 
-------------------------------

* **语法**

  ```sql
  ALTER TABLE tbl_name drds_partition_options;
  ```

  

* **示例1**

  假设已使用如下语句在PolarDB-X数据库中创建了一张分区表`t_order`（根据`order_id`列进行KEY分区）：

  ```sql
  CREATE TABLE t_order (
    `id` bigint(11) NOT NULL AUTO_INCREMENT,
    `order_id` varchar(20) DEFAULT NULL,
    `buyer_id` varchar(20) DEFAULT NULL,
    `seller_id` varchar(20) DEFAULT NULL,
    `order_snapshot` longtext DEFAULT NULL,
    `order_detail` longtext DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `l_i_order` (`order_id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  PARTITION BY KEY(`order_id`);
  ```

  

  现需要对`t_order`表的分区策略作出如下变更：
  * 根据`order_id`列以及`buyer_id`进行KEY分区。
  
  * 总共包含8个分区。
  

  

  您可以使用如下语句实现上述变更：

  ```sql
  ALTER TABLE t_order PARTITION BY KEY(order_id, buyer_id) PARTITIONS 8;
  ```

* **示例2**

  假设已使用如下语句在PolarDB-X数据库中创建了一张分区表`t_order`（根据`id`列进行RANGE分区）：

  ```sql
  CREATE TABLE t_order (
    `id` bigint(11) NOT NULL AUTO_INCREMENT,
    `order_id` varchar(20) DEFAULT NULL,
    `buyer_id` varchar(20) DEFAULT NULL,
    `seller_id` varchar(20) DEFAULT NULL,
    `order_snapshot` longtext DEFAULT NULL,
    `order_detail` longtext DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `l_i_order` (`order_id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  PARTITION BY RANGE(`id`)
  (
    PARTITION p1 VALUES LESS THAN (100),
    PARTITION p2 VALUES LESS THAN (1000),
    PARTITION P3 VALUES LESS THAN MAXVALUE
  );
  ```

  

  现需要对`t_order`表的分区策略作出如下变更：
  * 根据`order_id`列以及`buyer_id`进行KEY分区。
  
  * 总共包含16个分区。
  

  您可以使用如下语句实现上述变更：

  ```sql
  ALTER TABLE t_order PARTITION BY KEY(order_id, buyer_id) PARTITIONS 16;
  ```

相关文档 
-------------------------

分区策略变更后，您可以通过如下命令查看表的分区策略或拓扑结构：

* 查看表分区规则，请使用 SHOW CREATE TABLE tablename。

* 查看表拓扑结构，请参见[SHOW TOPOLOGY FROM tablename](show-topology.md)。




常见问题 
-------------------------

Q：为什么有时拆分键变更的DDL任务会执行失败？此时该如何处理？

A：实例崩溃或唯一索引存在冲突等因素会导致拆分规则变更的DDL任务执行失败。但这不会损坏原表任何数据，也不会阻塞正常的DML和查询语句执行。当拆分键变更的DDL任务执行失败时，您可以通过`CANCEL DDL`命令回滚该任务，然后再次尝试变更。关于`CANCEL DDL`命令的详情，请参见[回滚任务](cancel-ddl.md)。