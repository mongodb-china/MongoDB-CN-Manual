# 单字段索引

**在本页面**

- [在单个字段上创建升序索引](#升序)
- [在嵌入式字段上创建索引](#字段)
- [在内嵌文档上创建索引](#文档)
- [其他注意事项](#注意)

MongoDB为[文档](https://docs.mongodb.com/manual/reference/glossary/#term-document)[集合](https://docs.mongodb.com/manual/reference/glossary/#term-collection)中任何字段上的索引提供了完整的支持 。默认情况下，所有集合在[_id字段](https://docs.mongodb.com/manual/indexes/#index-type-id)上都有一个索引，应用程序和用户可以添加其他索引来支持重要的查询和操作。

本文档描述单个字段上的升序/降序索引。

![Diagram of an index on the ``score`` field (ascending).](https://docs.mongodb.com/manual/_images/index-ascending.bakedsvg.svg)

## <span id="升序">在单个字段上创建升序索引</span>

假设一个 **records**的集合，包含类似于如下所示的文档:

```powershell
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

下面的操作在**records**集合的**score**字段上创建一个升序索引:

```powershell
db.records.createIndex( { score: 1 } )
```

索引规范中的字段值描述该字段的索引类型。例如，值**' 1 '**指定一个索引，该索引按升序对项目排序。值**' -1 '**指定按降序排列项目的索引。有关其他索引类型，请参见[索引类型](https://docs.mongodb.com/manual/indexes/#index-types)

创建的索引将支持在字段`score`上选择查询，例如:

```powershell
db.records.find( { score: 2 } )
db.records.find( { score: { $gt: 10 } } )
```

## <span id="字段">在嵌入式字段上创建索引</span>

可以在嵌入文档中的字段上创建索引，就像索引文档中的顶级字段一样。嵌入字段上的索引不同于[嵌入文档上的索引](https://docs.mongodb.com/manual/core/index-single/#index-embeddeddocuments)，它包含了完整的内容，直到索引中嵌入文档的最大“索引大小”为止。相反，嵌入字段上的索引允许您使用“点表示法”来内省嵌入的文档。

考虑一个名为**“records”**的集合，它包含类似于以下示例文档的文档:

```powershell
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

以下操作在**"location.state"** 字段上创建索引：

```powershell
db.records.createIndex( { "location.state": 1 } )
```

创建的索引将支持选择字段**"location.state"**的查询。，例如:

```powershell
db.records.find( { "location.state": "CA" } )
db.records.find( { "location.city": "Albany", "location.state": "NY" } )
```

## <span id="文档">在内嵌文档上创建索引</span>

您还可以在整个内嵌文档上创建索引。

考虑一个名为**“records”**的集合，它包含类似于以下示例文档的文档:

```powershell
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

**“location”**字段是一个内嵌文档，包含嵌入式字段**city**和**state**。下面的命令创建一个索引的**"location"**字段作为一个整体:

```powershell
db.records.createIndex( { location: 1 } )
```

以下查询可以使用**"location"**字段的索引:

```powershell
db.records.find( { location: { city: "New York", state: "NY" } } )
```

> **[success] Note**
>
> 尽管查询可以使用索引，但结果集不包括上面的示例文档。在嵌入文档上执行相等匹配时，字段顺序很重要，内嵌文档必须精确匹配。有关[查询内嵌文档](https://docs.mongodb.com/manual/reference/method/db.collection.find/#query-embedded-documents)的更多信息，请参见[查询内嵌文档](https://docs.mongodb.com/manual/reference/method/db.collection.find/#query-embedded-documents)。

## <span id="注意">其他注意事项</span>

在索引构建期间，应用程序可能会遇到性能下降，包括对集合的读/写访问受限。有关索引构建过程的更多信息，请参见 [填充集合上索引构建”](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)，包括[复制环境中的索引构建](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)部分。

一些驱动程序可能指定索引，使用**NumberLong(1)**而不是**1**作为规范。这对结果索引没有任何影响。



译者：杨帅 莫薇

