如何异步删除大表 
=============================

本文介绍了异步删除大表的方法。

背景信息 
-------------------------

PolarDB-X底层存储节点，默认使用InnoDB引擎时，直接删除大表后会触发表文件的删除，导致POSIX文件系统出现严重的稳定性问题，因此InnoDB会启动一个后台线程来异步清理数据文件。当删除单个表空间时，会将对应的数据文件先重命名为临时文件，然后清除线程将异步、缓慢地清理文件。

**说明** 提供清除文件日志来保证DDL语句的原子性。

操作步骤 
-------------------------

1. 使用如下命令查看实例全局变量设置：

   ```sql
   SHOW GLOBAL VARIABLES LIKE '%data_file_purge%';
   ```

   

   返回结果如下：

   ```sql
   +----------------------------------------+-------+
   | Variable_name                          | Value |
   +----------------------------------------+-------+
   | innodb_data_file_purge                 | ON    |
   | innodb_data_file_purge_all_at_shutdown | OFF   |
   | innodb_data_file_purge_dir             |       |
   | innodb_data_file_purge_immediate       | OFF   |
   | innodb_data_file_purge_interval        | 100   |
   | innodb_data_file_purge_max_size        | 128   |
   | innodb_print_data_file_purge_process   | OFF   |
   +----------------------------------------+-------+
   ```

   

   参数说明如下：
   

   |                   参数                   |          说明           |
   |----------------------------------------|-----------------------|
   | innodb_data_file_purge                 | 是否启用异步清除策略。           |
   | innodb_data_file_purge_all_at_shutdown | 正常关机时全部清理。            |
   | innodb_data_file_purge_dir             | 临时文件目录。               |
   | innodb_data_file_purge_immediate       | 取消数据文件的链接但不清理。        |
   | innodb_data_file_purge_interval        | 清理时间间隔。单位：ms。         |
   | innodb_data_file_purge_max_size        | 每次清理单个文件大小的最大值。单位：MB。 |
   | innodb_print_data_file_purge_process   | 是否打印文件清理工作进程。         |

   

   可以使用如下命令设置参数：

   ```sql
   set global INNODB_DATA_FILE_PURGE = on;
   set global INNODB_DATA_FILE_PURGE_INTERVAL = 100;
   set global INNODB_DATA_FILE_PURGE_MAX_SIZE = 128;
   ```


2. 使用如下命令查看清理进度:

   ```sql
   select * from information_schema.innodb_purge_files;
   ```

   

   返回结果如下：

   ```sql
   +--------+---------------------+--------------------+---------------+-------------------------+--------------+
   | log_id | start_time          | original_path      | original_size | temporary_path          | current_size |
   +--------+---------------------+--------------------+---------------+-------------------------+--------------+
   |      0 | 2021-05-14 14:40:01 | ./file_purge/t.ibd |     146800640 | ./#FP_210514 14:40:01_9 |     79691776 |
   +--------+---------------------+--------------------+---------------+-------------------------+--------------+
   ```

   

   参数说明如下：
   

   |       参数       |          说明           |
   |----------------|-----------------------|
   | start_time     | 清理操作的开始时间。            |
   | original_path  | 表数据文件的原始路径。           |
   | original_size  | 表数据文件的原始大小，单位：byte。   |
   | temporary_path | 清理中的临时文件路径。           |
   | current_size   | 待清理的剩余临时文件大小，单位：byte。 |

   



