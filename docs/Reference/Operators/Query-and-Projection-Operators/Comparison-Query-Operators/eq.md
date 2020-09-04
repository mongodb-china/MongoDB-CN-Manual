# [ ](#)$eq

[]()

在本页面

*   [行为](#behavior)

*   [例子](#examples)

**$eq**

指定相等条件。`$eq`操作符匹配字段的值等于指定值的文档。

```powershell
{ <field>: { $eq: <value> } }
```

`$eq`表达式等效于。`{ field: <value> }`

## <span id="behavior">行为</span>

### 比较顺序

有关不同BSON类型值的比较，请参见指定的BSON比较顺序。

### 匹配一个文献价值

如果指定的`<value>`是文档，则文档中字段的顺序很重要。

### 匹配一个数组值

如果指定的`<value>`是数组，则MongoDB将`<field>`匹配与该数组完全匹配的文档，或者`<field>` 包含包含与该数组完全匹配的元素的文档。元素的顺序很重要。有关示例，请参见等于数组值。

## <span id="examples">例子</span>

以下示例`inventory`使用以下文档查询集合：

```powershell
{ _id: 1, item: { name: "ab", code: "123" }, qty: 15, tags: [ "A", "B", "C" ] }
{ _id: 2, item: { name: "cd", code: "123" }, qty: 20, tags: [ "B" ] }
{ _id: 3, item: { name: "ij", code: "456" }, qty: 25, tags: [ "A", "B" ] }
{ _id: 4, item: { name: "xy", code: "456" }, qty: 30, tags: [ "B", "A" ] }
{ _id: 5, item: { name: "mn", code: "000" }, qty: 20, tags: [ [ "A", "B" ], "C" ] }
```

### 等于指定值

下面的示例查询`inventory`集合以选择`qty`字段值等于的所有文档`20`：

```powershell
db.inventory.find( { qty: { $eq: 20 } } )
```

该查询等效于：

```powershell
db.inventory.find( { qty: 20 } )
```

这两个查询都匹配以下文档：

```powershell
{ _id: 2, item: { name: "cd", code: "123" }, qty: 20, tags: [ "B" ] }
{ _id: 5, item: { name: "mn", code: "000" }, qty: 20, tags: [ [ "A", "B" ], "C" ] }
```

### 嵌入式文档中的字段等于值

以下示例查询`inventory`集合以选择文档中`name`字段值`item` 等于`"ab"`的所有文档。要在嵌入式文档中的字段上指定条件，请使用点符号。

```powershell
db.inventory.find( { "item.name": { $eq: "ab" } } )
```

该查询等效于：

```powershell
db.inventory.find( { "item.name": "ab" } )
```

这两个查询都与以下文档匹配：

```powershell
{ _id: 1, item: { name: "ab", code: "123" }, qty: 15, tags: [ "A", "B", "C" ] }
```

> **也可以看看**
>
> 查询嵌入式文档

### 数组元素等于一个值

下面的示例查询`inventory`集合以选择`tags`数组包含值`"B"` [1]的元素的所有文档：

```powershell
db.inventory.find( { tags: { $eq: "B" } } )
```

该查询等效于：

```powershell
db.inventory.find( { tags: "B" } )
```

这两个查询都匹配以下文档：

```powershell
{ _id: 1, item: { name: "ab", code: "123" }, qty: 15, tags: [ "A", "B", "C" ] }
{ _id: 2, item: { name: "cd", code: "123" }, qty: 20, tags: [ "B" ] }
{ _id: 3, item: { name: "ij", code: "456" }, qty: 25, tags: [ "A", "B" ] }
{ _id: 4, item: { name: "xy", code: "456" }, qty: 30, tags: [ "B", "A" ] }
```

> **也可以看看**
>
> `$elemMatch`，查询数组

|      |                                                       |
| ---- | ----------------------------------------------------- |
| [1]  | 该查询还将匹配文档，其中`tags`字段的值为字符串`"B"`。 |

### 等于一个数组值

以下示例查询`inventory`集合，以选择该`tags`数组与指定数组完全相等或该`tags`数组包含等于该数组`[ "A", "B" ]`的元素的所有文档。

```powershell
db.inventory.find( { tags: { $eq: [ "A", "B" ] } } )
```

该查询等效于：

```powershell
db.inventory.find( { tags: [ "A", "B" ] } )
```

这两个查询都匹配以下文档：

```powershell
{ _id: 3, item: { name: "ij", code: "456" }, qty: 25, tags: [ "A", "B" ] }
{ _id: 5, item: { name: "mn", code: "000" }, qty: 20, tags: [ [ "A", "B" ], "C" ] }
```



译者：李冠飞

校对：