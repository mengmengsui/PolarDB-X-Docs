执行计划介绍 
===========================

本文介绍如何使用查询执行计划并介绍一些基本的算子含义和实现。

算子介绍 
-------------------------



|               含义                |                                           物理算子                                            |
|---------------------------------|-------------------------------------------------------------------------------------------|
| 下发给DN的算子                        | LogicalView，LogicalModifyView，PhyTableOperation, IndexScan                                |
| 连接（Join）                        | BKAJoin，NLJoin，HashJoin，SortMergeJoin，HashSemiJoin，SortMergeSemiJoin，MaterializedSemiJoin |
| 排序                              | MemSort，TopN， MergeSort                                                                   |
| 聚合（Group By）                    | HashAgg，SortAgg                                                                           |
| 数据重分布或者聚合                       | Exchange Gather                                                                           |
| 过滤                              | Filter                                                                                    |
| 投影                              | Project                                                                                   |
| 求并集                             | Union                                                                                     |
| 设置结果集输出行数(Limit/Offset...Fetch) | Limit                                                                                     |
| 窗口函数                            | OverWindow                                                                                |



**LogicalView** 

LogicalView是从存储层MySQL数据源拉取数据的算子，类似于其他数据库中的TableScan或IndexScan，但支持更多的下推。LogicalView中包含下推的SQL语句和数据源信息，更像一个视图。其中下推的SQL可能包含多种算子，如Project、Filter、聚合、排序、Join和子查询等。下述示例为您展示EXPLAIN中LogicalView的输出信息及其含义：

```sql
mysql> explain select * From sbtest1 where id > 1000;
Gather(concurrent=true)
   LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?)")
```

LogicalView 的信息由三部分构成：

* tables：存储层MySQL对应的表名，以英文句号（.）分割，英文句号（.）之前是分库对应的编号，之后是表名及其编号，如[000-127]表示表名编号从000到127的所有表。

* shardCount：需访问的分表总数，该示例会访问从000到127共计128张分表。

* sql：下发至存储层MySQL的SQL模版，PolarDB-X在执行时会将表名替换为物理表名，参数化的常量问号（?）替换成实际参数，详情请参见[执行计划管理](spm.md)。




**IndexScan** 

IndexScan和LogicalView一样也是表示从存储层MySQL数据源拉取数据的算子，扫描的是索引表。下述示例为您展示EXPLAIN中IndexScan的输出信息及其含义：

```sql
mysql> explain select * from sequence_one_base where integer_test=1;
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexScan(tables="DRDS_POLARX1_QATEST_APP_000000_GROUP.gsi_sequence_one_index_3a0A_01", sql="SELECT `pk`, `integer_test`, `varchar_test`, `char_test`, `blob_test`, `tinyint_test`, `tinyint_1bit_test`, `smallint_test`, `mediumint_test`, `bit_test`, `bigint_test`, `float_test`, `double_test`, `decimal_test`, `date_test`, `time_test`, `datetime_test`, `timestamp_test`, `year_test`, `mediumtext_test` FROM `gsi_dml_sequence_one_index_index1` AS `gsi_dml_sequence_one_index_index1` WHERE (`integer_test` = ?)") |
```

作为谓词条件对索引表做裁剪，所以实际上我们会去生成IndexScan算子，表示只会扫描gsi_sequence_one_index索引表一个分片。上述SQL正常应该扫描sequence_one_base表，由于integer_test不是分区键，需要扫描sequence_one_base所有的分片。但是由于sequence_one_base表在integer_test列上存在全局二级索引gsi_sequence_one_index，将

**Gather** 

Gather将多份数据合并成同份数据。上面的例子中，Gather将各个分表上查询到的数据合并成一份。Gather通常出现在LogicalView上方，表示收集合并各个分表的数据。

**Exchange** 

Exchange是一个逻辑算子，本身不对计算过程中的数据做计算，只是将输入的数据做重分布后，输出给下游算子。一般重分布策略分为

* SINGLETON: 将上游多份数据进行合并输出，这种重分布策略等价于Gather

* HASH_DISTRIBUTED: 将上游输入的数据按照某些列做repartition，常见于包含Join和Agg的执行计划中。

* BROADCAST_DISTRIBUTED: 将上游相同一份数据分发成多份，广播给下游多个节点，主要应用于MPP执行计划中。




**MergeSort** 

