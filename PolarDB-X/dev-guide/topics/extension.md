Grouping Sets、Rollup和Cube扩展 
================================================

在关系型数据库中，通常需要使用多个`SELECT + UNION`语句来实现按照多组维度的结果分组，PolarDB-X新增支持通过Grouping Sets、Rollup和Cube扩展来实现这一目的。此外，PolarDB-X还支持在SELECT命令或HAVING子句中使用GROUPING函数和GROUPING_ID函数，来帮助解释使用上述扩展时的结果。本文将介绍相关语法和示例。

注意事项 
-------------------------

* 本文介绍的所有GROUP BY相关的扩展语法，均不支持查询下推至`LogicalView`算子中执行。关于查询下推，请参见[查询改写与下推](../../sql-tunning/topics/query-rewriting.md)。

* 本文示例中所用测试数据信息如下：使用如下语句创建一张`requests`表：

  ```sql
  CREATE TABLE requests (
    `id` int(10) UNSIGNED NOT NULL,
    `os` varchar(20) DEFAULT NULL,
    `device` varchar(20) DEFAULT NULL,
    `city` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE = InnoDB DEFAULT CHARSET = utf8 dbpartition BY hash(`id`) tbpartition BY hash(`id`);
  ```

  

  在`requests`表中使用如下语句插入测试所需的数据：

  ```sql
  INSERT INTO requests (id, os, device, city) VALUES
  (1, 'windows', 'PC', 'Beijing'),
  (2, 'windows', 'PC', 'Shijiazhuang'),
  (3, 'linux', 'Phone', 'Beijing'),
  (4, 'windows', 'PC', 'Beijing'),
  (5, 'ios', 'Phone', 'Shijiazhuang'),
  (6, 'linux', 'PC', 'Beijing'),
  (7, 'windows', 'Phone', 'Shijiazhuang');
  ```

  

  




GROUPING SETS扩展 
------------------------------------

* **功能介绍**

  GROUPING SETS是GROUP BY子句的扩展，可以生成一个结果集，该结果集实际上是基于不同分组的多个结果集的串联（与UNION ALL运算结果类似），但UNION ALL运算和GROUPING SETS扩展并不会消除合并结果集中的重复行。
  

* **语法**

  ```sql
  GROUPING SETS (
    { expr_1 | ( expr_1a [, expr_1b ] ...) |
      ROLLUP ( expr_list ) | CUBE ( expr_list )
    } [, ...] )
  ```

  
  **说明** GROUPING SETS扩展可包含一个或多个由半角逗号（,）分隔表达式（如`expr_1`或`(expr_1a [, expr_1b ] ...)`）的任意组合，以及带半角圆括号（()）的表达式列表（如`( expr_list )`），其中：
  * 每个表达式都可用于确定结果集的分组方式。
  
  * GROUPING SETS内也支持嵌套使用ROLLUP或者CUBE。
  

  
  

