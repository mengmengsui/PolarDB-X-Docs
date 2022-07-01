慢SQL相关 
===========================

本文介绍了慢SQL相关的SHOW语句。

* SHOW \[FULL\] SLOW \[WHERE expr\] \[limit expr\]

* SHOW \[FULL\] PHYSICAL\\_SLOW \[WHERE expr\] \[limit expr\]

* CLEAR SLOW




SHOW \[FULL\] SLOW \[WHERE expr\] \[limit expr\] 
---------------------------------------------------------------------

执行时间超过1秒的SQL语句是慢SQL，逻辑慢SQL是指应用发送到PolarDB-X的慢SQL。

* `SHOW SLOW`: 查看自PolarDB-X启动或者上次执行`CLEAR SLOW`以来最慢的100条逻辑慢SQL;
  **说明** 此处记录的是最慢的100个，缓存在PolarDB-X 系统中，当实例重启或者执行`CLEAR SLOW`时会丢失。
  
* `SHOW FULL SLOW`: 查看实例启动以来记录的所有逻辑慢SQL（持久化到PolarDB-X的内置数据库中）。该记录数有一个上限（具体数值跟购买的实例规格相关），PolarDB-X会滚动删除比较老的慢SQL语句。实例的规格为4C4G时，最多记录10000条慢SQL语句（包括逻辑慢SQL和物理慢SQL）；实例的规格为8C8G时，最多记录20000条慢SQL语句（包括逻辑慢SQL和物理慢SQL），其它规格依此类推。




示例：

```sql
mysql> show slow where execute_time > 1000 limit 1;
+-----------+---------------------+--------------+------------+-----------+
| HOST      | START_TIME          | EXECUTE_TIME | AFFECT_ROW | SQL       |
+-----------+---------------------+--------------+------------+-----------+
| 127.0.0.1 | 2016-03-16 13:02:57 |         2785 |          7 | show rule |
+-----------+---------------------+--------------+------------+-----------+
1 row in set (0.02 sec)
```



重要列详解：

* **HOST** ：来源IP；

* **START_TIME** ：执行开始时间；

* **EXECUTE_TIME** ：执行时间；

* **AFFECT_ROW** ：对于DML语句是影响行数；对于查询语句是返回的记录数。




SHOW \[FULL\] PHYSICAL\\_SLOW \[WHERE expr\] \[limit expr\] 
--------------------------------------------------------------------------------

执行时间超过1秒的SQL语句是慢SQL，物理慢SQL是指PolarDB-X发送到RDS的慢SQL。

* `SHOW PHYSICAL_SLOW`: 查看自PolarDB-X启动或者上次执行`CLEAR SLOW`以来最慢的100条物理慢SQL（注意，这里记录的是最慢的100个，缓存在PolarDB-X系统中，当实例重启或者执行CLEAR SLOW时会丢失）;

* `SHOW FULL PHYSICAL_SLOW`: 查看实例启动以来记录的所有物理慢SQL（持久化到PolarDB-X的内置数据库中）。该记录数有一个上限（具体数值跟购买的实例规格相关），PolarDB-X会滚动删除比较老的慢SQL语句。实例的规格如果是4C4G，最多记录10000条慢SQL语句（包括逻辑慢SQL和物理慢SQL）；实例的规格如果是8C8G，最多记录20000条慢SQL语句（包括逻辑慢SQL和物理慢SQL），其它规格依此类推。




示例：

```sql
mysql> show physical_slow;
+----------------+-----------------------------------+---------------------+--------------+------------------+-------------------------+------------------------+------------+-----------------+
| GROUP_NAME     | DBKEY_NAME                        | START_TIME          | EXECUTE_TIME | SQL_EXECUTE_TIME | GETLOCK_CONNECTION_TIME | CREATE_CONNECTION_TIME | AFFECT_ROW | SQL             |
+----------------+-----------------------------------+---------------------+--------------+------------------+-------------------------+------------------------+------------+-----------------+
| TDDL5_00_GROUP | db218249098_sqa_zmf_tddl5_00_3309 | 2016-03-16 13:05:38 |         1057 |             1011 |                       0 |                      0 |          1 | select sleep(1) |
+----------------+-----------------------------------+---------------------+--------------+------------------+-------------------------+------------------------+------------+-----------------+
1 row in set (0.01 sec)
```



重要列详解：

* **GROUP_NAME** ：数据库分组；

* **START_TIME** ：执行开始时间；

* **EXECUTE_TIME** ：执行时间；

* **AFFECT_ROW** ：对于DML语句是影响行数；对于查询语句是返回的记录数。




CLEAR SLOW 
-------------------------------

清空自PolarDB-X启动或者上次执行`CLEAR SLOW`以来最慢的100条逻辑慢SQL和最慢的100条物理慢SQL。

示例：

```sql
mysql> clear slow;
Query OK, 0 rows affected (0.00 sec)
```


**说明** `SHOW SLOW`和`SHOW PHYSICAL_SLOW`展示的是最慢的100个SQL，如果长时间未执行`CLEAR SLOW`，可能是非常老的SQL，一般执行过SQL优化之后，建议执行`CLEAR SLOW`后，等待系统运行一段时间，再查看慢SQL的优化效果。
