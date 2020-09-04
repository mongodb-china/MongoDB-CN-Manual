# 常见问题解答：复制和副本集[¶](https://docs.mongodb.com/manual/faq/replica-sets/#faq-replication-and-replica-sets)


在本页面

- [MongoDB支持哪种复制？](https://docs.mongodb.com/manual/faq/replica-sets/#what-kind-of-replication-does-mongodb-support)
- [复制是否可以通过Internet和WAN连接进行？](https://docs.mongodb.com/manual/faq/replica-sets/#does-replication-work-over-the-internet-and-wan-connections)
- [MongoDB可以通过“noisy”连接进行复制吗？](https://docs.mongodb.com/manual/faq/replica-sets/#can-mongodb-replicate-over-a-noisy-connection)
- [如果复制已经提供了数据冗余，为什么还要使用journaling(预写日志，WAL)功能？](https://docs.mongodb.com/manual/faq/replica-sets/#why-use-journaling-if-replication-already-provides-data-redundancy)
- [仲裁节点与副本集的其它节点交换哪些信息？](https://docs.mongodb.com/manual/faq/replica-sets/#what-information-do-arbiters-exchange-with-the-rest-of-the-replica-set)
- [副本集成员使用不同大小的磁盘空间是否正常？](https://docs.mongodb.com/manual/faq/replica-sets/#is-it-normal-for-replica-set-members-to-use-different-amounts-of-disk-space)
- [我可以重命名副本集吗？](https://docs.mongodb.com/manual/faq/replica-sets/#can-i-rename-a-replica-set)


本文档回答了有关MongoDB中复制的常见问题。另请参见手册中的“ [复制”](https://docs.mongodb.com/manual/replication/)部分，其中[概述了复制](https://docs.mongodb.com/manual/replication/)，包括有关以下方面的详细信息：


- [副本集成员](https://docs.mongodb.com/manual/core/replica-set-members/)

- [副本集部署体系结构](https://docs.mongodb.com/manual/core/replica-set-architectures/)

- [副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/)

  

## MongoDB支持哪种复制？


MongoDB支持[副本集](https://docs.mongodb.com/manual/replication/)，[副本集](https://docs.mongodb.com/manual/replication/)最多可包含[50个节点](https://docs.mongodb.com/manual/release-notes/3.0/#replica-sets-max-members)。



## 复制是否可以通过Internet和WAN连接进行？


可以。

例如，在东海岸数据中心可以部署一个[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)和一个[副节点](https://docs.mongodb.com/manual/reference/glossary/#term-secondary) ，以及在西海岸数据中心部署一个作为灾难恢复的[从节点](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)成员。


参见

[部署异地冗余的副本集](https://docs.mongodb.com/manual/tutorial/deploy-geographically-distributed-replica-set/)



## MongoDB可以通过“noisy”的连接进行复制吗？


是的，但连接失败和非常明显的延迟的情况下不行。

集合中的成员将尝试重新连接到集合中的其他成员，以响应网络波动。这不需要管理员干预。但是，如果副本集中节点之间的网络连接非常慢，则节点成员可能无法跟上复制。

参见

[副本集选举](https://docs.mongodb.com/manual/core/replica-set-elections/)



## 如果复制已经提供了数据冗余，为什么还要使用journaling（预写日志，WAL）功能？

[Journaling](https://docs.mongodb.com/manual/reference/glossary/#term-journal)功能有助于加快崩溃恢复速度。

Journaling功能对于防止电源故障特别有用，尤其是当副本集位于单个数据中心或电源电路中时。


当[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)与Journaling一起运行时，您可以安全地重新启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例，而无需其他干预。


注意

Journaling记录需要一些资源开销来进行写操作。但是，Journaling对读取性能没有影响。

默认情况下，在MongoDB v2.0及更高版本的所有64位版本上都启用Journaling功能。



## 仲裁节点与副本集的其余节点交换哪些信息？


仲裁节点永远不会复制集合的数据内容，但会与副本集的其余节点交换以下数据：

- 用于副本集认证仲裁节点的凭据。这些交换数据是加密的。
- 副本集配置数据和投票数据。此信息未加密。仅加密交换凭证。


如果您的MongoDB部署使用TLS / SSL，则仲裁节点与副本集其他成员之间的所有通信都是安全的。

有关更多信息，请参阅有关[为TLS / SSL配置mongod和mongos](https://docs.mongodb.com/manual/tutorial/configure-ssl/)的文档。与所有MongoDB组件一样，应该在安全网络上运行仲裁节点。


参见

[副本集仲裁成员](https://docs.mongodb.com/manual/core/replica-set-members/#replica-set-arbiters)概述 。



## 副本集成员使用不同大小的磁盘空间是否正常？


正常。

因素包括：不同的oplog大小，不同程度的存储碎片以及MongoDB的数据文件预分配，都可能导致节点之间的存储利用率发生一些变化。当您在不同时间添加成员时，存储使用差异将最为明显。（译者注：可以理解为先后添加，因此上述存储碎片程度等差异就会比较明显，从而导致影响磁盘占用不同）。



## 我可以重命名副本集吗？


不可以。

您可以使用“ [从MongoDB备份还原副本集”](https://docs.mongodb.com/manual/tutorial/restore-replica-set-from-backup/)教程中描述的备份和还原过程来创建具有所需名称的新副本集。为了确保原始副本集和新副本集之间的奇偶校验，可能需要停机。



原文链接：https://docs.mongodb.com/manual/faq/replica-sets/

译者：钟秋

update：小芒果
