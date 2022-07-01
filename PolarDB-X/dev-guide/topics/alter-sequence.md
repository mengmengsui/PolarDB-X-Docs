修改Sequence 
===============================

本文主要介绍如何对Sequence的各种类型进行修改。

PolarDB-X支持对Sequence的各种类型进行如下修改：

* 修改Simple Sequence的参数：起始值、步长、最大值、循环或非循环。

* 修改Group Sequence或单元化 Group Sequence的参数：起始值。

* 不同类型Sequence间的转换（单元化Group Sequence除外）。




注意事项 
-------------------------

在对Sequence的类型进行修改时，您需要注意如下事项：

* Group Sequence和单元化Group Sequence是非连续的。START WITH参数对于它们仅具有指导意义，Group Sequence和单元化Group Sequence不会严格按照该参数作为起始值，但是保证起始值比该参数大。

* 单元化Group Sequence不支持转换到其它类型或修改单元化相关的参数。

* 对于Simple Sequence，如果修改Sequence时指定了START WITH，则会立即生效，下次取Sequence值时会从新的START WITH值开始。比如原先Sequene增长到100，这时把START WITH值改成了200，那么下一次获取的Sequence值就从200开始。

* 修改START WITH的参数值时，需要仔细评估已经产生的Sequence值，以及生成新Sequence值的速度，防止产生冲突。如非必要，请谨慎修改START WITH参数值。




Group Sequence 
-----------------------------------

**语法**

```sql
ALTER SEQUENCE <name> [ CHANGE TO SIMPLE | TIME ]
START WITH <numeric value>
[ INCREMENT BY <numeric value> ]
[ MAXVALUE <numeric value> ]
[ CYCLE | NOCYCLE ]
```

 **参数说明** 

|       参数        |                                                                     说明                                                                     |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| START WITH      | Sequence的起始值，无默认值，若未指定则忽略该参数，在转换类型时必须指定。                                                                                                   |
| INCREMENT BY    | 仅在将Group Sequence转换为Simple Sequence时有效，是Simple Sequence每次增长时的增量值（或称为间隔值或步长），若未指定，则默认值为1。                                                   |
| MAXVALUE        | 仅在将Group Sequence转换为Simple Sequence时有效，是Simple Sequence允许的最大值，若未指定，则默认值为有符号长整型（Signed BIGINT）的最大值，即9223372036854775807。                    |
| CYCLE 或 NOCYCLE | 仅在将Group Sequence转换为Simple Sequence时有效，两者只能选择其一，代表当Simple Sequence值增长到最大值后，是否允许继续循环（即从START WITH重新开始）使用该Simple Sequence，若未指定，则默认值为NOCYCLE。 |


**说明** 当修改的目标类型为TIME时，不支持上述参数。

单元化Group Sequence 
--------------------------------------

**语法**

```sql
ALTER SEQUENCE <name> 
START WITH <numeric value>
```

 **参数说明** 

|     参数     |                   说明                   |
|------------|----------------------------------------|
| START WITH | 单元化Group Sequence的起始值，无默认值，若未指定则忽略该参数。 |


**说明** **单元化Group Sequence** 不支持转换到其它类型或修改单元化相关的参数。

Time-based Sequence 
----------------------------------------

**语法**

```sql
ALTER SEQUENCE <name>[ CHANGE TO GROUP | SIMPLE ]
START WITH <numeric value>
[ INCREMENT BY <numeric value> ]
[ MAXVALUE <numeric value> ]
[ CYCLE | NOCYCLE ]
```

 **参数说明** 

|      参数       |                                                                     说明                                                                     |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| START WITH    | Sequence的起始值，无默认值，若未指定则忽略该参数，在转换类型时必须指定。                                                                                                   |
| INCREMENT BY  | Simple Sequence每次增长时的增量值（或称为间隔值或步长），若未指定，则默认值为1，将Simple Sequence转换为Group Sequence时该参数无效。                                                   |
| MAXVALUE      | Simple Sequence允许的最大值，若未指定，则默认值为有符号长整型（Signed BIGINT）的最大值，即9223372036854775807，将Simple Sequence转换为Group Sequence时该参数无效。                    |
| CYCLE或NOCYCLE | 两者只能选择其一，代表当Simple Sequence值增长到最大值后，是否允许继续循环（即仍从START WITH开始）使用该Simple Sequence，若未指定，则默认值为NOCYCLE，将Simple Sequence转换为Group Sequence时该参数无效。 |



Simple Sequence 
------------------------------------

**语法**

```sql
ALTER SEQUENCE <name> [ CHANGE TO GROUP | TIME ]
START WITH <numeric value>
[ INCREMENT BY <numeric value> ]
[ MAXVALUE <numeric value> ]
[ CYCLE | NOCYCLE ]
```

 **参数说明** 

|       参数        |                                                                     说明                                                                     |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| START WITH      | Sequence的起始值，无默认值，若未指定则忽略该参数，在转换类型时必须指定。                                                                                                   |
| INCREMENT BY    | Simple Sequence每次增长时的增量值（或称为间隔值或步长），若未指定，则默认值为1，将Simple Sequence转换为Group Sequence时该参数无效。                                                   |
| MAXVALUE        | Simple Sequence允许的最大值，若未指定，则默认值为有符号长整型（Signed BIGINT）的最大值，即9223372036854775807，将Simple Sequence转换为Group Sequence时该参数无效。                    |
| CYCLE 或 NOCYCLE | 两者只能选择其一，代表当Simple Sequence值增长到最大值后，是否允许继续循环（即仍从START WITH开始）使用该Simple Sequence，若未指定，则默认值为NOCYCLE，将Simple Sequence转换为Group Sequence时该参数无效。 |


**说明** 当修改的目标类型为TIME时，不支持上述参数。

不同类型Sequence间的转换 
-------------------------------------

在对Sequence的不同类型进行转换时，您需要了解如下事项：

* 通过`ALTER SEQUENCE`的`CHANGE TO <sequence_type>`子句实现。

* `ALTER SEQUENCE`如果指定了`CHANGE TO`子句，则强制必须加上START WITH参数，避免忘记指定起始值而造成取值时得到重复值；若没有CHANGE TO（可选参数），则不强制。

* 不支持单元化Group Sequence作为源或目标的类型转换。




示例 
-----------------------

* 将Simple Sequence seq4的起始值改为3000，步长改为5，最大值改为1000000，增长到最大值后改为继续循环。语句如下：

  ```sql
  mysql> ALTER SEQUENCE seq4 START WITH 3000 INCREMENT BY 5 MAXVALUE 1000000 CYCLE;
  ```

  

* 将Group Sequence转换为Simple Sequence。

  ```sql
  mysql> ALTER SEQUENCE seq1 CHANGE TO SIMPLE START WITH 1000000;
  ```

  



