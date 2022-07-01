MMDD 
=========================

本文介绍MMDD函数的使用方式。

描述 
-----------------------

根据分库键时间值在一年中所对应的日期进行取余运算并得到分表下标。

使用限制 
-------------------------

* 拆分键的类型必须是DATE、DATETIME或TIMESTAMP中的一种。

* 只能作为分表函数使用，但不能作为分库函数。

* 按MMDD进行分表，由于一年最多只有366天，所以各个分库的分表数目不能超过366。




使用场景 
-------------------------

MMDD函数适用于按一年中的日期进行分表，分表的表名下标就是一年中的某个日期。

使用示例 
-------------------------

假设需要先按ID对用户进行分库，再将create_time列按一年中的日期（包括月份与日期）进行建表，使得一年中每一天的日期都能对应一张物理表，则您可以使用如下的建表DDL：

```sql
create table test_mmdd_tb (    
    id int, 
    name varchar(30) DEFAULT NULL,  
    create_time datetime DEFAULT NULL,
    primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by HASH(name) 
tbpartition by MMDD(create_time) tbpartitions 366; 
```


