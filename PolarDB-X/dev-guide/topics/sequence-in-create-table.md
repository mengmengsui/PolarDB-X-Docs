CREATE TABLE 
=================================

在为拆分表或广播表的主键定义AUTO_INCREMENT后，Sequence可以用于自动填充主键，由PolarDB-X自动维护。

扩展标准建表语法，增加了自增列的Sequence类型，如果未指定类型关键字，则默认类型为GROUP。PolarDB-X自动创建的、跟表相关联的Sequence名称，都是以`AUTO_SEQ_`为前缀，后面加上表名。

Group Sequence、Time-based Sequence或Simple Sequence 
-----------------------------------------------------------------------

创建 **Group Sequence** 、 **Time-based Sequence** 或 **Simple Sequence** 作为自增列的表语法如下：

```sql
CREATE TABLE <name> (
   <column> ... AUTO_INCREMENT [ BY GROUP | SIMPLE | TIME ],
   <column definition>,
   ...
) ... AUTO_INCREMENT=<start value>
```


**说明** 如果指定了`BY TIME`，即 **Time-based Sequence** ，则该列类型必须为BIGINT。

单元化Group Sequence 
--------------------------------------

创建 **单元化Group Sequence** 的表语法如下：

```sql
CREATE TABLE <name> (
   <column> ... AUTO_INCREMENT [ BY GROUP ] [ UNIT COUNT <numeric value> INDEX <numeric value> ],
   <column definition>,
   ...
) ... AUTO_INCREMENT=<start value>
```



示例 
-----------------------

**示例一：默认创建一张使用Group Sequence作为自增列的表。** 

请在命令行输入如下代码：

```sql
mysql> CREATE TABLE tab1 (
col1 BIGINT NOT NULL AUTO_INCREMENT,
col2 VARCHAR(16),
PRIMARY KEY(col1)
) DBPARTITION BY HASH(col1);
```



**示例二：创建3张同名的、使用相同单元数量和不同单元索引的单元化Group Sequence作为自增列的表，分别用于3个不同的实例或库。** 

1. 实例1/库1请在命令行输入如下代码：

   ```sql
   mysql> CREATE TABLE tab2 (
   col1 BIGINT NOT NULL AUTO_INCREMENT UNIT COUNT 3 INDEX 0,
   col2 VARCHAR(16),
   PRIMARY KEY(col1)
   ) DBPARTITION BY HASH(col1);
   ```

   

2. 实例2/库2请在命令行输入如下代码：

   ```sql
   mysql> CREATE TABLE tab2 (
   col1 BIGINT NOT NULL AUTO_INCREMENT UNIT COUNT 3 INDEX 1,
   col2 VARCHAR(16),
   PRIMARY KEY(col1)
   ) DBPARTITION BY HASH(col1);
   ```

   

3. 实例3/库3请在命令行输入如下代码：

   ```sql
   mysql> CREATE TABLE tab2 (
   col1 BIGINT NOT NULL AUTO_INCREMENT UNIT COUNT 3 INDEX 2,
   col2 VARCHAR(16),
   PRIMARY KEY(col1)
   ) DBPARTITION BY HASH(col1);
   ```

   




**示例三：创建一张使用Time-based Sequence作为自增列的表。** 

请在命令行输入如下代码：

```sql
mysql> CREATE TABLE tab3 (
col1 BIGINT NOT NULL AUTO_INCREMENT BY TIME, 
col2 VARCHAR(16), 
PRIMARY KEY(col1)
) DBPARTITION BY HASH(col1);
```



**示例四：创建一张使用Simple Sequence作为自增列的表。** 

请在命令行输入如下代码：

```sql
mysql> CREATE TABLE tab4 ( 
col1 BIGINT NOT NULL AUTO_INCREMENT BY SIMPLE, 
col2 VARCHAR(16), 
PRIMARY KEY(col1)
) DBPARTITION BY HASH(col1);
```


