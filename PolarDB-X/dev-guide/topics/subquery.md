子查询 
========================

本文介绍PolarDB-X支持的子查询类别及在PolarDB-X中使用子查询的相关限制和注意事项。

使用限制 
-------------------------

相比原生MySQL，PolarDB-X在子查询使用上增加了如下限制：

* 不支持在HAVING子句中使用子查询，示例如下：

  ```sql
  SELECT name, AVG( quantity )
  FROM tb1
  GROUP BY name
  HAVING AVG( quantity ) > 2* (
     SELECT AVG( quantity )
     FROM tb2
   );
  ```

  

* 不支持在JOIN ON子句中使用子查询，示例如下：

  ```sql
  SELECT * FROM tb1 p JOIN tb2 s on (p.id=s.id and p.quantity>All(select quantity from tb3))
  ```

  

* 等号操作行符的标量子查询（The Subquery as Scalar Operand）不支持ROW语法。示例如下：

  ```sql
  select * from tb1 where row(id, name) = (select id, name from tb2)        
  ```

  

* 不支持在UPDATE SET子句中使用子查询，示例如下：

  ```sql
  UPDATE t1 SET c1 = (SELECT c2 FROM t2 WHERE t1.c1 = t2.c1) LIMIT 10
  ```

  




注意事项 
-------------------------

PolarDB-X中部分子查询仅能以APPLY的方式执行，查询效率低下。在实际使用中请尽量避免如下例子中的低效SQL：

* WHERE条件中OR与子查询共存时，执行效率会依外表数据情况大幅降低。示例如下：

  ```sql
  高效：select * from tb1 where id in (select id from tb2)
  高效：select * from tb1 where id in (select id from tb2) and id>3
  低效：select * from tb1 where id in (select id from tb2) or  id>3
  ```

  

* 关联子查询（Correlated Subqueries）的关联项中带函数或非等号运算符。示例如下：

  ```sql
  高效：select * from tb1 a where id in
        (select id from tb2 b where a.name=b.name)
  低效：select * from tb1 a where id in
        (select id from tb2 b where UPPER(a.name)=b.name)
  低效：select * from tb1 a where id in
        (select id from tb2 b where a.decimal_test=abs(b.decimal_test))
  低效：select * from tb1 a where id in
        (select id from tb2 b where a.name!=b.name)
  低效：select * from tb1 a where id in
        (select id from tb2 b where a.name>=b.name) 
  ```

  

* 关联子查询（Correlated Subqueries）关联项与其它条件的逻辑运算符为OR。示例如下：

  ```sql
  高效：select * from tb1 a where id in
        (select id from tb2 b where a.name=b.name
                                    and b.date_test<'2015-12-02')
  低效：select * from tb1 a where id in
        (select id from tb2 b where a.name=b.name
                                    or b.date_test<'2015-12-02')
  低效：select * from tb1 a where id in
        (select id from tb2 b where a.name=b.name
                                    or b.date_test=a.date_test)
  ```

  

* 标量子查询（The Subquery as Scalar Operand）带关联项。示例如下：

  ```sql
  高效：select * from tb1 a where id >
          (select id from tb2 b where b.date_test<'2015-12-02')
  低效：select * from tb1 a where id >
          (select id from tb2 b where a.name=b.name 
                                      and b.date_test<'2015-12-02')
  ```

  

* 跨关联层子查询。示例如下：
  * SQL多层关联，每层子查询关联项仅与直接上层关联，此类高效。

    ```sql
    高效：select * from tb1 a where id in(select id from tb2 b 
    where a.name=b.name and 
    exists (select name from tb3 c where b.address=c.address))
    ```

    
  
  * SQL多层关联，但`表c`的子查询关联项中与`表a`的列进行了关联，此类低效。

    ```sql
    低效：select * from tb1 a where id in(select id from tb2 b 
    where a.name=b.name and 
    exists (select name from tb3 c where a.address=c.address))
    ```

    
  

  
  **说明** 上述示例中，`表a`和`表b`、`表b`和`表c`为直接层级关联，`表a`和`表c`间为跨层关联。
  

* 子查询中包含GROUP BY，请确保GROUP BY的分组列包含关联项。示例如下：
  * SQL子查询中包含聚合函数和关联项，关联项`b.pk`包含于分组列`pk`之中，此类高效。

    ```sql
    高效：select * from tb1 a where exists 
        (select pk from tb2 b 
                    where a.pk=b.pk and  b.date_test='2003-04-05' 
                    group by pk);
    ```

    
  
  * SQL子查询中包含聚合函数和关联项，关联项`b.date_test`不包含于分组列`pk`之中，此类低效。

    ```sql
    低效：select * from tb1 a where exists 
        (select pk from tb2 b 
                    where a.date_test=b.date_test and b.date_test='2003-04-05' 
                    group by pk);
    ```

    
  

  




支持的子查询 
---------------------------

PolarDB-X目前支持如下类别的子查询：

