# [ ](#)$avg (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$avg**

返回数值的平均值。`$avg`忽略非数字值。

`$avg`在以下阶段可用：

- `$group`
- `$project`
- `$addFields`（从MongoDB 3.4开始可用）
- `$set`（从MongoDB 4.2开始可用）
- `$replaceRoot`（从MongoDB 3.4开始可用）
- `$replaceWith`（从MongoDB 4.2开始可用）
- `$match`包含`$expr`表达的阶段

在MongoDB 3.2和更早版本中，`$avg`仅在此`$group`阶段可用 。

在`$group`阶段中使用时，`$avg`具有以下语法，并返回通过将指定的表达式应用于按键共享同一组的一组文档中的每个文档而得到的所有数值的总平均值：

```powershell
{ $avg: <expression> }
```

在其他受支持的阶段中使用时，`$avg`返回每个文档的指定表达式或表达式列表的平均值，并具有以下两种语法之一：

- `$avg` 有一个指定的表达式作为其操作数：

  ```powershell
  { $avg: <expression> }
  ```

- `$avg` 有一个指定表达式的列表作为其操作数：

  ```powershell
  { $avg: [ <expression1>, <expression2> ... ]  }
  ```

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

### 非数值或缺失值

`$avg`忽略非数字值，包括缺失值。如果平均值的所有操作数均为非数值，则由于未定义零值的平均值，因此`$avg`返回 `null`。

### 数组操作数

在此`$group`阶段，如果表达式解析为数组，`$avg`则将操作数视为非数值。

在其他受支持的阶段：

- 使用单个表达式作为其操作数，如果表达式解析为数组，则`$avg`遍历数组以对数组的数字元素进行操作以返回单个值。
- 使用表达式列表作为其操作数，如果任何表达式都解析为数组，`$avg`则**不会**遍历数组，而是将数组视为非数字值。

## <span id="examples">例子</span>

### 在`$group`阶段上使用

考虑`sales`包含以下文档的集合：

```powershell
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") }
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") }
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") }
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") }
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
```

按`item`字段对文档进行分组，以下操作使用`$avg`累加器计算每个分组的平均数量和平均数量。

```powershell
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: "$item",
           avgAmount: { $avg: { $multiply: [ "$price", "$quantity" ] } },
           avgQuantity: { $avg: "$quantity" }
         }
     }
   ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : "xyz", "avgAmount" : 37.5, "avgQuantity" : 7.5 }
{ "_id" : "jkl", "avgAmount" : 20, "avgQuantity" : 1 }
{ "_id" : "abc", "avgAmount" : 60, "avgQuantity" : 6 }
```

### 在`$project`阶段上使用

集合`students`包含以下文档：

```powershell
{ "_id": 1, "quizzes": [ 10, 6, 7 ], "labs": [ 5, 8 ], "final": 80, "midterm": 75 }
{ "_id": 2, "quizzes": [ 9, 10 ], "labs": [ 8, 8 ], "final": 95, "midterm": 80 }
{ "_id": 3, "quizzes": [ 4, 5, 5 ], "labs": [ 6, 5 ], "final": 78, "midterm": 70 }
```

以下示例`$avg`在 `$project`阶段中使用来计算平均测验分数，平均实验室分数以及期末和期中考试的平均值：

```powershell
db.students.aggregate([
   { $project: { quizAvg: { $avg: "$quizzes"}, labAvg: { $avg: "$labs" }, examAvg: { $avg: [ "$final", "$midterm" ] } } }
])
```

该操作产生以下文档：

```powershell
{ "_id" : 1, "quizAvg" : 7.666666666666667, "labAvg" : 6.5, "examAvg" : 77.5 }
{ "_id" : 2, "quizAvg" : 9.5, "labAvg" : 8, "examAvg" : 87.5 }
{ "_id" : 3, "quizAvg" : 4.666666666666667, "labAvg" : 5.5, "examAvg" : 74 }
```

在其他受支持的阶段：

- 使用单个表达式作为其操作数，如果表达式解析为数组，则`$avg`遍历数组以对数组的数字元素进行操作以返回单个值。
- 使用表达式列表作为其操作数，如果任何表达式都解析为数组，`$avg`则**不会**遍历数组，而是将数组视为非数字值。



译者：李冠飞

校对：