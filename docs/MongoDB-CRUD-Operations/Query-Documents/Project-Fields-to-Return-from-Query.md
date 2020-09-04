# 从查询返回的项目字段
默认情况下，MongoDB中的查询返回匹配文档中的所有字段。 要限制MongoDB发送给应用程序的数据量，可以包含一个[`projection`](https://docs.mongodb.com/master/reference/glossary/#term-projection) 文档以指定或限制要返回的字段。

本页提供使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中的[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法进行[`projection`](https://docs.mongodb.com/master/reference/glossary/#term-projection) 的查询操作示例。 此页面上的示例使用**inventory**收集。 要填充**inventory**收集，请运行以下命令：

```shell
db.inventory.insertMany( [ 
	{ item: "journal", status: "A", size: { h: 14, w: 21, uom: "cm" }, instock: [ { warehouse: "A", qty: 5 } ] },
	{ item: "notebook", status: "A",  size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "C", qty: 5 } ] },
	{ item: "paper", status: "D", size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "A", qty: 60 } ] },
	{ item: "planner", status: "D", size: { h: 22.85, w: 30, uom: "cm" }, instock: [ { warehouse: "A", qty: 40 } ] },
	{ item: "postcard", status: "A", size: { h: 10, w: 15.25, uom: "cm" }, instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```

## 返回匹配文档中的所有字段

如果未指定[projection](https://docs.mongodb.com/master/reference/glossary/#term-projection)文档，则 [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法将返回匹配文档中的所有字段。

下面的例子返回**inventory**集合中状态为**“A”**的所有文档中的所有字段:

```shell
db.inventory.find( { status: "A" } )
```

该操作对应于以下SQL语句：

```sql
SELECT * from inventory WHERE status = "A"
```

## 仅返回指定的字段和_id字段

通过将投影文档中的**<`field`>**设置为1，投影可以显式包括多个字段。 以下操作返回与查询匹配的所有文档。在结果集中，只有项目、状态和_id字段(默认情况下)在匹配的文档中返回。

```shell
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )
```

该操作对应于以下SQL语句：

```sql
SELECT _id, item, status from inventory WHERE status = "A"
```

## 禁止`_id` Field

您可以通过在投影中将**_id**字段设置为**0**来从结果中删除，如下面的示例所示:

```shell
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )
```

该操作对应于以下SQL语句:

```sql
SELECT item, status from inventory WHERE status = "A"
```

> **[success] Note**
>
> 除了**_id**字段之外，您不能在projection文档中合并包含和排除语句。

## 返回除了被排除的字段之外的所有字段

您可以使用projection来排除特定的字段，而不是列出要在匹配的文档中返回的字段。下面的例子将返回匹配文档中除了**status**和**instock**字段之外的所有字段:

```powershell
db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )
```

> 注意
>
> 除了**_id**字段之外，您不能在projection文档中合并包含和排除语句。

## 返回嵌入式文档中的特定字段

您可以返回嵌入式文档中的特定字段。 使用[点表示法](https://docs.mongodb.com/master/core/document/#document-dot-notation)引用嵌入式字段，并在投影文档中将其设置为**1**。

以下示例返回：

* _id字段(默认情况下返回).

* 项目字段.

* 状态字段.

* 大小文档中的uom字段.

**uom**字段仍然嵌入在**size**文档中。

```shell
db.inventory.find(
  { status: "A" },
  { item: 1, status: 1, "size.uom": 1 }
 )
```

从MongoDB 4.4开始，你也可以使用嵌套的形式指定嵌入的字段，例如**{item: 1, status: 1, size: {uom: 1}}**。

## 禁止嵌入文档中的特定字段

您可以隐藏嵌入式文档中的特定字段。 使用[点表示法](https://docs.mongodb.com/master/core/document/#document-dot-notation)引用projection文档中的嵌入字段并将其设置为0。

以下示例指定一个投影，以排除**size**文档内的**uom**字段。 其他所有字段均在匹配的文档中返回：

  ```shell
 db.inventory.find( 
  	{ status: "A" },
  	{ "size.uom": 0 }
 )
  ```

从MongoDB 4.4开始，你也可以使用嵌套的形式指定嵌入的字段，例如**{size: {uom: 0}}**。

 ## 数组中嵌入式文档的Projection

使用[点表示法](https://docs.mongodb.com/master/core/document/#document-dot-notation)可将特定字段projection在嵌入数组的文档中。

以下示例指定要返回的projection：

  * _id字段（默认情况下返回）
  * 项目字段
  * 状态字段
  * **instock**数组中嵌入的文档中的数量字段

 ```shell
 db.inventory.find( { status: "A" }, { item: 1, status: 1, "instock.qty": 1 } )
 ```

## 返回数组中的项目特定数组元素

 对于包含数组的字段，MongoDB提供以下用于操纵数组的投影运算符:[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/projection/elemMatch/#proj._S_elemMatch)，[`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice)和[`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_)。

以下示例使用[`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice)projection运算符返回**instock**数组中的最后一个元素：

 ```shell
db.inventory.find( { status: "A" }, { item: 1, status: 1, instock: { $slice: -1 } } )
 ```

[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/projection/elemMatch/#proj._S_elemMatch)，[`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice)和[`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_)是projection要包含在返回数组中的特定元素的唯一方法。 例如，您不能使用数组索引来投影特定的数组元素。 例如 **{“ instock.0”：1}**projection不会projection第一个元素的数组。

## 额外的注意事项

从MongoDB 4.4开始，MongoDB对projections施加了额外的限制。有关详细信息，请参阅[`Projection Restrictions`](https://docs.mongodb.com/master/reference/limits/#Projection-Restrictions) 。

 另请参考：

- [Projection](https://docs.mongodb.com/master/reference/method/db.collection.find/#find-projection)
- [Query Documents](https://docs.mongodb.com/master/tutorial/query-documents/)

   

译者：杨帅

校对：杨帅