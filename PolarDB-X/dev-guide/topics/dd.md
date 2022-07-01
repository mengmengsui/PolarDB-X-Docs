DD 
=======================

本文将介绍DD函数的使用方式。

描述 
-----------------------

根据分库键时间值日期的天数进行取余运算并得到分表下标。

使用限制 
-------------------------

* 拆分键的类型必须是DATE、DATETIME或TIMESTAMP中的一种。

* 只能作为分表函数使用，不能作为分库函数使用。

* 按DD进行分表，由于一个月中日期（DATE_OF_MONTH）的取值范围是1\~31，所以各分库的分表数不能超过31。




使用场景 
-------------------------

DD函数适用于按日期的天数进行分表，分表的表名即为日期的天数。

使用示例 
-------------------------

假设需要先按ID对用户进行分库，再将create_time列按日期进行分表，使得每个日期能够对应一张物理表，则您可以使用如下的建表DDL：

```sql
create table test_dd_tb (    
    id int, 
    name varchar(30) DEFAULT NULL,  
    create_time datetime DEFAULT NULL,
    primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by HASH(id) 
tbpartition by DD(create_time) tbpartitions 31;
```


