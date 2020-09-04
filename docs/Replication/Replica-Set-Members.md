# 副本集成员[¶](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-members)


MongoDB 的副本集是一组提供冗余和高可用性的[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 进程。一个副本集的成员有：

- [主节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-primary-member)

  主节点接受所有的写操作。

- [从节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-secondary-members)

  从节点通过复制主节点的操作来维护一个相同的数据集。从节点为特殊用途的配置文件提供了额外的配置项。例如，从节点可配置成[无投票权](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-non-voting-members) 或[0优先级](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members)。


副本集的最小推荐配置是一个包含三个数据承载成员的三成员副本集：一个[主节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-primary-member) 和两个[从节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-secondary-members)。在某些情况下（例如你有一个主节点和一个从节点，但由于成本约束无法添加另一个从节点），你可以选择使用一个[仲裁节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-arbiters)。仲裁节点参与[选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)但不持有数据（即不提供数据冗余）。

一个副本集最多可以有[50个成员](https://docs.mongodb.com/manual/release-notes/3.0/#replica-sets-max-members)，但仅能有7个可投票成员。



## 主节点[¶](https://docs.mongodb.com/manual/core/replica-set-members/#primary)


副本集的主节点是唯一一个可以接受写操作的成员。MongoDB在[主节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-primary-member) 上应用写操作，然后将这些操作记录到主节点的[oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/)中。[从节点](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-secondary-members)成员复制这个日志然后应用到它们的数据集中。


在下图的三成员副本集中，主节点接受所有写操作。然后从节点复制oplog应用至它们的数据集中。

![Diagram of default routing of reads and writes to the primary.](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)


副本集所有的成员都可以接受读操作。但是，默认情况下，应用程序会将其读操作定向至主节点。有关更改默认读行为的详细信息，请参阅[读偏好](https://docs.mongodb.com/manual/core/read-preference/) 。


副本集最多有一个主节点。 [[2\]](https://docs.mongodb.com/manual/core/replica-set-members/#edge-cases-2-primaries) 如果当前主节点不可用，一个选举会抉择出新的主节点。更多详细信息请参见[副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/)。



## 从节点[¶](https://docs.mongodb.com/manual/core/replica-set-members/#secondaries)


一个从节点维护了主节点数据集的一个副本。为了复制数据，从节点通过异步的方式将主节点[oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/) 应用至自己的数据集中。一个副本集可以有一个或多个从节点。


下图的三成员副本集有两个副本成员。从节点复制主节点的oplog并应用到它们的数据集上。

![Diagram of a 3 member replica set that consists of a primary and two secondaries.](https://docs.mongodb.com/manual/_images/replica-set-primary-with-two-secondaries.bakedsvg.svg)


虽然客户端不能将数据写入到从节点，但客户端可以由从节点读取数据。有关客户端如何将读操作直接读入副本集的详细信息，请参阅[读偏好](https://docs.mongodb.com/manual/core/read-preference/) 。


从节点可以成为主节点。如果当前主节点不可用，副本集会发起选举来选择哪个从节点成为新的主节点。

更多详细信息请参见[副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/)。

您可以出于特殊目的来配置从节点成员。您可以配置一个从节点用于:

- 阻止它在选举中成为主节点，适用于将该节点部署在备用数据中心或者充当一个冷备节点。请查考[0优先级副本集成员](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/)。
- 防止应用程序从它读取数据，适用于在该节点上运行需要与正常流量分离的应用程序。请参考[隐藏副本集成员](https://docs.mongodb.com/manual/core/replica-set-hidden-member/)。
- 保持一个运行的“历史”快照，以便在从某些错误(如无意中删除的数据库)恢复时使用。请参考[延迟副本集成员](https://docs.mongodb.com/manual/core/replica-set-delayed-member/)。

 [[1\]](https://docs.mongodb.com/manual/core/replica-set-members/#id2) | **从4.2版本开始（从4.0.6也支持），副本集的副本成员会记录oplog中应用时间超过慢操作阈值的慢操作条目。这些慢oplog信息被记录在从节点的[诊断日志](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-logpath) 中，其路径位于[`REPL`](https://docs.mongodb.com/manual/reference/log-messages/#REPL) 组件的文本`applied op: took ms`中。这些慢日志条目仅仅依赖于慢操作阈值。它们不依赖于日志级别（无论是系统还是组件级别）、过滤级别，或者慢操作采样比例。过滤器不会捕获慢日志条目。** 


## 仲裁节点[¶](https://docs.mongodb.com/manual/core/replica-set-members/#arbiter)


在某些情况下（例如有一个主节点和一个从节点，但由于成本约束无法添加另一个从节点），你可以在副本集中添加一个仲裁节点。仲裁节点没有数据集的副本，并且不能成为主节点。然而，仲裁节点可以参与[主节点选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)。一个仲裁节点只有 `1` 票选举权。


3.6版本的变化：从MongoDB 3.6版本开始，仲裁节点优先级为0。当您升级一个副本集至3.6版本时，如果当前配置中有一个优先级为`1`的仲裁节点，则MongoDB 3.6会将仲裁节点的优先级重新配置为`0`。

重要

不要在同时承载副本集的主节点或副本成员的系统上运行仲裁节点。

需要添加一个仲裁节点，请参考[添加一个仲裁节点至副本集](https://docs.mongodb.com/manual/tutorial/add-replica-set-arbiter/)。

使用仲裁节点时的注意事项，请参考[副本集仲裁节点](https://docs.mongodb.com/manual/core/replica-set-arbiter/)。


[[2\]](https://docs.mongodb.com/manual/core/replica-set-members/#id1) | **在 [某些场景下](https://docs.mongodb.com/manual/core/read-preference-use-cases/#edge-cases), 一个副本集中的两个节点可能会认为它们是主节点，但至多，他们中的一个将能够完成写关心级别为[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")的写操作。 可以完成 [`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 写的节点是当前主节点，而另一个节点是原先的主节点，通常是由于[网络分区](https://docs.mongodb.com/manual/reference/glossary/#term-network-partition)导致它还没有意识到自己的降级。当这种情况发生时，连接到原先主节点的客户端尽管已经请求了读偏好[`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary)，但可能还会观察到过时的数据，并且对原先主节点新写的操作最终将回滚掉。** 

随附副本集主节点、从节点和仲裁节点的参看链接：

https://docs.mongodb.com/manual/core/replica-set-primary/

https://docs.mongodb.com/manual/core/replica-set-secondary/

https://docs.mongodb.com/manual/core/replica-set-arbiter/



原文链接：https://docs.mongodb.com/manual/core/replica-set-members/

译者：李正洋
