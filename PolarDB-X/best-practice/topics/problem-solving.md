如何快速定位及解决数据库问题 
===================================

本文介绍了数据库发生故障时的快速判断方法和解决办法。

如何定位系统瓶颈是否在数据库上 
------------------------------------

* **通过Processlist来判断**

  执行以下语句，显示PolarDB-X上所有正在执行的SQL语句。

  ```sql
  SHOW PROCESSLIST WHERE INFO IS NOT NULL
  ```

  

  一般情况下，语句堆积会伴随着数据库卡慢一起出现，因此如果该语句的显示结果中没有大量执行时间大于0的语句，则基本可以断定问题不在数据库层面，反之，则说明数据库可能存在瓶颈。
  

* **通过堆栈信息来判断**

  应用与数据库之间通过TCP协议进行交互，如果数据库层出现瓶颈，则会产生应用将请求通过socket发送给了数据库，但是数据库不返回结果的情况，此时socket会阻塞在read方法上。因此我们可以通过应用当前的堆栈信息来判断是否在数据库层面发生了阻塞。本文以Java应用为例说明：
  1. 通过jstack命令dump堆栈信息。
  
  2. 在dump出的信息中搜索mysql驱动等待请求返回的堆栈，内容如下：

     ```java
     at java.net.SocketInputStream.socketRead0(Native Method)
     at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
     at java.net.SocketInputStream.read(SocketInputStream.java:171)
     at java.net.SocketInputStream.read(SocketInputStream.java:141)
     at com.mysql.jdbc.util.ReadAheadInputStream.fill(ReadAheadInputStream.java:101)
     at com.mysql.jdbc.util.ReadAheadInputStream.readFromUnderlyingStreamIfNecessary(ReadAheadInputStream.java:144)
     at com.mysql.jdbc.util.ReadAheadInputStream.read(ReadAheadInputStream.java:174)
     - locked <0x00000002eb8f2d98> (a com.mysql.jdbc.util.ReadAheadInputStream)
     at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3183)
     at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3659)
     at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3649)
     at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4090)
     at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:972)
     at com.mysql.jdbc.MysqlIO.readAllResults(MysqlIO.java:2497)
     at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2870)
     at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2806)
     ```

     
  

  

  如果有大量的线程的堆栈情况如上例所示，则代表大量线程阻塞在等待数据库返回，说明瓶颈可能在数据库层面，反之，则应重点排查应用本身是否存在瓶颈。
  




数据库问题快速处置 
------------------------------

在通过上述方法判断数据库存在瓶颈之后，推荐依次使用以下方法进行快速恢复。

**方法一：KILL所有语句**

如果Processlist中显示堆积了很多SQL，建议立即KILL掉所有正在执行的语句，PolarDB-X提供了如下指令进行这个操作：

```sql
KILL "ALL"
```



该语句会KILL掉计算节点与数据节点之间的每一个连接，从而达到结束掉所有语句的效果。

**方法二：重启应用**

执行方法一后，等待一段时间如果再次产生语句堆积，建议重启应用，避免应用因为处于某种错误的状态，不断的重试高代价的SQL。

**方法三：SQL限流**

方法二依然无法解决问题之后，建议使用PolarDB-X的CCL_RULES（限流功能）。

1. 执行`SHOW FULL PROCESSLIST`命令，找到占比比较高的SQL的模板ID。

   ```sql
   +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
   | ID | USER          | HOST            | DB       | COMMAND                       | TIME | STATE | INFO                  | SQL_TEMPLATE_ID |
   +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
   |  2 | polardbx_root | ***.*.*.*:62787 | polardbx | Query                         |    0 |       | show full processlist | NULL            |
   |  1 | polardbx_root | ***.*.*.*:62775 | polardbx | Query(Waiting-selectrulereal) |   12 |       | select 1              | 9037e5e2        |
   +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
   2 rows in set (0.08 sec)
   ```

   

2. 通过模板ID对该类型的SQL进行限流，例如：

   ```sql
   CREATE CCL_RULE IF NOT EXISTS `test` ON *.* TO 'ccltest'@'%'
   FOR SELECT
   FILTER BY TEMPLATE('9037e5e2')
   WITH MAX_CONCURRENCY=10;
   ```

   


 **方法四：重启数据库**

以上方法都无效的情况下，请重启数据库。
