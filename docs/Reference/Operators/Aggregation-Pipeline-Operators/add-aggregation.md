# [ ](#)$add (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [例子](#examples)

## <span id="definition">定义</span>

**$add**

添加数字或添加数字和日期。如果其中一个参数是 date，则$add将其他参数视为要添加到 date 的毫秒数。

$add表达式具有以下语法：

```powershell
{ $add: [ <expression1>, <expression2>, ... ] }
```

参数可以是任何有效的表达，只要它们可以解析为所有数字或数字和日期。有关表达式的更多信息，请参阅表达式。

## <span id="examples">例子</span>

以下示例使用带有以下文档的`sales`集合：

```powershell
{ "_id" : 1, "item" : "abc", "price" : 10, "fee" : 2, date: ISODate("2014-03-01T08:00:00Z") }
{ "_id" : 2, "item" : "jkl", "price" : 20, "fee" : 1, date: ISODate("2014-03-01T09:00:00Z") }
{ "_id" : 3, "item" : "xyz", "price" : 5,  "fee" : 0, date: ISODate("2014-03-15T09:00:00Z") }
```

### 添加数字

以下聚合使用$project管道中的$add表达式来计算总成本：

```powershell
db.sales.aggregate(
    [
        { $project: { item: 1, total: { $add: [ "$price", "$fee" ] } } }
    ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "item" : "abc", "total" : 12 }
{ "_id" : 2, "item" : "jkl", "total" : 21 }
{ "_id" : 3, "item" : "xyz", "total" : 5 }
```

### 在 Date 上执行加法

以下聚合使用$add表达式`billing_date`通过将`3*24*60*60000`毫秒(即：3 天)添加到`date`字段来计算：

```powershell
db.sales.aggregate(
    [
        { $project: { item: 1, billing_date: { $add: [ "$date", 3*24*60*60000 ] } } }
    ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "item" : "abc", "billing_date" : ISODate("2014-03-04T08:00:00Z") }
{ "_id" : 2, "item" : "jkl", "billing_date" : ISODate("2014-03-04T09:00:00Z") }
{ "_id" : 3, "item" : "xyz", "billing_date" : ISODate("2014-03-18T09:00:00Z") }
```



译者：李冠飞

校对：