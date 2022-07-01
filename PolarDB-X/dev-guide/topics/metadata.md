# 元数据库和数据字典 

云原生分布式数据库PolarDB-X的元数据库为information_schema库，兼容MySQL的元数据库。查询元数据库可以直接在JDBC连接中使用SQL语句进行查询。


|          Information_schema视图           |             兼容情况              |
|-----------------------------------------|-------------------------------|
| *SCHEMATA*                              | 兼容                            |
| *TABLES*                                | 兼容                            |
| *COLUMNS*                               | 兼容                            |
| *STATISTICS*                            | 兼容                            |
| *COLUMN_STATISTICS*                     | 直方图信息为PolarDB-X格式             |
| *ENGINES*                               | 兼容                            |
| *KEYWORDS*                              | 兼容                            |
| *COLLATIONS*                            | 兼容                            |
| *TABLE_CONSTRAINTS*                     | 兼容                            |
| *PROCESSLIST*                           | 兼容                            |
| *SESSION_VARIABLES*                     | 兼容                            |
| *GLOBAL_VARIABLES*                      | 兼容                            |
| *INNODB_LOCKS*                          | 兼容                            |
| *INNODB_TRX*                            | 兼容                            |
| *INNODB_BUFFER_PAGE*                    | 兼容                            |
| *INNODB_BUFFER_POOL_STATS*              | 兼容                            |
| *INNODB_BUFFER_PAGE_LRU*                | 兼容                            |
| *INNODB_LOCK_WAITS*                     | 兼容                            |
| *USER_PRIVILEGES*                       | 兼容                            |
| *PARTITIONS*                            | 仅支持PolarDB-X分区表               |
| *WORKLOAD*                              | PolarDB-X HTAP负载信息            |
| *GLOBAL_INDEXES*                        | PolarDB-X全局二级索引               |
| *METADATA_LOCK*                         | PolarDB-X MDL锁信息              |
| *TABLE_GROUP*                           | PolarDB-X表组信息                 |
| *TABLE_DETAIL*                          | PolarDB-X分区表各分区存储信息           |
| *LOCALITY_INFO*                         | PolarDB-X Locality信息          |
| *PHYSICAL_PROCESSLIST*                  | PolarDB-X CN到DN的Processlist信息 |
| *PLAN_CACHE*                            | PolarDB-X计划缓存信息               |
| *STATISTIC_TASK*                        | PolarDB-X统计信息任务信息             |
| *CCL_RULE*                              | PolarDB-X CCL规则信息             |
| *CCL_TRIGGER*                           | PolarDB-X CCL触发信息             |
| *SCHEMA_PRIVILEGES*                     | 未兼容                           |
| *TABLE_PRIVILEGES*                      | 未兼容                           |
| *INNODB_TEMP_TABLE_INFO*                | 未兼容                           |
| *INNODB_SYS_INDEXES*                    | 未兼容                           |
| *INNODB_SYS_FIELDS*                     | 未兼容                           |
| *INNODB_CMP_PER_INDEX_RESET*            | 未兼容                           |
| *INNODB_FT_DEFAULT_STOPWORD*            | 未兼容                           |
| *INNODB_FT_INDEX_TABLE*                 | 未兼容                           |
| *INNODB_FT_INDEX_CACHE*                 | 未兼容                           |
| *INNODB_SYS_TABLESPACES*                | 未兼容                           |
| *INNODB_METRICS*                        | 未兼容                           |
| *INNODB_SYS_FOREIGN_COLS*               | 未兼容                           |
| *INNODB_CMPMEM*                         | 未兼容                           |
| *INNODB_SYS_COLUMNS*                    | 未兼容                           |
| *INNODB_SYS_FOREIGN*                    | 未兼容                           |
| *INNODB_SYS_TABLESTATS*                 | 未兼容                           |
| *INNODB_FT_CONFIG*                      | 未兼容                           |
| *INNODB_SYS_VIRTUAL*                    | 未兼容                           |
| *INNODB_CMP*                            | 未兼容                           |
| *INNODB_FT_BEING_DELETED*               | 未兼容                           |
| *INNODB_CMP_PER_INDEX*                  | 未兼容                           |
| *INNODB_CMPMEM_RESET*                   | 未兼容                           |
| *INNODB_CMP_RESET*                      | 未兼容                           |
| *INNODB_FT_DELETED*                     | 未兼容                           |
| *INNODB_SYS_TABLES*                     | 未兼容                           |
| *INNODB_SYS_DATAFILES*                  | 未兼容                           |
| *PROFILING*                             | 未兼容                           |
| *REFERENTIAL_CONSTRAINTS*               | 未兼容                           |
| *SESSION_STATUS*                        | 未兼容                           |
| *TABLESPACES*                           | 未兼容                           |
| *EVENTS*                                | 未兼容                           |
| *TRIGGERS*                              | 未兼容                           |
| *ROUTINES*                              | 未兼容                           |
| *COLUMN_PRIVILEGES*                     | 未兼容                           |
| *FILES*                                 | 未兼容                           |
| *KEY_COLUMN_USAGE*                      | 未兼容                           |
| *OPTIMIZER_TRACE*                       | 未兼容                           |
| *PARAMETERS*                            | 未兼容                           |
| *CHARACTER_SETS*                        | 未兼容                           |
| *COLLATION_CHARACTER_SET_APPLICABILITY* | 未兼容                           |
| *PLUGINS*                               | 未兼容                           |



