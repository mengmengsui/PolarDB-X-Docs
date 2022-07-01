如何限流慢SQL 
=============================

本文介绍了如何对慢SQL进行有效限流。

在数据库会话或者慢日志中发现大量慢SQL，大量占用数据库资源，同时活跃会话数、CPU使用率、IOPS、内存使用率等监控指标一项或者多项处于高位。分析后发现这些慢SQL不属于核心业务，是优化不足的烂SQL，为保障核心业务的稳定运行，此时我们需要对其进行限流。

相关限流语法，请参见[SQL限流](../../dev-guide/topics/sql-concurrency-control-devguide.md)。

SQL限流的运维操作步骤 
---------------------------------

1. 使用如下语句发现慢SQL。

   ```sql
   select *
     from information_schema.processlist
    where COMMAND!= 'SLEEP'
      and TIME>= 1000
    order by TIME DESC;
   ```

   

2. 分析慢SQL，请参见[如何分析及优化慢SQL](slow-query-tuning.md)。

3. 创建限流规则，可使用SQL命令。

4. 从以下几方面观察限流规则效果。
   * 监控指标恢复情况；
   
   * 业务侧反馈；
   
   * `show ccl_rules`查看每个限流规则的限流情况的统计信息；
   
   * 查看会话和SQL日志。
   
   
   
5. 创建索引、修改SQL、增加资源等。

6. 关闭限流规则，使用`DROP CCL_RULE`或者`CLEAR CCL_RULES`语句。




如下案例说明了如何对发现的慢SQL进行限流，您可以参照案例中的限流规则，修改后使用。

案例1: 慢SQL属于同一个SQL模版 
----------------------------------------

某DBA收到了数据库资源某指标处于高位的报警，查看数据库慢日志和会话后均发现有如下的慢SQL：

```sql
+--------+---------------+---------------------+--------------------+---------+------+-------+----------------------------------------------+-----------------+
| ID     | USER          | HOST                | DB                 | COMMAND | TIME | STATE | INFO                                         | SQL_TEMPLATE_ID |
+--------+---------------+---------------------+--------------------+---------+------+-------+----------------------------------------------+-----------------+
| 951494 | userxxxxxxxxx | 222.0.0.1:33830     | analy_db           | Query   |   40 |       | select * from bmsql_oorder where `o_id` > 12 | 65c92c88        |
| 952468 | userxxxxxxxxx | 222.0.0.1:33517     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where `o_id` > 10 | 65c92c88        |
| 953468 | userxxxxxxxxx | 222.0.0.1:33527     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where `o_id` > 23 | 65c92c88        |
| 954468 | userxxxxxxxxx | 222.0.0.1:33537     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where `o_id` > 25 | 65c92c88        |
| 955468 | userxxxxxxxxx | 222.0.0.1:33547     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where `o_id` > 27 | 65c92c88        |
+--------+---------------+---------------------+--------------------+---------+------+-------+----------------------------------------------+-----------------+
```



可见，这些慢SQL属于同一个SQL模版（模版ID为65c92c88）：

```sql
select * from bmsql_oorder where `o_id` > ?
```



bmsql_oorder为一个数据量较大的表，而且列o_id上没有索引，显然这个一个未经优化的SQL，占尽了数据库资源影响了其他重要SQL的正常执行。这是一个非常适合利用模版ID去做SQL限流的场景。

**创建限流规则**

* 如果这个SQL模版的SQL不应该在当时执行，而且应该在业务低峰期执行，则我们可以创建SQL限流规则不让它执行：

  ```sql
  CREATE CCL_RULE `KILL_CCL`       //限流规则名称为KILL_CCL
      ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
    TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
      FOR SELECT                     //&匹配是SELECT类型的SQL语句
    FILTER BY TEMPLATE '65c92c88'  //&匹配模版ID
    WITH MAX_CONCURRENCY = 0;      //设置单节点并发度为0，不允许匹配到的SQL执行
  ```

  

  客户端再次执行这类SQL的时候将会返回报错信息：

  ```sql
  ERROR 3009 (HY000): [13172dbaf2801000][10.93.159.222:3029][analy_db]Exceeding the max concurrency 0 per node of ccl rule KILL_CCL
  ```

  

