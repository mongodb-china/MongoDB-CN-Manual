# $arrayElemAt \(aggregation\)

在本页面

* [定义](arrayelemat-aggregation.md#definition)
* [行为](arrayelemat-aggregation.md#behavior)
* [例子](arrayelemat-aggregation.md#examples)

## 定义

**$arrayElemAt**

_3.2版中的新功能。_

返回指定数组索引处的元素。

`$arrayElemAt` 具有以下语法：

```text
{ $anyElement: [ <expression> ] }
```

`<array>`表达式可以是任何有效的表达式，只要它可以解析为数组。

`<idx>`表达式可以是任何有效表达式，只要它可以解析为整数。

* 如果为正，则从数组开始算起`$arrayElemAt`返回该`idx`位置的元素 。
* 如果为负，则从数组末尾算起`$arrayElemAt`返回该`idx`位置处的元素 。

如果`idx`超过了数组界限，`$arrayElemAt`则不返回任何结果。

有关表达式的更多信息，请参见 表达式。

## 行为

有关表达式的更多信息，请参见 表达式。

| 例子 | 结果 |
| :--- | :--- |
| { $arrayElemAt: \[ \[ 1, 2, 3 \], 0 \] } | 1 |
| { $arrayElemAt: \[ \[ 1, 2, 3 \], -2 \] } | 2 |
| { $arrayElemAt: \[ \[ 1, 2, 3 \], 15 \] } |  |

## 例子

名为的集合`users`包含以下文档：

```text
{ "_id" : 1, "name" : "dave123", favorites: [ "chocolate", "cake", "butter", "apples" ] }
{ "_id" : 2, "name" : "li", favorites: [ "apples", "pudding", "pie" ] }
{ "_id" : 3, "name" : "ahn", favorites: [ "pears", "pecans", "chocolate", "cherries" ] }
{ "_id" : 4, "name" : "ty", favorites: [ "ice cream" ] }
```

下面的示例返回`favorites`数组中的第一个和最后一个元素 ：

```text
db.users.aggregate([
   {
     $project:
      {
         name: 1,
         first: { $arrayElemAt: [ "$favorites", 0 ] },
         last: { $arrayElemAt: [ "$favorites", -1 ] }
      }
   }
])
```

该操作返回以下结果：

```text
{ "_id" : 1, "name" : "dave123", "first" : "chocolate", "last" : "apples" }
{ "_id" : 2, "name" : "li", "first" : "apples", "last" : "pie" }
{ "_id" : 3, "name" : "ahn", "first" : "pears", "last" : "cherries" }
{ "_id" : 4, "name" : "ty", "first" : "ice cream", "last" : "ice cream" }
```

译者：李冠飞

校对：

