# 常见问题解答：索引[¶](https://docs.mongodb.com/manual/faq/indexes/#faq-indexes)


在本页面

- [如何创建索引？](https://docs.mongodb.com/manual/faq/indexes/#how-do-i-create-an-index)
- [索引构建如何影响数据库性能？](https://docs.mongodb.com/manual/faq/indexes/#how-does-an-index-build-affect-database-performance)
- [如何查看集合中存在哪些索引？](https://docs.mongodb.com/manual/faq/indexes/#how-do-i-see-what-indexes-exist-on-a-collection)
- [如何查看查询是否使用索引？](https://docs.mongodb.com/manual/faq/indexes/#how-can-i-see-if-a-query-uses-an-index)
- [如何确定要索引的字段？](https://docs.mongodb.com/manual/faq/indexes/#how-do-i-determine-which-fields-to-index)
- [如何查看索引的大小？](https://docs.mongodb.com/manual/faq/indexes/#how-can-i-see-the-size-of-an-index)
- [写操作如何影响索引？](https://docs.mongodb.com/manual/faq/indexes/#how-do-write-operations-affect-indexes)


本文档解决了有关MongoDB [索引的](https://docs.mongodb.com/manual/indexes/)一些常见问题 。有关索引的更多信息，请参见 [Indexes](https://docs.mongodb.com/manual/indexes/)。



## 如何创建索引？[¶](https://docs.mongodb.com/manual/faq/indexes/#how-do-i-create-an-index)


要在集合上创建索引，请使用 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法。创建索引是一种管理性操作。通常，应用程序不应定期调用 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)。


注意

索引构建会影响性能；请参见 [索引构建如何影响数据库性能？](https://docs.mongodb.com/manual/faq/indexes/#faq-index-performance)。管理员应在建立索引之前考虑性能影响。



## 索引构建如何影响数据库性能？[¶](https://docs.mongodb.com/manual/faq/indexes/#how-does-an-index-build-affect-database-performance)


针对已填充集合的MongoDB索引构建，需要针对该集合进行排他性读写锁定。在[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)释放锁定之前需要对集合进行读取或写入锁定的操作必须等待。

**在版本4.2中进行了更改。*


- 对于[功能兼容版本（fcv）](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.2"`，MongoDB使用优化的构建过程，该过程仅在索引构建的开始和结束时都保留排他锁。其余的构建过程将产生交叉的读写操作。
- 对于[功能兼容版本（fcv）](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.0"`，默认的前台索引构建过程将保留整个索引构建的互斥锁。`background`索引在构建过程中*不会*获得排他锁。


有关索引构建过程的更多信息，请参见[填充集合](https://docs.mongodb.com/manual/core/index-creation/#index-operations)上的索引构建 。


基于副本集的索引具有特定的性能考虑因素和风险。有关更多信息，请参见[复制环境中的索引构建](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)。为了最大程度地减少对副本集（包括分片副本集）建立索引的影响，请使用在[副本集上建立索引中](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/)所述的滚动索引[生成](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/)过程。


要返回当前正在运行的索引创建操作的相关信息，请参阅[Active Indexing Operations](https://docs.mongodb.com/manual/reference/method/db.currentOp/#currentop-index-creation)。要在[主数据库](https://docs.mongodb.com/manual/reference/glossary/#term-primary)或独立数据库上终止正在运行的索引创建操作[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)，请使用 [`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/#db.killOp)。部分构建的索引将被删除。


您不能在副本集的辅助成员上终止*复制*索引构建。您必须首先在主数据库上删除索引。二级服务器将复制删除操作，并在索引构建完成*后*删除索引。索引建立和删除之后的所有其他复制将会终止。



## 如何查看集合中存在哪些索引？[¶](https://docs.mongodb.com/manual/faq/indexes/#how-do-i-see-what-indexes-exist-on-a-collection)


要列出集合的索引，请使用 [`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#db.collection.getIndexes)方法。



## 如何查看查询是否使用索引？

要检查MongoDB如何处理查询，请使用 [`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法。

## 如何确定要索引的字段？


许多因素决定要索引的字段，包括 [选择性](https://docs.mongodb.com/manual/tutorial/create-queries-that-ensure-selectivity/#index-selectivity)，对多种[查询](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)的支持 以及[索引的大小](https://docs.mongodb.com/manual/tutorial/ensure-indexes-fit-ram/)。更多信息，请参见 [索引操作注意事项](https://docs.mongodb.com/manual/core/data-model-operations/#data-model-indexes)和 [索引策略](https://docs.mongodb.com/manual/applications/indexes/)。


## 如何查看索引的大小？[¶](https://docs.mongodb.com/manual/faq/indexes/#how-can-i-see-the-size-of-an-index)


[`db.collection.stats()`](https://docs.mongodb.com/manual/reference/method/db.collection.stats/#db.collection.stats)包括一个为集合中的每个索引提供了大小信息的[`indexSizes`](https://docs.mongodb.com/manual/reference/command/collStats/#collStats.indexSizes)文档。


根据其大小，一个索引可能无法放入内存。当您的服务器具有足够的RAM用于索引和其余[工作集](https://docs.mongodb.com/manual/reference/glossary/#term-working-set)时，索引将加载进内存。当索引太大而无法放入RAM时，MongoDB必须从磁盘读取索引，这比从RAM读取要慢得多。


在某些情况下，索引不必*完全*适合RAM。有关详细信息，请参阅[仅在RAM中保存最近使用值的索引](https://docs.mongodb.com/manual/tutorial/ensure-indexes-fit-ram/#indexing-right-handed)。



## 写操作如何影响索引？


写操作可能需要更新索引：

- 如果写入操作修改了索引字段，则MongoDB将更新所有键中包含该字段的索引。

因此，如果您的应用程序是大量写入操作，则索引可能会影响性能。



原文链接：https://docs.mongodb.com/manual/faq/indexes/

译者：钟秋

update：小芒果
