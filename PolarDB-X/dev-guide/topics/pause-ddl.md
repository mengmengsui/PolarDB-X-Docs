PAUSE DDL 
==============================

`PAUSE DDL`命令可用于暂停当前状态为RUNNING或ROLLBACK_RUNNING的DDL任务。

语法 
-----------------------

JOB_ID可通过`SHOW DDL`命令获取。

```sql
PAUSE DDL JOB_ID;
```



示例 
-----------------------

```sql
mysql> pause ddl 1359630950316638208;
Query OK, 1 row affected (0.04 sec)
-- 再次
mysql> show ddl\G;
*************************** 1. row ***************************
           JOB_ID: 1359630950316638208
    OBJECT_SCHEMA: d2
      OBJECT_NAME: t1
           ENGINE: DAG
         DDL_TYPE: CREATE_TABLE
            STATE: PAUSED
         PROGRESS: 0%
       START_TIME: 2021-08-04 14:02:19.781
         END_TIME: 2021-08-04 14:02:35.276
 ELAPSED_TIME(MS): 15495
      PHY_PROCESS: 
BACKFILL_PROGRESS: 0%
1 row in set (0.02 sec)
```


