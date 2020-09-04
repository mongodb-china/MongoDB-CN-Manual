# [ ](#)$in

[]()

在本页面

*   [例子](#examples)

**$in**

`$in`操作符选择字段值等于指定数组中任何值的文档。要指定一个`$in`表达式，使用下面的原型:

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

```powershell
{ field: { $in: [<value1>, <value2>, ... <valueN> ] } }
```

如果`field`持有的阵列，则`$in`操作符选择字段中包含至少一个与指定数组中的值匹配的元素的文档（例如，`<value1>`，`<value2>`等）

## <span id="examples">例子</span>

### 使用`$in`运算符来匹配值

考虑以下示例：

```powershell
db.inventory.find( { qty: { $in: [ 5, 15 ] } } )
```

该查询选择`inventory` 集合中`qty`字段值为`5`或的`15`所有文档。尽管可以使用`$or`运算符表示此查询 ，但是在同一字段上执行相等性检查时，请选择`$in`运算符而不是`$or`运算符。

### 使用`$in`运算符匹配数组中的值

集合`inventory`包含包含字段的文档， `tags`如下所示：

```powershell
{ _id: 1, item: "abc", qty: 10, tags: [ "school", "clothing" ], sale: false }
```

然后，下面的`update()`操作将设定的`sale`字段值`true`，其中`tags`字段保持与至少一个元素匹配任一阵列`"appliances"`或 `"school"`。

```powershell
db.inventory.update(
                     { tags: { $in: ["appliances", "school"] } },
                     { $set: { sale:true } }
                   )
```

有关查询数组的其他示例，请参见：

- 查询数组
- 查询嵌入式文档数组

有关查询的其他示例，请参见：

- 查询文件

### 将`$in`运算符与正则表达式一起使用

`$in`操作符可利用形式的正则表达式匹配指定值`/pattern/`。您*不能在`$in`中*使用`$regex`运算符表达式。

考虑以下示例：

```powershell
db.inventory.find( { tags: { $in: [ /^be/, /^st/ ] } } )
```

此查询选择`inventory`集合中的所有文档，其中`tags`字段包含以`be`或`st`开头的字符串，或至少有一个以`be`或`st`开头的元素的数组。

> **也可以看看**
> `find()`，`update()`，`$or`，`$set`，`$elemMatch`。



译者：李冠飞

校对：