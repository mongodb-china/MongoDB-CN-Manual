# 分片命令

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

| 名称 | 描述 |
| :--- | :--- |
| [`addShard`](sharding-commands.md) | 添加一个分片到分片集群。 |
| [`addShardToZone`](sharding-commands.md) | 将分片与`zone`关联。支持在分片群集中配置`zone`。 |
| [`balancerStart`](sharding-commands.md) | 启动平衡器线程。 |
| [`balancerStatus`](sharding-commands.md) | 返回有关平衡器状态的信息。 |
| [`balancerStop`](sharding-commands.md) | 停止平衡器线程。 |
| [`checkShardingIndex`](sharding-commands.md) | 验证分片键索引的内部命令。 |
| [`clearJumboFlag`](sharding-commands.md) | 清除`jumbo`数据块的标志。 |
| [`cleanupOrphaned`](sharding-commands.md) | 删除分片键值超出分片拥有的数据块范围之外的孤立数据。 |
| [`enableSharding`](sharding-commands.md) | 在特定数据库上启用分片。 |
| [`flushRouterConfig`](sharding-commands.md) | 强制`mongod`/ `mongos`实例更新其缓存的路由元数据。 |
| [`getShardMap`](sharding-commands.md) | 报告分片群集状态的内部命令。 |
| [`getShardVersion`](sharding-commands.md) | 返回配置服务器版本的内部命令。 |
| [`isdbgrid`](sharding-commands.md) | 验证进程是否是`mongos`。 |
| [`listShards`](sharding-commands.md) | 返回已配置的分片列表。 |
| [`medianKey`](sharding-commands.md) | 不推荐使用的内部命令。请参阅`splitVector`。 |
| [`moveChunk`](sharding-commands.md) | 在分片之间迁移块的内部命令。 |
| [`movePrimary`](sharding-commands.md) | 从分片集群中删除分片时，重新分配主分片。 |
| [`mergeChunks`](sharding-commands.md) | 提供在单个分片上组合块的功能。 |
| [`removeShard`](sharding-commands.md) | 启动从分片群集中删除分片的进程。 |
| [`removeShardFromZone`](sharding-commands.md) | 删除分片和`zone`之间的关联。支持在分片群集中配置`zone`。 |
| [`setShardVersion`](sharding-commands.md) | 内部命令，用于设置配置服务器版本。 |
| [`shardCollection`](sharding-commands.md) | 启用集合的分片功能，从而可以对集合进行分片。 |
| [`shardingState`](sharding-commands.md) | 报告`mongod` 是否为分片群集的成员。 |
| [`split`](sharding-commands.md) | 创建一个新的块。 |
| [`splitChunk`](sharding-commands.md) | 拆分块的内部命令。而是使用方法`sh.splitFind()`和`sh.splitAt()`。 |
| [`splitVector`](sharding-commands.md) | 确定分割点的内部命令。 |
| [`unsetSharding`](sharding-commands.md) | 影响MongoDB部署中实例之间的连接的内部命令。 |
| [`updateZoneKeyRange`](sharding-commands.md) | 添加或删除范围内的分片数据与`zone`之间的关联。支持在分片群集中配置`zone`。 |

译者：李冠飞

校对：

