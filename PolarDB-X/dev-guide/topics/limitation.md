开发限制 
=========================

本文介绍了PolarDB-X数据库开发过程中的使用限制。

PolarDB-X高度兼容MySQL协议和语法，但由于分布式数据库和单机数据库存在较大的架构差异，存在SQL使用限制。本文将介绍数据库开发过程中使用限制。

常见标识符限制 
----------------------------



|      类型       | 最大字符长度 |        取值范围         |
|---------------|--------|---------------------|
| Database      | 32     | 大小写字母、数字和下划线（_）。    |
| Sequence      | 128    | 任意符合Unicode编码方式的字符。 |
| Table         | 64     | 任意符合Unicode编码方式的字符。 |
| Column        | 64     | 任意符合Unicode编码方式的字符。 |
| Partition     | 64     | 任意符合Unicode编码方式的字符。 |
| Partition Key | 64     | 任意符合Unicode编码方式的字符。 |
| View          | 64     | 任意符合Unicode编码方式的字符。 |
| Sequence      | 64     | 任意符合Unicode编码方式的字符。 |
| Variables     | 64     | 任意符合Unicode编码方式的字符。 |
| Constraint    | 64     | 任意符合Unicode编码方式的字符。 |



资源使用限制 
---------------------------



|  资源  |         类型         |                数量                |
|------|--------------------|----------------------------------|
|      | Database           | 最多包含32个数据库。                      |
|      | Table              | 每个数据库最多包含8192张表。                 |
|      | Partition          | 每张表最多包含8192个分区。                  |
|      | Column             | 每张表最多包含1017列。                    |
|      | Sequence           | 每个数据库最多支持创建16384个序列。             |
|      | View               | 每个数据库最多支持创建8192个视图。              |
|      | Global Index       | 每张表最多支持创建32个全局索引。                |
|      | User               | 单个数据库最大支持2048个用户，用户名最大长度255。    |
| 物理资源 | 数据库中单个计算节点最多支持的连接数 | 默认不要超过20000。                     |
| 物理资源 | 数据库中最多支持增加的计算节点个数  | 一次性默认最多支持增加99个节点，如需添加更多，请联系技术支持。 |
| 物理资源 | 数据库中最多支持增加的存储节点个数  | 一次性默认最多支持增加99个节点，如需添加更多，请联系技术支持。 |



SQL语法使用限制 
------------------------------



| SQL语法    | 操作                                                         | 使用约束                                    |
| ---------- | ------------------------------------------------------------ | ------------------------------------------- |
| 自定义操作 | 自定义函数                                                   | 暂不支持。                                  |
| 自定义操作 | 自定义类型                                                   | 暂不支持。                                  |
| 自定义操作 | 存储过程                                                     | 暂不支持。                                  |
| 自定义操作 | 触发器                                                       | 暂不支持。                                  |
| 自定义操作 | 游标                                                         | 暂不支持。                                  |
| 自定义操作 | 视图                                                         | 暂不支持。                                  |
| DDL        | CREATE TABLE ... LIKE ...                                    | 暂不支持拆分表。                            |
| DDL        | CREATE TABLE ... SELECT ...                                  | 暂不支持拆分表。                            |
| DDL        | RENAME TABLE                                                 | 暂不支持同时RENAME多表。                    |
| DDL        | ALTER TABLE                                                  | 暂不支持ALTER TABLE修改拆分<br />字段类型。 |
| DDL        | Merge/Split/Add/Drop 二级分区表                              | 暂不支持。                                  |
| DDL        | Merge/Split/Add/Drop 索引分区表                              | 暂不支持。                                  |
| DML        | STRAIGHT_JOIN                                                | 暂不支持。                                  |
| DML        | NATURAL JOIN                                                 | 暂不支持。                                  |
| DML        | INSERT DELAYED                                               | 暂不支持。                                  |
| DML        | 对变量的引用和操作<br />（例如SET @c=1, @d=@c+1; SELECT @c, @d） | 暂不支持。                                  |
| DML        | LOAD XML                                                     | 暂不支持。                                  |
| DQL        | HAVING子句中包含子查询                                       | 暂不支持。                                  |
| DQL        | JOIN ON子句中包含子查询                                      | 暂不支持。                                  |
| DQL        | 等号操作行符的标量子查询<br />（The Subquery as Scalar Operand） | 暂不支持。                                  |
| 数据库管理 | SHOW WARNINGS                                                | 暂不支持LIMIT和COUNT的组合。                |
| 数据库管理 | SHOW ERRORS                                                  | 暂不支持LIMIT和COUNT的组合。                |
| 数据库管理 | HELP                                                         | 暂不支持。                                  |
| 运算符     | :=                                                           | 暂不支持。                                  |
| 函数       | 全文检索函数                                                 | 暂不支持。                                  |
| 函数       | XML函数                                                      | 暂不支持。                                  |
| 函数       | GTID函数                                                     | 暂不支持。                                  |
| 类型       | 空间类型(GEOMETRY/LINESTRING...)                             | 暂不支持。                                  |
| 类型       | Json类型                                                     | 暂不支持做分区键。                          |
| 关键字     | MILLISECOND                                                  | 暂不支持。                                  |
| 关键字     | MICROSECOND                                                  | 暂不支持。                                  |

