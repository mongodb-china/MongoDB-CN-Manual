# [ ](#)$ceil (aggregation)
[]()
在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$ceil**

*3.2版中的新功能。*

返回大于或等于指定数字的最小整数。

`$ceil`具有以下语法：

```powershell
{ $ceil: <number> }
```

`<number>`表达式可以是任何有效的表达，因为它解析为数字。有关表达式的更多信息，请参阅表达式。

## <span id="behavior">行为</span>

如果参数解析为的值或引用缺少的字段，则`$ceil`返回`null`。如果参数解析为`NaN`，则`$ceil`返回`NaN`。

| 例子              | 结果 |
| ----------------- | ---- |
| `{ $ceil: 1 }`    | `1`  |
| `{ $ceil: 7.80 }` | `8`  |
| `{ $ceil: -2.8 }` | `-2` |

## <span id="example">例子</span>

名为`samples`的集合包含以下文档：

```powershell
{ _id: 1, value: 9.25 }
{ _id: 2, value: 8.73 }
{ _id: 3, value: 4.32 }
{ _id: 4, value: -5.34 }
```

以下事例返回原始值和上限值：

```powershell
db.samples.aggregate([
    { $project: { value: 1, ceilingValue: { $ceil: "$value" } } }
])
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "value" : 9.25, "ceilingValue" : 10 }
{ "_id" : 2, "value" : 8.73, "ceilingValue" : 9 }
{ "_id" : 3, "value" : 4.32, "ceilingValue" : 5 }
{ "_id" : 4, "value" : -5.34, "ceilingValue" : -5 }
```



译者：李冠飞

校对：