MergeSort即归并排序算子，表示将有序的数据流进行归并排序，合并成一个有序的数据流。例如：

```sql
mysql> explain select * from sbtest1 where id > 1000 order by id limit 5,10; 
MergeSort(sort="id ASC", offset=?1, fetch=?2)   
   LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?) ORDER BY `id` LIMIT (? + ?)")
```

MergeSort算子包含三部分内容：

* sort：表示排序字段以及排列顺序，id ASC表示按照ID字段递增排序，DESC表示递减排序。

* offset：表示获取结果集时的偏移量，例子中被参数化了，实际值为5。

* fetch：表示最多返回的数据行数。与offset类似，同样是参数化的表示，实际对应的值为10。




**Project** 

Project表示投影操作，即从输入数据中选择部分列输出，或者对某些列进行转换（通过函数或者表达式计算）后输出，当然也可以包含常量。

```sql
mysql> explain select '你好, DRDS', 1 / 2, CURTIME(); 
Project(你好, DRDS="_UTF-16'你好, DRDS'", 1 / 2="1 / 2", CURTIME()="CURTIME()")
```

Project的计划中包括每列的列名及其对应的列、值、函数或者表达式。

**Filter** 

Filter表示过滤操作，其中包含一些过滤条件。该算子对输入数据进行过滤，若满足条件，则输出，否则丢弃。如下是一个较复杂的例子，包含了以上介绍的大部分算子。

```sql
mysql> explain select k, avg(id) avg_id from sbtest1 where id > 1000 group by k having avg_id > 1300;
Filter(condition="avg_id > ?1")
  Project(k="k", avg_id="sum_pushed_sum / sum_pushed_count")
    SortAgg(group="k", sum_pushed_sum="SUM(pushed_sum)", sum_pushed_count="SUM(pushed_count)")
      MergeSort(sort="k ASC")
        LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT `k`, SUM(`id`) AS `pushed_sum`, COUNT(`id`) AS `pushed_count` FROM `sbtest1` WHERE (`id` > ?) GROUP BY `k` ORDER BY `k`")
```

WHERE id &gt; 1000中的条件没有对应的Filter算子，是因为这个算子最终被下推到了LogicalView中，可以在LogicalView的SQL中看到WHERE (id &gt; ?) 。

**Union All与Union Distinct** 

顾名思义，Union All对应UNIONALL，Union Distinct对应UNIONDISTINCT。该算子通常有2个或更多个输入，表示将多个输入的数据合并在一起。例如：

```sql
mysql> explain select * From sbtest1 where id > 1000 union distinct select * From sbtest1 where id < 200;
UnionDistinct(concurrent=true)
  Gather(concurrent=true)
    LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` > ?)")
  Gather(concurrent=true)
    LogicalView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="SELECT * FROM `sbtest1` WHERE (`id` < ?)")
