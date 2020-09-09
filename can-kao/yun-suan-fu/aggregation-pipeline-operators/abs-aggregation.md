# $abs \(aggregation\)

在本页面

* [定义](abs-aggregation.md#definition)
* [行为](abs-aggregation.md#behavior)
* [例子](abs-aggregation.md#example)

## 定义

**$abs**

version 3.2 中的新内容。

返回数字的绝对 value。

`$abs`具有以下语法：

```text
{ $abs: <number> }
```

`<number>`表达式可以是任何有效的表达，因为它解析为数字。有关表达式的更多信息，请参阅表达式。

## 行为

如果参数解析为的值或引用缺少的字段，则`$abs`返回`null`。如果参数解析为`NaN`，则`$abs`返回`NaN`。

| 例子 | 结果 |
| :--- | :--- |
| `{ $abs: -1 }` | `1` |
| `{ $abs: 1 }` | `1` |
| `{ $abs: null }` | `null` |

## 例子

集合`ratings`包含以下文档：

```text
{ _id: 1, start: 5, end: 8 }
{ _id: 2, start: 4, end: 4 }
{ _id: 3, start: 9, end: 7 }
{ _id: 4, start: 6, end: 7 }
```

以下 example 计算`start`和`end`评级之间的差异大小：

```text
db.ratings.aggregate([
   {
       $project: { delta: { $abs: { $subtract: [ "$start", "$end" ] } } }
   }
])
```

该操作返回以下结果：

```text
{ "_id" : 1, "delta" : 3 }
{ "_id" : 2, "delta" : 0 }
{ "_id" : 3, "delta" : 2 }
{ "_id" : 4, "delta" : 1 }
```

译者：李冠飞

校对：

