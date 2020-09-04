# [ ](#)$or

[]()

在本页面

* [行为](#behavior)

**$or**

`$or`操作符对包含两个或多个`<expressions>`的数组执行逻辑或操作，并选择满足至少一个`<expressions>`的文档。`$or`的语法如下:

```powershell
{ $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }
```

考虑下面的例子:

```powershell
db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
```

该查询将选择`inventory`集合中`quantity`字段值小于`20`或`price`字段值等于`10`的所有文档。

## <span id="behavior">行为</span>

### `$or`子句和索引

当对`$or`表达式中的子句求值时，MongoDB要么执行集合扫描，要么执行索引扫描(如果所有子句都被索引支持)。也就是说，MongoDB要使用索引对`$or`表达式求值，索引必须支持`$or`表达式中的所有子句。否则，MongoDB将执行一次收集扫描。

当对`$or`查询使用索引时，`$or`的每个子句都可以使用自己的索引。考虑以下查询:

```powershell
db.inventory.find( { $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] } )
```

为了支持此查询，而不是复合索引，您将创建一个关于`quantity`的索引和另一个关于`price`的索引:

```powershell
db.inventory.createIndex( { quantity: 1 } )
db.inventory.createIndex( { price: 1 } )
```

MongoDB可以使用除`geoHaystack`索引之外的所有索引来支持`$or`子句。

### `$not`和正则表达式

如果`$or`包含`$text`查询，则`$or`数组中的所有子句必须由索引支持。这是因为`$text`查询必须使用索引，而`$or`只能在索引支持其所有子句的情况下使用索引。如果`$text`查询不能使用索引，则查询将返回一个错误。

### `$or`和地理空间查询

`$or`支持地理空间子句，但`near`子句有以下例外(`near`子句包括`$nearSphere`和`$near`)。`$or`不能包含任何其他子句的`near`子句。

### `$or`和排序操作

当使用`sort()`执行`$or`查询时，MongoDB现在可以使用支持`$or`子句的索引。以前的版本不使用索引。

### `$or`与`$in`

当使用`$or` `<expressions>`来检查相同字段的值时，使用`$in`操作符而不是`$or`操作符。

例如，要选择数量字段值为`20`或`50`的库存集合中的所有文档，使用`$in`操作符:

```powershell
db.inventory.find ( { quantity: { $in: [20, 50] } } )
```

### Nested `$or` Clauses

你可能会嵌套`$or`操作。

> **也可以看看**
>
> [`$and`](), [`find()`](), [`sort()`](), [`$in`]()



译者：李冠飞

校对：