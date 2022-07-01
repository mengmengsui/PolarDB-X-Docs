聚合函数 
=========================

本文介绍了PolarDB-X支持及不支持的聚合函数。

支持函数 
-------------------------

PolarDB-X支持如下聚合函数：


| 函数名                        | 描述                                |
|:---------------------------|:----------------------------------|
| AVG(expression)            | 返回一个表达式的平均值，expression是一个字段。      |
| COUNT(expression)          | 返回查询的记录总数，expression参数是一个字段或者\*号。 |
| COUNT(DISTINCT expression) | 返回去重后查询的记录总数。                     |
| MAX(expression)            | 返回字段expression中的最大值。              |
| MIN(expression)            | 返回字段expression中的最小值。              |
| SUM(expression)            | 返回字段expression加起来的值。              |
| BIT_OR(expression)         | 以或方式聚合expression                  |
| BIT_XOR(expression)        | 以异或方式聚合expression                 |
| BIT_AND(expression)        | 以与方式聚合expression                  |
| GROUP_CONCAT(expression))  | 拼凑expression                      |



不支持函数 
--------------------------

与MySQL5.7相比，PolarDB-X暂不支持如下聚合函数：


|      函数名       |           描述            |
|----------------|-------------------------|
| STD()，STDDEV() | 返回标准差。                  |
| STDDEV_POP()   | 计算总体标准偏离，并返回总体变量的平方根。   |
| STDDEV_SAMP()  | 计算累积样本标准偏离，并返回总体变量的平方根。 |