* **示例**
  * 通过GROUPING SETS扩展对数据进行分组查询，语法如下：

    ```sql
    select os,device, city ,count(*)
    from requests
    group by grouping sets((os, device), (city), ());
    上述语句等效于如下语句：
    select os, device, NULL, count(*)
    from requests group by os, device
    union all
    select NULL, NULL, NULL, count(*)
    from requests
    union all
    select null, null, city, count(*)
    from requests group by city;
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | windows | PC     | NULL         |        3 |
    | linux   | PC     | NULL         |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | ios     | Phone  | NULL         |        1 |
    | NULL    | NULL   | Shijiazhuang |        3 |
    | NULL    | NULL   | Beijing      |        4 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    
    **说明** 未在分组集中使用的表达式，会用NULL充当占位符，便于对这些未在分组集使用的结果集进行操作，例如结果`city`列中显示为NULL的行。

    
  
  * 通过在GROUPING SETS中嵌套ROLLUP来对数据进行分组，语法如下：

    ```sql
    select os,device, city ,count(*) from requests 
    group by grouping sets((city), ROLLUP(os, device));
    上述语句等效于如下语句：
    select os,device, city ,count(*) from requests 
    group by grouping sets((city), (os), (os, device), ());
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | NULL    | NULL   | Shijiazhuang |        3 |
    | NULL    | NULL   | Beijing      |        4 |
    | windows | PC     | NULL         |        3 |
    | linux   | PC     | NULL         |        1 |
    | ios     | Phone  | NULL         |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | windows | NULL   | NULL         |        4 |
    | linux   | NULL   | NULL         |        2 |
    | ios     | NULL   | NULL         |        1 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  
  * 通过在GROUPING SETS中嵌套CUBE扩展来对数据进行分组，语法如下：

    ```sql
    select os,device, city ,count(*) from requests 
    group by grouping sets((city), CUBE(os, device));
    上述语句等效于如下语句：
    select os,device, city ,count(*) from requests 
    group by grouping sets((city), (os), (os, device), (), (device));
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | NULL    | NULL   | Beijing      |        4 |
    | NULL    | NULL   | Shijiazhuang |        3 |
    | windows | PC     | NULL         |        3 |
    | ios     | Phone  | NULL         |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | linux   | PC     | NULL         |        1 |
    | windows | NULL   | NULL         |        4 |
    | ios     | NULL   | NULL         |        1 |
    | linux   | NULL   | NULL         |        2 |
    | NULL    | PC     | NULL         |        4 |
    | NULL    | Phone  | NULL         |        3 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  
  * 通过GROUP BY、CUBE和GROUPING SETS组合产生GROUPING SETS，示例如下：

    ```sql
    select os,device, city, count(*)
    from requests 
    group by os, cube(os,device), grouping sets(city);
    上述语句等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by grouping sets((os,device,city),(os,city),(os,device,city));
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | linux   | Phone  | Beijing      |        1 |
    | windows | Phone  | Shijiazhuang |        1 |
    | windows | PC     | Shijiazhuang |        1 |
    | linux   | PC     | Beijing      |        1 |
    | windows | PC     | Beijing      |        2 |
    | ios     | Phone  | Shijiazhuang |        1 |
    | linux   | NULL   | Beijing      |        2 |
    | windows | NULL   | Shijiazhuang |        2 |
    | windows | NULL   | Beijing      |        2 |
    | ios     | NULL   | Shijiazhuang |        1 |
    +---------+--------+--------------+----------+
    ```

    

    
  

  




ROLLUP扩展 
-----------------------------

* **功能介绍**

  ROLLUP扩展生成一系列有总计的分层组，每个分层组都有小计。该层次结构的顺序由ROLLUP表达式列表中给定的表达式的顺序确定。该层次结构的顶部是列表中最左侧的项。每个连续项都会沿右侧在该层次结构中向下移动，最右侧的项是最低级别。
  

* **语法**

  ```sql
  ROLLUP ( { expr_1 | ( expr_1a [, expr_1b ] ...) }
  [, expr_2 | ( expr_2a [, expr_2b ] ...) ] ...)
  ```

  
  **说明**
  * 每个表达式都会用于确定结果集的分组方式。如果采用带圆括号形式的表达式，例如`( expr_1a, expr_1b, ...)`，则 `expr_1a`和`expr_1b`返回的值组合定义层次结构的单个分组级别。
  
  * 对于列表中的第一项，例如`expr_1`或`( expr_1a, expr_1b, ...)`的组合，PolarDB-X将为每个唯一值返回一个小计。对于列表中的第二项，例如`expr_2` 或`( expr_2a, expr_2b, ...)`的组合，PolarDB-X将为第二项的每个分组中的每个唯一值返回一个小计，依此类推。最后，PolarDB-X将为整个结果集返回一个总计。
  
  * 对于小计行，将为小计包含的各项返回NULL。
  

  
  

* **示例**
  * 通过ROLLUP对`(os, device, city)`按层级聚合的方式产生GROUPING SETS，语法如下：

    ```sql
    select os,device, city, count(*)
    from requests 
    group by rollup (os, device, city);
    上述语句等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by os, device, city with rollup;
    也等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by grouping sets ((os, device, city),(os, device),(os),());
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | windows | PC     | Beijing      |        2 |
    | ios     | Phone  | Shijiazhuang |        1 |
    | windows | PC     | Shijiazhuang |        1 |
    | linux   | PC     | Beijing      |        1 |
    | linux   | Phone  | Beijing      |        1 |
    | windows | Phone  | Shijiazhuang |        1 |
    | windows | PC     | NULL         |        3 |
    | ios     | Phone  | NULL         |        1 |
    | linux   | PC     | NULL         |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | windows | NULL   | NULL         |        4 |
    | ios     | NULL   | NULL         |        1 |
    | linux   | NULL   | NULL         |        2 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  
  * 通过ROLLUP对`os, (os,device), city`按层级聚合的方式产生GROUPING SETS，语法如下：

    ```sql
    select os,device, city, count(*)
    from requests 
    group by rollup (os, (os,device), city);
    上述语句等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by os, (os,device), city with rollup;
    也等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by grouping sets ((os, device, city),(os, device),(os),());
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | windows | PC     | Beijing      |        2 |
    | windows | PC     | Shijiazhuang |        1 |
    | linux   | PC     | Beijing      |        1 |
    | linux   | Phone  | Beijing      |        1 |
    | windows | Phone  | Shijiazhuang |        1 |
    | ios     | Phone  | Shijiazhuang |        1 |
    | windows | PC     | NULL         |        3 |
    | linux   | PC     | NULL         |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | ios     | Phone  | NULL         |        1 |
    | windows | NULL   | NULL         |        4 |
    | linux   | NULL   | NULL         |        2 |
    | ios     | NULL   | NULL         |        1 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  

  




CUBE扩展 
---------------------------

* **功能介绍**

  CUBE扩展与ROLLUP扩展类似，但与生成分组并基于ROLLUP表达式列表中从左到右的项列表生成层次结构的ROLLUP扩展不同，CUBE是基于CUBE表达式列表中所有项的每个排列生成分组和小计。因此，与对同一表达式列表执行的ROLLUP相比，CUBE结果集会包含更多的行。
  

* **语法**

  ```sql
  CUBE ( { expr_1 | ( expr_1a [, expr_1b ] ...) }
  [, expr_2 | ( expr_2a [, expr_2b ] ...) ] ...)
  ```

  
  **说明**
  * 每个表达式都会用于确定结果集的分组方式。如果采用带半角圆括号的形式，例如`( expr_1a, expr_1b, ...)`，则 `expr_1a`和`expr_1b`返回的值组合定义单个组。
  
  * 对于列表中的第一项，例如`expr_1`或`( expr_1a, expr_1b, ...)`的组合，PolarDB-X将为每个唯一值返回一个小计。对于列表中的第二项，例如`expr_2` 或`( expr_2a, expr_2b, ...)`的组合，PolarDB-X在为每个唯一值返回一个小计的同时，还将为第一项和第二项的每个唯一组合返回一个小计。如果存在第三项，PolarDB-X则会为第三项的每个唯一值、第三项和第一项组合的每个唯一值、第三项和第二项组合的每个唯一值以及第三项、第二项和第一项组合的每个唯一值返回一个小计。最后，再将为整个结果集返回一个总计。
  
  * 对于小计行，将为小计包含的各项返回NULL。
  

  
  

* **示例**
  * 通过CUBE枚举`(os, device, city)`的所有可能列为GROUPING SETS，语法如下：

    ```sql
    select os,device, city, count(*)
    from requests 
    group by cube (os, device, city);
    上述语句等效于如下语句：
    select os,device, city, count(*)
    from requests 
    group by grouping sets ((os, device, city),(os, device),(os, city),(device,city),(os),(device),(city),());
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | linux   | Phone  | Beijing      |        1 |
    | windows | Phone  | Shijiazhuang |        1 |
    | windows | PC     | Beijing      |        2 |
    | ios     | Phone  | Shijiazhuang |        1 |
    | windows | PC     | Shijiazhuang |        1 |
    | linux   | PC     | Beijing      |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | windows | PC     | NULL         |        3 |
    | ios     | Phone  | NULL         |        1 |
    | linux   | PC     | NULL         |        1 |
    | linux   | NULL   | Beijing      |        2 |
    | windows | NULL   | Shijiazhuang |        2 |
    | windows | NULL   | Beijing      |        2 |
    | ios     | NULL   | Shijiazhuang |        1 |
    | linux   | NULL   | NULL         |        2 |
    | windows | NULL   | NULL         |        4 |
    | ios     | NULL   | NULL         |        1 |
    | NULL    | Phone  | Beijing      |        1 |
    | NULL    | Phone  | Shijiazhuang |        2 |
    | NULL    | PC     | Beijing      |        3 |
    | NULL    | PC     | Shijiazhuang |        1 |
    | NULL    | Phone  | NULL         |        3 |
    | NULL    | PC     | NULL         |        4 |
    | NULL    | NULL   | Beijing      |        4 |
    | NULL    | NULL   | Shijiazhuang |        3 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  
  * 通过CUBE枚举`(os, device),(device, city)`所有可能列为GROUPING SETS，语法如下：

    ```sql
    select os,device, city, count(*) 
    from requests 
    group by cube ((os, device), (device, city));
    上述语句等效于如下语句：
    select os,device, city, count(*) 
    from requests 
    group by grouping sets ((os, device, city),(os, device),(device,city),());
    ```

    

    返回结果如下：

    ```sql
    +---------+--------+--------------+----------+
    | os      | device | city         | count(*) |
    +---------+--------+--------------+----------+
    | linux   | Phone  | Beijing      |        1 |
    | windows | Phone  | Shijiazhuang |        1 |
    | windows | PC     | Beijing      |        2 |
    | windows | PC     | Shijiazhuang |        1 |
    | linux   | PC     | Beijing      |        1 |
    | ios     | Phone  | Shijiazhuang |        1 |
    | linux   | Phone  | NULL         |        1 |
    | windows | Phone  | NULL         |        1 |
    | windows | PC     | NULL         |        3 |
    | linux   | PC     | NULL         |        1 |
    | ios     | Phone  | NULL         |        1 |
    | NULL    | Phone  | Beijing      |        1 |
    | NULL    | Phone  | Shijiazhuang |        2 |
    | NULL    | PC     | Beijing      |        3 |
    | NULL    | PC     | Shijiazhuang |        1 |
    | NULL    | NULL   | NULL         |        7 |
    +---------+--------+--------------+----------+
    ```

    

    
  

  




