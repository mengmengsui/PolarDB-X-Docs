Replication 
================================



特性 
-----------------------

* 兼容MySQL Binlog文件格式和Dump协议。PolarDB-X的全局Binlog是在DN节点的物理Binlog基础之上生成的，剔除了分布式事务的细节，只保留了单机事务的特性。同时，全局Binlog兼容MySQL Binlog文件格式，在数据订阅方式上也完全兼容MySQL Dump协议，您可以像使用单机MySQL一样来订阅PolarDB-X的事务日志。

* 保证分布式事务的完整性和有序性。全局Binlog不是将物理Binlog简单地汇总到一起，而是通过合并和排序保证了分布式事务的完整性和有序性，从而实现高数据一致性。例如，在转账场景下，基于全局Binlog能力，接入PolarDB-X的下游MySQL，可以在任何时刻查询到一致的余额。

* 提供7x24小时服务能力，运维简单。全局Binlog剔除了PolarDB-X的内部细节（此时您可以将PolarDB-X看作一个单机MySQL）， 来避免实例内部发生的变化对数据订阅链路的影响。PolarDB-X通过一系列的协议和算法来保证全局Binlog的服务能力，确保实例内部发生的各种变更（如HA切换、增删节点、执行Scale Out或分布式DDL等操作）不会影响数据订阅链路的正常工作。




使用限制 
-------------------------

* 暂不支持Gtid（Global Transaction Identifier）模式下的数据订阅方式。

* 仅当事务策略指定为TSO时（即更高强度的一致性保证），才支持对分布式事务的合并。




数据订阅源端支持的SQL语句 
-----------------------------------

**说明** 执行下述SQL语句需要有SUPER或REPLICATION CLIENT权限。权限操作请参见[账号和权限系统](../../maintance/topics/account.md)。

* 查看PolarDB-X全局Binlog文件列表。

  ```sql
  SHOW BINARY LOGS
  ```


* 查看PolarDB-X作为主Master角色的Binlog信息。

  ```sql
  SHOW MASTER STATUS
  ```


* 查看全局Binlog文件中的具体事件信息。

  ```sql
  SHOW BINLOG EVENTS
  [IN 'log_name']
  [FROM pos]
  [LIMIT [offset,] row_count]
  ```



数据订阅目标端支持的SQL语句（WIP）
------------------------------------

如果数据订阅目标端是标准MySQL，目前支持MySQL的Replicate指令。

* 在数据订阅目标端设置需要同步的源端数据源信息。

  ```sql
  CHANGE MASTER TO option [, option] ... [ channel_option ]
  option: {
      MASTER_BIND = 'interface_name'
    | MASTER_HOST = 'host_name'
    | MASTER_USER = 'user_name'
    | MASTER_PASSWORD = 'password'
    | MASTER_PORT = port_num
    | PRIVILEGE_CHECKS_USER = {'account' | NULL}
    | REQUIRE_ROW_FORMAT = {0|1}
    | REQUIRE_TABLE_PRIMARY_KEY_CHECK = {STREAM | ON | OFF}
    | ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS = {OFF | LOCAL | uuid}
    | MASTER_LOG_FILE = 'source_log_name'
    | MASTER_LOG_POS = source_log_pos
    | MASTER_AUTO_POSITION = {0|1}
    | RELAY_LOG_FILE = 'relay_log_name'
    | RELAY_LOG_POS = relay_log_pos
    | MASTER_HEARTBEAT_PERIOD = interval
    | MASTER_CONNECT_RETRY = interval
    | MASTER_RETRY_COUNT = count
    | SOURCE_CONNECTION_AUTO_FAILOVER = {0|1}
    | MASTER_DELAY = interval
    | MASTER_COMPRESSION_ALGORITHMS = 'value'
    | MASTER_ZSTD_COMPRESSION_LEVEL = level
    | MASTER_SSL = {0|1}
    | MASTER_SSL_CA = 'ca_file_name'
    | MASTER_SSL_CAPATH = 'ca_directory_name'
    | MASTER_SSL_CERT = 'cert_file_name'
    | MASTER_SSL_CRL = 'crl_file_name'
    | MASTER_SSL_CRLPATH = 'crl_directory_name'
    | MASTER_SSL_KEY = 'key_file_name'
    | MASTER_SSL_CIPHER = 'cipher_list'
    | MASTER_SSL_VERIFY_SERVER_CERT = {0|1}
    | MASTER_TLS_VERSION = 'protocol_list'
    | MASTER_TLS_CIPHERSUITES = 'ciphersuite_list'
    | MASTER_PUBLIC_KEY_PATH = 'key_file_name'
    | GET_MASTER_PUBLIC_KEY = {0|1}
    | NETWORK_NAMESPACE = 'namespace'
    | IGNORE_SERVER_IDS = (server_id_list)
  }
  channel_option:
      FOR CHANNEL channel
  server_id_list:
      [server_id [, server_id] ... ]
  ```

  

* 开启主备同步

  ```sql
  START {SLAVE | REPLICA}
  ```

  

* 停止主备同步

  ```sql
  STOP {SLAVE | REPLICA}
  ```

  

* 重置主备同步，需要先停止主备同步

  ```sql
  RESET {SLAVE | REPLICA} [ALL] [channel_option]
  channel_option:
      FOR CHANNEL channel
  ```

  
  **说明** 如果目标端是PolarDB-X，目前暂时不支持相关Replicate指令。
  



