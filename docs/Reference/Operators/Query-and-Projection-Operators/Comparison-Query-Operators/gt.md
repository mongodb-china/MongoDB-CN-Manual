# [ ](#)$gt

[]()

**$gt**

*语法*：`{field: {$gt: value} }`

`$gt`选择的值`field`大于（即`>`）指定的那些文档 `value`。

对于大多数数据类型，比较运算符仅对BSON类型与查询值的类型匹配的字段执行比较 。MongoDB通过Type Bracketing支持有限的跨BSON比较。

考虑以下示例：

```powershell
db.inventory.find( { qty: { $gt: 20 } } )
```

此查询将选择`inventory`集合中`qty`字段值大于`20`的所有文档。

考虑以下示例，该示例将`$gt`运算符与嵌入式文档中的字段一起使用：

```powershell
db.inventory.update( { "carrier.fee": { $gt: 2 } }, { $set: { price: 9.99 } } )
```

`update()`操作将设置`price`找到的第一个文档中包含嵌入文档`carrier`的`fee`字段的值，该嵌入文档的字段值大于`2`。

要`price`在包含嵌入文档的*所有*文档中设置该字段的值，该嵌入文档`carrier`的`fee`字段值大于`2`，请在`update()`方法中指定`multi:true`选项：

```powershell
db.inventory.update(
   { "carrier.fee": { $gt: 2 } },
   { $set: { price: 9.99 } },
   { multi: true }
)
```

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：