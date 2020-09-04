# [ ](#)$nor

[]()

在本页面

* [例子](#examples)

**$nor**

`$nor`对包含一个或多个查询表达式的数组执行逻辑`nor`操作，并选择数组中所有查询表达式失败的文档。`$nor`有以下语法:

```powershell
{ $nor: [ { <expression1> }, { <expression2> }, ...  { <expressionN> } ] }
```

> **也可以看看**
>
> [`find()`](), [`update()`](), [`$or`](), [`$set`](), and [`$exists`]().

## <span id="examples">例子</span>

### `$nor`查询有两种表述

考虑以下仅使用`$nor`操作符的查询:

```powershell
db.inventory.find( { $nor: [ { price: 1.99 }, { sale: true } ]  } )
```

此查询将返回以下所有文档:

* 包含值不等于`1.99`的`price`字段和不等于`true`或的`sale`字段
* 包含`price`字段，其值不等于`1.99`，但不包含`sale`字段
* 不包含`price`字段，但`sale`包含值不等于`true`
* 不包含`price`字段和不包含`sale`字段

### `$nor`和另外的比较

考虑以下查询:

```powershell
db.inventory.find( { $nor: [ { price: 1.99 }, { qty: { $lt: 20 } }, { sale: true } ] } )
```

此查询将选择库存集合中的所有文档，其中:

* `price`字段值不等于`1.99`和
* `qty`字段值不小于`20`和
* `sales`字段的值不等于`true`

包括那些不包含这些字段的文档。

返回不包含`$nor`表达式中字段的文档的例外情况是，`$nor`操作符与`$exists`操作符一起使用。

### `$nor`和`$exists`

下面的查询使用`$nor`操作符和`$exists`操作符:

```powershell
db.inventory.find( { $nor: [ { price: 1.99 }, { price: { $exists: false } },
                             { sale: true }, { sale: { $exists: false } } ] } )
```

此查询将返回以下所有文档:

包含值不等于`1.99`的`price`字段和值不等于`true`的`sale`字段



译者：李冠飞

校对：