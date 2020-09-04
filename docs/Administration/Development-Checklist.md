# [开发检查表](https://docs.mongodb.com/manual/administration/production-checklist-development/#development-checklist )


- [数据持久性](https://docs.mongodb.com/v4.2/administration/production-checklist-development/#data-durability)

- [架构设计](https://docs.mongodb.com/v4.2/administration/production-checklist-development/#schema-design)

- [复制](https://docs.mongodb.com/v4.2/administration/production-checklist-development/#replication)

- [分片](https://docs.mongodb.com/v4.2/administration/production-checklist-development/#sharding)

- [驱动](https://docs.mongodb.com/v4.2/administration/production-checklist-development/#drivers)                                                                                                

下面的清单以及[操作清单](https://docs.mongodb.com/v4.2/administration/production-checklist-operations/)列表提供了一些建议，帮助我们在生产环境下，避免MongoDB部署出现中的问题。



## 数据持久性

- 确保您的副本集包含至少三个带有`w:majority` [写关注](https://docs.mongodb.com/v4.2/reference/write-concern/)的数据承载节点。副本集范围内的数据持久性需要三个数据承载节点。
- 确保所有实例都使用[日志](https://docs.mongodb.com/v4.2/core/journaling/)。



## 架构设计

MongoDB中的数据有一个*动态设计*。[集合](https://docs.mongodb.com/v4.2/reference/glossary/#term-collection)强制执行[文档](https://docs.mongodb.com/v4.2/reference/glossary/#term-document)结构。这有助于迭代开发和多态性。然而，集合通常保存具有高度同质结构的文档。 有关详细信息，请参阅[数据建模概念](https://docs.mongodb.com/v4.2/core/data-models/)。

- 确定支持查询所需的集合集和所需的索引。除了`_id` 索引之外，您必须显式地创建所有索引：MongoDB不会自动创建除`_id`之外的任何索引。                                                                                          
- 确保架构设计支持您的部署类型：如果您计划使用[分片集群](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster)进行水平扩展，请设计您的架构以包含一个强健的片键。片键通过确定MongoDB如何划分数据来影响读写性能。请参见：[片键对集群操作的影响](https://docs.mongodb.com/v4.2/core/sharding-shard-key/)以获取有关片键应具有哪些质量的信息。一旦设置了片键，就不能更改它。
- 请确保您的架构设计不依赖长度不受限制的索引数组。通常，当这种索引数组的元素少于1000个时，可以获得最佳性能。
- 设计架构时请考虑文档大小限制。[BSON文档大小](https://docs.mongodb.com/v4.2/reference/limits/#BSON-Document-Size)限制为每个文档16MB。如果需要更大的文档，请使用[GridFS](https://docs.mongodb.com/v4.2/core/gridfs/)。



## 复制

- 使用奇数个有投票权的成员来确保选举顺利进行。最多可以有7个有投票权的成员。如果您有偶数个投票成员，并且限制条件（如成本）禁止添加另一个辅助成员作为投票成员，则可以添加[仲裁节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-arbiter)以确保票数为奇数。有关对3成员副本集（P-S-a）使用仲裁节点时的其他注意事项，请参阅[副本集仲裁节点](https://docs.mongodb.com/v4.2/core/replica-set-arbiter/)。                                                                                                                                   



**注意**

对于以下MongoDB版本，对于具有仲裁器的副本集，与`pv0`（MongoDB 4.0+中不再支持）相比， `pv1`增加了 [`w:1`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.) 回滚的可能性：

- MongoDB 3.4.1
- MongoDB 3.4.1
- MongoDB 3.4.0
- MongoDB 3.4.0
- MongoDB 3.2.11 or earlier
- MongoDB 3.2.11 或者更早的版本

参见[副本集协议版本](https://docs.mongodb.com/v4.2/reference/replica-set-protocol-versions/)。



- 通过使用[监视工具](https://docs.mongodb.com/v4.2/administration/monitoring/) 和指定适当的[写入机制,](https://docs.mongodb.com/v4.2/reference/write-concern/)，确保您的辅助文件保持最新。
- 不要使用辅助读取来扩展总体读吞吐量。请参阅：[是否可以使用更多副本节点进行扩展](http://askasya.com/post/canreplicashelpscaling)，以了解读取扩展的概述。有关辅助读取的信息，请参阅：[读取偏好](https://docs.mongodb.com/v4.2/core/read-preference/) 。



## 分片

- 确保片键将负载均匀地分配到分片上。请参见：[片键](https://docs.mongodb.com/v4.2/core/sharding-shard-key/)以获取更多信息。

- 对需要根据切片数量进行扩展的工作负载使用[目标操作](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/#sharding-mongos-targeted)。

- **对于MongoDB 3.4和更早版本**，从主节点读取[非目标或广播](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/#sharding-mongos-broadcast)查询，因为这些查询可能对[过时或孤立的数据](http://blog.mongodb.org/post/74730554385/background-indexing-on-secondaries-and-orphaned)敏感。

- **对于MongoDB 3.6和更高版本**，辅助设备不再返回孤立数据，除非使用[可用的](https://docs.mongodb.com/v4.2/reference/read-concern-available/#readconcern."available")读策略（这是与[因果一致会话](https://docs.mongodb.com/v4.2/core/read-isolation-consistency-recency/#sessions)不关联时针对辅助设备读取的默认读取策略）。

  从 MongoDB 3.6 开始，分片副本集的所有成员都维护块元数据，允许它们在不使用[“可用”](https://docs.mongodb.com/v4.2/reference/read-concern-available/#readconcern."available")时过滤出孤立的数据。因此，不使用[“可用”](https://docs.mongodb.com/v4.2/reference/read-concern-available/#readconcern."available")的[非目标或广播](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/#sharding-mongos-broadcast)查询可以安全地在任何成员上运行，并且不会返回孤立的数据。

   ["可用"](https://docs.mongodb.com/v4.2/reference/read-concern-available/#readconcern."available")的读取策略可以从辅助成员返回[孤立文档](https://docs.mongodb.com/v4.2/reference/glossary/#term-orphaned-document)，因为它不检查更新的块元数据。但是如果孤立文档的返回对于应用程序来说无关紧要，那么["可用"](https://docs.mongodb.com/v4.2/reference/read-concern-available/#readconcern."available")的读取策略提供了各种读取关注点中可能的最低延迟读取。

- 在将大数据集插入新的非哈希分片集合时需要[预分割并手动平衡块](https://docs.mongodb.com/v4.2/tutorial/create-chunks-in-sharded-cluster/)。预分割和手动平衡使插入负载能够在分片之间分布，从而提高初始负载的性能。



## 驱动

- 利用连接池。大多数MongoDB驱动程序支持连接池。调整连接池大小以适合您的用例，从典型并发数据库请求数的110-115%开始。
- 请确保您的应用程序在副本集选择期间处理短暂的写入和读取错误。
- 请确保应用程序处理失败的请求，并在适用的情况下重试。驱动程序**不会**自动重试失败的请求。
- 对数据库请求重试使用指数退避逻辑。
- 如果需要限制数据库操作的执行时间。使用 [`cursor.maxTimeMS()`](https://docs.mongodb.com/v4.2/reference/method/cursor.maxTimeMS/#cursor.maxTimeMS)读取和 [wtimeout ](https://docs.mongodb.com/v4.2/reference/write-concern/#wc-wtimeout)写入。





原文链接：https://docs.mongodb.com/v4.2/administration/production-checklist-development/

译者：孔令升

校对：徐扬
