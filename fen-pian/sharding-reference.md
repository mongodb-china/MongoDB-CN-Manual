# Sharding Reference 分片参考

On this page

- [Sharding Methods in the `mongo` Shell](https://docs.mongodb.com/v4.2/reference/sharding/#sharding-methods-in-the-mongo-shell)
- [Sharding Database Commands](https://docs.mongodb.com/v4.2/reference/sharding/#sharding-database-commands)
- [Reference Documentation](https://docs.mongodb.com/v4.2/reference/sharding/#reference-documentation)

在本页面

- [mongo Shell的分片方法](Sharding Methods in the mongo Shell)
- [分片数据库命令](https://docs.mongodb.com/v4.2/reference/sharding/#sharding-database-commands)
- [参考文档](https://docs.mongodb.com/v4.2/reference/sharding/#reference-documentation)



## Sharding Methods in the `mongo` Shell mongo Shell中的分片方法

| 名称                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| sh.addShard()            | Adds a [shard](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard) to a sharded cluster. 将分片添加到分片集群中。 |
| sh.addShardTag()         | In MongoDB 3.4, this method aliases to [`sh.addShardToZone()`](https://docs.mongodb.com/v4.2/reference/method/sh.addShardToZone/#sh.addShardToZone). 在MongoDB 3.4中，此方法别名为sh.addShardToZone（）。 |
| sh.addShardToZone()      | Associates a shard to a zone. Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 将分片与区域关联。支持在分片群集中配置区域。 |
| sh.addTagRange()         | In MongoDB 3.4, this method aliases to [`sh.updateZoneKeyRange()`](https://docs.mongodb.com/v4.2/reference/method/sh.updateZoneKeyRange/#sh.updateZoneKeyRange). 在MongoDB 3.4中，此方法别名为sh.updateZoneKeyRange（）。 |
| sh.disableBalancing()    | Disable balancing on a single collection in a sharded database. Does not affect balancing of other collections in a sharded cluster. 在分片数据库中的单个集合上禁用平衡。不会影响分片集群中其他集合的平衡。 |
| sh.enableBalancing()     | Activates the sharded collection balancer process if previously disabled using [`sh.disableBalancing()`](https://docs.mongodb.com/v4.2/reference/method/sh.disableBalancing/#sh.disableBalancing). 如果以前使用[`sh.disableBalancing()`](https://docs.mongodb.com/v4.2/reference/method/sh.disableBalancing/#sh.disableBalancing)禁用了分片集合平衡器进程，则将其激活。 |
| sh.disableAutoSplit()    | Disables auto-splitting for the sharded cluster. 禁用分片集群的自动拆分。 |
| sh.enableAutoSplit()     | Enables auto-splitting for the sharded cluster. 启用分片集群的自动拆分。 |
| sh.enableSharding()      | Enables sharding on a specific database. 在特定数据库上启用分片。 |
| sh.getBalancerHost()     | *Deprecated since MongoDB 3.4* 自MongoDB 3.4起不推荐使用。   |
| sh.getBalancerState()    | Returns a boolean to report if the [balancer](https://docs.mongodb.com/v4.2/reference/glossary/#term-balancer) is currently enabled. 返回一个布尔值以报告当前是否启用了平衡器。 |
| sh.removeTagRange()      | In MongoDB 3.4, this method aliases to [`sh.removeRangeFromZone()`](https://docs.mongodb.com/v4.2/reference/method/sh.removeRangeFromZone/#sh.removeRangeFromZone). 在MongoDB 3.4中，此方法别名为[`sh.removeRangeFromZone()`](https://docs.mongodb.com/v4.2/reference/method/sh.removeRangeFromZone/#sh.removeRangeFromZone)。 |
| sh.removeRangeFromZone() | Removes an association between a range of shard keys and a zone. Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 删除一系列分片键和区域之间的关联。支持在分片集群中配置区域。 |
| sh.help()                | Returns help text for the `sh` methods. 返回sh的帮助文档。   |
| sh.isBalancerRunning()   | Returns a boolean to report if the balancer process is currently migrating chunks. 返回一个布尔值以报告平衡器进程当前是否正在迁移块。 |
| sh.moveChunk()           | Migrates a [chunk](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk) in a [sharded cluster](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster). 迁移分片集群中的块。 |
| sh.removeShardTag()      | In MongoDB 3.4, this method aliases to [`sh.removeShardFromZone()`](https://docs.mongodb.com/v4.2/reference/method/sh.removeShardFromZone/#sh.removeShardFromZone). 在MongoDB 3.4中，此方法别名为sh.removeShardFromZone（）。 |
| sh.removeShardFromZone() | Removes the association between a shard and a zone. Use to manage [zone sharding](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding). 删除分片和区域之间的关联。用于管理区域分片。 |
| sh.setBalancerState()    | Enables or disables the [balancer](https://docs.mongodb.com/v4.2/reference/glossary/#term-balancer) which migrates [chunks](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk) between [shards](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard). 启用或禁用在分片之间迁移块的平衡器。 |
| sh.shardCollection()     | Enables sharding for a collection. 为集合启用分片。          |
| sh.splitAt()             | Divides an existing [chunk](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk) into two chunks using a specific value of the [shard key](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key) as the dividing point. 使用分片键的特定值作为分割点将现有的块分为两个块。 |
| sh.splitFind()           | Divides an existing [chunk](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk) that contains a document matching a query into two approximately equal chunks. 将包含与查询匹配的文档的现有块分为两个大致相等的块。 |
| sh.startBalancer()       | Enables the [balancer](https://docs.mongodb.com/v4.2/reference/glossary/#term-balancer) and waits for balancing to start. 启用平衡器并等待平衡开始。 |
| sh.status()              | Reports on the status of a [sharded cluster](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster), as [`db.printShardingStatus()`](https://docs.mongodb.com/v4.2/reference/method/db.printShardingStatus/#db.printShardingStatus). 报告分片群集的状态，如[`db.printShardingStatus()`](https://docs.mongodb.com/v4.2/reference/method/db.printShardingStatus/#db.printShardingStatus)。 |
| sh.stopBalancer()        | Disables the [balancer](https://docs.mongodb.com/v4.2/reference/glossary/#term-balancer) and waits for any in progress balancing rounds to complete. 禁用平衡器，并等待任何进行中的平衡回合完成。 |
| sh.waitForBalancer()     | Internal. Waits for the balancer state to change. 内部。等待平衡器状态更改。 |
| sh.waitForBalancerOff()  | Internal. Waits until the balancer stops running. 内部。等待直到平衡器停止运行。 |
| sh.waitForPingChange()   | Internal. Waits for a change in ping state from one of the [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) in the sharded cluster. 内部。等待分片群集中的一个mongos的ping状态更改。 |
| sh.updateZoneKeyRange()  | Associates a range of shard keys to a zone. Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 将一系列分片键与区域关联。支持在分片群集中配置区域。 |
| converShardKeyToHashed() | Returns the hashed value for the input. 返回输入的哈希值。   |



## Sharding Database Commands 分片数据库命令

The following database commands support [sharded clusters](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster).

以下数据库命令支持分片群集。

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| addShard            | Adds a [shard](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard) to a [sharded cluster](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster). 添加一个分片到分片集群中。 |
| addShardToZone      | Associates a shard with a [zone](https://docs.mongodb.com/v4.2/reference/glossary/#term-zone). Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 将分片与区域关联。支持在分片群集中配置区域。 |
| balancerStart       | Starts a balancer thread. 启动平衡器线程。                   |
| balancerStatus      | Returns information on the balancer status. 返回有关平衡器状态的信息。 |
| balancerStop        | Stops the balancer thread. 停止平衡器线程。                  |
| checkShardingIndex  | Internal command that validates index on shard key. 验证分片键索引的内部命令。 |
| clearJumboFlag      | Clears the `jumbo` flag for a chunk. 清除块的巨型标志。      |
| cleanupOrphaned     | Removes orphaned data with shard key values outside of the ranges of the chunks owned by a shard. 删除分片键值超出分片所拥有的块范围之外的孤立数据。 |
| enableSharding      | Enables sharding on a specific database. 在特定数据库上启用分片。 |
| flushRouterConfig   | Forces a [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance to update its cached routing metadata. 强制mongod / mongos实例更新其缓存的路由元数据。 |
| getShardMap         | Internal command that reports on the state of a sharded cluster. 报告分片群集状态的内部命令。 |
| getShardVersion     | Internal command that returns the [config server](https://docs.mongodb.com/v4.2/reference/glossary/#term-config-database) version. 返回配置服务器版本的内部命令。 |
| isdbgrid            | Verifies that a process is a [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos). 验证一个进程是mongos。 |
| listShards          | Returns a list of configured shards.返回已配置分片的列表。   |
| medianKey           | Deprecated internal command. See [`splitVector`](https://docs.mongodb.com/v4.2/reference/command/splitVector/#dbcmd.splitVector). 不推荐使用的内部命令。请参见[`splitVector`](https://docs.mongodb.com/v4.2/reference/command/splitVector/#dbcmd.splitVector)。 |
| moveChunk           | Internal command that migrates chunks between shards. 在分片之间迁移块的内部命令。 |
| movePrimary         | Reassigns the [primary shard](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary-shard) when removing a shard from a sharded cluster. 从分片群集中删除分片时，重新分配主分片。 |
| mergeChunks         | Provides the ability to combine chunks on a single shard. 提供在单个分片上合并块的功能。 |
| removeShard         | Starts the process of removing a shard from a sharded cluster. 开始从分片群集中删除分片的过程。 |
| removeShardFromZone | Removes the association between a shard and a [zone](https://docs.mongodb.com/v4.2/reference/glossary/#term-zone). Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 删除分片和[区域](https://docs.mongodb.com/v4.2/reference/glossary/#term-zone)之间的关联。支持在分片群集中配置区域。 |
| setShardVersion     | Internal command to sets the [config server](https://docs.mongodb.com/v4.2/reference/glossary/#term-config-database) version. 内部命令，用于设置配置服务器版本。 |
| shardCollection     | Enables the sharding functionality for a collection, allowing the collection to be sharded. 启用集合的分片功能，从而可以对集合进行分片。 |
| shardingState       | Reports whether the [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) is a member of a sharded cluster. 报告mongod是否为分片集群的成员。 |
| split               | Creates a new [chunk](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk). 创建一个新的块。 |
| splitChunk          | Internal command to split chunk. Instead use the methods [`sh.splitFind()`](https://docs.mongodb.com/v4.2/reference/method/sh.splitFind/#sh.splitFind) and [`sh.splitAt()`](https://docs.mongodb.com/v4.2/reference/method/sh.splitAt/#sh.splitAt). 拆分块的内部命令。而是使用方法sh.splitFind（）和sh.splitAt（）。 |
| splitVector         | Internal command that determines split points. 确定分割点的内部命令。 |
| unsetSharding       | Internal command that affects connections between instances in a MongoDB deployment. 影响MongoDB部署中实例之间的连接的内部命令。 |
| updateZoneKeyRange  | Adds or removes the association between a range of sharded data and a [zone](https://docs.mongodb.com/v4.2/reference/glossary/#term-zone). Supports configuring [zones](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding) in sharded clusters. 添加或删除范围内的分片数据与区域之间的关联。支持在分片群集中配置区域。 |



## Reference Documentation 参考文档

[Operational Restrictions](https://docs.mongodb.com/v4.2/core/sharded-cluster-requirements/)

Requirement for deploying a sharded cluster

[Troubleshoot Sharded Clusters](https://docs.mongodb.com/v4.2/tutorial/troubleshoot-sharded-clusters/)

Common strategies for troubleshooting sharded cluster deployments.

[Config Database](https://docs.mongodb.com/v4.2/reference/config-database/)

Complete documentation of the content of the `local` database that MongoDB uses to store sharded cluster metadata.

**[操作限制](https://docs.mongodb.com/v4.2/core/sharded-cluster-requirements/)**

​	部署分片集群的要求

**[对分片群集进行故障排除](https://docs.mongodb.com/v4.2/tutorial/troubleshoot-sharded-clusters/)**

​	解决分片群集部署的常见策略。

**[配置数据库](https://docs.mongodb.com/v4.2/reference/config-database/)**

​	MongoDB用于存储分片群集元数据的本地数据库内容的完整文档。



原文链接：https://docs.mongodb.com/v4.2/reference/sharding/

译者：张建威

