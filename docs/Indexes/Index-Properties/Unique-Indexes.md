## 唯一索引

**在本页面**

- [创建唯一索引](#创建)
- [行为](#行为)

唯一索引确保索引字段不会存储重复值;例如，强制索引字段的唯一性。默认情况下，MongoDB在创建集合期间在[_id](https://docs.mongodb.com/master/core/document/#document-id-field)字段上创建一个唯一的索引。

> 新的内部格式
>
> 从MongoDB 4.2开始，对于4.2（或更高版本）的[featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)（[fCV](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)），MongoDB使用一种新的内部格式来存储与早期MongoDB版本不兼容的唯一索引。新格式适用于现有的唯一索引以及新创建/重建的唯一索引。

### <span id="创建">创建唯一索引</span>

要创建一个唯一的索引，使用[` db.collection.createIndex() `](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法，并将`unique`选项设置为` true`。

```powershell
db.collection.createIndex( <key and index type specification>, { unique: true } )
```

#### 单个字段上的唯一索引

例如，要在` members `集合的` user_id `字段上创建一个唯一的索引，在[`mongo `](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中使用以下操作:

```powershell
db.members.createIndex( { "user_id": 1 }, { unique: true } )
```

#### 独特的复合索引

您还可以在[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)上强制执行唯一约束。如果您在[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)上使用唯一约束，那么MongoDB将对索引键值的*组合*执行惟一性。

例如，要在` members `集合的` groupNumber `， ` lastname `和` firstname `字段上创建一个唯一的索引，在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中使用以下操作:

```powershell
db.members.createIndex( { groupNumber: 1, lastname: 1, firstname: 1 }, { unique: true } )
```

创建的索引强制`groupNumber`、`lastname`和`firstname`值的*组合*的唯一性。

再举一个例子，考虑一个包含以下文档的集合:

```powershell
{ _id: 1, a: [ { loc: "A", qty: 5 }, { qty: 10 } ] }
```

创建一个独特的复合[multikey](https://docs.mongodb.com/master/core/index-multikey/)索引在`a.loc`和`a.qty`:

```powershell
db.collection.createIndex( { "a.loc": 1, "a.qty": 1 }, { unique: true } )
```

唯一索引允许将以下文档插入到集合中，因为索引强制`a.loc`和`a.qty`值组合的唯一性：

```powershell
db.collection.insert( { _id: 2, a: [ { loc: "A" }, { qty: 5 } ] } )
db.collection.insert( { _id: 3, a: [ { loc: "A", qty: 10 } ] } )
```

也可以看看：

[跨不同文档的唯一约束](https://docs.mongodb.com/master/core/index-separatedes-documents)和[唯一索引和丢失字段](https://docs.mongodb.com/master/core/index-unique/# unique-index-mising-field)

### <span id="行为">行为</span>

#### 限制

如果集合已经包含了违反索引的唯一约束的数据，MongoDB不能在指定的索引字段上创建一个[唯一索引](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)。

不能在[hashed索引](https://docs.mongodb.com/master/core/index-hashed/#index-type-hashed)上指定唯一的约束。

#### 在复制集和分片集群上建立唯一索引

对于复制集和分片集群，使用[滚动过程](https://docs.mongodb.com/master/tutorial/build-indexes-onreplica-sets/)创建唯一索引需要在过程中停止对集合的所有写操作。如果不能在过程中停止对集合的所有写操作，则不要使用滚动过程。相反，在集合上建立你的唯一索引:

- [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)在主数据库上发布副本集，
- [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)在分片[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)群集上发出。

#### 跨不同文档的唯一约束

唯一约束适用于集合中的不同文档。也就是说，唯一索引防止*单独的*文档对索引键具有相同的值。

因为约束适用于单独的文档,一个独特的[多键](https://docs.mongodb.com/master/core/index-multikey/)索引,一个文档可能数组元素,导致重复索引键值,只要文档不重复的索引键值的另一个文档。在本例中，重复索引条目只插入索引一次。

例如，考虑一个包含以下文档的集合:

```powershell
{ _id: 1, a: [ { loc: "A", qty: 5 }, { qty: 10 } ] }
{ _id: 2, a: [ { loc: "A" }, { qty: 5 } ] }
{ _id: 3, a: [ { loc: "A", qty: 10 } ] }
```

在`a.loc`和`a.qty`上创建唯一的复合多键索引：

```powershell
db.collection.createIndex( { "a.loc": 1, "a.qty": 1 }, { unique: true } )
```

如果集合中的其他文档的索引 key value 为`{ "a.loc": "B", "a.qty": null }`，则唯一索引允许将以下文档插入到集合中。

```powershell
db.collection.insert( { _id: 4, a: [ { loc: "B" }, { loc: "B" } ] } )
```

#### 唯一索引和丢失字段

如果文档在唯一索引中没有索引字段的值，索引将为该文档存储空值。由于唯一的约束，MongoDB将只允许一个没有索引字段的文档。如果有多个文档没有索引字段的值或缺少索引字段，索引构建将失败，并出现重复键错误。

例如，一个集合在`x `上有一个唯一的索引:

```powershell
db.collection.createIndex( { "x": 1 }, { unique: true } )
```

如果集合中没有包含缺少`x`字段的文档，唯一索引允许插入没有`x`字段的文档:

```powershell
db.collection.insert( { y: 1 } )
```

但是，如果集合中已经包含了一个没有字段`x`的文档，则在插入一个没有字段`x`的文档时出现唯一的索引错误:

```powershell
db.collection.insert( { z: 1 } )
```

由于违反了字段` x `值的唯一约束，操作无法插入文档:

```shell
WriteResult({
   "nInserted" : 0,
   "writeError" : {
      "code" : 11000,
      "errmsg" : "E11000 duplicate key error index: test.collection.$a.b_1 dup key: { : null }"
   }
})
```

也可以看看：

[Unique Partial Indexes](https://docs.mongodb.com/master/core/index-unique/#unique-partial-indexes)

#### 独特的部分索引

*3.2版本新增.*

部分索引只索引集合中满足指定筛选器表达式的文档。如果您同时指定了**partialFilterExpression**和一个[unique约束](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)，唯一约束只适用于满足筛选器表达式的文档。

如果文档不满足筛选条件，则具有唯一约束的部分索引不会阻止插入不满足唯一约束的文档。例如，请参阅[带有唯一约束的部分索引](https://docs.mongodb.com/master/core/index-partial/#partial-index-with-unique-constraints)。

#### 分片集群和唯一索引

您不能在[hashed索引](https://docs.mongodb.com/master/core/index-hashed/#index-type-hashed)上指定唯一的约束。

对于一个范围分片集合，只有以下索引可以是[唯一的](https://docs.mongodb.com/master/core/index-unique/#):

- 分片键上的索引

- 一个[复合索引](https://docs.mongodb.com/master/reference/glossary/#term-compound-index)，其中片键是一个[前缀](https://docs.mongodb.com/master/core/index-compound/#compound-index-prefix)

- 默认`_id`索引；**不过**，该`_id`指数仅实施每碎片的唯一性约束**，如果**该`_id`字段是**不是**分片键或片键的前缀。

  > 唯一性和` _ID `索引

  如果` _id `字段不是分片键或分片键的前缀，` _id `索引只对每个分片强制唯一性约束，而对各个分片强制**而不是**。


唯一的索引约束意味着:

- 对于要分片的集合，如果该集合有其他唯一索引，则不能对该集合进行分片。
- 对于已经分片的集合，不能在其他字段上创建唯一索引。