CREATE TABLE 
=================================

本文主要介绍使用DDL语句进行建表的语法、子句、参数和基本方式。

语法 
-----------------------

```sql
CREATE [PARTITION] TABLE [IF NOT EXISTS] tbl_name
    (create_definition, ...)
    [table_options]
    [table_partition_definition]
create_definition:
    col_name column_definition
  | mysql_create_definition
  | [UNIQUE] GLOBAL INDEX index_name [index_type] (index_sharding_col_name,...)
      [global_secondary_index_option]
      [index_option] ...
index_sharding_col_name:
    col_name [(length)] [ASC | DESC]
index_option:
    KEY_BLOCK_SIZE [=] value
  | index_type
  | WITH PARSER parser_name
  | COMMENT 'string'
index_type:
    USING {BTREE | HASH}
# 全局二级索引相关
global_secondary_index_option:
    [COVERING (col_name,...)]
    [partition_options]
# 分区表类型定义
table_partition_definition:
		single
  |	broadcast
  | partition_options
# 分区策略定义
partition_options:
    PARTITION BY
          HASH({column_name | partition_func(column_name)})
        | KEY(column_list)
        | RANGE{({column_name | partition_func(column_name)}) 
        | RANGE COLUMNS(column_list)}
        | LIST{({column_name | partition_func(column_name)}) 
        | LIST COLUMNS(column_list)} }
    partition_list_spec
# 分区函数定义
partition_func：
    YEAR
  | TO_DAYS
  | TO_SECOND
  | UNIX_TIMESTAMP
  | MONTH
# 分区列表定义
partition_list_spec:
		hash_partition_list
  | range_partition_list
  | list_partition_list
# Hash / Key 分区表列定义
hash_partition_list:
	PARTITIONS partition_count
# Range / Range Columns 分区表列定义
range_partition_list:
    range_partition [, range_partition ...]
range_partition:
    PARTITION partition_name VALUES LESS THAN {(expr | value_list)} [partition_spec_options]
# List / List Columns 分区表列定义
list_partition_list:
    list_partition [, list_partition ...]
list_partition:
    PARTITION partition_name VALUES IN (value_list) [partition_spec_options]
partition_spec_options:
	      [[STORAGE] ENGINE [=] engine_name]
        [COMMENT [=] 'string']
        [{CHARSET | CHARACTER SET} [=] charset]
        [COLLATE [=] collation]
        [TABLEGROUP [=] table_group_id]
        [LOCALITY [=] locality_option]
locality_option:
    'dn=storage_inst_id_list'
storage_inst_id_list:
    storage_inst_id[,storage_inst_id_list]
```


**说明** PolarDB-X DDL语法基于MySQL语法，以上主要列出了差异部分，详细语法请参见[MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/create-table.html) 。

默认自动分区 
---------------------------

* 建表SQL在不指定分区键的情况下，PolarDB-X默认会按主键（如果表没有指定主键，则使用隐式主键）并使用KEY分区策略进行分区，默认分区的分区数目是64。

* 用户也可以通过指定`CREATE PARTITION TABLE`语法关键字进行建表，来显示让PolarDB-X选择主键进行自动分区，示例如下：

  ```sql
  CREATE PARTITION TABLE auto_part_tbl(
   id bigint not null auto_increment, 
   bid int, 
   name varchar(30), 
   primary key(id)
  );
  ```

  




单表与广播表 
---------------------------

PolarDB-X允许创建表时通过指定关键字SINGLE来建单表（不进行任何分区的表），示例如下：

```sql
CREATE TABLE single_tbl(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30), 
 primary key(id)
) SINGLE;
```



PolarDB-X允许创建表时通过指定关键字BROADCAST来建广播表（该表将在所有DN节点上有一份数据完全相同的拷贝），示例如下：

```sql
CREATE TABLE broadcast_tbl(
 id bigint not null auto_increment, 
 bid int, 
 name varchar(30), 
 primary key(id)
) BROADCAST;
```



分区表 
------------------------

