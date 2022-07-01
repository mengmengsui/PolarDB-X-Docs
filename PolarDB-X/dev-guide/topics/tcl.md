TCL语句 
==========================

该语句用于开启事务。数据库事务（Database Transaction）是指作为单个逻辑工作单元执行的一系列操作。事务处理可以用来维护数据库的完整性，保证成批的SQL操作全部执行或全部不执行。

默认情况下，PolarDB-X以开启autocommit的模式运行，也就是每条SQL语句单独构成一个事务（出于性能考虑，跨分片的DML语句默认未开启分布式事务，仅对更新GSI、更新广播表、更新拆分键的DML语句默认开启分布式事务）。用户可以通过执行SET AUTOCOMMIT=0或者显式事务的方式手动开启多条语句组成的交互式事务。

显示事务是用户自定义或用户指定的事务。通过START TRANSACTION，或BEGIN（被作为START TRANSACTION的别名受到支持）语句显示开始，以COMMIT或ROLLBACK语句显示结束。

语法：

```sql
START TRANSACTION
  [transaction_characteristic [, transaction_characteristic] ...]
transaction_characteristic: {
    WITH CONSISTENT SNAPSHOT
  | ISOLATION LEVEL {REPEATABLE READ | READ COMMITTED}
  | READ WRITE
  | READ ONLY
}
BEGIN
COMMIT
ROLLBACK
SET autocommit = {0 | 1}
```



|                                  参数                                   |                                                            说明                                                             |
|-----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| START TRANSACTION [READ ONLY &#124; READ WRITE]                         | 启动新的事务，随后执行的DML语句（即INSERT、UPDATE、DELETE等）直到事务提交时才会生效。READ ONLY子句标记事务以只读方式开启，事务内不允许执行修改操作。READ WRITE子句标记事务以读写方式开启，默认为这种模式。 |
| START TRANSACTION WITH CONSISTENT SNAPSHOT                            | 启动新的事务，如果没有显式指定隔离级别，则为事务设置隔离级别REPEATABLE READ。                                                                            |
| START TRANSACTION ISOLATION LEVEL {REPEATABLE READ &#124; READ COMMITTED} | 启动新的事务，并为事务设置指定的隔离级别。                                                                                                     |
| BEGIN                                                                 | BEGIN被作为START TRANSACTION的别名受到支持。                                                                                         |
| COMMIT                                                                | 提交当前事务。                                                                                                                   |
| ROLLBACK                                                              | 回滚当前事务。                                                                                                                   |
| SET autocommit = {0 &#124; 1}                                             | 为当前会话（session）关闭/开启autocommit模式。                                                                                          |