GROUPING和GROUPING_ID函数 
-------------------------------------------

* **功能介绍**
  * **GROUPING函数**

    在GROUP BY子句使用GROUPING SETS、ROLLUP、或CUBE扩展时，GROUPING SETS结果中会使用NULL来充当占位符，导致无法区分占位符NULL与数据中真正的NULL。此时，您可以使用PolarDB-X提供的GROUPING函数来作区分。

    GROUPING函数接受一个列名作为参数，如果结果对应行使用了参数列做聚合，则结果返回0，此时意味着NULL来自输入数据。如果结果对应行未使用参数列做聚合，则返回1，此时意味着NULL来自GROUPING SETS结果中的占位符。
    
  
  * **GROUPING_ID函数**

    GROUPING_ID函数简化了GROUPING函数，用于确定ROLLBACK、CUBE或GROUPING SETS扩展的结果集中行的小计级别。GROUPING函数仅采用一个列表达式并返回一个值来指示行是否为给定列的所有值的小计。因此，当解释具有多个分组列的查询的小计级别时，可能需要多个 GROUPING函数。GROUPING_ID函数接受ROLLBACK、CUBE或GROUPINGSETS扩展中已使用的一个或多个列表达式，并返回单个整数，该整数可用于确定其中哪一列已聚合小计。
    
  

  

