# 复制[¶](https://docs.mongodb.com/manual/replication/#replication)

在本页

- [冗余和数据可用性](https://docs.mongodb.com/manual/replication/#redundancy-and-data-availability) 
- [MongoDB中的复制](https://docs.mongodb.com/manual/replication/#replication-in-mongodb) 
- [异步复制](https://docs.mongodb.com/manual/replication/#asynchronous-replication) 
- [自动故障转移](https://docs.mongodb.com/manual/replication/#automatic-failover) 
- [读操作](https://docs.mongodb.com/manual/replication/#read-operations) 
- [事务](https://docs.mongodb.com/manual/replication/#transactions) 
- [变更流](https://docs.mongodb.com/manual/replication/#change-streams) 
- [附加功能](https://docs.mongodb.com/manual/replication/#additional-features) 


MongoDB中的副本集是一组维护相同数据集合的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)进程。副本集提供了冗余和[高可用性](https://docs.mongodb.com/manual/reference/glossary/#term-high-availability)，并且这是所有生产部署的基础。本节介绍MongoDB中的复制以及副本集的组件和体系结构。本节还提供了副本集常见任务的教程。


## 冗余和数据可用性[¶](https://docs.mongodb.com/manual/replication/#redundancy-and-data-availability)


复制提供了冗余并增加了 [数据可用性](https://docs.mongodb.com/manual/reference/glossary/#term-high-availability)。对于不同数据库服务器上的多个数据副本，复制为防止单台数据库服务器故障提供了一定程度的容错能力。


在某些情况下，复制可以提高读取性能，因为客户端可以将读操作发送到不同的服务器上。在不同的数据中心维护数据副本可以提高分布式应用程序的数据本地化和可用性。您还可以维护额外的副本以实现特殊用途，比如灾难恢复、报告或备份。


## MongoDB的复制[¶](https://docs.mongodb.com/manual/replication/#replication-in-mongodb)


副本集是一组维护相同数据集合的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。副本集包含多个数据承载节点和一个可选的仲裁节点。在数据承载节点中，有且仅有一个成员为主节点，其他节点为从节点。


 [主节点](https://docs.mongodb.com/manual/core/replica-set-primary/) 接收所有的写操作。一个副本集仅有一个主节点能够用[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 写关注点级别来确认写操作；虽然在某些情况下，另一个mongod的实例也可以暂时认为自己是主节点。[[1\]](https://docs.mongodb.com/manual/replication/#edge-cases-2-primaries) 主节点会将其数据集合所有的变化记录到操作日志中，即[oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/).。有关主节点操作的更多信息，请参见 [副本集主节点](https://docs.mongodb.com/manual/core/replica-set-primary/)。

![Diagram of default routing of reads and writes to the primary.](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)


从节点复制主节点的oplog，并将这些操作应用于它们的数据集，这样以便从节点的数据集能反映出主节点的数据集。如果主节点不可用，一个候选的从节点将会发起选举并使之成为新的主节点。有关副本成员的更多信息，请参见[副本成员](https://docs.mongodb.com/manual/core/replica-set-secondary/)。

![Diagram of a 3 member replica set that consists of a primary and two secondaries.](https://docs.mongodb.com/manual/_images/replica-set-primary-with-two-secondaries.bakedsvg.svg)


在某些情况下(比如您有一个主节点和一个从节点，但由于成本约束无法添加另一个从节点)，您可以选择将一个 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例作为 [仲裁节点](https://docs.mongodb.com/manual/core/replica-set-arbiter/)添加到一个副本集中。仲裁节点参与[选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)但不持有数据(即不提供数据冗余)。有关仲裁节点的更多信息，请参见[副本集仲裁节点](https://docs.mongodb.com/manual/core/replica-set-arbiter/)。

![Diagram of a replica set that consists of a primary, a secondary, and an arbiter.](https://docs.mongodb.com/manual/_images/replica-set-primary-with-secondary-and-arbiter.bakedsvg.svg)


[仲裁节点](https://docs.mongodb.com/manual/core/replica-set-arbiter/) 永远只能是仲裁节点，但在选举过程中[主节点](https://docs.mongodb.com/manual/core/replica-set-primary/)也许会降级成为 [从节点](https://docs.mongodb.com/manual/core/replica-set-secondary/)， [从节点](https://docs.mongodb.com/manual/core/replica-set-secondary/)也可能会升级成为主节点。



## 异步复制[¶](https://docs.mongodb.com/manual/replication/#asynchronous-replication)



从节点复制主节点的oplog并异步地应用操作到它们的数据集。通过让从节点的数据集反映主服务器的数据集，副本集可以在一个或多个成员失败的情况下继续运行。


有关复制机制的更多信息，请参见 [副本集Oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/#replica-set-oplog) 和 [副本集数据同步](https://docs.mongodb.com/manual/core/replica-set-sync/#replica-set-sync)。


### 慢操作[¶](https://docs.mongodb.com/manual/replication/#slow-operations)


从4.2版本开始（从4.0.6开始也是可行的），副本集的副本成员会记录oplog中应用时间超过慢操作阈值的慢操作条目。这些慢oplog信息被记录在从节点的[诊断日志](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-logpath) 中，其路径位于[`REPL`](https://docs.mongodb.com/manual/reference/log-messages/#REPL) 组件的文本`applied op: took ms`中。这些慢日志条目仅仅依赖于慢操作阈值。它们不依赖于日志级别（无论是系统还是组件级别）、过滤级别，或者慢操作采样比例。过滤器不会捕获慢日志条目。


### 复制延迟和流控制[¶](https://docs.mongodb.com/manual/replication/#replication-lag-and-flow-control)


[复制延迟](https://docs.mongodb.com/manual/reference/glossary/#term-replication-lag) 指的是将[主节点](https://docs.mongodb.com/manual/core/replica-set-primary/)的写操作拷贝(即复制)到 [从节点](https://docs.mongodb.com/manual/core/replica-set-secondary/)所花费的时间。一些小的延迟期可能是可以接受的，但是随着复制延迟的增长，会出现严重的问题，包括引起主节点的缓存压力。


从MongoDB 4.2开始，管理员可以限制主节点应用写操作的速度，目的是将[`majority committed`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime) 延迟保持在可配置参数[`flowControlTargetLagSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.flowControlTargetLagSeconds)的最大值之下。


默认情况下，流控制是[启用的](https://docs.mongodb.com/manual/reference/parameters/#param.enableFlowControl)。



注意


为了进行流控制，复制集/分片集群必须满足：参数[featureCompatibilityVersion (FCV)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) 设置为`4.2`并启用[majority](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern)读关注点。也就是说，如果FCV不是 `4.2` ，或者读关注点majority被禁用，那么启用流控制将不起作用。


启用流控制后，当延迟快接近 [`flowControlTargetLagSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.flowControlTargetLagSeconds) 参数指定的秒数时，主节点上的写操作必须首先获得许可单才可以获取写锁。通过限制每秒发出的许可单的数量，流控制机制可以将延迟保持在目标数值之下。


为获取更多信息，请参见检查[复制延迟](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#replica-set-replication-lag)和[流控制](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#flow-control)。



### 自动故障转移[¶](https://docs.mongodb.com/manual/replication/#automatic-failover)



当主节点无法和集群中其他节点通信的时间超过参数`electionTimeoutMillis`配置的期限时（默认10s），一个候选的从节点会发起选举来推荐自己成为新主节点。集群会尝试完成一次新主节点的选举并恢复正常的操作。

![Diagram of an election of a new primary. In a three member replica set with two secondaries, the primary becomes unreachable. The loss of a primary triggers an election where one of the secondaries becomes the new primary](https://docs.mongodb.com/manual/_images/replica-set-trigger-election.bakedsvg.svg)


副本集在选举成功前是无法处理写操作的。如果读请求被配置运行在[从节点](https://docs.mongodb.com/manual/core/read-preference/#replica-set-read-preference) 上，则当主节点下线时，副本集可以继续处理这些请求。


假设采用默认的[副本配置选项](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings)，集群选择新主节点的中间过渡时间通常不应超过12秒。这包括了将主节点标记为[unavailable](https://docs.mongodb.com/manual/replication/#replication-auto-failover)、发起以及完成一次 [选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)的时间。您可以通过修改[`settings.electionTimeoutMillis`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.electionTimeoutMillis) 复制配置选项来调整这个时间期限。网络延迟等因素可能会延长完成副本集选举所需的时间，从而影响您的集群在没有主节点的情况下运行的时间。这些因素取决于您实际的集群架构情况。


将[`electionTimeoutMillis`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.electionTimeoutMillis)复制配置选项从默认的`10000`(10秒)降低可以更快地检测主节点故障。然而，由于诸如临时性的网络延迟等因素，集群可能会更频繁地发起选举，即使主节点在其他方面是健康的。这也许会增加[w : 1](https://docs.mongodb.com/manual/reference/write-concern/#wc-w) 级别写操作发生[回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#replica-set-rollback)的可能性。


您的应用程序连接逻辑应该包括对自动故障转移和后续选举的容错处理能力。从MongoDB 3.6开始，MongoDB驱动程序可以探测到主节点的丢失，并自动[重试某些写操作](https://docs.mongodb.com/manual/core/retryable-writes/#retryable-writes) 一次，提供额外的自动故障转移和选举的内置处理:

- MongoDB 4.2兼容的驱动程序默认启用可重试写
- MongoDB 4.0和3.6兼容的驱动程序必须通过在 [连接字符串](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-uri)中包含[`retryWrites=true`](https://docs.mongodb.com/manual/reference/connection-string/#urioption.retryWrites)来显式地启用可重试写。

请参见 [副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)来获取副本集选举的完整信息。


为了解更多关于MongoDB失败处理的信息，请参见：

- [副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections) 
- [可重试写](https://docs.mongodb.com/manual/core/retryable-writes/#retryable-writes) 
- [副本集故障期间的回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#replica-set-rollback) 


## 读操作[¶](https://docs.mongodb.com/manual/replication/#read-operations)

### 读偏好[¶](https://docs.mongodb.com/manual/replication/#read-preference)


默认情况下，客户端从主节点读取[[1\]](https://docs.mongodb.com/manual/replication/#edge-cases-2-primaries)；然而，客户端可以定义一个[读偏好](https://docs.mongodb.com/manual/core/read-preference/) 将读操作发送给从节点。

![Diagram of an application that uses read preference secondary.](https://docs.mongodb.com/manual/_images/replica-set-read-preference-secondary.bakedsvg.svg)


[异步复制](https://docs.mongodb.com/manual/replication/#asynchronous-replication)至从节点，意味着从节点读取返回的数据不能反映主节点上数据的状态。


包含读操作的[多文档事务](https://docs.mongodb.com/manual/core/transactions/)必须使用读偏好[`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary)。在给定的事务中所有操作都必须路由至相同的成员节点。


为了解更多关于副本集读的信息，请参见[读偏好](https://docs.mongodb.com/manual/core/read-preference/)。


### 数据可见性[¶](https://docs.mongodb.com/manual/replication/#data-visibility)


根据读关注，客户端可以在写[持久化](https://docs.mongodb.com/manual/reference/glossary/#term-durable)前看到写结果：

- 不管写的 [write concern](https://docs.mongodb.com/manual/reference/write-concern/)级别是什么，其他使用了读关注级别为 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") 或 [`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern."available") 的客户端，可以在发起写操作的客户端确认其写成功之前查看该客户端写的结果。
- 使用了读关注级别为 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") 或 [`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern."available") 的客户端，能读取在副本集故障转移期间可能随后被[回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/) 掉的数据。


对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)中的操作，当事务提交时，在事务中所做的所有数据更改都会被保存并在事务外部可见。也就是说，事务在回滚其他更改时不会提交某些更改。


在事务提交之前，事务中所做的数据更改在事务外部是不可见的。


然而，当一个事务写入多个分片时，并不是所有外部的读操作都需要等待提交的事务的结果在分片中可见。例如，如果提交了一个事务，并且在分片a上可以看到写1，但是在分片B上还不能看到写2，那么外部读关注为 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") 的读可以在不看到写2的情况下读取写1的结果。


有关MongoDB读隔离、一致性和近因性的更多信息，请参见[Read Isolation, Consistency, and Recency](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/)。


## 事务[¶](https://docs.mongodb.com/manual/replication/#transactions)


从MongoDB 4.0开始，副本集支持[多文档事务](https://docs.mongodb.com/manual/core/transactions/)。


包含读操作的[多文档事务](https://docs.mongodb.com/manual/core/transactions/)必须使用读偏好 [`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary)。给定事务中所有的操作都必须路由至相同的成员节点。


在事务提交之前，事务中所做的数据更改在事务外部是不可见的。


然而，当一个事务写入多个分片时，并不是所有外部的读操作都需要等待提交的事务的结果在分片中可见。例如，如果提交了一个事务，并且在分片a上可以看到写1，但是在分片B上还不能看到写2，那么外部读关注点为 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") 的读可以在不看到写2的情况下读取写1的结果。


## 变更流[¶](https://docs.mongodb.com/manual/replication/#change-streams)


从MongoDB 3.6开始，副本集和分片集群支持[变更流](https://docs.mongodb.com/manual/changeStreams/)。变更流允许应用程序访问实时数据更改，而不需要跟踪oplog的复杂性和风险。应用程序可以使用变更流来订阅一个或多个集合上的所有数据更改。


## 附加功能[¶](https://docs.mongodb.com/manual/replication/#additional-features)


副本集提供了许多选项来支持应用程序的需求。例如，你可以使用[多数据中心中的成员](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/)来部署一个副本集，或者通过调整一些成员的[`members[n].priority`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].priority) 来控制选举结果。副本集还支持用于报告、灾难恢复或备份功能的专用成员。


更多有关信息请参见[优先级0的副本集成员](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members)，[隐藏副本集成员](https://docs.mongodb.com/manual/core/replica-set-hidden-member/#replica-set-hidden-members)和[延迟副本集成员](https://docs.mongodb.com/manual/core/replica-set-delayed-member/#replica-set-delayed-members) 。


| [1]  | *([1](https://docs.mongodb.com/manual/replication/#id2), [2](https://docs.mongodb.com/manual/replication/#id4))* 在 [某些场景下](https://docs.mongodb.com/manual/core/read-preference-use-cases/#edge-cases), 一个复制集中的两个节点可能会认为它们是主节点，但最多，他们中的一个将能够完成写关注点为[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写操作。 可以完成 [`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 写的节点是当前主节点，而另一个节点是原先的主节点，通常是由于[网络分区](https://docs.mongodb.com/manual/reference/glossary/#term-network-partition)导致它还没有意识到自己的降级。当这种情况发生时，连接到原先主节点的客户端尽管已经请求了读偏好[`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary)，但可能还 会观察到过时的数据，并且对原先主节点新写的操作最终将回滚掉。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



原文链接：[https://docs.mongodb.com/manual/replication/](https://docs.mongodb.com/manual/replication/)

译者：李正洋
