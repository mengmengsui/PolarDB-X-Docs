调优方法论 
==========================

找出需调优的慢SQL后，先通过EXPLAIN查看执行计划，然后通过如下方法优化SQL：下推更多计算至存储层MySQL，适当增加索引，优化执行计划。

下推更多的计算 
----------------------------

PolarDB-X会尽可能将更多的计算下推到存储层MySQL。下推计算能够减少数据传输，减少网络层和PolarDB-X层的开销，提升SQL语句的执行效率。PolarDB-X支持下推几乎所有算子，包括：

* 过滤条件，如WHERE或HAVING中的条件。

* 聚合算子，如COUNT，GROUP BY等，会分成两阶段进行聚合计算。

* 排序算子，如ORDER BY。

* JOIN和子查询，两边JOIN Key分片方式必须一样，或其中一边为广播表。


如下示例讲解如何将更多的计算下推到MySQL来加速执行

```sql
> EXPLAIN select * from customer, nation where c_nationkey = n_nationkey and n_regionkey = 3;
Project(c_custkey="c_custkey", c_name="c_name", c_address="c_address", c_nationkey="c_nationkey", c_phone="c_phone", c_acctbal="c_acctbal", c_mktsegment="c_mktsegment", c_comment="c_comment", n_nationkey="n_nationkey", n_name="n_name", n_regionkey="n_regionkey", n_comment="n_comment")
  BKAJoin(condition="c_nationkey = n_nationkey", type="inner")
    Gather(concurrent=true)
      LogicalView(tables="nation", shardCount=2, sql="SELECT * FROM `nation` AS `nation` WHERE (`n_regionkey` = ?)")
    Gather(concurrent=true)
      LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT * FROM `customer` AS `customer` WHERE (`c_nationkey` IN ('?'))")
```



若执行计划中出现了BKAJOIN，BKAJOIN每次从左表获取一批数据，就会拼成一个IN查询取出右表相关联的行，并在最后执行JOIN操作。由于左表数据量很大，需要取很多次才能完成查询，执行很慢。

无法下推JOIN的原因是：当前情况下，nation是按主键n_nationkey切分的，而本查询的JOIN Key是c_custkey，二者不同，所以下推失败。

考虑到nation （国家）表数据量并不大、且几乎没有修改操作，可以将其重建成如下广播表：

```sql
--- 修改后 ---
CREATE TABLE `nation` (
  `n_nationkey` int(11) NOT NULL,
  `n_name` varchar(25) NOT NULL,
  `n_regionkey` int(11) NOT NULL,
  `n_comment` varchar(152) DEFAULT NULL,
  PRIMARY KEY (`n_nationkey`)
) BROADCAST;  --- 声明为广播表
```



修改后，可以看到执行计划中不再出现JOIN，几乎所有计算都被下推到存储层MySQL执行了（LogicalView中），而上层仅仅是将结果收集并返回给用户（Gather算子），执行性能大大增强。

```sql
> EXPLAIN select * from customer, nation where c_nationkey = n_nationkey and n_regionkey = 3;
Gather(concurrent=true)
  LogicalView(tables="customer_[0-7],nation", shardCount=8, sql="SELECT * FROM `customer` AS `customer` INNER JOIN `nation` AS `nation` ON ((`nation`.`n_regionkey` = ?) AND (`customer`.`c_nationkey` = `nation`.`n_nationkey`))")
```

更多关于下推的原理和优化，请参见[查询改写与下推](query-rewriting.md)。

增加索引 
-------------------------

PolarDB-X支持[全局二级索引](../../features/topics/gsi.md)

以下以慢SQL示例来讲解如何通过GSI下推更多算子

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders, customer
          where o_custkey = c_custkey and o_orderdate = '2019-11-11' and o_totalprice > 100;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
  HashJoin(condition="o_custkey = c_custkey", type="inner")
    Gather(concurrent=true)
      LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer`")
    Gather(concurrent=true)
      LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` > ?))")
