错误码 
========================

本文档列出了PolarDB-X返回的常见错误码及解决方法。



## PXC-1305 ERR_UNKNOWN_SAVEPOINT

* 描述：指定名称的SAVEPOINT不存在。

* 示例：`ERR-CODE: [PXC-1305][ERR_UNKNOWN_SAVEPOINT] SAVEPOINT ***** does not exist `

* 说明：在PolarDB-X上执行`ROLLBACK TO SAVEPOINT`或者`RELEASE SAVEPOINT`命令时，如果指定的SAVEPOINT名称不存在，会提示PXC-1305错误。

  建议检查SAVEPOINT命令返回的名称是否和使用的名称一致。
  




## PXC-1094 ERR_UNKNOWN_THREAD_ID

* 描述：KILL命令指定的会话ID不存在。

* 示例：`ERR-CODE: [PXC-1094][ERR_UNKNOWN_THREAD_ID] Unknown thread id: *****`

* 说明：在PolarDB-X上执行`KILL`命令取消执行的SQL语句时，如果指定的会话ID不存在，或者对应的SQL语句已经结束执行，会提示PXC-1094错误。

  建议使用`SHOW PROCESSLIST`命令查看正在执行的SQL语句会话ID，并只针对返回的ID执行`KILL`命令。
  




PXC-4006 ERR_TABLE_NOT_EXIST 
-------------------------------------------------

* 描述：PolarDB-X数据表不存在。

* 示例：`ERR-CODE: [PXC-4006][ERR_TABLE_NOT_EXIST] Table '*****' doesn't exist.`

* 说明：该错误码表示PolarDB-X数据表不存在，或者由于未知原因，PolarDB-X无法加载数据表的元数据信息。




PXC-4007 ERR_CANNOT_FETCH_TABLE_META 
---------------------------------------------------------

* 描述：PolarDB-X无法加载数据表的元数据信息。

* 示例：`ERR-CODE: [PXC-4007][ERR_CANNOT_FETCH_TABLE_META] Table '*****' metadata cannot be fetched because Table '*****.*****' doesn't exist.`

* 说明：该错误码表示PolarDB-X尝试读取数据表的元数据信息失败。可能的错误原因如下：
  * 数据表未创建。
  
  * 维护的元数据库信息不一致。
  
  * 表被删除或者改名。
  
  
  

  出现该错误时，首先检查表名是否存在，或者执行`check table` 命令确认PolarDB-X维护的元数据库信息是否一致。如果确定表被人为删除或改名，可以通过PolarDB-X提供的数据恢复功能修复。如果仍无法修复，请联系技术支持。
  




PXC-4018 ERR_INVALID_DDL_PARAMS 
----------------------------------------------------

* 描述：PolarDB-X执行DDL过程失败。

* 示例：`ERR-CODE: [PXC-4018][ERR_INVALID_DDL_PARAMS] invalid '*****'.`

* 说明：该错误表示用户执行DDL的参数有错误，请检查参数正确性。如果确认参数正确，请联系技术支持。




PXC-4100 ERR_ATOM_NOT_AVALILABLE 
-----------------------------------------------------

* 描述：PolarDB-X后端数据节点暂时不可用。

* 示例：`ERR-CODE: [PXC-4100][ERR_ATOM_NOT_AVALILABLE] Atom : ***** isNotAvailable`

* 说明：如果PolarDB-X探测到后端某个数据节点状态异常，会临时阻止访问该实例并提示PXC-4100错误。

  当遇到该错误，请检查PolarDB-X后端所有数据节点是否异常。当后端数据节点从异常状态恢复后，PolarDB-X将自动解除不可用状态，恢复应用正常访问。
  




PXC-4101 ERR_ATOM_GET_CONNECTION_FAILED_UNKNOWN_REASON 
---------------------------------------------------------------------------

* 描述：PolarDB-X计算节点和数据节点连接获取失败。

* 示例：`ERR-CODE: [PXC-4101][ERR_ATOM_GET_CONNECTION_FAILED_UNKNOWN_REASON] Get connection for db '*****' from pool failed. AppName:*****, Env:*****, UnitName:null. Message from pool: wait millis 5000, active 0, maxActive 5. You should look for the following logs which contains the real reason.`

* 说明：PolarDB-X在处理请求时会向数据节点异步创建连接。如果无法在等待时间内完成数据节点连接创建，而异步任务又尚未返回错误原因，PolarDB-X会向应用返回PXC-4101错误。

  该错误通常是由后端数据节点异常导致的。如果排除数据节点问题后仍然出现该错误，请联系技术支持。
  




PXC-4102 ERR_ATOM_GET_CONNECTION_FAILED_KNOWN_REASON 
-------------------------------------------------------------------------

* 描述：已知原因的PolarDB-X后端连接获取失败。

* 示例：`ERR-CODE: [PXC-4102][ERR_ATOM_GET_CONNECTION_FAILED_KNOWN_REASON] Get connection for db '*****' failed because wait millis 5000, active 0, maxActive 5`

* 说明：PolarDB-X计算节点获取数据节点连接时出错，错误原因已经在ERR-CODE消息中给出。常见PolarDB-X数据节点连接失败的原因如下：
  * 后端数据节点连接数已满
  
  * 计算节点到数据节点的连接超时
  
  * 数据节点拒绝连接
  
  
  

  如果排除后端数据节点问题后仍然出现该错误，请联系技术支持。
  




PXC-4103 ERR_ATOM_CONNECTION_POOL_FULL 
-----------------------------------------------------------

* 描述：PolarDB-X后端数据节点连接池已满。

* 示例：`ERR-CODE: [PXC-4103][ERR_ATOM_CONNECTION_POOL_FULL] Pool of DB '*****' is full. Message from pool: wait millis 5000, active 5, maxActive 5. AppName:*****, Env:*****, UnitName:null.`