PolarDB-X允许创建表时通过指定分区子句的语义，来创建符合业务需要求的分区表。从总体上，PolarDB-X支持三种类型的分区策略：

* **Hash分区策略** ：基于用户指定的分区列或分区函数表达式的值，使用内置的一致性哈希算法计算其哈希值并进行分区路由的策略。按是否支持使用分区函数表达式或使用多个分区列作为分区键，Hash分区策略又可以进行步细分为Hash分区与Key分区两种类型。

* **Range分区策略** ：基于用户指定的分区列或分区函数表达式的值，通过比较计算来确定其所落在哪些预定义分区的范围并进行分区路由的策略。按是否支持使用分区函数表达式或使用多个分区列作为分区键，Range分区策略又可以进行步细分为Range分区与Range Columns分区两种类型。

* **List分区策略** ：与Range分区策略类似，基于用户指定的分区列或分区函数表达式的值，通过比较计算来确定其所落在哪些预指定义分区的取值集合并进行分区路由的策略。按是否支持使用分区函数表达式或使用多个分区列作为分区键，List 分区策略又可以进行步细分为List分区与List Columns分区两种类型。




Hash分区策略 
-----------------------------

PolarDB-X按是否支持使用分区函数表达式或使用多个分区列作为分区键，Hash分区策略进行步细分为Hash分区与Key分区两种类型。

* **Hash分区**

  Hash分区建表只支持使用一个整数类型的分区列作为分区健，但对于时间类型的分区列，它支持这些分区列外层套用一个分区函数表达式（例如，YEAR/TO_DAYS/TO_SECOND/MONTH等）来转换成整数类型。其次需要值得注意的是，Hash分区是不支持直接使用字符串类型作为分区列。

  如果想要按照用户ID列进行分区，预建的Hash分区数目是8，可以执行以下命令建表：

  ```sql
  CREATE TABLE hash_tbl(
   id bigint not null auto_increment, 
   bid int, 
   name varchar(30),
   birthday datetime not null,
   primary key(id)
  ) 
  partition by  hash(bid) 
  partitions 8;
  ```

  

  如果想要按找用户出生日期birthday列进行分区，预建的Hash分区数目是8，可以执行以下命令建表：

  ```sql
  CREATE TABLE hash_tbl_todays(
   id bigint not null auto_increment, 
   bid int, 
   name varchar(30),
   birthday datetime not null,
   primary key(id)
  ) 
  PARTITION BY HASH(TO_DAYS(birthday)) 
  PARTITIONS 8;
  ```

  

  目前，PolarDB-X的分区函数仅支持以下列表：
  * YEAR
  
  * TO_DAYS
  
  * TO_SECOND
  
  * MONTH
  
  * UNIX_TIMESTAMP
  

  

  

* **Key分区**

  与Hash分区不同，使用Key分区建表，支持同时使用多个分区列作为分区键，但它不允许分区列外层套用任何分区函数表达。Key分区的分区列的类型支持比Hash分区更为丰富，支持的类型如下所示：
  * 整数类型：BIGINT/BIGINT UNSINGEDINT/INT UNSINGED/MEDIUMINT/MEDIUMINT UNSINGED/SMALLINT/SMALLINT UNSINGED/TINYINT/TINYINT UNSINGED
  
  * 时间类型：DATETIME/DATE/TIMESTAMP
  
  * 字符串类型：CHAR/VARCHR
  

  

  因此，Key分区是PolarDB-X的默认分区策略。

  如果想要按照用户ID列与用户出生日期列作为分区键进行分区，预建的Hash分区数目是8，可以执行以下命令建表：

  ```sql
  CREATE TABLE key_tbl(
   id bigint not null auto_increment, 
   bid int, 
   name varchar(30),
   birthday datetime not null,
   primary key(id)
  ) 
  PARTITION BY KEY(id, birthday) 
  PARTITIONS 8;
  ```

  

  




Range分区策略 
------------------------------

PolarDB-X按照是否支持使用分区函数表达式或使用多个分区列作为分区键，将Range分区策略进一步细分为Range分区与Range Columns分区两种类型。

