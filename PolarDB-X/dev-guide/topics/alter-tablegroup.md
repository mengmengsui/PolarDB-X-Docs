ALTER TABLEGROUP 
=====================================

ALTER TABLEGROUP语句用于修改表组的分区规则，例如分裂、合并和迁移等。

语法 
-----------------------

```sql
ALTER TABLEGROUP tg_name alter_tablegroup_specification;
```



* **分区组分裂**

  ```sql
  -- 对于Range分区，支持以下语法：
  -- 将分区分裂成指定范围的多个分区
  ALTER TABLEGROUP tg_name split PARTITION identifier INTO
   (partition_definition,partition_definition, [, partition_definition] ...);
   partition_definition:
      PARTITION partition_name
          [VALUES
              {LESS THAN {(value_list) | MAXVALUE}
              |
              IN (value_list)}]
  -- 对于Range分区，支持以下语法
  -- 按照指定值将一个range分区一分为二
  ALTER TABLEGROUP tg_name split PARTITION partition_name at(number) into
   (PARTITION partition_name,
    PARTITION partition_name);
  -- 对于List分区，支持以下语法：
  ALTER TABLEGROUP tg_name split PARTITION partition_name into
   (partition_definition,partition_definition, [, partition_definition] ...);
  -- 对于Hash分区，支持以下语法：
  -- 用分区均值作为分裂点，将一个分区切分为两个分区
  ALTER TABLEGROUP tg_name SPLIT PARTITION partition_name; 
  -- 将热点值提取到单独到分区
  ALTER tablegroup tgName extract to partition by hot value(xxx)
  ```

  

* **分区合并**

  将多个分区（两个或者两个以上）合并成一个分区。

  ```sql
  ALTER TABLEGROUP tg_name MERGE PARTITIONS partition_name,...,partition_name TO partition_name;
  e.g.
  ALTER TABLEGROUP tbl_tg MERGE PARTITIONS p2,p3 to p23;
  ```

  

* **分区迁移**

  将分区迁移到指定的DN节点。

  ```sql
  ALTER TABLEGROUP tg_name MOVE PARTITIONS partition_name,...,partition_name TO dn_id
  e.g.
  ALTER TABLEGROUP tg_name MOVE PARTITIONS p2,p4 to 'dn-0' ;
  ```

  

  

* **增加分区**

  对于Range/List策略的分区组，支持增加分区。

  ```sql
  ALTER TABLEGROUP tg_name add partition (partition_definition [,partition_definition] ...)
  ```

  

* **删除分区**

  对于Range/List分区策略的分区组，支持删除分区。

  ```sql
  ALTER TABLEGROUP tg_name drop partition partitiion_name [,partition_name] ...
  ```

  

* **重命名分区**

  ```sql
  ALTER TABLEGROUP tg_name rename partition partitiion_name to partitiion_name[, partitiion_name to partitiion_name]
  ```

  

* **修改分区值**

  只支持对list/list column分区策略的分区修改分区值。

  ```sql
  ALTER TABLEGROUP tg_name modify partition_name add/drop values in (value_list)
  ```

  

  



