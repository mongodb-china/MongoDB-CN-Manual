# [ ](#)$lte

[]()

**$lte**

*语法*：`{field: {$lte: value} }`

`$lte`选择的值`field`小于或等于`<=`指定`value`的文档 。

对于大多数数据类型，比较运算符仅对BSON类型与查询值的类型匹配的字段执行比较 。MongoDB通过Type Bracketing支持有限的跨BSON比较。

考虑以下示例：

```powershell
db.inventory.find( { qty: { $lte: 20 } } )
```

此查询将选择`inventory`集合中`qty`字段值小于或等于`20`的所有文档。

考虑以下示例，该示例将`$lte`运算符与嵌入式文档中的字段一起使用：

```powershell
db.inventory.update( { "carrier.fee": { $lte: 5 } }, { $set: { price: 9.99 } } )
```

`update()`操作将`price`在包含嵌入式文档的文档中设置字段值，该嵌入式文档`carrier`的`fee`字段值小于或等于`5`。

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：