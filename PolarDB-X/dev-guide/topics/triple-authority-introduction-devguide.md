三权分立功能介绍（WIP）
=========================

PolarDB-X新增三权分立模式，您可以将高权限账号拥有的权限分给系统管理员、安全管理员和审计管理员这3个角色，避免因权限高度集中带来的风险，增强数据库的安全性。

风险与解决方案 
----------------------------

* **风险**

  传统数据库运维模式下，数据库管理员DBA（Database Administrator）拥有的权限过高且集中，容易在某些场景下给业务带来风险：
  * DBA误判导致系统安全事故。
  
  * DBA出于某种目的进行非法操作。
  
  * DBA、第三方外包人员或程序开发人员越权访问敏感信息。
  
  
  
  
  
* **解决方案**

  PolarDB-X新增支持三权分立模式，改进传统数据库运维由DBA行使特权的独立控制体系，使得数据库管理员DBA、安全管理员DSA（Department Security Administrator）和审计管理员DAA（Data Audit Administrator）3者的权责更加清晰。其中：
  * 数据库管理员（DBA）：只具备DDL（Data Definition Language）权限。
  
  * 安全管理员（DSA）：只具备管理角色（Role）或用户（User）以及为其他账号授予权限的权限。
  
  * 审计管理员（DAA）：只具备查看审计日志的权限。
  
  
  




数据库系统账号的权限对比 
---------------------------------

下表展示了在默认模式和三权分立模式下，不同数据库系统账号的权限对比。

**说明**

* 默认模式下的高权限账号即系统管理员账号。更多关于高权限账号的详情，请参见[账号类型](permissions.md)。

* 开启或关闭三权分立模式，仅对系统账号（即高权限账号、系统管理员账号、安全管理员账号和审计管理员账号）的权限有影响，普通账号权限不受模式变更的影响。

* 三权分立模式下，虽然所有系统账号均不具备DML（Data Manipulation Language）、DQL（Data Query Language）或DAL（Data Administration Language）权限，但安全管理员仍然能够将这些权限授予给普通账号。

* 表中✔️表示具备该权限，❌表示不具备该权限。





| 权限分类       | 说明                                                         | 默认模式（高权限账号） | 三权分立模式（系统管理员账号） | 三权分立模式（安全管理员账号） | 三权分立模式（审计管理员账号） |
| -------------- | ------------------------------------------------------------ | ---------------------- | ------------------------------ | ------------------------------ | ------------------------------ |
| DDL            | * ALTER TABLE  * CREATE TABLE  * CREATE VIEW  * CREATE INDEX  * CREATE CCL_RULE  * DROP VIEW  * DROP INDEX  * DROP TABLE  * TRUNCATE TABLE | ✔️                      | ✔️                              | ❌                              | ❌                              |
| DML            | * DELETE  * UPDATE  * INSERT                                 | ✔️                      | ❌                              | ❌                              | ❌                              |
| DQL            | * SELECT  * EXPLAIN                                          | ✔️                      | ❌                              | ❌                              | ❌                              |
| DAL            | * SHOW CCL_RULE  * SHOW INDEX                                | ✔️                      | ❌                              | ❌                              | ❌                              |
| 账号或角色相关 | [账号权限管理](account.md) [角色权限管理](role-devguide.md)  | ✔️                      | ❌                              | ✔️                              | ❌                              |
| 查看审计日志   | 查看如下两张表中的审计日志信息： * `information_schema.polardbx_audit_log`  * `information_schema.polardbx_ddl_log` | ✔️                      | ❌                              | ❌                              | ✔️                              |



使用限制 
-------------------------

三权分立模式下的系统账号（包括系统管理员账号、安全管理员账号和审计管理员账号）存在如下限制：

* 不支持对系统账号执行GRANT ROLE或REVOKE ROLE命令。

* 不支持对系统账号执行GRANT PRIVILEGES 或REVOKE PRIVILEGES命令。

* 系统账号的密码只能由对应的账号修改，如系统管理员账号的密码仅能由系统管理员账号修改，不能被其他帐号修改。

* 系统账号均不支持SET DEFAULT ROLE命令。