```



执行计划中，orders按照o_orderkey拆分而customer按照c_custkey拆分，由于拆分维度不同JOIN算子不能下推。考虑到2019-11-11当天总价高于100的订单非常多，跨分片JOIN耗时很高，需要在orders表上创建一个GSI来使得JOIN算子可以下推。查询中使用到了orders表的o_orderkey, o_custkey, o_orderdate, o_totalprice四列，其中o_orderkey, o_custkey分别是主表和索引表的拆分键，o_orderdate, o_totalprice作为覆盖列包含在索引中用于避免回表。

```sql
> create global index i_o_custkey on orders(`o_custkey`) covering(`o_orderdate`, `o_totalprice`)
DBPARTITION BY HASH(`o_custkey`) TBPARTITION BY HASH(`o_custkey`) TBPARTITIONS 4;
```



增加GSI并通过force index(i_o_custkey)强制使用索引后，跨分片JOIN变为MySQL上的局部JOIN （IndexScan中），并且通过覆盖列避免了回表操作，查询性能得到提升。

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders force index(i_o_custkey), customer
          where o_custkey = c_custkey and o_orderdate = '2019-11-11' and o_totalprice > 100;
Gather(concurrent=true)
  IndexScan(tables="i_o_custkey_[0-7],customer_[0-7]", shardCount=8, sql="SELECT `i_o_custkey`.`o_orderkey`, `customer`.`c_custkey`, `customer`.`c_name` FROM `i_o_custkey` AS `i_o_custkey` INNER JOIN `customer` AS `customer` ON (((`i_o_custkey`.`o_orderdate` = ?) AND (`i_o_custkey`.`o_custkey` = `customer`.`c_custkey`)) AND (`i_o_custkey`.`o_totalprice` > ?))")
```

更多关于全局二级索引的使用细节，请参见[全局二级索引](../../dev-guide/topics/gsi-faq.md)。

执行计划调优 
---------------------------

大多数情况下，PolarDB-X的查询优化器可以自动产生最佳的执行计划。但是，少数情况下，可能因为统计信息存在缺失、误差等，导致生成的执行计划不够好，这时，可以通过Hint来干预优化器行为，使之生成更好的执行计划。如下示例将讲解执行计划的调优。

```sql
> EXPLAIN select o_orderkey, c_custkey, c_name from orders, customer
          where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
  HashJoin(condition="o_custkey = c_custkey", type="inner")
    Gather(concurrent=true)
      LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer`")
    Gather(concurrent=true)
      LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` < ?))")
```



实际上2019-11-15这一天总价低于10元的订单数量很小，只有几条，这时候用BKAJOIN是比Hash JOIN更好的选择（关于BKAJOIN和Hash JOIN的介绍，请参见[JOIN优化和执行](join-optimizing.md)

通过如下/\*+TDDL:BKA_JOIN(orders, customer)\*/ Hint强制优化器使用BKAJOIN（LookupJOIN）:

```sql
> EXPLAIN /*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer
          where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
Project(o_orderkey="o_orderkey", c_custkey="c_custkey", c_name="c_name")
  BKAJoin(condition="o_custkey = c_custkey", type="inner")
    Gather(concurrent=true)
      LogicalView(tables="orders_[0-7]", shardCount=8, sql="SELECT `o_orderkey`, `o_custkey` FROM `orders` AS `orders` WHERE ((`o_orderdate` = ?) AND (`o_totalprice` < ?))")
    Gather(concurrent=true)
      LogicalView(tables="customer_[0-7]", shardCount=8, sql="SELECT `c_custkey`, `c_name` FROM `customer` AS `customer` WHERE (`c_custkey` IN ('?'))")
```

可以选择执行加如下Hint的查询：

```sql
/*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer where o_custkey = c_custkey and o_orderdate = '2019-11-15' and o_totalprice < 10;
```

以上操作加快了SQL查询速度。为了让Hint发挥作用，可以将应用中的SQL加上Hint，或者更方便的方式是使用执行计划管理（Plan Management）功能对该SQL固定执行计划。具体操作如下：

```sql
BASELINE FIX SQL /*+TDDL:BKA_JOIN(orders, customer)*/ select o_orderkey, c_custkey, c_name from orders, customer where o_custkey = c_custkey and o_orderdate = '2019-11-15';
```



这样一来，对于这条SQL（参数可以不同），PolarDB-X都会采用如上固定的执行计划。更多关于执行计划管理的信息，请参见[执行计划管理](spm.md)

