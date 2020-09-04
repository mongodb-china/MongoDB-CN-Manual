# [ ](#)$addToSet (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$addToSet**

返回所有*唯一*值的数组，这些值是通过将表达式应用于一组按键共享相同组的文档中的每个文档而得到的。未指定输出数组中元素的顺序。

$addToSet仅在$group阶段可用。

`$addToSet`具有以下语法：

```powershell
{ $addToSet: <expression> }
```

有关表达式的更多信息，请参阅表达式。

## <span id="behavior">行为</span>

### 数组表达式

如果表达式的 value 是 array，则$addToSet将整个 array 作为单个元素追加。

### 文档表达

如果表达式的值是一个文档，则如果数组中的另一个文档与要添加的文档完全匹配，则MongoDB将确定该文档是重复的。也就是说，现有文档具有完全相同的字段和值，并且顺序完全相同

### 内存限制

从版本4.2.3（和4.0.14、3.6.17）开始， `$addToSet`内存限制也为100 MiB（100 * 1024 * 1024），即使`db.collection.aggregate()`使用allowDiskUse：true运行 。

有关更多信息，请参见聚集管道限制。

## <span id="examples">例子</span>

考虑带有以下文档的`sales`集合：

```powershell
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") }
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") }
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") }
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") }
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
```

按`date`字段的日期和年份对文档进行分组，以下操作使用$addToSet累加器计算为每个 group 销售的唯一商品的列表：

```powershell
db.sales.aggregate(
    [
        {
            $group:{
                _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
                itemsSold: { $addToSet: "$item" }
            }
        }
    ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : { "day" : 46, "year" : 2014 }, "itemsSold" : [ "xyz", "abc" ] }
{ "_id" : { "day" : 34, "year" : 2014 }, "itemsSold" : [ "xyz", "jkl" ] }
{ "_id" : { "day" : 1, "year" : 2014 }, "itemsSold" : [ "abc" ] }
```



译者：李冠飞

校对：