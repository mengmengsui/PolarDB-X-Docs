SHOW INDEX 
===============================

您可以使用SHOW INDEX语句查看PolarDB-X表上的局部索引和全局索引信息。

语法 
-----------------------

```sql
SHOW {INDEX | INDEXES | KEYS}
{FROM | IN} tbl_name
[{FROM | IN} db_name]
[WHERE expr]
```



示例 
-----------------------

```sql
mysql> show index from t_order;
+--------------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| TABLE   | NON_UNIQUE | KEY_NAME  | SEQ_IN_INDEX | COLUMN_NAME    | COLLATION | CARDINALITY | SUB_PART | PACKED | NULL | INDEX_TYPE | COMMENT  | INDEX_COMMENT |
+--------------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
| t_order |          0 | PRIMARY   |            1 | id             | A         |           0 |     NULL | NULL   |      | BTREE      |          |               |
| t_order |          1 | l_i_order |            1 | order_id       | A         |           0 |     NULL | NULL   | YES  | BTREE      |          |               |
| t_order |          0 | g_i_buyer |            1 | buyer_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | INDEX    |               |
| t_order |          1 | g_i_buyer |            2 | id             | NULL      |           0 |     NULL | NULL   |      | GLOBAL     | COVERING |               |
| t_order |          1 | g_i_buyer |            3 | order_id       | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
| t_order |          1 | g_i_buyer |            4 | order_snapshot | NULL      |           0 |     NULL | NULL   | YES  | GLOBAL     | COVERING |               |
+--------------+------------+-----------+--------------+----------------+-----------+-------------+----------+--------+------+------------+----------+---------------+
6 rows in set (0.01 sec)
```



|      列名       |                                                                                              说明                                                                                              |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TABLE         | 表名                                                                                                                                                                                           |
| NON_UNIQUE    | 是否为唯一约束全局二级索引，取值范围如下： * 1：普通全局二级索引   * 0：唯一约束全局二级索引                                                     |
| KEY_NAME      | 索引名                                                                                                                                                                                          |
| SEQ_IN_INDEX  | 索引列在索引中的序号，取值从1开始。                                                                                                                                                                           |
| COLUMN_NAME   | 索引列名。                                                                                                                                                                                        |
| COLLATION     | 排序方式，取值范围如下： * A：升序  * D：降序  * NULL：不排序                               |
| CARDINALITY   | 预计的唯一值数目                                                                                                                                                                                     |
| SUB_PART      | 索引前缀（NULL索引前缀为整个列）。                                                                                                                                                                          |
| PACKED        | 字段压缩信息（NULL表示没有压缩）。                                                                                                                                                                          |
| NULL          | 是否允许空。                                                                                                                                                                                       |
| INDEX_TYPE    | 索引类型，取值范围如下： * NULL（即未指定）  * BTREE  * HASH                            |
| COMMENT       | 索引信息，取值范围如下： * NULL：局部索引  * INDEX：全局二级索引的索引列  * COVERING：全局二级索引的覆盖列   |
| INDEX_COMMENT | 其他信息                                                                                                                                                                                         |
[列名说明]


