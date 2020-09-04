# [ ](#)mongo Shell 方法

[]()
> **注意**
>
> 有关特定方法(包括语法和示例)的详细信息，请单击特定方法以转到其 reference 页面。

| 名称                                   | 描述                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| db.collection.aggregate()              | 提供对聚合管道的访问。                                       |
| db.collection.bulkWrite()              | 提供批量写入操作功能。                                       |
| db.collection.copyTo()                 | 已过时。包装EVAL以在单个 MongoDB 实例中的集合之间复制数据。  |
| db.collection.count()                  | 包装计数以_return 计算集合或视图中的文档数。                 |
| db.collection.createIndex()            | 在集合上构建索引。                                           |
| db.collection.createIndexes()          | 在集合上构建一个或多个索引。                                 |
| db.collection.dataSize()               | 返回集合的大小。包装collStats输出中的尺寸字段。              |
| db.collection.deleteOne()              | 删除集合中的单个文档。                                       |
| db.collection.deleteMany()             | 删除集合中的多个文档。                                       |
| db.collection.distinct()               | 返回具有指定字段的不同值的文档的 array。                     |
| db.collection.drop()                   | 从数据库中删除指定的集合。                                   |
| db.collection.dropIndex()              | 删除集合上的指定索引。                                       |
| db.collection.dropIndexes()            | 删除集合上的所有索引。                                       |
| db.collection.ensureIndex()            | 已过时。使用db.collection.createIndex()。                    |
| db.collection.explain()                | 返回有关各种方法的查询执行的信息。                           |
| db.collection.find()                   | 对集合或视图执行查询并返回游标 object。                      |
| db.collection.findAndModify()          | 以原子方式修改并返回单个文档。                               |
| db.collection.findOne()                | 执行查询并返回单个文档。                                     |
| db.collection.findOneAndDelete()       | 查找单个文档并将其删除。                                     |
| db.collection.findOneAndReplace()      | 查找单个文档并替换它。                                       |
| db.collection.findOneAndUpdate()       | 查找单个文档并进行更新。                                     |
| db.collection.getIndexes()             | 返回描述集合上现有索引的文档的 array。                       |
| db.collection.getShardDistribution()   | 对于分片群集中的集合，db.collection.getShardDistribution()报告块分布的数据。 |
| db.collection.getShardVersion()        | 分片 cluster 的内部诊断方法。                                |
| db.collection.group()                  | 已过时。提供简单的数据聚合 function。通过 key 对集合中的文档进行分组，并处理结果。使用aggregate()进行更复杂的数据聚合。 |
| db.collection.insert()                 | 在集合中创建新文档。                                         |
| db.collection.insertOne()              | 在集合中插入新文档。                                         |
| db.collection.insertMany()             | 在集合中插入几个新文档。                                     |
| db.collection.isCapped()               | 报告集合是否为上限集合。                                     |
| db.collection.latencyStats()           | 返回集合的延迟统计信息。                                     |
| db.collection.mapReduce()              | 执行 map-reduce 样式数据聚合。                               |
| db.collection.reIndex()                | 重建集合上的所有现有索引。                                   |
| db.collection.remove()                 | 从集合中删除文档。                                           |
| db.collection.renameCollection()       | 更改集合的 name。                                            |
| db.collection.replaceOne()             | 替换集合中的单个文档。                                       |
| db.collection.save()                   | 在insert()和update()周围提供 wrapper 以插入新文档。          |
| db.collection.stats()                  | 报告集合的 state。在collStats周围提供 wrapper。              |
| db.collection.storageSize()            | 报告集合使用的总大小(以字节为单位)。在collStats输出的storageSize字段周围提供 wrapper。 |
| db.collection.totalIndexSize()         | 报告集合上索引使用的总大小。在collStats输出的totalIndexSize字段周围提供 wrapper。 |
| db.collection.totalSize()              | 报告集合的总大小，包括所有文档的大小和集合上的所有索引。     |
| db.collection.update()                 | 修改集合中的文档。                                           |
| db.collection.updateOne()              | 修改集合中的单个文档。                                       |
| db.collection.updateMany()             | 修改集合中的多个文档。                                       |
| db.collection.watch()                  | 在集合上建立变更流。                                         |
| db.collection.validate()               | 对集合执行诊断操作。                                         |
| db.collection.countDocuments()         | $group包装聚合阶段用$sum表达式，以返回集合或视图中文档数量的计数。 |
| db.collection.estimatedDocumentCount() | 包装count以返回集合或视图中文档的大概数量。                  |



译者：李冠飞

校对：