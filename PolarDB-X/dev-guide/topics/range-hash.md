RANGE_HASH 
===============================

本文将介绍RANGE_HASH函数的使用方式。

描述 
-----------------------

根据任一拆分键后N位计算哈希值，然后再按分库数取余，完成路由计算。N为函数第三个参数。

例如，RANGE_HASH(COL1, COL2, N) ，计算时会优先选择COL1，截取其后N位进行计算。 COL1不存在时再选择COL2。

使用限制 
-------------------------

* 拆分键的类型必须是字符类型或数字类型，两个拆分键类型必须保持一致。

* 两个拆分键皆不能修改。

* 折分键暂时不支持做范围查询。

* 插入数据时两个拆分键的后N位需确保一致。

* 字符串长度需不少于N位。




使用场景 
-------------------------

适用于需要有两个拆分键，并且查询时仅有其中一个拆分键值的场景。

使用示例 
-------------------------

假设PolarDB-X里已经分了8个物理库，现在需要按买家ID和订单ID对订单表进行分库；查询时条件仅有买家ID或订单ID，那么您可以使用如下DDL语句构建订单表：

```sql
create table test_order_tb (  
id int, 
buyer_id varchar(30) DEFAULT NULL,  
order_id varchar(30) DEFAULT NULL, 
create_time datetime DEFAULT NULL,
primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by RANGE_HASH(buyer_id,order_id, 10) 
tbpartition by RANGE_HASH (buyer_id,order_id, 10) tbpartitions 3;
```


