如何使用HINT 
=============================

本文介绍了HINT的语法级示例。

HINT作为一种SQL补充语法，在关系型数据库中扮演着非常重要的角色。它允许用户通过相关的语法影响SQL的执行方式，对SQL进行特殊的优化。同样，PolarDB-X也提供了特殊的HINT语法。

语法 
-----------------------

```sql
/*+TDDL: hint_command [hint_command ...]*/
/!+TDDL: hint_command [hint_command ...]*/
```


**说明** 如果使用 /\*+TDDL:hint_command\*/ 格式，在使用MySQL官方命令行客户端执行带有PolarDB-X自定义HINT的SQL时，请在登录命令中加上-c参数。否则，由于PolarDB-X自定义HINT是以 [MySQL 注释](https://dev.mysql.com/doc/refman/5.6/en/comments.html) 形式使用的，该客户端会将注释语句删除后再发送到服务端执行，导致PolarDB-X自定义HINT失效。详情请参见[MySQL 官方客户端命令](https://dev.mysql.com/doc/refman/5.6/en/mysql-command-options.html#option_mysql_comments) 。

示例 
-----------------------

```sql
# 查询每个分库中的物理表名
/*+TDDL:scan()*/SHOW TABLES;
# 将查询下发到RDS只读实例的0000分库上    
/*+TDDL:node(0) slave()*/SELECT * FROM t1;
# 强制指定workload为AP
/*+TDDL:WORKLOAD=AP*/SELECT * FROM t1;
```



PolarDB-X支持在HINT语句中使用多个HINT命令：

```sql
SELECT /*+TDDL:node(0) slave()*/ ...;
```



PolarDB-X不支持通过以下方式使用多个HINT命令：

```sql
# 不支持单条SQL语句中包含多个HINT语句
SELECT /*+TDDL:node(0)*/ /*+TDDL:slave()*/ ...;
# 不支持HINT语句中包含重复的HINT命令 
SELECT /*+TDDL:node(0) node(1)*/ ...;
```


