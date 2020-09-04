
# 查询空字段或缺少字段
MongoDB中不同的查询操作符对待**null**值是不同的。

该页面提供了使用mongo shell中的[db.collection.find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法查询空值的操作示例。 此页面上的示例使用库存收集。 要填充库存收集，请运行以下命令：

这个页面提供了使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中的 [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法查询**null**值的操作示例。此页上的示例使用 **inventory** 集合。要填充 **inventory**集合，请运行以下操作:

```powershell
db.inventory.insertMany([
   { _id: 1, item: null },
   { _id: 2 }
])
```

## 平等过滤器

**{item：null}**查询匹配包含值是**null**的**item**字段或不包含**item**字段的文档。

```shell
db.inventory.find( { item: null } )
```

该查询返回集合中的两个文档。

## 类型检查

{**item：{$ type：10}**}查询只匹配包含**item**字段值为**null**的文档； 即**item**字段的值为[`BSON Type`](https://docs.mongodb.com/manual/reference/bson-types/)为**Null**（类型编号10）：

```shell
db.inventory.find( { item : { $type: 10 } } )
```

该查询仅返回**item**字段值为**null**的文档。

## 存在检查

以下示例查询不包含字段的文档。

{**item：{$ exists：false}**}查询与不包含**item**字段的文档匹配：

```shell
db.inventory.find( { item : { $exists: false } } )
```

该查询仅返回不包含项目字段的文档。

另请参考：

[`$type`](https://docs.mongodb.com/master/reference/operator/query/type/#op._S_type) 和[`$exists`](https://docs.mongodb.com/master/reference/operator/query/exists/#op._S_exists)运算符的参考文档。



译者：杨帅

校对：杨帅

