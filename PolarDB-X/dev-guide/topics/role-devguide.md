角色权限管理 
===========================

本文介绍角色权限管理相关语法级示例。

PolarDB-X兼容原生MySQL 8.0基于角色的权限控制，请参见[基于角色的权限控制](https://dev.mysql.com/doc/refman/8.0/en/roles.html) 。

创建角色 
-------------------------

语法：

```sql
CREATE ROLE role [, role]...
```



role同user一样也由Name和Host这两部分组成，其中：

* Name不能为空；

* Host需满足如下规则：
  * 必须是纯IP地址，可以包含下划线（_）和百分号（%），但这两个符号仅代表2个普通字符，并不具备通配符意义；
  
  * Host留空等于%，但也是精准匹配，不具备通配符意义。
  

  




示例：

```sql
mysql> CREATE ROLE 'role_ro'@'%', 'role_write';
```



删除角色 
-------------------------

语法：

```sql
DROP ROLE role [, role] ...
```



示例：

```sql
mysql> DROP ROLE 'role_ro'@'%';
```



授予角色 
-------------------------

**将权限授予角色**

语法：

```sql
GRANT priv_type [, priv_type] ... ON priv_level TO role [, role]... [WITH GRANT OPTION]
```



示例：

```sql
mysql> GRANT ALL PRIVILEGES ON db1.* TO 'role_write';
```

 **将角色授予用户**

语法：

```sql
GRANT role [, role] ...
TO user_or_role [, user_or_role] ...
[WITH ADMIN OPTION]
```



说明：

* 执行该命令必须满足如下条件的其中之一：
  * 当前用户有CREATE_USER权限；
  
  * 当前用户对Role有admin权限；
  

  

* 如果包含WITH ADMIN OPTION选项，则目标用户对该Role拥有admin权限；

* 将角色授予用户并不代表此用户已拥有该角色下的权限，您还需要通过`SET DEFAULT ROLE`语句和`SET ROLE`语句为用户设置需要激活的角色。




示例：

```sql
mysql> GRANT 'role_write' TO 'user1'@'127.0.0.1';
```

 **设置默认角色**

语法：

```sql
SET DEFAULT ROLE
{NONE | ALL | role [, role ] ...}
TO user [, user ] ...
```



执行该命令必须满足如下条件的其中之一：

* 语句中所提到的Role已通过GRANT命令授予给目标用户；

* 当前用户为目标用户，或当前用户有CREATE_USER权限。




示例：

```sql
mysql> SET DEFAULT ROLE 'role_write' TO 'user1'@'127.0.0.1';
```

 **设置当前连接角色**

语法：

```sql
SET ROLE {
    DEFAULT
  | NONE
  | ALL
  | ALL EXCEPT role [, role ] ...
  | role [, role ] ...
}
```


**说明**

* 若选择执行`SET ROLE DEFAULT` ，则当前激活的角色为`SET DEFAULT ROLE`命令中选择的角色；

* 通过该语法激活的角色仅对使用当前连接的用户生效。




示例：

```sql
mysql> SET ROLE 'role_write';;
```



查看角色权限 
---------------------------

语法：

```sql
SHOW GRANTS
    [FOR user_or_role
        [USING role [, role] ...]]
```



示例：

```sql
mysql>  SHOW GRANTS FOR 'role_write'@'%';
+---------------------------------------------------+
| GRANTS FOR 'ROLE_WRITE'@'%'                       |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO 'role_write'@'%'            |
| GRANT ALL PRIVILEGES ON db1.* TO 'role_write'@'%' |
+---------------------------------------------------+
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1' USING 'role_write';
+------------------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                       |
+------------------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'            |
| GRANT ALL PRIVILEGES ON db1.* TO 'user1'@'127.0.0.1' |
| GRANT 'role_write'@'%' TO 'user1'@'127.0.0.1'        |
+------------------------------------------------------+
-- 以user1的会话执行
mysql> SELECT CURRENT_ROLE();
+------------------+
| CURRENT_ROLE()   |
+------------------+
| 'role_write'@'%' |
+------------------+
```



回收角色 
-------------------------

**回收角色的权限**

语法：

```sql
REVOKE priv_type [, priv_type] ... ON priv_level FROM role [, role]...
```



示例：

```sql
mysql> REVOKE ALL PRIVILEGES ON db1.* FROM 'role_write';
mysql> SHOW GRANTS FOR 'role_write'@'%';
+----------------------------------------+
| GRANTS FOR 'ROLE_WRITE'@'%'            |
+----------------------------------------+
| GRANT USAGE ON *.* TO 'role_write'@'%' |
+----------------------------------------+
```

 **回收用户的权限**

语法：

```sql
REVOKE role [, role ] ... FROM user_or_role [, user_or_role ] ...
```



示例：

```sql
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+-----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'                |
+-----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'     |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1'  |
| GRANT 'role_write'@'%' TO 'user1'@'127.0.0.1' |
+-----------------------------------------------+
mysql> REVOKE 'role_write' FROM 'user1'@'127.0.0.1';
mysql> SHOW GRANTS FOR 'user1'@'127.0.0.1';
+----------------------------------------------+
| GRANTS FOR 'USER1'@'127.0.0.1'               |
+----------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'127.0.0.1'    |
| GRANT SELECT ON db1.* TO 'user1'@'127.0.0.1' |
+----------------------------------------------+
```


