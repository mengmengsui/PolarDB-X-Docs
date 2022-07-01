信息函数 
=========================

信息函数用于返回动态的数据库信息，本文介绍了PolarDB-X支持及不支持的信息函数。

支持函数 
-------------------------

PolarDB-X支持如下信息函数：


| 函数名                          | 描述                           |
|:-----------------------------|:-----------------------------|
| CONNECTION_ID()              | 返回唯一的连接ID                    |
| CURRENT_USER(), CURRENT_USER | 返回当前用户                       |
| DATABASE()                   | 返回当前数据库                      |
| LAST_INSERT_ID()             | 返回最近生成的AUTO_INCREMENT值       |
| SCHEMA()                     | 和DATABASE()同义                |
| SESSION_USER()               | 和 USER()同义                   |
| SYSTEM_USER()                | 和 USER()同义                   |
| USER()                       | 返回当前用户                       |
| VERSION()                    | 返回当前版本号                      |
| BENCHMARK()                  | 重复执行一个表达式                    |
| CHARSET()                    | 返回当前字符集                      |
| COLLATION()                  | 返回当前Collation                |
| FOUND_ROWS()                 | 返回上次查询结果集的记录数                |
| ROW_COUNT()                  | 返回上一条SQL语句，对表数据进行修改操作后影响的记录数 |



不支持函数 
--------------------------

与 MySQL5.7 相比，PolarDB-X暂不支持如下信息函数：


|       函数       |       描述        |
|----------------|-----------------|
| COERCIBILITY() | 返回字符串参数的整序可压缩性值 |


