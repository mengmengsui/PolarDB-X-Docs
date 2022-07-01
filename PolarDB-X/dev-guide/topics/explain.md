EXPLAIN 
============================

该语句用于解释SQL语句的执行计划，包括SELECT、DELETE、INSERT、REPLACE或UPDATE语句。

语法 
-----------------------

获取SQL计划信息：

```sql
EXPLAIN
{LOGICALVIEW | LOGIC | SIMPLE | DETAIL | EXECUTE | PHYSICAL | OPTIMIZER | SHARDING
 | COST | ANALYZE | BASELINE | JSON_PLAN | ADVISOR} 
 {SELECT statement | DELETE statement | INSERT statement | REPLACE statement| UPDATE statement}
```



示例 
-----------------------

* explain语句：展示基本的SQL执行计划，该执行计划是算子组成，主要体现SQL在CN上的整个执行过程。

  ```sql
  mysql> explain select count(*) from lineitem group by L_ORDERKEY;
  +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | LOGICAL EXECUTIONPLAN                                                                                                                                                              |
  +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Project(count(*)="count(*)")                                                                                                                                                       |
  |   HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))")                                                                                                                            |
  |     Gather(concurrent=true)                                                                                                                                                        |
  |       LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_ORDERKEY`") |
  | HitCache:false                                                                                                                                                                     |                                                                                                                                                               |
  | TemplateId: 5819c807                                                                                                                                                               |
  +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  ```

  其中，`HitCache`标记该查询是否命中`PlanCache`，取值为`false` or `true`；`TemplateId`表示对该计划的标识，具有全局唯一性。

* explain logicalview语句：展示LogicalView所表示的下推SQL在DN上的执行计划。

  ```sql
  mysql> explain LOGICALVIEW select  mysql> explain select logialview count(*) from lineitem group by L_ORDERKEY;
  +----------------------------------------------------------+
  | LOGICAL EXECUTIONPLAN                                    |
  +----------------------------------------------------------+
  | Project(count(*)="count(*)")                             |
  |   HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))")  |
  |     Gather(concurrent=true)                              |
  |       LogicalView                                        |
  |         MysqlAgg(group="L_ORDERKEY", count(*)="COUNT()") |
  |           MysqlTableScan(name=[ads, lineitem])           |
  | HitCache:true                                            |
  | Source:PLAN_CACHE                                        |
  | TemplateId: 5819c807
  ```

  

* explain execute语句：表示下推SQL在mysql的执行情况，这个语句和mysql的explain语句同义。通过该语句可以查看下推SQL在DN上有没有使用索引，有没有做全表扫描。

  ```sql
  mysql> explain EXECUTE  select  count(*) from lineitem group by L_ORDERKEY;
  +----+-------------+----------+------------+-------+---------------+---------+---------+-----+------+----------+----------------------------------------------+
  | id | select_type | table    | partitions | type  | possible_keys | key     | key_len | ref | rows | filtered | Extra                                        |
  +----+-------------+----------+------------+-------+---------------+---------+---------+-----+------+----------+----------------------------------------------+
  | 1  | SIMPLE      | lineitem | NULL       | index | PRIMARY       | PRIMARY | 8       | NULL | 1    | 100      | Using index; Using temporary; Using filesort |
  +----+-------------+----------+------------+-------+---------------+---------+---------+-----+------+----------+----------------------------------------------+
  1 row in set (0.24 sec)
  ```

  

* explain sharding语句：展示当前查询在DN上扫描的物理分片情况。

  ```sql
  mysql> explain sharding  select  count(*) from lineitem group by L_ORDERKEY;
  +---------------+----------------------------------+-------------+-----------+-----------+
  | LOGICAL_TABLE | SHARDING                         | SHARD_COUNT | BROADCAST | CONDITION |
  +---------------+----------------------------------+-------------+-----------+-----------+
  | lineitem      | [000000-000003].lineitem_[00-15] | 16          | false     |           |
  +---------------+----------------------------------+-------------+-----------+-----------+
  1 row in set (0.04 sec)
  ```

  

* explain cost语句：相对于explain语句，除了展示执行计划以外，还会显示各个算子基于统计信息估算的代价，以及这条查询被优化器识别的WORKLOAD。

  ```sql
  mysql> explain COST  select  count(*) from lineitem group by L_ORDERKEY;
  +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | LOGICAL EXECUTIONPLAN                                                                                                                                                                                                                                                                                            |
  +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Project(count(*)="count(*)"): rowcount = 2508.0, cumulative cost = value = 2.4867663E7, cpu = 112574.0, memory = 88984.0, io = 201.0, net = 4.75, id = 182                                                                                                                                                       |
  |   HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))"): rowcount = 2508.0, cumulative cost = value = 2.4867662E7, cpu = 112573.0, memory = 88984.0, io = 201.0, net = 4.75, id = 180                                                                                                                            |
  |     Gather(concurrent=true): rowcount = 2508.0, cumulative cost = value = 2.4860069E7, cpu = 105039.0, memory = 29796.0, io = 201.0, net = 4.75, id = 178                                                                                                                                                        |
  |       LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_ORDERKEY`"): rowcount = 2508.0, cumulative cost = value = 2.4860068E7, cpu = 105038.0, memory = 29796.0, io = 201.0, net = 4.75, id = 109 |
  | HitCache:true                                                                                                                                                                                                                                                                                                    |
  | Source:PLAN_CACHE                                                                                                                                                                                                                                                                                                |
  | WorkloadType: TP                                                                                                                                                                                                                                                                                                 |
  | TemplateId: 5819c807
  ```

  

