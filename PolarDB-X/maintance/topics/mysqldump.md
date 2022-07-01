使用mysqldump导入导出数据 
======================================

本文介绍了通过mysqldump工具将PolarDB-X数据导入导出的几种常见场景和详细操作步骤。

PolarDB-X支持MySQL官方数据导出工具mysqldump。mysqldump命令的详细说明请参见[MySQL 官方文档](http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html) 。

**说明** mysqldump适合小数据量（低于1000万）的离线导入导出。如果需要完成更大数据量或者实时的数据迁移任务，建议使用DTS进行数据迁移。

mysqldump工具介绍 
----------------------------------

mysqldump能够导出表结构信息和表内数据，并转化成SQL语句的格式方便用户直接导入，SQL语法如下：

```sql
DROP TABLE IF EXISTS `table_name`;
CREATE TABLE `table_name` (
    `id` int(11) NOT NULL,
    `k` int(11) NOT NULL DEFAULT '0',
    ...
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4  dbpartition by hash(`id`);
INSERT INTO `table_name` VALUES (...),(...),...;
INSERT INTO `table_name` VALUES (...),(...),...;
...
```



mysqldump工具导出数据的命令使用方式举例：

```shell
shell> mysqldump -h ip -P port -u user -pPassword --default-character-set=char-set --net_buffer_length=10240 --no-create-db --no-create-info --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset  [--hex-blob] [--no-data] database [table1 table2 table3...] > dump.sql
```



mysqldump的参数说明可通过`mysqldump --help`命令查看或查询[MySQL 官方文档](http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html) ，常用参数说明如下，请根据实际情况输入：


|         参数名         |                                 说明                                  |
|---------------------|---------------------------------------------------------------------|
| ip                  | PolarDB-X实例的IP。                                                     |
| port                | PolarDB-X实例的端口。                                                     |
| user                | PolarDB-X的用户名。                                                      |
| password            | PolarDB-X的密码，注意前面有个-p，之间没有空格。                                       |
| char-set            | 指定的编码。                                                              |
| --hex-blob          | 使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用本选项。影响的字段类型包括BINARY、VARBINARY、BLOB。 |
| --no-data           | 不导出数据。                                                              |
| table               | 指定导出某个表。默认导出该数据库所有的表。                                               |
| --no-create-info    | 不导出建表信息                                                             |
| --net_buffer_length | 传输缓冲区大小。影响Insert语句的长度，默认值1046528。                                   |



导出的SQL语句格式文件，有两种方法导入数据库中：


* SOURCE语句导入数据

  ```shell
  ## 1.登录数据库
  shell> mysql -h ip -P port -u user -pPassword --default-character-set=char-set 
  ## 2.通过source语句执行文件中的sql语句导入数据
  mysql> source dump.sql
  ```

   

* mysql命令导入数据

  ```shell
  shell> mysql -h ip -P port -u user -pPassword --default-character-set=char-set< /yourpath/dump.sql
  ```

   




下面从不同场景介绍mysqldump工具的使用实例。

PolarDB-X和MySQL之间数据传输时不推荐导出表结构，因为PolarDB-X包含分库分表功能，[CREATE TABLE](../../dev-guide/topics/create-table.md)中的拆分函数等与MySQL不兼容，不兼容的关键字包括：


* DBPARTITION BY hash(partition_key)

* TBPARTITION BY hash(partition_key)

* TBPARTITIONS N

* BROADCAST




如果导出表结构，需在导出的SQL语句文件中修改建表语句，才能正确导入。所以推荐只导出表内数据，手动登录数据库进行建表操作，然后再导入数据。

场景一：从MySQL导入到PolarDB-X 
----------------------------------------

从MySQL导入数据到PolarDB-X，请按照以下步骤进行操作。