* 说明：该错误表示PolarDB-X后端连接池已满。导致PXC-4103错误的常见原因如下：
  * 应用SQL语句执行比较慢，占用单个连接的时间过长，导致连接数不够；
  
  * 应用端没有关闭数据库连接，导致连接泄露；
  
  * 有很多跨库查询（例如聚合统计类查询，未带分库条件的查询）同时执行，占用大量连接。
  
  
  

  解决方法建议如下：
  * 尽量使用框架访问数据库，如Spring JDBC、MyBatis等；
  
  * 按性能分析报告与DBA建议优化业务SQL语句；
  
  * 使用PolarDB-X读写分离将跨库查询转发至读库处理；
  
  * 升级更高规格的PolarDB-X，提升后端处理能力；
  
  * 联系技术支持调整PolarDB-X后端连接数。
  
  
  
  
  




PXC-4104 ERR_ATOM_CREATE_CONNECTION_TOO_SLOW 
-----------------------------------------------------------------

* 描述：PolarDB-X后端数据节点连接创建太慢。

* 示例：`ERR-CODE: [PXC-4104][ERR_ATOM_CREATE_CONNECTION_TOO_SLOW] Get connection for db '*****' from pool timeout. AppName:*****, Env:*****, UnitName:null. Message from pool: wait millis 5000, active 3, maxActive 5.`

* 说明：PolarDB-X向后端数据节点异步创建连接时，如果在短时间创建大量连接，或者数据节点建立连接速度太慢，会出现等待超时。该问题通常是由于后端数据节点压力过大或异常导致的，建议使用PolarDB-X读写分离，或者升级更高规格，减轻后端处理压力。

  如果排除后端数据节点问题后仍然出现该错误，请联系技术支持。如果问题是由短时间创建大量连接导致，建议联系技术支持调整PolarDB-X最小连接数。
  




PXC-4105 ERR_ATOM_ACCESS_DENIED 
----------------------------------------------------

* 描述：PolarDB-X后端数据节点拒绝创建连接。

* 示例：`ERR-CODE: [PXC-4105][ERR_ATOM_ACCESS_DENIED] DB '*****' Access denied for user '*****'@'*****'. AppName:*****, Env:*****, UnitName:null. Please contact DBA to check.`

* 说明：该错误表明PolarDB-X通过用户名和密码连接数据节点时被拒绝访问，请联系技术支持。




PXC-4106 ERR_ATOM_DB_DOWN 
----------------------------------------------

* 描述：PolarDB-X后端数据节点无法连接。

* 示例：`ERR-CODE: [PXC-4106][ERR_ATOM_DB_DOWN] DB '*****' cannot be connected. AppName:*****, Env:*****, UnitName:null. It seems a very real possibility that this DB IS DOWN. Please contact DBA to check.`

* 说明：该错误表明PolarDB-X向后端数据节点创建连接超时或者没有响应。遇到该错误的通常原因是数据节点故障，请联系技术支持。




PXC-4108 ERR_VARIABLE_CAN_NOT_SET_TO_NULL_FOR_NOW 
----------------------------------------------------------------------

* 描述：变量（variable）不允许被设置为NULL。

* 示例：`ERR-CODE: [PXC-4108][ERR_VARIABLE_CAN_NOT_SET_TO_NULL_FOR_NOW] System variable ***** can''t set to null for now;`

* 说明：有些数据节点变量（variable）不允许用`SET var = x`语句设置成NULL值。遇到这种情况，PolarDB-X会提示PXC-4108错误。




PXC-4200 ERR_GROUP_NOT_AVALILABLE 
------------------------------------------------------

* 描述：PolarDB-X下的某个数据节点暂时不可用。

* 示例：`ERR-CODE: [PXC-4200][ERR_GROUP_NOT_AVALILABLE] The TDDL Group ***** is running in fail-fast status, caused by this SQL:***** which threw a fatal exception as *****.`

* 说明：当分库包含的数据节点出现访问异常，并且分库下没有其他可用数据节点时，PolarDB-X会将分库置于fail-fast状态并提示PXC-4200错误。

  通常该错误是由于数据节点故障导致的。请根据包含的数据节点异常信息定位和解决问题。当故障数据节点恢复后，PolarDB-X将自动取消fail-fast状态。

  如果数据节点故障解决后仍然出现PXC-4200错误，请联系技术支持。
  




PXC-4201 ERR_GROUP_NO_ATOM_AVALILABLE 
----------------------------------------------------------

* 描述：PolarDB-X分库内暂时没有可用数据节点。

* 示例：`ERR-CODE: [PXC-4201][ERR_GROUP_NO_ATOM_AVALILABLE] All weights of DBs in Group '*****' is 0. Weights is: *****.`

* 说明：当分库包含的数据节点全都不可用，或者处于fail-fast状态时，PolarDB-X会提示PXC-4201错误。

  通常该错误是由于数据节点故障导致的。请检查后端数据节点状态以定位和解决问题。如果故障解决后仍然出现PXC-4201错误，请联系技术支持。
  




PXC-4202 ERR_SQL_QUERY_TIMEOUT 
---------------------------------------------------

* 描述：PolarDB-X查询超时。

* 示例：`ERR-CODE: [PXC-4202][ERR_SQL_QUERY_TIMEOUT] Slow query leads to a timeout exception, please contact DBA to check slow sql. SocketTimout:*** ms, Atom:*****, Group:*****, AppName:*****, Env:*****, UnitName:null.`

