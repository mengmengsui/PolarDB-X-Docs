WEEK 
=========================

本文介绍WEEK函数的使用方式。

描述 
-----------------------

根据分库键的时间值所对应的一周之中的日期进行取余运算并得到分表下标。

使用限制 
-------------------------

* 拆分键的类型必须是DATE、DATETIME或TIMESTAMP中的一种。

* 只能作为分表函数使用，但不能作为分库函数。




使用场景 
-------------------------

WEEK适用于按周数的日期进行分表，分表表名的下标分别对应一周中的各个日期（星期一到星期天）。

使用示例 
-------------------------

假设先按ID对用户进行分库，再需要对create_time列按周进行分表，并且每周7天（星期一到星期天）各对应一张物理表，则应该使用如下的建表DDL：

```sql
create table test_week_tb (    
    id int, 
    name varchar(30) DEFAULT NULL,  
    create_time datetime DEFAULT NULL,
    primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by HASH(name) 
tbpartition by WEEK(create_time) tbpartitions 7;
```


**说明** 当按WEEK进行分表时，由于一周共有7天，所以各分库的分表数不能超过7张。
