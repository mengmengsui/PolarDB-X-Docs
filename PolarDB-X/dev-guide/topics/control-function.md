流程控制函数 
===========================

本文介绍了PolarDB-X支持的流程控制函数。

PolarDB-X支持如下流程控制函数：


| 函数名                                                                                                                       | 描述                                                                                                           |
|:--------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------|
| CASE expression WHEN condition1 THEN result1 WHEN condition2 THEN result2 ... WHEN conditionN THEN resultN ELSE resultEND | CASE表示函数开始，END表示函数结束。如果condition1成立，则返回result1，如果condition2成立，则返回result2，当全部不成立则返回result，而当有一个成立之后，后面的就不执行了。 |
| IF(expr,v1,v2)                                                                                                            | 如果表达式expr成立，返回结果v1；否则，返回结果v2。                                                                                |
| IFNULL(v1,v2)                                                                                                             | 如果v1的值不为NULL，则返回v1，否则返回v2。                                                                                   |
| NULLIF(expr1, expr)                                                                                                       | 比较两个字符串，如果字符串expr与expr2相等，返回NULL，否则返回expr1。                                                                  |