## SCHEMATA 

SCHEMATA表提供了关于数据库的信息。包含如下列：

* CATALOG_NAME：库所属的catalog名，值固定为def.

  

* SCHEMA_NAME：库名字

  

* DEFAULT_CHARACTER_SET_NAME：库默认character set名字

  

* DEFAULT_COLLATION_NAME：库默认collation名字

  

* SQL_PATH：值固定为NULL

  




## TABLES 

TABLES表提供关于数据库表的信息。包含如下列：

* TABLE_CATALOG：表所属的catalog名，值固定为def.

  

* TABLE_SCHEMA：表所属的库名

  

* TABLE_NAME：表名

  

* TABLE_TYPEBASE：表类型，普通表为TABLE，视图为VIEW，INFORMATION_SCHEMA表为SYSTEM VIEW

  

* ENGINE：数据库存储引擎

  

* VERSION：版本

  

* ROW_FORMAT：行格式

  

* TABLE_ROWS：表行数估算值

  

* AVG_ROW_LENGTH：平均行长度

  

* DATA_LENGTH：主表空间估算值

  

* MAX_DATA_LENGTH：最大表空间值，默认为NULL

  

* INDEX_LENGTH：表索引空间估算值

  

* DATA_FREE：表空间空闲值

  

* AUTO_INCREMENT：下一个AUTO_INCREMENT值

  

* CREATE_TIME：表创建时间

  

* UPDATE_TIME：表更新时间

  

* CHECK_TIME：表校验时间，值固定为NULL

  

* TABLE_COLLATION：表默认collation值

  

* CHECKSUM：表CHECKSUM

  

* CREATE_OPTIONS：建表语句可选项

  

* TABLE_COMMENT：表注释

  




## COLUMNS 

COLUMNS表提供关于数据库列的信息。包含如下列：

* TABLE_CATALOG：列所属表所属的catalog名，值固定为def.

  

* TABLE_SCHEMA：列所属表所属的库名

  

* TABLE_NAME：列所属表名

  

* COLUMN_NAME：列名

  

* ORDINAL_POSITION：列在表中的顺序

  

* COLUMN_DEFAULT：列默认值

  

* IS_NULLABLE：列是否可以为Null

  

* DATA_TYPE：列类型（无精度）

  

* CHARACTER_MAXIMUM_LENGTH：列最大长度（单位字符）

  

* CHARACTER_OCTET_LENGTH：列最大长度（单位字节）

  

* NUMERIC_PRECISION：列数字精度

  

* NUMERIC_SCALE：列数字scale

  

* DATETIME_PRECISION：列datetime精度

  

* CHARACTER_SET_NAME：列character set名

  

* COLLATION_NAME：列collation名

  

* COLUMN_TYPE：列类型（包含精度）

  

* COLUMN_KEY：列索引信息

  

* EXTRA：列额外信息

  

* PRIVILEGES：列权限信息

  

* COLUMN_COMMENT：列注释

  

* GENERATION_EXPRESSION：生成列

  




## STATISTICS 

STATISTICS表提供关于数据库索引的信息。包含如下列：

* TABLE_CATALOG：索引所属表所属的catalog名，值固定为def.

  

* TABLE_SCHEMA：索引所属表所属的schema名

  

* TABLE_NAME：索引所属表所属的表名.

  

* NON_UNIQUE：索引是否唯一

  

* INDEX_SCHEMA：索引所属表所属的schema名

  

* INDEX_NAME：索引名

  

* SEQ_IN_INDEX：列所属索引序号

  

* COLUMN_NAME：列名

  

* COLLATION：列名collation信息

  

* CARDINALITY：列Cardinality值

  

* SUB_PART：索引前缀

  

* PACKED：索引PACKED信息

  

* NULLABLE：列是否可以为NULL

  

* INDEX_TYPE：索引类型

  

* COMMENT：索引注释（非列维度）

  

* INDEX_COMMENT：索引注释

  




有关Information_schema的更多信息，请参见[MySQL 官网](https://dev.mysql.com/doc/refman/5.7/en/information-schema.html) 。
