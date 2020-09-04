
# 删除文档
此页面使用以下[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell方法

- [db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany)
- [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)

此页面上的示例使用**inventory**收集。 要填充**inventory**收集，请运行以下命令：

```powershell
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
] );
```

## 删除所有文档

要删除集合中的所有文档，请将空的[filter](https://docs.mongodb.com/master/core/document/#document-query-filter)文档{}传递给[`db.collection.deleteMany()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteMany/#db.collection.deleteMany) 方法。

以下示例从**inventory**收集中删除所有文档：

```powershell
db.inventory.deleteMany({})
```

该方法返回具有操作状态的文档。 有关更多信息和示例，请参见[`deleteMany()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteMany/#db.collection.deleteMany).

## 删除所有符合条件的文档

您可以指定标准或过滤器，以标识要删除的文档。 [filter](https://docs.mongodb.com/master/core/document/#document-query-filter)使用与读取操作相同的语法。

要指定相等条件，请在[查询过滤器文档](https://docs.mongodb.com/master/core/document/#document-query-filter):中使用**<`field`>**：**<`value`>**表达式：

```powershell
{ <field1>: <value1>, ... }
```

[查询过滤器文档](https://docs.mongodb.com/master/core/document/#document-query-filter)可以使用[查询操作符](https://docs.mongodb.com/master/reference/operator/query/#query-selectors) 以以下形式指定条件:

```powershell
{ <field1>: { <operator1>: <value1> }, ... }
```

要删除所有符合删除条件的文档，请将[filter](https://docs.mongodb.com/master/core/document/#document-query-filter)参数传递给[`deleteMany()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteMany/#db.collection.deleteMany)方法。

以下示例从状态字段等于**“ A”**的**inventory**集合中删除所有文档：

```powershell
db.inventory.deleteMany({ status : "A" })
```

该方法返回具有操作状态的文档。 有关更多信息和示例，请参见[`deleteMany()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteMany/#db.collection.deleteMany).

## 仅删除一个符合条件的文档

要删除最多一个与指定过滤器匹配的文档(即使多个文档可以与指定过滤器匹配)，请使用[`db.collection.deleteOne()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteOne/#db.collection.deleteOne)方法。

下面的示例删除状态为**“ D”**的第一个文档：

```powershell
db.inventory.deleteOne( { status: "D" } )
```

## 删除行为

### 索引

即使从集合中删除所有文档，删除操作也不会删除索引。

### 原子性

MongoDB中的所有写操作都是单个文档级别的原子操作。 有关MongoDB和原子性的更多信息，请参见[原子性和事务](https://docs.mongodb.com/manual/core/write-operations-atomicity/)。

### 写确认

对于写入问题，您可以指定从MongoDB请求的写入操作的确认级别。 有关详细信息，请参见 [写关注](https://docs.mongodb.com/manual/reference/write-concern/)。

另请参考：

- [db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany)
- [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)
- [Additional Methods](https://docs.mongodb.com/manual/reference/delete-methods/#additional-deletes)



译者：杨帅

校对：杨帅
