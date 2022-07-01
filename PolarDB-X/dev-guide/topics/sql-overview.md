SQL概述 
==========================

本文简单介绍了PolarDB-X语法及操作。

PolarDB-X是一款分布式关系数据库，高度兼容MYSQL语法，但由于分布式数据库和单机数据库架构的差异，又有着自身的语法特点。所以本章节除了介绍数据类型、运算符和函数等基本元素，还会从下面五种类别介绍PolarDB-X的语法。

* 数据定义语言DDL（Data Definition Language）：对数据库中资源进行定义、修改和删除，如新建表和删除表等。

* 数据操作语言DML（Data Manipulation Language）：用以改变数据库中存储的数据内容，即增加、修改和删除数据。

* 数据查询语言DQL（Data Query Language）：也称为数据检索语言，用以从表中获得数据，并描述怎样将数据返回给程序输出。

* 事务控制语言TCL（Transaction Control Language）：保证数据库的完整性、一致性，在同一个事务中的 DML 语句要么同时成功，要么同时失败。

* 数据控制语句DAL（Data Access Language）：相关运维指令。可以配合这些运维指令，协助DBA操作和优化数据库。



