如何使用全局二级索引 
===============================

PolarDB-X支持全局二级索引，本文将在分库分表语法下介绍如何创建、使用全局二级索引功能。

使用限制 
-------------------------

如果在分区表下，该文档依然试用，只不过创建语法需要参考[CREATE INDEX](create-index2.md)。


创建GSI 
--------------------------

PolarDB-X对MySQL DDL语法进行了扩展，增加定义GSI的语法。使用方式与在MySQL上创建索引一致。

* 建表时定义GSI

  ![定义GSI](../images/p335107.png)

* 建表后添加GSI

  ![添加GSI](../images/p335110.png)



**说明**

* 索引名：作为索引表的名字，用于创建索引表。

* 索引列：索引表的分库分表键，即索引分库分表子句中用到的所有列。

* 覆盖列：索引表中的其他列，默认包含主键和主表的全部分库分表键。

* 索引分库分表子句：索引表的分库分表算法，与CREATE TABLE中分库分表子句的语法一致。

* 上述是在分库分表下的创建GSI语法，如果是在分区表下GSI语法只需要可以参考[CREATE INDEX](create-index2.md)。




示例：

```sql
# 建表时定义GSI
CREATE TABLE t_order (
 `id` bigint(11) NOT NULL AUTO_INCREMENT,
 `order_id` varchar(20) DEFAULT NULL,
 `buyer_id` varchar(20) DEFAULT NULL,
 `seller_id` varchar(20) DEFAULT NULL,
 `order_snapshot` longtext DEFAULT NULL,
 `order_detail` longtext DEFAULT NULL,
 PRIMARY KEY (`id`),
 GLOBAL INDEX `g_i_seller`(`seller_id`) COVERING (`id`, `order_id`, `buyer_id`, `order_snapshot`) dbpartition by hash(`seller_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
# 添加GSI
CREATE UNIQUE GLOBAL INDEX `g_i_buyer` ON `t_order`(`buyer_id`) 
    COVERING(`seller_id`, `order_snapshot`) 
    dbpartition by hash(`buyer_id`) tbpartition by hash(`buyer_id`) tbpartitions 3
