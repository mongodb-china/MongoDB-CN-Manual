# MongoDB CRUD 操作

**在本页面**

* [创建操作](./#创建)
* [读取操作](./#读取)
* [更新操作](./#更新)
* [删除操作](./#删除)
* [批量写入](./#批量)

  **CURD操作指的是文档的**_**创建**_**、**_**读**_**、**_**更新**_**以及**_**删除**_**操作。**

## 创建操作

创建或插入操作会将新[文档](https://docs.mongodb.com/master/core/document/#bson-document-format)添加到[集合](https://docs.mongodb.com/master/core/databases-and-collections/#collections)中。 如果该集合当前不存在，则插入操作将创建该集合。

MongoDB提供以下将文档插入集合的方法：

* [`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) _3.2版中的新功能_
* [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) _3.2版中的新功能_

在MongoDB中，插入操作针对单个[集合](https://docs.mongodb.com/master/core/databases-and-collections/#collections)。 MongoDB中的所有写操作都是单个文档级别的[原子](https://docs.mongodb.com/master/core/write-operations-atomicity/)操作。![](https://docs.mongodb.com/master/_images/crud-annotated-mongodb-insertOne.bakedsvg.svg)

有关示例，请参见[插入文档](https://docs.mongodb.com/manual/tutorial/insert-documents/)。

## 读取操作

读取操作从集合中检索文档； 即查询集合中的文档。 MongoDB提供了以下方法来从集合中读取文档：

* [db.collection.find\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)

您可以指定查询过滤器或条件以标识要返回的文档。

![](https://docs.mongodb.com/master/_images/crud-annotated-mongodb-find.bakedsvg.svg)

**有关示例，请参见：**

* [查询文件](https://docs.mongodb.com/manual/tutorial/query-documents/)
* [查询嵌入/嵌套文档](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)
* [查询数组](https://docs.mongodb.com/manual/tutorial/query-arrays/)
* [查询嵌入式文档数组](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)

## 更新操作

更新操作会修改集合中的现有文档。 MongoDB提供了以下更新集合文档的方法：

* [`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne) _3.2版中的新功能_
* [`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany) _3.2版中的新功能_
* [`db.collection.replaceOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne) _3.2版中的新功能_

在MongoDB中，更新操作针对单个集合。 MongoDB中的所有写操作都是单个文档级别的原子操作。

您可以指定标准或过滤器，以标识要更新的文档。 这些过滤器使用与读取操作相同的语法。![](https://docs.mongodb.com/master/_images/crud-annotated-mongodb-updateMany.bakedsvg.svg)

有关示例，请参见[更新文档](https://docs.mongodb.com/manual/tutorial/update-documents/)。

## 删除操作

删除操作从集合中删除文档。 MongoDB提供以下删除集合文档的方法：

* [`db.collection.deleteOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne) _3.2版中的新功能_
* [`db.collection.deleteMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany) _3.2版中的新功能_

在MongoDB中，删除操作只针对单个集合。MongoDB中的所有写操作都是单个文档级别的原子 操作。

你可以指定查询过滤器或条件来标识要更新的文档，这里的过滤器和读操作的语法是一致的。

![](https://docs.mongodb.com/master/_images/crud-annotated-mongodb-deleteMany.bakedsvg.svg)

有关示例，请参见[删除文档](https://docs.mongodb.com/manual/tutorial/remove-documents/)。

## 批量写入

MongoDB提供了批量执行写入操作的功能。有关详细信息，请参见[批量写入操作](https://docs.mongodb.com/manual/core/bulk-write-operations/)。

译者：刘翔 杨帅

校对：徐雷 杨帅 王恒