```



**LogicalModifyView** 

LogicalView表示从底层数据源获取数据的算子，与之对应的，LogicalModifyView表示对底层数据源的修改算子，其中也会记录一个SQL语句，该SQL可能是INSERT、UPDATE或者DELETE。

```sql
mysql> explain update sbtest1 set c='Hello, DRDS' where id > 1000;
LogicalModifyView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="UPDATE `sbtest1` SET `c` = ? WHERE (`id` > ?)"
```



```sql
mysql> explain delete from sbtest1 where id > 1000;
LogicalModifyView(tables="[0000-0031].sbtest1_[000-127]", shardCount=128, sql="DELETE FROM `sbtest1` WHERE (`id` > ?)")
```

LogicalModifyView查询计划的内容与LogicalView类似，包括下发的物理分表，分表数以及SQL模版。同样，由于开启了执行计划缓存，对SQL做了参数化处理，SQL模版中的常量会用?替换。

**PhyTableOperation** 

PhyTableOperation表示对某个物理分表直接执行一个操作。

**说明** 通常情况下，该算子仅用于INSERT语句。但当路由分发分到一个分片时，该算子也会出现在SELECT语句中。

```sql
mysql> explain insert into sbtest1 values(1, 1, '1', '1'),(2, 2, '2', '2');
PhyTableOperation(tables="SYSBENCH_CORONADB_1526954857179TGMMSYSBENCH_CORONADB_VGOC_0000_RDS.[sbtest1_001]", sql="INSERT INTO ? (`id`, `k`, `c`, `pad`) VALUES(?, ?, ?, ?)", params="`sbtest1_001`,1,1,1,1")
PhyTableOperation(tables="SYSBENCH_CORONADB_1526954857179TGMMSYSBENCH_CORONADB_VGOC_0000_RDS.[sbtest1_002]", sql="INSERT INTO ? (`id`, `k`, `c`, `pad`) VALUES(?, ?, ?, ?)", params="`sbtest1_002`,2,2,2,2")
```

示例中，INSERT插入两行数据，每行数据对应一个PhyTableOperation算子。PhyTableOperation算子的内容包括三部分：

* tables：物理表名，仅有唯一一个物理表名。

* sql：SQL模版，该SQL模版中表名和常量均被参数化，用?替换，对应的参数在随后的params中给出。

* params：SQL模版对应的参数，包括表名和常量。




执行计划介绍 
---------------------------

一条SQL进入到PolarDB-X分布式数据库后，经过解析优化，会生成一个可运行的执行计划。该执行计划是按照算子执行过程中的依赖关系组成。一般通过执行计划，可以窥探SQL在数据库内部是如何高效运行的。为了方便理解，这里罗列几个例子。

**示例1** 



```sql
mysql> explain select count(*) from lineitem group by L_LINESTATUS;
|   HashAgg(group="L_LINESTATUS", count(*)="SUM(count(*))")                                                                                                                              |
|     Exchange(distribution=hash[0], collation=[])                                                                                                                                       |
|       LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_LINESTATUS`, COUNT(*) AS `count(*)` FROM `lineitem` AS `lineitem` GROUP BY `L_LINESTATUS`")
```

Exchange: 汇总LogicalView返回的数据，按照L_LINESTATUS字段做重分布式，输出给下游算子；由于group by 的列和表lineitem分区键不对齐，group by是没法完全下推给DN执行。所以group by会拆分成两阶段，将partition agg下推给DN，先做部分聚合；然后在CN层将数据做重分布式，再做一次最终的聚合，输出结果。

**示例2** 



```sql
mysql> explain select * from lineitem, orders where L_ORDERKEY= O_ORDERKEY;
                                                                                                                                                                                                                                                                                                                                          |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| HashJoin(condition="O_ORDERKEY = L_ORDERKEY", type="inner")                                                                                                                                                                                                                                                                                                      |
|   Exchange(distribution=hash[0], collation=[])                                                                                                                                                                                                                                                                                                                   |
|     LogicalView(tables="[000000-000003].lineitem_[00-15]", shardCount=16, sql="SELECT `L_ORDERKEY`, `L_PARTKEY`, `L_SUPPKEY`, `L_LINENUMBER`, `L_QUANTITY`, `L_EXTENDEDPRICE`, `L_DISCOUNT`, `L_TAX`, `L_RETURNFLAG`, `L_LINESTATUS`, `L_SHIPDATE`, `L_COMMITDATE`, `L_RECEIPTDATE`, `L_SHIPINSTRUCT`, `L_SHIPMODE`, `L_COMMENT` FROM `lineitem` AS `lineitem`") |
|   Exchange(distribution=hash[0], collation=[])                                                                                                                                                                                                                                                                                                                   |
|     LogicalView(tables="[000000-000003].orders_[00-15]", shardCount=16, sql="SELECT `O_ORDERKEY`, `O_CUSTKEY`, `O_ORDERSTATUS`, `O_TOTALPRICE`, `O_ORDERDATE`, `O_ORDERPRIORITY`, `O_CLERK`, `O_SHIPPRIORITY`, `O_COMMENT` FROM `orders` AS `orders`")
```

示例2是典型的两张表做关联(join)，由于两表分区键没对齐，所有join没有下推，整个执行是将两个表数据都扫描出来，在CN层做关联计算。

* LogicalView: 扫描表数据。

* Exchange: 汇总LogicalView返回的数据，分布按照关联条件的列做重分布式，输出给下游Join算子。

* HashJoin: 接受两边输入，通过构建HashTable的方式来计算关联结果。




**示例3** 