* **Range分区**

  Range分区只支持使用一个整数类型的分区列作为分区健，但对于时间类型的分区列，它支持这些分区列外层套用一个分区函数表达式（例如，YEAR/TO_DAYS/TO_SECOND/MONTH等）来转换成整数类型。另外值得注意的是，Range分区是不支持直接使用字符串类型作为分区列。
  **说明** Range不支持使用NULL值作为边界值。

  如果业务想按订单的日期进行Range分区，并且每个季度一个分区，可以执行以下命令建表：

  ```sql
  CREATE TABLE orders(
   id int, 
   order_time datetime not null) 
  PARTITION BY RANGE(to_days(order_time)) 
  (
    PARTITION p1 VALUES LESS THAN (to_days('2021-01-01')),
    PARTITION p2 VALUES LESS THAN (to_days('2021-04-01')),
    PARTITION p3 VALUES LESS THAN (to_days('2021-07-01')),
    PARTITION p4 VALUES LESS THAN (to_days('2021-10-01')),
    PARTITION p5 VALUES LESS THAN (to_days('2022-01-01'))
  );
  ```

  

  

* **Range Columns分区**

  Range Columns分区支持同时使用多个分区列作为分区键，但它不允许分区列外层套用任何分区函数表达。Range Columns分区的分区列的类型支持比Range分区更为丰富，支持的类型如下所示：
  * 整数类型：BIGINT/BIGINT UNSINGEDINT/INT UNSINGED/MEDIUMINT/MEDIUMINT UNSINGED/SMALLINT/SMALLINT UNSINGED/TINYINT/TINYINT UNSINGED
  
  * 时间类型：DATETIME/DATE
  
  * 字符串类型：CHAR/VARCHR
  

  

  **说明**
  * Range Columns分区目前还不支的使用TIMESTAMP类型，后续版本会完善。
  
  * 不支持使用NULL值作为边界值。
  

  

  如果业务想要按照订单ID与订单日期进行Range分区， 可以执行以下命令建表：

  ```sql
  CREATE TABLE orders(
   order_id int, 
   order_time datetime not null) 
  PARTITION BY range columns(order_id,order_time) 
  (
    PARTITION p1 VALUES LESS THAN (10000,'2021-01-01'),
    PARTITION p2 VALUES LESS THAN (20000,'2021-01-01'),
    PARTITION p3 VALUES LESS THAN (30000,'2021-01-01'),
    PARTITION p4 VALUES LESS THAN (40000,'2021-01-01'),
    PARTITION p5 VALUES LESS THAN (50000,'2021-01-01')
  );
  ```

  

  




List分区策略 
-----------------------------

与Range分区类似，PolarDB-X按是否支持使用分区函数表达式或使用多个分区列作为分区键，将List分区策略进一步细分为List分区与List Columns分区两种类型。

* **List分区**

  List分区只支持使用一个整数类型的分区列作为分区健，但对于时间类型的分区列，它支持这些分区列外层套用一个分区函数表达式（例如，YEAR/TO_DAYS/TO_SECOND/MONTH等）来转换成整数类型。另外值得注意的是，Range分区是不支持直接使用字符串类型作为分区列。

  如果业务想按订单的日期进行List分区，并且每个季度一个分区，可以执行以下命令建表：

  ```sql
  CREATE TABLE orders(
   id int, 
   order_region varchar(64),
   order_time datetime not null) 
  PARTITION BY LIST(YEAR(order_time)) 
  (
    PARTITION p1 VALUES IN (1990,1991,1992,1993,1994,1995,1996,1997,1998,1999),
    PARTITION p2 VALUES IN (2000,2001,2002,2003,2004,2005,2006,2007,2008,2009),
    PARTITION p3 VALUES IN (2010,2011,2012,2013,2014,2015,2016,2017,2018,2019)
  );
  ```

  

  