1. 从MySQL中导出数据到文件。 输入以下命令，从MySQL中导出表内数据（不推荐导出表结构），假设导出文件为dump.sql。

   ```shell
   mysqldump -h ip -P port -u user -pPassword --default-character-set=char-set --net_buffer_length=204800 --no-create-db --no-create-info --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset --hex-blob database [table1 table2 table3...] > dump.sql
   ```

    

2. 登录PolarDB-X，手动建立目标表，关于PolarDB-X建表语句的语法请参见[CREATE TABLE](../../dev-guide/topics/create-table.md)。如果未加--no-create-info参数，导出的dump.sql文件中包含MySQL端的建表语句，也可在文件中进行修改建表语句。

3. 导入数据文件到PolarDB-X中。您可以通过如下两种方式导入数据文件到PolarDB-X：
   * 通过`mysql -h ip -P port -u user -pPassword --default-character-set=char-set`命令登录目标PolarDB-X，执行`source /yourpath/dump.sql`命令将数据导入到目标PolarDB-X。
   
   * 直接通过`mysql -h ip -P port -u user -pPassword --default-character-set=char-set< /yourpath/dump.sql`命令将数据导入到目标PolarDB-X。
   

   
   **说明**
   * 上述两个命令中`default-character-set`要设置成实际的数据编码。如果是Windows平台，source命令指定的文件路径需要对分隔符转义。
   
   * 第一种方式会把所有的步骤回显到屏幕上，速度略慢，但是可以观察导入过程。
   
   * 导入的时候，由于某些PolarDB-X和MySQL实现上的不同，可能会报错，错误信息类似`ERROR 1231 (HY000): [a29ef6461c00000][10.117.207.130:3306][****]Variable @saved_cs_client can't be set to the value of @@character_set_client`。此类错误信息并不影响导入数据的正确性。
   

   
   




场景二：从一个PolarDB-X导入到另一个PolarDB-X 
-------------------------------------------------

假设您之前有一个测试环境的PolarDB-X，测试完毕以后，需要把测试过程中的一些表结构和数据导入到生产环境中的PolarDB-X中，那么可以按照以下步骤进行操作。

1. 从源PolarDB-X中导出数据到文本文件。请参见场景一步骤一。

2. 导入数据文件到PolarDB-X。请参见场景一步骤三。

3. 手动创建Sequence对象。 mysqldump并不会导出PolarDB-X中的Sequence对象，所以如果在源PolarDB-X中使用了Sequence对象，并且需要在目标PolarDB-X中继续使用相同的Sequence对象，则需要手动在目标PolarDB-X中创建同名的Sequence的对象。具体步骤如下：
   

   1. 在源PolarDB-X上执行SHOW SEQUENCES，获取当前PolarDB-X中的Sequence对象的状态。
   
   2. 在目标PolarDB-X数据库上通过CREATE SEQUENCE命令创建新的Sequence对象。
   

   

   Sequence命令详情请参见[Sequence](../../dev-guide/topics/sequence.md)。
   




场景三：从PolarDB-X导出数据到MySQL 
------------------------------------------

从PolarDB-X导出数据到MySQL，和在PolarDB-X之间相互导入数据的过程类似，也分为以下几个步骤。


1. 从源PolarDB-X中导出数据到文本文件。请参见场景一步骤一。

2. 登录MySQL，手动创建目标表。如果导出数据包含建表语句，则需要在导出文件中修改建表语句，删除PolarDB-X中不兼容MySQL的关键字等信息。

   例如PolarDB-X中的某个拆分表：

   ```sql
   CREATE TABLE `table_name` (
       `id` int(11) NOT NULL,
       `k` int(11) NOT NULL DEFAULT '0',
       ...
   ) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 dbpartition by hash(`id`);
   ```

   

   需去掉MySQL不兼容的拆分函数语句，改成：

   ```sql
   CREATE TABLE `table_name` (
       `id` int(11) NOT NULL,
       `k` int(11) NOT NULL DEFAULT '0',
       ...
   ) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
   ```

    

3. 导入数据文件到PolarDB-X。请参见场景一步骤三。




