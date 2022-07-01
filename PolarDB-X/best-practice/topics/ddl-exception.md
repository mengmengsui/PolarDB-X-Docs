如何正确处理DDL异常 
================================

本文介绍处理DDL异常的常用方法。

概述 
-----------------------

作为一款分布式数据库，PolarDB-X中的一条DDL语句背后隐藏着复杂的数据处理流程。例如创建一张拆分表时，实际上会在多个数据节点上创建很多张物理MySQL表。PolarDB-X的DDL处理框架拥有一定的容错能力，会保证DDL的正确性和一致性。但在特殊情况下，需要手动处理DDL异常。

处理步骤 
-------------------------

下面以创建全局唯一二级索引为例，介绍处理DDL异常的步骤。创建全局唯一二级索引时，会校验列中数据的全局唯一性，如果发现列中数据不唯一，则会导致DDL的执行失败。此时DDL的状态可能变为"PAUSED"或"ROLLBACK_PAUSED"。

1. **查看DDL的执行状态**

   执行`SHOW DDL`语句，查看DDL的执行状态。只有当DDL状态为"PAUSED"或"ROLLBACK_PAUSED"时，才需要手动处理。

   ```sql
   mysql> show ddl\G;
   *************************** 1. row ***************************
              JOB_ID: 1359992239576580096
       OBJECT_SCHEMA: d1
         OBJECT_NAME: t1
              ENGINE: DAG
            DDL_TYPE: ALTER_TABLE
               STATE: PAUSED
   BACKFILL_PROGRESS: 0%
    PHY_DDL_PROGRESS: 100%
            PROGRESS: 80%
          START_TIME: 2021-08-05 13:57:57.852
            END_TIME: 2021-08-05 13:58:30.804
    ELAPSED_TIME(MS): 32952
         PHY_PROCESS: 
          CANCELABLE: true
   1 row in set (0.03 sec)
   ```

   

2. **判断错误原因**

   通常DDL异常发生时，会直接返回错误信息。有时DDL的错误信息可能难以获取（例如DDL是异步执行的），可以通过`SHOW DDL RESULT`语句查看近期执行过的DDL。在本示例中，从返回的错误信息中看到出现了UNIQUE KEY的冲突。

   ```sql
   mysql> alter table t1 add unique global INDEX `idx_c2`(`c2`) DBPARTITION BY HASH(`c2`) tbpartition by hash(c2) tbpartitions 3;
   ERROR 3009 (HY000): [12dfa9bc5c800000][30.39.226.95:8527][d1]Failed to execute the DDL task. Caused by: ERR-CODE: [TDDL-5321][ERR_GLOBAL_SECONDARY_INDEX_BACKFILL_DUPLICATE_ENTRY] Duplicated entry '100018' for key 'PRIMARY'
   mysql> show ddl result\G;
   *************************** 1. row ***************************
           JOB_ID: 1359992239576580096
      SCHEMA_NAME: d1
      OBJECT_NAME: t1
         DDL_TYPE: ALTER_TABLE
      RESULT_TYPE: ERROR
   RESULT_CONTENT: Failed to execute the DDL task. Caused by: ERR-CODE: [TDDL-5321][ERR_GLOBAL_SECONDARY_INDEX_BACKFILL_DUPLICATE_ENTRY] Duplicated entry '100018' for key 'PRIMARY' 
   ```

   

3. **恢复或取消DDL**

   大多数情况下，DDL出错时都能够自动恢复或取消。特殊情况需要手动处理DDL异常。各个DDL的容错策略可能不同，例如`CREATE TABLE`语句的容错策略是：自动尝试恢复，多次失败则自动取消。

   可以手动恢复或取消DDL任务。
   * 手动恢复：使用`CONTINUE DDL`语句可以恢复DDL任务。
   
   * 手动取消：使用`CANCEL DDL`语句可以取消DDL任务。有些DDL任务可能是无法取消的，可以通过`SHOW DDL`语句返回的"CANCELABLE"字段查看是否可以取消。
   

   

   ```sql
   -- 处理完UNIQUE KEY的数据冲突后，可以使用continue ddl语句继续执行
   mysql> continue ddl 1359992239576580096;
   Query OK, 1 row affected (1.63 sec)
   -- 如果不想再继续添加这个全局唯一二级索引，则可以直接取消任务
   mysql> cancel ddl 1359992239576580096;
   Query OK, 1 row affected (0.03 sec)
   ```

   

4. **检查表一致性和正确性**

   DDL执行成功后，建议使用以下语句检查表的一致性和表结构的正确性：
   * 使用`CHECK TABLE`语句检查表的一致性：如果返回结果为OK，说明表已经处于一致性状态，可以正常使用；如果返回报错信息，提示表处于不一致状态，请根据具体报错信息进行处理，或者在社区中寻求帮助。
   
   * 使用`SHOW CREATE TABLE`语句查看表结构的正确性。
   

   

   ```sql
   mysql> check table t1;
   +-------+-------+----------+----------+
   | TABLE | OP    | MSG_TYPE | MSG_TEXT |
   +-------+-------+----------+----------+
   | d1.t1 | check | status   | OK       |
   +-------+-------+----------+----------+
   1 row in set (0.05 sec)
   mysql> show create table t1\G;
   *************************** 1. row ***************************
          Table: t1
   Create Table: CREATE TABLE `t1` (
       `c1` bigint(20) NOT NULL AUTO_INCREMENT BY GROUP,
       `c2` bigint(20) DEFAULT NULL,
       `c3` bigint(20) DEFAULT NULL,
       PRIMARY KEY USING BTREE (`c1`),
       UNIQUE GLOBAL KEY `idx_c2` (`c2`) COVERING (`c1`) DBPARTITION BY HASH(`c2`) TBPARTITION BY HASH(`c2`) TBPARTITIONS 3
   ) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4  dbpartition by hash(`c1`) tbpartition by hash(`c1`) tbpartitions 3
   1 row in set (0.02 sec)
   ```

   

   



