索引推荐 
===========================

索引优化通常需要依赖运维或开发人员对数据库引擎内部优化和执行原理的深入理解。为优化体验和降低操作门槛，PolarDB-X推出了基于代价优化器的索引推荐功能，可根据查询语句分析并推荐索引，帮助您降低查询耗时，提升数据库性能。

注意事项 
-------------------------

索引推荐功能仅针对您当前指定的SQL查询语句进行分析与推荐。在根据推荐的信息创建索引前，您需要评估创建该索引对其它查询的影响。

环境说明 
-------------------------

TPC-H是业界常用的基准测试方法，由TPC委员会制定发布，用于评测数据库的分析型查询能力。TPC-H基准测试方法包含8张数据表、22条复杂的SQL查询（即Q1\~Q22）。下图为执行TPC-H中的Q17（小订单收入查询）的返回信息，可查看到执行该查询语句消耗的时间为28.76秒。本文将通过索引推荐功能，优化该查询语句的执行效率。

1. **查询索引推荐信息**

   如需查询某个查询语句的索引推荐信息，您只需在该查询语句前增加EXPLAIN ADVISOR命令，示例如下：

   ```sql
   EXPLAIN ADVISOR
   SELECT sum(l_extendedprice) / 7.0 AS avg_yearly
   FROM lineitem,
        part
   WHERE p_partkey = l_partkey
     AND p_brand = 'Brand#23'
     AND p_container = 'MED BOX'
     AND l_quantity <
       (SELECT 0.2 * avg(`l_quantity`)
        FROM lineitem
        WHERE l_partkey = p_partkey);
   ```

   执行上述命令后，PolarDB-X将返回推荐的索引创建语句、添加索引前后的代价等信息，详细的返回信息及其注释如下所示：
   **说明**
   * 本案例中，预计磁盘I/O提升百分比为3024.7%，表明使用推荐的索引将带来较大的收益。
   
   * 当PolarDB-X无法推荐索引时，返回信息中会建议您在业务低峰期，对目标表执行Analyze Table命令刷新统计信息（该操作会消耗较大的I/O资源）。当统计信息更新后，再次执行索引推荐可获得更准确的索引。SQL复制代码。

     ```sql
     IMPROVE_VALUE: 2465.3%        # 预计综合代价提升百分比
       IMPROVE_CPU: 59377.4%       # 预计CPU提升百分比
       IMPROVE_MEM: 0.4%           # 预计内存提升百分比
        IMPROVE_IO: 3024.7%        # 预计磁盘I/O提升百分比
       IMPROVE_NET: 2011.1%        # 预计网络传输提升百分比
      BEFORE_VALUE: 4.711359845E8  # 添加索引前综合代价值
        BEFORE_CPU: 1.19405577E7   # 添加索引前CPU估算值
        BEFORE_MEM: 426811.2       # 添加索引前内存消耗估算值
         BEFORE_IO: 44339          # 添加索引前磁盘I/O估算值
        BEFORE_NET: 47.5           # 添加索引前网络传输估算值
       AFTER_VALUE: 1.83655008E7   # 添加索引后综合代价值
         AFTER_CPU: 20075.8        # 添加索引后CPU估算值
         AFTER_MEM: 425016         # 添加索引后内存消耗估算值
          AFTER_IO: 1419           # 添加索引后磁盘I/O估算值
         AFTER_NET: 2.2            # 添加索引后网络传输估算值
      ADVISE_INDEX: ALTER TABLE `lineitem` ADD  INDEX `__advise_index_lineiteml_partkey`(`l_partkey`);
     /* ADVISE_INDEX中的内容为推荐的索引创建语句 */
          NEW_PLAN:                # 添加索引后预计执行计划
     Project(avg_yearly="$f0 / ?0")
       HashAgg($f0="SUM(l_extendedprice)")
         Filter(condition="l_quantity < $16 * f17w0$o0")
           SortWindow(p_partkey="p_partkey", l_partkey="l_partkey", l_quantity="l_quantity", l_extendedprice="l_extendedprice", $16="$16", f5w0$o0="window#0AVG($2)", Reference Windows="window#0=window(partition {1} order by [] range between UNBOUNDED PRECEDING and UNBOUNDED PRECEDING aggs [AVG($2)])")
             MemSort(sort="l_partkey ASC")
               BKAJoin(condition="l_partkey = p_partkey", type="inner")
                 Gather(concurrent=true)
                   LogicalView(tables="[0000,0001].part", shardCount=2, sql="SELECT `p_partkey` FROM `part` AS `part` WHERE ((`p_brand` = ?) AND (`p_container` = ?))")
                 Gather(concurrent=true)
                   LogicalView(tables="[0000,0001].lineitem", shardCount=2, sql="SELECT `l_partkey`, `l_quantity`, `l_extendedprice`, ? AS `$16` FROM `lineitem` AS `lineitem` WHERE (`l_partkey` IN (...))")
              INFO: LOCAL_INDEX    # 其它信息
     ```

     
   

   

   

2. **根据推荐信息创建索引**
   1. 评估创建该索引带来的收益，然后根据返回结果ADVISE_INDEX中的SQL语句创建索引。

      ```sql
      ALTER TABLE `lineitem` ADD  INDEX `__advise_index_lineiteml_partkey`(`l_partkey`);
      ```

      
   
   2. 再次执行TPC-H中的Q17（小订单收入查询），耗时减少至1.41秒，查询效率得到大幅提升。
   
![SQL](../images/p332510.png)
   

   



