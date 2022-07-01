如何自定义SQL超时时间 
=================================



在PolarDB-X中，PolarDB-X节点与RDS的默认SQL执行超时时间是900秒（可以调整），但是对于某些特定的慢SQL，其执行时间可能超过了900秒 。针对这种慢SQL，PolarDB-X提供了调整超时时间的自定义HINT。通过这个自定义HINT可以任意调整SQL执行时长。

注意事项 
-------------------------

* PolarDB-X自定义HINT支持`/*+TDDL:hint_command*/`和`/!+TDDL:hint_command*/`两种格式。

* 如果使用`/*+TDDL:hint_command*/`格式，在使用MySQL官方命令行客户端执行带有PolarDB-X自定义HINT的SQL时，请在登录命令中加上-c参数。否则，由于PolarDB-X自定义HINT是以[MySQL 注释](https://dev.mysql.com/doc/refman/5.6/en/comments.html) 形式使用的，该客户端会将注释语句删除后再发送到服务端执行，导致PolarDB-X自定义HINT失效。具体请参见[MySQL 官方客户端命令](https://dev.mysql.com/doc/refman/5.6/en/mysql-command-options.html#option_mysql_comments) 。




语法 
-----------------------

PolarDB-X自定义SQL超时时间HINT的语法如下：

```sql
/*+TDDL:SOCKET_TIMEOUT(time)*/
```



其中，SOCKET_TIMEOUT的单位是毫秒。通过该HINT您可以根据业务需要，自由调整SQL语句的超时时间。

示例 
-----------------------

设置SQL超时时间为40秒：

```sql
/*+TDDL:SOCKET_TIMEOUT(40000)*/SELECT * FROM t_item;
```



超时时间设置得越长，占用数据库资源的时间就会越长。如果同一时间长时间执行的SQL过多，可能消耗大量的数据库资源，从而导致无法正常使用数据库服务。所以，对于长时间执行的SQL语句，尽量对SQL语句进行优化。
