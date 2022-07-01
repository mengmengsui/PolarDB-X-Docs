CONTINUE DDL 
=================================

`CONTINUE DDL`命令用于控制状态为PAUSED或ROLLBACK_PAUSED的DDL任务继续执行。

语法 
-----------------------

JOB_ID可通过`SHOW DDL`命令获取。

```sql
CONTINUE DDL JOB_ID;
```



示例 
-----------------------

```sql
mysql> continue ddl 1359630057722609664;
Query OK, 1 row affected (0.04 sec)
```


