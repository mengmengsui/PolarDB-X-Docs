转换函数 
=========================

本文介绍了PolarDB-X支持的转换函数。

PolarDB-X支持如下转换函数：


| 函数名                 | 描述​             |                                                                                                                                      示例                                                                                                                                       |
|:--------------------|:----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| BINARY              | 将字符串s转换为二进制字符串  | `SELECT BINARY "RUNOOB"; ` 输出： `-> RUNOOB`                                                                                                                                                                                               |
| CAST(x AS type)     | 转换数据类型          | 字符串日期转换为日期： `SELECT CAST("2017-08-29" AS DATE);` 输出： `-> 2017-08-29`                                                                                                                                                     |
| CONVERT(s USING cs) | 函数将字符串s的字符集变成cs | * `SELECT CHARSET('ABC')` 输出： `->utf-8 `   * `SELECT CHARSET(CONVERT('ABC' USING gbk))` 输出： `->gbk`    |


