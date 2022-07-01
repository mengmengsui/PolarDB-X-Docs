STR_HASH 
=============================

本文将介绍STR_HASH函数使用方式。

描述 
-----------------------

STR_HASH函数通过指定字符串的开始位置下标与结束下标，以截取拆分键的字符串的某段子串，然后将其作为字符串（或整数）输入进行分库分表的路由计算具体的物理分片，函数如下所示：

```sql
STR_HASH( shardKey [, startIndex, endIndex [, valType [, randSeed ] ] ] )
```



|     参数     |                                                                                                                                                                                                                                                                                                                                                                                                                                                              说明                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| shardKey   | 拆分键列的列名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| startIndex | 目标子串的开始位置的下标。取值从0开始（即原字符串的第1个字符的下标用0表示），默认值为-1（即不做任何截取）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| endIndex   | 目标子串的结束位置的下标。取值从0开始（即原字符串的第1个字符的下标用0表示），默认值为-1（即不做任何截取）。 startIndex和endIndex时需注意如下几种取值情况： <ul><li>当startIndex == j && endIndex = k (j>=0, k>=0 ,k>j)时，表示截取原字符串的[ j, k )区间的字符串作为子串。例如： <ul><li>对于字符串ABCDEFG，子串区间[1,5)的值是BCDE。</li>  <li>对于字符串ABCDEFG,，子串区间[2,2)的值是''。</li>  <li>对于字符串ABCDEFG，子串区间[4,100)的值是EFG。</li>  <li>对于字符串 ABCDEFG，子串区间[100,105)的值是''。</li></ul>    <li>当startIndex == -1 && endIndex = k (k>=0)时，表示截取原字符串最后k个字符作为子串，原字符串不足k个字符则直接获取整个字符串。</li>  <li>当startIndex = k && endIndex == -1 (k>=0)时，表示截取原字符串开头k个字符作为子串，原字符串不足k个字符则直接获取整个字符串。</li>  <li>当startIndex == -1 && endIndex == -1时， 表示不做任何截取，子串与原字符串完全一致。</li></ul>   |
| valType    | 表示截取后的子串在计算分库分表时所使用的类型，取值范围如下： <ul><li>0（默认值）：表示PolarDB-X将截取后的子串当作字符串类型来计算路由。</li>  <li>1：表示PolarDB-X将截取后的子串当作整数类型来计算路由（子串面值的整数不能大于9223372036854775807，也不支持浮点数）</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| randSeed   | 当子串以字符串类型来计算路由的哈希值时PolarDB-X所使用的随机种子的值，通常不用需要填写，仅当用于使用默认值随机种子（randSeed=31）的STR_HASH在实际业务中出现路由不均衡的场景，达到用哈希均衡数据的目的。该参数默认值为31，可取其他值（如131，13131，1313131等）。 **说明：** <ul><li>仅当valType取值为0时，才支持配置该参数。</li>  <li>该参数调整后您需要手动将所有数据导出来，再将新的拆分算法导入数据（即您需要对数据进行重分布）。</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |



注意事项 
-------------------------

使用STR_HASH做拆分的表仅适用于点查场景，如果在业务中范围查询，则会直接触发全表扫描导致慢查询。

使用限制 
-------------------------

* 拆分键的数据类型需为字符串类型（CHAR或VARCHAR）。

* 不支持在建表完成后再调整STR_HASH的参数。




使用场景 
-------------------------

* 实现一个分表（或分库）只对应一个拆分表键的取值（字符串类型）的精准路由效果。例如，某个应用是按年月（YYYYMM）分库，然后按订单号分表，该应用的订单号有个特点，就是订单号的最后3位字符串是一个整数，其取值范围是000\~999。该应用的需求是需要在一个物理分库内，要将订单号后3位的每一个数值只单独路由到一个物理分表。那么，该应用分库采用YYYYMM，然后分表采用拆分函数STR_HASH，每个库1024个分表，就可以达到效果。具体的SQL语句如下：

  ```sql
  create table test_str_hash_tb (    
      id int NOT NULL AUTO_INCREMENT,
      order_id varchar(30) NOT NULL, 
      create_time datetime DEFAULT NULL,
      primary key(id)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  dbpartition by YYYYMM(`create_time`) 
  tbpartition by STR_HASH(`order_id`, -1, 3, 1) tbpartitions 1024;
  ```

  

  应用采用这样建表SQL，原因是分表的字符串截取后3位并转换为整数（整数范围是000\~999）后再取模做分表路由（共1024个分表），其路由结果能保证每一个物理分表只对一个拆分建的取值。而原来PolarDB-X默认拆分函数HASH无法达到这样的效果，是因为字符串经过hashCode计算后的整数是不可预知的，有可能会出现一个物理分表要对应多个不同拆分建的取值。
  

* STR_HASH拆分函数适用于使用字符串类型作为拆分键并且绝大部分都是点查的场景，如根据ID查交易订单、物流订单等。




使用示例 
-------------------------

假设order_id的类型为VARCHAR(32)，现在需要将order_id作为拆分键，计划分4个库，分8个表。

* 假设需要使用order_id的最后4位的字符串作为整数来计算分库分表路由，则您可以使用如下SQL进行建表。

  ```sql
  create table test_str_hash_tb (    
      id int NOT NULL AUTO_INCREMENT,
      order_id varchar(32) NOT NULL, 
      create_time datetime DEFAULT NULL,
      primary key(id)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  dbpartition by STR_HASH(`order_id`, -1, 4, 1) 
  tbpartition by STR_HASH(`order_id`, -1, 4, 1) tbpartitions 2;
  ```

  

* 假设需要截取order_id的第3个字符（即starIndex=2）与第7个字符（即endIndex=7）之间子串来计算分库分表路由，则您可以使用如SQL进行建表。

  ```sql
  create table test_str_hash_tb (    
      id int NOT NULL AUTO_INCREMENT,
      order_id varchar(32) NOT NULL, 
      create_time datetime DEFAULT NULL,
      primary key(id)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  dbpartition by STR_HASH(`order_id`, 2, 7) 
  tbpartition by STR_HASH(`order_id`, 2, 7) tbpartitions 2;
  ```

  

* 假设需要截取order_id的前5个字符串作为子串来计算分库分表路由，则您可以使用如SQL进行建表。

  ```sql
  create table test_str_hash_tb (    
      id int NOT NULL AUTO_INCREMENT,
      order_id varchar(32) NOT NULL, 
      create_time datetime DEFAULT NULL,
      primary key(id)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 
  dbpartition by STR_HASH(`order_id`, 5, -1) 
  tbpartition by STR_HASH(`order_id`, 5, -1) tbpartitions 2;
  ```

  




常见问题 
-------------------------

Q：`dbpartition by STR_HASH(order_id) `与`dbpartition by HASH(order_id)`有什么区别？

A：两者虽然都是直接根据字符串取值做分库分表的哈希路由，但是两者的分库分表的路由算法实现不一样。前者支持用户建表时自行设定截取子串相关参数，且在根据字符串的哈希值计算分库分表路由时是基于UNI_HASH算法进行计算；而后者是只对字符串的哈希值做简单取模。
