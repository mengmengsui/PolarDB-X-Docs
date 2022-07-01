UPDATE 
===========================

您可以使用UPDATE语法修改表中符合条件的行。

语法 
-----------------------

* 单逻辑表

  ```sql
  UPDATE [LOW_PRIORITY] [IGNORE] [schema_name.]tbl_name
      SET assignment_list
      [WHERE where_condition]
  value:
      {expr | DEFAULT}
  assignment:
      col_name = value
  assignment_list:
      assignment [, assignment] ...
  ```

  

* 多逻辑表

  ```sql
  UPDATE [LOW_PRIORITY] [IGNORE] table_references
  SET assignment_list
  [WHERE where_condition]
  ```

  



**说明**

* UPDATE支持如下修饰符：
  * 若设置LOW_PRIORITY，UPDATE操作将在该表没有任何读操作之后执行。
  
  * 若设置IGNORE，将会忽略更新过程中的错误，即更新不会被错误中断。
  

  

* UPDATE语句中的修饰符均会原样下推至存储层MySQL，不会对PolarDB-X的修饰符操作产生影响。




语法限制 
-------------------------

与原生MySQL的UPDATE语法相比，PolarDB-X的UPDATE语法存在以下限制。

不支持在SET子句中使用子查询（相关子查询和非相关子查询），例如：

```unknow
UPDATE t1 SET name = (SELECT name FROM t2 WHERE t2.id = t1.id) WHERE id > 10;
```


**说明** t1和t2的拆分键为ID。

相关文献 
-------------------------

MySQL [UPDATE](https://dev.mysql.com/doc/refman/5.7/en/update.html) 语法。
