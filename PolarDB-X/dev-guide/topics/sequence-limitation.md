使用限制 
=========================

本文将介绍使用Sequence过程中的注意事项及问题处理的方法。

限制与注意事项 
----------------------------

在使用Sequence时，您需要注意如下事项：

* 转换Sequence类型时，必须指定START WITH起始值。

* 单元化Group Sequence不支持作为源或目标的类型转换，也不支持起始值以外的参数修改。

* 属于同一个全局唯一数字序列分配空间的每个单元化Group Sequence ，必须指定相同的单元数量和不同的单元索引。

* 在PolarDB-X非拆分模式库（即后端仅关联一个已有的RDS物理库）、或拆分模式库中仅有单表（即所有表都是单库单表，且无广播表）的场景下执行INSERT时， PolarDB-X会自动优化并直接下推语句，绕过优化器中分配Sequence值的部分。此时`INSERT INTO ... VALUES （seq.nextval, ...)`这种用法不支持，建议使用后端RDS/MySQL自增列机制代替。

* 如果将指定分库的Hint用在INSERT语句上，比如INSERT INTO ... VALUES ... 或INSERT INTO ... SELECT ...，且目标表使用了Sequence，则PolarDB-X会绕过优化器直接下推语句，使Sequence不生效，目标表最终会使用后端RDS/MySQL表中的自增机制生成id。

* 必须对同一个表采用一种统一的方式分配自增id：或者依赖于Sequence，或者依赖于后端RDS/MySQL表的自增列；应避免两种机制混用，否则很可能会造成id冲突（INSERT时产生重复id）的情况，且难于排查。

* 将Time-based Sequence用于表中自增列时，该列必须使用BIGINT类型。




如何处理主键冲突 
-----------------------------

如果直接在RDS中写入了数据，而对应的主键值不是PolarDB-X生成的Sequence值，那么后续让PolarDB-X自动生成主键写入数据库，可能会和这些数据发生主键冲突，您可以通过如下步骤解决此问题：

1. 通过`SHOW SEQUENCES`来查看当前已有Sequence。AUTO_SEQ_ 开头的Sequence是隐式Sequence（创建表时加上AUTO_INCREMENT参数的字段产生的Sequence）。

   请在命令行输入如下代码：

   ```sql
   mysql> SHOW SEQUENCES;
   ```

   

   返回结果如下：

   ```sql
   +---------------------+-------+--------------+------------+-----------+-------+-------+ 
   | NAME                | VALUE | INCREMENT_BY | START_WITH | MAX_VALUE | CYCLE | TYPE  | 
   +---------------------+-------+--------------+------------+-----------+-------+-------+ 
   | AUTO_SEQ_xkv_t_item | 0     | N/A          | N/A        | N/A       | N/A   | GROUP | 
   | AUTO_SEQ_xkv_shard  | 0     | N/A          | N/A        | N/A       | N/A   | GROUP | 
   +---------------------+-------+--------------+------------+-----------+-------+-------+ 
   2 rows in set (0.04 sec)
   ```

   

2. 若xkv_t_item表有冲突，并且xkv_t_item表主键是ID，那么从PolarDB-X获取这个表最大主键值。

   请在命令行输入如下代码：

   ```sql
   mysql> SELECT MAX(id) FROM xkv_t_item;
   ```

   

   返回结果如下：

   ```sql
   +-----------+ 
   | MAX(id)   | 
   +-----------+ 
   | 8231      | 
   +-----------+ 
   1 row in set (0.01 sec)
   ```

   

3. 更新Sequence表中对应的值，这里更新成比8231要大的值，比如9000，更新完成后，后续插入语句生成的自增主键将不再报错。请在命令行输入如下代码：

   ```sql
   mysql> ALTER SEQUENCE AUTO_SEQ_xkv_t_item START WITH 9000;
   ```

   



