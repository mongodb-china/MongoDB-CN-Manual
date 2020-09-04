# [ ](#)$ne

[]()

**$ne**

*语法*：`{field: {$ne: value} }`

`$ne`选择的值`field`不等于指定的文档 `value`。这包括不包含的文档`field`。

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

考虑以下示例：

```powershell
db.inventory.find( { qty: { $ne: 20 } } )
```

此查询将选择`inventory`集合中`qty`字段值不等于`20`的所有文档，包括不包含该`qty`字段的那些文档。

考虑以下示例，该示例`$ne`在嵌入式文档中使用运算符和字段：

```powershell
db.inventory.update( { "carrier.state": { $ne: "NY" } }, { $set: { qty: 20 } } )
```

`update()`操作将`qty`在包含嵌入式文档的文档中设置字段值，该嵌入式文档`carrier`的`state`字段值不等于“ NY”，或者该`state`字段或`carrier`嵌入式文档不存在。

不等式操作符`$ne`是*不*非常有选择性的，因为它往往是指数的很大一部分相匹配。结果，在许多情况下，`$ne`带有索引的查询的性能可能不比`$ne`必须扫描集合中所有文档的查询更好。另请参阅查询选择性。

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：