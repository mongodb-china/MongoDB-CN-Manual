# [ ](#)$lt

[]()

**$lt**

*语法*：`{field: {$lt: value} }`

`$lt`选择的值`field`小于（即`<`）指定的文档 `value`。

对于大多数数据类型，比较运算符仅对BSON类型与查询值的类型匹配的字段执行比较 。MongoDB通过Type Bracketing支持有限的跨BSON比较。

考虑以下示例：

```powershell
db.inventory.find( { qty: { $lt: 20 } } )
```

此查询将选择`inventory`集合中`qty`字段值小于`20`的所有文档。

考虑以下示例，该示例将`$lt`运算符与嵌入式文档中的字段一起使用：

```powershell
db.inventory.update( { "carrier.fee": { $lt: 20 } }, { $set: { price: 9.99 } } )
```

`update()`操作将`price`在包含嵌入文档的文档中设置字段值，该嵌入文档`carrier`的`fee`字段值小于`20`。

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：