SHOW METADATA LOCK 
=======================================

本文将介绍如何在PolarDB-X上使用SHOW METADATA LOCK语句查询持有锁的事务。

背景信息 
-------------------------

PolarDB-X在创建全局二级索引时使用了内建的METADATA LOCK，保证事务以及数据的一致性。在已有表上建立全局二级索引通常需要较长的时间，若此时同时存在持有锁的事务在运行则可能出现SCHEMA变更等待事务完成的情况。此时您可以使用SHOW METADATA LOCK语句查询持有锁的事务以及对应正在执行的SQL语句，方便您排查阻塞SCHEMA变更的长时间事务。

**说明** PolarDB-X支持Online Schema Change，添加全局二级索引过程中，会发生4次元数据版本切换，其中有两次会先获取METADATA LOCK的写锁加载元数据完成后立即解锁，其余的时间均不会持有METADATA LOCK的写锁。

语法 
-----------------------

```sql
SHOW METADATA {LOCK | LOCKS} [schema_name[.table_name]]
```



`schema_name`和`tbl_name`是可选的，用于过滤显示的数据库名或表名。

```sql
show metadata lock; # 显示该节点上所有持有metadata lock的连接
show metadata lock xxx_db; # 显示该节点上 xxx_db 中所有持有metadata lock的连接
show metadata lock xxx_db.tb_name; # 显示该节点上 xxx_db 中 tb_name 上所有持有metadata lock的连接
```



示例 
-----------------------

```sql
mysql> show metadata lock;
+---------+--------+-----------------+---------------------+--------------+------------------+-----------------+----------+-------------------------------------+-----------------------------------------------+
| CONN_ID | TRX_ID | TRACE_ID        | SCHEMA              | TABLE        | TYPE             | DURATION        | VALIDATE | FRONTEND                            | SQL                                           |
+---------+--------+-----------------+---------------------+--------------+------------------+-----------------+----------+-------------------------------------+-----------------------------------------------+
| 4       |      0 | f88cf71cbc00001 | XXXX_DRDS_LOCAL_APP | full_gsi_ddl | MDL_SHARED_WRITE | MDL_TRANSACTION |        1 | XXXX_DRDS_LOCAL_APP@127.0.0.1:54788 | insert into `full_gsi_ddl` (id) VALUE (null); |
| 5       |      0 | f88cf71cbc00000 | XXXX_DRDS_LOCAL_APP | full_gsi_ddl | MDL_SHARED_WRITE | MDL_TRANSACTION |        1 | XXXX_DRDS_LOCAL_APP@127.0.0.1:54789 | insert into `full_gsi_ddl` (id) VALUE (null); |
+---------+--------+-----------------+---------------------+--------------+------------------+-----------------+----------+-------------------------------------+-----------------------------------------------+
2 rows in set (0.00 sec)
```


**说明** 该语句仅用于显示已持有锁的连接，不显示等待锁的连接。


|    列名    |      说明       |
|----------|---------------|
| CONN_ID  | 持有锁的连接ID      |
| TRX_ID   | 持有锁的事务ID      |
| TRACE_ID | 持有锁的SQL的跟踪 ID |
| SCHEMA   | 库名            |
| TABLE    | 表名            |
| TYPE     | 持有锁类型         |
| DURATION | 持有锁的周期        |
| VALIDATE | 是否有效          |
| FRONTEND | 前端连接信息        |
| SQL      | 持有锁的SQL语句     |
[列名说明]


