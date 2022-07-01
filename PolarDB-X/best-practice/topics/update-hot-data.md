如何支持热点更新场景 
===============================

背景介绍 
-------------------------

数据库中更新的模式为lock -&gt; update -&gt; unlock，当对数据库中的同一条记录有大量修改请求时，会造成大量的锁争抢与锁等待。请求量增加会导致TPS下降，延迟飙升。例如，秒杀场景中对于商品库存的扣减。

您可以选择在数据库内核中进行批处理，即对该条记录进行的更新操作使用组提交，更新的模式更改为lock-&gt; group update -&gt; unlock，从而减少锁争抢。结合流水线处理等优化，可以大大提高该场景的TPS，详情可见测试结果。

使用方法 
-------------------------

1. 开启hotspot相关功能。

   ```sql
   set global hotspot=on;
   set global hotspot_lock_type=on
   ```

   

2. 切换事务类型为XA，并在业务的update语句中添加inventory hint。

   ```sql
   begin;
   set drds_transaction_policy=xa; // 若默认事务策略是XA，则无须重复设置，否则每次均需设置事务策略为XA以使用热点更新hint
   UPDATE /*+ commit_on_success rollback_on_fail target_affect_row(number)*/ table_reference 
       SET assignment_list
       [WHERE where_condition];
   ```

   




注意事项 
-------------------------

* where条件应为主键更新或唯一键更新，且不支持带有全局索引的表（可包含本地索引）。

* 若已开启共享read_view，则应当先将共享read_view进行关闭，而后使用热点更新的能力。

* 热点更新hint的使用场景为单分片事务，无法在跨库场景中使用。




Inventory Hint各参数含义 
----------------------------------------

* commit_on_success（必选）如果该语句成功，则进行提交，连同该语句之前的未提交语句一并提交。

  

* rollback_on_fail如果该语句失败，则进行回滚，连同该语句之前的未提交语句一并回滚。

  

* target_affect_row(number)校验更新的行数是否符合预期，若不符合则更新失败。

  




示例 
-----------------------

* 添加commit_on_success以使用组提交等针对热点更新场景的优化（id为主键，使用如下语句对id=1的记录进行更新时，若更新成功则自动提交）

  ```sql
  begin;
  set drds_transaction_policy=xa;
  UPDATE /*+ commit_on_success*/ table_test SET c = c - 1 WHERE id = 1;
  ```

  

* 使用rollback_on_fail，可使得更新失败时自动进行回滚

  ```sql
  begin;
  set drds_transaction_policy=xa;
  UPDATE /*+ commit_on_success rollback_on_fail*/ table_test SET c = c - 1 WHERE id = 1;
  ```

  

* 使用target_affect_row(number)，使得该update语句的预期更新行数为number，若不为number，则更新失败

  ```sql
  begin;
  set drds_transaction_policy=xa;
  UPDATE /*+ commit_on_success rollback_on_fail target_affect_row(1)*/ table_test SET c = c - 1 WHERE id = 1;
  ```

  

* 在带有热点更新hint的update语句前，可对同一个物理库中的表进行DML操作

  ```sql
  begin;
  set drds_transaction_policy=xa;
  INSERT into table_test_2 values (1,1);
  UPDATE /*+ commit_on_success rollback_on_fail target_affect_row(1)*/ table_test SET c = c - 1 WHERE id = 1;
  ```

  

* 在带有热点更新hint的update语句后继续进行DML操作，原因是其后的语句会重新开启新的事务（非必须情况下不推荐）

  ```sql
  begin;
  set drds_transaction_policy=xa;
  UPDATE /*+ commit_on_success rollback_on_fail target_affect_row(1)*/ table_test SET c = c - 1 WHERE id = 1;
  INSERT into table_test_2 values (1,1); // 不推荐，因为该条sql会重新开启一个新的事务
  ```

  




查看inventory hint是否生效 
-----------------------------------------

