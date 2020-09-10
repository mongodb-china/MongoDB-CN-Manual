# MongoDB简介

在本页

* [文档数据库](https://docs.mongodb.com/v4.2/introduction/#document-database)
* [主要特性](https://docs.mongodb.com/v4.2/introduction/#key-features)

欢迎使用MongoDB 4.2手册！MongoDB是一个文档数据库，旨在简化开发和扩展。该手册介绍了MongoDB中的关键概念，介绍了查询语言，并提供了操作和管理上的注意事项和过程以及全面的参考章节。

MongoDB提供数据库的_社区_版和_企业_版：

* MongoDB社区版是MongoDB的[可用源和免费](https://github.com/mongodb/mongo/)版本。
* MongoDB企业版作为MongoDB高级企业版订阅的一部分提供，并且包括对MongoDB部署的全面支持。MongoDB企业版还添加了以企业为中心的功能，例如LDAP和Kerberos支持，磁盘加密和审核。

## 文档数据库

MongoDB中的记录是一个文档，它是由字段和值对组成的数据结构。MongoDB文档类似于JSON对象。字段的值可以包括其他文档，数组和文档数组。

![A MongoDB document.](https://docs.mongodb.com/v4.2/_images/crud-annotated-document.bakedsvg.svg)

使用文档的优点是：

* 文档（即对象）对应于许多编程语言中的内置数据类型。
* 嵌入式文档和数组减少了对昂贵连接的需求。
* 动态模式支持流畅的多态性。

### 集合/视图/按需实例化视图

MongoDB将文档存储在[集合中](https://docs.mongodb.com/v4.2/core/databases-and-collections/#collections)。集合类似于关系数据库中的表。

除集合外，MongoDB还支持：

* 只读[视图](https://docs.mongodb.com/v4.2/core/views/)（从MongoDB 3.4开始）
* [按需实例化视图](https://docs.mongodb.com/v4.2/core/materialized-views/)（从MongoDB 4.2开始）。

## 主要特性

### 高性能

MongoDB提供高性能的数据持久化。特别是，

* 对嵌入式数据模型的支持减少了数据库系统上的I / O操作。
* 索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。

### 丰富的查询语言

MongoDB支持丰富的查询语言以支持[读写操作（CRUD）](https://docs.mongodb.com/v4.2/crud/)以及：

* [数据聚合](https://docs.mongodb.com/v4.2/core/aggregation-pipeline/)
* [文本搜索](https://docs.mongodb.com/v4.2/text-search/)和[地理空间查询](https://docs.mongodb.com/v4.2/tutorial/geospatial-tutorial/)。

也可以看看

* [SQL到MongoDB的映射图](https://docs.mongodb.com/v4.2/reference/sql-comparison/)
* [SQL到聚合的映射图](https://docs.mongodb.com/v4.2/reference/sql-aggregation-comparison/)

### 高可用

MongoDB的复制工具（称为[副本集](https://docs.mongodb.com/v4.2/replication/)）提供：

* _自动_故障转移
* 数据冗余。

[副本集](https://docs.mongodb.com/v4.2/replication/)是一组维护相同数据集合的 mongod实例，提供了冗余和提高了数据可用性。

### 水平拓展

MongoDB提供水平可伸缩性作为其_核心_ 功能的一部分：

* [分片](https://docs.mongodb.com/v4.2/sharding/#sharding-introduction)将数据分布在一个集群的机器上。
* 从3.4开始，MongoDB支持基于[分片键](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key)创建数据[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)。在平衡群集中，MongoDB仅将区域覆盖的读写定向到区域内的那些分片。有关 更多信息，请参见[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)章节。

### 支持多种存储引擎

MongoDB支持[多个存储引擎](https://docs.mongodb.com/v4.2/core/storage-engines/)：

* [WiredTiger存储引擎](https://docs.mongodb.com/v4.2/core/wiredtiger/)（包括对[静态](https://docs.mongodb.com/v4.2/core/wiredtiger/)[加密的](https://docs.mongodb.com/v4.2/core/security-encryption-at-rest/)支持 ）
* [内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)。

另外，MongoDB提供可插拔的存储引擎API，允许第三方为MongoDB开发存储引擎。

← [MongoDB手册内容](https://docs.mongodb.com/v4.2/contents/)

原文链接：[https://docs.mongodb.com/v4.2/introduction/](https://docs.mongodb.com/v4.2/introduction/)


### MongoDB中文社区

![MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x2014;MongoDB&#x7231;&#x597D;&#x8005;&#x6280;&#x672F;&#x4EA4;&#x6D41;&#x5E73;&#x53F0;](https://mongoing.com/wp-content/uploads/2020/09/6de8a4680ef684d-2.png)

| 资源列表推荐 | 资源入口 |
| :--- | :--- |
| MongoDB中文社区官网 | [https://mongoing.com/](https://mongoing.com/) |
| 微信服务号 ——最新资讯和优质文章 | Mongoing中文社区（mongoing-mongoing） |
| 微信订阅号 ——发布文档翻译内容 | MongoDB中文用户组（mongoing123） |
| 官方微信号 —— 官方最新资讯 | MongoDB数据库（MongoDB-China） |
| MongoDB中文社区组委会成员介绍 | [https://mongoing.com/core-team-members](https://mongoing.com/core-team-members) |
| MongoDB中文社区翻译小组介绍 | [https://mongoing.com/translators](https://mongoing.com/translators) |
| MongoDB中文社区微信技术交流群 | 添加社区助理小芒果微信（ID:mongoingcom），并备注 mongo |
| MongoDB中文社区会议及文档资源 | [https://mongoing.com/resources](https://mongoing.com/resources) |
| MongoDB中文社区大咖博客 | [基础知识](https://mongoing.com/basic-knowledge)  [性能优化](https://mongoing.com/performance-optimization)  [原理解读](https://mongoing.com/interpretation-of-principles)  [运维监控](https://mongoing.com/operation-and-maintenance-monitoring)  [最佳实践](https://mongoing.com/best-practices) |
| MongoDB白皮书 | [https://mongoing.com/mongodb-download-white-paper](https://mongoing.com/mongodb-download-white-paper) |
| MongoDB初学者教程-7天入门 | [https://mongoing.com/mongodb-beginner-tutorial](https://mongoing.com/mongodb-beginner-tutorial) |
| 社区活动邮件订阅 | [https://sourl.cn/spszjN](https://sourl.cn/spszjN) |



