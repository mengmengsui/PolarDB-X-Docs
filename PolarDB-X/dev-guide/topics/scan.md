扫描全部/部分分库分表 
================================

本文介绍了扫描全部/部分分库分表的HINT语法和示例。

除了可以将SQL单独下发到一个或多个分库执行，PolarDB-X还提供了扫描全部/部分分库与分表的`SCAN HINT`。使用`SCAN HINT`，您可以一次将SQL下发到每一个分库执行, 比如查看某个分库上的所有分表，或者查看某个逻辑表的每张物理表中的数据量等。

通过`SCAN HINT`，可以指定四种执行SQL的方式：

1. 在所有分库的所有分表上执行；

2. 在指定分库的所有分表上执行；

3. 在指定分库分表上执行，根据条件计算物理表名称；

4. 在指定分库分表上执行，显式指定物理表名；




`SCAN HINT`支持 DML、DDL和部分DAL语句。

语法 
-----------------------

```sql
# SCAN HINT
# 将SQL语句下发到所有分库的所有分表上执行
SCAN()                               
# 将SQL语句下发到指定分库的所有分表上执行
SCAN(NODE="node_list")               # 指定分库
# 将SQL语句下发到指定分库分表上执行，根据条件计算物理表名称
SCAN(
  [TABLE=]"table_name_list"          # 逻辑表名
  , CONDITION="condition_string"     # 使用TABLE和CONDITION中的内容计算物理库表名称
  [, NODE="node_list"] )             # 过滤通过CONDITION计算出的结果，仅保留指定物理库
# 将SQL语句下发到指定分库分表上执行，显式指定物理表名
SCAN(
  [TABLE=]"table_name_list"          # 逻辑表名
  , REAL_TABLE=("table_name_list")   # 物理表名，对所有物理库使用相同的物理表名
  [, NODE="node_list"] )             # 过滤通过CONDITION计算出的结果，仅保留指定物理库
# 物理/逻辑表名列表
table_name_list: 
    table_name [, table_name]...
# 物理库列表，支持GROUP_KEY和GROUP的序号, 可以通过`SHOW NODE`语句获得 
node_list: 
    {group_key | group_index} [, {group_key | group_index}]...
# 支持SQL WHERE的语法，需要为每一张表设置条件，如：t1.id = 2 and t2.id = 2
condition_string: 
    where_condition
            
```



注意事项 
-------------------------

* PolarDB-X自定义HINT支持`/*+TDDL:hint_command*/`和`/!+TDDL:hint_command*/`两种格式。

* 如果使用`/*+TDDL:hint_command*/`格式，在使用MySQL官方命令行客户端执行带有PolarDB-X自定义HINT的SQL时，请在登录命令中加上`-c` 参数。否则，由于PolarDB-X自定义HINT是以[MySQL 注释](https://dev.mysql.com/doc/refman/5.6/en/comments.html) 形式使用的，该客户端会将注释语句删除后再发送到服务端执行，导致PolarDB-X自定义HINT失效。具体请查看[MySQL 官方客户端命令](https://dev.mysql.com/doc/refman/5.6/en/mysql-command-options.html#option_mysql_comments) 。




示例 
-----------------------

* 在所有分库的所有分表上执行：

  ```sql
  SELECT /*+TDDL:scan()*/ COUNT(1) FROM t1      
  ```

  

  执行后会下发SQL语句到`t1`的所有物理表上执行，并将结果集合并后返回。
  

* 在指定分库的所有分表上执行：

  ```sql
  SELECT /*+TDDL:scan(node='0,1,2')*/ COUNT(1) FROM t1          
  ```

  

  执行后会首先计算出`t1`在0000, 0001, 0002分库上的所有物理表，然后下发SQL语句并将结果集合并后返回。
  

* 按条件在指定分表上执行：

  ```sql
  SELECT /*+TDDL:scan('t1', condition='t1.id = 2')*/ COUNT(1) FROM t1           
  ```

  

  执行后会首先计算出逻辑表`t1`满足`condition`条件的所有物理表，然后下发SQL语句并将结果集合并后返回。
  

* 按条件在指定分表上执行，有JOIN的情况：

  ```sql
  SELECT /*+TDDL:scan('t1, t2', condition='t1.id = 2 and t2.id = 2')*/ * FROM t1 a JOIN t2 b ON a.id = b.id WHERE b.name = "test"
              
  ```

  

  执行后会首先计算出逻辑表`t1``t2`满足 `condition` 条件的所有物理表，然后下发 SQL 语句并将结果集合并后返回。 
  
**注意** ：使用该自定义注释需要保证两张表的分库和分表数量一致，否则PolarDB-X计算出的两个键值对应的分库不一致，就会报错。
  

* 在指定分库分表上执行，显式指定物理表名：

  ```sql
  SELECT /*+TDDL:scan('t1', real_table=("t1_00", "t1_01"))*/ COUNT(1) FROM t1
              
  ```

  

  执行后会下发SQL语句到所有分库的```t1_00``t1_01```分表上，合并结果集后返回。
  

* 在指定分库分表上执行，显式指定物理表名, 有JOIN的情况：

  ```sql
  SELECT /*+TDDL:scan('t1, t2', real_table=("t1_00,t2_00", "t1_01,t2_01"))*/ * FROM t1 a JOIN t2 b ON a.id = b.id WHERE b.name = "test";
              
  ```

  

  执行后会下发SQL语句到所有分库的`t1_00``t2_00``t1_01``t2_01`分表上，合并结果集后返回。
  