并发执行 
-------------------------

用户可以通过HINT /\*+TDDL:PARALLELISM=4\*/ 指定并行度，充分利用多核能力加速计算。比如以下例子:

```sql
mysql> explain physical select a.k, count(*) cnt from sbtest1 a, sbtest1 b where a.id = b.k and a.id > 1000 group by k having cnt > 1300 or
der by cnt limit 5, 10;
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PLAN                                                                                                                                                              |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ExecutorType: AP_LOCAL                                                                                                                                                 |
| The Query's MaxConcurrentParallelism: 2                                                                                                                           |
| Fragment 1                                                                                                                                                        |
|     Shuffle Output layout: [BIGINT, BIGINT] Output layout: [BIGINT, BIGINT]                                                                                       |
|     Output partitioning: SINGLE [] Parallelism: 1                                                                                                                 |
|     TopN(sort="cnt ASC", offset=?2, fetch=?3)                                                                                                                     |
|   Filter(condition="cnt > ?1")                                                                                                                                    |
|     HashAgg(group="k", cnt="COUNT()")                                                                                                                             |
|       BKAJoin(condition="k = id", type="inner")                                                                                                                   |
|         RemoteSource(sourceFragmentIds=[0], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k))                                                             |
|         Gather(concurrent=true)                                                                                                                                   |
|           LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `k` FROM `sbtest1` AS `sbtest1` WHERE ((`k` > ?) AND (`k` IN (...)))") |
| Fragment 0                                                                                                                                                        |
|     Shuffle Output layout: [BIGINT, BIGINT] Output layout: [BIGINT, BIGINT]                                                                                       |
|     Output partitioning: SINGLE [] Parallelism: 1 Splits: 16                                                                                                      |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `id`, `k` FROM `sbtest1` AS `sbtest1` WHERE (`id` > ?)")                     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



默认的并行度并不高，通过强制指定并行度，利用单机或者多机并行模式来加速。

```sql
mysql> explain physical /*+TDDL:PARALLELISM=8*/select a.k, count(*) cnt from sbtest1 a, sbtest1 b where a.id = b.k and a.id > 1000 group by k having cnt > 1300 order by cnt limit 5, 10;                                                                                                                                                     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ExecutorMode: AP_LOCAL                                                                                                                                      |
| Fragment 0 dependency: [] parallelism: 8                                                                                                                    |
| BKAJoin(condition="k = id", type="inner")                                                                                                                   |
|   Gather(concurrent=true)                                                                                                                                   |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `id`, `k` FROM `sbtest1` AS `sbtest1` WHERE (`id` > ?)")               |
|   Gather(concurrent=true)                                                                                                                                   |
|     LogicalView(tables="[000000-000003].sbtest1_[00-15]", shardCount=16, sql="SELECT `k` FROM `sbtest1` AS `sbtest1` WHERE ((`k` > ?) AND (`k` IN (...)))") |
| Fragment 1 dependency: [] parallelism: 8                                                                                                                    |
| LocalBuffer                                                                                                                                                 |
|   RemoteSource(sourceFragmentIds=[0], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k, INTEGER_UNSIGNED k0))                                        |
| Fragment 2 dependency: [0, 1] parallelism: 8                                                                                                                |
| Filter(condition="cnt > ?1")                                                                                                                                |
|   HashAgg(group="k", cnt="COUNT()")                                                                                                                         |
|     RemoteSource(sourceFragmentIds=[1], type=RecordType(INTEGER_UNSIGNED id, INTEGER_UNSIGNED k, INTEGER_UNSIGNED k0))                                      |
| Fragment 3 dependency: [0, 1] parallelism: 1                                                                                                                |
| LocalBuffer                                                                                                                                                 |
|   RemoteSource(sourceFragmentIds=[2], type=RecordType(INTEGER_UNSIGNED k, BIGINT cnt))                                                                      |
| Fragment 4 dependency: [2, 3] parallelism: 1                                                                                                                |
| TopN(sort="cnt ASC", offset=?2, fetch=?3)                                                                                                                   |
|   RemoteSource(sourceFragmentIds=[3], type=RecordType(INTEGER_UNSIGNED k, BIGINT cnt))                                                                      |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



