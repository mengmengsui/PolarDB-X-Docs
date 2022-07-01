排序优化和执行 
============================

本文介绍如何排序（Order-by）算子，以达到减少数据传输量和提高执行效率的效果。

基本概念 
-------------------------

排序操作（Sort）语义为按照指定的ORDER BY列对输入进行排序。本文介绍均为不下推的Sort的算子的实现。如果已被下推到LogicalView中，则由存储层MySQL来选择执行方式。

排序（Sort） 
-----------------------------

PolarDB-X中的排序算子主要包括 MemSort、TopN，以及 MergeSort。

**MemSort** 

PolarDB-X中的通用的排序实现为MemSort算子，即内存中运行快速排序（Quick Sort）算法。下面是一个用到MemSort算子的例子：

```sql
> explain select t1.name from t1 join t2 on t1.id = t2.id order by t1.name,t2.name;
Project(name="name")
  MemSort(sort="name ASC,name0 ASC")
    Project(name="name", name0="name0")
      BKAJoin(condition="id = id", type="inner")
        Gather(concurrent=true)
          LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
        Gather(concurrent=true)
          LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```



**TopN** 

当SQL中ORDER BY和LIMIT一起出现时，Sort算子和Limit算子会合并成TopN算子。

TopN算子维护一个最大或最小堆，按照排序键的值，堆中始终保留最大或最小的N行数据。当处理完全部的输入数据时，堆中留下的N个行（或小于N个）就是需要的结果。

```sql
> explain select t1.name from t1 join t2 on t1.id = t2.id order by t1.name,t2.name limit 10;
Project(name="name")
  TopN(sort="name ASC,name0 ASC", offset=0, fetch=?0)
    Project(name="name", name0="name0")
      BKAJoin(condition="id = id", type="inner")
        Gather(concurrent=true)
          LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
        Gather(concurrent=true)
          LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```



**MergeSort** 

通常，只要语义允许，SQL中的排序操作会被下推到MySQL上执行，而PolarDB-X执行层只做最后的归并操作，即MergeSort。严格来说，MergeSort 不仅仅是排序，更是一种数据重分布算子（类似 Gather）。下面的SQL是对t1表进行排序，经过PolarDB-X查询优化器的优化，Sort算子被下推至各个MySQL分片中执行，最终只在上层做归并操作。

```sql
> explain select name from t1 order by name;
MergeSort(sort="name ASC")
  LogicalView(tables="t1", shardCount=2, sql="SELECT `name` FROM `t1` AS `t1` ORDER BY `name`")
```

相比 MemSort，MergeSort 算法可以减少PolarDB-X层的内存消耗，并充分利用 MySQL 层的计算能力。

