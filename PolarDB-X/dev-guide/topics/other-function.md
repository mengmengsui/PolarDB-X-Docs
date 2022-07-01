其他函数 
=========================

本文介绍了PolarDB-X支持的其他函数。

PolarDB-X还支持如下函数：


| 函数名​                | 描述​                             |
|:--------------------|:--------------------------------|
| RAND()              | 返回0到1的随机数                       |
| RELEASE_ALL_LOCKS() | 释放所有锁                           |
| RELEASE_LOCK()      | 释放指定名称的锁                        |
| UUID()              | 返回UUID                          |
| UUID_SHORT()        | 返回值是一个unsigned long long类型的UUID |
| GET_LOCK()          | 获取锁                             |
| IS_USED_LOCK()      | 检查锁是否正在被使用                      |
| IS_FREE_LOCK()      | 检查锁是否被释放                        |



PolarDB-X不支持如下函数：


| 函数名​              | 描述​                           |
|:------------------|:------------------------------|
| ANY_VALUE()       | 选择被分到同一组的数据里第一条数据的指定列值作为返回数据。 |
| DEFAULT(col_name) | 返回字段col_name的默认值。             |
| INET_ATON()       | 返回IPv4地址对应的数值。                |
| INET_NTOA()       | 返回数值对应的IPv4地址。                |
| INET6_ATON()      | 返回IPv6地址对应的数值。                |
| INET6_NTOA()      | 返回数值对应的IPv6地址。                |
| IS_IPV4()         | 判断是否为一个IPv4地址。                |
| IS_IPV4_COMPAT()  | 判断expr是否为一个IPv4兼容地址。          |
| IS_IPV4_MAPPED()  | 判断expr是否为一个IPv4映射地址。          |
| IS_IPV6()         | 判断是否是IPV6地址。                  |
| MASTER_POS_WAIT() | 待当前从库达到这个位置后返回，返回期间执行的事务个数。   |
| NAME_CONST()      | 根据指定的列名创建一列。                  |