* explain analyze语句：相对于explain cost语句，除了显示各个算子基于统计信息估算的代价以外，该语句可以收集真实运行过程中算子输出的rowCount等信息。

  ```sql
  mysql> explain ANALYZE  select  count(*) from lineitem group by L_ORDERKEY;
  +----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | LOGICAL EXECUTIONPLAN                                                                                                                                                                                                                                                                                                                                                                                    |
  +----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Project(count(*)="count(*)"): rowcount = 2508.0, cumulative cost = value = 2.4867663E7, cpu = 112574.0, memory = 88984.0, io = 201.0, net = 4.75, actual time = 0.001 + 0.000, actual rowcount = 2506, actual memory = 0, instances = 1, id = 182                                                                                                                                                        |
  |   HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))"): rowcount = 2508.0, cumulative cost = value = 2.4867662E7, cpu = 112573.0, memory = 88984.0, io = 201.0, net = 4.75, actual time = 0.000 + 0.000, actual rowcount = 2506, actual memory = 0, instances = 1, id = 180                                                                                                                             |
  |     Gather(concurrent=true): rowcount = 2508.0, cumulative cost = value = 2.4860069E7, cpu = 105039.0, memory = 29796.0, io = 201.0, net = 4.75, actual time = 0.000 + 0.000, actual rowcount = 0, actual memory = 0, instances = 0, id = 178                                                                                                                                                            |
  |       LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_ORDERKEY`"): rowcount = 2508.0, cumulative cost = value = 2.4860068E7, cpu = 105038.0, memory = 29796.0, io = 201.0, net = 4.75, actual time = 0.030 + 0.025, actual rowcount = 10000, actual memory = 0, instances = 0, id = 109 |
  | HitCache:true                                                                                                                                                                                                                                                                                                                                                                                            |
  | Source:PLAN_CACHE                                                                                                                                                                                                                                                                                                                                                                                        |
  | TemplateId: 5819c807                                                                                                                                                                                                                                                                                                                                                                                     |
  +----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  7 rows in set (1.08 sec)
  ```

  

* explain physical语句：展示查询在运行过程中执行模式、各个执行片段（Fragment）的依赖关系和并行度。该查询被识别为单机单线程计划模式（TP_LOCAL），执行计划被分为三个片段Fragment-0、Fragment-1和Fragment-2，先做预聚合再做最终的聚合计算，每个片段的执行度可以不同。

  ```sql
  mysql> explain physical   select  count(*) from lineitem group by L_ORDERKEY;
  +--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | PLAN                                                                                                                                                                           |
  +--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | ExecutorMode: TP_LOCAL                                                                                                                                                         |
  | Fragment 0 dependency: [] parallelism: 4                                                                                                                                       |
  | Gather(concurrent=true)                                                                                                                                                        |
  |   LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_ORDERKEY`") |
  | Fragment 1 dependency: [] parallelism: 8                                                                                                                                       |
  | LocalBuffer                                                                                                                                                                    |
  |   RemoteSource(sourceFragmentIds=[0], type=RecordType(INTEGER L_ORDERKEY, BIGINT count(*)))                                                                                    |
  | Fragment 2 dependency: [0, 1] parallelism: 8                                                                                                                                   |
  | Project(count(*)="count(*)")                                                                                                                                                   |
  |   HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))")                                                                                                                        |
  |     RemoteSource(sourceFragmentIds=[1], type=RecordType(INTEGER L_ORDERKEY, BIGINT count(*)))                                                                                  |
  +--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  11 rows in set (0.10 sec)
  ```

  

* explain advisor语句：主要是会基于统计信息，分析当前查询的执行计划，给用户推荐可以加速查询的全局二级索引。

  ```sql
  mysql> explain advisor   select  count(*) from lineitem group by L_ORDERKEY \G;
  *************************** 1. row ***************************
  IMPROVE_VALUE: 4.4%
    IMPROVE_CPU: 340.8%
    IMPROVE_MEM: 0.0%
     IMPROVE_IO: 1910.0%
    IMPROVE_NET: 0.0%
   BEFORE_VALUE: 2.48676627E7
     BEFORE_CPU: 112573.7
     BEFORE_MEM: 88983.8
      BEFORE_IO: 201
     BEFORE_NET: 4.7
    AFTER_VALUE: 2.38256249E7
      AFTER_CPU: 25536
      AFTER_MEM: 88983.8
       AFTER_IO: 10
      AFTER_NET: 4.7
   ADVISE_INDEX: ALTER TABLE `ads`.`lineitem` ADD GLOBAL INDEX `__advise_index_gsi_lineitem_L_ORDERKEY`(`L_ORDERKEY`) DBPARTITION BY HASH(`L_ORDERKEY`) TBPARTITION BY HASH(`L_ORDERKEY`) TBPARTITIONS 4;
       NEW_PLAN:
  Project(count(*)="count(*)")
    HashAgg(group="L_ORDERKEY", count(*)="SUM(count(*))")
      Gather(concurrent=true)
        IndexScan(tables="[000000-000003].lineitem__what_if_gsi_L_ORDERKEY_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, COUNT(*) AS `count(*)` FROM `lineitem__what_if_gsi_L_ORDERKEY` AS `lineitem__what_if_gsi_L_ORDERKEY` GROUP BY `L_ORDERKEY`")
           INFO: GLOBAL_INDEX
  1 row in set (0.13 sec)
  ```

   



