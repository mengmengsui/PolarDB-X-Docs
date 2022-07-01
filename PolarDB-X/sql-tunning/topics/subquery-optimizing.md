子查询优化和执行 
=============================

子查询是指在父查询的WHERE子句或HAVING子句中嵌套另一个SELECT语句的查询，本文主要介绍如何子查询。

基本概念 
-------------------------

根据是否存在关联项，子查询可以分为非关联子查询和关联子查询。非关联子查询是指该子查询的执行不依赖外部查询的变量，这种子查询一般只需要计算一次；而关联子查询中存在引用自外层查询的变量，逻辑上，这种子查询需要每次带入相应的变量、计算多次。

```sql
/* 例子：非关联子查询 */
SELECT * FROM lineitem WHERE l_partkey IN (SELECT p_partkey FROM part);
/* 例子：关联子查询（l_suppkey 是关联项） */
SELECT * FROM lineitem WHERE l_partkey IN (SELECT ps_partkey FROM partsupp WHERE ps_suppkey = l_suppkey);
```

PolarDB-X子查询支持绝大多数的子查询写法，具体参见[SQL使用限制](../../dev-guide/topics/limitation.md)。

子查询执行 
--------------------------

对于多数常见的子查询形式，PolarDB-X可以将其改写为高效的SemiJoin或类似的基于JOIN的计算方式。这样做的好处是显而易见的。当数据量较大时，无需真正带入不同参数循环迭代，大大降低了执行代价。这种查询改写技术称为子查询的去关联化（Unnesting）。

如下示例中2个子查询去关联化可以看到执行计划中使用JOIN代替了子查询。

```sql
> EXPLAIN SELECT p_partkey, (
      SELECT COUNT(ps_partkey) FROM partsupp WHERE ps_suppkey = p_partkey
      ) supplier_count FROM part;
Project(p_partkey="p_partkey", supplier_count="CASE(IS NULL($10), 0, $9)", cor=[$cor0])
  HashJoin(condition="p_partkey = ps_suppkey", type="left")
    Gather(concurrent=true)
      LogicalView(tables="part_[0-7]", shardCount=8, sql="SELECT * FROM `part` AS `part`")
    Project(count(ps_partkey)="count(ps_partkey)", ps_suppkey="ps_suppkey", count(ps_partkey)2="count(ps_partkey)")
      HashAgg(group="ps_suppkey", count(ps_partkey)="SUM(count(ps_partkey))")
        Gather(concurrent=true)
          LogicalView(tables="partsupp_[0-7]", shardCount=8, sql="SELECT `ps_suppkey`, COUNT(`ps_partkey`) AS `count(ps_partkey)` FROM `partsupp` AS `partsupp` GROUP BY `ps_suppkey`")
```



```sql
> EXPLAIN SELECT p_partkey, (
      SELECT COUNT(ps_partkey) FROM partsupp WHERE ps_suppkey = p_partkey
      ) supplier_count FROM part;
Project(p_partkey="p_partkey", supplier_count="CASE(IS NULL($10), 0, $9)", cor=[$cor0])
  HashJoin(condition="p_partkey = ps_suppkey", type="left")
    Gather(concurrent=true)
      LogicalView(tables="part_[0-7]", shardCount=8, sql="SELECT * FROM `part` AS `part`")
    Project(count(ps_partkey)="count(ps_partkey)", ps_suppkey="ps_suppkey", count(ps_partkey)2="count(ps_partkey)")
      HashAgg(group="ps_suppkey", count(ps_partkey)="SUM(count(ps_partkey))")
        Gather(concurrent=true)
          LogicalView(tables="partsupp_[0-7]", shardCount=8, sql="SELECT `ps_suppkey`, COUNT(`ps_partkey`) AS `count(ps_partkey)` FROM `partsupp` AS `partsupp` GROUP BY `ps_suppkey`")
```



某些场景下，PolarDB-X无法将子查询进行去关联化，这时会采用迭代执行的方式。如果外层查询数据量很大，迭代执行可能会非常慢。

如下示例由于OR l_partkey &lt; 50的存在，导致子查询无法被去关联化，因而采用了迭代执行：

```sql
> EXPLAIN SELECT * FROM lineitem WHERE l_partkey IN (SELECT ps_partkey FROM partsupp WHERE ps_suppkey = l_suppkey) OR l_partkey IS NOT
Filter(condition="IS(in,[$1])[29612489] OR l_partkey < ?0")
  Gather(concurrent=true)
    LogicalView(tables="QIMU_0000_GROUP,QIMU_0001_GROUP.lineitem_[0-7]", shardCount=8, sql="SELECT * FROM `lineitem` AS `lineitem`")
>> individual correlate subquery : 29612489
Gather(concurrent=true)
  LogicalView(tables="QIMU_0000_GROUP,QIMU_0001_GROUP.partsupp_[0-7]", shardCount=8, sql="SELECT * FROM (SELECT `ps_partkey` FROM `partsupp` AS `partsupp` WHERE (`ps_suppkey` = `l_suppkey`)) AS `t0` WHERE (((`l_partkey` = `ps_partkey`) OR (`l_partkey` IS NULL)) OR (`ps_partkey` IS NULL))")
```

这种情形下，建议改写SQL去掉子查询的OR条件。

