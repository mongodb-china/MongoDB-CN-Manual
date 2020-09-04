## 创建索引来支持查询

**在本页面**

- [如果所有查询都使用相同的单键，则创建单键索引](#id1)
- [创建复合索引以支持几种不同的查询](#id2)
- [索引使用和排序](#id3)

当索引包含查询扫描的所有字段时，索引就支持查询。查询扫描的是索引而不是集合。创建支持查询的索引可以极大地提高查询性能。

本文档描述创建支持查询的索引的策略。

### <span id="id1">如果所有查询使用相同的单键，则创建单键索引</span>

如果只查询给定集合中的单个键，则只需为该集合创建一个单键索引。例如，您可以在**product**集合中创建**category**索引:

```powershell
db.products.createIndex( { "category": 1 } )
```

### <span id="id2">创建复合索引来支持几个不同的查询</span>

如果有时只查询一个键，而有时又查询该键和第二个键的组合，那么创建复合索引比创建单键索引更有效。MongoDB将对两个查询使用复合索引。例如，您可以在`category`和两者上创建索引`item`。

```powershell
db.products.createIndex( { "category": 1, "item": 1 } )
```

这允许您两个选择。您可以只查询`category`，也可以与`category`组合查询`item`。多个字段上的单个[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)可以支持所有搜索这些字段的“前缀”子集的查询。

> 例子
>
> 以下是一个集合的索引:

```powershell
{ x: 1, y: 1, z: 1 }
```

以下索引可以支持查询:

```powershell
{ x: 1 }
{ x: 1, y: 1 }
```

在某些情况下，前缀索引可能提供更好的查询性能:例如，如果' z '是一个大数组。

`{x: 1, y: 1, z: 1} `索引也可以支持与以下索引相同的许多查询:

```powershellshell
{ x: 1, z: 1 }
```

此外，` {x: 1, z: 1} `还有其他用途。给定以下查询:

```powershell
db.collection.find( { x: 5 } ).sort( { z: 1} )
```

`{x: 1, z: 1}`索引同时支持查询和排序操作，而`{x: 1, y: 1, z: 1}`索引只支持查询。有关排序的更多信息，请参见[使用索引对查询结果排序](https://docs.mongodb.com/master/tutorial/sort-results-with-indexes/# soring-with-indexes)。

从2.6版本开始，MongoDB可以使用[索引交集](https://docs.mongodb.com/master/core/index-intersection/)来完成查询。是创建支持查询的复合索引，还是依赖索引交集，这取决于系统的具体情况。更多细节请参见[索引交集和复合索引](https://docs.mongodb.com/master/core/index-intersection/#index-intersec-compound-indexes)。

### <span id="id3">索引的使用和排序</span>

若要使用索引进行字符串比较，操作还必须指定相同的排序规则。也就是说，如果索引指定了不同的排序规则，则具有排序规则的索引不能支持对索引字段执行字符串比较的操作。

例如，集合' **myColl** '在字符串字段' **category** '上有一个索引，其排序区域设置为' **fr**'。

```powershell
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

下面的查询操作指定了与索引相同的排序规则，可以使用索引:

```powershell
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

但是，以下查询操作，默认使用**“simple”**二进制排序器，不能使用索引:

```powershell
db.myColl.find( { category: "cafe" } )
```

对于索引前缀键不是字符串、数组和嵌入文档的复合索引，指定不同排序规则的操作仍然可以使用索引来支持对索引前缀键的比较。

例如，集合**' myColl '**在数值字段` score`和` price `以及字符串字段` category `上有一个复合索引;索引是用collation locale **"fr"** 创建的，用于字符串比较:

```powershell
db.myColl.createIndex(
   { score: 1, price: 1, category: 1 },
   { collation: { locale: "fr" } } )
```

以下使用 **"simple"** 二进制排序来进行字符串比较的操作可以使用索引:

```powershell
db.myColl.find( { score: 5 } ).sort( { price: 1 } )
db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

下面的操作使用 **"simple"**二进制排序对索引的` category `字段进行字符串比较，可以使用索引来完成查询的` score: 5 `部分:

```powershell
db.myColl.find( { score: 5, category: "cafe" } )
```

