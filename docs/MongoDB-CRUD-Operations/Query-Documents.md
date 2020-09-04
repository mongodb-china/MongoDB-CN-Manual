# 查询文档
这个页面提供了使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中的[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法的查询操作示例。此页上的示例使用**inventory**集合。要填充**inventory**集合，请运行以下操作:

```shell
db.inventory.insertMany([
{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
{ item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
{ item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
{ item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
{ item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```

## 选择集合中的所有文档

要选择集合中的所有文档，请将空文档作为查询过滤器参数传递给**find**方法。 查询过滤器参数确定选择条件：

```shell
db.inventory.find( {} )
```

此操作对应于以下SQL语句：

```sql
SELECT * FROM inventory
```

有关该方法的语法的更多信息，请参见[find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)。

## 指定平等条件

要指定相等条件，请在[查询筛选文档](https://docs.mongodb.com/manual/core/document/#document-query-filter)使用**<`field`>：<`value`>**表达式：

```shell
{ <field1>: <value1>, ... }
```

下面的示例从inventory中选择状态等于**" D"**的所有文档：

```shell
db.inventory.find( { status: "D" } )
```

此操作对应于以下SQL语句：

```sql
SELECT * FROM inventory WHERE status = "D"
```

## 使用查询运算符指定条件

查询过滤器文档可以使用查询运算符以以下形式指定条件：

```shell
{ <field1>: { <operator1>: <value1> }, ... }
```

下面的例子从状态等于**" A"**或**" D"**的**inventory**集合中检索所有文档:

```shell
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```

> **[success] Note**
>
> 尽管可以使用[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or) 操作符表示此查询，但在对同一字段执行相等性检查时，请使用 [`$in`](https://docs.mongodb.com/master/reference/operator/query/in/#op._S_in)操作符而不是[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or)操作符。

该操作对应于以下SQL语句：

```sql
SELECT * FROM inventory WHERE status in ("A", "D")
```

有关MongoDB查询运算符的完整列表，请参阅[查询和投影运算符文档](https://docs.mongodb.com/master/reference/operator/query/)。

## 指定和条件

复合查询可以为集合文档中的多个字段指定条件。逻辑和连词隐式地连接复合查询的子句，以便查询在集合中选择符合所有条件的文档。

下面的示例 `inventory` 状态为**"A"**且数量小于([`$lt`](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt)) 30的库存集合中的所有文档:

```shell
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```

该操作对应于以下SQL语句:

```sql
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```

有关其他MongoDB比较运算符，请参阅[比较运算符 ](https://docs.mongodb.com/master/reference/operator/query-comparison/#query-selectors-comparison) 。

## 指定或条件

使用[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or)操作符，可以指定一个复合查询，用逻辑**OR**连词连接每个子句，以便查询在集合中选择至少匹配一个条件的文档。

下面的示例**retrieve**状态为**“A”**或**qty**小于([`$lt`](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt))30的集合中的所有文档:

```powershell
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```

该操作对应于以下SQL语句:

```sql
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```

>**[success] Note**
>
>使用[比较运算符](https://docs.mongodb.com/master/reference/operator/query-comparison/#query-selectors-comparison)的查询需要使用[括号类型](https://docs.mongodb.com/master/reference/method/db.collection.find/#type-bracketing)。

### 指定和以及或条件

在下面的例子中，复合查询文档选择状态为**“A”**且**qty**小于([`$lt`](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt)) 30或**item**以字符**p**开头的集合中的所有文档:

```powershell
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```

该操作对应于以下SQL语句:

```sql
SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
```

>**[success] Note**
>
>MongoDB支持正则表达式[`$regex`](https://docs.mongodb.com/master/reference/operator/query/regex/#op._S_regex)查询来执行字符串模式匹配。

## 附加查询教程

有关其他查询示例，请参见：

- [Query on Embedded/Nested Documents](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)
- [Query an Array](https://docs.mongodb.com/manual/tutorial/query-arrays/)
- [Query an Array of Embedded Documents](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)
- [Project Fields to Return from Query](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/)
- [Query for Null or Missing Fields](https://docs.mongodb.com/manual/tutorial/query-for-null-fields/)

## 行为

### 游标

[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法将[游标](https://docs.mongodb.com/master/tutorial/iterate-a-cursor/) 返回到匹配的文档。

### 读取隔离

*3.2版中的新功能*

对于[复制集](https://docs.mongodb.com/master/replication/) 和复制集[分片](https://docs.mongodb.com/master/sharding/)的读取，读取关注允许客户端为其读取选择隔离级别。 有关更多信息，请参见阅读关注。

## 附加方法

以下方法也可以从集合中读取文档：

- [db.collection.findOne](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#db.collection.findOne)
- 在[聚合管道](https://docs.mongodb.com/master/core/aggregation-pipeline/)中，[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match) 管道步骤提供对MongoDB查询的访问.

> **[success] Note**
>
> [`db.collection.findOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#db.collection.findOne)方法还执行读取操作以返回单个文档。在内部，[`db.collection.findOne()`](https://docs.mongodb.com/master/reference/method/db.collection.findOne/#db.collection.findOne)方法是 [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) 方法，限制为1。



译者：杨帅

校对：杨帅
