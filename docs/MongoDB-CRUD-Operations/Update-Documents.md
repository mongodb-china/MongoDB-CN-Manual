
# 更新文档
此页面使用以下 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell方法：

- [db.collection.updateOne(<`filter`>, <`update`>, <`options`>)](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)
- [db.collection.updateMany(<`filter`>, <`update`>, <`options`>)](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)
- [db.collection.replaceOne(<`filter`>, <`update`>, <`options`>)](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)

此页面上的示例使用库存收集。 要创建和/或填充清单集合，请运行以下命令：

此页上的示例使用**inventory**集合。要创建和/或填充**inventory**集合，请运行以下操作:

```shell
db.inventory.insertMany( [
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
   { item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
   { item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );
```

## 更新集合中的文档

为了更新文档，MongoDB提供了[更新操作符](https://docs.mongodb.com/manual/reference/operator/update)（例如[`$set`](https://docs.mongodb.com/master/reference/operator/update/set/#up._S_set)）来修改字段值。

要使用更新运算符，请将以下形式的更新文档传递给更新方法：

```sql
{
		<update operator>: { <field1>: <value1>, ... },
		<update operator>: { <field2>: <value2>, ... },
		...
}
```

如果字段不存在，则某些更新操作符（例如[`$set`](https://docs.mongodb.com/master/reference/operator/update/set/#up._S_set)）将创建该字段。 有关详细信息，请参见各个更新操作员参考。

> **[success] Note**
>
> **从MongoDB 4.2开始，MongoDB可以接受聚合管道来指定要进行的修改而不是更新文档。 有关详细信息，请参见方法参考页。**

### 更新单个文档

下面的示例在**inventory**集合上使用[`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)方法更新项目等于“ **paper**”的第一个文档：

```shell
db.inventory.updateOne(
	{ item: "paper" },
	{
		$set: { "size.uom": "cm", status: "P" }, 
		$currentDate: { lastModified: true }
	}
)
```

**更新操作：**

* 使用[`$set`](https://docs.mongodb.com/master/reference/operator/update/set/#up._S_set) 运算符将**size.uom**字段的值更新为“ **cm**”，将状态字段的值更新为“ **P**”，

* 使用[`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate)运算符将**lastModified**字段的值更新为当前日期。 如果**lastModified**字段不存在，则[`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate)将创建该字段。 有关详细信息，请参见[`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate)。

### 更新多个文档

*3.2版中的新功能*

以下示例在清单集合上使用[`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)方法来更新数量小于**50**的所有文档：

```shell
  db.inventory.updateMany( 
  	{ "qty": { $lt: 50 } },
  	{  
  		$set: { "size.uom": "in", status: "P" }, 
  		$currentDate: { lastModified: true }  
  	}
  )
```

**更新操作：**

* 使用[`$set`](https://docs.mongodb.com/master/reference/operator/update/set/#up._S_set)运算符将**size.uom**字段的值更新为“ **in**”，将状态字段的值更新为“ **P**”.

* 使用 [`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate) 运算符将**lastModified**字段的值更新为当前日期。如果**lastModified**字段不存在，则[`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate) 将创建该字段。有关详细信息，请参见[`$currentDate`](https://docs.mongodb.com/master/reference/operator/update/currentDate/#up._S_currentDate) 。

## 更换文档

要替换**_id**字段以外的文档的全部内容，请将一个全新的文档作为第二个参数传递给[`db.collection.replaceOne()`](https://docs.mongodb.com/master/reference/method/db.collection.replaceOne/#db.collection.replaceOne)。

当替换一个文档时，替换文档必须只包含字段/值对;即不包括更新操作符表达式。

替换文档可以具有与原始文档不同的字段。在替换文档中，由于**_id**字段是不可变的，因此可以省略**_id**字段。但是，如果您确实包含**_id**字段，则它必须与当前值具有相同的值。

下面的示例替换了**inventory**集合中的第一个文件，其中项为**"paper"**:

 ```powershell
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
 ```

## 行为

### 原子性

MongoDB中的所有写操作都是单个文档级别上的原子操作。有关MongoDB和原子性的更多信息，请参见原子性和事务。

### _id Field

设置后，您将无法更新**_id**字段的值，也无法将现有文档替换为具有不同**_id**字段值的替换文档。

### 字段顺序

除以下情况外，MongoDB会在执行写操作后保留文档字段的顺序：

* **_id**字段始终是文档中的第一个字段。
* 包含字段名称[`renaming`](https://docs.mongodb.com/master/reference/operator/update/rename/#up._S_rename) 的更新可能导致文档中字段的重新排序。

###  增补选项

如果[`updateOne()`](https://docs.mongodb.com/master/reference/method/db.collection.updateOne/#db.collection.updateOne), [`updateMany()`](https://docs.mongodb.com/master/reference/method/db.collection.updateMany/#db.collection.updateMany), or [`replaceOne()`](https://docs.mongodb.com/master/reference/method/db.collection.replaceOne/#db.collection.replaceOne) 包含**upsert：true**，并且没有文档与指定的过滤器匹配，则该操作将创建一个新文档并将其插入。 如果存在匹配的文档，则该操作将修改或替换一个或多个匹配的文档。

有关创建的新文档的详细信息，请参见各个方法的参考页。

### 写确认书

对于写入问题，您可以指定从MongoDB请求的写入操作的确认级别。 有关详细信息，请参见[写关注](https://docs.mongodb.com/manual/reference/write-concern/)。      

  另请参考：

- [Updates with Aggregation Pipeline](https://docs.mongodb.com/manual/tutorial/update-documents-with-aggregation-pipeline/)
- [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)
- [db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)
- [db.collection.replaceOne()](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)
- [Additional Methods](https://docs.mongodb.com/manual/reference/update-methods/#additional-updates)



译者：杨帅

校对：杨帅