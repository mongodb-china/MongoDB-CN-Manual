# 数据建模介绍

在本页中

- [灵活的模式](https://docs.mongodb.com/manual/core/data-modeling-introduction/#flexible-schema)
- [文档结构](https://docs.mongodb.com/manual/core/data-modeling-introduction/#document-structure)
- [写操作原子性](https://docs.mongodb.com/manual/core/data-modeling-introduction/#atomicity-of-write-operations)
- [数据使用和性能](https://docs.mongodb.com/manual/core/data-modeling-introduction/#data-use-and-performance)
- [扩展阅读](https://docs.mongodb.com/manual/core/data-modeling-introduction/#further-reading)

数据建模的关键挑战是平衡应用程序的需求、数据库引擎的性能特征和数据检索模式。在设计数据模型时，始终考虑数据的应用程序使用（即数据的查询、更新和处理）以及数据本身的固有结构。



## 灵活的模式


与SQL数据库不同，在SQL数据库中，在插入数据之前必须确定并声明表的架构，MongoDB的 [集合](https://docs.mongodb.com/manual/reference/glossary/#term-collection)，默认情况下，集合不需要其[文档](https://docs.mongodb.com/manual/core/document/) 有相同的模式。即：

- 单个集合中的文档不需要具有相同的字段集，并且字段的数据类型可以在集合中的各个文档之间有所不同。
- 若要更改集合中文档的结构（如添加新字段、删除现有字段或将字段值更改为新类型），请将文档更新为新结构。

这种灵活性有助于将文档映射到实体或对象。每个文档都可以匹配所表示实体的数据字段，即使该文档与集合中的其他文档有很大的差异。

但实际上，集合中的文档共享类似的结构，您可以强制执行[文档验证规则](https://docs.mongodb.com/manual/core/schema-validation/)用于在更新和插入操作期间的集合。请参阅[模式验证](https://docs.mongodb.com/manual/core/schema-validation/)详细情况。



## 文档结构


为MongoDB应用程序设计数据模型的关键决策是围绕文档的结构以及应用程序如何表示数据之间的关系。MongoDB允许将相关数据嵌入到单个文档中。



### 嵌入数据方式

嵌入式文档通过在单个文档结构中存储相关数据来捕获数据之间的关系。MongoDB文档使得在文档中的字段或数组中嵌入文档结构成为可能。这些*非规范化*数据模型允许应用程序在单个数据库操作中检索和操作相关数据。

![Data model with embedded fields that contain all related information.](https://docs.mongodb.com/manual/_images/data-model-denormalized.bakedsvg.svg)

对于MongoDB中的许多用例，非规范化数据模型是最优的。

参见[嵌入式数据模型](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)为嵌入文档的优点和缺点建模。



### 引用数据方式


引用通过包含从一个文档到另一个文档的链接或*引用*来存储数据之间的关系。应用程序可以解析这些[参考](https://docs.mongodb.com/manual/reference/database-references/)访问相关数据。一般来说，这些是“标准化”数据模型。

![Data model using references to link documents. Both the ``contact`` document and the ``access`` document contain a reference to the ``user`` document.](https://docs.mongodb.com/manual/_images/data-model-normalized.bakedsvg.svg)

参见 [规范化数据模型](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-referencing) 了解使用参考的优点和缺点。



## 写操作原子性

### 单文档原子性


在MongoDB中，写操作在单个文档的级别上是原子的，即使该操作修改了单个文档中的多个嵌入文档。

带有嵌入数据的非规范化数据模型将所有相关数据合并到单个文档中，而不是跨多个文档和集合进行规范化。这个数据模型有助于原子操作。

当一次写入操作（例如[`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/édb.collection.updateMany))修改多个文档，每个文档的修改是原子的，但整个操作不是原子的。

在执行多文档写入操作时，无论是通过单个写入操作还是通过多个写入操作，其他操作都可能交叉进行。

对于需要对多个文档（在单个或多个集合中）进行原子性读写的情况，MongoDB支持多文档事务：

- **在4.0版中**，MongoDB支持副本集上的多文档事务。
- **在4.2版中**，MongoDB引入了分布式事务，增加了对分片集群上多文档事务的支持，并结合了对副本集上多文档事务的现有支持。

有关MongoDB中事务的详细信息，请参阅[事务](https://docs.mongodb.com/manual/core/transactions/)章节。


### 多文档事务

对于需要对多个文档（在单个或多个集合中）进行原子性读写的情况，MongoDB支持多文档事务：

- **在4.0版中**，MongoDB支持副本集上的多文档事务。
- **在4.2版中**，MongoDB引入了分布式事务，增加了对分片集群上多文档事务的支持，并结合了对副本集上多文档事务的现有支持。

有关MongoDB中事务的详细信息，请参阅[事务](https://docs.mongodb.com/manual/core/transactions/)章节。



> 注意事项:
>
> 在大多数情况下，多文档事务比单文档写入带来更高的性能成本，而且多文档事务的可用性不应替代有效的模式设计。对于许多情况，[非规范化数据模型（嵌入文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)将继续是数据和用例的最佳选择。也就是说，对于许多场景，对数据进行适当的建模将最大限度地减少对多文档事务的需求。
>
> 
>
> 有关其他事务使用注意事项（如运行时限制和oplog大小限制），另请参阅[生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/).

另请参见:

[原子性注意事项](https://docs.mongodb.com/manual/core/data-model-operations/#data-model-atomicity)



## 数据使用和性能


设计数据模型时，请考虑应用程序将如何使用数据库。例如，如果您的应用程序只使用最近插入的文档，请考虑使用[Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/). 或者，如果应用程序需要的主要是对集合的读取操作，则添加索引以支持常见查询可以提高性能。

见[操作因素和数据模型](https://docs.mongodb.com/manual/core/data-model-operations网站/)有关影响数据模型设计的这些和其他操作注意事项的详细信息。



## 扩展阅读


有关MongoDB数据建模的更多信息，请下载[MongoDB应用程序现代化指南](https://www.mongodb.com/modernize?tck=docs_server)。

下载包括以下资源：

- 用MongoDB实现数据建模的方法论
- 白皮书涵盖了从[RDBMS](https://docs.mongodb.com/manual/reference/glossary/#term-rdbms)数据模型迁移到MongoDB的最佳实践和考虑事项
- 参考MongoDB模式及其等价的RDBMS概念
- 应用程序现代化记分卡

←  [数据模型](https://docs.mongodb.com/manual/data-modeling/)  [模式验证](https://docs.mongodb.com/manual/core/schema-validation/) →



原文链接：https://docs.mongodb.com/manual/core/data-modeling-introduction/

译者：张鹏
