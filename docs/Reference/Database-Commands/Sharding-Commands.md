# [ ](#)分片命令

[]()

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

| 名称                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| [`addShard`]()            | 添加一个分片到分片集群。                                     |
| [`addShardToZone`]()      | 将分片与`zone`关联。支持在分片群集中配置`zone`。             |
| [`balancerStart`]()       | 启动平衡器线程。                                             |
| [`balancerStatus`]()      | 返回有关平衡器状态的信息。                                   |
| [`balancerStop`]()        | 停止平衡器线程。                                             |
| [`checkShardingIndex`]()  | 验证分片键索引的内部命令。                                   |
| [`clearJumboFlag`]()      | 清除`jumbo`数据块的标志。                                    |
| [`cleanupOrphaned`]()     | 删除分片键值超出分片拥有的数据块范围之外的孤立数据。         |
| [`enableSharding`]()      | 在特定数据库上启用分片。                                     |
| [`flushRouterConfig`]()   | 强制`mongod`/ `mongos`实例更新其缓存的路由元数据。           |
| [`getShardMap`]()         | 报告分片群集状态的内部命令。                                 |
| [`getShardVersion`]()     | 返回配置服务器版本的内部命令。                               |
| [`isdbgrid`]()            | 验证进程是否是`mongos`。                                     |
| [`listShards`]()          | 返回已配置的分片列表。                                       |
| [`medianKey`]()           | 不推荐使用的内部命令。请参阅`splitVector`。                  |
| [`moveChunk`]()           | 在分片之间迁移块的内部命令。                                 |
| [`movePrimary`]()         | 从分片集群中删除分片时，重新分配主分片。                     |
| [`mergeChunks`]()         | 提供在单个分片上组合块的功能。                               |
| [`removeShard`]()         | 启动从分片群集中删除分片的进程。                             |
| [`removeShardFromZone`]() | 删除分片和`zone`之间的关联。支持在分片群集中配置`zone`。     |
| [`setShardVersion`]()     | 内部命令，用于设置配置服务器版本。                           |
| [`shardCollection`]()     | 启用集合的分片功能，从而可以对集合进行分片。                 |
| [`shardingState`]()       | 报告`mongod` 是否为分片群集的成员。                          |
| [`split`]()               | 创建一个新的块。                                             |
| [`splitChunk`]()          | 拆分块的内部命令。而是使用方法`sh.splitFind()`和`sh.splitAt()`。 |
| [`splitVector`]()         | 确定分割点的内部命令。                                       |
| [`unsetSharding`]()       | 影响MongoDB部署中实例之间的连接的内部命令。                  |
| [`updateZoneKeyRange`]()  | 添加或删除范围内的分片数据与`zone`之间的关联。支持在分片群集中配置`zone`。 |



译者：李冠飞

校对：