# 多键索引范围

**在本页面**

- [多键索引的交集边界](#相交)
- [多键索引的复合边界](#复合)

索引扫描的边界定义查询期间要搜索的索引部分。当索引上存在多个谓词时，MongoDB将尝试通过交集或复合的方式组合这些谓词的边界，以产生具有更小边界的扫描。

## <span id="相交">多键索引的交集边界</span>

边界交集指的是多个边界的逻辑连接(即:**AND**)。例如，给定两个边界[`[3，∞]`]和[`[-∞，6]`]，边界的交集得到[`[3,6]`]。

给定一个[索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey)数组字段，请考虑一个查询，该查询在数组上指定多个谓词，并且可以使用 [多键索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey)。如果联接连接谓词，则MongoDB可以与[多键索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey)边界相交 [`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)。

给定[索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey)数组字段，考虑一个在数组上指定多个谓词并可以使用[多键索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey)的查询。如果一个[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)连接谓词，MongoDB可以交叉多键索引边界。

例如，一个集合`survey`包含带有一个字段`item`和一个数组字段的文档 `ratings`：

```powershell
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }
{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

在`ratings`数组上创建一个多键索引:

```powershell
db.survey.createIndex( { ratings: 1 } )
```

下面的查询使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)要求数组至少包含一个匹配这两个条件的元素:

```shell
db.survey.find( { ratings : { $elemMatch: { $gte: 3, $lte: 6 } } } )
```

分别取谓词:

* 大于或等于3的谓词(即`$gte: 3`)的边界为[`[3，∞]`];
* 小于或等于6谓词(即`$lte: 6`)的边界为[`[-∞，6]`]。

因为查询使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)来连接这些谓词，MongoDB可以交叉边界到:

```powershell
ratings: [ [ 3, 6 ] ]
```

如果查询没有将数组字段的条件与[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)连接起来，MongoDB就不能与多键索引边界相交。考虑以下查询:

```powershell
db.survey.find( { ratings : { $gte: 3, $lte: 6 } } )
```

查询在**ratings**数组中搜索至少一个大于或等于3的元素和至少一个小于或等于6的元素。因为单个元素不需要同时满足两个条件，所以MongoDB不相交边界，使用[`[3，∞]`]或[`[-∞，6]`]。MongoDB不保证它选择这两个边界中的哪一个。

## <span id="复合">多键索引的复合边界</span>

复合边界是指对[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)的多个键使用边界。例如，给定一个复合索引`{a: 1, b: 1}`，其中a字段的界值为[`[3，∞]`]，b字段的界值为[`[-∞，6]`]，复合这些界值可以得到两个界值的使用:

```powershell
{ a: [ [ 3, Infinity ] ], b: [ [ -Infinity, 6 ] ] }
```

如果MongoDB不能复合这两个边界，MongoDB总是按照前场的边界约束索引扫描，在这种情况下，**a:[`[3，∞]`]**。

### 数组字段的复合索引

考虑一个复合的多键索引；即[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)，其中索引字段之一是数组。例如，一个集合`survey`包含带有一个字段`item`和一个数组字段的文档 `ratings`：

```powershell
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }
{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

在**item**字段和**ratings**字段上创建[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound):

```powershell
db.survey.createIndex( { item: 1, ratings: 1 } )
```

下面的查询在索引的两个键上指定一个条件:

```powershell
db.survey.find( { item: "XYZ", ratings: { $gte: 3 } } )
```

分别取谓词:

* 谓词**"XYZ"**的边界是**[`["XYZ"， "XYZ"]`]**;
* 评级:{`$gte: 3`}谓词的边界是[`[3，∞]`];

MongoDB可以复合这两个边界使用的组合边界:

```powershell
{ item: [ [ "XYZ", "XYZ" ] ], ratings: [ [ 3, Infinity ] ] }
```

### 对标量索引字段的范围查询(WiredTiger)

*3.4版本的改变:仅针对WiredTiger和内存存储引擎*

从MongoDB 3.4开始，对于使用MongoDB 3.4或更高版本创建的多键索引，MongoDB会跟踪哪个索引字段或哪些字段导致一个索引成为多键索引。跟踪这些信息允许MongoDB查询引擎使用更紧密的索引边界

上述[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)位于标量字段`item`和数组字段`ratings`:

```powershell
db.survey.createIndex( { item: 1, ratings: 1 } )
```

对于WiredTiger和内存中的存储引擎，如果一个查询操作在MongoDB 3.4或更高版本中创建的复合多键索引的索引标量字段上指定多个谓词，MongoDB将与字段的边界相交。

例如，下面的操作指定了标量字段的范围查询以及数组字段的范围查询:

```shell
db.survey.find( {
   item: { $gte: "L", $lte: "Z"}, ratings : { $elemMatch: { $gte: 3, $lte: 6 } }
} )
```

MongoDB将**item**到[`[“L”，“Z”]`]和评级到[`[3.0,6.0]`]的边界相交，使用以下的组合边界:

```powershell
"item" : [ [ "L", "Z" ] ], "ratings" : [ [3.0, 6.0] ]
```

再举一个例子，考虑标量字段属于嵌套文档的位置。例如，一个集合`survey`包含以下文档：

```powershell
{ _id: 1, item: { name: "ABC", manufactured: 2016 }, ratings: [ 2, 9 ] }
{ _id: 2, item: { name: "XYZ", manufactured: 2013 },  ratings: [ 4, 3 ] }
```

在标量字段`“item.name”`和`“item”`上创建复合多键索引。数组字段`ratings`:

```powershell
db.survey.createIndex( { "item.name": 1, "item.manufactured": 1, ratings: 1 } )
```

考虑以下操作，它在标量字段上指定查询谓词:

```powershell
db.survey.find( {
   "item.name": "L" ,
   "item.manufactured": 2012
} )
```

对于这个查询，MongoDB可以使用以下的组合边界:

```powershell
"item.name" : [ ["L", "L"] ], "item.manufactured" : [ [2012.0, 2012.0] ]
```

早期版本的MongoDB不能合并标量字段的这些边界。

### 对嵌入文档数组中的字段进行复合索引

如果数组包含嵌入的文档，要对嵌入文档中包含的字段进行索引，请使用索引规范中的[虚线字段名](https://docs.mongodb.com/master/core/document/#document-dot-notation)。例如，给定以下嵌入文档数组:

```powershell
ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
```

分数字段的虚线字段名是**“ratings.score”**。

### 非数组字段和数组字段的复合边界

考虑一个包含字段`item`和数组字段`ratings`的文档的集合**survey2**:

```powershell
{
  _id: 1,
  item: "ABC",
  ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
}
{
  _id: 2,
  item: "XYZ",
  ratings: [ { score: 5, by: "anon" }, { score: 7, by: "wv" } ]
}
```

在非数组字段`item`和数组`ratings`中的两个字段上创建复合索引。`score`和`ratings.by`:

```powershell
db.survey2.createIndex( { "item": 1, "ratings.score": 1, "ratings.by": 1 } )
```

下面的查询为所有三个字段指定了一个条件:

```powershell
db.survey2.find( { item: "XYZ",  "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
```

分别取谓词:

* 谓词`"XYZ"`的边界是**[`["XYZ"， "XYZ"]`]**;
* `{$lte: 5}`谓词的边界是**[`[-∞，5]`]**;
* **by: "anon"**谓词的边界是**["anon"， "anon"]**。

MongoDB的可以复合边界为`item`与键或者为边界`"ratings.score"`或界限为`"ratings.by"`取决于查询谓词和索引关键字的值，。MongoDB不保证与`item` 领域的界限。例如，MongoDB将选择将`item`边界与`"ratings.score"`边界复合 ：

```powershell
{

  "item" : [ [ "XYZ", "XYZ" ] ],

  "ratings.score" : [ [ -Infinity, 5 ] ],

  "ratings.by" : [ [ MinKey, MaxKey ] ]
}
```

或者，MongoDB可以选择将`item`范围与 `"ratings.by"`范围进行组合：

```powershell
{

  "item" : [ [ "XYZ", "XYZ" ] ],

  "ratings.score" : [ [ MinKey, MaxKey ] ],

  "ratings.by" : [ [ "anon", "anon" ] ]

}
```

然而，为了复合“评级”的界限。带有`“ratings.by”`边界的`“score”`。查询必须使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)。有关更多信息，请参见 [数组中索引字段的复合边界](https://docs.mongodb.com/master/core/multikey-index-bounds/#compound-fields-from-array)。

### 数组中索引字段的复合边界

将同一个数组的索引键的边界复合在一起:

* 索引键必须共享相同的字段路径，但不包括字段名称。
* 查询必须使用该路径上的$elemMatch在字段上指定谓词。

对于嵌入文档中的字段，[虚线字段名](https://docs.mongodb.com/master/core/document/#document-dot-notation)，例如**“a.b.c”.d"**，是d的字段路径。要复合同一个数组的索引键的边界，[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)必须在到但不包括字段名本身的路径上;即.`“a.b.c”`。

例如，在**ratings.score**和**ratings.by**字段创建一个符合索引：

```powershell
db.survey2.createIndex( { "ratings.score": 1, "ratings.by": 1 } )
```

字段`"ratings.score"`和`"ratings.by"`共享字段路径`ratings`。以下查询使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)的字段`ratings`，以要求所述阵列包含至少有一个元素匹配这两个条件:

```powershell
db.survey2.find( { ratings: { $elemMatch: { score: { $lte: 5 }, by: "anon" } } } )
```

分别取谓词:

* **{ $lte: 5 }**谓词的边界是**[-∞，5]**;
* **by: "anon"**谓词的边界是**["anon"， "anon"]**

MongoDB可以复合这两个边界使用的组合边界:

```powershell
{ "ratings.score" : [ [ -Infinity, 5 ] ], "ratings.by" : [ [ "anon", "anon" ] ] }
```

### 查询没有`$elemMatch`

如果查询没有将索引数组字段的条件与[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)h连接起来，MongoDB就不能复合它们的边界。考虑以下查询:

```powershell
db.survey2.find( { "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
```

因为数组中嵌入的单个文档不需要同时满足这两个条件，所以MongoDB不复合边界。使用复合索引时，如果MongoDB不能约束索引的所有字段，MongoDB总是约束索引的前导字段，这里是`“ratings.score”`:

```powershell
{
  "ratings.score": [ [ -Infinity, 5 ] ],
  "ratings.by": [ [ MinKey, MaxKey ] ]
}
```

### `$elemMatch`在不完整路径上

如果查询没有在嵌入字段的路径上指定[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)，最多但不包括字段名，MongoDB不能复合来自同一数组的索引键的边界。

例如，集合`survey3`包含一个字段`item`和一个数组字段`ratings`的文档:

```powershell
{
  _id: 1,
  item: "ABC",
  ratings: [ { scores: [ { q1: 2, q2: 4 }, { q1: 3, q2: 8 } ], loc: "A" },
             { scores: [ { q1: 2, q2: 5 } ], loc: "B" } ]
}
{
  _id: 2,
  item: "XYZ",
  ratings: [ { scores: [ { q1: 7 }, { q1: 2, q2: 8 } ], loc: "B" } ]
}
```

在`ratings.scores.q1`和`ratings.scores.q2`字段上创建一个[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)。

```powershell
db.survey3.createIndex( { "ratings.scores.q1": 1, "ratings.scores.q2": 1 } )
```

字段`"ratings.scores.q1"`和`"ratings.scores.q2"`共享字段路径`"ratings.scores"`，并且[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)必须在该路径上。

但是，下面的查询使用了[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)，但不是在必需的路径上:

```powershell
db.survey3.find( { ratings: { $elemMatch: { 'scores.q1': 2, 'scores.q2': 8 } } } )
```

因此，MongoDB **无法**混合边界，并且 `"ratings.scores.q2"`在索引扫描期间该字段将不受限制。要增加界限，查询必须[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)在路径上使用`"ratings.scores"`：

```powershell
db.survey3.find( { 'ratings.scores': { $elemMatch: { 'q1': 2, 'q2': 8 } } } )
```



译者：杨帅