
# 可重试写入
**在本页面**

* [前提条件](#prerequisites)

* [可重试写入和多文档交易](#transactions)

* [启用可重试写入](#enabling)

* [可重试的写操作](#write)

* [行为](#behavior)

*3.6版的新功能*

可重试写入允许MongoDB驱动程序在遇到网络错误或在复制集或分片群集中找不到正常的主操作时自动重试特定的写操作一次。 

## <span id="prerequisites">前提条件</span>

可重试写入具有以下要求：

### 支持的部署Topologie

​        可重试写入需要 [复制集](https://docs.mongodb.com/master/replication/#replication)或[分片群集](https://docs.mongodb.com/master/sharding/#sharding-introduction)，并且不支持独立实例。

### 支持的存储引擎

​        可重试写入需要支持文档级锁定的存储引擎，例如[WiredTiger](https://docs.mongodb.com/master/core/wiredtiger/)或[内存中](https://docs.mongodb.com/master/core/inmemory/) 存储引擎。

### 3.6+ MongoDB驱动程序

客户端需要为MongoDB 3.6或更高版本更新的MongoDB驱动程序：

| Java 3.6+<br />Python 3.6+<br />C 1.9+ | C# 2.5+<br />Node 3.0+<br />Ruby 2.5+ | Perl 2.0+<br />PHPC 1.4+<br />Scala 2.2+ |
| -------------------------------------- | ------------------------------------- | ---------------------------------------- |
|                                        |                                       |                                          |

### MongoDB版本

集群中每个节点的MongoDB版本必须为**3.6**或更高，集群中每个节点的**featureCompatibilityVersion**必须为**3.6**或更高。有关**featureCompatibilityVersion**标志的更多信息，请参见[setFeatureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#dbcmd.setFeatureCompatibilityVersion)。

### 写确认书

[`Write Concern`](https://docs.mongodb.com/master/reference/write-concern/)为**0**的写操作是不可重试的。

## <span id="transactions">可重试写入和多文档交易</span>

*版本4.0中的新功能*

[事务提交和中止操作](https://docs.mongodb.com/master/core/transactions-in-applications/#transactions-retry)是可重试的写操作。如果提交操作或中止操作遇到错误，MongoDB驱动程序将重试操作一次，而不管[`retryWrites`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites)是否被设置为**false**。

有关交易的更多信息，请参见[Transactions](https://docs.mongodb.com/master/core/transactions/)。

## <span id="enabling">启用可重试写入</span>

### MongoDB驱动程序

官方的MongoDB 3.6和4.0兼容驱动程序需要在连接字符串中包含[`retryWrites=true`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites)选项，以启用该连接的可重试写操作。

官方的MongoDB 4.2兼容驱动程序在默认情况下启用了可重试写。升级到与4.2兼容的驱动程序，要求可重试写的应用程序可能会忽略[`retryWrites=true`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites)选项。升级到与4.2兼容的驱动程序，要求禁用可重试写的应用程序必须在连接字符串中包含[`retryWrites=false`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites)。

**Mongo shell**

要在mongo shell中启用可重试写入，请使用[`--retryWrites`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-retrywrites)命令行选项：

```shell
mongo --retryWrites
```

## <span id="write">可重试的写操作</span>

当发出已确认的写关注时，可以重试以下写操作; 例如,[`Write Concern`](https://docs.mongodb.com/manual/reference/write-concern/)不能为**{w：0}**。

> **[success] Note**
>
> 事务中的写操作不能单独重试。

| 方法 | 说明 |
| --- | --- |
| [db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)<br />[db.collection.insert()](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert)<br />[db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | 插入操作 |
| [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)<br />[db.collection.replaceOne()](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)<br />[db.collection.save()](https://docs.mongodb.com/manual/reference/method/db.collection.save/#db.collection.save)<br />[db.collection.update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update) where `multi` is `false` | 单文档更新操作。 |
| [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)<br />[db.collection.remove()](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#db.collection.remove) where justOne is true | 单个文档删除操作 |
| [db.collection.findAndModify()](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)<br />[db.collection.findOneAndDelete()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndDelete/#db.collection.findOneAndDelete)<br />[db.collection.findOneAndReplace()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace)<br />[db.collection.findOneAndUpdate()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate) | **findAndModify**操作。所有**findAndModify**操作都是单个文档操作。 |
| <br /> [db.collection.bulkWrite()](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite) 具有以下写操作：<br />. [insertOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-insertone)<br />. [updateOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-updateonemany)<br />. [replaceOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-replaceone)<br />. [deleteOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-deleteonemany)<br /> | 只包含单文档写操作的批量写操作。可重试的大容量操作可以包括指定的写操作的任何组合，但不能包括任何多文档写操作，比如**updateMany**。 |
| [Bulk](https://docs.mongodb.com/manual/reference/method/Bulk/#Bulk) operations for:<br />. [Bulk.find.removeOne()](https://docs.mongodb.com/manual/reference/method/Bulk.find.removeOne/#Bulk.find.removeOne)<br />. [Bulk.find.replaceOne()](https://docs.mongodb.com/manual/reference/method/Bulk.find.replaceOne/#Bulk.find.replaceOne)<br />. [Bulk.find.replaceOne()](https://docs.mongodb.com/manual/reference/method/Bulk.find.replaceOne/#Bulk.find.replaceOne)<br /> | 仅由单文档写操作组成的批量写操作。可重试的大容量操作可以包括指定的写操作的任何组合，但不能包括任何多文档写操作，比如**update**，它为**multi**选项指定**true**。 |

> **分片键值更新**
>
> 从MongoDB 4.2开始，您可以通过发布可重试写入或事务处理中的单文档**update / findAndModify**操作来更新文档的分片键值(除非分片键字段是不可变的**_id**字段)。 有关详细信息，请参见[更改文档的分片键值](https://docs.mongodb.com/master/core/sharding-shard-key/#update-shard-key).。

* MongoDB 4.2将重试遇到重复密钥异常的某些单文档upsert（更新使用**upsert：true**和**multi：false**）。 有关条件，请参阅 [Duplicate Key Errors on Upsert](https://docs.mongodb.com/master/core/retryable-writes/#retryable-update-upsert) .
* 在MongoDB 4.2之前，MongoDB不会重试遇到重复键错误的upsert操作。

## <span id="behavior">行为</span>

### 持续的网络错误

MongoDB可重试写只做一次重试尝试。这有助于解决暂时的网络错误和复制集选举，但不能解决持久的网络错误。

### 故障转移期

如果驱动程序在目标复制集中或分片集群分片中找不到正常的主服务器，则驱动程序在重试之前会等待[`serverSelectionTimeoutMS`](https://docs.mongodb.com/manual/reference/connection-string/#urioption.serverSelectionTimeoutMS)毫秒来确定新的主服务器。可重试写操作不会处理故障转移周期超过[`serverSelectionTimeoutMS`](https://docs.mongodb.com/manual/reference/connection-string/#urioption.serverSelectionTimeoutMS)的实例。

> **[warning]  Warning**
>
> 如果客户端应用程序在发出写操作后的时间超过[`localLogicalSessionTimeoutMinutes`](https://docs.mongodb.com/master/reference/parameters/#param.localLogicalSessionTimeoutMinutes)，那么当客户端应用程序开始响应时(无需重新启动)，可能会重试并再次应用写操作。

### Upsert上的重复键错误

MongoDB 4.2将重试单文档的upsert操作(即:**upsert: true**和**multi: false**)由于重复的键错误而失败，只有当操作满足以下所有条件:

* 目标集合具有导致重复键错误的唯一索引。

* 更新匹配条件为：
  * 单个相等谓词

    **{ "fieldA" : "valueA" }**，

  * 相等谓词的逻辑

    **{ "fieldA" : "valueA", "fieldB" : "valueB" }**

* 唯一索引键模式中的字段集与更新查询谓词中的字段集匹配。

* 更新操作不会修改查询谓词中的任何字段。

  下表包含服务器可以或不能在重复键错误时重试的upsert操作示例：

| **唯一索引键模式**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | **更新操作**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | **可重试** |
| ---- | --- | --- |
| {  _id  ： **1**  } | db.collName.updateOne(  <br />      { _id : ObjectId("**1aa1c1efb123f14aaa167aaa**") },  <br/>      { $set : { fieldA : **25** } },  <br />      { upsert : **true** } <br />) | 是 |
| {  fieldA  ： **1**  } | db.collName.updateOne(  <br />        { fieldA : { $in : [ **25** ] } },  <br />        { $set : { fieldB : "**someValue**" } }, <br />        { upsert : **true** } <br />) | 是 |
| {<br />       fieldA：**1**，<br />  fieldB  ：**1**<br />} | db.collName.updateOne(  <br />      { fieldA : **25**, fieldB : "**someValue**" },  <br />      { $set : { fieldC : **false** } },  <br />      { upsert : **true** } <br />) | 是 |
| {  fieldA  ： **1**  } | db.collName.updateOne(  <br />      { fieldA : { $lte : **25** } }, <br />      { $set : { fieldC : **true** } },  <br />      { upsert : **true** } <br />) | 没有<br />查询谓词**fieldA**不等于 |
| {  fieldA  ： **1**  } | db.collName.updateOne(  <br />      { fieldA : { $in : [ **25** ] } },   <br />      { $set : { fieldA : **20** } }, <br />      { upsert : **true** } <br />) | 没有<br />更新操作修改查询谓词中指定的字段。 |
| {  _id  ： **1**  } | db.collName.updateOne(  <br />       { fieldA : { $in : [ **25** ] } }, <br />       { $set : { fieldA : **20** } },  <br />       { upsert : **true** } <br />) | 没有<br />查询谓词字段集（**fieldA**）与索引关键字字段集（）不匹配**_id**。 |
| {  fieldA  ： **1**  } | db.collName.updateOne( <br />       { fieldA : 25, fieldC : **true** }, <br />       { $set : { fieldD : **false** } }, <br />       { upsert : **true** } <br />) | 没有<br />这组查询谓词的字段（**fieldA**，**fieldC**）不匹配组索引键的字段（**fieldA**） |

在MongoDB 4.2之前，MongoDB可重试写不支持由于重复的键错误而失败的重试更新。

### 诊断程序

*版本3.6.3中的新功能*

[`serverStatus`](https://docs.mongodb.com/master/reference/command/serverStatus/#dbcmd.serverStatus)命令及其mongo shell帮助程序 [`db.serverStatus()`](https://docs.mongodb.com/master/reference/method/db.serverStatus/#db.serverStatus) 在[`transactions`](https://docs.mongodb.com/master/reference/command/serverStatus/#serverstatus.transactions)节中包含有关可重试写入的统计信息。

**针对本地数据库的可重试写入**

官方的MongoDB 4.2系列驱动程序默认情况下启用重试写入。 除非明确禁止重试写入，否则写入本地数据库的应用程序在升级到4.2系列驱动程序时将遇到写入错误。

要禁用可重试写入，请在MongoDB集群的[连接字符串](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-uri)中指定[`retryWrites=false`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites) 。



译者：杨帅

校对：杨帅