* 说明：该错误表示SQL语句在后端数据节点上的执行时间超过PolarDB-X设置的socketTimeout参数限制。默认的PolarDB-X超时（socketTimeout）时间设置是900秒。

  建议优化SQL语句，以及在后端数据节点上创建适合的索引以提升SQL语句的执行性能。如果优化后的SQL语句仍然较慢，可以参见如下Hint语法临时设置PolarDB-X的超时时间：`/*TDDL:SOCKET_TIMEOUT=900000*/ SELECT * FROM dual;`，其中SOCKET_TIMEOUT设置的单位是毫秒。关于PolarDB-X的Hint用法，详情请参见[自定义SQL超时时间](https://help.aliyun.com/document_detail/51246.htm) 。如果需要永久调整PolarDB-X超时设置，请联系技术支持。
  




PXC-4203 ERR_SQL_QUERY_MERGE_TIMEOUT 
---------------------------------------------------------

* 描述：PolarDB-X分布式查询超时。

* 示例：`ERR-CODE: [PXC-4203][ERR_SQL_QUERY_MERGE_TIMEOUT] Slow sql query leads to a timeout exception during merging results, please optimize the slow sql. The the default timeout is *** ms. DB is *****`

* 说明：PolarDB-X执行分布式查询超时，默认的超时设置是900秒。产生PXC-4203错误表示SQL语句扫描了多个分库的数据并且执行时间超过900秒。

  建议进行如下优化：
  * 在WHERE条件中添加分库键（Sharding key）条件，将SQL语句优化成单库执行；
  
  * 检查是否可以在后端数据节点上创建适合的索引，提升扫描各个分库数据的性能；
  
  * 设法消除分布式查询中的跨库JOIN，数据重排序等耗时操作，降低PolarDB-X数据合并阶段的消耗。
  
  
  

  如果优化后的SQL语句仍然较慢，可以使用如下Hint语法临时设置PolarDB-X的超时时间：`/*TDDL:SOCKET_TIMEOUT=900000*/ SELECT * FROM dual;`，其中SOCKET_TIMEOUT设置的单位是毫秒。关于PolarDB-X的Hint用法，详情请参见[自定义SQL超时时间。](https://help.aliyun.com/document_detail/51246.htm)
  




PXC-4400 ERR_SEQUENCE 
------------------------------------------

* 描述：处理Sequence（全局唯一序列）失败。

* 示例：`ERR-CODE: [PXC-4400][ERR_SEQUENCE] Sequence : All dataSource faild to get value!`

* 说明：处理Sequence出错，错误信息在`Sequence :`中给出。导致PXC-4400的常见原因是数据节点故障，无法访问Sequence有关的数据表。

  建议先检查后端数据节点状态。如果排除数据节点故障后仍然发生错误，请联系技术支持。
  




PXC-4401 ERR_MISS_SEQUENCE 
-----------------------------------------------

* 描述：Sequence不存在。

* 示例：`ERR-CODE: [PXC-4401][ERR_MISS_SEQUENCE] Sequence '*****' is not found`

* 说明：命令中使用的Sequence名称不存在。建议用`SHOW SEQUENCES`命令检查PolarDB-X中所有已创建的Sequence名称，并且选择正确的Sequence使用。

  如果使用的Sequence尚不存在，可以用如下语句创建：

  ```sql
  CREATE SEQUENCE <sequence name> [ START WITH <numeric value> ] 
  [ INCREMENT BY <numeric value> ] [ MAXVALUE <numeric value> ] 
  [ CYCLE | NOCYCLE ]`
  ```

  

  如果使用的Sequence已经存在，但是仍然提示PXC-4401错误，请联系技术支持。关于PolarDB-X的Sequence用法，详情请参见[PolarDB-X Sequence介绍](https://help.aliyun.com/document_detail/29675.htm) 。
  




PXC-4403 ERR_MISS_SEQUENCE_TABLE_ON_DEFAULT_DB 
-------------------------------------------------------------------

* 描述：Sequence使用的数据表不存在。

* 示例：`ERR-CODE: [PXC-4403][ERR_MISS_SEQUENCE_TABLE_ON_DEFAULT_DB] Sequence table is not in default db.`

* 说明：无法在PolarDB-X后端的数据库里访问名称叫sequence或者sequence_opt的数据表，请联系技术支持。




PXC-4404 ERR_SEQUENCE_TABLE_META 
-----------------------------------------------------

* 描述：Sequence数据表结构错误。

* 示例：`ERR-CODE: [PXC-4404][ERR_SEQUENCE_TABLE_META] the meta of sequence table is error, some columns missed`

* 说明：Sequence相关数据表（如sequence或sequence_opt）中缺少相应的字段，请联系技术支持。




PXC-4405 ERR_INIT_SEQUENCE_FROM_DB 
-------------------------------------------------------

* 描述：初始化Sequence错误。

* 示例：`ERR-CODE: [PXC-4405][ERR_INIT_SEQUENCE_FROM_DB] init sequence manager error: *****`

* 说明：初始化需要访问的Sequence时出错，错误信息在init sequence manager error:后给出。建议先检查PolarDB-X后端数据节点状态。如果排除数据节点故障后仍然提示PXC-4405错误，请联系技术支持。




PXC-4407 ERR_OTHER_WHEN_BUILD_SEQUENCE 
-----------------------------------------------------------

* 描述：访问Sequence数据表出错。

* 示例：`ERR-CODE: [PXC-4407][ERR_OTHER_WHEN_BUILD_SEQUENCE] error when build sequence: *****`

* 说明：在访问Sequence相关数据表（如sequence或sequence_opt）时发生错误。错误信息在error when build sequence:后给出。建议先检查PolarDB-X后端数据节点状态。如果排除数据节点故障后仍然提示PXC-4407错误，请联系技术支持。

  




PXC-4408 ERR_SEQUENCE_NEXT_VALUE 
-----------------------------------------------------

* 描述：获取Sequence值出错。

* 示例：`ERR-CODE: [PXC-4408][ERR_SEQUENCE_NEXT_VALUE] error when get sequence's next value, sequence is: *****, error: *****`

* 说明：使用PolarDB-X自增主键，或者使用`<sequence name>.NEXTVAL`语法手工获取全局唯一ID时发生错误。错误原因在error:提示后给出。

  产生PXC-4408错误的原因是后端数据节点故障。建议先检查PolarDB-X后端数据节点状态和访问压力。如果排除数据节点故障后仍然提示PXC-4408错误，请联系技术支持。
  




PXC-4500 ERR_PARSER 
----------------------------------------

* 描述：解析SQL语句失败。

* 示例：`ERR-CODE: [PXC-4500][ERR_PARSER] not support statement: '*****'`

* 说明：PolarDB-X支持符合SQL-92标准的SQL语法，以及MySQL支持的语法扩展与函数。请检查执行的SQL语句是否符合PolarDB-X兼容的SQL标准及MySQL规范。

  关于SQL标准语法，请参见[SQL标准语法](https://www.w3schools.com/sql/) 。关于SQL兼容性，请参见[SQL兼容性](https://help.aliyun.com/document_detail/71252.htm) 。关于MySQL 5.6 SQL语法，请参见[MySQL 5.6 SQL语法](https://dev.mysql.com/doc/refman/5.6/en/sql-syntax.html) 。如果您的SQL语句符合上述语法规则仍然提示PXC-4500错误，请联系技术支持。
  




PXC-4501 ERR_OPTIMIZER 
-------------------------------------------

* 描述：优化器转换SQL语句失败。

* 示例：`ERR-CODE: [PXC-4501][ERR_OPTIMIZER] optimize error by: Unknown column '*****' in 'order clause'`

* 说明：PolarDB-X优化器的工作是转换SQL语句到内部语法树。如果SQL语句中出现逻辑错误，优化器转换就会失败，产生PXC-4501错误。

  建议按照optimize error by:后的提示检查和调整您的SQL语句。如果调整SQL语句后仍然提示PXC-4501错误，请联系技术支持。
  




PXC-4502 ERR_OPTIMIZER_MISS_ORDER_FUNCTION_IN_SELECT 
-------------------------------------------------------------------------

* 描述：ORDER BY包含的函数列在SELECT子句中不存在。

* 示例：```ERR-CODE: [PXC-4502][ERR_OPTIMIZER_MISS_ORDER_FUNCTION_IN_SELECT] Syntax Error: orderBy/GroupBy Column ***** is not existed in select clause```

* 说明：当SQL语句中的ORDER BY子句包含函数列（例如RAND()）时，PolarDB-X要求同样的函数列必须也在SELECT子句中出现，否则提示PXC-4502错误。

  建议在SELECT子句中添加相应的函数列。
  




PXC-4504 ERR_OPTIMIZER_SELF_CROSS_JOIN 
-----------------------------------------------------------

* 描述：相同表JOIN的条件不足。

* 示例：`ERR-CODE: [PXC-4504][ERR_OPTIMIZER_SELF_CROSS_JOIN] self cross join case, add shard column filter on right table`

* 说明：PolarDB-X在执行相同表的JOIN时，如果WHERE子句只包含其中一张左表（或右表）的拆分字段（sharding column）条件，会提示PXC-4504错误。

  建议调整SQL语句，在WHERE子句中补全JOIN左表（或右表）的拆分字段条件。
  




PXC-4506 ERR_MODIFY_SHARD_COLUMN 
-----------------------------------------------------

* 描述：禁止更新拆分键。

* 示例：`ERR-CODE: [PXC-4506][ERR_MODIFY_SHARD_COLUMN] Column '*****' is a sharding key of table '*****', which is forbidden to be modified.`

* 说明：这是禁止修改拆分键（sharding key）的异常，PolarDB-X目前静止修复包含GSI的表的分区键。

  建议将对应UPDATE语句修改为相同效果的INSERT+DELETE语句。
  




PXC-4508 ERR_OPTIMIZER_NOT_ALLOWED_SORT_MERGE_JOIN 
-----------------------------------------------------------------------

* 描述：无法执行合并排序JOIN。

* 示例：`ERR-CODE: [PXC-4508][ERR_OPTIMIZER_NOT_ALLOWED_SORT_MERGE_JOIN] sort merge join is not allowed when missing equivalent filter`

* 说明：如果SQL语句中需要JOIN的数据表分别来自不同的数据节点，PolarDB-X会优先选择合并排序（Sort-merge Join）算法。该算法要求JOIN的左表与右表必须包含字段相等的关联条件，否则PolarDB-X将提示PXC-4508错误。

  建议调整SQL语句，在JOIN或WHERE部分添加相应的关联条件。
  




PXC-4509 ERR_OPTIMIZER_ERROR_HINT 
------------------------------------------------------

* 描述：Hint语法错误。

* 示例：`ERR-CODE: [PXC-4509][ERR_OPTIMIZER_ERROR_HINT] Hint Syntax Error: unexpected operation: *****.`

* 说明：该错误表示SQL语句中的Hint语法无法被PolarDB-X解析。更多关于Hint语法信息，请参见[自定义HINT简介](hint-faq.md)




PXC-4510 ERR_CONTAINS_NO_SHARDING_KEY 
----------------------------------------------------------

* 描述：缺少拆分键（sharding key）条件。

* 示例：`ERR-CODE: [PXC-4510][ERR_CONTAINS_NO_SHARDING_KEY] Your SQL contains NO SHARDING KEY '*****' for table '*****', which is not allowed in DEFAULT.`

* 说明：禁止全表扫描（full-table scan）功能，PolarDB-X在建表时默认开启全表扫描功能。如果手动关闭全表扫描，建议确认与该表有关的SQL语句都已添加拆分键条件。




PXC-4511 ERR_INSERT_CONTAINS_NO_SHARDING_KEY 
-----------------------------------------------------------------

* 描述：INSERT语句缺少拆分键 (sharding key）。

* 示例：`ERR-CODE: [PXC-4511][ERR_INSERT_CONTAINS_NO_SHARDING_KEY] Your INSERT SQL contains NO SHARDING KEY '*****' for table '*****'.`

* 说明：当INSERT语句的目标是一张PolarDB-X拆分表时，必须在插入数据中包含拆分键的值（拆分键是自增主键例外）。否则，PolarDB-X将提示PXC-4511错误。

  如果遇到该错误，建议修改INSERT语句补充缺少的拆分键值。
  




PXC-4515 ERR_CONNECTION_CHARSET_NOT_MATCH 
--------------------------------------------------------------

* 描述：输入字符集不匹配。

* 示例：`ERR-CODE: [PXC-4515][ERR_CONNECTION_CHARSET_NOT_MATCH] Caused by MySQL's character_set_connection doesn't match your input charset. Partition DDL can only take ASCII or chinese column name. If you want use chinese table or column name, Make sure MySQL connection's charset support chinese character. Use "set names xxx" to set correct charset.`

* 说明：PolarDB-X支持用中文字符命名表名及字段名。在执行含有中文字符的SQL语句时，如果数据库连接的字符集设置（character_set_connection）不支持中文（如 latin1），会提示PXC-4515错误。

  您可以使用`SHOW VARIABLES LIKE 'character_set_connection'`查询MySQL客户端当前的连接字符集，使用`SET NAMES`命令修改当前连接字符集。如果是Java程序JDBC方式连接PolarDB-X，请设置数据库连接参数characterEncoding。
  




PXC-4517 ERR_MODIFY_SYSTEM_TABLE 
-----------------------------------------------------

* 描述：禁止修改系统表。

* 示例：`ERR-CODE: [PXC-4617][ERR_MODIFY_SYSTEM_TABLE] Table '*****' is PolarDB-XSYSTEM TABLE, which is forbidden to be modified.`

* 说明：PolarDB-X内部维护了一些系统表，使用SQL语句更新其中的数据会提示PXC-4517错误。限制的系统表包括sequence、sequence_opt、txc_undo_log等，请避免在业务或数据库设计中使用这些表名。




PXC-4520 ERR_DML_WITH_SUBQUERY 
---------------------------------------------------

* 描述：不支持在DML使用子查询语句。

* 示例：`ERR-CODE: [PXC-4520][ERR_DML_WITH_SUBQUERY] DO NOT support UPDATE/DELETE with subQuery`

* 说明：目前PolarDB-X中禁止在DML语句中包含子查询语句，遇到此类问题建议先从业务上改写SQL，避免使用子查询。




PXC-4521 ERR_INSERT_SHARD 
----------------------------------------------

* 描述：Insert过程中一条记录被路由到了多个分片。

* 示例：`ERR-CODE: [PXC-4521][ERR_INSERT_SHARD] Cannot decide which group to insert`

* 说明：Insert过程中一条记录被路由到了多个分片，导致不知道这条记录被插入到哪个分片，出现这类问题，请联系技术支持。




PXC-4523 ERR_TODNF_LIMIT_EXCEED 
----------------------------------------------------

* 描述：where后面的过滤条件太多。

* 示例：`ERR-CODE: [PXC-4523][ERR_TODNF_LIMIT_EXCEED] toDnf has exceed the limit size`

* 说明：PolarDB-X会对用户SQL的查询条件（即where子句）进行CNF/DNF的条件范式转换，并进行条件推导与优化，为了优化稳定性，限制了条件数2000。用户可以将DNF_REX_NODE_LIMIT参数调整大。




PXC-4524 ERR_TOCNF_LIMIT_EXCEED 
----------------------------------------------------

* 描述：where后面的过滤条件太多。

* 示例：`ERR-CODE: [PXC-4524][ERR_TODNF_LIMIT_EXCEED] toCnf has exceed the limit size`

* 说明：PolarDB-X会对用户SQL的查询条件（即where子句）进行CNF/DNF的条件范式转换，并进行条件推导与优化，为了优化稳定性，限制了条件数2000。用户可以将CNF_REX_NODE_LIMIT参数调整大。




PXC-4526 ERR_FUNCTION_NOT_FOUND 
----------------------------------------------------

* 描述：该函数不支持调用。

* 示例：`ERR-CODE: [PXC-4526][ERR_FUNCTION_NOT_FOUND] No match found for function signature`

* 说明：该错误表明在SQL语句中使用了错误的语法或不支持的函数。建议仔细检查SQL语句中的函数调用部分，使用正确的参数个数和类型调用函数。




PXC-4527 ERR_MODIFY_SHARD_COLUMN_ON_TABLE_WITHOUT_PK 
-------------------------------------------------------------------------

* 描述：不允许在无主键的表上修改分库键。

* 示例：`ERR-CODE: [PXC-4527][ERR_MODIFY_SHARD_COLUMN_ON_TABLE_WITHOUT_PK] `

* 说明：PolarDB-X目前不允许在无主键的表上修改分库键。




PXC-4595 ERR_UNKNOWN_TZ 
--------------------------------------------

* 描述：设置错误的时区。

* 示例：`ERR-CODE: [PXC-4595][ERR_UNKNOWN_TZ] `

* 说明：建议检查时间的语法和格式是否设置正确。




PXC-4600 ERR_FUNCTION 
------------------------------------------

* 描述：错误的函数调用。

* 示例：`ERR-CODE: [PXC-4600][ERR_FUNCTION] function compute error by Incorrect parameter count in the call to native function '*****' `

* 说明：该错误表明在SQL语句中使用了错误的语法或参数调用函数。建议仔细检查SQL语句中的函数调用部分，使用正确的参数个数和类型调用函数。




PXC-4602 ERR_CONVERTOR 
-------------------------------------------

* 描述：错误的类型转换。

* 示例：`ERR-CODE: [PXC-4602][ERR_CONVERTOR] convertor error by Unsupported convert: [*****]`

* 说明：该错误表明PolarDB-X在执行SQL时进行数据类型转换失败。请检查SQL语句中是否存在需要隐式类型转换的数据，并且尽量使用相同类型进行比较和计算。




PXC-4603 ERR_ACCROSS_DB_TRANSACTION 
--------------------------------------------------------

* 描述：错误的类型转换。

* 示例：`ERR-CODE: [PXC-4602][ERR_CONVERTOR] convertor error by Unsupported convert: [*****]`

* 说明：该错误表明PolarDB-X在执行SQL时进行数据类型转换失败。请检查SQL语句中是否存在需要隐式类型转换的数据，并且尽量使用相同类型进行比较和计算。




PXC-4604 ERR_CONCURRENT_TRANSACTION 
--------------------------------------------------------

* 描述：嵌套事务失败。

* 示例：`ERR-CODE: [PXC-4604][ERR_CONCURRENT_TRANSACTION] Concurrent query is not supported on transaction group, transaction group is: {0}.`

* 说明：PolarDB-X不支持嵌套事务，如果在同一个数据库连接里尝试同时开启2个以上事务，将提示PXC-4604错误。

  建议在应用开发时避免使用嵌套事务，或者使用应用层的事务框架防止产生嵌套事务。
  




PXC-4606 ERR_QUERY_CANCLED 
-----------------------------------------------

* 描述：当前执行的SQL被取消。

* 示例：`ERR-CODE: [PXC-4606][ERR_QUERY_CANCLED] Getting connection is not allowed when query has been cancled, group is *****`

* 说明：使用`KILL`取消某条SQL语句的执行时，被取消的SQL语句会返回该错误。如果经常出现这一情况，请排查是否有客户端或程序在执行KILL命令。




PXC-4610 ERR_CONNECTION_CLOSED 
---------------------------------------------------

* 描述：连接已经关闭。

* 示例：`ERR-CODE: [PXC-4610][ERR_CONNECTION_CLOSED] connection has been closed`

* 说明：当事务中的SQL语句执行出错，或者被`KILL`命令取消后，重复使用同一个数据库连接执行其他SQL语句会提示PXC-4610错误。

  建议在该情况下关闭连接，重新获取一个新的数据库连接。
  




PXC-4612 ERR_CHECK_SQL_PRIV 
------------------------------------------------

* 描述：由于权限不够，SQL语句无法执行。

* 示例：`ERR-CODE: [PXC-4612][ERR_CHECK_SQL_PRIV] check user ***** on db ***** sql privileges failed.`

* 说明：PolarDB-X的新版本支持账号和授权，类似MySQL账号权限体系，只有拥有对应类型权限的账号才能执行该SQL语句。如果账号权限不足，PolarDB-X将提示PXC-4612错误。

  建议检查用户拥有的PolarDB-X权限。如果权限不足，请在PolarDB-X控制台设置。关于PolarDB-X账号与权限设置，请参见[PolarDB-X账号和权限系统](permissions.md)
  




PXC-4614 ERR_EXECUTE_ON_MYSQL 
--------------------------------------------------

* 描述：SQL语句在DN上执行报错。

* 示例：

  ```text
  ERR-CODE: [PXC-4614][ERR_EXECUTE_ON_MYSQL] Error occurs when execute on  GROUP '*****': Duplicate entry '*****' for key 'PRIMARY'
  PolarDB-X在后端数据节点数据库上执行SQL语句报错，末尾包含了从数据节点返回的原始错误信息，例如：
  Duplicate entry '*****' for key 'PRIMARY'
  表示写入数据节点数据表发生了主键冲突。
  The table '*****' is full
  表示数据节点使用的临时表已满，需要调整临时表空间或优化SQL语句。
  Deadlock found when trying to get lock;
  表示在数据节点中出现了死锁，通常是数据写入存在较多事务冲突导致的。
  ```

  

* 说明：建议参见PXC-4614提供的原始错误信息排查问题。更多关于SQL语句错误信息请参见[MySQL 5.6文档](https://dev.mysql.com/doc/refman/5.6/en/error-handling.html) 。如果排除应用或数据节点问题后仍然发生PXC-4614错误，请联系技术支持。




PXC-4616 ERR_UNKNOWN_DATABASE 
--------------------------------------------------

* 描述：错误的数据库。

* 示例：`ERR-CODE: [PXC-4616][ERR_UNKNOWN_DATABASE] Unknown database '*****'`

* 说明：PolarDB-X允许在DDL语句中指定数据库名称。如果指定的数据库名称与PolarDB-X提供的数据库名称不一致，将返回PXC-4616错误。

  建议修改DDL语句中的数据库名称，确保与PolarDB-X数据库名称一致。
  




PXC-4620 ERR_FORBID_EXECUTE_DML_ALL 
--------------------------------------------------------

说明：PolarDB-X禁止不带where条件执行delete和update操作。

PXC-4633 ERR_DB_STATUS_READ_ONLY 
-----------------------------------------------------

* 示例：`ERR-CODE: [PXC-4633][ERR_DB_STATUS_READ_ONLY] Database is read-only, only read sql are supported`

* 说明：该错误码表示数据库只有读权限，请检查下当前账号的权限是否受限。




PXC-4707 ERR_OUT_OF_MEMORY 
-----------------------------------------------

* 描述：临时表使用内存超限。

* 示例：`ERR-CODE: [PXC-4707][ERR_OUT_OF_MEMORY]`

* 说明：为了保证数据库稳定性，PolarDB-X限制了每条查询的临时表使用内存量，说明用户当前的查询涉及到的数据比较多，可以打开SPILL或者调大当前查询临时表内存限制。




PXC-4709 ERR_IVENTORY_HINT_NOT_SUPPORT_CROSS_SHARD 
-----------------------------------------------------------------------

* 示例：`ERR-CODE: [PXC-4709][ERR_IVENTORY_HINT_NOT_SUPPORT_CROSS_SHARD]`

* 说明：热点秒杀功能要求是单分片事务，如果涉及到多分片事务，会出现该错误。请调整业务逻辑，确保在单分片事务下使用热点功能。




PXC-4994 ERR_FLOW_CONTROL 
----------------------------------------------

* 描述：流量已达上限。

* 示例：`ERR-CODE: [PXC-4994][ERR_FLOW_CONTROL] [*****] flow control by *****`

* 说明：该错误代表PolarDB-X处理SQL请求已达到内部流量上限，当前请求被拒绝。建议检查SQL请求量是否存在异常峰值。如果观察到SQL请求量下降后，仍然大量提示PXC-4994错误，请联系技术支持。




PXC-4998 ERR_NOT_SUPPORT 
---------------------------------------------

* 描述：不支持的特性。

* 示例：`ERR-CODE: [PXC-4998][ERR_NOT_SUPPORT] ***** not support yet!`

* 说明：该错误代表使用的SQL语法或者功能PolarDB-X尚不支持。如果这些SQL语法或者功能对您十分重要，请联系技术支持。




PXC-5001 ERR_TRANS 
---------------------------------------

* 描述：一般性的事务错误。

* 示例：`ERR-CODE: [PXC-5001][ERR_TRANS] Too many lines updated in statement.`

* 说明：请参见错误信息处理。
  * `Too many lines updated in statement`

    事务中的UPDATE语句更新行数超出限制（1000），建议检查UPDATE语句的WHERE条件。如果需要在事务中执行大批量数据更新，可以使用PolarDB-X`Hint/*TDDL:UNDO_LOG_LIMIT={number}*/`调整限制值。
    
  * `Deferred execution is only supported in Flexible or XA Transaction`
  
    后置执行功能仅仅在柔性事务与XA事务策略下可用。在用PolarDB-X的`Hint/*TDDL:DEFER*/`语句提交后置执行功能之前，请先用`SET drds_transaction_policy = ***`命令更改PolarDB-X事务策略。
    
  * 其他错误信息，请联系技术支持。
  
  
  




PXC-5002 ERR_TRANS_UNSUPPORTED 
---------------------------------------------------

* 描述：事务中的语法或功能尚不支持。

* 示例：`ERR-CODE: [PXC-5002][ERR_TRANS_UNSUPPORTED] Table without primary keys is not supported.`

* 说明：该功能在PolarDB-X事务中尚不支持。如果此功能很重要，请联系技术支持。




PXC-5003 ERR_TRANS_LOG 
-------------------------------------------

* 描述：无法访问事务日志。

* 示例：`ERR-CODE: [PXC-5003][ERR_TRANS_LOG] Failed to update transaction state: *****`

* 说明：为保证分布式事务的原子性，PolarDB-X在事务中会访问后端数据节点上的事务日志。如果PolarDB-X在读写事务日志时出错，将返回PXC-5003错误。

  产生PXC-5003错误的原因通常来自后端的数据节点故障。建议检查PolarDB-X后端数据节点状态和访问压力。如果排除数据节点问题后仍然产生PXC-5003错误，请联系技术支持。
  




PXC-5006 ERR_TRANS_COMMIT 
----------------------------------------------

* 描述：事务提交过程中出错。

* 示例：`ERR-CODE: [PXC-5006][ERR_TRANS_COMMIT] Failed to commit primary group *****: *****, TRANS_ID = *****`

* 说明：PolarDB-X在提交事务分支过程中出错，TRANS_ID对应的事务将被自动回滚。产生PXC-5006错误的原因通常来自后端的数据节点故障。建议检查PolarDB-X后端数据节点状态和访问压力。如果排除数据节点问题后仍然产生PXC-5006错误，请联系技术支持。




PXC-5008 ERR_TRANS_TERMINATED 
--------------------------------------------------

* 描述：事务已被Kill或超时中止。

* 示例：`ERR-CODE: [PXC-5008][ERR_TRANS_TERMINATED] Current transaction was killed or timeout. You may need to set a longer timeout value.`

* 说明：如果PolarDB-X事务在执行中被Kill或者超时（执行时间超出drds_transaction_timeout值），则出现该错误。如果是事务超时导致报错，建议使用`SET drds_transaction_timeout = ***`命令修改PolarDB-X事务的执行时间上限，单位是毫秒。




PXC-5010 ERR_TRANS_CONTINUE_AFTER_WRITE_FAIL 
-----------------------------------------------------------------

* 描述：写入失败后，不允许继续进行事务操作。

* 示例：`ERR-CODE: [PXC-5010][ERR_TRANS_CONTINUE_AFTER_WRITE_FAIL] Cannot continue or commit transaction after writing failed`

* 说明：如果PolarDB-X事务中涉及到分布式事务，执行失败后，不允许继续进行事务，这个时候需要前端主动执行rollback，重试事务操作。




PXC-5108 ERR_CHECK_PRIVILEGE_FAILED_ON_TABLE 
-----------------------------------------------------------------

说明：当前账号没有对当前表的操作权限，请检查权限。

PXC-5119 ERR_FILE_CANNOT_BE_CREATE 
-------------------------------------------------------

说明：PolarDB-X默认关闭对`select outfile`语句的支持，如有需要请联系技术支持。

PXC-5302 ERR_GLOBAL_SECONDARY_INDEX_UNSUPPORTED 
--------------------------------------------------------------------

说明：当前表不支持创建全局二级索引。可能是如下原因导致的：1. 当前表是单表或者广播表；2. 全局二级索引的列不包含分区键。其他情况请联系技术支持。

PXC-5306 ERR_GLOBAL_SECONDARY_INDEX_INSERT_DUPLICATE_VALUES 
--------------------------------------------------------------------------------

说明：在数据写入到全局二级索引表过程中存在主键冲突，请根据报错信息提示的记录值，确认是否存在主键冲突。

PXC-5308 ERR_GLOBAL_SECONDARY_INDEX_MODIFY_UNIQUE_KEY 
--------------------------------------------------------------------------

说明：在执行DML过程中，全局二级索引表存在唯一键冲突，请根据报错信息提示的记录值，确认是否存在唯一键冲突。

PXC-5310 ERR_GLOBAL_SECONDARY_INDEX_ONLY_SUPPORT_XA 
------------------------------------------------------------------------

说明：PolarDB-X只有在XA/TSO分布式事务下，才支持全局二级索引，如果产生此报错，可能业务之前调整过默认事务策略，请调整回XA/TSO分布式事务，再创建全局二级索引。

PXC-5313 ERR_GLOBAL_SECONDARY_INDEX_MODIFY_GSI_TABLE_WITH_DDL 
----------------------------------------------------------------------------------

说明：PolarDB-X默认不支持对全局二级索引表做DDL操作，如有需要请联系技术支持。

PXC-5316 ERR_GLOBAL_SECONDARY_INDEX_INDEX_AND_SHARDING_COLUMNS_NOT_MATCH 
---------------------------------------------------------------------------------------------

说明：PolarDB-X在创建全局二级索引时，要求索引表字段必须包含主表分区键，因为在回表过程中需要根据分区键定位到具体的分片和具体记录。如果产生此报错，请检查DDL语句中索引表字段是否包含了分区键。

PXC-5317 ERR_GLOBAL_SECONDARY_INDEX_CONTINUE_AFTER_WRITE_FAIL 
----------------------------------------------------------------------------------

* 示例：`ERR-CODE: [PXC-5317][ERR_GLOBAL_SECONDARY_INDEX_CONTINUE_AFTER_WRITE_FAIL] Cannot continue or commit transaction after writing global secondary index failed`

* 说明：在包含GSI的表上执行DML语句时，如果产生此报错，则不允许继续提交包含该失败DML语句的事务。需要修改业务代码，DML报错后需要回滚事务之后重试。




PXC-5321 ERR_GLOBAL_SECONDARY_INDEX_BACKFILL_DUPLICATE_ENTRY 
---------------------------------------------------------------------------------

说明：在创建全局二级索引过程中，数据回填出现了索引表主键冲突，请确认索引表的主键是否存在相同值。

PXC-8007 ERR_ABANDONED_TASK 
------------------------------------------------

说明：查询过慢或者由于不明原因卡住超过2小时，数据库系统会终止查询，出现该异常。请先优化查询，确保不会出现超慢查询，若无法解决，请联系技术支持。

PXC-8008 ERR_EXECUTE_SPILL 
-----------------------------------------------

说明：查询过程中，数据过大会触发临时表落盘，这个错误是在数据落盘过程中出现的异常，请联系技术支持。

PXC-8011 ERR_OUT_OF_SPILL_SPACE 
----------------------------------------------------

说明：查询过程中，数据过大会触发临时表落盘，如果临时表生产的文件过多，超过了系统允许的可落盘的最大磁盘空间就会出现此错误。请先优化查询，减少计算过程中对临时表的依赖。若无法解决请联系技术支持。

PXC-8012 ERR_OUT_OF_SPILL_FD 
-------------------------------------------------

说明：查询过程中，数据过大会触发临时表落盘，如果临时表生产的文件过多，超过了系统允许的文件句柄个数就会出现此错误。请联系技术支持，确保是否存在文件句柄泄漏，若没有泄漏，可以适当调大文件句柄个数限制。

PXC-8102 ERR_PAGE_TOO_LARGE 
------------------------------------------------

说明：在MPP并计算过程中，数据是按批在多个计算节点网络交互，如果一个批的数据过大超过RPC限制最大值，就会出现此错误，可以尝试调小默认CHUNK_SIZE值。

PXC-8103 ERR_NO_NODES_AVAILABLE 
----------------------------------------------------

说明：在MPP并行计算过程中可能有计算节点出现故障，导致执行调度之初没有计算节点可调度，请确认计算节点服务是否正常，如果计算节点服务都正常仍然出现报此类错误，请联系技术支持。

PXC-9301 ERR_DUPLICATED_PARTITION_NAME 
-----------------------------------------------------------

说明：在执行分区表相关的DDL操作过程中，使用了相同的分区表名。

PXC-9305 ERR_PARTITION_NAME_NOT_EXISTS 
-----------------------------------------------------------

说明：在执行分区表相关的DDL操作过程中，提示分区表名不存在。请检查下分表名是否拼写正常，使用`show create table`和 `check table`检查表元数据是否一致，如果不一致，可能是元数据维护信息不一致导致。请联系技术支持。

PXC-10004 ERR_X_PROTOCOL_RESULT 
----------------------------------------------------

* 示例：`ERR-CODE: [PXC-10004][ERR_X_PROTOCOL_RESULT] Should use chunk2chunk to fetch data`

* 说明：PolarDB-X计算节点和数据节点采用的是私有RPC通信，出现这类异常主要是计算节点和数据节点建立的连接异常，具体原因有很多种，需要查看异常信息，如果无法定位具体问题，请联系技术支持。



