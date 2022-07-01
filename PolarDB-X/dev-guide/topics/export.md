EXPORT 
===========================

本文介绍了PolarDB-X两种导出数据语法。

mysqldump 
------------------------------

PolarDB-X支持MySQL官方数据导出工具mysqldump。mysqldump命令的详细说明请参见[mysqldump](https://dev.mysql.com/doc/refman/5.6/en/mysqldump.html) 。

* **语法**

  ```sql
         mysqldump -h ip -P port -u user -ppassword --default-character-set=char-set --net_buffer_length=10240 --no-create-db --skip-add-locks --skip-lock-tables --skip-tz-utc --set-charset  [--hex-blob] [--no-data] database [table1 table2 table3...] > dump.sql
  ```

  

* **参数说明** 

  |    参数名称    |                                 说明                                  | 是否必选 |
  |------------|---------------------------------------------------------------------|------|
  | ip         | PolarDB-X实例的IP。                                                     | 是    |
  | port       | PolarDB-X实例的端口。                                                     | 否    |
  | user       | PolarDB-X实例的用户名。                                                    | 是    |
  | password   | PolarDB-X实例的密码。 **说明** 密码前存在-p，之间没有空格。                              | 是    |
  | char-set   | 指定的编码。                                                              | 是    |
  | --hex-blob | 使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用本选项。影响的字段类型包括BINARY、VARBINARY、BLOB。 | 否    |
  | --no-data  | 不导出数据。                                                              | 否    |
  | table      | 指定导出某个表。默认导出该数据库所有的表。                                               | 否    |

  




SELECT ... INTO OUTFILE 
--------------------------------------------

* **语法**

  ```sql
  SELECT ... INTO OUTFILE 'file_name'
          [CHARACTER SET charset_name]
          [export_options]
  export_options:
      [{FIELDS | COLUMNS}
          [TERMINATED BY 'string']
          [[OPTIONALLY] ENCLOSED BY 'char']
          [ESCAPED BY 'char']
      ]
      [LINES
          [STARTING BY 'string']
          [TERMINATED BY 'string']
  ```

  

* **示例**

  ```sql
  SELECT customer_id, firstname, surname INTO OUTFILE '/exportdata/customers.txt'
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  FROM customers;
  ```

  

* **限制**

  目前OUTFILE和mysql语法一致，导出的文件会保存在执行该SQL会话对应的PolarDB-X计算节点。
  



