ALTER TABLE 
================================

本文主要介绍如何对表相关的Sequence类型进行修改。

暂不支持通过`ALTER TABLE`来修改对应Sequence的类型，但您可以参见如下语法通过`ALTER TABLE`修改起始值：

```sql
ALTER TABLE <name> ... AUTO_INCREMENT=<start value>
```


**说明**

* 如果想要修改表相关的Sequence类型，需要通过`SHOW SEQUENCES`指令查找出Sequence的具体名称和类型，然后再用`ALTER SEQUENCE`指令去修改，具体操作请参见[修改Sequence](alter-sequence.md)。

* 使用Sequence后，请谨慎修改`AUTO_INCREMENT`的起始值（仔细评估已经产生的Sequence值，以及生成新Sequence值的速度，防止产生冲突）。