* 如果允许这个SQL模版的SQL少量执行，只要不占尽数据库资源就行，则我们可以在创建限流规则的时候设置一定的并发度：

  ```sql
  CREATE CCL_RULE `KILL_CCL_2`       //限流规则名称为KILL_CCL
      ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
    TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
      FOR SELECT                     //&匹配是SELECT类型的SQL语句
    FILTER BY TEMPLATE '65c92c88'  //&匹配模版ID 65c92c88
    WITH MAX_CONCURRENCY = 2;      //允许单个节点可以同时有两个这样的SQL在执行
  ```


* 如果希望这个SQL模版的SQL执行的时候可以慢，但尽量不要出错，则可以设置等待队列和等待超时时间（默认为600秒）：

  ```sql
  CREATE CCL_RULE `QUEUE_CCL_2`       //限流规则名称为KILL_CCL
      ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
    TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
      FOR SELECT                     //&匹配是SELECT类型的SQL语句
    FILTER BY TEMPLATE '65c92c88'  //&匹配模版ID
    WITH MAX_CONCURRENCY = 2, WAIT_QUQUE_SIZE=20, WAIT_TIMEOUT=500; //单节点并发度为2，单节点等待队列长度为20，等待超时时间为500秒
  ```

  

  创建完后，可以通过`show ccl_rules`指令查询各个限流规则的实际效果，比如当前匹配到某个限流规则的正在执行的SQL的数量、被限流报错的SQL数量、总匹配成功次数等。如果想放开被限流SQL，比如在增加了某个索引后，被限流SQL的执行效率变高了，则可以通过`drop ccl_rule`命令来关闭指定限流规则，或者使用`clear ccl_rules`来关闭所有的限流规则。

  当然上面的SQL也可以通过关键字来限流，将SQL语句上的关键字做拆分，我们得到关键字列表：
  * select
  
  * from
  
  * bmsql_oorder
  
  * where
  
  * \`o_id\`
  
  
  
  
  

  创建限流规则：

  ```sql
  CREATE CCL_RULE `KILL_CCL`       //限流规则名称为KILL_CCL
      ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
    TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
      FOR SELECT                     //&匹配是SELECT类型的SQL语句
    FILTER BY KEYWORD('select','from','bmsql_oorder','where','`o_id`')  //&匹配模版ID
    WITH MAX_CONCURRENCY = 0;      //设置单节点并发度为0，不允许匹配到的SQL执行
  ```

  

  在能获取到模版ID（在SQL日志、explain命令、会话中）的情况下，我们还推荐使用更精准的基于模版ID的限流。

案例2: 慢SQL都是同一个SQL 
--------------------------------------

某DBA收到了数据库资源某指标处于高位的报警，查看数据库慢日志和会话后均发现有如下的慢SQL：

```sql
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
| ID     | USER          | HOST                | DB                 | COMMAND | TIME | STATE | INFO                                              | SQL_TEMPLATE_ID |
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
| 951494 | userxxxxxxxxx | 222.0.0.1:33830     | analy_db           | Query   |   40 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
| 952468 | userxxxxxxxxx | 222.0.0.1:33517     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
| 953468 | userxxxxxxxxx | 222.0.0.1:33527     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
| 954468 | userxxxxxxxxx | 222.0.0.1:33537     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
| 955468 | userxxxxxxxxx | 222.0.0.1:33547     | analy_db           | Query   |   43 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
```



bmsql_oorder中的符合o_carrier_id = 2条件的有较多记录，导致了慢SQL，如果使用模版ID限流，则会影响o_carrier_id不是2的SQL语句，如果使用关键字限流则会影响类似如下的正常SQL：

```sql
select * from bmsql_oorder where o_carrier_id = 2 limit 1;
select * from bmsql_oorder where o_carrier_id = 2 and o_c_id = 1;
```



限流具体的SQL，可以使用模版ID加关键字的方法，创建如下限流规则：

```sql
CREATE CCL_RULE `KILL_CCL`       //限流规则名称为KILL_CCL
    ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
  TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
    FOR SELECT                     //&匹配是SELECT类型的SQL语句
  FILTER BY TEMPLATE '438b00e4'  //&匹配模版ID 438b00e4
  FILTER BY KEYWORD('o_carrier_id','2')  //&匹配参数关键字
  WITH MAX_CONCURRENCY = 0;      //设置单节点并发度为0，不允许匹配到的SQL执行