* Comparisons Using SubqueriesComparisons Using Subqueries指带有比较运算符的子查询，这类子查询最为常见。

  * 语法

    ```sql
    non_subquery_operand comparison_operator (subquery)
    comparison_operator： =  >  <  >=  <=  <>  !=  <=> like
    ```

    
  
  * 示例

    ```sql
    select * from tb1 WHERE 'a' = (SELECT column1 FROM t1)  
    ```

    
    **说明** 目前仅支持子查询在比较运算符的右边。
    
  

  

* Subqueries with ANY、ALL、IN/NOT IN、EXISTS/NOT EXISTS
  * 语法

    ```sql
    operand comparison_operator ANY (subquery)
    operand comparison_operator ALL (subquery)
    operand IN (subquery)
    operand NOT IN (subquery)
    operand EXISTS (subquery)
    operand NOT EXISTS (subquery)
    comparison_operator:=  >  <  >=  <=  <>  !=
    ```

    
  
  * 示例
    * ANY：如果子查询返回的任意一行满足ANY前的表达式，返回TRUE，否则返回FALSE。
    
    * ALL：如果子查询返回所有行都满足ALL前的表达式，返回TRUE，否则返回FALSE。
    
    * IN：在子查询前使用时，IN等价于`=ANY`。示例如下：

      ```sql
      SELECT s1 FROM t1 WHERE s1 = ANY (SELECT s1 FROM t2);
      SELECT s1 FROM t1 WHERE s1 IN    (SELECT s1 FROM t2);
      ```

      
    
    * NOT IN：NOT IN在子查询前使用时，等价于`<>ALL`。示例如下：

      ```sql
      SELECT s1 FROM t1 WHERE s1 <> ALL (SELECT s1 FROM t2);
      SELECT s1 FROM t1 WHERE s1 NOT IN (SELECT s1 FROM t2);
      ```

      
    
    * EXISTS：如果子查询返回任意行，EXISTS子查询结果为TRUE；如果子查询返回空值，EXISTS子查询结果为FALSE。示例如下：

      ```sql
      SELECT column1 FROM t1 WHERE EXISTS (SELECT * FROM t2);
      ```

      
      **说明** 如果EXISTS子查询中包含任意行，即使只包含NULL的行值，WHERE条件也会返回TRUE。
      
    
    * NOT EXISTS：如果子查询返回任意行，NOT EXISTS子查询结果为FALSE；如果子查询返回空值，NOT EXISTS子查询结果为TRUE。
    

    
  

  

* Row Subqueries
  * Row Subqueries支持如下比较运算符：

    ```sql
    comparison_operator：=  >  <  >=  <=  <>  !=  <=>     
    ```

    
  
  * 示例

    ```sql
    SELECT * FROM t1
      WHERE (col1,col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);
    SELECT * FROM t1
      WHERE ROW(col1,col2) = (SELECT col3, col4 FROM t2 WHERE id = 10);  
    ```

    

    以上两个SQL是等价的，只有同时满足以下条件时，t1表的数据行才会返回：
    * 子查询（`SELECT col3, col4 FROM t2 WHERE id=10 `）仅返回一行记录，返回多行会报错。
    
    * 子查询返回的`col3`，`col4`结果与主表中`col1`，`col2`的值需一一对应。
    

    

    
  

  

* Correlated SubqueriesCorrelated Subqueries指子查询中包含对外层查询表的引用。示例如下：

  ```sql
  SELECT * FROM t1
    WHERE column1 = ANY (SELECT column1 FROM t2
                         WHERE t2.column2 = t1.column2);
  ```

  

  示例子查询SQL中并没有包含表t1及其列名column2，此时会向上一层寻找表t1的引用。
  

* Derived Tables（Subqueries in the FROM Clause）Derived Tables指在FROM子句中的子查询。

  * 语法

    ```sql
    SELECT ... FROM (subquery) [AS] tbl_name ...
    ```

    
  
  * 示例
    1. 数据准备：使用如下语法创建表t1：

       ```sql
       CREATE TABLE t1 (s1 INT, s2 CHAR(5), s3 FLOAT);
       INSERT INTO t1 VALUES (1,'1',1.0);
       INSERT INTO t1 VALUES (2,'2',2.0);
       ```

       

       使用如下查询并得到查询结果为`2, '2', 4.0`。

       ```sql
       SELECT sb1,sb2,sb3
       FROM (SELECT s1 AS sb1, s2 AS sb2, s3*2 AS sb3 FROM t1) AS sb
       WHERE sb1 > 1;
       ```

       

       
    
    2. 查询需求：获取分组数据SUM后的平均值。若直接使用如下SQL则会报错，无法执行：

       ```sql
       SELECT AVG(SUM(s1)) FROM t1 GROUP BY s1;
       ```

       

       此时可使用如下Derived Tables子查询，并得到查询结果为`1.5000`：

       ```sql
       SELECT AVG(sum_s1)
         FROM (SELECT SUM(s1) AS sum_s1
               FROM t1 GROUP BY s1) AS t1;
       ```

       

       **说明**
       * Derived Tables必须拥有一个别名（如示例中的`t1`）。
       
       * Derived Tables可以返回一个标量、列、行或表。
       
       * Derived Tables不可以成为Correlated Subqueries，即不能包含子查询外部表的引用。
       

       
       
    

    
  

  



