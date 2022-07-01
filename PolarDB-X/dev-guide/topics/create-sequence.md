创建Sequence 
===============================

本文主要介绍如何创建各种类型的Sequence。

Group Sequence 
-----------------------------------

**语法**

```sql
CREATE [ GROUP ] SEQUENCE <name>
[ START WITH <numeric value> ]
```

 **参数说明** 

|     参数     |                   说明                    |
|------------|-----------------------------------------|
| START WITH | Group Sequence 的起始值，若未指定，则默认起始值为100001。 |

 **示例**

创建一个Group Sequence：

* 方法一请在命令行输入如下代码：

  ```sql
  mysql> CREATE SEQUENCE seq1;
  ```

  

* 方法二请在命令行输入如下代码：

  ```sql
  mysql> CREATE GROUP SEQUENCE seq1;
  ```

  




单元化 Group Sequence 
---------------------------------------

**语法**

```sql
CREATE [ GROUP ] SEQUENCE <name>
[ START WITH <numeric value> ]
[ UNIT COUNT <numeric value> INDEX <numeric value> ]
```

 **参数说明** 

|     参数     |                                     说明                                     |
|------------|----------------------------------------------------------------------------|
| START WITH | 单元化Group Sequence的起始值，默认起始值依赖于单元数量和单元索引；若单元数量和单元索引未被指定或为默认值，则默认起始值为100001。 |
| UNIT COUNT | 单元化Group Sequence的单元数量，默认值为1。                                              |
| INDEX      | 单元化Group Sequence的单元索引，取值范围为 \[ 0, 单元数量 - 1 \]，默认值为0。                      |


**说明**

* 如果未指定类型关键字，则默认类型为 **Group Sequence** 。

* **Group Sequence** 和 **单元化Group Sequence** 是非连续的。START WITH参数对于它们仅具有指导意义， **Group Sequence** 和 **单元化Group Sequence** 不会严格按照该参数作为起始值，但是保证起始值比该参数大。

* 可以将 **Group Sequence** 看作 **单元化Group Sequence** 的一个特例，即UNIT COUNT = 1 且 INDEX = 0 时的 **单元化Group Sequence** 。



**示例**

创建包含3个单元的全局唯一数字序列（将3个同名的、指定了相同单元数量和不同单元索引的单元化Group Sequence，分别用于3个不同的实例或库，组成一个全局唯一数字序列）。

1. 实例1/库1：

   ```sql
   mysql> CREATE GROUP SEQUENCE seq2 UNIT COUNT 3 INDEX 0;
   ```

   

2. 实例2/库2：

   ```sql
   mysql> CREATE GROUP SEQUENCE seq2 UNIT COUNT 3 INDEX 1;
   ```

   

3. 实例3/库3：

   ```sql
   mysql> CREATE GROUP SEQUENCE seq2 UNIT COUNT 3 INDEX 2;
   ```

   




Time-based Sequence 
----------------------------------------

**语法**

```sql
CREATE TIME SEQUENCE <name>
```


**注意** 存储 **Time-based Sequence** 值的列必须为BIGINT类型。
**示例**

创建一个Time-based Sequence：

请在命令行输入如下代码：

```sql
7mysql> CREATE TIME SEQUENCE seq3;
```



Simple Sequence 
------------------------------------

**语法**

```sql
CREATE SIMPLE SEQUENCE <name>
[ START WITH <numeric value> ]
[ INCREMENT BY <numeric value> ]
[ MAXVALUE <numeric value> ][ CYCLE | NOCYCLE ]
```

 **参数说明** 

|       参数        |                                                 说明                                                 |
|-----------------|----------------------------------------------------------------------------------------------------|
| START WITH      | Simple Sequence的起始值，若未指定，则默认起始值为1。                                                                 |
| INCREMENT BY    | Simple Sequence每次增长时的增量值（或称为间隔值或步长），若未指定，则默认值为1。                                                   |
| MAXVALUE        | Simple Sequence允许的最大值，若未指定，则默认值为有符号长整型（Signed BIGINT）的最大值，即9223372036854775807。                    |
| CYCLE 或 NOCYCLE | 两者只能选择其一，代表当Simple Sequence增长到最大值后，是否允许继续循环（即从START WITH重新开始）使用该Simple Sequence。若未指定，则默认值为NOCYCLE。 |

 **示例**

创建一个Simple Sequence，起始值是1000，步长为2，最大值为99999999999，增长到最大值后不继续循环。

请在命令行输入如下代码：

```sql
mysql> CREATE SIMPLE SEQUENCE seq4 START WITH 1000 INCREMENT BY 2 MAXVALUE 99999999999 NOCYCLE;
```


