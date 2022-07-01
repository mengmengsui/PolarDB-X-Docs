CREATE DATABASE 
====================================

CREATE DATABASE语句用于创建数据库，并可以指定数据库的默认属性（如数据库默认字符集，校验规则等）。

语法 
-----------------------

```sql
create_database_stmt:
    CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] database_name [database_option_list]
database_option_list:
    database_option [database_option ...]
database_option:
    [DEFAULT] {CHARACTER SET | CHARSET} [=] charset_name
  | [DEFAULT] COLLATE [=] collation_name
  |  LOCALITY=locality_option}
  | [PARTITION_MODE = partition_mode_option]
locality_option:
    'dn=storage_inst_id_list'
storage_inst_id_list:
    storage_inst_id[,storage_inst_id_list]
partition_mode_option:
     'partitioning'
    |'sharding'
```



参数说明 
-------------------------



|           参数           |                                                                                                                                                                     说明                                                                                                                                                                      |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| database_name          | 指定要修改属性的数据库名称。如果不指定，会对当前默认数据库进行修改。                                                                                                                                                                                                                                                                                                          |
| CHARSET charset_name   | 指定要修改的字符集。                                                                                                                                                                                                                                                                                                                                  |
| COLLATE collation_name | 指定校对规则。                                                                                                                                                                                                                                                                                                                                     |
| LOCALITY               | 创建数据库时指定该库的存储位置。                                                                                                                                                                                                                                                                                                                            |
| PARTITION_MODE         | 指定逻辑库所使用的分区模式，支持两种分区模式： * partitioning：使用MySQL类型的分区表建表语法（例如，partition byHash/Range/List）进行分区建表。  * sharding：使用DRDS模式的分库分表建表语法（例如，dbpartition by / tbpartition by）。   如果不指定，默认是partitioning。 数据库的分区模式不可更改，建库时一旦指定，不能变更。 |



示例 
-----------------------

* 创建数据库test，并指定字符集为UTF8。

  ```sql
  mysql> create database test PARTITION_MODE=sharding default CHARACTER SET UTF8;
  Query OK, 1 row affected (0.00 sec)
  ```

  

* 在实例中创建一个数据库，并通过以下命令指定其存储位置为 *polardbx-storage-0-master* 节点。

  ```sql
  mysql> CREATE DATABASE db1 PARTITION_MODE=sharding LOCALITY='dn=polardbx-storage-0-master';
  ```

  
  **说明**
  * 如果在创建数据库时未指定数据库的存储位置，系统将默认在所有存储节点中均匀分布数据库。
  
  * 数据库中分表的存储位置与该库的存储位置保持一致，以实现分表上的数据隔离。
  

  
  




创建成功后，您可以通过以下语句查看数据库的存储位置信息。

```sql
mysql> SHOW CREATE DATABASE db1 PARTITION_MODE=sharding;
```

返回结果如下所示：

```sql
+----------+------------------------------------------------------------------------+
| DATABASE | CREATE DATABASE                                                        |
+----------+------------------------------------------------------------------------+
| db1      | CREATE DATABASE `db1` /* LOCALITY = "dn=polardbx-storage-0-master" */  |
+----------+------------------------------------------------------------------------+
1 row in set
```



