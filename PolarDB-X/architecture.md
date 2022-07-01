# 产品架构

![PolarDB-X Architecture](about/images/architecture.png)

PolarDB-X 采用 Shared-nothing 与存储计算分离架构进行设计，系统由4个核心组件组成。

- 计算节点（CN, Compute Node）

计算节点是系统的入口，采用无状态设计，包括 SQL 解析器、优化器、执行器等模块。负责数据分布式路由、计算及动态调度，负责分布式事务 2PC 协调、全局二级索引维护等，同时提供 SQL 限流、三权分立等企业级特性。


- 存储节点（DN, Data Node）

存储节点负责数据的持久化，基于多数派 Paxos 协议提供数据高可靠、强一致保障，同时通过 MVCC 维护分布式事务可见性。


- 元数据服务（GMS, Global Meta Service）

元数据服务负责维护全局强一致的 Table/Schema, Statistics 等系统 Meta 信息，维护账号、权限等安全信息，同时提供全局授时服务（即 TSO）。


- 日志节点（CDC, Change Data Capture）

日志节点提供完全兼容 MySQL Binlog 格式和协议的增量订阅能力，提供兼容 MySQL Replication 协议的主从复制能力。

PolarDB-X 提供通过 K8S Operator 方式管理以上4个组件，同时计算节点与存储节点之间可通过私有协议进行 RPC 通信，这些组件对应的仓库如下在：

| 组件名称                                     | 仓库地址                                                 |
| -------------------------------------------- | -------------------------------------------------------- |
| 计算节点<br />（CN, Compute Node）           | [galaxysql](https://github.com/ApsaraDB/galaxysql)       |
| 元数据服务<br />（GMS, Global Meta Service） | [galaxyengine](https://github.com/ApsaraDB/galaxyengine) |
| 存储节点<br />（DN, Data Node）              | [galaxyengine](https://github.com/ApsaraDB/galaxyengine) |
| 日志节点<br />（CDC, Change Data Capture）   | [galaxycdc](https://github.com/ApsaraDB/galaxycdc)       |
| 私有协议                                     | [galaxyglue](https://github.com/ApsaraDB/galaxyglue)     |
| K8S Operator                                 | [galaxykube](https://github.com/ApsaraDB/galaxykube)     |