如何优化数据导入导出 
===============================

数据库实际应用场景中经常需要进行数据导入导出，本文将介绍如何使用数据导入导出工具。

测试环境 
-------------------------

本文档的测试环境要求如下表：


|     环境      |                       参数                        |
|-------------|-------------------------------------------------|
| PolarDB-X版本 | polarx-kernel_5.4.11-16282307_xcluster-20210805 |
| 节点规格        | 16核64GB                                         |
| 节点个数        | 4个                                              |



测试用表如下：

```sql
CREATE TABLE `sbtest1` (
    `id` int(11) NOT NULL,
    `k` int(11) NOT NULL DEFAULT '0',
    `c` char(120) NOT NULL DEFAULT '',
    `pad` char(60) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`),
    KEY `k_1` (`k`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 dbpartition by hash(`id`);
```



导入导出工具介绍 
-----------------------------

PolarDB-X常见的数据导出方法有：

* mysql -e命令行导出数据

* musqldump工具导出数据

* select into outfile语句导出数据（默认关闭）

* Batch Tool工具导出数据（PolarDB-X配套的导入导出工具）




PolarDB-X常见的数据导入方法有：

* source语句导入数据

* mysql命令导入数据

* 程序导入数据

* load data语句导入数据

* Batch Tool工具导入数据（PolarDB-X配套的导入导出工具）




MySQL原生命令使用示例 
----------------------------------

mysql -e命令可以连接本地或远程服务器，通过执行sql语句，例如select方式获取数据，原始输出数据以制表符方式分隔，可通过字符串处理改成`','`分隔，以csv文件方式存储，方法示例：

```shell
mysql -h ip  -P port -u usr -pPassword db_name -N -e "SELECT id,k,c,pad FROM sbtest1;" >/home/data_1000w.txt
## 原始数据以制表符分隔，数据格式：188092293    27267211    59775766593-64673028018-...-09474402685    01705051424-...-54211554755
mysql -h ip  -P port -u usr -pPassword db_name -N -e "SELECT id,k,c,pad FROM sbtest1;" | sed 's/\t/,/g' >/home/data_1000w.csv
## csv文件以逗号分隔，数据格式：188092293,27267211,59775766593-64673028018-...-09474402685,01705051424-...-54211554755
```



原始数据格式适合load data语句导入数据，使用方法可参考：[LOAD DATA 语句](https://dev.mysql.com/doc/refman/5.7/en/load-data.html) ，示例如下：

```sql
LOAD DATA LOCAL INFILE '/home/data_1000w.txt' INTO TABLE sbtest1;
## LOCAL代表从本地文件导入，local_infile参数必须开启
```



csv文件数据适合程序导入，具体方式可查看[使用程序进行数据导入](../../maintance/topics/import-by-coding.md)。

mysqldump工具使用示例 
------------------------------------

mysqldump工具可以连接到本地或远程服务器，详细使用方法请参见[使用mysqldump导入导出数据](../../maintance/topics/mysqldump.md)。

* 导出数据示例：

  ```shell
  mysqldump -h ip  -P port -u usr -pPassword --default-character-set=utf8mb4 --net_buffer_length=10240 --no-tablespaces --no-create-db --no-create-info --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset  --hex-blob db_name [table_name] > /home/dump_1000w.sql
                      
  ```

  

  mysqldump导出数据可能会出现的问题及解决方法，这两个问题通常是mysql client和mysql server版本不一致导致的。
  1. 问题：mysqldump: Couldn't execute 'SHOW VARIABLES LIKE 'gtid\\_mode''解决方法：添加"--set-gtid-purged=OFF"参数关闭gtid_mode。

     
  
  2. 问题：mysqldump: Couldn't execute 'SHOW VARIABLES LIKE 'ndbinfo\\_version''解决方法：查看mysqldump --version和mysql版本是否一致，使用和mysql版本一致的mysql client。

     
  

  

  导出的数据格式是SQL语句方式，以Batch Insert语句为主体，包含多条SQL语句，``INSERT INTO `sbtest1` VALUES (...),(...)``，"net_buffer_length"参数将影响batch size大小。
  

* SQL语句格式合适的导入数据方式：

  ```shell
  方法一：souce语句导入数据
  source /home/dump_1000w.sql
  方法二：mysql命令导入数据
  mysql -h ip  -P port -u usr -pPassword --default-character-set=utf8mb4 db_name < /home/dump_1000w.sql
  ```

  




Batch Tool工具使用示例 
-------------------------------------

Batch Tool是阿里云内部开发的数据导入导出工具，支持多线程操作。

* 导出数据：

  ```shell
  ## 导出"默认值=分片数"个文件
  java -jar batch-tool.jar -h ip  -P port -u usr -pPassword -D db_name -o export -t sbtest1 -s ,
  ## 导出整合成一个文件
  java -jar batch-tool.jar -h ip  -P port -u usr -pPassword -D db_name -o export -t sbtest1 -s , -F 1
  ```

  

* 导入数据：

  ```shell
  ## 导入32个文件
  java -jar batch-tool.jar -hpxc-spryb387va1ypn.polarx.singapore.rds.aliyuncs.com  -P3306 -uroot -pPassw0rd -D sysbench_db -o import -t sbtest1 -s , -f "sbtest1_0;sbtest1_1;sbtest1_2;sbtest1_3;sbtest1_4;sbtest1_5;sbtest1_6;sbtest1_7;sbtest1_8;sbtest1_9;sbtest1_10;sbtest1_11;sbtest1_12;sbtest1_13;sbtest1_14;sbtest1_15;sbtest1_16;sbtest1_17;sbtest1_18;sbtest1_19;sbtest1_20;sbtest1_21;sbtest1_22;sbtest1_23;sbtest1_24;sbtest1_25;sbtest1_26;sbtest1_27;sbtest1_28;sbtest1_29;sbtest1_30;sbtest1_31" -np -pro 64 -con 32
  ## 导入1个文件
  java -jar batch-tool.jar -h ip  -P port -u usr -p password -D db_name -o import -t sbtest1 -s , -f "sbtest1_0" -np
  ```

  




导出方法对比 
---------------------------

测试方法以PolarDB-X导出1000w行数据为例，数据量大概2GB左右。


|                    方式                    |  数据格式   |  文件大小  |   耗时    | 性能（行/每秒） | 性能（MB/S） |
|------------------------------------------|---------|--------|---------|----------|----------|
| **mysql -e命令 导出原始数据**                    | 原始数据格式  | 1998MB | 33.417s | 299248   | 59.8     |
| **mysql -e命令导出csv格式**                    | csv格式   | 1998MB | 34.126s | 293031   | 58.5     |
| **mysqldump工具（net-buffer-length=10KB）**  | sql语句格式 | 2064MB | 30.223s | 330873   | 68.3     |
| **mysqldump工具（net-buffer-length=200KB）** | sql语句格式 | 2059MB | 32.783s | 305036   | 62.8     |
| **batch tool工具文件数=32（分片数）**              | csv格式   | 1998MB | 4.715s  | 2120890  | 423.7    |
| **batch tool工具文件数=1**                    | csv格式   | 1998MB | 5.568s  | 1795977  | 358.8    |



总结：

1. mysql -e命令和mysqldump工具原理上主要是单线程操作，性能差别并不明显。

2. Batch Tool工具采用多线程方式导出，并发度可设置，能够极大提高导出性能。




导入方法对比 
---------------------------

测试方法以PolarDB-X导入1000w行数据为例，源数据是上一个测试中导出的数据，数据量大概2GB左右。


|                   方式                   |  数据格式   |   耗时    | 性能（行/每秒） | 性能（MB/S） |
|----------------------------------------|---------|---------|----------|----------|
| **source语句（net-buffer-length=10KB）**   | sql语句格式 | 10m24s  | 16025    | 3.2      |
| **source语句（net-buffer-length=200KB）**  | sql语句格式 | 5m37s   | 29673    | 5.9      |
| **mysql命令导入（net-buffer-length=10KB）**  | sql语句格式 | 10m27s  | 15948    | 3.2      |
| **mysql命令导入（net-buffer-length=200KB）** | sql语句格式 | 5m38s   | 29585    | 5.9      |
| **load data语句导入**                      | 原始数据格式  | 4m0s    | 41666    | 8.3      |
| **程序导入batch-1000thread-1**             | csv格式   | 5m40s   | 29411    | 5.9      |
| **程序导入batch-1000thread-32**            | csv格式   | 19s     | 526315   | 105.3    |
| **batch tool工具文件数=32（分片数）**            | csv格式   | 19.836s | 504133   | 100.8    |
| **batch tool工具文件数=1**                  | csv格式   | 10.806s | 925411   | 185.1    |



总结：

1. source语句和mysql命令导入方式，都是单线程执行SQL语句导入，实际是Batch Insert语句的运用，Batch size大小会影响导入性能。Batch size和mysqldump导出数据时的"net-buffer-length"参数有关。建议优化点如下：
   * 推荐将"net-buffer-length"参数设置大，不超过256K，以增大batch size大小，来提高插入性能。
   
   * 使用第三方工具，例如mysqldump，进行mydumper（备份）和myloader（导入）等，可多线程操作。
   

   

2. load data语句是单线程操作，性能优于mysql命令和source语句。

3. 程序导入灵活性较好，可自行设置合适的batch size和并发度，可以达到较好性能。推荐batch大小为1000，并发度为16\~32。

4. Batch Tool工具支持多线程导入，且贴合分布式多分片的操作方式，性能优异。




总结 
-----------------------

1. PolarDB-X兼容MySQL运维上常用的数据导入导出方法，但这些方法大多为MySQL单机模式设计，只支持单线程操作，性能上无法充分利用所有分布式资源。

2. PolarDB-X提供Batch Tool工具，非常贴合分布式场景，在多线程操作下，能够达到极快的数据导入导出性能。