```



使用GSI 
--------------------------

GSI创建完成后，可以通过如下方式指定查询使用的索引表：

* 通过HINT指定索引您可以选择以下两种HINT语句中的任意一种指定使用目标索引进行查询。

* 方式一

```sql
FORCE INDEX({index_name})
```

示例：

```sql
SELECT a.*, b.order_id 
FROM t_seller a 
JOIN t_order b FORCE INDEX(g_i_seller) ON a.seller_id = b.seller_id 
WHERE a.seller_nick="abc";
```


- 方式二
  
```sql
/*+TDDL:INDEX({table_name/table_alias}, {index_name})*/
```

示例：

```sql
/*+TDDL:index(a, g_i_buyer)*/ SELECT * FROM t_order a WHERE a.buyer_id = 123
```

​    

  

**说明**：如果查询需要使用索引中未包含的列，则首先查询索引表取得所有记录的主键和主表分库分表键，然后回查主表中取得缺少列的值，详细说明请参见。

* 直接查询索引表如果索引表中包含了查询需要的所有列，可以直接查询索引表获得结果。

  

* 索引选择对于带有全局二级索引的主表查询，PolarDB-X会自动选择出优化器认为代价最低的索引表（目前只支持覆盖索引选择）。下面SQL查询的主表是t_order，带有seller_id等值过滤条件，同时涉及的id、order_snapshot和seller_id等列被全局二级索引g_i_seller覆盖。选择了覆盖索引g_i_seller既可以不回表，又可以明确减少分表的扫描数目（seller_id是g_i_seller的拆分键）。通过EXPLAIN可以看到PolarDB-X优化器确实选择了g_i_seller。

  ```sql
  EXPLAIN SELECT t_order.id,t_order.order_snapshot FROM t_order WHERE t_order.seller_id = 's1';
  IndexScan(tables="g_i_seller_sfL1_2", sql="SELECT `id`, `order_snapshot` FROM `g_i_seller` AS `g_i_seller` WHERE (`seller_id` = ?)")
  ```

  

* IGNORE INDEX与USE INDEX您可以通过以下HINT指定优化器使用或不使用某些索引。

* 方式一

    ```sql
    IGNORE INDEX({index_name},...)
    ```
  
    示例：

    ```sql
    SELECT t_order.id,t_order.order_snapshot FROM t_order IGNORE INDEX(g_i_seller) WHERE t_order.seller_id = 's1';
    ```

- 方式二  

    ```sql
    USE INDEX({index_name},...)
    ```

    示例：
  
    ```sql
    SELECT t_order.id,t_order.order_snapshot FROM t_order USE INDEX(g_i_seller) WHERE t_order.seller_id = 's1';
    ```

    

注意事项 
-------------------------

* 创建GSI过程的约束
  * 不支持在单表或广播表上创建GSI。
  
  * 不支持在无主键的表上创建GSI。
  
  * 不支持在UNIQUE GSI中通过任何方式使用前缀索引。
  
  * 创建索引表时必须指定索引名。
  
  * 创建索引表时必须指定分库或分库加分表组合的规则，不允许仅指定分表规则或不指定任何拆分规则。
  
  * 索引表的INDEX列必须包含全部拆分键。
  
  * GSI定义子句中，索引列与覆盖列不可重复。
  
  * 索引表默认包含主表的全部主键和拆分键，如果没有显式包含在索引列中，默认添加到覆盖列。
  
  * 对主表中的每个局部索引，如果引用的所有列均包含在索引表中，默认添加该局部索引到索引表。
  
  * 对GSI的每个索引列，如果没有已经存在的索引，默认单独创建一个索引。
  
  * 对包含多个索引列的GSI，默认创建一个联合局部索引，包含所有索引列。
  
  * 索引定义中，索引列的length参数仅用于在索引表拆分键上创建局部索引。
  
  * 建表后创建GSI时，会在GSI创建结束时自动进行数据校验，只有通过校验，创建GSI的DDL语句才能执行成功。
  
  
  
  **说明**：您也可以使用CHECK GLOBAL INDEX对索引数据进行校验或订正。
  
* Alter Table过程的约束

  
  
  |                         语句                         | 是否支持变更主表拆分键 | 是否支持变更主表主键（也即索引表主键） | 是否支持变更本地唯一索引列 | 是否支持变更索引表拆分键 | 是否支持变更Unique Index列 | 是否支持变更Index列 | 是否支持变更Covering列 |
  |----------------------------------------------------|-------------|---------------------|---------------|--------------|---------------------|--------------|-----------------|
  | ADD COLUMN                                         | 无该场景        | 不支持                 | 无该场景          | 无该场景         | 无该场景                | 无该场景         | 无该场景            |
  | ALTER COLUMN SET DEFAULT和ALTER COLUMN DROP DEFAULT | 不支持         | 不支持                 | 支持            | 不支持          | 不支持                 | 不支持          | 不支持             |
  | CHANGE COLUMN                                      | 不支持         | 不支持                 | 支持            | 不支持          | 不支持                 | 不支持          | 不支持             |
  | DROP COLUMN                                        | 不支持         | 不支持                 | 仅当唯一键中只有1列时支持 | 不支持          | 不支持                 | 不支持          | 不支持             |
  | MODIFY COLUMN                                      | 不支持         | 不支持                 | 支持            | 不支持          | 不支持                 | 不支持          | 不支持             |
  
  
  **说明**
  * 考虑到全局二级索引的稳定性和性能情况，目前禁止直接使用DROP COLUMN命令删除全局二级索引中的列。如需删除全局二级索引中的某些列，您可以先使用DROP INDEX删除对应的全局二级索引，再重新创建一个新的二级索引，或提交工单联系技术支持进行删除。
  
  * 以上对列的分类存在重叠（如Index列包含索引表拆分键，Covering列包含主表拆分键、主键以及指定的列），若存在支持情况冲突情况，不支持的优先级高于支持。
  
  

  下表汇总了使用ALTER TABLE语句变更索引的支持情况
  
  | 语句                                                    | 是否支持                                                     |
  | ------------------------------------------------------- | ------------------------------------------------------------ |
  | ALTER TABLE ADD PRIMARY KEY                             | 支持                                                         |
  | ALTER TABLE ADD \[UNIQUE/FULLTEXT/SPATIAL/FOREIGN\] KEY | 支持，您可以同时在主表和索引表上添加局部索引，索引名称不可与GSI重复。 |
  | ALTER TABLE ALTER INDEX index_name {VISIBLE/INVISIBLE}  | 支持，仅在主表执行（禁止变更GSI状态）。                      |
  | ALTER TABLE {DISABLE/ENABLE} KEYS                       | 支持，仅在主表执行（禁止变更GSI状态）。                      |
  | ALTER TABLE DROP PRIMARY KEY                            | 禁止                                                         |
  | ALTER TABLE DROP INDEX                                  | 仅支持删除普通索引或全局二级索引。                           |
  | ALTER TABLE DROP FOREIGN KEY fk_symbol                  | 支持，仅在主表执行。                                         |
  | ALTER TABLE RENAME INDEX                                | 禁止                                                         |
  
  
  **说明** 考虑到全局二级索引的稳定性和性能情况，目前禁止直接使用DROP COLUMN命令重命名全局二级索引。如需修改全局二级索引名，您可以先使用DROP INDEX删除对应的全局二级索引，再重新创建一个新的二级索引，或提交工单联系技术支持进行修改。
  
* Alter GSI Table的约束条件
  * 不支持在索引表上执行DDL、DML语句。
  
  * 不支持带有NODE HINT的DML语句更新主表、索引表。
  
  
  
* 在包含GSI的表上使用其他DDL时的约束

  

  |         语句         | 是否支持 |
  |--------------------|------|
  | DROP TABLE         | 支持   |
  | DROP INDEX         | 支持   |
  | TRUNCATE TABLE     | 不支持  |
  | RENAME TABLE       | 不支持  |
  | ALTER TABLE RENAME | 不支持  |


  **说明**
  * 考虑主表与索引表的数据一致性，目前禁止执行TRUNCATE TABLE语句 。如需清空主表与索引表数据，您可以使用DELETE语句删除对应的数据，或强制使用HINT `/*+TDDL:CMD_EXTRA(TRUNCATE_TABLE_WITH_GSI=TRUE)*/`。

  * 考虑到全局二级索引的稳定性和性能情况，目前禁止直接使用RENAME TABLE或ALTER TABLE RENAME命令重命名全局二级索引。如需修改全局二级索引名，您可以先使用DROP INDEX删除全局二级索引，修改表名后再重新创建新的二级索引，或提交工单联系技术支持进行修改。

  

  

* 在包含GSI的表上使用DML语句时的约束
  * 不支持在索引表上执行DML语句。
  
  * 在主表上执行DML语句的有如下限制：写索引失败后，不允许继续执行其他语句或提交事务。
  
  
  

  ```sql
  CREATE TABLE t_order(
    `id` bigint(11) NOT NULL AUTO_INCREMENT,
    `order_id` varchar(20) DEFAULT NULL,
    `buyer_id` varchar(20) DEFAULT NULL,
    `seller_id` varchar(20) DEFAULT NULL,
    `order_snapshot` longtext DEFAULT NULL,
    `order_detail` longtext DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `l_i_order` (`order_id`),
    GLOBAL INDEX `g_i_seller` (`seller_id`) dbpartition by hash(`seller_id`) tbpartition by hash(`seller_id`),
    GLOBAL UNIQUE INDEX `g_i_buyer` (`buyer_id`) COVERING (order_snapshot) dbpartition by hash(`buyer_id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
  SET DRDS_TRANSACTION_POLICY='XA';
  INSERT INTO t_order(order_id, buyer_id, seller_id) VALUES('order_1', 'buyer_1', 'seller_1');
  # 失败
  INSERT IGNORE INTO t_order(order_id, buyer_id, seller_id) VALUES('order_2', 'buyer_1', 'seller_1');
  # 失败不允许继续执行
  INSERT IGNORE INTO t_order(order_id, buyer_id, seller_id) VALUES('order_2', 'buyer_2', 'seller_2');
  # 失败后不允许提交事务
  COMMIT;
  ```

  



