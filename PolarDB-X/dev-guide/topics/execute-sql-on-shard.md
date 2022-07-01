指定分库执行SQL 
==============================

本文介绍了指定分库执行SQL的HINT语法和示例。

在使用PolarDB-X的过程中，如果遇到某个PolarDB-X不支持的SQL语句，可以通过PolarDB-X提供的`NODE HINT`，直接将SQL下发到一个或多个分库上去执行。此外如果需要单独查询某个分库或者已知分库的某个分表中的数据，也可以使用`NODE HINT`，直接将SQL语句下发到分库中执行。

语法 
-----------------------

`NODE HINT`支持通过分片名指定SQL在分库上执行。其中分片名是PolarDB-X中分库的唯一标识，可以通过`SHOW NODE`语句得到。

通过分库名指定SQL在分库上执行分两种使用方式，分别是指定SQL在某个分库上执行和指定SQL在多个分库上执行。

**注意** 如果在目标表包含Sequence的INSERT语句上使用了指定分库的HINT，那么Sequence将不生效。更多相关信息，请参考[使用限制](sequence-limitation.md)。

* 指定SQL在某个分库上执行：

  ```sql
  /*+TDDL:node('node_name')*/            
  ```

  

  `node_name`为分片名，通过这个PolarDB-X自定义HINT，就可以将SQL下发到`node_name`对应的分库中执行。
  

* 指定SQL在多个分库上执行：

  ```sql
  /*+TDDL:node('node_name'[,'node_name1','node_name2'])*/               
  ```

  

  在参数中指定多个分片名，将SQL下发到多个分库上执行，分片名之间使用逗号分隔。
  
**说明**
  * 使用该自定HINT 时，PolarDB-X会将SQL直接下发到分库上执行，所以在SQL语句中，表名必须是该分库中已经存在的表名。
  
  * `NODE HINT`支持 DML、DDL、DAL语句。
  

  
  




注意事项 
-------------------------

* PolarDB-X自定义HINT支持`/*+TDDL:hint_command*/`和`/!+TDDL:hint_command*/`两种格式。

* 如果使用`/*+TDDL:hint_command*/`格式，在使用MySQL官方命令行客户端执行带有PolarDB-X自定义HINT的SQL时，请在登录命令中加上`-c` 参数。否则，由于PolarDB-X自定义HINT是以[MySQL 注释](https://dev.mysql.com/doc/refman/5.6/en/comments.html) 形式使用的，该客户端会将注释语句删除后再发送到服务端执行，导致PolarDB-X自定义HINT失效。具体请查看[MySQL 官方客户端命令](https://dev.mysql.com/doc/refman/5.6/en/mysql-command-options.html#option_mysql_comments) 。




示例 
-----------------------

对于名为`drds_test`的PolarDB-X数据库，`SHOW NODE`的结果如下：

```sql
mysql> SHOW NODE\G
*************************** 1. row ******************
                 ID: 0
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0000_RDS
  MASTER_READ_COUNT: 212
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 2. row ******************
                 ID: 1
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0001_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 3. row ******************
                 ID: 2
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0002_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 4. row ******************
                 ID: 3
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0003_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 5. row ******************
                 ID: 4
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0004_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 6. row ******************
                 ID: 5
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0005_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 7. row ******************
                 ID: 6
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0006_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
*************************** 8. row ******************
                 ID: 7
               NAME: DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0007_RDS
  MASTER_READ_COUNT: 29
   SLAVE_READ_COUNT: 0
MASTER_READ_PERCENT: 100%
 SLAVE_READ_PERCENT: 0%
8 rows in set (0.02 sec)
            
```



可以看到每个分库都有`NAME`这个属性，这就是分库的分片名。每个分片名都唯一对应一个分库名，比如`DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0003_RDS`这个分片名对应的分库名是`drds_test_vtla_0003`。得到了分片名，就可以使用PolarDB-X的自定义HINT指定分库执行SQL语句了。

* 指SQL在第0个分库上执行：

  ```sql
  SELECT /*TDDL:node('DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0000_RDS')*/ * FROM table_name;                 
  ```

  

* 指定SQL在多个分库上执行：

  ```sql
  SELECT /*TDDL:node('DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0000_RDS','DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0006_RDS')*/ * FROM table_name;
                
  ```

  

  这条SQL语句将在`DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0000_RDS`、`DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0006_RDS`这两个分片上执行。
  

* 查看SQL在第0个分库上物理执行计划：

  ````sql
  /*TDDL:node('DRDS_TEST_1473471355140LRPRDRDS_TEST_VTLA_0000_RDS')*/ EXPLAIN SELECT * FROM table_name; ```
  ````

  