* 使用命令`show global status like "%Group_update%"`查看组提交状态，Group_update_leader_count一直增加则说明触发了热点组提交的优化逻辑。

  ```sql
  mysql> show global status like "%Group_update%";
  +---------------------------------------+--------+
  | Variable_name                         | Value  |
  +---------------------------------------+--------+
  | Group_update_fail_count               | 54     |
  | Group_update_follower_count           | 962869 |
  | Group_update_free_count               | 2      |
  | Group_update_group_same_count         | 0      |
  | Group_update_gu_leak_count            | 0      |
  | Group_update_ignore_count             | 0      |
  | Group_update_insert_dup               | 0      |
  | Group_update_leader_count             | 168292 |
  | Group_update_lock_fail_count          | 0      |
  | Group_update_mgr_recycle_queue_length | 0      |
  | Group_update_recycle_queue_length     | 0      |
  | Group_update_reuse_count              | 23329  |
  | Group_update_total_count              | 2      |
  +---------------------------------------+--------+
  13 rows in set (0.01 sec)
  ```

  

* 使用`show physical full processlist`查看update的状态，是否出现hotspot字样

  ```sql
  mysql> show physical full processlist where command != 'Sleep';
  +-------+------+-----+---------+-------------+---------+------+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Group | Atom | Id  | User    | db          | Command | Time | State                   | Info                                                                                                                                                                                         |
  +-------+------+-----+---------+-------------+---------+------+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  |     0 |    0 |  56 | diamond | test_000001 | Query   |    0 | hotspot wait for commit | /*DRDS /127.0.0.1/12e774cab8800000-128/0// */UPDATE  /*+COMMIT_ON_SUCCESS ROLLBACK_ON_FAIL TARGET_AFFECT_ROW(1) */ `test_hotline_lZTr` AS `test_hotline` SET `b` = (`b` + 1) WHERE (`a` = 1) |
  |     0 |    0 | 822 | diamond | test_000001 | Query   |    0 | query end               | /*DRDS /127.0.0.1/12e774c4e9400000-563/0// */UPDATE  /*+COMMIT_ON_SUCCESS ROLLBACK_ON_FAIL TARGET_AFFECT_ROW(1) */ `test_hotline_lZTr` AS `test_hotline` SET `b` = (`b` + 1) WHERE (`a` = 1) |
  |     0 |    0 | 831 | diamond | test_000001 | Query   |    0 | hotspot wait for commit | /*DRDS /127.0.0.1/12e774c551000000-509/0// */UPDATE  /*+COMMIT_ON_SUCCESS ROLLBACK_ON_FAIL TARGET_AFFECT_ROW(1) */ `test_hotline_lZTr` AS `test_hotline` SET `b` = (`b` + 1) WHERE (`a` = 1) |
  |     0 |    0 | 838 | diamond | test_000000 | Query   |    0 | starting                | show full processlist                                                                                                                                                                        |
  +-------+------+-----+---------+-------------+---------+------+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  4 rows in set (0.33 sec)
  ```

  




热点更新测试 
---------------------------

**测试表定义**

```sql
CREATE TABLE sbtest(id INT UNSIGNED NOT NULL PRIMARY KEY, c BIGINT UNSIGNED NOT NULL);
```

 **测试语句**

```sql
UPDATE /*+ COMMIT_ON_SUCCESS ROLLBACK_ON_FAIL TARGET_AFFECT_ROW(1) */ sbtest SET c=c+1 WHERE id = 1;
```

 **测试工具**

sysbench
**机器规格**

4C8G×2 (两节点)
**测试结果** 

|  场景  | 1线程 | 4线程 | 8线程  | 16线程 | 32线程 | 64线程  | 128线程 | 256线程 | 512线程 |
|------|-----|-----|------|------|------|-------|-------|-------|-------|
| 热点更新 | 298 | 986 | 1872 | 3472 | 6315 | 10138 | 13714 | 15803 | 23262 |
| 普通更新 | 318 | 423 | 409  | 409  | 412  | 428   | 448   | 497   | 615   |



* 以上结果的单位为TPS，即每秒处理的交易数（Transaction per second）

* 热点更新的TPS与机器规格、并发请求数、更新语句有关，测试结果仅供参考



