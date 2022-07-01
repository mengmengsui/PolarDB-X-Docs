# MySQL 生态兼容 

PolarDB-X将兼容MySQL以及周边生态作为核心设计目标之一。本文从SQL语法、事务行为、导入导出等维度总结了兼容性相关特性。

## MySQL 协议 

PolarDB-X通讯协议兼容MySQL协议，可以使用常见的MySQL客户端直接连接到PolarDB-X集群，包括JDBC Driver、ODBC Driver、Golang Driver等。兼容MySQL SSL、Prepare、Load等传输协议。

## SQL 兼容性 

PolarDB-X兼容MySQL的各种DML、DAL、DDL语法，其中包括：


* 兼容绝大部分MySQL函数（包括JSON函数、加密解密函数等）。

* 兼容MySQL8.0的视图、CTE、窗口函数、分析函数等。

* 支持MySQL的各种数据类型，包括类型精度支持（比如时间戳、Decimal类型）。

* 兼容常见的MySQL字符串Charset及Collation。

* 兼容绝大部分information_schema视图。


更多信息参见[开发指南](../../dev-guide/topics/developer-guide.md)。

## ACID 事务 

PolarDB-X事务采用基于乐观读（ReadView）、悲观锁的设计，支持ANSI标准中的四种隔离级别，且行为和MySQL一致。与MySQL相同，PolarDB-X默认采用可重复读（Repeatable Read）隔离级别，该级别下更新范围条件会引入间隙锁（Gap Lock）。

PolarDB-X基于回滚段的MVCC机制对大事务比较友好，可支持小时级长时间事务、GB级写入量大事务。

## 账号与权限 

PolarDB-X账号权限系统的用法跟MySQL一致，支持GRANT、REVOKE、SHOW GRANTS、CREATE USER、DROP USER、SET PASSWORD等语句。目前支持库级和表级权限的授予，暂不支持列级别权限。更多信息参见[账号与权限](../../dev-guide/topics/permissions.md)。

PolarDB-X还支持基于角色（Role）的权限管理。角色（Role）是分配给账号（User）的权限的集合，定义了允许账号在应用程序中查看和执行的操作。PolarDB-X中的Role操作与MySQL的操作兼容。

## 数据备份与安全（WIP）

PolarDB-X兼容MySQL的常规备份策略。备份操作通常都发生在非主节点上，可以确保备份操作对在线业务流量无影响，同时支持定时全量备份、实时binlog增量备份的能力。支持任意时间点的一致性恢复。

PolarDB-X兼容MySQL的透明数据加密TDE，支持将数据表空间的文件做加密处理，确保业务数据的安全性。

## 数据导入导出 

PolarDB-X兼容MySQL binlog复制协议。用户可以将PolarDB-X集群看作一个普通的MySQL节点，将其他MySQL节点作为PolarDB-X的同步源端或目标端。

PolarDB-X的binlog格式和MySQL原生格式一致，因此也可以用于CDC场景，例如利用 Canal 等将PolarDB-X的写入数据同步到其他存储中。

## 生态工具 

PolarDB-X 对 MySQL 生态工具兼容性比较友好，目前已验证过的工具有 [Canal](https://github.com/alibaba/canal) , MySQL Client 等，更多生态工具正在验证中。
