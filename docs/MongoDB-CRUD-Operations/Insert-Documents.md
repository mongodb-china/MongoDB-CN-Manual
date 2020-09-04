# 插入文档
该页面提供了MongoDB中插入操作的示例。

> **建立集合**
>
> 如果该集合当前不存在，则插入操作将创建该集合。

## **插入一个文件**

 [`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)将单个[文档](https://docs.mongodb.com/master/core/document/#bson-document-format)插入集合中。

以下示例将一个新文档插入库存集合。 如果文档未指定**_id**字段，则MongoDB将具有**ObjectId**值的**_id**字段添加到新文档中。 请参阅[插入行为](https://docs.mongodb.com/manual/tutorial/insert-documents/#write-op-insert-behavior)。

```shell
db.inventory.insertOne(  
		{ item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
```

[`insertOne()`](https://docs.mongodb.com/master/reference/method/db.collection.insertOne/#db.collection.insertOne)返回一个文档，其中包含新插入的文档的_id字段值。有关返回文档的示例，请参阅[`db.collection.insertOne() reference`](https://docs.mongodb.com/master/reference/method/db.collection.insertOne/#insertone-examples)引用。

要检索刚刚插入的文档，[查询集合:](https://docs.mongodb.com/master/core/document/#document-query-filter)

```shell
db.inventory.find( { item: "canvas" } )
```

## **插入多个文件**

*3.2版中的新功能*

[db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)可以将多个文档插入一个集合中。 将文档数组传递给该方法。

下面的示例将三个新文档插入库存集合。 如果文档未指定**_id**字段，则MongoDB向每个文档添加带有**ObjectId**值的**_id**字段。 请参阅 [插入行为](https://docs.mongodb.com/manual/tutorial/insert-documents/#write-op-insert-behavior)。

```shell
db.inventory.insertMany([
		{ item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } }, 
		{ item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
		{ item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
	])
```

返回包含新插入的文档**_id**字段值的文档。 有关示例，请参见[参考](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#insertmany-examples)。

要检索插入的文档，[查询集合:](https://docs.mongodb.com/master/core/document/#document-query-filter)

```shell
db.inventory.find( {} )
```

## **插入行为**

### 集合创建

如果该集合当前不存在，则插入操作将创建该集合。

### `_id` Field

在MongoDB中，存储在集合中的每个文档都需要一个唯一的**_id**字段作为主键。 如果插入的文档省略**_id**字段，则MongoDB驱动程序会自动为**_id**字段生成**ObjectId**。

这也适用于通过[upsert：true](#)通过更新操作插入的文档。

### 原子性

MongoDB中的所有写操作都是单个文档级别的原子操作。 有关MongoDB和原子性的更多信息，请参见[原子性和事务](https://docs.mongodb.com/manual/core/write-operations-atomicity/).

### 写确认书

对于写入问题，您可以指定从MongoDB请求的写入操作的确认级别。 有关详细信息，请参见[写关注](https://docs.mongodb.com/manual/reference/write-concern/)。

另可参考：

- [db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)
- [db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)
- [Additional Methods for Inserts](https://docs.mongodb.com/manual/reference/insert-methods/#additional-inserts)



译者：杨帅

校对：杨帅