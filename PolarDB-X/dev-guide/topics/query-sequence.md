查询与获取Sequence 
==================================

本文主要介绍如何查询Sequence类型以及获取Sequence值。

查询Sequence 
-------------------------------

**语法** 

```sql
SHOW SEQUENCES
```



**示例** 

请在命令行输入如下代码：

```sql
mysql> SHOW SEQUENCES;
```



返回结果如下：

```sql
+------+--------+------------+------------+------------+--------------+------------+-------------+-------+--------+
| NAME | VALUE  | UNIT_COUNT | UNIT_INDEX | INNER_STEP | INCREMENT_BY | START_WITH | MAX_VALUE   | CYCLE | TYPE   |
+------+--------+------------+------------+------------+--------------+------------+-------------+-------+--------+
| seq1 | 100000 | 1          | 0          | 100000     | N/A          | N/A        | N/A         | N/A   | GROUP  |
| seq2 | 400000 | 3          | 1          | 100000     | N/A          | N/A        | N/A         | N/A   | GROUP  |
| seq3 | N/A    | N/A        | N/A        | N/A        | N/A          | N/A        | N/A         | N/A   | TIME   |
| seq4 | 1006   | N/A        | N/A        | N/A        | 2            | 1000       | 99999999999 | N     | SIMPLE |
+------+--------+------------+------------+------------+--------------+------------+-------------+-------+--------+
4 rows in set (0.00 sec)
```


**说明** 返回结果中的TYPE列，显示的是Sequence类型的缩写。

获取显式Sequence值 
----------------------------------

**语法** 

```sql
[<schema_name>.]<sequence name>.NEXTVAL
```



**示例** 

* 方法一请在命令行输入如下代码：

  ```sql
  mysql> SELECT sample_seq.nextval FROM dual;
  ```

  

  返回结果如下：

  ```sql
  +--------------------+
  | SAMPLE_SEQ.NEXTVAL |
  +--------------------+
  |             101001 |
  +--------------------+
  1 row in set (0.04 sec)
  ```

  

* 方法二请在命令行输入如下代码：

  ```sql
  mysql> INSERT INTO some_users (name,address,gmt_create,gmt_modified,intro) VALUES ('sun',sample_seq.nextval,now(),now(),'aa');
  ```

  
  **说明**
  * 该方法是把sample_seq.nextval当做一个值写入了 SQL中。
  
  * 如果建表时已经指定了AUTO_INCREMENT参数，INSERT时不需要指定自增列，可以让PolarDB-X自动维护。
  

  
  




批量获取Sequence值 
----------------------------------

**语法** 

批量获取Sequence值的语法如下：

```sql
SELECT [<schema_name>.]<sequence name>.NEXTVAL FROM DUAL WHERE COUNT = <numeric value>
```



**示例** 

请在命令行输入如下代码：

```sql
mysql> SELECT sample_seq.nextval FROM dual WHERE count = 10;
```



返回结果如下：

```sql
+--------------------+
| SAMPLE_SEQ.NEXTVAL |
+--------------------+
|             101002 |
|             101003 |
|             101004 |
|             101005 |
|             101006 |
|             101007 |
|             101008 |
|             101009 |
|             101010 |
|             101011 |
+--------------------+
10 row in set (0.04 sec)
```


