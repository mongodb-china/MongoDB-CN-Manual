# 副本集部署架构

在本页

- [策略](https://docs.mongodb.com/manual/core/replica-set-architectures/#strategies)
- [副本集命名](https://docs.mongodb.com/manual/core/replica-set-architectures/#replica-set-naming)
- [部署方式](https://docs.mongodb.com/manual/core/replica-set-architectures/#deployment-patterns)

副本集的架构影响副本集的容量和性能。本文档提供了副本集部署的策略，并描述了常见的体系结构。

生产系统的标准副本集部署是一个三成员的副本集。这些副本集提供了冗余和容错能力。应尽可能避免复杂性，但是要让您的应用程序需求来决定架构体系。


## 策略

### 确定成员的数量

依据下面这些策略在副本集中添加成员


#### 最大的投票成员数量

一个副本集至多可以有50个成员，但可投票成员最多只能有7个。如果副本集已经有7个有投票权的成员了，那其他的成员只能作为无投票权成员。


#### 部署奇数个成员


确保副本集有奇数个投票成员。一个副本集最多可以有7个投票成员。如果您有*偶数*个有投票权的成员，则部署另一个有投票权的数据成员，或者如果有约束禁止部署另一个有投票权的数据成员，则可以部署一个仲裁节点。

仲裁节点不存储数据的副本，并且需要的资源更少。因此，您可以在应用程序服务器或其他共享进程上运行仲裁节点。在没有数据副本的情况下，可能会在不放置副本集其他成员的环境中放置仲裁节点。请参考安全策略。

在下列的MongoDB版本，对于带有仲裁节点的副本集，协议版本 `pv1`相较于`pv0` （在MongoDB4.0+中不再支持）增加了 [`w:1`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.) 级别写操作回滚的可能性。

- MongoDB 3.4.1
- MongoDB 3.4.0
- MongoDB 3.2.11或更早期版本

请参[考副本集协议版本](https://docs.mongodb.com/manual/reference/replica-set-protocol-versions/)。


警告

通常来说，应避免在单个副本集上部署多个仲裁节点。


##### 考虑容错性

副本集的容错性是指在有些节点变为不可用之后仍能保留足够数量的成员来完成主节点选举的成员个数。换句话说，它是副本集的成员个数与完成主节点选举所需要的大多数投票成员个数之间的差值。如果没有主节点，副本集就不能接受写操作。容错性受到副本集大小的影响，但两者之间的关系并不直接。见下表:

| Number of Members | Majority Required to Elect a New Primary | Fault Tolerance |
| :---------------- | :--------------------------------------- | :-------------- |
| 3                 | 2                                        | 1               |
| 4                 | 3                                        | 1               |
| 5                 | 3                                        | 2               |
| 6                 | 4                                        | 2               |


向副本集添加一个成员并不总是能提高容错性。但是，在这些情况下，添加的新成员可用作特殊的用途，例如备份或报告。

从4.2.1版本开始，[`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#rs.status) 命令可以为副本集返回[`majorityVoteCount`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.majorityVoteCount) 值。


#### 用隐藏和延迟成员实现特殊用途

添加隐藏或延迟成员来支持特殊的用途，比如备份或者生成报告。


#### 在读压力大的部署上实现负载均衡


在读流量非常大的部署中，可以通过将读流量分发给副本成员来提高读吞吐量。随着部署规模的增长，可以将成员添加或移动到备用数据中心，以提高冗余和可用性。

说明

将副本集的成员分布在双数据中心中会比分布在单数据中心更有利。在双数据中心分布中，

- 如果其中一个数据中心发生故障，数据仍然可以读取，这与单个数据中心分布不同。
- 如果只有少数成员的数据中心发生故障，副本集仍然可以提供写操作和读操作。
- 但是，如果具有大多数成员的数据中心宕机，则副本集会变为只读。

如果可能，将成员分布到至少三个数据中心中。对于配置服务器副本集(CSRS)，最佳实践是在三个(或更多，取决于成员数量)数据中心之间分布成员。如果第三个数据中心的成本过高，一种可能的分布是将数据成员均匀地分布到两个数据中心，并在公司政策允许的情况下将剩余的成员部署在云上。

始终确保主数据中心能够选举出主节点。


#### 在有需求前进行扩容

副本集现有成员必须具有空闲容量来支持添加新成员。总是在当前副本集容量需求饱和之前添加新的成员。

### 按地理位置分布成员


若要在数据中心发生故障时保护数据，请在备用数据中心中至少保留一个成员。如果可能，使用奇数个数据中心，并选择一个成员分布，这样即使在数据中心故障的情况下，也能最大限度地提高剩余副本集成员构成大多数或至少提供数据副本的可能性。

说明

将副本集的成员分布在双数据中心中会比分布在单数据中心更有利。在双数据中心分布中，

- 如果其中一个数据中心发生故障，数据仍然可以读取，这与单个数据中心分布不同。
- 如果只有少数成员的数据中心发生故障，副本集仍然可以提供写操作和读操作。
- 但是，如果具有大多数成员的数据中心宕机，则副本集会变为只读。

如果可能，将成员分布到至少三个数据中心中。对于配置服务器副本集(CSRS)，最佳实践是在三个(或更多，取决于成员数量)数据中心之间分布成员。如果第三个数据中心的成本过高，一种分布的可能性是将数据成员均匀地分布到两个数据中心，并在公司政策允许的情况下将剩余的成员部署在云上。

为了确保主数据中心中的成员先于备用数据中心中的成员当选为主节点，可以将备数据中心成员的参数 [`members[n].priority`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].priority) 值设置低于主数据中心的成员。

更多有关信息，请参考[分布在两个或多个数据中心的副本集](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/)


### 使用标签集进行目标操作


使用副本集标签集将读操作定向到特定成员，或者自定义写关注来确认特定成员的请求。

也可参考[数据中心意识](https://docs.mongodb.com/manual/data-center-awareness/) 和[MongoDB部署的工作负载隔离](https://docs.mongodb.com/manual/core/workload-isolation/)。


### 使用journaling来防止停电


MongoDB默认是启用 [journaling](https://docs.mongodb.com/manual/core/journaling/)。日志可以防止在服务中断（如电源故障和意外重启）时发生数据丢失。


### 主机名

提示

如果可能，使用一个逻辑DNS主机名来替代IP地址，尤其是在配置副本集成员或者分片集群成员时。使用逻辑DNS主机名可以避免因IP地址变化引起配置变更。


## 副本集命名

如果您的应用程序连接了多个副本集，每个副本集应该设置一个唯一的名字。一些驱动程序根据副本集名称对副本集连接进行分组。


## 部署的方式


以下文档描述了常见的副本集部署模式。根据应用程序的需求，其它的模式是可能和有效的。如果需要，在您自己的部署中结合每个架构的特点:

- [三成员副本集](https://docs.mongodb.com/manual/core/replica-set-architecture-three-members/)

  三成员副本集提供了一个副本集的最小推荐架构。

- [分布在两个或多个数据中心的副本集](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/)

  地理分布的集合包括位于多个位置的成员，以防止特定设备的故障，例如停电。



原文链接：https://docs.mongodb.com/manual/core/replica-set-architectures/

译者：李正洋
