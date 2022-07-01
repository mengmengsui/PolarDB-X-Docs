# 在线实验体验

我们在阿里云开发者社区的云起实验室，提供零门槛快速上手实践PolarDB-X的云上实验平台。通过在线实验平台，您可以一键预置实验环境，并参考详细的实验手册指导，快速体验PolarDB-X的安装部署和功能应用，直观感受产品所提供的能力和服务。

目前，您可以通过云起实验室中[跟我学PolarDB-X](https://developer.aliyun.com/adc/scenarioSeries/cf58beaf1d3d4aafb127dfb3f7bfd549?spm=a2c6h.27088027.devcloud-scenarioSeriesList.22.427679a9YE067x)实践系列课，学习相关知识并完成以下实验。

## 初学PolarDB-X

### 实验1：如何一键安装部署PolarDB-X

[**立即体验**](https://developer.aliyun.com/adc/scenario/b299867c133b46688912399149997335?spm=a2c6h.13858378.0.0.2d6c207cWBxcDO)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 2小时              |

**实验简介**：

在体验PolarDB-X水平扩容、分布式事务、全局二级索引、全局Binlog等特性之前，您需要在本地部署PolarDB-X环境。本实验将介绍如何基于一台配置了CentOS 8.5操作系统的ECS实例（云服务器）通过Docker镜像部署PolarDB-X。

通过完成本实验，您将能够：

- 安装Docker

- 部署PolarDB-X

- 体验PolarDB-X的分布式特性

---

### 实验2：如何使用PolarDB-X

[**立即体验**](https://developer.aliyun.com/adc/scenario/6e7827274b004c7b9fad58ecf5404c6c)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

本实验将基于一台配置了CentOS 8.5操作系统的ECS实例（云服务器），以Spring和WordPress官方教程为例，介绍Spring Boot+PolarDB-X和WordPress+PolarDB-X的应用开发过程和方法。

通过完成本实验，您将能够：

- 基于PolarDB-X进行应用开发

---
## PolarDB-X进阶
### 实验1：如何将PolarDB-X与大数据等系统互通

[**立即体验**](https://developer.aliyun.com/adc/scenario/a734d982339845f18baa71d3cd5a4387)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

一款新的数据库必须要解决一个问题：如何避免成为数据孤岛。PolarDB-X提供了完全兼容MySQL Binlog的增量数据订阅能力，即全局Binlog。

本实验将基于一台配置了CentOS 8.5操作系统的ECS实例（云服务器），介绍将PolarDB-X通过Canal与ClickHouse进行互通，搭建一个实时分析系统的过程和方法。

通过完成本实验，您将能够：

- 如何安装PolarDB-X
- 如何部署Canal
- 如何部署ClickHouse
- 搭建基于PolarDB-X、Canal和ClickHouse的实时分析系统

---

### 实验2：如何对PolarDB-X集群做动态扩缩容

[**立即体验**](https://developer.aliyun.com/adc/scenario/413a0f2ada9049a39e249414698385f1)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

PolarDB-X作为一款云原生+分布式数据库系统，弹性扩缩容是PolarDB-X的亮点特性之一。

本实验将基于一台配置了CentOS 8.5操作系统的ECS实例（云服务器），介绍如何使用PolarDB-X Operator安装PolarDB-X，并带您体验PolarDB-X的扩缩容能力。

通过完成本实验，您将能够：

- 使用PolarDB-X Operator安装PolarDB-X
- PolarDB-X集群动态扩缩容

---

### 实验3：如何在PolarDB-X中进行Online DDL

[**立即体验**](https://developer.aliyun.com/adc/scenario/32d8fd0efdb7414cba05f5f4cf88ac05)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

建表、加列、加索引等是数据库系统中最常见的DDL操作，分布式数据库系统中还有新建全局二级索引、改变拆分方式等特有的DDL操作，如何在不影响业务的前提下进行在线DDL操作，是PolarDB-X的核心功能之一。

本实验将基于一台配置了CentOS 8.5操作系统和安装部署了PolarDB-X集群的ECS实例（云服务器），带您体验PolarDB-X的Online DDL能力。

通过完成本实验，您将能够：

- 如何使用Sysbench OLTP场景来模拟业务负载
- 体验PolarDB-X的Online DDL能力

---

### 实验4：如何在PolarDB-X中优化慢SQL

[**立即体验**](https://developer.aliyun.com/adc/scenario/e7cd646ddc4c43479cb51321d6607ff0)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

慢SQL是开发和运维过程中最常见的问题之一。PolarDB-X提供原生的慢SQL分析能力。本实验将围绕这一场景演示如何在 PolarDB-X 中分析和解决慢 SQL。

本实验将基于一台配置了CentOS 8.5操作系统和安装部署了PolarDB-X集群的ECS实例（云服务器），带您体验如何在PolarDB-X中分析和解决慢SQL。

通过完成本实验，您将能够：

- 如何使用Sysbench OLTP场景来模拟业务负载
- 使用SQL限流
- 使用SQL Advisor优化慢SQL

---

### 实验5：使用PolarDB-X+Flink搭建实时数据大屏

[**立即体验**](https://developer.aliyun.com/adc/scenario/884a36afe52d4850821b85f2c1e40691)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

数据大屏在业务展示和决策中扮演重要角色，而背后的实时计算是数据大屏实现的关键技术之一。

本实验将基于一台配置了CentOS 8.5操作系统的ECS实例（云服务器），带您体验如何使用PolarDB-X与Flink搭建一个实时数据链路，模拟阿里巴巴双十一GMV大屏。

通过完成本实验，您将能够：

- 如何安装PolarDB-X
- 如何使用Flink
- 如何搭建基于PolarDB-X与Flink的实时数据大屏

------

### 实验6： 使用PolarDB-X搭建一个高可用系统

[**立即体验**](https://developer.aliyun.com/adc/scenario/70d3ad96a23e4cfeabbd72fb9e729644)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

大型后台系统往往都采用分布式架构，该架构要考虑的核心问题之一是系统高可用如何设计。PolarDB-X在4月1日发布了2.1.0版本，该版本引入的X-Paxos多副本能力使得系统的高可用能力有了质的提升。

本实验基于一台配置了CentOS 8.5操作系统和安装部署了PolarDB-X集群的ECS实例（云服务器），带您体验如何使用PolarDB-X搭建一个高可用系统，通过直接kill容器模拟节点故障，以观察PolarDB-X 的自动恢复情况。

通过完成本实验，您将能够：

- 使用PolarDB-X+Sysbench OLTP搭建高可用系统
- 体验PolarDB-X高可用能力

------

### 实验7：体验PolarDB-X分布式事务和数据分区

[**立即体验**](https://developer.aliyun.com/adc/scenario/49f35884e3b54bb8bf9a004d781ad446)

| 建议实验时长 | 云产品资源使用时长 |
| ------------ | ------------------ |
| 1小时        | 1小时              |

**实验简介**：

PolarDB-X 是一个基于存储计算分离和Shared-Nothing架构的分布式数据库，存储计算分离是指系统中包含计算节点（CN）和数据节点（DN）。计算节点是无状态节点，通过增加计算节点，提供算力的水平扩展能力。数据按照分区键切分为多个分区，保存在不同的数据节点上，通过将分区迁移到新增的数据节点，提供存储容量的水平扩展能力。

本实验基于一台配置了CentOS 8.5操作系统和安装部署了PolarDB-X集群的ECS实例（云服务器），带您体验PolarDB-X分布式事务和数据分区，通过示例展示分布式事务相关特性，以及通过实际操作了解数据分区的实现细节。

通过完成本实验，您将能够：

- 体验PolarDB-X分布式事务
- 体验PolarDB-X数据分区





