SHOW HELP 
==============================

本文介绍了如何使用SHOW HELP命令。

SHOW HELP用于展示PolarDB-X所有辅助SQL指令及其说明。

```sql
mysql> show help;
+-----------------------------------------+---------------------------------------------------------+---------------------------------------------+
| STATEMENT                               | DESCRIPTION                                             | EXAMPLE                                     |
+-----------------------------------------+---------------------------------------------------------+---------------------------------------------+
| show rule                               | Report all table rule                                   |                                             |
| show rule from TABLE                    | Report table rule                                       | show rule from user                         |
| show full rule from TABLE               | Report table full rule                                  | show full rule from user                    |
| show topology from TABLE                | Report table physical topology                          | show topology from user                     |
| show partitions from TABLE              | Report table dbPartition or tbPartition columns         | show partitions from user                   |
| show broadcasts                         | Report all broadcast tables                             |                                             |
| show datasources                        | Report all partition db threadPool info                 |                                             |
| show node                               | Report master/slave read status                         |                                             |
| show slow                               | Report top 100 slow sql                                 |                                             |
| show physical_slow                      | Report top 100 physical slow sql                        |                                             |
| clear slow                              | Clear slow data                                         |                                             |
| trace SQL                               | Start trace sql, use show trace to print profiling data | trace select count(*) from user; show trace |
| show trace                              | Report sql execute profiling info                       |                                             |
| explain SQL                             | Report sql plan info                                    | explain select count(*) from user           |
| explain detail SQL                      | Report sql detail plan info                             | explain detail select count(*) from user    |
| explain execute SQL                     | Report sql on physical db plan info                     | explain execute select count(*) from user   |
| show sequences                          | Report all sequences status                             |                                             |
| create sequence NAME [start with COUNT] | Create sequence                                         | create sequence test start with 0           |
| alter sequence NAME [start with COUNT]  | Alter sequence                                          | alter sequence test start with 100000       |
| drop sequence NAME                      | Drop sequence                                           | drop sequence test                          |
| show db status                          | Report size of each physical database                   | show db status                              |
| show full db status                     | Report size of each physical table, support like        | show full db status like user               |
| show ds                                 | Report simple partition db info                         | show ds                                     |
| show connection                         | Report all connections info                             | show connection                             |
| show trans                              | Report trans info                                       | show trans/show trans limit 2               |
| show full trans                         | Report all trans info                                   | show full trans/show full trans limit 2     |
| show stats                              | Report all requst stats                                 | show stats                                  |
| show stc                                | Report all requst stats by partition                    | show stc                                    |
| show htc                                | Report the CPU/LOAD/MEM/NET/GC stats                    | show htc                                    |
| show ccl_rules                          | Report the ccl rule info                                | show ccl_rules                              |
| clear ccl_rules                         | Clear the ccl rules                                     | clear ccl_rules                             |
| show ccl_triggers                       | Report the ccl trigger info                             | show ccl_triggers                           |
| clear ccl_triggers                      | Clear the ccl triggers                                  | clear ccl_triggers                          |
+-----------------------------------------+---------------------------------------------------------+---------------------------------------------+
```


