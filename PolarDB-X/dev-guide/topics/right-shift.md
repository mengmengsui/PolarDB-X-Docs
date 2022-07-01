RIGHT_SHIFT 
================================

本文将介绍RIGHT_SHIFT函数的使用方式。

描述 
-----------------------

根据分库键的键值（键值必须是整数）有符号地向右移二进制指定的位数（位数可通过DDL指定），然后将得到的整数值按分库（表）数目取余。

使用限制 
-------------------------

拆分键的类型必须整数类型。

使用场景 
-------------------------

当拆分键大部分键值的低位部分区分度比较低而高位部分区分度比较高时，则适用于通过此拆分函数提高散列结果的均匀度。

例如有4个拆分键的键值，分别为0x0100、0x0200、0x0300和0x0400， 这4个值的第8位部分都是0。通常一些业务后N位可能只是一些业务上的标志位，如果直接对键值进行取余散列，其散列效果可能会比较差。但如果通过RIGHT_SHIFT（shardKey, 8）将拆分键的值进行二进制右移8位，则分别变成了0x01、0x02、0x03和0x04，这样的散列效果就会比较均匀（若分4个库，刚好可以每个值对应一个分库）。

使用示例 
-------------------------

假设需要将ID作为拆分键，并将ID的值向右移二进制8位的值作为哈希值，则可以如下建表：

```sql
create table test_hash_tb (    
    id int, 
    name varchar(30) DEFAULT NULL,  
    create_time datetime DEFAULT NULL,
    primary key(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 
dbpartition by RIGHT_SHIFT(id, 8) 
tbpartition by RIGHT_SHIFT(id, 8) tbpartitions 4;
```


