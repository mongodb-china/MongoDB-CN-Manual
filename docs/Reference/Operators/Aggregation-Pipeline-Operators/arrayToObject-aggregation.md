# [ ](#)$arrayToObject (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$arrayToObject**

*3.4.4版中的新功能。*

将数组转换为单个文档；数组必须为：

* 一个由两个元素组成的数组，其中第一个元素是字段名称，第二个元素是字段值：

  ```powershell
  [ [ "item", "abc123"], [ "qty", 25 ] ]
  ```

-OR-

* 文件数组，包含两个字段，`k`并且`v` 其中：

  * 该`k`字段包含字段名称。
  * 该`v`字段包含该字段的值。

  ```powershell
  [ { "k": "item", "v": "abc123"}, { "k": "qty", "v": 25 } ]
  ```

`$arrayToObject`具有以下语法：

```powershell
{ $arrayToObject: <expression> }
```

`<expression>`可以是任何有效的表达解析为两个元件阵列或包含“k”和“V”域的文档阵列的阵列。

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

如果字段名称在数组中重复，

- 从4.0.5开始，`$arrayToObject`使用该字段的最后一个值。对于4.0.0-4.0.4，使用的值取决于驱动程序。
- 从3.6.10开始，`$arrayToObject`使用该字段的最后一个值。对于3.6.0-3.6.9，使用的值取决于驱动程序。
- 从3.4.19开始，`$arrayToObject`使用该字段的最后一个值。对于3.4.0-3.4.19，使用的值取决于驱动程序。

| 例子                                                         | 结果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| { $arrayToObject:  { $literal: [       { "k": "item", "v": "abc123"},       { "k": "qty", "v": 25 } ] } } | { "item" : "abc123", "qty" : 25 }                            |
| { $arrayToObject: { $literal:  [    [ "item", "abc123"], [ "qty", 25 ] ] } } | { "item" : "abc123", "qty" : 25 }                            |
| { $arrayToObject: { $literal: [    { "k": "item", "v": "123abc"},    { "k": "item", "v": "abc123" } ] } } | { "item" : "abc123" }<br />从版本4.0.5+（3.6.10+和3.4.19+）开始，如果字段名称在数组中重复，则`$arrayToObject` 使用该字段的最后一个值。 |

## <span id="examples">例子</span>

### `$arrayToObject` 例子

考虑`inventory`包含以下文档的集合：

```powershell
{ "_id" : 1, "item" : "ABC1",  dimensions: [ { "k": "l", "v": 25} , { "k": "w", "v": 10 }, { "k": "uom", "v": "cm" } ] }
{ "_id" : 2, "item" : "ABC2",  dimensions: [ [ "l", 50 ], [ "w",  25 ], [ "uom", "cm" ] ] }
{ "_id" : 3, "item" : "ABC3",  dimensions: [ [ "l", 25 ], [ "l",  "cm" ], [ "l", 50 ] ] }
```

以下聚合管道操作使用 `$arrayToObject`将该`dimensions`字段作为文档返回：

```powershell
db.inventory.aggregate(
   [
      {
         $project: {
            item: 1,
            dimensions: { $arrayToObject: "$dimensions" }
         }
      }
   ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "item" : "ABC1", "dimensions" : { "l" : 25, "w" : 10, "uom" : "cm" } }
{ "_id" : 2, "item" : "ABC2", "dimensions" : { "l" : 50, "w" : 25, "uom" : "cm" } }
{ "_id" : 3, "item" : "ABC3", "dimensions" : { "l" : 50 } }
```

从版本4.0.5+（3.6.10+和3.4.19+）开始，如果字段名称在数组中重复，则`$arrayToObject`使用该字段的最后一个值。

### `$objectToArray`+ `$arrayToObject`示例

考虑`inventory`包含以下文档的集合：

```powershell
{ "_id" : 1, "item" : "ABC1", instock: { warehouse1: 2500, warehouse2: 500 } }
{ "_id" : 2, "item" : "ABC2", instock: { warehouse2: 500, warehouse3: 200} }
```

以下聚合管道操作将计算每个物料的总存货并将其添加到`instock`凭证中：

```powershell
db.inventory.aggregate( [
   { $addFields: { instock: { $objectToArray: "$instock" } } },
   { $addFields: { instock: { $concatArrays: [ "$instock", [ { "k": "total", "v": { $sum: "$instock.v" } } ] ] } } } ,
   { $addFields: { instock: { $arrayToObject: "$instock" } } }
] )
```

该操作返回以下内容：

```powershell
{ "_id" : 1, "item" : "ABC1", "instock" : { "warehouse1" : 2500, "warehouse2" : 500, "total" : 3000 } }
{ "_id" : 2, "item" : "ABC2", "instock" : { "warehouse2" : 500, "warehouse3" : 200, "total" : 700 } }
```

> 也可以看看
> 
> `$objectToArray`



译者：李冠飞

校对：