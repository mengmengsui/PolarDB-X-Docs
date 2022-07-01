CANCEL DDL 
===============================

`CANCEL DDL`命令可用于取消状态为RUNNING或PAUSED的DDL任务。取消之后，当前DDL任务已发生的变更将会回滚，数据表将恢复到DDL任务执行之前的状态。

语法 
-----------------------

JOB_ID可通过`SHOW DDL`命令获取。

```sql
CANCEL DDL JOB_ID;
```



示例 
-----------------------

```sql
mysql> cancel ddl 1359628009396502528;
Query OK, 1 row affected (0.03 sec)
```


