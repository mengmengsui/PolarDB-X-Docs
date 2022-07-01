SELECT 
===========================

SELECT用于从一个或多个表中查询数据。

语法 
-----------------------

```sql
SELECT
    [ALL | DISTINCT]
    select_expr [, select_expr ...]
    [FROM table_references
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [FOR UPDATE]
```



SELECT子句说明：

* select_expr：表示查询的列，SELETE必须至少有一个select_expr。

* table_references：表示从哪些表中获取数据。

* WHERE子句：指定查询条件，即从表中获取满足`where_condition`的行，若没有指定，则获取所有行。

* GROUP BY子句：支持列名，表达式以及输出列中的位置引用。

* HAVING子句：与WHERE类似，不同点在于可以使用聚合函数。

* ORDER BY：指定排序，支持列名、表达式以及输出列中的位置引用，同时支持指定排序方向，ASC（升序）或 DESC（降序）。

* LIMIT/OFFSET：限定输出结果集的偏移量和大小。LIMIT 接受一个或两个数字参数。参数必须是一个整数常量。如果给定两个参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目。初始记录行的偏移量是 0(而不是 1)： 为了与PostgreSQL兼容，MySQL也支持句法： LIMIT # OFFSET #。

* FOR UPDATE：对查询结果所有行加排他锁，以阻止其他事务的并发修改，或阻止在某些事务隔离级别时的并发读取。




注意事项 
-------------------------

* 不支持在HAVING中使用适用于WHERE子句的表达式，如下SQL1应改写为SQL2。SQL1：

  ```sql
  SELECT col_name FROM tbl_name HAVING col_name > 0;
  ```

  

  SQL2：

  ```sql
  SELECT col_name FROM tbl_name WHERE col_name > 0;
  ```

  

  

* HAVING子句可以引用聚合函数，但WHERE子句不可以。

  ```sql
  SELECT user, MAX(salary) FROM users
  GROUP BY user HAVING MAX(salary) > 10;
  ```

  

* LIMIT若有两个参数，第一个参数表示返回第一行的偏移量，第二个参数表示返回的行数；若仅有一个参数，则表示返回的行数，默认偏移量为0。

* GROUP BY子句不支持ASC和DESC。

* 同时存在GROUP BY和ORDER BY时，ORDER BY后面的表达式必须在SELECT表达式或GROUP BY表达式中，如不支持以下SQL。

  ```sql
  SELECT user FROM users GROUP BY age ORDER BY salary;     
  ```

  

* 暂不支持ORDER BY子句中使用聚合函数以及包含聚合函数的表达式，可将表达式作为`select_expr`，并赋予别名，在ORDER BY子句中引用该别名。

* 暂不支持以空字符串作为别名。




JOIN 
-------------------------

PolarDB-X支持在SELECT语句的table_references中使用如下JOIN语法： 

```sql
table_references:
    escaped_table_reference [, escaped_table_reference] ...
escaped_table_reference:
    table_reference
  | { OJ table_reference }
table_reference:
    table_factor
  | join_table
table_factor:
    [schema_name.]tbl_name [[AS] alias] [index_hint_list]
  | table_subquery [AS] alias
  | ( table_references )
join_table:
    table_reference [INNER | CROSS] JOIN table_factor [join_condition]
  | table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_condition
join_condition:
    ON conditional_expr
  | USING (column_list)
index_hint_list:
    index_hint [, index_hint] ...
index_hint:
    USE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] ([index_list])
  | IGNORE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] (index_list)
  | FORCE {INDEX|KEY}
      [FOR {JOIN|ORDER BY|GROUP BY}] (index_list)
index_list:
    index_name [, index_name] ...
```



使用JOIN语句时，应考虑如下因素：

* JOIN、CROSS JOIN与INNER JOIN语法是等价的，这种设定与MySQL保持一致。

* 如果INNER JOIN没有ON条件，其与"逗号"连接是等价的，均表示笛卡尔积。如下两条SQL等价：

  ```sql
  SELECT * FROM t1 INNER JOIN t2 WHERE t1.id > 10
  SELECT * FROM t1, t2 WHERE t1.id > 10
  ```

  

* `USING(column_list)`指定连接两表中都存在的列名，PolarDB-X会按照这些列构建等值条件。如下两条SQL等价：

  ```sql
  a LEFT JOIN b USING(c1, c2)
  a LEFT JOIN b ON a.c1 = b.c1 AND a.c2 = b.c2
  ```

  

* JOIN的优先级高于"逗号"操作符，对于连接表达式t1, t2 JOIN t3会转换为(t1, (t2 JOIN t3)，而不是((t1, t2) JOIN t3)。

* 外连接LEFT/RIGHT JOIN必须有ON条件。

* index_hint用于告知MySQL使用哪个索引，PolarDB-X会将该Hint下推至底层MySQL。

* 暂不支持STRAIGHT_JOIN和NATURAL JOIN。




UNION 
--------------------------

PolarDB-X支持如下UNION语法：

```sql
SELECT ...
UNION [ALL | DISTINCT] SELECT ...
[UNION [ALL | DISTINCT] SELECT ...]
```


**说明** 对于UNION中的每个SELECT，PolarDB-X暂不支持使用多个同名的列。例如以下SQL的SELECT中存在重复的列名，暂不支持。

```sql
SELECT id, id, name FROM t1 UNION SELECT pk, pk, name FROM t2;          
```



相关文档 
-------------------------

* MySQL [SELECT](https://dev.mysql.com/doc/refman/5.7/en/select.html) 语法

* MySQL [JOIN](https://dev.mysql.com/doc/refman/5.7/en/join.html) 语法

* MySQL [UNION](https://dev.mysql.com/doc/refman/5.7/en/union.html) 语法



