# MongoDB简介

在本页

- [文档数据库](https://docs.mongodb.com/v4.2/introduction/#document-database)
- [主要特性](https://docs.mongodb.com/v4.2/introduction/#key-features)



欢迎使用MongoDB 4.2手册！MongoDB是一个文档数据库，旨在简化开发和扩展。该手册介绍了MongoDB中的关键概念，介绍了查询语言，并提供了操作和管理上的注意事项和过程以及全面的参考章节。

MongoDB提供数据库的*社区*版和*企业*版：

- MongoDB社区版是MongoDB的[可用源和免费](https://github.com/mongodb/mongo/)版本。
- MongoDB企业版作为MongoDB高级企业版订阅的一部分提供，并且包括对MongoDB部署的全面支持。MongoDB企业版还添加了以企业为中心的功能，例如LDAP和Kerberos支持，磁盘加密和审核。



## 文档数据库

MongoDB中的记录是一个文档，它是由字段和值对组成的数据结构。MongoDB文档类似于JSON对象。字段的值可以包括其他文档，数组和文档数组。

![A MongoDB document.](https://docs.mongodb.com/v4.2/_images/crud-annotated-document.bakedsvg.svg)

使用文档的优点是：

- 文档（即对象）对应于许多编程语言中的内置数据类型。
- 嵌入式文档和数组减少了对昂贵连接的需求。
- 动态模式支持流畅的多态性。



### 集合/视图/按需实例化视图

MongoDB将文档存储在[集合中](https://docs.mongodb.com/v4.2/core/databases-and-collections/#collections)。集合类似于关系数据库中的表。

除集合外，MongoDB还支持：

- 只读[视图](https://docs.mongodb.com/v4.2/core/views/)（从MongoDB 3.4开始）
- [按需实例化视图](https://docs.mongodb.com/v4.2/core/materialized-views/)（从MongoDB 4.2开始）。



## 主要特性

### 高性能

MongoDB提供高性能的数据持久化。特别是，

- 对嵌入式数据模型的支持减少了数据库系统上的I / O操作。
- 索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。



### 丰富的查询语言

MongoDB支持丰富的查询语言以支持[读写操作（CRUD）](https://docs.mongodb.com/v4.2/crud/)以及：

- [数据聚合](https://docs.mongodb.com/v4.2/core/aggregation-pipeline/)
- [文本搜索](https://docs.mongodb.com/v4.2/text-search/)和[地理空间查询](https://docs.mongodb.com/v4.2/tutorial/geospatial-tutorial/)。



也可以看看

- [SQL到MongoDB的映射图](https://docs.mongodb.com/v4.2/reference/sql-comparison/)
- [SQL到聚合的映射图](https://docs.mongodb.com/v4.2/reference/sql-aggregation-comparison/)



### 高可用

MongoDB的复制工具（称为[副本集](https://docs.mongodb.com/v4.2/replication/)）提供：

- *自动*故障转移
- 数据冗余。

[副本集](https://docs.mongodb.com/v4.2/replication/)是一组维护相同数据集合的 mongod实例，提供了冗余和提高了数据可用性。



### 水平拓展

MongoDB提供水平可伸缩性作为其*核心* 功能的一部分：

- [分片](https://docs.mongodb.com/v4.2/sharding/#sharding-introduction)将数据分布在一个集群的机器上。
- 从3.4开始，MongoDB支持基于[分片键](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key)创建数据[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)。在平衡群集中，MongoDB仅将区域覆盖的读写定向到区域内的那些分片。有关 更多信息，请参见[区域](https://docs.mongodb.com/v4.2/core/zone-sharding/#zone-sharding)章节。



### 支持多种存储引擎

MongoDB支持[多个存储引擎](https://docs.mongodb.com/v4.2/core/storage-engines/)：

- [WiredTiger存储引擎](https://docs.mongodb.com/v4.2/core/wiredtiger/)（包括对[静态](https://docs.mongodb.com/v4.2/core/wiredtiger/)[加密的](https://docs.mongodb.com/v4.2/core/security-encryption-at-rest/)支持 ）
- [内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)。

另外，MongoDB提供可插拔的存储引擎API，允许第三方为MongoDB开发存储引擎。



←  [MongoDB手册内容](https://docs.mongodb.com/v4.2/contents/)



原文链接：https://docs.mongodb.com/v4.2/introduction/

译者：小芒果
