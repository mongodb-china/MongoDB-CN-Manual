# 多键索引

**在本页面**

- [创建多键索引](#创建)
- [索引界限](#界限)
- [唯一多键索引](#唯一)
- [局限性](#局限)
- [例子](#例子)

为了索引包含数组值的字段，MongoDB为数组中的每个元素创建一个索引键。这些多键索引支持对数组字段的高效查询。多键索引可以在包含标量值(例如字符串、数字)和嵌套文档的数组上构造。

![addr.zip字段上的多键索引图。 addr字段包含地址文档数组。 地址文档包含``zip''字段。](https://docs.mongodb.com/manual/_images/index-multikey.bakedsvg.svg)

**标量值指的是既不是嵌入式文档也不是数组的值。**

## <span id="创建">创建多键索引</span>

使用 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法创建一个多键索引:

```powershell
db.coll.createIndex( { <field>: < 1 or -1 > } )
```

MongoDB自动创建一个多键索引，如果任何索引字段是一个数组;您不需要显式地指定多键类型。

*3.4版本的改变:仅针对WiredTiger和内存存储引擎，*

从MongoDB 3.4开始，对于使用MongoDB 3.4或更高版本创建的多键索引，MongoDB会跟踪哪个索引字段或哪些字段导致一个索引成为多键索引。跟踪这些信息允许MongoDB查询引擎使用更紧密的索引边界。

## <span id="界限">索引界限</span>

如果索引是多键的，那么索引边界的计算遵循特殊规则。有关多键索引边界的详细信息，请参见[多键索引边界](https://docs.mongodb.com/manual/core/multikey-index-bounds/)。

## <span id="唯一">唯一多键索引</span>

对于[唯一](https://docs.mongodb.com/manual/core/index-unique/)索引，唯一约束适用于集合中的各个单独文档，而不是在单个文档中。

由于**unique**约束适用于单独的文档，对于 [唯一多键索引](https://docs.mongodb.com/manual/core/index-unique/#unique-separate-documents)，只要文档的索引键值不复制另一个文档的索引键值，文档可能具有导致重复索引键值的数组元素。

有关更多信息，请参见[跨单独文档的唯一约束](https://docs.mongodb.com/manual/core/index-unique/#unique-separate-documents)。

## <span id="局限">局限性</span>

### 复合多键索引

对于[复合](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)多键索引，每个索引文档最多只能有一个索引字段，其值是一个数组。那就是:

- 如果文档的多个待索引字段是数组，则无法创建复合多键索引。例如，考虑一个包含以下文档的集合:

  ```powershell
  { _id: 1, a: [ 1, 2 ], b: [ 1, 2 ], category: "AB - both arrays" }
  ```

  因为**a**和**b**字段都是数组，所以不能在集合上创建复合多键索引**{a: 1, b: 1}**。

- 或者，如果复合多键索引已经存在，则不能插入违反此限制的文档。

  假设一个包含以下文档的集合：

  ```powershell
  { _id: 1, a: [1, 2], b: 1, category: "A array" }
  { _id: 2, a: 1, b: [1, 2], category: "B array" }
  ```

  允许使用复合多键索引**{A: 1, b: 1}**，因为对于每个文档，只有一个复合多键索引的字段是一个数组;也就是说，没有文档同时包含**a**和**b**字段的数组值。

  但是，在创建复合多键索引之后，如果您试图插入一个**a**和**b**字段都是数组的文档，MongoDB将导致插入失败。

如果字段是文档数组，则可以索引嵌入的字段以创建复合索引。例如，考虑一个包含以下文档的集合:

```powershell
{ _id: 1, a: [ { x: 5, z: [ 1, 2 ] }, { z: [ 1, 2 ] } ] }
{ _id: 2, a: [ { x: 5 }, { z: 4 } ] }
```

你可以在{**"a.x": 1, "a.z": 1** }上创建一个复合索引。数组最多只能有一个索引字段的限制也适用。

有关示例，请参见[带有嵌入式文档的索引数组](https://docs.mongodb.com/manual/core/index-multikey/#multikey-embedded-documents)。

也可以看看

- [跨单独文档的唯一约束](https://docs.mongodb.com/manual/core/index-unique/#unique-separate-documents)
- [单个字段上的唯一索引](https://docs.mongodb.com/manual/core/index-unique/#index-unique-index)

### 排序

由于MongoDB 3.6中数组字段排序行为的改变，当对多键索引的数组排序时，查询计划包括一个阻塞排序阶段。新的排序行为可能会对性能产生负面影响。

在阻塞排序中，在生成输出之前，排序步骤必须使用所有输入。在非阻塞排序或索引排序中，排序步骤扫描索引以按请求的顺序生成结果。

### 分片键

不能指定多键索引为分片键。

但是，如果分片键索引是复合索引的[前缀](https://docs.mongodb.com/manual/core/index-compound/#compound-index-prefix)，那么如果其他键中的一个(即不属于切分键的键)索引了数组，那么复合索引就可以变成复合多键索引。复合多键索引会对性能产生影响。

### Hashed索引

[Hashed](https://docs.mongodb.com/manual/core/index-hashed/)索引不能为多键。

### 覆盖查询

[多键索引](https://docs.mongodb.com/manual/core/index-multikey/#index-type-multikey)不能覆盖对数组字段的查询。

然而，从3.6开始，如果索引跟踪哪个或哪个字段导致索引为多键，那么多键索引可以覆盖对非数组字段的查询。在MongoDB 3.4或更高版本的存储引擎上创建的多键索引，而不是MMAPv1[#]_跟踪该数据。

**从4.2版本开始，MongoDB删除了已弃用的MMAPv1存储引擎。**

### 整体查询数组字段

当一个查询过滤器为一个数组指定了一个精确的匹配，MongoDB可以使用**multikey**索引来查找查询数组的第一个元素，但是不能使用**multikey**索引扫描来查找整个数组。相反，在使用**multikey**索引查找查询数组的第一个元素之后，MongoDB检索相关的文档，并筛选其数组与查询中的数组匹配的文档。

例如，假设一个包含以下文档的`inventory`集合:

```powershell
{ _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
{ _id: 6, type: "food", item: "bbb", ratings: [ 5, 9 ] }
{ _id: 7, type: "food", item: "ccc", ratings: [ 9, 5, 8 ] }
{ _id: 8, type: "food", item: "ddd", ratings: [ 9, 5 ] }
{ _id: 9, type: "food", item: "eee", ratings: [ 5, 9, 5 ] }
```

该集合在`ratings`字段上有一个多键索引:

```powershell
db.inventory.createIndex( { ratings: 1 } )
```

下面的查询查找`ratings`字段为数组**[5,9]**的文档:

```powershell
db.inventory.find( { ratings: [ 5, 9 ] } )
```

MongoDB可以使用多键索引来查找**ratings**数组中任何位置有**5**的文档。然后，MongoDB检索这些文档，筛选`ratings`数组等于查询数组的文档**[5,9]**。

### $expr

[`$expr`](https://docs.mongodb.com/manual/reference/operator/query/expr/#op._S_expr) 不支持多键索引。

## <span id="例子">例子</span>

### 索引基本数组

假设一个包含以下文档的`survey`集合:

```powershell
{ _id: 1, item: "ABC", ratings: [ 2, 5, 9 ] }
```

在`ratings`上创建索引:

```powershell
db.survey.createIndex( { ratings: 1 } )
```

由于`ratings`字段包含一个数组，`ratings`的索引是多键的。多键索引包含以下三个索引键，每个都指向同一个文档:

- `2`，
- `5`，
- `9`。

### 数组索引与嵌入式文件

可以在包含嵌套对象的数组字段上创建多键索引。

假设使用以下形式的文档进行`inventory`收集:

```powershell
{
  _id: 1,
  item: "abc",
  stock: [
    { size: "S", color: "red", quantity: 25 },
    { size: "S", color: "blue", quantity: 10 },
    { size: "M", color: "blue", quantity: 50 }
  ]
}
{
  _id: 2,
  item: "def",
  stock: [
    { size: "S", color: "blue", quantity: 20 },
    { size: "M", color: "blue", quantity: 5 },
    { size: "M", color: "black", quantity: 10 },
    { size: "L", color: "red", quantity: 2 }
  ]
}
{
  _id: 3,
  item: "ijk",
  stock: [
    { size: "M", color: "blue", quantity: 15 },
    { size: "L", color: "blue", quantity: 100 },
    { size: "L", color: "red", quantity: 25 }
  ]
}

...
```

以下操作在`stock.size` 和`stock.quantity`字段上创建一个多键索引：

```powershell
db.inventory.createIndex( { "stock.size": 1, "stock.quantity": 1 } )
```

复合多键索引可以支持具有谓词的查询，这些谓词既包括索引字段，也包括仅包括索引前缀**“stock.size”**的谓词。，如以下例子所示:

```powershell
db.inventory.find( { "stock.size": "M" } )
db.inventory.find( { "stock.size": "S", "stock.quantity": { $gt: 20 } } )
```

有关MongoDB如何组合多键索引边界的详细信息，请参见[多键索引边界](https://docs.mongodb.com/manual/core/multikey-index-bounds/)。有关复合索引和前缀的行为的更多信息，请参见[复合索引和前缀](https://docs.mongodb.com/manual/core/index-compound/#compound-index-prefix)。

复合多键索引也可以支持排序操作，例如下面的例子:

```powershell
db.inventory.find( ).sort( { "stock.size": 1, "stock.quantity": 1 } )
db.inventory.find( { "stock.size": "M" } ).sort( { "stock.quantity": 1 } )
```

有关复合索引和排序操作的行为的更多信息，请参见[使用索引对查询结果进行排序](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/)。



译者：杨帅