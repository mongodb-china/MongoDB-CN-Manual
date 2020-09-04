## 通过设置TTL使集合中的数据过期

**在本页面**

- [程序](#程序)

本文介绍了MongoDB的“生存时间”或[TTL](https://docs.mongodb.com/master/reference/glossary/#term-ttl)集合特性。**TTL**集合使它可能存储数据在MongoDB和有[' mongod '](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)自动删除数据后指定的秒数或在一个特定的时钟时间。

数据过期对于某些类别的信息很有用，包括机器生成的事件数据、日志和会话信息，这些信息只需要持续有限的一段时间。

一个特殊的[TTL索引属性](https://docs.mongodb.com/master/core/index-ttl/)支持**TTL**集合的实现。**TTL**特性依赖于[' mongod '](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)中的一个后台线程，该线程读取索引中的日期类型值并从集合中删除过期的[document](https://docs.mongodb.com/master/reference/glossary/#term-document)。

### <span id="程序">程序</span>

若要创建[TTL索引](https://docs.mongodb.com/master/core/index-ttl/)，请在其值为[日期](https://docs.mongodb.com/master/reference/bson-types/#document-bson-type-date)或包含[日期值](https://docs.mongodb.com/master/reference/bson-types/#document-bson-type-date)的数组 的字段上使用[`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)带有`expireAfterSeconds`选项 的 方法 。

> 请注意
>
> TTL索引是单个字段索引。复合索引不支持**TTL**属性。有关**TTL**索引的更多信息，请参见[TTL索引](https://docs.mongodb.com/master/core/index-ttl/)。

You can modify the `expireAfterSeconds` of an existing TTL index using the [`collMod`](https://docs.mongodb.com/master/reference/command/collMod/#dbcmd.collMod) command.

#### 在指定的秒数之后使文档过期

要在索引字段之后的指定秒数过期数据，在保存BSON日期类型值或BSON日期类型对象数组的字段上创建一个TTL索引，并在**expireAfterSeconds**字段中指定一个正的非零值。当**' expireAfterSeconds '**字段中的秒数超过索引字段中指定的时间时,文档将过期。

例如，下面的操作在**“log_events”**集合的**“createdAt”**字段上创建一个索引，并指定**“expireAfterSeconds”**的值**“3600”**，将过期时间设置为**“createdAt”**指定的时间之后一小时。

```powershell
db.log_events.createIndex( { "createdAt": 1 }, { expireAfterSeconds: 3600 } )
```

当向**“log_events”**集合添加文档时，将**“createdAt”**字段设置为当前时间:

```powershell
db.log_events.insert( {
   "createdAt": new Date(),
   "logEvent": 2,
   "logMessage": "Success!"
} )
```

当文档的**createdAt**值大于**expireAfterSeconds**中指定的秒数时，MongoDB将自动从**log_events**集合中删除文档。

#### 在特定的时钟时间时过期文档

要使文档在特定的时钟时间过期，首先在一个字段上创建一个**TTL**索引，该字段包含**BSON**日期类型的值或**BSON**日期类型对象的数组*和*指定**expireAfterSeconds**值为**0**。对于集合中的每个文档，将索引日期字段设置为与文档应该过期的时间对应的值。如果索引日期字段包含过去的日期，MongoDB认为文档过期。

例如，以下操作在**“log_events”**集合的**“expireAt”**字段上创建一个索引，并指定**“expireAfterSeconds”**的值为**“0”**:

```powershell
db.log_events.createIndex( { "expireAt": 1 }, { expireAfterSeconds: 0 } )
```

对于每个文档，将**expireAt**的值设置为与文档应该过期的时间对应。例如，下面的[ insert() ](https://docs.mongodb.com/master/reference/method/db.collection.insert/#db.collection.insert)操作添加了一个应该在**July 22, 2013 14:00:00**到期的文档。

```powershell
db.log_events.insert( {
   "expireAt": new Date('July 22, 2013 14:00:00'),
   "logEvent": 2,
   "logMessage": "Success!"
} )
```

当文档' **expireAt**的值大于**expireAfterSeconds**中指定的秒数时，MongoDB将自动从**log_events**集合中删除文档。在本例中是**“0”**秒。因此，数据在指定的**expireAt**值过期。

