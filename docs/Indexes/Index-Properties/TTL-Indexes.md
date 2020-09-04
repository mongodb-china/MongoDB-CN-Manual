## TTL索引

**在本页面**

- [行为](#行为)
- [限制](#限制)

> 请注意
>
> 如果您要删除文档以节省存储成本，考虑[MongoDB Atlas](https://www.mongodb.com/cloud?jmp=docs)中的[Online Archive](https://docs.atlas.mongodb.com/online-archive/manage-online-archive)。在线归档自动将不经常访问的数据归档到完全托管的S3 bucket，从而实现经济有效的数据分层。

TTL索引是一种特殊的单字段索引，MongoDB可以使用它在一定的时间或特定的时钟时间后自动从集合中删除文档。数据过期对于某些类型的信息很有用，比如机器生成的事件数据、日志和会话信息，这些信息只需要在数据库中保存有限的时间。

要创建一个TTL索引,使用[db.collection.createIndex ()](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/ db.collection.createIndex)方法**expireAfterSeconds**选项字段的值是一个[日期](https://docs.mongodb.com/master/reference/bson-types/ document-bson-type-date)或一个数组,其中包含[日期值](https://docs.mongodb.com/master/reference/bson-types/).

例如，要在` eventlog`集合的`lastModifiedDate`字段上创建一个TTL索引，在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中使用以下操作:

```powershell
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```

### <span id="行为">行为</span>

#### 过期的数据

TTL索引会在指定的秒数之后使文档过期；即:过期阈值是索引字段值加上指定的秒数。

如果字段是一个数组，并且索引中有多个日期值，MongoDB使用数组中的最低(即最早)日期值来计算过期阈值。

如果文档中的索引字段不是[date](https://docs.mongodb.com/master/reference/glossary/#term-bson-types)或包含日期值的数组，文档将不会过期。

如果文档不包含索引字段，则文档将不会过期。

#### 删除操作

在后台线程中的[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)读取索引中的值并从集合中删除过期的[document](https://docs.mongodb.com/master/reference/glossary/#term-document)。

当TTL线程处于活动状态时，您将[`db.currentOp()`](https://docs.mongodb.com/master/reference/method/db.currentOp/#db.currentOp)在[数据库概要分析器](https://docs.mongodb.com/master/tutorial/manage-the-database-profiler/#database-profiler)的输出或数据中看到删除操作。

##### 删除操作的时间

一旦索引在[主数据库](https://docs.mongodb.com/master/reference/glossary/#term-primary)上构建完成，MongoDB就开始删除过期的文档。有关索引构建过程的更多信息，请参见[填充集合](https://docs.mongodb.com/master/core/index-creation/#index-operations)上的索引构建。

TTL索引不能保证过期数据在过期时立即删除。在文档过期和MongoDB从数据库中删除文档之间可能存在延迟。

删除过期文档的后台任务*每60秒运行一次*。因此，文档可能在文档到期和后台任务运行之间保持在集合中。

因为移除操作的持续时间取决于你的[`mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例的工作负载，过期的数据可能存在一段时间*超过*运行后台任务的60秒周期。

##### 复制集

在[副本集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)成员上，仅当成员处于[primary](https://docs.mongodb.com/master/reference/glossary/#term-primary)状态时，TTL后台线程*才会*删除文档。当成员处于[辅助](https://docs.mongodb.com/master/reference/glossary/#term-secondary)状态时，TTL背景线程处于空闲状态。[次要](https://docs.mongodb.com/master/reference/glossary/#term-secondary)成员从主要成员复制删除操作。

#### 支持查询

TTL索引支持查询的方式与非TTL索引相同。

## <span id="限制">限制</span>

- TTL索引是单字段索引。[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)不支持TTL，并且忽略该 `expireAfterSeconds`选项。
- 该`_id`字段不支持TTL索引。
- 您无法在[上限集合](https://docs.mongodb.com/master/core/capped-collections/)上创建TTL索引，因为MongoDB无法从上限集合中删除文档。
- 您不能用于[`createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)更改`expireAfterSeconds`现有索引的值。而是将 [`collMod`](https://docs.mongodb.com/master/reference/command/collMod/#dbcmd.collMod)database命令与[`index`](https://docs.mongodb.com/master/reference/command/collMod/#index)collection标志一起使用 。否则，要更改现有索引的选项的值，必须首先删除索引并重新创建。
- 如果某个字段已经存在非TTL单字段索引，则无法在同一字段上创建TTL索引，因为您无法创建具有相同键规范且仅选项不同的索引。要将非TTL单字段索引更改为TTL索引，必须首先删除该索引，然后使用该`expireAfterSeconds`选项重新创建 。