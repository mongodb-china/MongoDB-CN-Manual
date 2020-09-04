# 分布式查询

**在本页面**

- [读取复制集的操作](#读取)
- [在复制集上进行写操作](#复制集写)
- [读取分片群集的操作](#读分片)
- [在分片群集上写操作](#分片写)


## <span id="读取">读取复制集的操作</span>

默认情况下，客户端读取复制集的[主](https://docs.mongodb.com/master/reference/glossary/#term-primary)副本;但是，客户端可以指定一个[读首选项](https://docs.mongodb.com/master/core/read-preference/) ，以便对其他成员进行直接读操作。例如，客户端可以配置读取偏好，从二级或从最近的成员读取到:

*   减少多数据中心部署中的延迟，
*   通过分配高读取量（相对于写入量）来提高读取吞吐量，
*   执行备份操作，和/或
*   允许读取直到选择一个[新的主节点](https://docs.mongodb.com/manual/core/replica-set-high-availability/#replica-set-failover)。

![将操作读取到副本集。 默认读取首选项将读取路由到主数据库。 ``最近''的读取首选项会将读取路由到最近的成员。](https://docs.mongodb.com/manual/_images/replica-set-read-preference.bakedsvg.svg)

来自复制集的次要成员的读取操作可能无法反映主要数据库的当前状态。将读取操作定向到不同服务器的读取首选项可能会导致非单调读取。

*在3.6版中进行了更改：*从MongoDB 3.6开始，客户端可以使用[因果一致的](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)会话，这提供了各种保证，包括单调读取。

您可以基于每个连接或每个操作配置读取首选项。有关读取首选项或读取首选项模式的更多信息，请参见[读取首选项](https://docs.mongodb.com/manual/core/read-preference/)和 [读取首选项模式](https://docs.mongodb.com/manual/core/read-preference/#replica-set-read-preference-modes)。


## <span id="复制集写">在复制集上进行写操作</span>

在[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set),中，所有的写操作都指向集合的[主](https://docs.mongodb.com/master/reference/glossary/#term-primary)节点。主服务器应用写操作并将操作记录在主服务器的操作日志或[oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)上。oplog是对数据集的可重复操作序列。集合中的次要成员不断复制oplog，并在一个异步进程中将这些操作应用到自己身上。

![Diagram of default routing of reads and writes to the primary.](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)

有关复制集和写入操作的更多信息，请参见[复制](https://docs.mongodb.com/manual/replication/)和 [写入问题](https://docs.mongodb.com/manual/reference/write-concern/)。


## <span id="读分片">读取分片群集的操作</span>

[分片集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)允许您以一种对应用程序几乎透明的方式在[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例集群之间划分数据集。有关分片集群的概述，请参阅本手册的[分片](https://docs.mongodb.com/manual/sharding/)部分。

对于分片群集，应用程序向[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)与该群集关联的实例之一发出操作 。

![分片群集的示意图。](https://docs.mongodb.com/manual/_images/sharded-cluster.bakedsvg.svg)

当分片群集上的读取操作定向到特定分片时，效率最高。分片集合的查询应包含集合的分片[键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key)。当查询包含分片键时，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以使用[配置数据库中的](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/#sharding-config-server)群集元数据将查询路由到分片。

![将操作读取到分片群集。 查询条件包括分片键。 查询路由器``mongos''可以将查询定位到适当的一个或多个分片。](https://docs.mongodb.com/manual/_images/sharded-cluster-targeted-query.bakedsvg.svg)

如果查询不包含分片键，则[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)必须将查询定向到集群中的*所有分*片。这些*分散的收集*查询可能效率很低。在较大的群集上，分散收集查询对于常规操作是不可行的。

![将操作读取到分片群集。 查询条件不包含分片键。 查询路由器``mongos''必须向所有分片广播查询以进行收集。](https://docs.mongodb.com/manual/_images/sharded-cluster-scatter-gather-query.bakedsvg.svg)

对于复制集分片，从复制集的辅助成员进行的读取操作可能无法反映主副本的当前状态。将读取操作定向到不同服务器的读取首选项可能会导致非单调读取。

> **[success] 注意**
>
> 从MongoDB 3.6开始，
>
> * 客户端可以使用[因果一致的](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency) 会话，从而提供各种保证，包括单调读取。
>
> * 分片复制集的所有成员(不仅是主节点)都维护有关块元数据的元数据。如果不使用读取关注点，这将防止从辅助节点读取返回[孤立的数据](https://docs.mongodb.com/manual/reference/glossary/#term-orphaned-document)[`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern."available")。在较早的版本中，无论是否关注读操作，从辅助对象进行的读操作都可能返回孤立的文档。

有关分片群集中读取操作的更多信息，请参见 [mongos](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/)和[Shard Keys](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key) 部分。


## <span id="分片写">在分片群集上写操作</span>

对于[分片群集](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster)中的分片集合，该 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)指令将写操作从应用程序定向到负责数据集特定部分的分片。在[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)使用来自集群的元数据 的[配置数据库](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/#sharding-config-server)以路由写操作到适当的分片。

![分片群集的示意图。](https://docs.mongodb.com/manual/_images/sharded-cluster.bakedsvg.svg)

MongoDB根据[分片键](https://docs.mongodb.com/master/reference/glossary/#term-shard-key)的值将分片集合中的数据划分为范围。然后，MongoDB将这些块分配为分片。分片键决定块到分片的分布。这可能会影响集群中的写操作的性能。

![分片键值空间划分成较小范围或块的图。](https://docs.mongodb.com/manual/_images/sharding-range-based.bakedsvg.svg)

> **[warning] 重要**
>
> 影响*单个*文档的 更新操作**必须**包含[分片键](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) 或`_id` 字段。如果具有[分片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)，则影响多个文档的更新在某些情况下会更有效，但可以广播到所有分片。

如果分片键的值在每次插入时增加或减少，则所有插入操作都将针对单个分片。结果，单个分片的容量成为分片簇的插入容量的限制。

欲了解更多信息，请参阅[分片](https://docs.mongodb.com/manual/sharding/)和 [批量写入操作](https://docs.mongodb.com/manual/core/bulk-write-operations/)。

​	也可以看看：

​	[可重试写入](https://docs.mongodb.com/manual/core/retryable-writes/#retryable-writes)



译者：杨帅

校对：杨帅