* **语法**
  * **GROUPING函数**

    ```sql
    SELECT [ expr ...,] GROUPING( col_expr ) [, expr ] ...
    FROM ...
    GROUP BY { ROLLUP | CUBE | GROUPING SETS }( [...,] col_expr
      [, ...] ) [, ...]
    ```

    
    **说明** GROUPING函数采用单个参数，该参数必须是GROUP BY子句中ROLLUP、CUBE或GROUPING SETS扩展的表达式列表中指定的维度列的表达式。
    
  
  * **GROUPING_ID函数**

    ```sql
    SELECT [ expr ...,]
      GROUPING_ID( col_expr_1 [, col_expr_2 ] ... )
      [, expr ] ...
    FROM ...
    GROUP BY { ROLLUP | CUBE | GROUPING SETS }( [...,] col_expr_1
      [, col_expr_2 ] [, ...] ) [, ...]
    ```

    
  

  

* **示例**

  通过GROUPING_ID函数将多个列名作为参数，并将参数列的GROUPING结果按照Bitmap的方式组成整数，语法如下：

  ```sql
  select a,b,c,count(*),
  grouping(a) ga, grouping(b) gb, grouping(c) gc, grouping_id(a,b,c) groupingid 
  from (select 1 as a ,2 as b,3 as c)
  group by cube(a,b,c);
  ```

  

  返回结果如下：

  ```sql
  +------+------+------+----------+------+------+------+------------+
  | a    | b    | c    | count(*) | ga   | gb   | gc   | groupingid |
  +------+------+------+----------+------+------+------+------------+
  |    1 |    2 |    3 |        1 |    0 |    0 |    0 |          0 |
  |    1 |    2 | NULL |        1 |    0 |    0 |    1 |          1 |
  |    1 | NULL |    3 |        1 |    0 |    1 |    0 |          2 |
  |    1 | NULL | NULL |        1 |    0 |    1 |    1 |          3 |
  | NULL |    2 |    3 |        1 |    1 |    0 |    0 |          4 |
  | NULL |    2 | NULL |        1 |    1 |    0 |    1 |          5 |
  | NULL | NULL |    3 |        1 |    1 |    1 |    0 |          6 |
  | NULL | NULL | NULL |        1 |    1 |    1 |    1 |          7 |
  +------+------+------+----------+------+------+------+------------+
  ```

  

  



