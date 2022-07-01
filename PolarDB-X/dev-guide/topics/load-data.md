LOAD DATA 
==============================

本文介绍在PolarDB-X中使用load data进行数据导入的相关限制和注意事项。

注意事项 
-------------------------

* 使用load data进行数据导入时，load data语句并非一个事务，因此各种原因（如数据库宕机等）可能导致部分数据插入，剩余部分未插入。

* 客户端需要开启local-infile。




语法 
-----------------------

```sql
LOAD DATA   
    [LOCAL] 
    INFILE 'file_name' 
    [REPLACE | IGNORE] 
    INTO TABLE tbl_name 
    [CHARACTER SET charset_name] 
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string'] 
        [ENCLOSED BY 'char'] 
        [ESCAPED BY 'char'] 
    ]
    [LINES
        [STARTING BY 'string'] 
        [TERMINATED BY 'string'] 
    ]
    [IGNORE number {LINES | ROWS}] 
    [(col_name_or_mask [, col_name_or_mask] ...)]  
```



参数说明 
-------------------------



|                     参数名称                      |                                                                                                                                          说明                                                                                                                                           |
|-----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LOAD DATA \[LOCAL\] INFILE                    | 文件位于服务端还是client端。                                                                                                                                                                                                                                                                     |
| file_name                                     | 使用相对路径时，为相对于客户端启动时的路径。                                                                                                                                                                                                                                                                |
| REPLACE                                       | 导入数据时，遇到主键重复则强制用当前数据覆盖已有数据。                                                                                                                                                                                                                                                           |
| IGNORE                                        | 导入数据时，遇到主键重复则自动忽略该行。                                                                                                                                                                                                                                                                  |
| \[FIELDS\] TERMINATED BY 'string'             | 定义每行数据的分隔符，默认为\\t。                                                                                                                                                                                                                                                                    |
| \[FIELDS\] ENCLOSED BY 'char'                 | 每列数据的包围符。例如，某一列数据为"test"，定义enclosed by '"'后，导入数据时先将"test"前后的"移除，然后再导入数据。                                                                                                                                                                                                              |
| \[LINES\] TERMINATED BY 'string'              | 定义行分隔符，默认为\\n。                                                                                                                                                                                                                                                                        |
| IGNORE number LINES                           | 导入数据时忽略开始的某几行。例如，`IGNORE 1 LINES`，导入数据时忽略第一行数据。                                                                                                                                                                                                                  |
| (col_name_or_mask \[, col_name_or_mask\] ...) | 1. 设置导入的列，如果不设置，默认按照表中的列顺序来导入数据。  2. 掩盖掉文件中的某些列，使文件中对应列失效，例如，`table test(x int, y int)`，导入文件有三列，导入时使用` (x, @name, y) `则会忽略文件中的第二列，使用第一列填充x，使用第二列填充y。   |



示例 
-----------------------

创建测试表：

```sql
CREATE TABLE test ( a int(11) NOT NULL DEFAULT '0',  b varchar(8) NOT NULL,  PRIMARY KEY (a)  ) DBPARTITION by hash(a);
```



本地待导入文件：

```text
x,y
test1,2
test2,3
test3,4
test4,5
test5,6
test7,8
test8,9
```



load data语句：

```sql
LOAD DATA LOCAL INFILE '~/test.txt' IGNORE INTO TABLE test FIELDS TERMINATED BY ',' LINES STARTING BY 'test' TERMINATED BY '\n' IGNORE 1 LINES;
```



结果如下：

```sql
mysql> select * from test order by a;
+------+------+
| a    | b    |
+------+------+
|    1 | 2    |
|    2 | 3    |
|    3 | 4    |
|    4 | 5    |
|    5 | 6    |
|    7 | 8    |
|    8 | 9    |
+------+------+
7 rows in set (0.02 sec)
```



