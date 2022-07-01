INDEX HINT 
===============================

PolarDB-X支持全局二级索引（Global Secondary Index，简称GSI） ，您可以通过INDEX HINT命令指定从GSI中获取查询结果。

使用限制 
-------------------------

INDEX HINT仅对SELECT语句生效。

注意事项 
-------------------------

PolarDB-X自定义HINT支持`/*+TDDL:hint_command*/` 和`/!+TDDL:hint_command*/`两种格式。若使用`/*+TDDL:hint_command*/`格式时，在使用MySQL 官方命令行客户端执行带有PolarDB-X自定义HIN的SQL时，请在登录命令中加上-c参数，否则由于PolarDB-X自定义HINT是以[MySQL注释](https://dev.mysql.com/doc/refman/5.6/en/comments.html?spm=a2c4g.11186623.2.13.30952e844fy7FE) 形式使用的，该客户端会将注释语句删除后再发送到服务端执行，导致PolarDB-X自定义HINT失效，详情请参见[MySQL官方客户端命令](https://dev.mysql.com/doc/refman/5.6/en/mysql-command-options.html?spm=a2c4g.11186623.2.14.30952e844fy7FE#option_mysql_comments) 。

语法 
-----------------------

PolarDB-X支持如下两种语法INDEX HINT：

* `FORCE INDEX()`：语法与[MySQL FORCE INDEX](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html) 相同，若指定的索引不是GSI，则会将FORCE INDEX下发到MySQL上执行。

  ```sql
  # FORCE INDEX()
  tbl_name [[AS] alias] [index_hint]
  index_hint:
      FORCE INDEX({index_name})               
  ```

  

* `INDEX()`： 通过表名和索引名组合或表在当前查询块中的别名和索引名组合来使用指定的GSI。

  ```sql
  # INDEX()
  /*+TDDL:
      INDEX({table_name | table_alias}, {index_name})
  */
  ```

  
  **说明** 上述语句在以下情况中不会生效：
  * 查询中不存在指定的表名或别名。
  
  * 指定的索引不是指定表上的GSI。
  

  
  




示例 
-----------------------

```sql
CREATE TABLE t_order (
 `id` bigint(11) NOT NULL AUTO_INCREMENT,
 `order_id` varchar(20) DEFAULT NULL,
 `buyer_id` varchar(20) DEFAULT NULL,
 `seller_id` varchar(20) DEFAULT NULL,
 `order_snapshot` longtext DEFAULT NULL,
 `order_detail` longtext DEFAULT NULL,
 PRIMARY KEY (`id`),
 GLOBAL INDEX `g_i_seller`(`seller_id`) dbpartition by hash(`seller_id`),
 UNIQUE GLOBAL INDEX `g_i_buyer` (`buyer_id`) COVERING(`seller_id`, `order_snapshot`) 
  dbpartition by hash(`buyer_id`) tbpartition by hash(`buyer_id`) tbpartitions 3 
) ENGINE=InnoDB DEFAULT CHARSET=utf8 dbpartition by hash(`order_id`);
```



* 在FROM子句中通过FORCE INDEX指定使用`g_i_seller`

  ```sql
  SELECT a.*, b.order_id
     FROM t_seller a
       JOIN t_order b FORCE INDEX(g_i_seller) ON a.seller_id = b.seller_id
     WHERE a.seller_nick="abc";
  ```

  

* 通过INDEX加上表的别名指定使用`g_i_buyer`

  ```sql
  /*+TDDL:index(a, g_i_buyer)*/ SELECT * FROM t_order a WHERE a.buyer_id = 123
  ```

  



