使用Batch Tool工具导入导出数据 
=========================================

本文介绍了通过Batch Tool工具导入导出数据的方法。

工具介绍 
-------------------------

Batch Tool工具是PolarDB-X团队开发的专为PolarDB-X数据库提供数据导入导出服务的工具。

Batch Tool工具采用JAVA语句实现，核心是生产者消费者模型，支持多线程操作，提供批量导出、批量导入、批量删除、批量更新等功能。数据以csv文件格式传输，方便用户进行数据交互。

Batch Tool工具的命令用法和参数如下：

```shell
usage: BatchTool [-?] [-batchsize <arg>] [-con <consumer count>] [-cs
           <charset>] -D <database> [-dir <directory>] [-f <from>] [-F
           <filenum>] [-func] -h <host> [-header] [-i] [-in] [-L <line>]
           [-lastSep] [-lb] [-local] [-maxConn <arg>] [-minConn <arg>]
           [-noesc] [-np] [-O <order by type>] -o <operation> [-OC <ordered
           column>] -p <password> [-P <port>] [-para] [-pre <prefix>] [-pro
           <producer count>] [-quote <auto/force/none>] [-readsize <arg>]
           [-rfonly] [-ringsize <arg>] -s <sep> [-t <table>] -u <user> [-w
           <where>]
     -?,--help                              Help message.
     -batchsize,--batchSize <arg>           Batch size of emitted tuples.
     -con,--consumer <consumer count>       Configure number of consumer
                                            threads.
     -cs,--charset <charset>                Define charset of files.
     -D,--database <database>               Database to use.
     -dir,--dir <directory>                 Directory path including files to
                                            import.
     -f,--from <from>                       Source file(s), separated by ; .
     -F,--filenum <filenum>                 Fixed number of exported files.
     -func,--sqlfunc                        Use sql function to update.
     -h,--host <host>                       Connect to host.
     -header,--header                       Whether the header line is column
                                            names.
     -H,--historyFile <filename>            history file name for resuming from breakpoint
     -i,--ignoreandresume                   Flag of insert ignore & resume from breakpoint
     -in,--wherein                          Using where ... in (...)
     -L,--line <line>                       Max line limit of exported files.
     -lastSep,--withLastSep                 Whether line ends with separator.
     -lb,--loadbalance                      If using load balance.
     -local,--localmerge                    o local merge sort.
     -maxConn,--maxConnection <arg>         Max connection number limit.
     -minConn,--minConnection <arg>         Mim connection number limit.
     -noesc,--noescape                      Don't escape values.
     -np,--noparttion                       No use of parttion.
     -O,--orderby <order by type>           asc or desc.
     -o,--operation <operation>             Batch operation type: export /
                                            import / delete / update.
     -OC,--orderCol <ordered column>        col1;col2;col3.
     -p,--password <password>               Password to use when connecting to
                                            server.
     -P,--port <port>                       Port number to use for connection.
     -para,--paraMerge                      Using parallel merge when doing
                                            order by export.
     -pre,--prefix <prefix>                 Export file name prefix.
     -pro,--producer <producer count>       Configure number of producer
                                            threads.
     -quote,--quoteMode <auto/force/none>   The mode of how field values are
                                            enclosed by double-quotes when
                                            exporting table. Default value is
                                            auto.
     -readsize,--readSize <arg>             Read block size in MB.
     -rfonly,--rfonly                       Only read and process file, no sql
                                            execution.
     -ringsize,--ringBufferSize <arg>       Ring buffer size.
     -s,--sep <sep>                         Separator between fields
                                            (delimiter).
     -t,--table <table>                     Target table.
     -tps,--tpsLimit <arg>                  Tps limit
     -u,--user <user>                       User for login.
     -w,--where <where>                     Where condition: col1>99 AND
                                            col2<100 ...
```

 **参数说明**

常用参数说明如下：

* -o：批处理操作，包括export、import、delete、update四个选项。

* -t：指定目标表名，只能为单个表。

* -s：指定分隔符，可以为字符或字符串。

* -f：指定源文件，多个文件名之间使用分号";"分隔。

* -OC：指定导出时排序使用的列名，多个列之间使用分号";"分隔。

* -cs：指定文本文件的字符集，默认为utf-8。

* -lastSep：文件每行是否以分隔符结尾。

* -quote：指定导出或导入时引号包围模式，包括以下三个可选值：
  * auto：默认模式，将根据字段值是否包含特殊字符（如分隔符、换行符等）来添加双引号；
  
  * force：强制每个字段值都添加双引号；
  
  * none：强制不添加双引号（适用于已知表字段类型都是数值型、或字符串型字段中不包含特殊字符的情况）。
  

  

* -header：首行是否为字段名。

* -i：是否开启insert ignore与断点续传。

* -pre：指定导出文件名的前缀。

* -F：指定导出文件数量。




工具获取 
-------------------------

Batch Tool工具的jar包，单击下载：[Batch_tool工具](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/183466/cn_zh/1633679163870/batch-tool%20%281%29.jar) 。

使用示例 
-------------------------

以编译生成的batch-tool.jar为例，查看参数说明：

```shell
java -jar batch-tool.jar -?
```



* 批量导出数据

  ```shell
  ## 1.默认导出（文件数等于表的分片数）
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s ,
  ## 2.导出文件数=3 （-F:指定文件数）
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -F 3
  ## 3.指定单个文件最大行数=10000 (-L:指定单文件行数)
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -L 10000
  ## 4.带where条件 若条件带有空格，需要使用引号 (-w:where条件语句)
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o export -t table_name -s , -w "id < 150 and id > 120"
  ```

  

* 批量导入数据（需手动创建目标表，Batch Tool只包含数据传输）

  ```shell
  ## 1.多个文件用分号 (;) 分隔
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1;table_name_2;table_name_3"
  ## 2.默认根据拆分键进行sharding插入, 如不采用，打开-np开关即可
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "file0;file2" -np
  ## 3.指定生产者、消费者线程（-pro:生产者线程，读取文件线程; -con:消费者线程，导入数据线程）
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1" -np -pro 16 -con 16
  ## 4.打开insert ignore和断点续传
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o import -t table_name -s , -f "table_name_0;table_name_1" -i
  ```

  

* 批量删除数据（删除数据库中包含文件中的数据，原理：构建DELETE语句，根据表结构填充文件中的数据）

  ```shell
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o delete -t table_name -s , -f "file0"
  ```

  

* 批量更新数据（更新数据库中包含文件中的数据，原理：构建UPDTATE语句，根据表结构填充文件中的数据）

  ```shell
  java -jar batch-tool.jar -h 127.0.0.1 -u testUser -p testPasswd -P 3306 -D db_name -o update -t table_name -s , -f "file0"
  ```

  



