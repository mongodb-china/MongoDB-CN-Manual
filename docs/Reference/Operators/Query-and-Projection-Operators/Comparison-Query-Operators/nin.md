# [ ](#)$nin

[]()

**$nin**

*语法*：`{ field: { $nin: [ <value1>, <value2> ... <valueN> ]} }`

`$nin`选择以下位置的文档：

- 该`field`值不在指定的范围内`array` **或**
- 在`field`不存在。

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

考虑以下查询：

```powershell
db.inventory.find( { qty: { $nin: [ 5, 15 ] } } )
```

这个查询将选择库存集合中`qty`字段值不等于`5`或`15`的所有文档。所选文档将包括那些不包含`qty`字段的文档。

如果字段包含数组，那么`$nin`操作符将选择字段中没有元素等于指定数组中的值的文档(例如`<value1>`， `<value2>`，等等)。

考虑以下查询：

```powershell
db.inventory.update( { tags: { $nin: [ "appliances", "school" ] } }, { $set: { sale: false } } )
```

这个`update()`操作将设置库存集合中的`sale`字段值，其中，`tags`字段包含一个数组，数组中没有与数组`["appliances"， "school"]`中的元素匹配的元素，或者文档不包含`tags`字段。

不等运算符`$nin`的选择性不是很强，因为它经常匹配索引的很大一部分。因此，在许多情况下，带有索引的`$nin`查询的性能可能不比必须扫描集合中所有文档的`$nin`查询好。请参见查询选择性。

> **也可以看看**
>
> `find()`，`update()`，`$set`。



译者：李冠飞

校对：