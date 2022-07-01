DDL常见问题 
============================

本文汇总了PolarDB-X上常见的DDL执行问题。

创建表执行出错 
----------------------------

DDL的执行是一个分布式处理过程，出错可能导致各个分片表结构不一致，所以需要进行手动清理，详细操作步骤如下：

1. PolarDB-X会提供基本的错误描述信息，比如语法错误等。如果错误信息太长，则会提示您使用`SHOW WARNINGS`来查看每个分库执行失败的原因。

2. 执行`SHOW TOPOLOGY`命令来查看物理表的拓扑结构。

   ```sql
   SHOW TOPOLOGY FROM multi_db_multi_tbl;
   +------+-----------------+-----------------------+
   | ID   | GROUP_NAME      | TABLE_NAME            |
   +------+-----------------+-----------------------+
   |    0 | corona_qatest_0 | multi_db_multi_tbl_00 |
   |    1 | corona_qatest_0 | multi_db_multi_tbl_01 |
   |    2 | corona_qatest_0 | multi_db_multi_tbl_02 |
   |    3 | corona_qatest_1 | multi_db_multi_tbl_03 |
   |    4 | corona_qatest_1 | multi_db_multi_tbl_04 |
   |    5 | corona_qatest_1 | multi_db_multi_tbl_05 |
   |    6 | corona_qatest_2 | multi_db_multi_tbl_06 |
   |    7 | corona_qatest_2 | multi_db_multi_tbl_07 |
   |    8 | corona_qatest_2 | multi_db_multi_tbl_08 |
   |    9 | corona_qatest_3 | multi_db_multi_tbl_09 |
   |   10 | corona_qatest_3 | multi_db_multi_tbl_10 |
   |   11 | corona_qatest_3 | multi_db_multi_tbl_11 |
   +------+-----------------+-----------------------+
   12 rows in set (0.21 sec)
   ```

   

3. 执行`CHECK TABLE tablename`语句来查看逻辑表是否创建成功。例如下面的例子展示了multi_db_multi_tbl的某个物理分表没有创建成功时的情况。

   ```sql
   mysql> check table multi_db_multi_tbl;
   +-------------------------------------------------+-------+----------+---------------------------------------------------------------------------+
   | TABLE                                           | OP    | MSG_TYPE | MSG_TEXT                                                                  |
   +-------------------------------------------------+-------+----------+---------------------------------------------------------------------------+
   | andor_mysql_qatest. multi_db_multi_tbl | check | Error    | Table 'corona_qatest_0. multi_db_multi_tbl_02' doesn't exist |
   +-------------------------------------------------+-------+----------+---------------------------------------------------------------------------+
   1 row in set (0.16 sec)
   ```

   

4. 使用幂等的方式重新执行建表或删表操作，该操作会创建或删除剩余的物理表。

   ```sql
   CREATE TABLE IF NOT EXISTS table1
   (id int, name varchar(30), primary key(id))
   dbpartition by hash(id);  
   DROP TABLE IF EXISTS table1;
   ```

   




建索引失败或加列失败 
-------------------------------

建索引失败或加列失败的处理方法跟上面建表失败的处理类似，详情请参见[如何正确处理DDL异常](../../best-practice/topics/ddl-exception.md)。
