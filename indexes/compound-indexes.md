# 复合索引

**在本页面**

* [创建复合索引](compound-indexes.md#创建)
* [排序顺序](compound-indexes.md#排序)
* [前缀](compound-indexes.md#前缀)
* [索引交集](compound-indexes.md#交集)
* [其他注意事项](compound-indexes.md#注意)

MongoDB支持复合索引，其中单个索引结构持有对集合文档中多个字段 [\[1\]](https://docs.mongodb.com/manual/core/index-compound/#compound-index-field-limit)的引用。下图展示了两个字段上的复合索引示例:

![&#x5728;userid&#x5B57;&#x6BB5;&#xFF08;&#x5347;&#x5E8F;&#xFF09;&#x548C;score&#x5B57;&#x6BB5;&#xFF08;&#x964D;&#x5E8F;&#xFF09;&#x4E0A;&#x7684;&#x590D;&#x5408;&#x7D22;&#x5F15;&#x56FE;&#x3002; &#x7D22;&#x5F15;&#x9996;&#x5148;&#x6309;&#x201C; userid&#x201D;&#x5B57;&#x6BB5;&#x6392;&#x5E8F;&#xFF0C;&#x7136;&#x540E;&#x6309;&#x201C; score&#x201D;&#x5B57;&#x6BB5;&#x6392;&#x5E8F;&#x3002;](https://www.mongodb.com/docs/manual/images/index-compound-key.bakedsvg.svg)

[\[1\]](https://docs.mongodb.com/manual/core/index-compound/#id1) **mongodb对任何复合索引施加32个字段的限制。**

复合索引可以支持在多个字段上匹配的查询。

## 创建复合索引

要创建一个[复合索引](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)，使用类似如下原型的操作:

```text
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )
```

索引规范中的字段值描述该字段的索引类型。例如，值**1**指定按升序对项排序的索引。值**-1**指定按降序对项排序的索引。有关其他索引类型，请参见[索引类型](https://docs.mongodb.com/manual/indexes/#index-types)。

> **\[warning\] 重要**
>
> 不能创建具有**hashed索引**类型的复合索引。如果试图创建包含[hashed索引字段](https://docs.mongodb.com/manual/core/index-hashed/)的复合索引，将收到一个错误。

考虑一个名为**products**的集合，它包含类似于以下文档的文档:

```text
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases"
}
```

以下操作在`item`和 `stock`字段上创建一个升序索引：

```text
db.products.createIndex( { "item": 1, "stock": 1 } )
```

复合索引中列出的字段的顺序很重要。索引将包含对文档的引用，这些文档首先按`item`字段的值排序，然后在该字段的每个值内`item`，按`stock`字段的值排序。有关更多信息，请参见[排序顺序](https://docs.mongodb.com/manual/core/index-compound/#index-ascending-and-descending)。

除了支持在所有索引字段上都匹配的查询之外，复合索引还可以支持在索引字段的前缀上匹配的查询。也就是说，索引支持对`item`字段以及`item`和`stock`字段的查询：

```text
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { $gt: 5 } } )
```

有关详细信息，请参见[前缀](compound-indexes.md)。

## 排序顺序

索引以升序（`1`）或降序（`-1`）排序顺序存储对字段的引用。对于单字段索引，键的排序顺序无关紧要，因为MongoDB可以在任一方向上遍历索引。但是，对于[复合索引](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)，属性的顺序决定了索引是否支持结果集的排序。

假设一个包含字段`username`和`date`的文档的集合事件。应用程序可以发出查询，返回的结果首先按升序`username`值排序，然后按降序\(即从最近到最后\)`date`排序，例如:

```text
db.events.find().sort( { username: 1, date: -1 } )
```

或先按`username` 降序再按`date`升序返回结果的查询，例如:

```text
db.events.find().sort( { username: -1, date: 1 } )
```

以下索引可以支持这两种排序操作：

```text
db.events.createIndex( { "username" : 1, "date" : -1 } )
```

但是，上面的索引不支持先升序`username`值再升序 `date`值排序，例如:

```text
db.events.find().sort( { username: 1, date: 1 } )
```

有关排序顺序和复合索引的更多信息，请参见 [使用索引对查询结果进行排序](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/)。

## 前缀

索引前缀是索引字段的开始子集。例如，假设以下复合索引:

```text
{ "item": 1, "location": 1, "stock": 1 }
```

索引具有以下索引前缀：

* `{ item: 1 }`
* `{ item: 1, location: 1 }`

对于复合索引，MongoDB可以使用索引来支持对索引前缀的查询。这样，MongoDB可以将索引用于以下字段的查询：

* the `item` 字段,
* the `item` 字段 _and_ the `location` 字段,
* the `item` 字段 _and_ the `location` 字段 和 the `stock` 字段.

MongoDB还可以使用索引来支持对`item`和 `stock`字段的查询，因为`item`字段对应于一个前缀。但是，在支持查询方面，索引的效率不如只支持`item`和`stock`的索引。索引字段按顺序解析；如果查询省略了特定的索引前缀，则无法使用该前缀之后的任何索引字段。

由于查询 `item` 和 `stock` 省略了 `location` 索引前缀，因此它不能使用 `stock` 其后的索引字段 `location`。只有 `item` 索引中的字段可以支持此查询。有关更多信息，请参见[创建索引以支持您的查询](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/#std-label-create-indexes-to-support-queries)。

然而，MongoDB不能使用索引来支持包含以下字段的查询，因为没有`item`字段，列出的字段都不对应前缀索引:

* the `location` 字段,
* the `stock` 字段, 
* the `location`  `stock` 字段.

如果你的集合同时具有复合索引和其前缀的索引\(例如:**{a:1,b: 1}和{a:1}**\),如果两个索引都没有稀疏约束或唯一约束，那么您可以删除前缀上的索引\(例如**{a: 1}**\)。MongoDB将在所有使用前缀索引的情况下使用复合索引。

## 索引交集

从2.6版本开始，MongoDB可以使用[索引交集](https://docs.mongodb.com/manual/core/index-intersection/)来完成查询。是创建支持查询的复合索引，还是依赖索引交集，这取决于系统的具体情况。有关更多细节，请参见 [索引交集和复合索引](https://docs.mongodb.com/manual/core/index-intersection/#index-intersection-compound-indexes)。

## 其他注意事项

在索引构建期间，应用程序可能会遇到性能下降，包括对集合的读/写访问受限。有关索引构建过程的更多信息，，请参见 [“填充集合上的索引构建”](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)，包括“ [复制环境中的索引构建”](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)部分。

一些驱动程序可能使用`NumberLong(1)`而不是 `1`将规范指定为索引。这对结果索引没有任何影响。

译者：莫薇

