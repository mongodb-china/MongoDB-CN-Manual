# [ ](#)$not

[]()

在本页面

* [行为](#behavior)

**$not**

*语法*：`{ field: { $not: { <operator-expression> } } }`

$not对指定的`<operator-expression>`执行逻辑`NOT`操作，并选择与`<operator-expression>`不匹配的文档。这包括不包含该字段的文档。

考虑以下查询：

```powershell
db.inventory.find( { price: { $not: { $gt: 1.99 } } } )
```

此查询将选择`inventory`集合中的所有文档，其中：

- `price`字段的值小于或等于`1.99` **或**
- `price`字段不存在

`{$not: {$gt: 1.99}}`与`$lte`运算符不同。`{$lte: 1.99}`只返回`price`字段存在且其值小于或等于`1.99`的文档。

请记住，`$not`操作符只影响其他操作符，不能独立地检查字段和文档。因此，使用`$not`操作符进行逻辑分离，使用`$ne`操作符直接测试字段的内容。

## <span id="behavior">行为</span>

### `$not`和数据类型

`$not`运算符的操作与其他运算符的行为一致，但对于某些数据类型（如数组）可能会产生意外结果。

### `$not`和正则表达式

`$not`操作符可以对以下内容执行逻辑`NOT`运算：

* 正则表达式对象(例如：/pattern/)

  例如：下面的查询选择`inventory`集合中`item`字段值不以字母`p`开头的所有文档。

  ```powershell
  db.inventory.find( { item: { $not: /^p.*/ } } )
  ```

* `$regex`运算符表达式(从MongoDB 4.0.7开始)

  例如，下面的查询选择`inventory`集合中`item`字段值不以字母`p`开头的所有文档。

  ```powershell
  db.inventory.find( { item: { $not: { $regex: "^p.*" } } } )
  db.inventory.find( { item: { $not: { $regex: /^p.*/ } } } )
  ```

* 驱动程序语言的正则表达式对象

  例如，下面的PyMongo查询使用Python的`re.compile()`方法编译一个正则表达式:

  ```python
  import re
  for noMatch in db.inventory.find( { "item": { "$not": re.compile("^p.*") } } ):
      print noMatch
  ```

> **也可以看看**<br />
> [`find()`](), [`update()`](), [`$set`](), [`$gt`](), [`$regex`](), [PyMongo](), [driver]().



译者：李冠飞

校对：