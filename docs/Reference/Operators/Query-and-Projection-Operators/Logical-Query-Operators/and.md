# [ ](#)$and

[]()

在本页面

* [例子](#examples)

**$and**

*语法*：`{ $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }`

`$and`执行的逻辑`AND`的阵列上操作*的一个或多个*表达式（例如`<expression1>`， `<expression2>`等），并且选择满足该文件 *的所有*阵列中的表达式。`$and`运算符使用*短路计算*。如果第一个表达式（例如`<expression1>`）的计算结果为`false`，则MongoDB将不计算其余的表达式。

> **注意**
>
> `AND`当指定逗号分隔的表达式列表时，MongoDB提供隐式操作。

## <span id="examples">例子</span>

### `AND`使用指定同一字段的多个表达式进行查询

考虑以下示例：

```powershell
db.inventory.find( { $and: [ { price: { $ne: 1.99 } }, { price: { $exists: true } } ] } )
```

此查询将选择`inventory` 集合中的所有文档，其中：

- `price`字段值不等于`1.99` **与**
- `price`字段存在。

`AND` 通过组合`price` 字段的运算符表达式，也可以使用隐式操作构造此查询。例如，此查询可以写为：

```powershell
db.inventory.find( { price: { $ne: 1.99, $exists: true } } )
```

### `AND`使用指定相同运算符的多个表达式进行查询

考虑以下示例：

```powershell
db.inventory.find( {
    $and: [
        { $or: [ { qty: { $lt : 10 } }, { qty : { $gt: 50 } } ] },
        { $or: [ { sale: true }, { price : { $lt : 5 } } ] }
    ]
} )
```

该查询将选择以下位置的所有文档：

- `qty`字段值小于`20`或大于`50`，**和**
- `sale`字段值是等于`true` **或**所述`price` 字段值小于`5`。

无法使用隐式`AND`操作构造此查询，因为它`$or`多次使用运算符。

> **也可以看看**
>
> [`find()`]()，[`update()`]()， [`$ne`]()，[`$exists`]()，[`$set`]()。



译者：李冠飞

校对：