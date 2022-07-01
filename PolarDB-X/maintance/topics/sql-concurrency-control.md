SQL限流 
==========================

为应对突发的数据库请求流量、资源消耗过高的语句访问以及SQL访问模型的变化等问题，PolarDB-X提供了节点级别的SQL限流功能来限制造成上述问题的SQL执行，从而保证实例的持续稳定运行。本文介绍如何使用SQL限流功能。

创建限流规则 
---------------------------

* **语法**

  ```sql
  CREATE CCL_RULE [ IF NOT EXISTS ] `ccl_rule_name`
  ON `database`.`table`
  TO '<usename>'@'<host>'
  FOR { UPDATE | SELECT | INSERT | DELETE }
  [ filter_options ]
  with_options
  filter_options:
      [ FILTER  BY KEYWORD('KEYWORD1', 'KEYWORD2',...) ]
      [ FILTER  BY TEMPLATE('template_id') ]
   with_options:
      WITH MAX_CONCURRENCY = value1 [ , WAIT_QUEUE_SIZE = value2 ] [ , WAIT_TIMEOUT = value3 ] [ ,FAST_MATCH = { 0 , 1 }]
  ```

  

  |                       参数                        || 是否必选 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                           说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
  |------------|-------------------------------------|------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | 限流规则匹配参数   | ```ccl_rule_name```                 | 必选   | 限流规则的名称。 **说明** 为避免名称与SQL关键字冲突，建议在规则名称前后各加一个反引号（\`）                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
  | 限流规则匹配参数   | ```database`.`table```              | 必选   | 数据库和数据表的名称，支持使用星号（\*）表示任意匹配。 **说明** 为避免名称与SQL关键字冲突，建议在库表名称前后各加一个反引号（\`）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
  | 限流规则匹配参数   | `'<usename>'@'<host>'`              | 必选   | 账号名称。其中Host部分支持用百分号（%）来表示任意匹配。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
  | 限流规则匹配参数   | `UPDATE | SELECT | INSERT | DELETE` | 必选   | SQL语句类型。当前支持UPDATE、SELECT、INSERT和DELETE类型。 **说明** 每条限流规则仅支持传入一种类型的SQL语句。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
  | 限流规则匹配参数   | `[ filter_options ]`                | 可选   | 过滤条件，支持包括如下两种条件： * 关键词（KEYWORD）：查看限流规则时，关键词列表会在查询结果中被转化为`["kwd1","kw2","kw3"...]`的字符串形式，最多支持512个字符。 **说明** * 若关键字是SQL语句中的参数值，匹配时大小写敏感。  * 若关键字是SQL语句中的其他词，匹配时大小写不敏感。     * 模版（TEMPLATE）：模版编号是SQL日志中的`sql_code`值，该值是参数化后的SQL语句（SQL模版）以16进制表示的哈希值。您可以通过SHOW FULL PROCESSLIST和EXPLAIN命令查看模版编号。                                                                                                                                                                                                                                                                                                                                                                                                                                         |
  | 限流规则行为控制参数 | ` with_options`                     | 必选   | WITH选项中支持如下4个参数来控制限流规则的行为： * MAX_CONCURRENCY：匹配到该限流规则的SQL语句的最大并发度，超过后进入等待队列。 取值范围：\[0\~2^31^ - 1\]，默认值为0。   * WAIT_QUEUE_SIZE：超过并发度后的最大等待队列长度。当等待队列长度超过该值后，SQL语句将报错。在队列中的语句仍然占用了线程资源，排队过多时也可能导致内存耗尽。 取值范围：\[0\~2^31^ - 1\]，默认值为0。   * WAIT_TIMEOUT：SQL语句在等待队列中的最长等待时间，超过该等待时间后，SQL语句将报错。 取值范围：\[0\~2^31^ - 1\]，单位为秒，默认值为600。   * FAST_MATCH：是否开启Cache来加速匹配。开启后，PolarDB-X 2.0会将模版编号作为Cache key的一部分，匹配结果作为value进行缓存，来加速匹配速度。 取值范围：0表示关闭，1表示开启，默认开启。    **说明** * 创建限流规则时，需从上述4个行为控制参数中至少选择一个传入。  * 当MAX_CONCURRENCY为默认值（0）时，可能会使匹配到的所有SQL返回错误。此时，建议您显式指定该参数为非0的值。  * PolarDB-X 2.0是分布式云原生数据库，计算层由多个节点组成，因此每个节点的并发度之和是整个实例的并发数最大值。在负载不均衡的情况下，整个实例的受限制SQL并发数可能无法达到最大并发数。   |
  [参数说明]

  
  **说明** 仅当一个SQL语句满足所有的匹配参数条件时，才会根据该规则的WITH选项进行限流。
  

* **限流结果**

  一条SQL匹配到该规则后，根据限流规则中WITH选项里配置的参数，会出现如下几种结果：
  * **RUN（可运行）**

    若并发度还未达到最大并发度（即MAX_CONCURRENCY参数值），该SQL正常执行不会被限流。
    
  
  * **WAIT（等待中）**

    若并发度已经达到最大并发度，但等待队列长度还未达到最大长度（即WAIT_QUEUE_SIZE参数值），该SQL进入等待状态，直到进入可运行（RUN）状态，或者等待超时（WAIT_TIMEOUT）状态。

    您可以通过如下命令查看由于匹配到限流规则而等待的SQL语句：

    ```sql
    mysql> SHOW FULL PROCESSLIST;
    ```

    

    返回结果示例如下：

    ```sql
    +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
    | ID | USER          | HOST            | DB       | COMMAND                       | TIME | STATE | INFO                  | SQL_TEMPLATE_ID |
    +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
    |  2 | polardbx_root | ***.*.*.*:62787 | polardbx | Query                         |    0 |       | show full processlist | NULL            |
    |  1 | polardbx_root | ***.*.*.*:62775 | polardbx | Query(Waiting-selectrulereal) |   12 |       | select 1              | 9037e5e2        |
    +----+---------------+-----------------+----------+-------------------------------+------+-------+-----------------------+-----------------+
    2 rows in set (0.08 sec)
    ```

    

    从上述查询结果可以看出：SQL语句`select 1`由于限流规则`selectrulereal`而处于等待（Waiting）状态。
    
  
  * **WAIT_TIMEOUT（等待超时）**

    SQL语句进入等待状态后，当等待时间超过最长等待时间（即WAIT_TIMEOUT参数值）时，该语句将会返回错误。

    例如，设置了一条最长等待时间为10秒的限流规则，执行`SELECT sleep(11)`语句时会因为等待超时而报错，示例如下：

    ```sql
    ERROR 3009 (HY000): [11a07e23fd800000][30.225.180.55:8527][polardbx]Exceeding the max concurrency 0 of ccl rule selectrulereal after waiting for 10060 ms
    ```

    
  
  * **KILL（结束）**

    并发度和等待队列长度均已经达到最大值，客户端将收到提示超过最大并发度的报错，报错信息中会包含匹配上的限流规则的名称。

    例如，在并发度和等待队列长度均已经达到最大值后执行`SELECT 1;`命令，会出现如下报错：

    ```sql
    ERROR 3009 (HY000): [11a07c4425c00000][**.***.***.**:8527][polardbx]Exceeding the max concurrency 0 of ccl rule selectrulereal
    ```

    

    上述结果表示：该SQL语句`SELECT 1;`由于超出了限流规则`selectrulereal`设置的最大并发度而执行失败。
    
  

  

  

* **示例**

  假设需要创建一条名为`selectrule`的规则，用于限制由`'ccltest'@'%'`用户发起的，包含`cclmatched`关键字的，且对任意表执行SELECT操作的SQL语句，同时将最大并发度设置为10。

  规则创建语句如下：

  ```sql
  CREATE CCL_RULE IF NOT EXISTS `selectrule` ON *.* TO 'ccltest'@'%'
  FOR SELECT
  FILTER BY KEYWORD('cclmatched')
  WITH MAX_CONCURRENCY=10;
  ```

  




查看限流规则 
---------------------------

* **语法**
  * **查看指定限流规则**

    语法如下：

    ```sql
    SHOW CCL_RULE `ccl_rule_name1` [, `ccl_rule_name2` ]
    ```

    
  
  * **查看所有限流规则**

    语法如下：

    ```sql
    SHOW CCL_RULES
    ```

    
  

  

* **示例**

  使用如下命令查看当前数据库下所有的限流规则：

  ```sql
  mysql> SHOW CCL_RULES \G
  ```

  

  返回结果如下：

  ```sql
  *************************** 1. row ***************************
                       NO.: 1
                 RULE_NAME: selectrulereal
                   RUNNING: 2
                   WAITING: 29
                    KILLED: 0
           MATCH_HIT_CACHE: 21374
               TOTAL_MATCH: 21406
         ACTIVE_NODE_COUNT: 2
  MAX_CONCURRENCY_PER_NODE: 1
  WAIT_QUEUE_SIZE_PER_NODE: 100
              WAIT_TIMEOUT: 600
                FAST_MATCH: 1
                  SQL_TYPE: SELECT
                      USER: ccltest@%
                     TABLE: *.*
                  KEYWORDS: ["SELECT"]
                TEMPLATEID: NULL
              CREATED_TIME: 2020-11-26 17:04:08
  ```

  

  |            参数            |                            说明                             |
  |--------------------------|-----------------------------------------------------------|
  | NO.                      | 匹配优先级，数字越小，优先级越高。                                         |
  | RULE_NAME                | 限流规则名称。                                                   |
  | RUNNING                  | 匹配到该限流规则且正常执行的SQL语句数量。                                    |
  | WAITING                  | 匹配到该限流规则且正在等待队列里的查询数量。                                    |
  | KILLED                   | 匹配到该限流规则且被KILL的SQL语句数量。                                   |
  | MATCH_HIT_CACHE          | 匹配到该限流规则且命中Cache的SQL语句数量。                                 |
  | TOTAL_MATCH              | 匹配到该限流规则的总次数。                                             |
  | ACTIVE_NODE_COUNT        | 计算层中启用了SQL限流的节点数。                                         |
  | MAX_CONCURRENCY_PER_NODE | 每个计算节点的并发度。                                               |
  | WAIT_QUEUE_SIZE_PER_NODE | 每个计算节点上等待队列的最大长度。                                         |
  | WAIT_TIMEOUT             | SQL语句在等待队列的最大等待时间。                                        |
  | FAST_MATCH               | 是否启动缓存加速匹配速度。                                             |
  | SQL_TYPE                 | SQL语句类型。                                                  |
  | USER                     | 用户名。                                                      |
  | TABLE                    | 数据库表。                                                     |
  | KEYWORDS                 | 关键词列表。                                                    |
  | TEMPLATEID               | SQL模版的编号。                                                 |
  | CREATED_TIME             | 创建时间（本地时间），格式为`yyyy-MM-dd HH:mm:ss`。 |
  [参数说明]

  

  




删除限流规则 
---------------------------

**说明** 被删除的限流规则会立即失效，此时该规则下等待队列中的SQL语句全部会被正常执行。

* 删除指定限流规则：

  ```sql
  DROP CCL_RULE [ IF EXISTS ] `ccl_rule_name1` [, `ccl_rule_name2`, ...]
  ```

  

* 删除所有限流规则：

  ```sql
  CLEAR CCL_RULES
  ```

  




慢SQL限流 
---------------------------

* 一键开启按语句类型分，默认为SELECT类型，相同语句类型的命令，有更新作用。语法结构如下：

  ```sql
  SLOW_SQL_CCL GO [ SQL_TYPE [MAX_CONCURRENCY] [SLOW_SQL_TIME] [MAX_CCL_RULE]]
  ```

  
  * SQL_TYPE取值：ALL，SELECT，UPDATE，INSERT，默认为SELECT。
  
  * MAX_CONCURRENCY默认值为CPU核数的一半。
  
  * SLOW_SQL_TIME默认值为系统参数SLOW_SQL_TIME的值。
  
  * MAX_CCL_RULE的默认值为1000。
  

  动作：
  <!-- -->

  * 遍历整个实例的session，识别出该语句类型慢SQL的TemlateId。
  
  * 创建针对慢SQL的限流触发器，名称为：_SYSTEM_SLOW_SQL_CCL_TRIGGER_{SQL_TYPE}_。
  
  * 传递慢SQL的TemplateId给限流触发器，由限流触发器创建限流规则。
  
  * Kill所有该语句类型的慢TemplateId查询。
  

  

* 一键关闭删除由SLOW_SQL_CCL创建的限流触发器，连带着会删除由限流触发器创建的限流规则。语法结构如下：

  ```sql
  SLOW_SQL_CCL BACK
  ```

  

* 查看限流情况。语法结构如下：

  ```sql
  SLOW_SQL_CCL SHOW
  ```

  plan_cache和ccl_rules里以模版ID作为join key的一次inner join。

* ![查看限流情况](../images/p333269.png)

* 如何干预慢SQL的阈值？
  * 设置SLOW_SQL_CCL GO中的语法。
  
  * 在一键开启SQL限流之前，设置用户变量slow_sql_time。如下：

    ```sql
    set @slow_sql_time=2000;
    slow_sql_ccl go;
    ```
  
  

  
  **说明** 后面的设置方式会被前面的设置方式所覆盖。
  



