

## 生产注意事项



以下内容列出了运行事务的一些生产注意事项。无论是在副本集还是分片集群上运行事务，这些都适用。要在分片集群上运行事务，另请参阅[生产注意事项（分片集群）](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/)来了解专门针对分片集群的额外注意事项。



## 可用性


- **在4.0版本**，MongoDB支持副本集上的多文档事务。

- **在4.2版本**，MongoDB引入了分布式事务，增加了对分片集群上多文档事务的支持，并整合了已有的对副本集上多文档事务的支持。

  要在MongoDB 4.2（副本集和分片集群）中使用事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。

> 注意
> 
> **分布式事务和多文档事务**
> 
> 从MongoDB 4.2开始，这两个术语是同义词。分布式事务是指分片集群和副本集上的多文档事务。从MongoDB 4.2开始，多文档事务（无论是在分片集群上还是副本集上）也称为分布式事务。





## 功能兼容性


要使用事务，所有成员的[featureCompatibilityVersion](https://docs.mongodb.com/v4.4/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)必须至少满足：

| 部署方式         | 最小`featureCompatibilityVersion`      |
| :-------------- | :------------------------------------ |
| Replica Set     | `4.0`                                 |
| Sharded Cluster | `4.2`                                 |

要检查成员的fCV，可以连接到该成员并运行以下命令：

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

更多信息，请参阅[`setFeatureCompatibilityVersion`](https://docs.mongodb.com/v4.4/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)参考页面。



## 运行时限制


默认情况下，事务的运行时间必须少于一分钟。可以使用[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)修改[`mongod`](https:/ /docs.mongodb.com/v4.4/reference/program/mongod/#mongodb-binary-bin.mongod)实例的该限制。对于分片集群，必须为所有分片副本集成员修改该参数。超过此限制的事务将被视为已过期，并将被定期清理的进程中止掉。

对于分片集群，也可以在`commitTransaction`上指定一个`maxTimeMS`限制。更多信息参见[Sharded Clusters Transactions Time Limit](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-time-limit)。




## Oplog大小限制


- 从4.2版本开始，

  MongoDB会根据需要创建尽可能多的oplog条目来封装事务中的所有写操作，而不是为事务中的所有写操作创建一个条目。这移除了单oplog条目对其所有写操作施加的事务总大小为16MB的限制。尽管删除了总大小限制，但每个oplog条目仍然必须满足BSON文档16MB大小的限制。

- 在4.0版本，

  如果事务包含任何写操作，MongoDB会在提交时创建一个[oplog（操作日志）](https://docs.mongodb.com/v4.4/core/replica-set-oplog/)条目。也就是说，事务中的各个操作没有对应的oplog条目。相反，由单个oplog条目包含事务中的所有写操作。事务的oplog条目必须满足BSON文档16MB大小的限制。

  

## WiredTiger缓存


为了防止存储缓存压力对性能产生负面影响：

- 当你放弃一个事务时，中止掉事务。
- 当你在事务中的单个操作过程中遇到错误时，中止并重试该事务。

参数[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)也可以确保过期的事务被定期中止掉，以减轻存储缓存的压力。



## 事务和安全



- 如果使用了[访问控制](https://docs.mongodb.com/v4.4/core/authorization/)，你必须具有用于[事务中操作](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-operations)的[权限](https://docs.mongodb.com/v4.4/reference/built-in-roles/)。
- 如果使用了[auditing](https://docs.mongodb.com/v4.4/core/auditing/)，被中止事务中的操作仍然会被审计到。但是，没有审计事件来表明事务已经中止了。



## 分片配置限制


如果一个集群的某个分片上的参数[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.4/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)被设置为`false`，那么不能在该分片集群上运行事务（例如具有投票成员的分片使用了[内存存储引擎](https://docs.mongodb.com/v4.4/core/inmemory/)）。



## 分片集群和仲裁者


如果任何事务操作从一个包含仲裁节点的分片中读取或写入，其写操作跨越多个分片的事务将出错并中止。

另请参阅[三成员主-从-仲裁架构](https://docs.mongodb.com/v4.4/core/transactions-production-thinking/#std-label-transactions-psa)了解在禁用了majority读关注分片上的事务限制 。


## 三成员的主-从-仲裁架构


对于具有主-从-仲裁 (PSA) 架构的三成员副本集或具有三成员PSA分片的分片集群，你可能已经[禁用读关注`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority)来避免缓存压力。

- 在分片集群上，

  如果事务涉及具有已[禁用读关注`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority)的分片，则不能在事务中使用读关注[`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。只能在事务中使用读关注[`"local"`](https://docs.mongodb.com/v4.4/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)或[`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)。如果使用读关注[`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)，事务会报错并中止。`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false`。如果任何事务的读或写操作涉及已禁用读关注`"majority"`的分片，其写操作跨越多个分片的事务将出错并中止。

- 在副本集上，

  即使已经[禁用读关注`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority)，也可以在副本集上定义读关注[`"local"`](https://docs.mongodb.com/v4.4/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)、[`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)和[`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。但是，如果计划过渡到禁用读关注`"majority"`分片的分片集群上，那么可能会希望避免使用读关注`"snapshot"`。


> 提示
> 
> 要检查读关注"majority"是否被禁用，可以在[`mongod`](https://docs.mongodb.com/v4.4/reference/program/mongod/#mongodb-binary-bin.mongod)实例上运行[`db.serverStatus()`](https://docs.mongodb.com/v4.4/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)并检查[`storageEngine. supportCommittedReads`](https://docs.mongodb.com/v4.4/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads)字段。如果为`false`，则表示已禁用读关注"majority"。


> 提示
> 
> 同样请参阅：
> 
> - [`--enableMajorityReadConcern false`](https://docs.mongodb.com/v4.4/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)
> - [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/v4.4/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)



## 获取锁


默认情况下，事务最多等待5毫秒来获取事务中操作所需的锁。如果事务无法在5毫秒内获得所需的锁，事务将中止。

事务在中止或提交时释放所有锁。

> 提示
> 
> 在开始事务之前立即创建或删除集合时，如果需要在事务内访问该集合，则在进行创建或删除操作时使用写关注[`"majority"`](https://docs.mongodb.com/v4.4/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)可以保证事务能获取到请求的锁。



### 锁请求超时


可以使用[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)参数来调整事务等待获取锁的时间。增加[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)允许事务中的操作等待指定的时间来获取所需的锁。这有助于避免在瞬时并发锁请求时事务发生中止，例如快速运行的元数据操作。但是，这可能会延迟死锁事务操作的中止。

还可以通过将[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)设置为`-1`来使用特定于操作的超时。



## 待处理的DDL操作和事务


如果一个多文档事务正在执行，则影响相同数据库或集合的新DDL操作会等待该事务完成。当这些挂起的DDL操作存在时，访问与挂起的DDL操作相同的数据库或集合的新事务无法获得所需的锁，并将在等待 [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)后超时中止。此外，访问相同数据库或集合的新的非事务操作将被阻塞，直到它们达到`maxTimeMS`限制。

考虑以下场景：

- 请求集合锁的DDL操作

  当一个正在进行的事务对`hr`数据库中`employees`集合执行各种CRUD操作时，管理员在`employees`集合上发起[`db.collection.createIndex()`](https://docs.mongodb.com/ v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)DDL操作。[`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)命令会请求该集合上的排他集合锁。直到正在进行的事务完成，[`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)操作必须等待获取锁。任何影响`employees`集合且当[`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)命令正在挂起时启动的新事务，都必须等到[`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)完成才能执行。挂起的[`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)DDL操作不会影响`hr`数据库中其他集合上的事务。例如，`hr`数据库中`contractors`集合上的新事务可以正常启动和完成。

- 请求数据库锁的DDL操作

  当一个正在进行的事务对`hr`数据库中`employees`集合执行各种CRUD操作时，管理员在相同数据库中的`contractors`集合发起[`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)DDL操作。[`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)命令在父`hr`数据库上请求数据库锁。在进行中的事务完成之前，[`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)操作必须等待获取锁。任何影响`hr`数据库或该库下的其它集合并且在[`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb- dbcommand-dbcmd.collMod)命令挂起时启动的新事务，都必须等到[`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)完成之后才能执行。

在任何一种情况下，如果DDL操作保持挂起的时间超过[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)，挂起的事务等待后面那个操作中止。即[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)的值必须至少涵盖正在进行的事务和挂起的DDL操作完成所需的时间。

> 提示
>
> 同样请参阅：
>
> - [In-progress Transactions and Write Conflicts](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-transactions-write-conflicts)
> - [In-progress Transactions and Stale Reads](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-transactions-stale-reads)
> - [Which administrative commands lock a database?](https://docs.mongodb.com/v4.4/faq/concurrency/#std-label-faq-concurrency-database-lock)
> - [Which administrative commands lock a collection?](https://docs.mongodb.com/v4.4/faq/concurrency/#std-label-faq-concurrency-collection-lock)




## 正在进行的事务和写入冲突



如果事务正在进行中，但事务外部的写入修改了该事务之后尝试修改的文档，则事务会因写入冲突而中止。

如果一个事务正在进行并且已经锁定修改文档，那么当事务外部的写操作试图修改同一个文档时，写操作会一直等到事务结束。

> 提示
> 
> 同样请参阅：
>
> - [获取锁](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txns-locks)
> - [待执行的DDL操作和事务](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
> - [`$currentOp output`](https://docs.mongodb.com/v4.4/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.prepareReadConflicts)


## 正在进行的事务和过时的读取


事务内的读取操作可能会返回陈旧数据。也就是说，事务内部的读操作不能保证看到其他已提交的事务或非事务性写入的内容。例如，假设有以下操作序列：1) 一个事务正在进行中 2) 事务外部的写操作删除了一个文档 3) 事务内部的读取操作能够读取已被删除的文档，因为该操作使用的是写操作发生之前的快照。

为避免事务内部单个文档的读取过时，可以使用[`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.findOneAndUpdate/#mongodb-method-db.collection.findOneAndUpdate)方法。例如：

```
session.startTransaction( { readConcern: { level: "snapshot" }, writeConcern: { w: "majority" } } );

employeesCollection = session.getDatabase("hr").employees;

employeeDoc = employeesCollection.findOneAndUpdate(
   { _id: 1, employee: 1, status: "Active" },
   { $set: { employee: 1 } },
   { returnNewDocument: true }
);
```

- 如果上面的employee文档在事务之外发生更改，则事务会中止。
- 如果上面的employee文档未更改，事务将返回文档并锁定该文档。




## 正在进行的事务和块迁移



[块迁移](https://docs.mongodb.com/v4.4/core/sharding-balancer-administration/#std-label-chunk-migration-procedure)在某些阶段会获取排他的集合锁。

如果正在进行的事务持有集合上的锁，并且涉及该集合的块迁移刚开始，则这些迁移阶段必须等待事务释放集合上的锁，从而会影响块迁移的性能。

如果块迁移与事务交错进行（例如，如果事务在块迁移正在进行时开始，并且迁移在事务锁定集合之前完成），则事务在提交期间出错并中止。

根据两个操作的交错方式，一些示例错误包括（错误信息被缩略展示）：

- `an error from cluster data placement change ... migration commit in progress for <namespace>`
- `Cannot find shardId the chunk belonged to at cluster time ...`

> 提示
> 
> 同样请参阅：
> 
> [`shardingStatistics.countDonorMoveChunkLockTimeout`](https://docs.mongodb.com/v4.4/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.shardingStatistics.countDonorMoveChunkLockTimeout)



## 提交期间的外部读取


在事务提交期间，外部的读操作可能会尝试读取将被事务修改的相同文档。如果事务写入多个分片，则在跨分片提交尝试期间

- 使用读关注为[`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)或[`"linearizable"`](https://docs.mongodb.com/v4.4/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)的外部读操作，或者是因果一致性会话的一部分（即包括 [afterClusterTime](https://docs.mongodb.com/v4.4/reference/read-concern/#std-label-afterClusterTime)）会等待事务的所有写入可见。
- 使用其他读关注的外部读操作不会等待事务的所有写入可见，而是读取事务之前版本的可用文档。



## 错误


### 使用MongoDB 4.0驱动程序



要在MongoDB 4.2（副本集和分片集群）上使用事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。

在具有多个[`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos)实例的分片集群上，使用为MongoDB 4.0更新的驱动程序执行事务（而不是 MongoDB 4.2）将失败并可能导致错误，包括：

> 注意
> 
> 你的驱动程序可能会返回不同的错误。有关详细信息，请参阅驱动程序的文档。

| 错误码      | 错误信息                                                 |
| :--------- | :------------------------------------------------------ |
| 251        | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940      | `cannot commit with no participants`                    |




## 附加信息


> 提示
>
> 同样请参阅：
>
> [生产注意事项（分片集群）](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/)



原文链接：https://docs.mongodb.com/manual/core/transactions-production-consideration/

译者：李正洋
