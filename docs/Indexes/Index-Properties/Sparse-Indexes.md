## Sparse索引

**在本页面**

- [创建sparse索引](#创建)
- [行为](#行为)
- [例子](#例子)

Sparse索引只包含有索引字段的文档的条目，即使索引字段包含空值。索引会跳过任何缺少索引字段的文档。索引是**“稀疏的”**，因为它不包括一个集合的所有文档。相反，非稀疏索引包含集合中的所有文档，为那些不包含索引字段的文档存储空值。

> 重要的
>
> 从MongoDB 3.2开始，MongoDB提供了创建[部分索引](https://docs.mongodb.com/master/core/index-partial/#index-type-partial)的选项。部分索引提供了sparse索引功能的超集。如果你正在使用MongoDB 3.2或更高版本，[部分索引](https://docs.mongodb.com/master/core/index-partial/#index-type-partial)应该比稀疏索引更受欢迎。

### <span id="创建">创建sparse索引</span>

要创建一个**“sparse”**索引，使用[db.collection.createIndex()](https://docs.mongodb.com/master/reference/db.collection.createindex /#db.collection.createIndex)方法，并将**“sparse”**选项设置为**“true”**。例如，下面的操作在[mongo](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中创建了一个稀疏的索引在**xmpp_id**字段的地址集合:

```powershell
db.addresses.createIndex( { "xmpp_id": 1 }, { sparse: true } )
```

索引不会索引不包含**“xmpp_id”**字段的文档。

> 注意
>
> 不要将MongoDB中的sparse索引与其他数据库中的[块级](http://en.wikipedia.org/wiki/Database_index#Sparse_index)索引混淆。可以将它们看作具有特定过滤器的密集索引。

### <span id="行为">行为</span>

#### “Sparse”索引和不完整结果

如果sparse索引会导致查询和排序操作的结果集不完整，MongoDB将不会使用该索引，除非[hint()](https://docs.mongodb.com/master/reference/method/cursor.hint/#cursor.hint)明确指定该索引。

例如，查询`{x: {$exists: false}}`不会在**x**字段上使用sparse索引，除非有明确提示。参见[集合上的稀疏索引不能返回完整的结果](https://docs.mongodb.com/master/core/index-sparse/# spars-index-incomplet-results)了解详细的行为示例。

*Changed in version 3.4*.

如果在执行集合中所有文档的count()(i.e.带有空查询谓词)时包含 [sparse索引](https://docs.mongodb.com/master/core/index-sparse/#index-type-sparse)[`count()`](https://docs.mongodb.com/master/reference/method/cursor.count/#cursor.count)，则即使sparse索引导致计数不正确，也会使用sparse索引。

```powershell
db.collection.insert({ _id: 1, y: 1 } );
db.collection.createIndex( { x: 1 }, { sparse: true } );

db.collection.find().hint( { x: 1 } ).count();
```

要获得正确的计数，在对集合中的所有文档执行计数时，不要使用[sparse索引](https://docs.mongodb.com/master/core/index-sparse/#index-type-sparse)的**“hint()**”。

```powershell
db.collection.find().count();

db.collection.createIndex({ y: 1 });
db.collection.find().hint({ y: 1 }).count();
```

#### 默认情况下是“sparse”的索引

[2dsphere(版本2)](https://docs.mongodb.com/master/core/2dsphere/#dsphere-v2)，[ 2d](https://docs.mongodb.com/master/core/2d/)， [geoHaystack](https://docs.mongodb.com/master/core/geohaystack/)和[文本](https://docs.mongodb.com/master/core/index-text/)索引始终为`sparse`。

#### Sparse复合索引

Sparse[复合索引](https://docs.mongodb.com/master/core/index-compound/)只包含升序/降序索引键将索引一个文档，只要该文档包含至少一个键。

包含一个地理空间的稀疏的复合索引键(即[2 dsphere](https://docs.mongodb.com/master/core/2dsphere/), [2d](https://docs.mongodb.com/master/core/2d/),或[geoHaystack](https://docs.mongodb.com/master/core/geohaystack/)索引键)连同升序/降序索引键,只有地理空间的存在领域(s)文档中确定索引文档的引用。

对于包含[text](https://docs.mongodb.com/master/core/index-text/)索引键和升序/降序索引键的sparse复合索引，只有**“text”**索引字段的存在决定该索引是否引用一个文档。

#### “Sparse”和“unique”属性

一个**“sparse”**和[unique](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)索引可以防止集合的文档具有一个字段的重复值，但允许多个文档忽略该键。

### <span id="例子">例子</span>

#### 在集合上创建sparse索引

考虑一个包含以下文档的集合**“scores”**:

```powershell
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
```

集合在**“score”**字段上有一个**sparse**索引:

```powershell
db.scores.createIndex( { score: 1 } , { sparse: true } )
```

然后，下面对**scores**集合的查询使用sparse索引返回**score**字段小于([' $lt '](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt))的文档**90**:

```powershell
db.scores.find( { score: { $lt: 90 } } )
```

由于userid的文档`"newbie"`不包含该 `score`字段，因此不满足查询条件，因此查询可以使用sparse索引返回结果：

```powershell
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
```

#### 集合上的sparse索引不能返回完整的结果

考虑一个包含以下文档的集合“**scores**”:

```powershell
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
```

集合在**“score”**字段上有一个sparse索引:

```powershell
db.scores.createIndex( { score: 1 } , { sparse: true } )
```

因为userid的文档 `"newbie" `不包含` score `字段，所以sparse索引不包含该文档的条目。

考虑以下查询返回` scores `集合中的**所有**文档，按` score `字段排序:

```powershell
db.scores.find().sort( { score: -1 } )
```

即使是按索引字段排序，MongoDB也不会选择sparse索引来完成查询，以返回完整的结果:

```powershell
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
```

要使用sparse索引，显式地用` hint() `指定索引:

```powershell
db.scores.find().sort( { score: -1 } ).hint( { score: 1 } )
```

使用索引只返回那些带有`score`字段的文档:

```powershell
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
```

也可以看看：

[`explain()`](https://docs.mongodb.com/master/reference/method/cursor.explain/#cursor.explain) 和 [Analyze Query Performance](https://docs.mongodb.com/master/tutorial/analyze-query-plan/)

### 具有唯一约束的sparse索引

考虑一个包含以下文档的集合**scores**:

```powershell
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
```

您可以使用以下操作在**score**字段上创建一个[unique constraint](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)和sparse过滤器:

```powershell
db.scores.createIndex( { score: 1 } , { sparse: true, unique: true } )
```

该索引将允许插入具有该`score`字段唯一值或不包含该`score`字段的文档。这样，鉴于`scores`集合中现有的文档，索引允许进行以下[插入操作](https://docs.mongodb.com/master/tutorial/insert-documents/)：

```powershell
db.scores.insert( { "userid": "AAAAAAA", "score": 43 } )
db.scores.insert( { "userid": "BBBBBBB", "score": 34 } )
db.scores.insert( { "userid": "CCCCCCC" } )
db.scores.insert( { "userid": "DDDDDDD" } )
```

但是，索引不允许添加以下文件，因为文件已经存在，其**score**值为**82**和**90**:

```powershell
db.scores.insert( { "userid": "AAAAAAA", "score": 82 } )
db.scores.insert( { "userid": "BBBBBBB", "score": 90 } )
```

