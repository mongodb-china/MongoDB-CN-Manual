# 原子性和事务

**在本页面**


*   [原子性](#原子性)
*   [多文档事务](#交易)
*   [并发控制](#控制)

## <span id="原子性">原子性</span>

在MongoDB中，写操作在单个文档级别上是原子的，即使该操作修改了单个文档中嵌入的多个文档。

## <span id="交易">多文档交易</span>

当单个写操作（例如 [`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)）修改多个文档时，对每个文档的修改是原子性的，但整个操作不是原子性的。

当执行多文档写操作时，无论是通过单个写操作还是通过多个写操作，其他操作都可能会交错。

对于需要原子性地读写多个文档（在单个或多个集合中）的情况，MongoDB支持多文档事务：

*   **在版本4.0中**，MongoDB支持复制集上的多文档事务。
*   **在版本4.2中**，MongoDB引入了分布式事务，它增加了对分片群集上多文档事务的支持，并合并了对复制集上多文档事务的现有支持。

有关MongoDB中事务的详细信息，请参阅 [事务](https://docs.mongodb.com/manual/core/transactions/)页面。

>**[warning] 重要**
>
>在大多数情况下，多文档事务比单文档写入带来更大的性能成本，而多文档事务的可用性不应该取代有效的模式设计。对于许多场景， [非规范化数据模型（嵌入式文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)将继续是数据和用例的最佳选择。也就是说，对于许多场景，适当地对数据建模将最小化对多文档事务的需求。
>
>有关其他事务使用注意事项(如运行时限制和oplog大小限制)，请参见 [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)。


## <span id="控制">并发控制</span>

并发控制允许多个应用程序同时运行，而不会导致数据不一致或冲突。

一种方法是在只能具有唯一值的字段上创建[唯一索引](https://docs.mongodb.com/manual/core/index-unique/#index-type-unique)。这样可以防止插入或更新创建重复数据。在多个字段上创建唯一索引以强制字段值的组合具有唯一性。有关用例的示例，请参见[update()和Unique Index](https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-with-unique-indexes)以及[findAndModify()和Unique Index](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#upsert-and-unique-index)。

另一种方法是在查询谓词中为写操作指定字段的期望当前值。

也可以看看：

[阅读隔离度，一致性和近效性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/)



译者：杨帅

校对：杨帅