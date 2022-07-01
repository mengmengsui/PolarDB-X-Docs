KILL 
=========================

您可以使用KILL指令终止一个正在PolarDB-X上执行的SQL。

语法 
-----------------------

KILL语法支持以下几种用法。

* 您可以通过如下语句终止此连接正在执行的逻辑SQL与物理SQL，并断开该连接。

  ```sql
  KILL PROCESS_ID
  ```

  
  **说明**
  * 您可以通过`SHOW [FULL] PROCESSLIST`语句查看`PROCESS_ID`。
  
  * PolarDB-X不支持`KILL QUERY`语句。
  

  
  

* 您可以通过如下语句终止一个特定的物理SQL：

  ```sql
  KILL 'PHYSICAL_PROCESS_ID'
  ```

  

  示例

  ```sql
  mysql> KILL '0-0-521570'; 
  Query OK, 0 rows affected (0.01 sec)
  ```

  
  **说明**
  * 您可以通过`SHOW PHYSICAL_PROCESS_ID`语句查看`PHYSICAL_PROCESS_ID`。
  
  * 由于`PHYSICAL_PROCESS_ID`列为一个字符串，并非一个数字，因此在该语句中`PHYSICAL_PROCESS_ID`需要使用英文单引号（''）括起来。
  

  
  

* 您可以使用如下语句终止PolarDB-X上所有正在执行的物理SQL：

  ```sql
  KILL 'ALL' 
  ```

  



