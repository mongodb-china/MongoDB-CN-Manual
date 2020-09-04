# 常见问题解答：使用MongoDB分片


在本页面

- [新部署是否适合进行分片？](https://docs.mongodb.com/manual/faq/sharding/#is-sharding-appropriate-for-a-new-deployment)
- [在对集合进行分片后是否可以更改片键？](https://docs.mongodb.com/manual/faq/sharding/#can-i-select-a-different-shard-key-after-sharding-a-collection)
- [为什么文档没有分布在各个分片上？](https://docs.mongodb.com/manual/faq/sharding/#why-are-my-documents-not-distributed-across-the-shards)
- [`mongos`如何检测分片群集配置中的更改？](https://docs.mongodb.com/manual/faq/sharding/#how-does-mongos-detect-changes-in-the-sharded-cluster-configuration)
- [日志中出现的`writebacklisten`是什么意思？](https://docs.mongodb.com/manual/faq/sharding/#what-does-writebacklisten-in-the-log-mean)
- [`mongos`是如何使用连接的？](https://docs.mongodb.com/manual/faq/sharding/#how-does-mongos-use-connections)


本文档回答有关[分片](https://docs.mongodb.com/manual/sharding/)的常见问题。参见手册的[分片](https://docs.mongodb.com/manual/sharding/)章节，它提供了一个 [分片的概述](https://docs.mongodb.com/manual/sharding/)，包括如下细节：

- [片键和选择片键的注意事项](https://docs.mongodb.com/manual/core/sharding-shard-key/)

- [查询路由](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/)

- [高可用性](https://docs.mongodb.com/manual/sharding/#sharding-availability)

- [数据分块](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)和 [数据块迁移过程](https://docs.mongodb.com/manual/core/sharding-balancer-administration/)[数据分区](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)

- [对分片群集进行故障排除](https://docs.mongodb.com/manual/tutorial/troubleshoot-sharded-clusters/)

  

## 新部署是否适合进行分片？[¶](https://docs.mongodb.com/manual/faq/sharding/#is-sharding-appropriate-for-a-new-deployment)


有时适合。但是，如果您的数据集适合放在一台服务器上，则应从非分片的部署开始，因为分片的数据集很小，*几乎没有优势*。



## 在对集合进行分片后是否可以更改片键？


不可以。

MongoDB中没有对集合进行分片后更改片键的自动支持。这一现实情况强调了选择好的[片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key)的重要性。如果 *必须*在对集合进行分片之后更改片键，最佳选择是：

- 将MongoDB中的所有数据转储为外部格式。
- 删除原始分片集合。
- 使用更理想的片键配置分片。
- [预分割片键](https://docs.mongodb.com/manual/tutorial/create-chunks-in-sharded-cluster/)范围，以确保初始均匀分配。
- 将转储的数据恢复到MongoDB中。


尽管您不能为分片集合选择其他片键，但是从MongoDB 4.2开始，您可以更新文档的片键值，除非分片键字段是不可变`_id`字段。有关更新片键值的详细信息，请参阅“ [更改文档的片键值”](https://docs.mongodb.com/manual/core/sharding-shard-key/#update-shard-key)。


在MongoDB 4.2之前，文档的片键字段值是不可变的。

参见[片键](https://docs.mongodb.com/manual/core/sharding-shard-key/)



## 为什么文档没有分布在各个分片上？


一旦数据块的分布达到特定阈值，均衡器就开始在各个分片之间迁移均衡数据。请参阅 [迁移阈值](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#sharding-migration-thresholds)。

此外，如果块中的文档数超过一定数量，MongoDB将无法移动块。请参阅 [每个要迁移的块的最大文档数](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#migration-chunk-size-limit)和[不可分割的块](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#jumbo-chunk)。



## `mongos`如何检测分片群集配置中的更改？[¶](https://docs.mongodb.com/manual/faq/sharding/#how-does-mongos-detect-changes-in-the-sharded-cluster-configuration)


[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例维护[配置数据库](https://docs.mongodb.com/manual/reference/glossary/#term-config-database)的缓存，该缓存包含分片[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)的元数据。

[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)通过向分片发出请求并发现其元数据已过期，从而延迟更新其缓存。要强制 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)重新加载其缓存，您可以针对每个[mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)直接运行[`flushRouterConfig`](https://docs.mongodb.com/manual/reference/command/flushRouterConfig/#dbcmd.flushRouterConfig)命令。



## 日志中出现的`writebacklisten`是什么意思？


回写监听器是一个进程，它打开一个长轮询，在迁移[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)或[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)后回写，以确保他们没有将其发送到错误的服务器。如果需要，回写监听器会将写入发送到正确的服务器。

这些消息是分片基础结构的关键部分，不需要引起关注。



## `mongos`是如何使用连接的？


每个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例都维护一个与分片集群成员的连接池。客户端请求一次使用一个连接；即，请求不是多路复用或流水线的。


客户端请求完成后，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)将连接返回到池中。当客户端数量减少时，这些池不会缩小。这可能会导致未使用的[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)占用大量打开的连接。如果[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)不再使用，则可以安全地重新启动进程以关闭现有连接。


要返回与[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)所使用的所有对外连接池相关的聚合统计信息，请将[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)shell 连接 到[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)，然后运行以下 [`connPoolStats`](https://docs.mongodb.com/manual/reference/command/connPoolStats/#dbcmd.connPoolStats)命令：

复制

```
db.adminCommand("connPoolStats");
```


请参阅“ [UNIX ulimit设置”](https://docs.mongodb.com/manual/reference/ulimit/) 文档的“ [系统资源利用率”](https://docs.mongodb.com/manual/reference/ulimit/#system-resource-utilization)部分。



原文链接：https://docs.mongodb.com/manual/faq/sharding/

译者：钟秋

update：小芒果
