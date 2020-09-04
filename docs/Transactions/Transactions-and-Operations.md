# 事务操作

本页中

- 支持多文档事务的操作

- [CRUD 操作](https://docs.mongodb.com/manual/core/transactions-operations/#crud-operations)
  
- [Count 操作](https://docs.mongodb.com/manual/core/transactions-operations/#count-operation)
  
- [Distinct 操作](https://docs.mongodb.com/manual/core/transactions-operations/#distinct-operation)
  
- [信息操作](https://docs.mongodb.com/manual/core/transactions-operations/#informational-operations)
  
- [限制的操作](https://docs.mongodb.com/manual/core/transactions-operations/#restricted-operations)


对于事务来说：

- 您可以在现有集合上指定读/写（CRUD）操作。其中集合可以在不同的数据库中。有关CRUD操作的列表，请参见 [CRUD 操作](https://docs.mongodb.com/manual/core/transactions-operations/#transactions-operations-crud)。

- 您无法写入 [capped](https://docs.mongodb.com/manual/core/capped-collections/) 集合。 （从MongoDB 4.2开始）

- 您无法在config，admin或local数据库中读写集合。

- 您无法写入`system.*`集合。

- 您无法返回支持的操作的查询计划（如：explain）。

- 对于在事务外部创建的游标，不能在事务内部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) 命令。

- 对于在事务中创建的游标，不能在事务外调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) 。

- 从MongoDB 4.2开始，您不能在 [事务](https://docs.mongodb.com/manual/core/transactions/)中将 [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#dbcmd.killCursors) 作为第一个操作。

多文档事务中不允许执行影响数据库目录的操作，例如创建或删除集合或索引。例如，多文档事务不能包含将导致创建新集合的插入操作。请参阅[限制的操作](https://docs.mongodb.com/manual/core/transactions-operations/#transactions-operations-ref-restricted)。



## 多文档事务支持的操作


### CRUD 操作


事务中允许以下读/写操作：

| 方法                                                 | 命令                                                | 备注                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate) | [`aggregate`](https://docs.mongodb.com/manual/reference/command/aggregate/#dbcmd.aggregate) | 不包括以下阶段：[`$collStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/collStats/#pipe._S_collStats)[`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#pipe._S_currentOp)[`$indexStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#pipe._S_indexStats)[`$listLocalSessions`](https://docs.mongodb.com/manual/reference/operator/aggregation/listLocalSessions/#pipe._S_listLocalSessions)[`$listSessions`](https://docs.mongodb.com/manual/reference/operator/aggregation/listSessions/#pipe._S_listSessions)[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge)[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)[`$planCacheStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/planCacheStats/#pipe._S_planCacheStats) |
| [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#db.collection.countDocuments) |                                                              | 不包含以下查询运算符表达式：[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where)[`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near)[`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) 。该方法使用[`$match`](https://docs.mongodb.com/manual/reference/operator/aggregation/match/#pipe._S_match)聚合阶段进行查询，并使用[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)聚合阶段带有[`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum)表达式来执行计数。 |
| [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) | [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) | 在未分片集合中可用。对于分片集合，请在 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)阶段使用聚合管道。可查看[Distinct Operation](https://docs.mongodb.com/manual/core/transactions-operations/#transactions-operations-distinct)。 |
| [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) | [`find`](https://docs.mongodb.com/manual/reference/command/find/#dbcmd.find) |                                                              |
|                                                              | [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#dbcmd.geoSearch) |                                                              |
| [`db.collection.deleteMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany)[`db.collection.deleteOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)[`db.collection.remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#db.collection.remove) | [`delete`](https://docs.mongodb.com/manual/reference/command/delete/#dbcmd.delete) |                                                              |
| [`db.collection.findOneAndDelete()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndDelete/#db.collection.findOneAndDelete)[`db.collection.findOneAndReplace()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace)[`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate) | [`findAndModify`](https://docs.mongodb.com/manual/reference/command/findAndModify/#dbcmd.findAndModify) | 仅在针对现有集合运行时使用`upsert`。 |
| [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)[`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)[`db.collection.insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert) | [`insert`](https://docs.mongodb.com/manual/reference/command/insert/#dbcmd.insert) | 仅在针对现有集合运行时使用。 |
| [`db.collection.save()`](https://docs.mongodb.com/manual/reference/method/db.collection.save/#db.collection.save) |                                                              | 如果插入，则仅在针对现有集合运行时。 |
| [`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)[`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)[`db.collection.replaceOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)[`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update) | [`update`](https://docs.mongodb.com/manual/reference/command/update/#dbcmd.update) | 仅在针对现有集合运行时使用`upsert`。 |
| [`db.collection.bulkWrite()`](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)Various [Bulk Operation Methods](https://docs.mongodb.com/manual/reference/method/js-bulk/) |                                                              | 如果插入，则仅在针对现有集合运行时。仅在针对现有集合运行时使用`upsert`。 |


分片键值更新

从MongoDB 4.2开始，您可以通过在事务中发出单文档update / findAndModify操作或作为[可重试写](https://docs.mongodb.com/manual/core/retryable-writes/)来更新文档的分片键值（除非分片键字段是不可变的_id字段）。有关详细信息，请参见 [更改文档的分片健值](https://docs.mongodb.com/manual/core/sharding-shard-key/#update-shard-key).



### 计数操作

要在事务中执行计数操作，请使用 [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#pipe._S_count) 聚合阶段或 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) （含 [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 表达式）聚合阶段。

与4.0功能兼容的MongoDB驱动程序提供了一个集合级API`countDocuments（filter，options）`作为辅助方法，该方法使用带有 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 表达式和[`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 表达式的计数方法。 4.0驱动程序已弃用`count（）`API。

从MongoDB 4.0.3开始， [`mongo shell`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) 提供了[`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#db.collection.countDocuments) 方法，该方法使用 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 和 [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 表达式来执行计数。



### Distinct 操作


在事务中执行distinct操作：


- 对于未分片的集合，可以使用 [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) 方法/[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) 命令以及具有 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 阶段的聚合管道。


- 对于分片集合，不能使用 [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) 方法或 [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) 命令。


要查找分片集合的distinct值，请使用带有 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 阶段的聚合管道。例如：

  - 替代 `db.coll.distinct("x")`，使用：

    ```
    db.coll.aggregate([
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  - 替代 `db.coll.distinct("x", { status: "A" })`，使用

    ```
    db.coll.aggregate([
       { $match: { status: "A" } },
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  对于一个文档，管道将返回一个游标：

  ```
  { "distinctValues" : [ 2, 3, 1 ] }
  ```

迭代游标以访问文档结果。


### 信息操作


信息命令，例如 [`isMaster`](https://docs.mongodb.com/manual/reference/command/isMaster/#dbcmd.isMaster), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#dbcmd.connectionStatus) （及其辅助方法）在事务中是被允许的；但是，它们不能是事务中的第一个操作。


## 受限制的操作

事务中不允许执行以下操作：

- 影响数据库目录的操作，如创建或删除集合或索引。例如，事务不能包含会导致创建新集合的插入操作。
  
  [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#dbcmd.listCollections) 和 [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#dbcmd.listIndexes) 命令及其辅助方法也被排除在外。

- 非CRUD和非信息性操作，例如 [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#dbcmd.count)等等及其辅助方法。


另请参见：

[待处理的DDL操作和事务](https://docs.mongodb.com/manual/core/transactions-production-consideration/#txn-prod-considerations-ddl)



原文链接：https://docs.mongodb.com/manual/core/transactions-operations/

译者：王金铷
