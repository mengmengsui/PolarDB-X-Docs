MM 
=======================

本文将介绍MM函数的使用方式。

描述 
-----------------------

根据分库键时间值的月份数进行取余运算并得到分表下标。

使用限制 
-------------------------

* 拆分键的类型必须是DATE、DATETIME或TIMESTAMP中的一种。

* 只能作为分表函数使用，不能作为分库函数使用。

* 按MM进行分表，由于一年的月份只有12个月，所以各分库的分表数不能超过12。




使用场景 
-------------------------

MM函数适用于按月份数进行分表，分表的表名即为月份数。

使用示例 
-------------------------

假设需要先按ID对用户进行分库，再将create_time列按月份进行分表，使得每个月份能够对应一张物理表，则您可以使用如下的建表DDL：

```sql
create table test_mm_tb (    
    id int, 
    name varchar(30) DEFAULT NULL,  
    create_time datetime DEFAULT NULL,
    primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by HASH(id) 
tbpartition by MM(create_time) tbpartitions 12;
```


