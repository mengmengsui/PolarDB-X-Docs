如何优化数据全量抽取 
===============================

本文介绍了在应用内通过代码高效抽取数据的方法。

简介 
-----------------------

数据抽取是指通过代码或者数据导出工具，从PolarDB-X中批量读取数据的操作。主要包括以下场景：

* 通过数据导出工具将数据全量抽取到下游系统。PolarDB-X支持多种数据导出工具，更多内容请参考[数据导入导出](../../maintance/topics/data-import-and-export.md)。

* 在应用内处理数据或者批量的将查询结果返回给用户浏览时，不能依赖外部工具，必须在应用内通过代码完成数据全量抽取。




本文主要介绍在应用内通过代码高效抽取数据的方法，根据是否一次性读取全量数据，分为全量抽取和分页查询。

全量抽取场景 
---------------------------

全量抽取使用的SQL通常不包含表的拆分键，以全表扫描的方式执行，随着读取数据量的增加，数据抽取操作的执行时间线性增长。为了避免占用过多网络/连接资源，可以使用HINT直接下发查询语句，从物理分片中拉取数据。以下示例采用JAVA代码编写，完整使用方法参考 [NODE HINT](../../dev-guide/topics/hint-faq.md)。

```java
public static void extractData(Connection connection, String logicalTableName, Consumer<ResultSet> consumer)
    throws SQLException {
    final String topology = "show topology from {0}";
    final String query = "/*+TDDL:NODE({0})*/select * from {1}";
    try (final Statement statement = connection.createStatement()) {
        final Map<String, List<String>> partitionTableMap = new LinkedHashMap<>();
        // Get partition id and physical table name of given logical table
        try (final ResultSet rs = statement.executeQuery(MessageFormat.format(topology, logicalTableName))) {
            while (rs.next()) {
                partitionTableMap.computeIfAbsent(rs.getString(2), (k) -> new ArrayList<>()).add(rs.getString(3));
            }
        }
        // Serially extract data from each partition
        for (Map.Entry<String, List<String>> entry : partitionTableMap.entrySet()) {
            for (String tableName : entry.getValue()) {
                try (final ResultSet rs = statement
                    .executeQuery(MessageFormat.format(query, entry.getKey(), tableName))) {
                    // Consume data
                    consumer.accept(rs);
                }
            }
        }
    }
}
```



分页查询场景 
---------------------------

向用户展示列表信息时，需要分页来提高页面的加载效率，避免返回过多冗余信息，用于处理分页显示需求的查询，称为分页查询。关系型数据库没有直接提供分段返回表中数据的能力，高效的实现分页查询，还需要结合数据库本身的特点来设计查询语句。

以MySQL为例，分页查询最直观的实现方法，是使用limit offset，pageSize来实现，例如如下查询：

```sql
select * from t_order where user_id = xxx order by gmt_create, id limit offset, pageSize
```



因为gmt_create可能重复，所以order by时应加上id，保证结果顺序的确定性。

**说明** 该方案在表规模较小的时候，能够正常运行。当t_order表增长到十万级，随着页数增加，执行速度明显变慢，可能降到几十毫秒的量级，如果数据量增长到百万级，则耗时达到秒级，数据量继续增长，耗时最终会变得不可接受。

**问题分析**

假设我们在user_id, gmt_create上创建了局部索引，由于只有user_id上的条件，每次需要扫描的总数据量为offset + pageSize ，随着offset的增大逐渐接近全表扫描，导致耗时增加。并且在分布式数据库中，全表排序的吞吐无法通过增加DN数量来提高。

**改进方案1**

每次获取下一页记录时，指定从上次结束的位置继续往后取，这样不需要设置offset ，能够避免出现全表扫描的情况。看一个按id进行分页查询的例子：

```sql
select * from t_order where id > lastMaxId order by id limit pageSize
```



第一次查询不指定条件，后续查询则传入前一次查询的最大id，在执行时，数据库首先在索引上定位到lastMaxId的位置，然后连续返回pageSize条记录即可，非常高效。

**说明** 当id为主键或者唯一键时，改进方案1可以达到分页查询的效果，也有不错的性能。但缺点也比较明显，当id上有重复值时，可能会漏掉部分记录。

**改进方案2**

MySQL支持通过 [Row Constructor Expression](https://dev.mysql.com/doc/refman/8.0/en/row-constructor-optimization.html) 实现多列比较的语义（PolarDB-X同样支持）。

```sql
(c2,c3) > (1,1) 
等价于 
c2 > 1 OR ((c2 = 1) AND (c3 > 1))
```



因此，可以用下面的方法实现分页查询语义：

```sql
select * from t_order 
where user_id = xxx and (gmt_create, id) > (lastMaxGmtCreate, lastMaxId)
order by user_id, gmt_create, id limit pageSize
```



第一次查询不指定条件，后续查询则传入前一次查询的最大gmt_create和id，通过Row Constructor Expression正确处理gmt_create存在重复的情况。

**说明** 示例中，为了提高查询性能，我们在user_id和gmt_create上建立联合索引，并在order by中加入user_id提示优化器可以通过索引来消除排序。由于Row Constructor Expression包含null值会导致表达式求值结果为null，当存在null值时需要使用OR表达式。PolarDB-X目前只在Row Constructor Expression仅包含拆分键时才将其用于分区裁剪，其他场景同样需要使用OR表达式。

结合上述分析，给出一个PolarDB-X上分页查询的最佳实践：

```sql
-- lastMaxGmtCreate is not null 
select * from t_order 
where user_id = xxx 
and (
      (gmt_create > lastMaxGmtCreate) 
      or ((gmt_create = lastMaxGmtCreate) and (id > lastMaxId))
    )
order by user_id, gmt_create, id limit pageSize
-- lastMaxGmtCreate is null
select * from t_order 
where user_id = xxx 
and (
      (gmt_create is not null)
      or (gmt_create is null and id > lastMaxId)
    )
order by user_id, gmt_create, id limit pageSize
```


