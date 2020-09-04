## 索引交集

**在本页面**

- [索引前缀交集](#交集)
- [索引交集和复合索引](#复合)
- [索引交集和排序](#排序)

MongoDB可以使用多个索引的交集来完成查询。通常，每个索引交集涉及两个索引。但是，MongoDB可以使用多个/嵌套索引交集来解析查询。

为了说明索引交集，请考虑`orders`具有以下索引的集合：

```powershell
{ qty: 1 }
{ item: 1 }
```

MongoDB可以使用两个索引的交集来支持以下查询：

```powershell
db.orders.find( { item: "abc123", qty: { $gt: 15 } } )
```

要确定MongoDB是否使用了索引交集，运行 [`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain);`explain()`的结果将包括`AND_SORTED`阶段或`AND_HASH`阶段。

### <span id="交集">索引前缀交集</span>

使用索引交集，MongoDB可以使用整个索引或索引前缀的交集。索引前缀是复合索引的子集，由一个或多个从索引开头开始的键组成。

考虑`orders`具有以下索引的集合：

```powershell
{ qty: 1 }
{ status: 1, ord_date: -1 }
```

为了完成以下查询，它在`qty`字段和`status`字段上都指定了一个条件，MongoDB可以使用两个索引的交集:

```powershell
db.orders.find( { qty: { $gt: 10 } , status: "A" } )
```

### <span id="复合">索引交集和复合索引</span>

索引交集并不能消除创建[复合索引](https://docs.mongodb.com/manual/core/index-compound/)的需要 。但是，由于[复合索引中](https://docs.mongodb.com/manual/core/index-compound/)的列表顺序（即，键在索引中的列出顺序）和排序顺序（即，升序或降序）都很重要 ，因此复合索引可能不支持不包含以下内容的查询条件：该[指数的前缀键](https://docs.mongodb.com/manual/core/index-compound/#compound-index-prefix)，或者指定一个不同的排序顺序。

例如，如果一个集合`orders`具有以下复合索引，且该`status`字段在字段之前列出`ord_date`：

```powershell
{ status: 1, ord_date: -1 }
```

复合索引可以支持以下查询：

```powershell
db.orders.find( { status: { $in: ["A", "P" ] } } )
db.orders.find(
   {
     ord_date: { $gt: new Date("2014-02-01") },
     status: {$in:[ "P", "A" ] }
   }
)
```

但不是以下两个查询：

```powershell
db.orders.find( { ord_date: { $gt: new Date("2014-02-01") } } )
db.orders.find( { } ).sort( { ord_date: 1 } )
```

但是，如果集合具有两个单独的索引：

```powershell
{ status: 1 }
{ ord_date: -1 }
```

这两个索引可以单独或通过索引交集来支持所有上述四个查询。

创建支持查询的复合索引还是依赖索引交集之间的选择取决于系统的具体情况。

也可以看看

[复合索引](https://docs.mongodb.com/manual/core/index-compound/)， [创建复合索引以支持多个不同的查询](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/#compound-key-indexes)

### <span id="排序">索引交集和排序</span>

当[`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort) 操作要求索引与查询谓词完全分开时，索引交集不适用。

例如，该`orders`集合具有以下索引：

```powershell
{ qty: 1 }
{ status: 1, ord_date: -1 }
{ status: 1 }
{ ord_date: -1 }
```

MongoDB不能对以下带有排序的查询使用索引交集：

```powershell
db.orders.find( { qty: { $gt: 10 } } ).sort( { status: 1 } )
```

也就是说，MongoDB不会将索引用于查询，而将单独索引或索引用于排序。`{ qty: 1 }``{ status: 1 }``{ status: 1, ord_date: -1 }`

也就是说，MongoDB不使用{ **qty: 1** }索引进行查询，使用单独的{ **status: 1** }或{ **status: 1, ord_date: -1** }索引进行排序。

然而，MongoDB可以使用索引交集来进行以下排序查询，因为索引{ **status: 1, ord_date: -1** }可以完成部分查询谓词。

```powershell
db.orders.find( { qty: { $gt: 10 } , status: "A" } ).sort( { ord_date: -1 } )
```