# 数据模型设计

在本页

- [嵌入式数据模型](https://docs.mongodb.com/manual/core/data-model-design/#embedded-data-models)
- [规范化数据模型](https://docs.mongodb.com/manual/core/data-model-design/#normalized-data-models)
- [进一步阅读](https://docs.mongodb.com/manual/core/data-model-design/#further-reading)


有效的数据模型支持您的应用程序需求。文档结构的关键考虑因素是决定[嵌入](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)或引用 [使用引用](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-referencing)。



## 嵌入式数据模型


使用MongoDB，您可以将相关数据嵌入到单个结构或文档中。这些模式通常被称为“非规范化”模型，并利用MongoDB的丰富文档。参考下图：

![Data model with embedded fields that contain all related information.](https://docs.mongodb.com/manual/_images/data-model-denormalized.bakedsvg.svg)

嵌入式数据模型允许应用程序在同一个数据库记录中存储相关信息。因此，应用程序可能需要发出更少的查询和更新来完成常见操作。

通常，在以下情况下使用嵌入式数据模型：

- 实体之间存在“包含”关系。 见[与嵌入文档建立一对一关系模型](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/#data-modeling-example-one-to-one).
- 实体之间有一对多的关系。在这些关系中，“多个”或子文档始终与“一个”或父文档一起出现或在其上下文中查看。 见[与嵌入文档建立一对多关系模型](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#data-modeling-example-one-to-many).

一般来说，嵌入为读取操作提供了更好的性能，并且能够在单个数据库操作中请求和检索相关数据。嵌入式数据模型使得在单个原子写入操作中更新相关数据成为可能。

要访问嵌入文档中的数据，请使用[点符号](https://docs.mongodb.com/manual/reference/glossary/#term-dot-notation) “访问”嵌入文档。有关访问数组和嵌入文档中的数据的更多示例，请参见[查询数组中的数据](https://docs.mongodb.com/manual/tutorial/query-arrays/#read-operations-arrays)和 [查询嵌入文档中的数据](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/#read-operations-embedded-documents)。



### 嵌入式数据模型和文档大小限制


MongoDB中的文档必须小于[`BSON文档的最大大小`](https://docs.mongodb.com/manual/reference/limits/#BSON-Document-Size)。

对于大容量二进制数据，请考虑[GridFS](https://docs.mongodb.com/manual/core/gridfs/)。



## 规范化数据模型


规范化数据模型使用文档之间的 [引用](https://docs.mongodb.com/manual/reference/database-references/)来描述关系。

![Data model using references to link documents. Both the ``contact`` document and the ``access`` document contain a reference to the ``user`` document.](https://docs.mongodb.com/manual/_images/data-model-normalized.bakedsvg.svg)

通常，使用规范化数据模型：

- 当嵌入将导致重复数据，但不会提供足够的读取性能优势，超过重复的影响。
- 表示更复杂的多对多关系。
- 为大型分层数据集建模。

要加入集合，MongoDB提供聚合阶段：

- [`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup) （从MongoDB 3.2开始提供）
- [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup)（从MongoDB 3.4开始提供）

MongoDB还提供了跨集合联接数据的引用。

对于规范化数据模型的示例, 见 [使用文档引用建立一对多关系模型](https://docs.mongodb.com/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/#data-modeling-publisher-and-books)。

各种树模型的示例, 见 [模型树结构](https://docs.mongodb.com/manual/applications/data-models-tree-structures/)。


## 进一步阅读


有关MongoDB数据建模的更多信息, 请下载[MongoDB应用程序现代化指南](https://www.mongodb.com/modernize?tck=docs_server) 。

下载包括以下资源：

- MongoDB的数据建模方法
- 白皮书涵盖了从 [RDBMS](https://docs.mongodb.com/manual/reference/glossary/#term-rdbms)数据模型迁移到MongoDB的最佳实践和考虑事项
- 引用MongoDB模式及其等效关系数据库
- 应用程序现代化记分卡



原文链接：https://docs.mongodb.com/manual/core/data-model-design/

译者：张鹏

