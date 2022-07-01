SHOW DDL RESULT 
====================================

可以通过`SHOW DDL RESULT`命令查看近期执行过的DDL任务的结果。

语法 
-----------------------

```sql
SHOW DDL RESULT;
```



示例 
-----------------------

```sql
mysql> show ddl result\G;
*************************** 1. row ***************************
        JOB_ID: 1359590227445809152
   SCHEMA_NAME: test
   OBJECT_NAME: test_table
      DDL_TYPE: CREATE_TABLE
   RESULT_TYPE: SUCCESS
RESULT_CONTENT: SUCCESS
1 row in set (0 min 0.83 sec)
```



返回参数说明 
---------------------------



|      参数名称      |                       说明                        |
|----------------|-------------------------------------------------|
| JOB_ID         | DDL任务唯一标识，取值需为64位有符号长整型数值。                      |
| SCHEMA_NAME    | DDL任务对象的Schema名称。                               |
| OBJECT_NAME    | DDL任务对象名称，例如，当前执行DDL的表名称。                       |
| DDL_TYPE       | DDL任务类型，例如，`CREATE_TABLE`。 |
| RESULT_TYPE    | DDL任务执行结果的类型。                                   |
| RESULT_CONTENT | DDL任务执行结果的详细信息。                                 |


