# [ ](#)$concatArrays (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$concatArrays**

*3.2版中的新功能。*

连接数组以返回连接的数组。

`$concatArrays`具有以下语法：

```powershell
{ $concatArrays: [ <array1>, <array2>, ... ] }
```

该`<array>`表达式可以是任何有效的表达式，只要它们解析为一个数组。有关表达式的更多信息，请参见表达式。

如果有任何参数解析为`null`或指向缺少的字段，则`$concatArrays`返回`null`。

## <span id="behavior">行为</span>

| 例子                                                         | 结果                                   |
| ------------------------------------------------------------ | -------------------------------------- |
| { $concatArrays: [    [ "hello", " "], [ "world" ] ] }       | [ "hello", " ", "world" ]              |
| { $concatArrays: [    [ "hello", " "],    [ [ "world" ], "again"] ] } | [ "hello", " ", [ "world" ], "again" ] |

## <span id="example">例子</span>

名为的集合`warehouses`包含以下文档：

```powershell
{ "_id" : 1, instock: [ "chocolate" ], ordered: [ "butter", "apples" ] }
{ "_id" : 2, instock: [ "apples", "pudding", "pie" ] }
{ "_id" : 3, instock: [ "pears", "pecans"], ordered: [ "cherries" ] }
{ "_id" : 4, instock: [ "ice cream" ], ordered: [ ] }
```

以下示例将`instock`和`ordered` 数组串联在一起：

```powershell
db.warehouses.aggregate([
   { $project: { items: { $concatArrays: [ "$instock", "$ordered" ] } } }
])
```

```powershell
{ "_id" : 1, "items" : [ "chocolate", "butter", "apples" ] }
{ "_id" : 2, "items" : null }
{ "_id" : 3, "items" : [ "pears", "pecans", "cherries" ] }
{ "_id" : 4, "items" : [ "ice cream" ] }
```

> 也可以看看
> 
> `$push`



译者：李冠飞

校对：