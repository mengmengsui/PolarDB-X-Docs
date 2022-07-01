# 连接PolarDB-X 

PolarDB-X 支持通过 MySQL Client 命令行、第三方客户端以及符合 MySQL 交互协议的第三方程序代码进行连接。本文主要介绍如何通过 MySQL Client 命令行连接到 PolarDB-X 数据库。

## 前提条件

如果您尚未安装 PolarDB-X，可通过[快速体验](../../quickstart/topics/quickstart.md) 安装一个 PolarDB-X 实例。

如果您尚未安装 MySQL 客户端，请前往 [MySQL 网站](https://dev.mysql.com/downloads/mysql/) 下载或通过其他方式安装。

## 通过 MySQL 命令行连接到数据库 

1. 获取 PolarDB-X 连接地址

安装 PolarDB-X 后，您将获取到地址、端口、账号和密码信息，保存这些信息，并进入下一步


2. 通过如下命令行进行连接：

```bash
mysql -h<连接地址> -P<端口> -u<用户名> -p -D<数据库名称>
```


| 选项 | 说明          | 示例            |
| ---- | ------------ | --------------- |
| `-h` | 实例的连接地址。                                             | `192.168.2.56`  |
| `-P` | 实例的端口号。  <br />**说明**：<br />- 此处`-P`为大写字母。<br />- 默认端口为 3306。 | `3306`          |
| `-u` | 实例中的账号名称。<br />关于如何创建账号，请参见[账号权限管理](../../dev-guide/topics/account-devguide.md) 。 | `polardbx_root` |
| `-p` | 以上账号的密码。 <br />**说明**：<br />- 为保障密码安全，`-p`后请不要填写密码，<br />会在执行整行命令后提示您输入密码，<br />输入后按回车即可登录。<br />- 如果填写该参数，`-p`与密码之间不能有空格。 | `passWord123`   |
| `-D` | 需要登录的数据库名称。 <br />**说明**：<br />- 该参数非必填参数。<br />- 可以不输入`-D`仅输入数据库名称。 | `poalrdbx`      |

## 通过第三方客户端连接到数据库 

PolarDB-X已验证支持如下第三方客户端进行连接，您可以去对应的官方网站下载客户端。 

* MySQL Workbench（推荐）

* SQLyog

* Sequel Pro

* Navicat for MySQL

**说明** 第三方GUI客户端可执行基础的数据库操作，包括数据的增删改查和DDL操作，对于工具高级特性，PolarDB-X可能并不支持。

## 通过第三方程序代码连接到数据库 

PolarDB-X支持通过如下符合MySQL官方交互协议的第三方程序代码进行连接：

* JDBC Driver for MySQL (Connector/J)

* Python Driver for MySQL (Connector/Python)

* C++ Driver for MySQL (Connector/C++)

* C Driver for MySQL (Connector/C)

* ADO.NET Driver for MySQL (Connector/NET)

* ODBC Driver for MySQL (Connector/ODBC)

* PHP Drivers for MySQL (mysqli, ext/mysqli, PDO_MYSQL, PHP_MYSQLND)

* Perl Driver for MySQL (DBD::mysql)

* Ruby Driver for MySQL (ruby-mysql)


以下为JDBC Driver for MySQL （Connector/J）程序代码示例。 

```java
//JDBC
Class.forName("com.mysql.jdbc.Driver"); 
Connection conn = DriverManager.getConnection("jdbc:mysql://192.168.2.56:3306/poc","poc","poc_password");
//...
conn.close();
```


以下为应用端连接池配置示例。

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
<property name="url" value="jdbc:mysql://192.168.2.56:3306/poc" />
<property name="username" value="poc" />
<property name="password" value="poc_password" />
<property name="filters" value="stat" />
<property name="maxActive" value="100" />
<property name="initialSize" value="20" />
<property name="maxWait" value="60000" />
<property name="minIdle" value="1" />
<property name="timeBetweenEvictionRunsMillis" value="60000" />
<property name="minEvictableIdleTimeMillis" value="300000" />
<property name="testWhileIdle" value="true" />
<property name="testOnBorrow" value="false" />
<property name="testOnReturn" value="false" />
<property name="poolPreparedStatements" value="true" />
<property name="maxOpenPreparedStatements" value="20" />
<property name="asyncInit" value="true" />
</bean>
```

**说明** 推荐使用 Druid 连接池连接，关于 Druid 的详细信息请参见[Druid Github资源](https://github.com/alibaba/druid) 。
