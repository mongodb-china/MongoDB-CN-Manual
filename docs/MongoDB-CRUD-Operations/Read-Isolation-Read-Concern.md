# 读关注
**在本页面**

- [读关注级别](#级别)
- [ReadConcern 支持](#支持)
- [注意事项](#注意)

**读关注** 选项允许你控制从复制集和分片集群读取数据的一致性和隔离性。

通过有效地使用[写关注](https://docs.mongodb.com/manual/reference/write-concern/)和读关注，你可以适当地调整一致性和可用性的保证级别，例如等待以保证更强的一致性，或放松一致性要求以提供更高的可用性。

将MongoDB驱动程序更新到MongoDB 3.2或更高版本以支持读关注。

## <span id="级别">阅读关注级别</span>
以下为可用的读关注级别：

| `level` | Description |
| :--- | :--- |
| [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) | 查询并从实例返回数据，但不能保证该数据已被写入大多数副本集成员（即可能已经回滚）。<br /><br />**默认为：**<br />        针对主节点读。<br />        如果读取与因果一致的会话相关联，则针对副节点读。<br /><br />**可用性：**读关注`local`可用于有或没有[因果关系一致的会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)和事务中。<br /><br />更多的信息，请参考[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)页 |
| [`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22) | 查询并从实例返回数据，但不能保证该数据已被写入大多数副本集成员（即可能已经回滚）。<br /><br />**默认为：**如果读取与[因果关系一致的会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)没有关联，则针对副节点读<br /><br />**可用性：**读关注`available`无法用于有因果关系一致的会话和事务中。<br /><br />对于分片群集，[`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22)读关注提供了各种读关注中尽可能最低的延迟。但是，这是以牺牲一致性为代价的，因为从分片的集合中进行读取时，[`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22)读关注会返回[孤立的文档](https://docs.mongodb.com/manual/reference/glossary/#term-orphaned-document)。为了避免从分片的集合中读取时返回孤立文档的风险，可使用其他读关注，如[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)读关注。<br /><br />更多的信息，请参考[`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22)页<br /><br />*3.6版本的新功能* |
| [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22) | 为了满足读关注“majority”，副本集成员从其内存视图中返回多数提交点提交的数据。这样，读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)在性能成本上可与其他读关注相媲美。<br /><br />**可用性：**<br />读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)可用于有或没有因果关系一致的会话和事务中。<br />对于具有三名成员的主从仲裁（PSA）架构的部署，可以禁用读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)；但是，这对change streams（仅在MongoDB 4.0和更早版本中）和分片群集上的事务有影响。有关更多信息，请参见[禁用读关注Marjority](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority).。<br />**要求：**若要使用[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)的[读关注](https://docs.mongodb.com/manual/reference/glossary/#term-read-concern)级别，副本集必须使用WiredTiger存储引擎。<br /><br />**注意：**<br />对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)中的操作，仅当事务以[写关注`"majority"`](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)提交时，读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)才提供其保证。否则，[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)读关注不能保证其在事务中读取的数据。<br /><br />更多的信息，请参考[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)页 |
| [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22) | 该查询返回的数据表示了这些数据在操作开始之前已成功在大多数节点确认写入。查询可能会等待并发执行的写操作传播到大多数副本集成员，然后返回结果。<br />如果大多数副本集成员崩溃并在读操作后重新启动，则如果将[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为默认状态true，则读操作返回的文档将还是有效的。<br />将[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`时，MongoDB不会等待[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)在确认写入之前先要写入磁盘日志。这样，如果给定副本集中大多数节点的瞬时丢失（例如崩溃和重新启动），`majority`写操作可能会回滚。<br />**可用性：**<br />读关注[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)不适用于因果一致的会话和事务。<br />你可以仅对主节点上的读操作指定为线性读关注。<br />你不能将[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)或[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge)操作与读关注[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate)指定为[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)读关注，则不能在管道中使用任何的操作。<br /><br />**要求：**linearizable读关注仅保证在读操作指定了唯一标识单个文档的查询过滤器时可用。<br /><br />请始终将`maxTimeMS`与linearizable读关注一起使用，以防止大多数数据承载成员不可用。`maxTimeMS`确保操作不会无限期地阻塞，而是确保如果无法满足读取要求，则操作将返回错误。<br />更多的信息，请参考[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)页 |
| [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22) | 如果事务不是[因果一致会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)的一部分，写关注为[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)且在事务提交后，可以确保事务操作已从多数提交数据的快照中读取。<br />如果事务是[因果一致会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)的一部分，写关注为[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)且在事务提交后，可以确保事务操作已从多数提交数据的快照中读取，该快照提供了与紧接事务开始之前的操作的因果一致性。<br />读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)仅可用于多文档事务。<br />对于分片群集上的事务，如果事务中的任何操作涉及[已被禁用读关注“majority”](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority)的分片，那你就不能对该事务使用读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)。你只能对事务使用读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)或[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)。 |

无论[读关注](https://docs.mongodb.com/manual/reference/glossary/#term-read-concern)级别如何，节点上的最新数据都可能无法反映系统中数据的最新版本

有关每个阅读关注级别的更多信息，请参见：

- [读关注 "local"](https://docs.mongodb.com/manual/reference/read-concern-local/)
- [读关注 "available"](https://docs.mongodb.com/manual/reference/read-concern-available/)
- [读关注 "majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/)
- [读关注 "linearizable"](https://docs.mongodb.com/manual/reference/read-concern-linearizable/)
- [读关注 "snapshot"](https://docs.mongodb.com/manual/reference/read-concern-snapshot/)

## <span id="支持">ReadConcern 支持</span>
### 读关注选项

对于不在[多文档事务](https://docs.mongodb.com/manual/core/transactions/)中的操作，你可以将 `readConcern` 级别指定为一个命令和方法的选项：

```powershell
readConcern: { level: <level> }
```
要为 [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell方法 [db.collection.find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) 指定阅读关注级别，请使用 [cursor.readConcern()](https://docs.mongodb.com/manual/reference/method/cursor.readConcern/#cursor.readConcern) 方法：
```powershell
db.collection.find().readConcern(<level>)
```
### 事务和可用的读关注

对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)，应在事务级别而不是在单个操作级别设置读关注。事务中的操作将使用事务级别的读关注。事务内部将忽略在集合和数据库级别设置的任何读关注。如果显式指定了事务级别的读关注点，则在事务内部也将忽略客户端级别的读关注点。

### 重要

不要为各个操作明确设置读关注。要设置事务的读关注，请参阅读 [Read Concern/Write Concern/Read Preference](https://docs.mongodb.com/manual/core/transactions/#transaction-options)。

你可以在事务开始时设置读关注：

- 对于多文档事务，读关注级别[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22), [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) 和 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)是可用的。
- [多文档事务](https://docs.mongodb.com/manual/core/transactions/)中的写命令可以支持事务级别的读关注。

如果未在事务开始时指定，则事务将使用会话级的读关注，或者如果未设置，则使用客户端级的读关注。

有关等多信息，请参考 [事务的读关注](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern).


#### 因果一致的会话和阅读相关的担忧

对于在[因果一致的会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)中的操作，[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) h和 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)级别可用。但是，为了保证因果一致性，你必须使用 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)。有关详细信息，请参见 [因果一致性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)。

如果多文档事务与因果一致的会话相关联，则[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22) 也可用于该事务。

#### 支持读关注的操作
下列的操作支持读关注：

#### 重要

在为事务中的操作设置读关注时，请在事务级别而不是在单个操作级别设置读关注。不要在事务中明确的设置单独操作的读关注。更多信息，查看[事务和读关注](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern)

| 命令/方法 | [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) | [`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22) | [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22) | [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22) | [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [`count`](https://docs.mongodb.com/manual/reference/command/count/#dbcmd.count) | ✓ | ✓ | ✓ |  | ✓ |
| [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) | ✓ | ✓ | ✓ | ✓ | ✓ |
| [`find`](https://docs.mongodb.com/manual/reference/command/find/#dbcmd.find) | ✓ | ✓ | ✓ | ✓ | ✓ |
| [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) via [`cursor.readConcern()`](https://docs.mongodb.com/manual/reference/method/cursor.readConcern/#cursor.readConcern) | ✓ | ✓ | ✓ | ✓ | ✓ |
| [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#dbcmd.geoSearch) | ✓ | ✓ | ✓ | ✓ | ✓ |
| [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) | ✓ |  |  |  | ✓ |
| [`aggregate`](https://docs.mongodb.com/manual/reference/command/aggregate/#dbcmd.aggregate) [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate) | ✓ | ✓ | ✓ | ✓ | ✓ |
| [`Session.startTransaction()`](https://docs.mongodb.com/manual/reference/method/Session.startTransaction/#Session.startTransaction) | ✓ |  | ✓ | ✓ |  |

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [[1]](https://docs.mongodb.com/manual/reference/read-concern/#id5) | 你不能将[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out) 或者 [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge)阶段与读关注的[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate)指定[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)读关注，则不能在管道中包括任何一个阶段。 |
| [[2]](https://docs.mongodb.com/manual/reference/read-concern/#id4) | 读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)仅适用于多文档事务。在事务中，不能在分片集合上使用`distinct`命令或其协助命令。 |

下列的写操作页能接受读关注，但必须是多文档事务的一部分：

#### 重要

在为事务中的操作设置读关注时，请在事务级别而不是在单个操作级别设置读关注

| Command 命令 | [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) | [`"available"`](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern.%22available%22) | [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22) | [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22) | [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [`delete`](https://docs.mongodb.com/manual/reference/command/delete/#dbcmd.delete)<br />[`db.collection.deleteMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany)<br />[`db.collection.deleteOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)<br />[`db.collection.remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#db.collection.remove) | ✓ |  |  | ✓ |  |
| [`findAndModify`](https://docs.mongodb.com/manual/reference/command/findAndModify/#dbcmd.findAndModify)<br />[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)<br />[`db.collection.findOneAndDelete()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndDelete/#db.collection.findOneAndDelete)<br />[`db.collection.findOneAndReplace()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace)<br />[`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate) | ✓ |  |  | ✓ |  |
| [`insert`](https://docs.mongodb.com/manual/reference/command/insert/#dbcmd.insert)<br />[`db.collection.insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert)<br />[`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)<br />[`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | ✓ |  |  | ✓ |  |
| [`update`](https://docs.mongodb.com/manual/reference/command/update/#dbcmd.update)<br />[`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update)<br />[`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)<br />[`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)<br />[`db.collection.replaceOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne) | ✓ |  |  | ✓ |  |

| [3] | _([1](https://docs.mongodb.com/manual/reference/read-concern/#id3), [2](https://docs.mongodb.com/manual/reference/read-concern/#id7))_读关注[“SNAPSHOT”](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot)仅适用于多文档事务，并且对于事务，您可以在事务级别设置读关注。支持[“SNAPSHOT”](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot)的操作对应于事务中可用的CRUD操作。有关更多信息，请参见[事务和读关注](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern) |
| --- | --- |
|  |  |


## <span id="注意">注意事项</span>
### 读自己的文章
在版本3.6中更改

从MongoDB 3.6版本开始，如果写请求确认，你可以使用[因果一致的会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)读你自己写入的内容。

在MongoDB 3.6之前，您必须使用 `{ w: "majority" }` 写关注发出写操作，然后对读操作使用 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22) 或者 [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22)读关注，以确保单个线程可以读取自己的写入内容


### 实时顺序
结合[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22) 写关注，[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22) 读关注使多个线程可以在单个文档上执行读写操作，就好像单个线程实时地执行了这些操作一样。 也就是说，这些读写的对应的计划被认为是线性的。

### 性能比较
与[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)不同，[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22) 的读关注通过从节点确认读操作正在从主节点读，该操作能够以[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)写关注来确认写入。 [[4]](https://docs.mongodb.com/manual/reference/read-concern/#edge-cases-2-primaries)因此，具有线性化读关注的读取可能比具有[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22) 或 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)读关注的读慢得多。

为了避免万一大多数数据承载成员不可用，请始终将 `maxTimeMS` 与可线性化的读确认一起使用。`maxTimeMS` 确保操作不会无限期地阻塞，而是确保如果无法满足读取要求，则操作将返回错误。

例如：

```shell
db.restaurants.find( { _id: 5 } ).readConcern("linearizable").maxTimeMS(10000)
db.runCommand( {
     find: "restaurants",
     filter: { _id: 5 },
     readConcern: { level: "linearizable" },
     maxTimeMS: 10000
} )
```
| [[4]](https://docs.mongodb.com/manual/reference/read-concern/#id8) | 在[某些情况](https://docs.mongodb.com/manual/core/read-preference-use-cases/#edge-cases)下，副本集中的两个节点可能会短暂地认为它们是主节点，但至多，其中一个节点将能够以[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)写关注完成。 可以完成[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)写入的节点是当前主节点，另一个节点是前主节点，由于[网络分区](https://docs.mongodb.com/manual/reference/glossary/#term-network-partition)的原因，该主节点尚未意识到其降级。 发生这种情况时，尽管请求的读优先级为[主节点](https://docs.mongodb.com/manual/core/read-preference/#primary)，但连接到前主界定啊的客户端仍可能会读到过时的数据，并且最终将对前主节点新写入的进行回滚。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

### 读操作和afterClusterTime
3.6 版本新加入

MongoDB 3.6引入了对[因果一致会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)的支持。 对于与因果一致的会话相关联的读操作，MongoDB 3.6引入了 `afterClusterTime` 读关注选项，驱动程序会自动将`afterClusterTime` 读关注选项设置为与因果一致的会话相关联的操作。

> **[warning] 重要**
>
> 不要手动为读操作设置 `afterClusterTime` 。 MongoDB驱动程序会针对与因果一致的会话相关联的操作自动设置此值。 但是，您可以提前会话的操作时间和群集时间，以便与另一个客户端会话的操作保持一致。 有关示例，请参见[示例](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency-examples)。

为了满足 `afterClusterTime` 值为`T`的读请求， [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 必须在其oplog到达时间`T`之后执行请求。如果其oplog尚未达到时间`T`，则 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 必须等待服务该请求。

使用指定的 `afterClusterTime` 的读操作将返回满足读关注级别要求和指定的 `afterClusterTime` 要求的数据。

对于与因果一致会话无关的读操作，未设置 `afterClusterTime`。

#### 阅读问题出处

从4.4版本开始，MongoDB跟踪阅读关注来源，表示某个特定读取关注点的来源。您可能会在[`getLastError`](https://docs.mongodb.com/master/reference/command/serverStatus/#serverstatus.metrics.getLastError)指标、读取关注错误对象和MongoDB日志中看到出处。

下表显示了可能的阅读问题**provenance**值及其重要性:

| 出处              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `clientSupplied`  | read关注点是在应用程序中指定的。                             |
| `customDefault`   | 读取关注点源自自定义的默认值。 参见 [`setDefaultRWConcern`](https://docs.mongodb.com/master/reference/command/setDefaultRWConcern/#dbcmd.setDefaultRWConcern). |
| `implicitDefault` | 在没有其他所有读取关注规范的情况下，读取关注源自服务器。     |



译者：杨帅 张琦

校对：杨帅
