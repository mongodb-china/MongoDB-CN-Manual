# 查询优化

**在本页面**

- [创建索引以支持读取操作](#创建)
- [查询选择性](#查询)
- [覆盖查询](#覆盖)

索引通过减少查询操作需要处理的数据量来提高读操作的效率。这简化了与在MongoDB中完成查询相关的工作。

## <span id=" 创建">创建索引以支持读取操作</span>

如果应用程序查询特定字段或字段集上的集合，那么查询字段上的[索引](https://docs.mongodb.com/manual/core/index-compound/)或字段集上的[复合索引](https://docs.mongodb.com/manual/core/index-compound/)可以防止查询扫描整个集合来查找和返回查询结果。有关索引的更多信息，请参阅[MongoDB中索引中完整文档](https://docs.mongodb.com/manual/indexes/)。

**例子**

应用程序查询类型字段上的库存集合。类型字段的值是用户驱动的。

```powershell
var typeValue = <someUserInput>;
db.inventory.find( { type: typeValue } );
```

要提高此查询的性能，请向**type**字段上的**inventory**集合添加升序或降序索引。在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中，您可以使用[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法创建索引:

```powershell
db.inventory.createIndex( { type: 1 } )
```

这个索引可以防止上述类型查询扫描整个集合返回结果。

要使用索引[分析查询的性能](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)，请参阅 [分析查询性能](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)。

除了优化读取操作外，索引还可以支持排序操作并允许更有效地利用存储。有关索引创建的更多信息，请参见 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)和 [索引](https://docs.mongodb.com/manual/indexes/)。

**对于单字段索引，升序和降序之间的选择并不重要。对于复合索引，选择很重要。有关更多详细信息，请参见[索引顺序](https://docs.mongodb.com/manual/core/index-compound/#index-ascending-and-descending)。**

## <span id="查询">查询选择性</span>

查询选择性指的是查询谓词排除或过滤集合中的文档的能力。查询选择性可以决定查询是否能够有效地使用索引，甚至根本不使用索引。

选择性更强的查询匹配的文档比例更小。例如，唯一**_id**字段上的相等匹配具有很高的选择性，因为它最多只能匹配一个文档。

选择性较低的查询匹配较大比例的文档。选择性较低的查询不能有效地使用索引，甚至根本不能使用索引。

例如，不等操作符[`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin)和 [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne)的选择性不是很强，因为它们通常匹配索引的很大一部分。因此，在许多情况下，带有索引的[`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin)或 [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne)查询的执行性能可能不比必须扫描集合中所有文档的[`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin)或 [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne)查询好。

正则表达式的选择性取决于表达式本身。有关详细信息，请参见[正则表达式和索引使用](https://docs.mongodb.com/manual/reference/operator/query/regex/#regex-index-use)。[`regular expressions`](https://docs.mongodb.com/manual/reference/operator/query/regex/#op._S_regex)

## <span id="覆盖">覆盖查询</span>

覆盖查询是可以使用索引完全满足而不需要检查任何文档的查询。当下列所有情况都适用时，索引将 [覆盖](https://docs.mongodb.com/manual/core/query-optimization/#indexes-covered-queries)查询：

- [查询](https://docs.mongodb.com/manual/tutorial/query-documents/#read-operations-query-document) 中的所有字段都是索引的一部分。
- 结果中返回的所有字段都在同一索引中。
- 查询中没有字段等于`null`(即**{“field”:null}**或**{“field”:{$eq: null}}**)。

例如，一个集合`inventory`在`type`和`item`字段上具有以下索引 ：

```shell
db.inventory.createIndex( { type: 1, item: 1 } )
```

该索引将涵盖以下操作，该操作在`type`和`item`字段上查询 并仅返回该`item`字段：

```shell
db.inventory.find(
   { type: "food", item:/^c/ },
   { item: 1, _id: 0 }
)
```

为了让指定的索引覆盖查询，投影文档必须显式地指定**_id: 0**来从结果中排除**_id**字段，因为索引不包括**_id**字段。

3.6版本的改变:索引可以覆盖对嵌入文档中的字段的查询。

例如，考虑一个**userdata**集合，它具有以下形式的文档:

```shell
{ _id: 1, user: { login: "tester" } }
```

该集合具有以下索引：

```shell
{ "user.login": 1 }
```

该索引将涵盖以下查询：`{ "user.login": 1 }`

```shell
db.userdata.find( { "user.login": "tester" }, { "user.login": 1, _id: 0 } )
```

**要为嵌入式文档中的字段建立索引，请使用[点符号](https://docs.mongodb.com/manual/reference/glossary/#term-dot-notation)。**

#### 多键覆盖

从3.6开始，如果索引跟踪哪个或哪个字段导致索引为多键，那么多键索引可以覆盖对非数组字段的查询。在MongoDB 3.4或更高版本的存储引擎(MMAPv1除外)上创建的多键索引跟踪该数据。

[多键索引](https://docs.mongodb.com/manual/core/index-multikey/#index-type-multikey)不能覆盖对数组字段的查询。

#### 性能

因为索引包含查询所需的所有字段，所以MongoDB既可以匹配[查询条件](https://docs.mongodb.com/manual/tutorial/query-documents/#read-operations-query-document) ，又可以仅使用索引返回结果。

仅查询索引要比查询索引之外的文档快得多。索引键通常比它们编目的文档小，索引通常在RAM中可用，或按顺序位于磁盘上。

#### 局限性

##### 索引字段的限制

- [地理空间索引](https://docs.mongodb.com/manual/geospatial-queries/#index-feature-geospatial)不能 [覆盖查询](https://docs.mongodb.com/manual/core/query-optimization/#covered-queries)。

- [多键索引](https://docs.mongodb.com/manual/core/index-multikey/#index-type-multikey)不能覆盖对数组字段的查询。

  也可以看看

  [多键覆盖](https://docs.mongodb.com/manual/core/query-optimization/#multikey-covering)



##### 分片集合的限制

在MongoDB中3.0开始，索引不能覆盖在查询 [分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)的时候对一个运行集合 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)，如果指数不包含片键，除了具有以下不同的`_id`指标：如果在分片集合的查询只规定了一个条件`_id`字段并仅返回该`_id`字段，即使该 字段不是分片键，`_id`索引也可以覆盖针对[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)该`_id`字段的查询。

在以前的版本中，在对[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)运行时，索引不能[覆盖](https://docs.mongodb.com/manual/core/query-optimization/#covered-queries) 对[分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)集合的查询。

#### 解释

要确定查询是否为覆盖查询，请使用 [`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)或[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) 方法，然后查看[结果](https://docs.mongodb.com/manual/reference/explain-results/#explain-output-covered-queries)。

[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)提供有关其他操作执行的信息，例如[`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update)。有关[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)详细信息，请参见 。

有关更多信息，请参见[度量索引使用](https://docs.mongodb.com/manual/tutorial/measure-index-use/#indexes-measuring-use)。



译者：杨帅

校对：杨帅