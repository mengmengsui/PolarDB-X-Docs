SHOW PROCESSLIST 
=====================================

本文介绍如何使用SHOW PROCESSLIST和SHOW PHYSICAL_PROCESSLIST语句。

SHOW PROCESSLIST 
-------------------------------------

您可以使用如下语句查看PolarDB-X中的连接与正在执行的SQL等信息：

* 语法

  ```sql
  SHOW PROCESSLIST
  ```

  

* 示例

  ```sql
  mysql> SHOW PROCESSLIST\G
       ID: 1971050
     USER: admin
     HOST: 111.111.111.111:4303
       DB: drds_test
  COMMAND: Query
     TIME: 0
    STATE: 
     INFO: show processlist
  1 row in set (0.01 sec)
  ```

  

  |   参数    |                                                                                                                说明                                                                                                                |
  |---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | ID      | 本次连接的ID，为一个Long型数字。                                                                                                                                                                                                              |
  | USER    | 建立此连接所使用的用户名。                                                                                                                                                                                                                    |
  | HOST    | 建立此连接的机器的IP与端口。                                                                                                                                                                                                                  |
  | DB      | 此连接所访问的数据库名称。                                                                                                                                                                                                                    |
  | COMMAND | 目前有如下两种取值： * Query：当前连接正在执行SQL语句。  * Sleep：当前连接正处于空闲状态。                                                                                     |
  | TIME    | 连接处于当前状态持续的时间。 * 当COMMAND为Query时，代表此连接上正在执行的SQL已经执行的时间。  * 当COMMAND为Sleep时，代表此连接空闲的时间。                                                      |
  | STATE   | 目前无意义，恒为空值。                                                                                                                                                                                                                      |
  | INFO    | * 当COMMAND为Query时，为此连接上正在执行的SQL的内容。 **说明** 当不带FULL参数时，最多返回正在执行的SQL的前 30 个字符。当带FULL参数时，最多返回正在执行的SQL的前1000个字符。   * 当COMMAND为Sleep时，为空值，无意义。   |

  




SHOW PHYSICAL_PROCESSLIST 
----------------------------------------------

您可以使用如下指令查看所有正在执行的物理SQL信息：

* 语法

  ```sql
  SHOW PHYSICAL_PROCESSLIST
  ```

  
  **说明** 当SQL比较长的时候，使用`SHOW PHYSICAL_PROCESSLIST`语句返回得到的SQL会被截断，这时可以使用`SHOW FULL PHYSICAL_PROCESSLIST`语句获取完整SQL。
  

* 示例

  ```sql
  mysql> SHOW PHYSICAL_PROCESSLIST\G
  *************************** 1. row ***************************
             ID: 0-0-521414
           USER: tddl5
             DB: tddl5_00
        COMMAND: Query
           TIME: 0
          STATE: init
           INFO: show processlist
  *************************** 2. row ***************************
             ID: 0-0-521570
           USER: tddl5
             DB: tddl5_00
        COMMAND: Query
           TIME: 0
          STATE: User sleep
           INFO: /*DRDS /88.88.88.88/b67a0e4d8800000/ */ select sleep(1000)
  2 rows in set (0.01 sec)
  ```

  
  **说明**
  * 返回结果中每一列的含义与MySQL的`SHOW PROCESSLIST` 指令等价，详情请参见 [SHOW PROCESSLIST Syntax](https://dev.mysql.com/doc/refman/5.5/en/show-processlist.html) 。
  
  * 但与MySQL不同，PolarDB-X返回的物理连接的ID列为一个字符串，并非一个数字。
  

  
  



