# [ ](#)$concat (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [例子](#example)

## <span id="definition">定义</span>

**$concat**

连接字符串并返回连接的字符串。

`$concat`具有以下语法：

```powershell
{ $concat: [ <expression1>, <expression2>, ... ] }
```

参数可以解析为字符串，可以是任何有效的表达式。有关表达式的更多信息，请参见 表达式。

如果参数解析为的值`null`或指向缺少的字段，则`$concat`返回`null`。

## <span id="example">例子</span>

考虑`inventory`包含以下文档的集合：

```powershell
{ "_id" : 1, "item" : "ABC1", quarter: "13Q1", "description" : "product 1" }
{ "_id" : 2, "item" : "ABC2", quarter: "13Q4", "description" : "product 2" }
{ "_id" : 3, "item" : "XYZ1", quarter: "14Q2", "description" : null }
```

以下操作使用`$concat`运算符将`item`字段和`description`带有“-”定界符的字段连接起来。

```powershell
db.inventory.aggregate(
   [
      { $project: { itemDescription: { $concat: [ "$item", " - ", "$description" ] } } }
   ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "itemDescription" : "ABC1 - product 1" }
{ "_id" : 2, "itemDescription" : "ABC2 - product 2" }
{ "_id" : 3, "itemDescription" : null }
```



译者：李冠飞

校对：