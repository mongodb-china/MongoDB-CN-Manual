## 部分索引

**在本页面**

- [创建部分索引](#部分)
- [行为](#行为)
- [限制](#限制)
- [例子](#例子)

*新版本3.2.*

部分索引只索引集合中满足指定筛选器表达式的文档。通过索引集合中文档的子集，部分索引可以降低存储需求，并降低创建和维护索引的性能成本。

### <span id="部分">创建部分索引</span>

使用[db.collection.createIndex()](https://docs.mongodb.com/master/reference/db.collection.createindex/ #db.collection.createIndex)方法和**'partialFilterExpression'**选项。**“partialFilterExpression”**选项接受指定筛选条件的文档，使用:

- 等式表达式（即 运算符），`field: value`[`$eq`](https://docs.mongodb.com/master/reference/operator/query/eq/#op._S_eq)
- [`$exists: true`](https://docs.mongodb.com/master/reference/operator/query/exists/#op._S_exists) 表达，
- [`$gt`](https://docs.mongodb.com/master/reference/operator/query/gt/#op._S_gt)，[`$gte`](https://docs.mongodb.com/master/reference/operator/query/gte/#op._S_gte)，[`$lt`](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt)，[`$lte`](https://docs.mongodb.com/master/reference/operator/query/lte/#op._S_lte)表情，
- [`$type`](https://docs.mongodb.com/master/reference/operator/query/type/#op._S_type) 表达式，
- [`$and`](https://docs.mongodb.com/master/reference/operator/query/and/#op._S_and) 只在顶层操作符

例如，下面的操作创建一个复合索引，该索引只对“**rating**”字段大于**5**的文档进行索引。

```powershell
db.restaurants.createIndex(
   { cuisine: 1, name: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
)
```

你可以为所有的MongoDB[索引类型](https://docs.mongodb.com/master/indexes/#index-types),指定一个**partialFilterExpression**选项.

### <span id="行为">行为</span>

#### 查询范围

如果使用索引导致结果集不完整，则MongoDB不会将部分索引用于查询或排序操作。

若要使用部分索引，查询必须将筛选器表达式(或指定筛选器表达式子集的经过修改的筛选器表达式)作为其查询条件的一部分。

例如，给定以下索引:

```powershell
db.restaurants.createIndex(
   { cuisine: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
)
```

下面的查询可以使用索引，因为查询谓词包含条件`“rating: {$gte: 8}”`，它匹配索引筛选器表达式`“rating: {$gt: 5}”`匹配的文档子集:

```powershell
db.restaurants.find( { cuisine: "Italian", rating: { $gte: 8 } } )
```

但是，以下查询不能在**“cuisine”**字段上使用部分索引，因为使用该索引会导致不完整的结果集。具体来说，查询谓词包括条件`rating: {$lt: 8}`，而索引有过滤器`rating: {$gt: 5}`。也就是说，查询`{cuisine: "Italian"， rating: {$lt: 8}}`匹配的文档(例如，一家评级为1的意大利餐厅)比编入索引的文档更多。

```powershell
db.restaurants.find( { cuisine: "Italian", rating: { $lt: 8 } } )
```

类似地，以下查询不能使用部分索引，因为查询谓词不包括筛选器表达式，并且使用索引将返回不完整的结果集。

```powershell
db.restaurants.find( { cuisine: "Italian" } )
```

#### 与“sparse”索引进行比较

> 提示
>
> 部分索引代表**sparse**索引提供的功能的超集，应优先于**sparse**索引。

部分索引提供了一种比[sparse索引](https://docs.mongodb.com/master/core/index-sparse/)索引更有表现力的机制来指定索引哪些文档。

**Sparse**索引根据索引字段的存在性选择文档进行索引，对于复合索引则根据索引字段的存在性选择文档。

部分索引根据指定的筛选器确定索引项。过滤器可以包括索引键以外的字段，并可以指定条件，而不仅仅是存在检查。例如，部分索引可以实现与**sparse**索引相同的行为:

```powershell
db.contacts.createIndex(
   { name: 1 },
   { partialFilterExpression: { name: { $exists: true } } }
)
```

此部分索引支持与**“name”**字段上的**sparse**索引相同的查询。

但是，部分索引还可以在索引键以外的字段上指定筛选器表达式。例如，下面的操作创建了一个部分索引，其中索引在**name**字段上，但是过滤器表达式在**email**字段上:

```powershell
db.contacts.createIndex(
   { name: 1 },
   { partialFilterExpression: { email: { $exists: true } } }
)
```

为了让查询优化器选择此部分索引，查询谓词必须包含**“name”**字段上的条件，以及**“email”**字段上的**非空**匹配。

例如，下面的查询可以使用索引，因为它包括**' name '**字段上的条件和**' email '**字段上的非空匹配:

```powershell
db.contacts.find( { name: "xyz", email: { $regex: /\.org$/ } } )
```

但是，以下查询不能使用索引，因为它在**“email”**字段上包含了一个**null**匹配，这是过滤器表达式`{email: {$exists: true}}`不允许的:

```powershell
db.contacts.find( { name: "xyz", email: { $exists: false } } )
```

### <span id="限制">限制</span>

在MongoDB中，您不能创建仅在选项上有所不同的多个索引版本。因此，您不能创建仅因过滤器表达式而不同的多个部分索引。

您不能同时指定`partialFilterExpression`选项和`sparse`选项。

MongoDB 3.0或更早版本不支持部分索引。要使用部分索引，必须使用MongoDB 3.2或更高版本。对于分片集群或复制集，所有节点必须是版本3.2或更高。

` _id `索引不能是部分索引。

分片键索引不能是部分索引。

### <span id="例子">例子</span>

#### 在集合上创建部分索引

考虑包含类似于以下文档的集合**restaurants**

```powershell
{
   "_id" : ObjectId("5641f6a7522545bc535b5dc9"),
   "address" : {
      "building" : "1007",
      "coord" : [
         -73.856077,
         40.848447
      ],
      "street" : "Morris Park Ave",
      "zipcode" : "10462"
   },
   "borough" : "Bronx",
   "cuisine" : "Bakery",
   "rating" : { "date" : ISODate("2014-03-03T00:00:00Z"),
                "grade" : "A",
                "score" : 2
              },
   "name" : "Morris Park Bake Shop",
   "restaurant_id" : "30075445"
}
```

您可以在`borough`和`cuisine`字段上添加部分索引，仅选择索引`rating.grade` 字段为的文档`A`：

```powershell
db.restaurants.createIndex(
   { borough: 1, cuisine: 1 },
   { partialFilterExpression: { 'rating.grade': { $eq: "A" } } }
)
```

然后，对`restaurants`集合的以下查询使用部分索引返回Bronx中`rating.grade`等于的餐厅`A`：

```powershell
db.restaurants.find( { borough: "Bronx", 'rating.grade': "A" } )
```

但是，以下查询不能使用部分索引，因为查询表达式不包含该`rating.grade`字段：

```powershell
db.restaurants.find( { borough: "Bronx", cuisine: "Bakery" } )
```

#### 具有唯一约束的部分索引

部分索引仅索引集合中符合指定过滤器表达式的文档。如果同时指定 `partialFilterExpression`和[约束](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)，则唯一约束仅适用于满足过滤器表达式的文档。如果文档不符合过滤条件，则具有唯一约束的部分索引不会阻止插入不符合唯一约束的文档。

例如，集合**users**包含以下文档:

```powershell
{ "_id" : ObjectId("56424f1efa0358a27fa1f99a"), "username" : "david", "age" : 29 }
{ "_id" : ObjectId("56424f37fa0358a27fa1f99b"), "username" : "amanda", "age" : 35 }
{ "_id" : ObjectId("56424fe2fa0358a27fa1f99c"), "username" : "rajiv", "age" : 57 }
```

下面的操作创建了一个索引，该索引在**“username”**字段上指定了一个[unique constraint](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)和一个部分过滤表达式`age: {$gte: 21}`。

```powershell
db.users.createIndex(
   { username: 1 },
   { unique: true, partialFilterExpression: { age: { $gte: 21 } } }
)
```

由于指定用户名的文档已经存在，且**“age”**字段大于**21**，因此索引防止插入以下文档:

```powershell
db.users.insert( { username: "david", age: 27 } )
db.users.insert( { username: "amanda", age: 25 } )
db.users.insert( { username: "rajiv", age: 32 } )
```

但是，允许使用重复用户名的以下文档，因为唯一约束只适用于**“age”**大于或等于**21**的文档。

```powershell
db.users.insert( { username: "david", age: 20 } )
db.users.insert( { username: "amanda" } )
db.users.insert( { username: "rajiv", age: null } )
```

