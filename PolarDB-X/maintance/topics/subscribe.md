# PolarDB-X CDC数据订阅

PolarDB-X CDC兼容标准Binlog协议，可以把它当做一个单机版的MySQL来使用，现支持Kafka、Flink等主流消息队列、流计算引擎、日志服务订阅。

![Subscribe All](../images/subscribe_all.png)

### 已支持下游订阅的类型

- 数据库：
  - MySQL
  - PolarDB-X
  - 其他支持MySQL协议数据库
- 订阅组件：
  - Canal
  - Debezium
  - MaxWell
- 流计算引擎：
  - Flink
- 消息队列：
  - rocketMQ
  - rabbitMQ
  - Kafka
- 日志服务：
  - Elasticsearch
  - Kibana