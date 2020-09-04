# 索引

**在本页面**

- [默认的id索引](#默认)
- [创建索引](#创建)
- [索引类型](#类型)
- [索引属性](#属性)
- [索引用途](#用途)
- [索引和排序规则](#规则)
- [覆盖查询](#查询)
- [索引交集](#交集)
- [限制条件](#条件)
- [其他注意事项](#注意)

索引支持在MongoDB中有效地执行查询。如果没有索引，MongoDB必须执行集合扫描，即扫描集合中的每个文档，以选择那些与查询语句匹配的文档。如果一个查询存在适当的索引，MongoDB可以使用该索引来限制它必须检查的文档数量。

索引是特殊的数据结构，它以一种易于遍历的形式存储集合数据集的一小部分。索引存储一个或一组特定字段的值，按字段的值排序。索引项的排序支持有效的相等匹配和基于范围的查询操作。此外，MongoDB可以通过使用索引中的排序返回排序后的结果。

下图说明了使用索引选择和排序匹配文档的查询：

![使用索引选择并返回排序结果的查询图。 索引按升序存储“分数”值。 MongoDB可以按升序或降序遍历索引以返回排序的结果。](https://docs.mongodb.com/manual/_images/index-for-sort.bakedsvg.svg)

基本上，MongoDB中的索引与其他数据库系统中的索引类似。MongoDB在[集合](https://docs.mongodb.com/manual/reference/glossary/#term-collection)级别定义索引，并支持在MongoDB集合中文档的任何字段或子字段上的索引。

## <span id="默认">默认id索引</span>

在创建集合期间，MongoDB 在[_id](https://docs.mongodb.com/manual/core/document/#document-id-field)字段上创建[唯一索引](https://docs.mongodb.com/manual/core/index-unique/#index-type-unique)。该索引可防止客户端插入两个具有相同值的文档。你不能将**_id**字段上的index删除。

> **[success] 注意**
>
> 在[分片群集中](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)，如果您不使用**_id**字段作为[分片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)，那么您的应用程序 **必须**确保**_id**字段中值的唯一性以防止错误。这通常是通过使用标准的自动生成的[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#term-objectid)来完成的。

## <span id="创建">创建索引</span> 

要在[Mongo Shell中](https://docs.mongodb.com/manual/tutorial/getting-started/)创建索引 ，请使用 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex).

```powershell
db.collection.createIndex( <key and index type specification>, <options> )
```

以下示例在`name`字段上创建单个键降序索引：

```powershell
db.collection.createIndex( { name: -1 } )
```

[`db.collection.createIndex`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法只在不存在相同规范的索引时创建索引。

### 索引名称

索引的默认名称是索引键和索引中每个键的方向(即1或-1)的连接，使用下划线作为分隔符。例如，在**{ item : 1, quantity: -1 }**上创建的索引名称为**item_1_quantity_-1**。

您可以创建具有自定义名称的索引，比如比默认名称更易于阅读的索引。例如，考虑一个经常查询`products`集合以填充现有库存数据的应用程序。下面的[`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) 方法在名为查询的商品和数量上创建一个索引:

```shell
db.products.createIndex(
  { item: 1, quantity: -1 } ,
  { name: "query for inventory" }
)
```

您可以使用[`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#db.collection.getIndexes) 方法查看索引名称。一旦创建索引，您将无法重命名。相反，您必须删除并使用新名称重新创建索引。

## <span id="类型">索引类型</span>

MongoDB提供了许多不同的索引类型来支持特定类型的数据和查询。

### 单个字段

除MongoDB定义的`_id`索引外，MongoDB还支持在[文档的单个字段](https://docs.mongodb.com/manual/core/index-single/)上创建用户定义的升序/降序索引。

![``分数''字段上的索引图（升序）。](https://docs.mongodb.com/manual/_images/index-ascending.bakedsvg.svg)

对于单字段索引和排序操作，索引键的排序顺序(升序或降序)并不重要，因为MongoDB可以从任何方向遍历索引。

有关[单字段索引](https://docs.mongodb.com/manual/core/index-single/)的更多信息，请参见[单字段索引](https://docs.mongodb.com/manual/core/index-single/)和[使用](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#sort-results-single-field)[单字段索引](https://docs.mongodb.com/manual/core/index-single/)[排序](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#sort-results-single-field)。

### 复合索引

MongoDB还支持多个字段上的用户定义索引，即 [复合索引](https://docs.mongodb.com/manual/core/index-compound/)。

复合索引中列出的字段的顺序具有重要意义。例如，如果一个复合索引由**{userid: 1, score: -1}**组成，索引首先按**userid**排序，然后在每个**userid**值内按**score**排序。

![在userid字段（升序）和score字段（降序）上的复合索引图。 索引首先按“ userid”字段排序，然后按“ score”字段排序。](https://docs.mongodb.com/manual/_images/index-compound-key.bakedsvg.svg)

对于复合索引和排序操作，索引键的排序顺序(升序或降序)可以决定索引是否支持排序操作。有关索引顺序对复合索引中的结果的影响的更多信息，请参见 [排序顺序](https://docs.mongodb.com/manual/core/index-compound/#index-ascending-and-descending)。

有关复合索引的更多信息，请参见[复合索引](https://docs.mongodb.com/manual/core/index-compound/)和[在多个字段上排序](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#sort-on-multiple-fields)。

### 多键索引

MongoDB使用[多键索引](https://docs.mongodb.com/manual/core/index-multikey/)来索引存储在数组中的内容。如果索引包含数组值的字段，MongoDB为数组的每个元素创建单独的索引项。这些[多键索引](https://docs.mongodb.com/manual/core/index-multikey/)允许查询通过匹配数组的一个或多个元素来选择包含数组的文档。MongoDB自动决定是否创建一个多键索引，如果索引字段包含数组值;您不需要显式地指定多键类型。

![addr.zip字段上的多键索引图。 addr字段包含地址文档数组。 地址文档包含``zip''字段。](https://docs.mongodb.com/manual/_images/index-multikey.bakedsvg.svg)

有关多键索引的更多信息，请参见 [Multikey Indexes](https://docs.mongodb.com/manual/core/index-multikey/) 和 [Multikey Index Bounds](https://docs.mongodb.com/manual/core/multikey-index-bounds/)。

### 地理空间索引

为了支持对地理空间坐标数据的高效查询，MongoDB提供了两个特殊的索引:在返回结果时使用平面几何的2d索引和使用球面几何返回结果的2dsphere索引。

有关地理空间索引的高级介绍，请参见[2d Index Internals](https://docs.mongodb.com/manual/core/geospatial-indexes/)。

### 文本索引

MongoDB提供了一种文本索引类型，它支持搜索集合中的字符串内容。这些文本索引不存储特定于语言的停止词(例如**“the”，“a”，“or”**)，并且在一个集合中只存储根词的词干。

有关文本索引和搜索的更多信息，请参见[文本索引](https://docs.mongodb.com/manual/core/index-text/)。

### Hashed索引

为了支持[基于Hashed的分片](https://docs.mongodb.com/manual/core/hashed-sharding/#sharding-hashed-sharding)，MongoDB提供了[Hashed索引](https://docs.mongodb.com/manual/core/index-hashed/)类型，该索引类型对字段值的Hashed进行索引。这些索引在其范围内具有更随机的值分布，但只支持相等匹配，而不支持基于范围的查询。

## <span id="属性">索引属性</span>

### 唯一索引

索引的[unique](https://docs.mongodb.com/manual/core/index-unique/)属性使MongoDB拒绝索引字段的重复值。除了唯一性约束，唯一索引和MongoDB其他索引功能上是一致的。

### 部分索引

*3.2版中的新功能。*

[部分索引](https://docs.mongodb.com/manual/core/index-partial/)仅索引集合中符合指定过滤器表达式的文档。通过对集合中的部分文档建立索引，部分索引可以降低存储需求，并降低创建和维护索引的性能成本。

部分索引提供了稀疏索引功能的超集，因此应优先于稀疏索引。

### 稀疏索引

索引的[稀疏](https://docs.mongodb.com/manual/core/index-sparse/)属性可确保索引仅包含具有索引字段的文档的条目。索引会跳过没有索引字段的文档。

可以将稀疏索引与唯一索引结合使用，以防止插入索引字段值重复的文档，并跳过索引缺少索引字段的文档。

### TTL索引

[TTL索引](https://docs.mongodb.com/manual/core/index-ttl/)是MongoDB可以使用的特殊索引，它可以在一定时间后自动从集合中删除文档。对于某些类型的信息（例如计算机生成的事件数据，日志和会话信息），它们仅需要在数据库中保留有限的时间，这是理想的选择。

参见:通过执行[指令设置TTL使集合中的数据过期](https://docs.mongodb.com/manual/tutorial/expire-data/)。

## <span id="用途">索引用途</span>

索引可以提高读操作的效率。[分析查询性能](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)教程提供了一个带有和不带有索引的查询的执行统计信息示例。

有关MongoDB如何选择要使用的索引的信息，请参阅[查询优化器](https://docs.mongodb.com/manual/core/query-plans/#read-operations-query-optimization)。

## <span id="规则">索引和排序</span>

*3.4版的新功能。*

[排序](https://docs.mongodb.com/manual/reference/collation/)允许用户为字符串比较指定特定的语言的规则，例如字母大小写和重音符号的规则。

1. **Mongo Shell** 

2. **Compass**

   > **[success] Note**
   >
   > 下面的示例演示了Mongo Shell中的索引和排序。
   >
   > 请参阅MongoDB Compass文档，了解如何使用自定义排序法与Compass中的索引。

3. **Python**

   > **[success] Note**
   >
   > 下面的示例演示了Mongo Shell中的索引和排序。
   >
   > 参考驱动程序文档，了解如何在特定驱动程序中使用排序创建索引。

4. **Java**

   > **[success] Note**
   >
   > 下面的示例演示了Mongo Shell中的索引和排序。
   >
   > 参考驱动程序文档，了解如何在特定驱动程序中使用排序创建索引。

5. **Node.js**

   > **[success] Note**
   >
   > 下面的示例演示了Mongo Shell中的索引和排序。
   >
   > 参考驱动程序文档，了解如何在特定驱动程序中使用排序创建索引。


若要使用索引进行字符串比较，操作还必须指定相同的排序。也就是说，如果索引指定了不同的排序，则具有排序的索引不能支持对索引字段执行字符串比较的操作。

例如，该集合`myColl`在`category`具有排序规则语言环境的字符串字段上具有索引`"fr"`。

```powershell
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

下面的查询操作指定了与索引相同的排序，可以使用索引:

```powershell
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

但是，以下查询操作（默认情况下使用“简单”二进制整理程序）无法使用索引：

```powershell
db.myColl.find( { category: "cafe" } )
```

对于索引前缀键不是字符串、数组和嵌入式文档的复合索引，指定不同排序规则的操作仍然可以使用索引来支持对索引前缀键的比较。

例如，集合**myColl**有一个关于数值字段**score**和**price**以及字符串字段类别的复合索引;索引是用collation locale **"fr"**创建的，用于字符串比较:

```powershell
db.myColl.createIndex(
   { score: 1, price: 1, category: 1 },
   { collation: { locale: "fr" } } )
```

使用`"simple"`二进制排序规则进行字符串比较的以下操作可以使用索引：

```shell
db.myColl.find( { score: 5 } ).sort( { price: 1 } )
db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

以下操作使用`"simple"`二进制排序规则对索引`category`字段进行字符串比较，该操作可以使用索引来完成查询的一部分：`score: 5`

```shell
db.myColl.find( { score: 5, category: "cafe" } )
```

有关整理的更多信息，请参见[整理参考页](https://docs.mongodb.com/manual/reference/collation/)。

以下索引仅支持简单的二进制比较，不支持[排序规则](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#collation)：

- [文字](https://docs.mongodb.com/manual/core/index-text/)索引
- [2d](https://docs.mongodb.com/manual/core/2d/)索引
- [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/)索引。

## <span id="查询">覆盖查询</span>

当查询条件和查询的[<投影>](https://docs.mongodb.com/manual/reference/glossary/#term-projection)只包含索引字段时，MongoDB直接从索引返回结果，而不扫描任何文档或将文档带入内存。这些覆盖的查询可能非常高效。

![仅使用索引来匹配查询条件并返回结果的查询图。 MongoDB无需检查索引之外的数据即可完成查询。](https://docs.mongodb.com/manual/_images/index-for-covered-query.bakedsvg.svg)

有关覆盖查询的更多信息，请参见 [覆盖查询](https://docs.mongodb.com/manual/core/query-optimization/#read-operations-covered-query)。

## <span id="交集">索引交集</span>

MongoDB可以使用[索引](https://docs.mongodb.com/manual/core/index-intersection/)的[交集](https://docs.mongodb.com/manual/core/index-intersection/)来完成查询。对于指定复合查询条件的查询，如果一个索引可以满足查询条件的一部分，而另一个索引可以满足查询条件的另一部分，则MongoDB可以使用两个索引的交集来满足查询。使用复合索引还是使用索引交集是否更有效取决于特定查询和系统。

有关索引交集的详细信息，请参见[索引交集](https://docs.mongodb.com/manual/core/index-intersection/)。

## <span id="条件">限制条件</span>

某些限制适用于索引，例如索引键的长度或每个集合的索引数。有关详细信息，请参见[索引限制](https://docs.mongodb.com/manual/reference/limits/#index-limitations)。

## <span id='注意'>其他注意事项</span>

尽管索引可以提高查询性能，但是索引还提出了一些操作上的考虑。有关更多信息，请参见[索引的操作注意事项](https://docs.mongodb.com/manual/core/data-model-operations/#data-model-indexes)。

应用程序在建立索引期间可能会遇到性能下降的情况，包括对集合的有限读/写访问权限。有关索引构建过程的更多信息，请参见["现存集合的索引构建"](https://docs.mongodb.com/manual/core/index-creation/#index-operations)，包括["在复制集环境下的索引构建"](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build)章节

一些驱动程序可能指定索引，使用`NumberLong(1)`而不是`1`作为规范。这对结果索引没有任何影响。



译者：杨帅  莫薇
