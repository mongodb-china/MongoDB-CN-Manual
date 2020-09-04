# [ ](#)$cmp (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [例子](#example)

## <span id="definition">定义</span>

**$cmp**

比较两个值并返回：

- `-1` 如果第一个值小于第二个值。
- `1` 如果第一个值大于第二个值。
- `0` 如果两个值相等。

在`$cmp`使用两个值和类型进行比较， 指定比较BSON为了 用于不同类型的值。

`$cmp`具有以下语法：

```powershell
{ $cmp: [ <expression1>, <expression2> ] }
```

有关表达式的更多信息，请参见表达式。

## <span id="example">例子</span>

考虑包含`inventory`以下文档的集合：

```powershell
{ "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 }
{ "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 }
{ "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 }
{ "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 }
{ "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }
```

以下操作使用`$cmp`运算符将`qty`值与进行比较`250`：

```powershell
db.inventory.aggregate(
   [
     {
       $project:
          {
            item: 1,
            qty: 1,
            cmpTo250: { $cmp: [ "$qty", 250 ] },
            _id: 0
          }
     }
   ]
)
```

该操作返回以下结果：

```powershell
{ "item" : "abc1", "qty" : 300, "cmpTo250" : 1 }
{ "item" : "abc2", "qty" : 200, "cmpTo250" : -1 }
{ "item" : "xyz1", "qty" : 250, "cmpTo250" : 0 }
{ "item" : "VWZ1", "qty" : 300, "cmpTo250" : 1 }
{ "item" : "VWZ2", "qty" : 180, "cmpTo250" : -1 }
```



译者：李冠飞

校对：