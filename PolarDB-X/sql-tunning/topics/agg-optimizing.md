聚合优化和执行 
============================

本文介绍如何优化器和执行器如何处理聚合（Group-by），以达到减少数据传输量和提高执行效率的效果。

基本概念 
-------------------------

聚合操作（Aggregate，简称Agg）语义为按照GROUP BY指定列对输入数据进行聚合的计算，或者不分组、对所有数据进行聚合的计算。PolarDB-X支持如下聚合函数：

* COUNT

* SUM

* AVG

* MAX

* MIN

* BIT_OR

* BIT_XOR

* GROUP_CONCAT




聚合（Agg） 
----------------------------

本文介绍均为不下推的Agg的实现。如果已被下推到LogicalView中，则由存储层MySQL来选择执行方式，聚合（Agg）由两种主要的算子HashAgg和SortAgg实现。

**HashAgg** 

HashAgg利用哈希表实现聚合：

1. 根据输入行的分组列的值，通过Hash找到对应的分组。

2. 按照指定的聚合函数，对该行进行聚合计算。

3. 重复以上步骤直到处理完所有的输入行，最后输出聚合结果。




```sql
> explain select count(*) from t1 join t2 on t1.id = t2.id group by t1.name,t2.name;
Project(count(*)="count(*)")
  HashAgg(group="name,name0", count(*)="COUNT()")
    BKAJoin(condition="id = id", type="inner")
      Gather(concurrent=true)
        LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
      Gather(concurrent=true)
        LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```

Explain结果中，HashAgg算子还包含以下关键信息：

* group：表示GROUP BY字段，示例中为name,name0分别引用t1,t2表的name列，当存在相同别名会通过后缀数字区分 。

* 聚合函数：等号（=） 前为聚合函数对应的输出列名，其后为对应的计算方法。示例中 count(\*)="COUNT()" ，第一个 count(\*) 对应输出的列名，随后的COUNT()表示对其输入数据进行计数。


HashAgg对应可以通过Hint来关闭：`/*+TDDL:cmd_extra(ENABLE_HASH_AGG=false)*/`

**SortAgg** 

SortAgg在输入数据已按分组列排序的情况，对各个分组依次完成聚合。

* 保证输入按指定的分组列排序（例如，可能会看到 MergeSort 或 MemSort）。

* 逐行读入输入数据，如果分组与当前分组相同，则对其进行聚合计算。

* 如果分组与当前分组不同，则输出当前分组上的聚合结果。


相比 HashAgg，SortAgg 每次只要处理一个分组，内存消耗很小；相对的，HashAgg 需要把所有分组存储在内存中，需要消耗较多的内存。

```sql
> explain select count(*) from t1 join t2 on t1.id = t2.id group by t1.name,t2.name order by t1.name, t2.name;
Project(count(*)="count(*)")
  MemSort(sort="name ASC,name0 ASC")
    HashAgg(group="name,name0", count(*)="COUNT()")
      BKAJoin(condition="id = id", type="inner")
        Gather(concurrent=true)
          LogicalView(tables="t1", shardCount=2, sql="SELECT `id`, `name` FROM `t1` AS `t1`")
        Gather(concurrent=true)
          LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `id`, `name` FROM `t2` AS `t2` WHERE (`id` IN ('?'))")
```

SortAgg对应可以通过Hint来关闭：`/*+TDDL:cmd_extra(ENABLE_SORT_AGG=false)*/`

**两阶段聚合优化** 

两阶段聚合，即通过将Agg拆分为部分聚合（Partial Agg）和最终聚合（Final Agg）的两个阶段，先对部分结果集做聚合，然后将这些部分聚合结果汇总，得到整体聚合的结果。

如下示例的SQL中，HashAgg 中拆分出的部分聚合（PartialAgg）会被下推至MySQL上的各个分表，而其中的AVG函数也被拆分成 SUM和 COUNT 以实现两阶段的计算：

```sql
> explain select avg(age) from t2 group by name
Project(avg(age)="sum_pushed_sum / sum_pushed_count")
  HashAgg(group="name", sum_pushed_sum="SUM(pushed_sum)", sum_pushed_count="SUM(pushed_count)")
    Gather(concurrent=true)
      LogicalView(tables="t2_[0-3]", shardCount=4, sql="SELECT `name`, SUM(`age`) AS `pushed_sum`, COUNT(`age`) AS `pushed_count` FROM `t2` AS `t2` GROUP BY `name`")
```

两阶段聚合的优化能大大减少数据传输量、提高执行效率。

总的来说，大部分场景做聚合的时候都倾向于选择HashAgg，只要当以下场景下才适合选择SortAgg做聚合：

1. 数据比较多，内存严重不足。

2. 聚合算子的输入已经按照Group By 列做好排序，这样做SortAgg就不需要额外排序，执行效率会更高。

3. 当数据有严重倾斜，导致HashAgg执行效率不高，优先使用SortAgg




