CREATE VIEW 
================================

本文将介绍如何使用CREATE VIEW语句为PolarDB-X创建视图。

语法 
-----------------------

```sql
CREATE
[OR REPLACE]
VIEW view_name [(column_list)]
AS select_statement
```



示例 
-----------------------

```sql
# 先建表
CREATE TABLE t_order (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `order_id` varchar(20) DEFAULT NULL,
  `buyer_id` varchar(20) DEFAULT NULL,
  `seller_id` varchar(20) DEFAULT NULL,
  `order_snapshot` longtext DEFAULT NULL,
  `order_detail` longtext DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `l_i_order` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
# 创建视图
create view t_detail as select order_id,order_detail from t_order;
# 查询视图
select * from t_detail;
```