* **List Columns分区**

  List Columns分区支持同时使用多个分区列作为分区键，但它不允许分区列外层套用任何分区函数表达。List Columns分区的分区列的类型支持比List分区更为丰富，支持的类型如下所示：
  * 整数类型：BIGINT/BIGINT UNSINGEDINT/INT UNSINGED/MEDIUMINT/MEDIUMINT UNSINGED/SMALLINT/SMALLINT UNSINGED/TINYINT/TINYINT UNSINGED
  
  * 时间类型：DATETIME/DATE
  
  * 字符串类型：CHAR/VARCHR
  

  

  **说明**
  * List Columns分区目前还不支的使用TIMESTAMP类型，后续版本进行完善。
  
  * 不支持使用NULL值作为边界值。
  

  

  如果业务按照订单ID与订单日期进行Range分区，可以执行以下命令建表：

  ```sql
  CREATE TABLE orders(
   id int, 
   order_region varchar(64),
   order_time datetime not null) 
  PARTITION BY LIST COLUMNS(order_region) 
  (
    PARTITION p1 VALUES IN ('Hangzhou', 'Shenzhen'),
    PARTITION p2 VALUES IN ('Beijing', 'Shanghai'),
    PARTITION p3 VALUES IN ('Qingdao')
  );
  ```

  

  




分区表数据类型 
----------------------------

各种分区策略及其数据类型支持如下：


|           数据类型            ||                            Hash类型（HASH）                             |          Hash类型（KEY）           |                           Range类型（Range）                            |      Range类型（Range Columns）      |                            List类型（List）                             |       List类型（List Columns）       |
|-------|--------------------|---------------------------------------------------------------------|--------------------------------|---------------------------------------------------------------------|----------------------------------|---------------------------------------------------------------------|----------------------------------|
| 数值类型  | TINYINT            | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | TINYINT UNSIGNED   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | SMALLINT           | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | SMALLINT UNSIGNED  | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | MEDIUMINT          | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | MEDIUMINT UNSIGNED | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | INT                | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | INT UNSIGNED       | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | BIGINT             | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 数值类型  | BIGINT UNSIGNED    | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)支持                                      | ![正确](../images/p276697.png)支持   |
| 时间类型  | DATE               | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持   |
| 时间类型  | DATETIME           | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持 | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持   | ![正确](../images/p276697.png)分区列需要套用year/month/to_days/to_seconds函数。 | ![正确](../images/p276697.png)支持   |
| 时间类型  | TIMESTAMP          | ![正确](../images/p276697.png)分区列必须要外套unix_timestamp函数配合使用。           | ![正确](../images/p276697.png)支持 | ![错误](../images/p276698.png)暂不支持                                    | ![错误](../images/p276698.png)暂不支持 | ![错误](../images/p276698.png)暂不支持                                    | ![错误](../images/p276698.png)暂不支持 |
| 字符串类型 | CHAR               | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持 | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持   | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持   |
| 字符串类型 | VARCHAR            | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持 | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持   | ![错误](../images/p276698.png)不支持                                     | ![正确](../images/p276697.png)支持   |



参数说明 
-------------------------



|            参数            |                                                                                                                                                                                                                                                             说明                                                                                                                                                                                                                                                             |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CHARSET &#124; CHARACTER SET | 指定表中列的默认字符集，可使用字符集如下： <ul><li>utf8</li>  <li>utf8mb4</li>  <li>gbk</li></ul>                                                                                                                                                                                                                                                                                                                                                      |
| COLLATE                  | 指定表中列的默认字符序，可使用字符序如下： <ul><li>utf8_bin</li>  <li>utf8_general_ci</li>  <li>utf8_unicode_ci</li>  <li>gbk_bin</li>  <li>gbk_chinese_ci</li>  <li>utf8mb4_general_ci</li>  <li>utf8mb4__general_cs</li>  <li>utf8mb4_bin</li>  <li>utf8mb4_unicode_ci</li></ul>   |
| TABLEGROUP               | 用于指定分区表所属于的表组。若不指定，会自动查找或创建与之分区方式完全一致的表组。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| LOCALITY                 | 用于指定分区表的所在DN节点。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |


