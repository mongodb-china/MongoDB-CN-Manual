# 常见问题解答：MongoDB基础知识[¶](https://docs.mongodb.com/manual/faq/fundamentals/#faq-mongodb-fundamentals)


在本页面

- [MongoDB支持哪些平台？](https://docs.mongodb.com/manual/faq/fundamentals/#what-platforms-does-mongodb-support)
- [MongoDB是否作为托管服务提供？](https://docs.mongodb.com/manual/faq/fundamentals/#is-mongodb-offered-as-a-hosted-service)
- [集合（collection）与表（table）有何不同？](https://docs.mongodb.com/manual/faq/fundamentals/#how-does-a-collection-differ-from-a-table)
- [如何创建数据库和集合？](https://docs.mongodb.com/manual/faq/fundamentals/#how-do-i-create-a-database-and-a-collection)
- [如何定义或修改集合模式？](https://docs.mongodb.com/manual/faq/fundamentals/#how-do-i-define-or-alter-the-collection-schema)
- [MongoDB是否支持SQL？](https://docs.mongodb.com/manual/faq/fundamentals/#does-mongodb-support-sql)
- [MongoDB是否支持事务？](https://docs.mongodb.com/manual/faq/fundamentals/#does-mongodb-support-transactions)
- [MongoDB是否处理缓存？](https://docs.mongodb.com/manual/faq/fundamentals/#does-mongodb-handle-caching)
- [MongoDB如何解决SQL或查询注入问题？](https://docs.mongodb.com/manual/faq/fundamentals/#how-does-mongodb-address-sql-or-query-injection)


本文档回答了有关MongoDB的一些常见问题。



## MongoDB支持哪些平台？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#what-platforms-does-mongodb-support)


有关支持的列表平台，请参阅 [支持的平台](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-supported-platforms)。



## MongoDB是否作为托管服务提供？


是的。[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)是一种云托管的数据库即服务。有关更多信息，请访问[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)。



## 集合与表格有何不同？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#how-does-a-collection-differ-from-a-table)


MongoDB数据库将数据存储在[集合collections中](https://docs.mongodb.com/manual/reference/glossary/#term-collection)，而不是表table。集合包含一个或多个 [BSON文档](https://docs.mongodb.com/manual/core/document/#bson-document-format)。文档类似于关系数据库表中的记录或行。每个文档都有 [一个或多个字段](https://docs.mongodb.com/manual/core/document/#document-structure)；字段类似于关系数据库表中的列。



也可以看看

[SQL到MongoDB映射图](https://docs.mongodb.com/manual/reference/sql-comparison/)，[ MongoDB简介](https://docs.mongodb.com/manual/introduction/)



## 如何创建数据库和集合？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#how-do-i-create-a-database-and-a-collection)


如果数据库不存在，MongoDB会在您第一次为该数据库存储数据时创建该数据库。


如果集合不存在，则在您第一次为该集合存储数据时创建该集合。[[1\]](https://docs.mongodb.com/manual/faq/fundamentals/#explicit-creation)


因此，您可以切换到一个不存在的数据库 (`use `) 并执行以下操作：

复制

```
use myNewDB

db.myNewCollection1.insertOne( { x: 1 } )
db.myNewCollection2.createIndex( { a: 1 } )
```


如果数据库`myNewDB`和集合`myNewCollection1`尚不存在，`insert`操作将同时创建它们。


发生在`myNewDB`库创建之后的`createIndex`操作，将创建索引，并且如果集合不存在的话也会创建`myNewCollection2` 集合。如果`myNewDb`不存在，则该 `createIndex`操作还将创建`myNewDB`。


| [[1\]](https://docs.mongodb.com/manual/faq/fundamentals/#id2) | 如果要指定特定集合选项，您也可以也可以使用显式[`db.createCollection`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection)创建集合，例如制定最大大小或文档验证规则。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |




## 如何定义或修改集合模式？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#how-do-i-define-or-alter-the-collection-schema)


在MongoDB中您无需为集合指定模式。尽管集合中的文档通常具有很大程度上相同的结构，但这不是必需的；也就是说，单个集合中的文档不需要具有相同的字段集。字段的数据类型在集合中的文档之间也可能不同。


要更改集合中文档的结构，请将文档更新为新结构。例如，添加新字段，删除现有字段或将字段值更新为新类型。


*在版本3.2中进行了更改：*从MongoDB 3.2开始，您可以在更新和插入操作期间对集合强制执行[文档验证规则](https://docs.mongodb.com/manual/core/schema-validation/)。


可以在显式创建集合时指定某些集合属性，例如指定最大大小，然后对其进行修改。请参阅[`db.createCollection`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection)和[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)。如果未指定这些属性，则无需显式创建集合，因为在首次存储集合的数据时，MongoDB会创建新的集合。



## MongoDB是否支持SQL？



不直接支持。但是，MongoDB自身确实支持丰富的查询语言。有关使用MongoDB查询语言的示例，请参阅 [MongoDB CRUD操作](https://docs.mongodb.com/manual/crud/)。


您还可以使用 [MongoDB Connector for BI](https://www.mongodb.com/products/bi-connector) 来使用SQL查询MongoDB集合。


如果您正在考虑将SQL应用程序迁移到MongoDB，请下载《[MongoDB应用程序现代化指南》](https://www.mongodb.com/modernize?tck=docs_server)以获取最佳实践迁移指南，参考模式和其他有用的资源。


也可以看看

[SQL到MongoDB的映射图](https://docs.mongodb.com/manual/reference/sql-comparison/)



## MongoDB是否支持事务？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#does-mongodb-support-transactions)


由于单个文档可以包含相关数据，否则这些相关数据将在关系模式中的各个父子表之间建模，因此MongoDB的原子单文档操作已经提供了满足大多数应用程序数据完整性需求的事务语义。可以在单个操作中写入一个或多个字段，包括对多个子文档和数组元素的更新。MongoDB提供的保证可确保在文档更新时完全隔离。任何错误都会导致操作回滚，以使客户端获得一致的文档视图。


但是，对于需要对多个文档（在单个或多个集合中）进行读写原子性的情况，MongoDB支持多文档事务：


- **在版本4.0中**，MongoDB支持副本集上的多文档事务。

- **在版本4.2中**，MongoDB引入了分布式事务，它增加了对分片群集上多文档事务的支持，并合并了对副本集上多文档事务的现有支持。

  有关MongoDB中事务的详细信息，请参阅 [事务](https://docs.mongodb.com/manual/core/transactions/)页面。


重要

在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应代替高效的模式设计。对于许多应用场景， [非规范化数据模型（嵌入式文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)将继续为您的数据和用例的最佳选择。也就是说，对于许多场景，对数据进行适当的建模将最大程度地减少对多文档事务的需求。


有关其他事务使用方面的注意事项（例如运行时限制和oplog大小限制），另请参见 [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)。


## MongoDB是否处理缓存？


是的。MongoDB将最近使用的数据保留在RAM中。如果您为查询创建了索引，并且您的工作数据集适合RAM，那么MongoDB将从内存中进行所有查询。

MongoDB不缓存查询结果，以便为相同查询返回缓存结果。

有关MongoDB和内存使用的更多信息，请参阅[WiredTiger和内存使用](https://docs.mongodb.com/manual/core/wiredtiger/#wiredtiger-ram)。


## MongoDB如何解决SQL或查询注入问题？[¶](https://docs.mongodb.com/manual/faq/fundamentals/#how-does-mongodb-address-sql-or-query-injection)

### BSON

当客户端程序在MongoDB中组合查询时，将构建BSON对象而不是字符串。因此，传统的SQL注入攻击并不是问题。更多细节和一些细微差别将在下面介绍。

MongoDB将查询表示为[BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson)对象。通常， [客户端驱动库](https://docs.mongodb.com/ecosystem/drivers)提供了一个方便的，无注入的过程来构建这些对象。考虑下面的C ++示例：

复制

```
BSONObj my_query = BSON( "name" << a_name );
auto_ptr<DBClientCursor> cursor = c.query("tutorial.persons", my_query);
```

示例中，`my_query`将有一个诸如`{ name : "Joe" }`的值。如果`my_query`包含特殊字符，例如，`,`，`:`和`{`，查询将不匹配任何文档。例如，用户无法劫持查询并将其转换为删除。


### JavaScript

注意

您可以通过[`--noscripting`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-noscripting)在命令行中传递选项或[`security.javascriptEnabled`](https://docs.mongodb.com/manual/reference/configuration-options/#security.javascriptEnabled)在配置文件中进行设置来禁用所有服务器端JavaScript的执行 。


以下所有MongoDB操作都允许您直接在服务器上运行任意JavaScript表达式：

- [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where)
- [`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce)


在这些情况下，您必须格外小心，以防止用户提交恶意JavaScript。

幸运的是，您可以在没有JavaScript的MongoDB中表达大多数查询，对于需要JavaScript的查询，可以在单个查询中混合使用JavaScript和非JavaScript。将所有用户提供的字段直接放在[BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson)字段中，并将JavaScript代码传递给该[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where)字段。

如果需要在[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where)子句中传递用户提供的值，则可以使用该`CodeWScope`机制来转义这些值。在范围文档中将用户提交的值设置为变量时，可以避免在数据库服务器上评估它们。



原文链接：https://docs.mongodb.com/manual/faq/fundamentals/

译者：钟秋

update：小芒果

