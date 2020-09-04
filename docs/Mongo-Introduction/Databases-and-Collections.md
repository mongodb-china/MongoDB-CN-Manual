# 数据库和集合

在本页面

- [数据库](https://docs.mongodb.com/v4.2/core/databases-and-collections/#databases)
- [集合](https://docs.mongodb.com/v4.2/core/databases-and-collections/#collections)

MongoDB将[BSON文档](https://docs.mongodb.com/v4.2/core/document/#bson-document-format)（即数据记录）存储在[集合中](https://docs.mongodb.com/v4.2/reference/glossary/#term-collection)；数据库中的集合。



![A collection of MongoDB documents.](https://docs.mongodb.com/v4.2/_images/crud-annotated-collection.bakedsvg.svg)



## 数据库

在MongoDB中，文档集合存在数据库中。

要选择使用的数据库，请在[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell程序中发出 `use <db>` 语句，如下方示例：

复制

```
use myDB
```



### 创建数据库

如果数据库不存在，则在您第一次为该数据库存储数据时，MongoDB会创建该数据库。这样，您可以切换到不存在的数据库并在[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell中执行以下操作 ：

复制

```
use myNewDB

db.myNewCollection1.insertOne( { x: 1 } )
```

该[`insertOne()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.insertOne/#db.collection.insertOne)操作将同时创建数据库`myNewDB`和集合`myNewCollection1`（如果它们尚不存在）。确保数据库名称和集合名称均遵循MongoDB [命名限制](https://docs.mongodb.com/v4.2/reference/limits/#restrictions-on-db-names)。



## 集合

MongoDB将文档存储在集合中。集合类似于关系数据库中的表。

### 创建集合

如果不存在集合，则在您第一次为该集合存储数据时，MongoDB会创建该集合。

复制

```
db.myNewCollection2.insertOne( { x: 1 } )
db.myNewCollection3.createIndex( { y: 1 } )
```

如果[`insertOne()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.insertOne/#db.collection.insertOne)和 [`createIndex()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.createIndex/#db.collection.createIndex)操作都还不存在，则会创建它们各自的集合。确保集合名称遵循MongoDB [命名限制](https://docs.mongodb.com/v4.2/reference/limits/#restrictions-on-db-names)。



### 显示创建

MongoDB提供了[`db.createCollection()`](https://docs.mongodb.com/v4.2/reference/method/db.createCollection/#db.createCollection)使用各种选项显式创建集合的方法，例如设置最大大小或文档验证规则。如果未指定这些选项，则无需显式创建集合，因为在首次存储集合数据时，MongoDB会创建新集合。

要修改这些收集选项，请参见[`collMod`](https://docs.mongodb.com/v4.2/reference/command/collMod/#dbcmd.collMod)。



### 文档验证

*3.2版中的新功能。*

默认情况下，集合不要求其文档具有相同的模式。也就是说，单个集合中的文档不需要具有相同的字段集，并且字段的数据类型可以在集合中的不同文档之间有所不同。

但是，从MongoDB 3.2开始，您可以在更新和插入操作期间对集合强制执行[文档验证规则](https://docs.mongodb.com/v4.2/core/schema-validation/)。有关详细信息，请参见[模式验证](https://docs.mongodb.com/v4.2/core/schema-validation/)。



### 修改文档结构

要更改集合中文档的结构，例如添加新字段，删除现有字段或将字段值更改为新类型，请将文档更新为新结构。



### 唯一标识符

*3.6版的新功能。*

注意

在`featureCompatibilityVersion`必须设置为`"3.6"`或更大。有关更多信息，请参见[View FeatureCompatibilityVersion](https://docs.mongodb.com/v4.2/reference/command/setFeatureCompatibilityVersion/#view-fcv)。

集合被分配了一个不变的UUID。副本集的所有成员和分片群集中的分片的集合UUID均相同。

要检索集合的UUID，请运行 [listCollections](https://docs.mongodb.com/manual/reference/command/listCollections)命令或[`db.getCollectionInfos()`](https://docs.mongodb.com/v4.2/reference/method/db.getCollectionInfos/#db.getCollectionInfos)方法。



原文链接：https://docs.mongodb.com/v4.2/core/databases-and-collections/

译者：小芒果