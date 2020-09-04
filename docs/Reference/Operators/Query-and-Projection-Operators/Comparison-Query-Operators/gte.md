# [ ](#)$gte

[]()

**$gte**

*语法*：`{field: {$gte: value} }`

`$gte`选择的值`field`大于或等于（即`>=`）指定值（例如`value`）的文档 。

对于大多数数据类型，比较运算符仅对BSON类型与查询值的类型匹配的字段执行比较 。MongoDB通过Type Bracketing支持有限的跨BSON比较。

考虑以下示例：

```powershell
db.inventory.find( { qty: { $gte: 20 } } )
```

此查询将选择的所有文件`inventory`，其中`qty`字段的值大于或等于`20`。

考虑以下示例，该示例将`$gte`运算符与嵌入式文档中的字段一起使用：

```powershell
db.inventory.update( { "carrier.fee": { $gte: 2 } }, { $set: { price: 9.99 } } )
```

`update()`操作将设置`price`包含嵌入文档`carrier`的`fee`字段的值，该嵌入文档 的字段值大于或等于 `2`。

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：