# 事务

在本页

- [事务API](https://docs.mongodb.com/manual/core/transactions/#transactions-api)
- [事务和原子性](https://docs.mongodb.com/manual/core/transactions/#transactions-and-atomicity)
- [事务和操作](https://docs.mongodb.com/manual/core/transactions/#transactions-and-operations)
- [事务和会话](https://docs.mongodb.com/manual/core/transactions/#transactions-and-sessions)
- [读策略/写策略/读偏好](https://docs.mongodb.com/manual/core/transactions/#read-concern-write-concern-read-preference)
- [一般信息](https://docs.mongodb.com/manual/core/transactions/#general-information)
- [其他事务问题](https://docs.mongodb.com/manual/core/transactions/#additional-transactions-topics)


在MongoDB中，对单个文档的操作是原子的。因为您可以使用嵌入式文档和数组来捕获单个文档结构中的数据之间的关系，而不是在多个文档和集合之间进行规范化，所以这种单文档原子性消除了许多实际用例中对多文档事务的需求。

对于需要原子性地读写多个文档（在单个或多个集合中）的情况，MongoDB支持多文档事务。使用分布式事务，可以跨多个操作，集合，数据库，文档和分片使用事务。


## 事务API


以下示例重点介绍了事务API的关键组成：

> 译者注，[原文](https://docs.mongodb.com/manual/core/transactions/)可看不同类型的代码

- PYTHON

- JAVA (SYNC)
- NODE.JS
- PHP
- MOTOR
- C
- C++11
- C#

该示例使用新的回调API来处理事务，该API启动事务，执行指定的操作并提交（可能因为错误而中止）。新的回调API还针对 `TransientTransactionError` 或`UnknownTransactionCommitResult` 提交错误合并了重试逻辑。


重点

- 对于MongoDB 4.2（副本集和分片群集）上的事务，客户端必须使用将MongoDB驱动程序更新为MongoDB 4.2。

- 使用驱动程序时，事务中的每个操作必须与会话相关联（即，将每个操作传递在会话中传递）。

参考

比如，在mongo shell中参考 [mongo Shell Example](https://docs.mongodb.com/manual/core/transactions-in-applications/#txn-mongo-shell-example).



## 事务和原子性


分布式事务和多文档事务

从MongoDB 4.2开始，这两个术语是同义词。 分布式事务是指分片群集和副本集上的多文档事务。 从MongoDB 4.2开始，多文档事务（无论是在分片群集或副本集上）也称为分布式事务。

对于需要原子性地读写多个文档（在单个或多个集合中）的情况，MongoDB支持多文档事务：

- **在4.0版中**，MongoDB支持副本集上的多文档事务。

- **在版本4.2中**，MongoDB引入了分布式事务，它增加了对分片群集上多文档事务的支持，并合并了对副本集上多文档事务的现有支持。

要在MongoDB 4.2部署（副本集和分片群集）上使用事务，客户端必须使用为MongoDB 4.2更新的MongoDB驱动程序。

多文档交易是原子性的（即提供“全有或全无”主张）：	

- 提交事务时，将保存在事务中进行的所有数据更改，并在事务外部可见。 也就是说，一个事务在回滚其他事务时将不会提交其某些更改。在提交事务之前，在事务外部看不到在事务中进行的数据更改。

但是，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在所有分片上可见。 例如，如果提交了一个事务，并且在分片A上可以看到写1，但是在分片B上却看不到写2，则外部读取时设置读关注 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22) 结果为可以读取写入1的结果而看不到写入2。

- 当事务中止时，在事务中进行的所有数据更改都将被丢弃，而不会变得可见。 例如，如果事务中的任何操作失败，则事务中止，并且在事务中进行的所有数据更改都将被丢弃，而不会变得可见。


重点

在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。 在许多情况下， [非规范化数据模型（嵌入的文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding) 仍将是最佳选择您的数据和用例。 也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。

有关其他事务使用方面的注意事项（例如运行时限制和oplog大小限制），另请参阅[生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/).

也可参考[提交期间的外部读取](https://docs.mongodb.com/manual/core/transactions-production-consideration/#transactions-prod-consideration-outside-reads)



## 事务和操作


分布式事务可用于多个操作，集合，数据库，文档，以及从MongoDB 4.2分片开始的。

对于事务：

- 您可以在**现有**集合上指定读/写（CRUD）操作。集合可以在不同的数据库中。有关CRUD操作的列表，请参考 [CRUD 操作](https://docs.mongodb.com/manual/core/transactions-operations/#transactions-operations-crud)。

- 您无法写入 [capped](https://docs.mongodb.com/manual/core/capped-collections/) 集合。 （从MongoDB 4.2开始）

- 您无法在`config`，`admin`或`local`数据库中读取/写入集合。

- 您无法写入`system。*`集合。

- 您无法返回支持的操作的查询计划（如 `explain`）。

- 对于在事务外部创建的游标，不能在事务内部调用 [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) 。

- 对于在事务中创建的游标，不能在事务外调用 [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) 。

- 从MongoDB 4.2开始，您不能将 [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#dbcmd.killCursors) 指定为事务的第一个操作。

事务中不允许执行影响数据库目录的操作，例如创建或删除集合或索引。例如，事务不能包含将导致创建新集合的插入操作。请参阅[受限操作](https://docs.mongodb.com/manual/core/transactions/#transactions-ops-restricted)。


提示

创建或删除集合后立即开始事务，如果在事务内访问了该集合，请在创建或者删除时设置write concern为 [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22) ，以确保该事务可以获取所需的锁。

可参考：[事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



### count 操作

要在事务中执行计数操作，请使用 [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#pipe._S_count) 聚合阶段或者 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) （带有 [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 表达式）聚合阶段。

与4.0功能兼容的MongoDB驱动程序提供了一个集合级API `countDocuments(filter, options)` 作为使用带有[`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 的 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 表达式进行计数。4.0驱动程序已弃用 `count()`  API。

从MongoDB 4.0.3开始， [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell提供使用[`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#grp._S_sum) 的 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 表达式进行计数的 [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#db.collection.countDocuments) 方法。


### distinct 操作

在事务中执行不同的操作：

- 对于未分片的集合，可以使用 [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) 方法或者[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) 命令以及具有 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 阶段的聚合管道。

- 对于分片集合，不可以使用 [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) 方法或者[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#dbcmd.distinct) 命令。

要查找分片集合的不同值，请使用带有 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group) 阶段的聚合管道，例如：

  - 代替`db.coll.distinct("x")`，使用

    ```
    db.coll.aggregate([
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  - 代替 `db.coll.distinct("x", { status: "A" })`，使用：

    ```
    db.coll.aggregate([
       { $match: { status: "A" } },
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  管道将游标返回到文档：

  ```
  { "distinctValues" : [ 2, 3, 1 ] }
  ```

  迭代光标以访问结果文档。

### 信息类操作


信息命令在事务中是允许的，如 [`isMaster`](https://docs.mongodb.com/manual/reference/command/isMaster/#dbcmd.isMaster), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#dbcmd.connectionStatus) （以及辅助方法）；但是他们不能是事务中的第一个操作。

### 限制的操作

事务中不允许以下的操作：

- 影响数据库目录的操作，例如创建或删除集合或索引。 例如，事务不能包含将导致创建新集合的插入操作。

[`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#dbcmd.listCollections) 和 [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#dbcmd.listIndexes) 命令及其辅助方法也被排除在外。

- 非CRUD和非信息性操作，例如 [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#dbcmd.getParameter),  [`count`](https://docs.mongodb.com/manual/reference/command/count/#dbcmd.count)等等及其辅助命令。

可参考

- [待处理的DDL操作和事务](https://docs.mongodb.com/manual/core/transactions-production-consideration/#txn-prod-considerations-ddl)

- [事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



## 事务和会话

- 事务与会话关联； 即您开始一个会话的事务。

- 在任何给定时间，一个会话最多只能有一个未完成的事务。

- 使用驱动程序时，事务中的每个操作必须与会话关联。 有关详细信息，请参阅驱动程序专用文档。

- 如果会话结束并且具有打开的事务，则事务中止。



## 读关注/写关注/读偏好

### 事务和读偏好

事务中的操作使用事务级别的[读偏好](https://docs.mongodb.com/manual/core/read-preference/#replica-set-read-preference)。

使用驱动程序，可以在事务开始时设置事务级别的 [读偏好](https://docs.mongodb.com/manual/core/read-preference/#replica-set-read-preference) ：

- 如果未设置事务级别的读取首选项，则事务将使用会话级别的读取首选项。

- 如果未设置事务级别和会话级别的读选项，则事务将使用客户端级别的读偏好。 默认情况下，客户端级别的读选项为[`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary)。

包含读取操作的[多文档事务](https://docs.mongodb.com/manual/core/transactions/#)必须使用读偏好[`primary`](https://docs.mongodb.com/manual / core / read-preference /＃primary)。 给定事务中的所有操作都必须路由到同一成员。


### 事务和读关注


事务中的操作使用事务级别的[读关注](https://docs.mongodb.com/manual/reference/read-concern/)。 也就是说，在事务内部忽略在集合和数据库级别设置的任何读取关注。

您可以在事务开始时设置事务级别的[读关注](https://docs.mongodb.com/manual/reference/read-concern/)。

- 如果未设置事务级读关注，则事务级读关注默认为会话级读关注。

- 如果未设置事务级别和会话级别的读关注，则事务级别的读取关注点默认为客户端级别的读关注。 默认情况下，对于主服务器的读取，客户端级别的读关注为[`“ local”`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)。 另请参见[事务和读选项](https://docs.mongodb.com/manual/core/transactions/#transactions-read-preference)。

事务支持一下读关注级别：


#### `"local"`

- 读关注点[`“ local”`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)返回该节点可用的最新数据，但可以回滚。

- 对于分片群集上的事务，[`“ local”`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)读关注不能保证数据是从整个分片的同一快照视图获取。 如果需要快照隔离，请使用[“snapshot”](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern-snapshot)读关注。

#### `"majority"`

- **如果**以[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)提交事务，读关注 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)返回大多数副本成员已确认的数据（即无法回滚数据）。

- 如果事务未使用 [写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern) 进行提交，则[`“majority”`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)读关注**不**保证读操作可以读取多数提交的数据。

- 对于分片群集上的事务，[`“majority”`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)读取关注不能保证数据是从整个分片的同一快照视图中获取。 如果需要快照隔离，请使用[“snapshot”](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern-snapshot)读关注。



#### `"snapshot"`

-  **如果**事务提交时带有[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)，读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)从大多数已提交数据的快照中返回数据。

- 如果事务未使用[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)进行提交，则[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)读关注**不**保证读操作使用了大部分提交的数据的快照。

- 对于分片群集上的事务，数据**的[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)视图 **跨分片同步。



### 事务和写关注


事务使用事务级别的[写关注](https://docs.mongodb.com/manual/reference/write-concern/)进行写操作。 必须在没有显式写关注规范的情况下发出事务内部的写操作，并使用默认写关注。 在提交时，然后使用事务级写关注来提交写操作。

提示：

不要为事务中的各个写操作明确设置写关注。 为事务中的各个写操作设置写关注点将导致错误。

您可以在事务开始时设置事务级别的[写关注](https://docs.mongodb.com/manual/reference/write-concern/)：

- 如果未设置事务级写关注，则事务级写关注默认为提交的会话级写关注。

- 如果未设置事务级写关注和会话级写关注，则事务级写关注默认为客户端级写关注。 默认情况下，客户端级别的写入问题为[`w：1`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%3Cnumber%3E)。

事务支持所有写关注[w](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)值，包括：


#### `w: 1`

- 在主节点提交写关注[w：1](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%3Cnumber%3E)后返回确认。

重要

当您使用[`w：1`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%3Cnumber%3E)提交时，您的事务[如果存在故障则可以回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/)。

- 当您提交[`w：1`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%3Cnumber%3E)时，会写成事物级别的[`“majority”` ](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)读关注，**不**保证事务中的读取操作会读取多数提交的数据 。

- 当您提交[`w：1`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%3Cnumber%3E)时，会写成事务级别的[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)，读取关注**不**保证事务中的读取操作使用多数快照提交的数据。


#### `w: "majority"`

- 在提交已应用于多数（ M）有投票权的成员后，写关注[`w：“majority”`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)返回确认； 即提交已应用于主要和（M-1）个投票辅助。

- 当您提交[`w：“ majority”`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)时，事务级别的[`“ majority “`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)读关注保证了操作已读取多数提交的数据。 对于分片群集上的事务，大多数分批提交的数据的视图不会在分片之间同步。

- 当您使用[`w：“majority”`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.%22majority%22)提交时，事务级别[“快照 “](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)读关注保证操作来自大多数提交的数据的同步快照。

注意

不管[事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)，分片集群事务的提交操作都包含使用` {w：“多数”，j：是}`写关注。


## 一般信息

### 注意事项

有关使用事务的各种注意事项，请参阅[注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)。 另外，分片群集，另请参见[注意事项（分片群集）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)。

### 仲裁者

如果任何事务操作读取或写入包含仲裁程序的分片，则其写操作跨越多个分片的事务将出错并中止。

另请参见[禁用读关注Majority](https://docs.mongodb.com/manual/core/transactions/#transactions-disabled-rc-majority)，以了解已禁用读关注majority的分片的事务限制。


### 禁用读关注Majority

一个含有3成员PSA（主-次-仲裁器）副本集，或具有3成员PSA分片的分片群集可能已禁用读关注Majority（[`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-enablemajorityreadconcern) 或[`replication.enableMajorityReadConcern：false`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern))

- 对于分片集群：

如果事务涉及的分片具有[禁用读关注Majority](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority)，则事务中不能使用的读关注[`“快照”`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)。 您只能在事务中使用读关注的[`“ local”`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)或[`“majority”`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)。 如果使用读关注[`“ snapshot”`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern.%22snapshot%22)，则事务错误并中止。 

 `当enableMajorityReadConcern = false时，分片群集中不支持读关注级别 'snapshot' 。`

如果任何事务的读或写操作涉及禁用了读关注`"majority"`的分片，则其写操作跨越多个分片的事务将出错并中止。

- 在[已禁用读关注“majority”](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority)副本集中，您可以指定读关注[`“ local”`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern.%22local%22)或[`“majority”`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern.%22majority%22)或[`“snapshot”`](https://docs.mongodb.com/manual/reference /read-concern-snapshot/#readconcern.%22snapshot%22)。但是，如果您打算过渡到具有禁用读关注majority分片的分片群集，则避免使用读关注的`"snapshot"`。

提示：

检查是否已禁用读关注“majority”，可以在 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)上运行[`db.serverStatus（）`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus)并检查[`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.storageEngine.supportsCommittedReads)字段。 如果为“ false”，则禁用读关注“majority” 。

有关更多信息，请参阅[3-成员主-从-仲裁者体系结构](https://docs.mongodb.com/manual/core/transactions-production-consideration/#transactions-psa)和[三成员主-从-仲裁者 分片](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#transactions-sharded-clusters-psa)。



### 分片配置限制

您不能在[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为“ false”的分片群集上运行事务（ 如 使用[内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)的具有投票成员的分片）。

注意

不管[为事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)，分片集群事务的提交操作都包含使用` {w：“majority”，j：true}`写关注。


### 诊断

MongoDB提供了各种指标：


| 方法                                                         |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus) 方法[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus)命令 | 返回 [事务](https://docs.mongodb.com/manual/reference/command/serverStatus/#server-status-transactions) 指标。 |
| [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#pipe._S_currentOp) 聚合管道 | 如果操作是事务的一部分，则返回[`$currentOp.transaction`](https://docs.mongodb.com/manual/reference/operator/aggregation/current)。 [无效会话](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#currentop-stage-idlesessions) 的信息作为事务的一部分持有锁。[`$currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#_S_currentOp.twoPhaseCommitCoordinator) 这些指标涉及向多个分片写入的分片事务。 |
| [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp) 方法[`currentOp`](https://docs.mongodb.com/manual/reference/command/currentOp/#dbcmd.currentOp) 命令 | 如果操作是事务的一部分，则返回[`currentOp.transaction`](https://docs.mongodb.com/manual/reference/command/currentOp/#currentOp.transaction) 。[`currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/command/currentOp/#currentOp.twoPhaseCommitCoordinator) 这些指标涉及写入多个分片的分片事务。 |
| [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 和 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#b)日志信息 | 包括[`TXN`](https://docs.mongodb.com/manual/reference/log-messages/#TXN)日志组件下慢事务信息（即超过[`operationProfiling.slowOpThresholdMs`](https://docs.mongodb.com/manual/reference/configuration-options/#operationProfiling.slowOpThresholdMs)阈值的事务信息）。 |



### 功能兼容版本 (FCV)


要使用事务，部署的所有成员的[featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv)必须至少为：

| 部署             | Minimum `featureCompatibilityVersion `//最小FCV |
| ---------------------- | ----------------------------------------------- |
| 副本     | `4.0`                                           |
| 分片 | `4.2`                                           |

要检查成员的FCV，请连接到该成员并运行以下命令：

`db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )`

有关更多信息，请参见[`设置FCV`](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#dbcmd.setFeatureCompatibilityVersion)参考页。



### 存储引擎


从MongoDB 4.2开始，副本集和分片群集支持[多文档事务](https://docs.mongodb.com/manual/core/transactions/#)：

- 主节点使用WiredTiger存储引擎，并且

- 从成员使用WiredTiger存储引擎或[内存](https://docs.mongodb.com/manual/core/inmemory/)存储引擎

在MongoDB 4.0中，仅使用WiredTiger存储引擎的副本集支持事务。

注意：

您无法在具有[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为“ false”的分片的分片集群上运行事务，例如使用[内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)具有投票成员的分片。


## 其他事务话题


[驱动 API](https://docs.mongodb.com/manual/core/transactions-in-applications/)

[注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)

[注意事项 (分片集群)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)

[事务和操作](https://docs.mongodb.com/manual/core/transactions-operations/)



译者：王金铷

原文链接：https://docs.mongodb.com/manual/core/transactions/

