# [ ](#)$and (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$and**

计算一个或多个表达式，如果所有表达式都为`true`，或者如果没有参数表达式调用，则返回`true`。否则，`$and`返回`false`。

`$and` 具有以下语法：

```powershell
{ $and: [ <expression1>, <expression2>, ... ] }
```

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

`$and`使用短路逻辑：遇到第一个`false`表达式后，运算将停止评估。

除了`false`布尔值，`$and`计算为`false`如下：`null`，`0`，和`undefined` 的值。在`$and`评估所有其它值`true`，包括非零数值和阵列。

| 例子                                     | 结果 |
| ---------------------------------------- | ---- |
| { $and: [ 1, "green" ] }                 | true |
| { $and: [ ] }                            | true |
| { $and: [ [ null ], [ false ], [ 0 ] ] } | true |
| { $and: [ null, true ] }                 | true |
| { $and: [ 0, true ] }                    | true |

## <span id="examples">例子</span>

`inventory`使用以下文档创建示例集合：

```powershell
db.inventory.insertMany([
   { "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 },
   { "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 },
   { "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 },
   { "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 },
   { "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
])
```

以下操作使用`$and`运算符确定是否`qty`大于100 *并*小于`250`：

```powershell
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            result: { $and: [ { $gt: [ "$qty", 100 ] }, { $lt: [ "$qty", 250 ] } ] }
          }
     }
   ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "item" : "abc1", "qty" : 300, "result" : false }
{ "_id" : 2, "item" : "abc2", "qty" : 200, "result" : true }
{ "_id" : 3, "item" : "xyz1", "qty" : 250, "result" : false }
{ "_id" : 4, "item" : "VWZ1", "qty" : 300, "result" : false }
{ "_id" : 5, "item" : "VWZ2", "qty" : 180, "result" : true }
```



译者：李冠飞

校对：