DELETE 
===========================

您可以使用DELETE语句删除表中符合条件的行。

语法 
-----------------------

下述DELETE语句表示从 `tbl_name`中删除满足`where_condition`的行，并返回删除的行数；若没有WHERE条件，将删除表中所有的数据。

* 单逻辑表

  ```sql
  DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM [schema_name.]tbl_name
  [WHERE where_condition]
  ```

  

* 多逻辑表

  ```sql
  DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
      tbl_name[.*] [, tbl_name[.*]] ...
      FROM table_references
      [WHERE where_condition]
  DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
      FROM [schema_name.]tbl_name[.*] [, [schema_name.]tbl_name[.*]] ...
      USING table_references
      [WHERE where_condition]           
  ```

  



**说明**

* DELETE支持如下修饰符：
  * 若设置LOW_PRIORITY，DELETE操作将在该表没有任何读操作之后执行。
  
  * 若设置IGNORE，则会忽略删除过程中产生的错误。
  
  * QUICK，与MySQL存储引擎相关，详情请参见[MySQL文档](https://dev.mysql.com/doc/refman/5.7/en/delete.html) 。
  

  

* DELETE语句中的修饰符均会原样下推至存储层MySQL，不会对PolarDB-X的修饰符操作产生影响。