```sql
mysql> explain select * from lineitem, orders where L_LINENUMBER= O_ORDERKEY;
| Gather(concurrent=true)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|   LogicalView(tables="[000000-000003].lineitem_[00-15],orders_[00-15]", shardCount=16, sql="SELECT `lineitem`.`L_ORDERKEY`, `lineitem`.`L_PARTKEY`, `lineitem`.`L_SUPPKEY`, `lineitem`.`L_LINENUMBER`, `lineitem`.`L_QUANTITY`, `lineitem`.`L_EXTENDEDPRICE`, `lineitem`.`L_DISCOUNT`, `lineitem`.`L_TAX`, `lineitem`.`L_RETURNFLAG`, `lineitem`.`L_LINESTATUS`, `lineitem`.`L_SHIPDATE`, `lineitem`.`L_COMMITDATE`, `lineitem`.`L_RECEIPTDATE`, `lineitem`.`L_SHIPINSTRUCT`, `lineitem`.`L_SHIPMODE`, `lineitem`.`L_COMMENT`, `orders`.`O_ORDERKEY`, `orders`.`O_CUSTKEY`, `orders`.`O_ORDERSTATUS`, `orders`.`O_TOTALPRICE`, `orders`.`O_ORDERDATE`, `orders`.`O_ORDERPRIORITY`, `orders`.`O_CLERK`, `orders`.`O_SHIPPRIORITY`, `orders`.`O_COMMENT` FROM `lineitem` AS `lineitem` INNER JOIN `orders` AS `orders` ON (`lineitem`.`L_LINENUMBER` = `orders`.`O_ORDERKEY`)") |
```

示例3也是典型的两张表做关联，由于两表分区键对齐，所有join下推到各个分片的DN来执行，上层的CN节点只需要通过Gather算子将DN返回的结果汇总输出。

**示例4** 



```sql
mysql> explain select * from gsi_dml_unique_multi_index_base where integer_test=1;                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Project(pk="pk", integer_test="integer_test", varchar_test="varchar_test", char_test="char_test", blob_test="blob_test", tinyint_test="tinyint_test", tinyint_1bit_test="tinyint_1bit_test", smallint_test="smallint_test", mediumint_test="mediumint_test", bit_test="bit_test", bigint_test="bigint_test", float_test="float_test", double_test="double_test", decimal_test="decimal_test", date_test="date_test", time_test="time_test", datetime_test="datetime_test", timestamp_test="timestamp_test", year_test="year_test", mediumtext_test="mediumtext_test") |
|   BKAJoin(condition="pk = pk", type="inner")                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|     IndexScan(tables="DRDS_POLARX1_QATEST_APP_000000_GROUP.gsi_dml_unique_multi_index_index1_a0ol_01", sql="SELECT `pk`, `integer_test`, `varchar_test`, `char_test`, `bit_test`, `bigint_test`, `double_test`, `date_test` FROM `gsi_dml_unique_multi_index_index1` AS `gsi_dml_unique_multi_index_index1` WHERE (`integer_test` = ?)")                                                                                                                                                                                                                              |
|     Gather(concurrent=true)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|       LogicalView(tables="[000000-000003].gsi_dml_unique_multi_index_base_[00-15]", shardCount=16, sql="SELECT `pk`, `blob_test`, `tinyint_test`, `tinyint_1bit_test`, `smallint_test`, `mediumint_test`, `float_test`, `decimal_test`, `time_test`, `datetime_test`, `timestamp_test`, `year_test`, `mediumtext_test` FROM `gsi_dml_unique_multi_index_base` AS `gsi_dml_unique_multi_index_base` WHERE ((`integer_test` = ?) AND (`pk` IN (...)))")                                                                                                                 |
| HitCache:true
```

这个例子很有意思，SQL本身只是带有谓词的简单查询，结果从执行计划看是两表做关联(BKAJoin)。主要是gsi_dml_unique_multi_index_base在列上integer_test有全局二级索引，命中索引可以减少扫描代价，但这个索引并不是覆盖索引，所以需要有回表操作。

* IndexScan: 根据integer_test=1扫描出索引表gsi_dml_unique_multi_index_index1_a0ol_01数据。

* BKAJoin: 收集IndexScan的结果，通过该算子和主表gsi_dml_unique_multi_index_base做回表关联，获取其他列值。


关于BKAJoin更多的介绍可以查看[《分布式数据库如何实现 Join》](https://zhuanlan.zhihu.com/p/363151441) 。

**说明** 通常情况下，通过查询执行计划，可以查看到是否命中了全局二级索引等信息。但是对于下推部分的SQL，还可以通过explain execute 指令，获取物理SQL在DN上的执行情况，比如是否命中了DN的局部索引。
