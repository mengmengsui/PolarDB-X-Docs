SOURCE 
===========================

本文介绍PolarDB-X两种导入数据命令。

mysql命令导入 
------------------------------

使用mysql命令导入语法格式：

```shell
mysql -u用户名    -p密码    <  要导入的数据库数据(runoob.sql)
```



source命令导入 
-------------------------------

source命令导入数据库需要先登录到数库终端：

```sql
mysql> create database abc;      # 创建数据库
mysql> use abc;                  # 使用已创建的数据库 
mysql> set names utf8;           # 设置编码
mysql> source /home/abc/abc.sql  # 导入备份数据库
```