```



如果使用PolarDB-X的CN内核版本为5.4.11以上，且该SQL不在prepare模式下执行，还可以使用如下高阶语法进行限流：

```sql
CREATE CCL_RULE `KILL_CCL`       //限流规则名称为KILL_CCL
    ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
  TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
    FOR SELECT                     //&匹配是SELECT类型的SQL语句
  FILTER BY QUERY 'select * from bmsql_oorder where o_carrier_id = 2'  //&匹配SQL语句
  WITH MAX_CONCURRENCY = 0;      //设置单节点并发度为0，不允许匹配到的SQL执行
```



案例3: 慢SQL集包含多个SQL模版 
----------------------------------------

某DBA收到了数据库资源某指标处于高位的报警，查看数据库慢日志和会话后均发现有如下的慢SQL：

```sql
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
| ID     | USER          | HOST                | DB                 | COMMAND | TIME | STATE | INFO                                              | SQL_TEMPLATE_ID |
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
| 951494 | userxxxxxxxxx | 222.0.0.1:33830     | analy_db           | Query   |   40 |       | select * from bmsql_oorder where o_carrier_id = 2 | 438b00e4        |
| 952468 | userxxxxxxxxx | 222.0.0.1:33517     | analy_db           | Query   |   43 |       | select * from bmsql_order_line where ol_o_id = 2  | 57a572f9        |
| 953468 | userxxxxxxxxx | 222.0.0.1:33527     | analy_db           | Query   |   43 |       | select * from bmsql_new_order where no_w_id = 2   | de6eefdb        |
+--------+---------------+---------------------+--------------------+---------+------+-------+---------------------------------------------------+-----------------+
```



此种情况较为复杂，如果一条明显执行效率很高的SQL也成了慢SQL，则不排除是由于网络抖动或者服务节点异常等原因导致运行效率降低从而产生大批量的慢SQL，也可能是由于真正的烂SQL完全耗尽了资源，导致原本正常的SQL也成了慢SQL，需要通过SQL分析具体原因，不在本文的讨论范围内。假设已经确定了需要限流的慢SQL，我们则可以针对每个模版ID创建一个限流规则。但随着限流规则增加，匹配效率会略有降低，当PolarDB-X的CN内核版本为5.4.11以上时，我们推荐使用多模版限流：

```sql
CREATE CCL_RULE `KILL_CCL`       //限流规则名称为KILL_CCL
    ON `analy_db`.`*`              //&匹配analy_db下的所有表上执行的SQL
  TO 'userxxxxxxxxx'@'%'         //&匹配来自userxxxxxxxxx用户的SQL
    FOR SELECT                     //&匹配是SELECT类型的SQL语句
  FILTER BY TEMPLATE('438b00e4','57a572f9','de6eefdb')  //&匹配中其中一个模版ID，则该匹配项算匹配成功
  WITH MAX_CONCURRENCY = 0;      //设置单节点并发度为0，不允许匹配到的SQL执行
```



如果确定会话中的慢SQL是都是需要限流的烂SQL，且PolarDB-X的CN内核版本为5.4.11以上时，可以开启[慢SQL限流](../../dev-guide/topics/sql-concurrency-control-devguide.md)。

总结 
-----------------------

SQL限流为应急措施，可在数据库由于烂SQL导致效率降低的时候，起到快速恢复的作用。对烂SQL进行限流后，用户需要将注意力集中在如何优化烂SQL上，并在合适的时机清空SQL限流规则。
