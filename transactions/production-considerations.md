# Production Considerations

## 生产注意事项

The following page lists some production considerations for running transactions. These apply whether you run transactions on replica sets or sharded clusters. For running transactions on sharded clusters, see also the [Production Considerations (Sharded Clusters)](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/) for additional considerations that are specific to sharded clusters.

以下内容列出了运行事务的一些生产注意事项。无论是在副本集还是分片集群上运行事务，这些都适用。要在分片集群上运行事务，另请参阅[生产注意事项（分片集群）](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/)来了解专门针对分片集群的额外注意事项。

## Availability

## 可用性

- **In version 4.0**, MongoDB supports multi-document transactions on replica sets.

- **In version 4.2**, MongoDB introduces distributed transactions, which adds support for multi-document transactions on sharded clusters and incorporates the existing support for multi-document transactions on replica sets.

  To use transactions on MongoDB 4.2 deployments (replica sets and sharded clusters), clients **must** use MongoDB drivers updated for MongoDB 4.2.

NOTE

Distributed Transactions and Multi-Document Transactions

Starting in MongoDB 4.2, the two terms are synonymous. Distributed transactions refer to multi-document transactions on sharded clusters and replica sets. Multi-document transactions (whether on sharded clusters or replica sets) are also known as distributed transactions starting in MongoDB 4.2.

- **在4.0版本**，MongoDB支持副本集上的多文档事务。

- **在4.2版本**，MongoDB引入了分布式事务，增加了对分片集群上多文档事务的支持，并整合了已有的对副本集上多文档事务的支持。

  要在MongoDB 4.2（副本集和分片集群）中使用事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。

> 注意
> 
> **分布式事务和多文档事务**
> 
> 从MongoDB 4.2开始，这两个术语是同义词。分布式事务是指分片集群和副本集上的多文档事务。从MongoDB 4.2开始，多文档事务（无论是在分片集群上还是副本集上）也称为分布式事务。



## Feature Compatibility

## 功能兼容性

To use transactions, the [featureCompatibilityVersion](https://docs.mongodb.com/v4.4/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) for all members of the deployment must be at least:

| Deployment      | Minimum `featureCompatibilityVersion` |
| :-------------- | :------------------------------------ |
| Replica Set     | `4.0`                                 |
| Sharded Cluster | `4.2`                                 |

To check the fCV for a member, connect to the member and run the following command:

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

For more information, see the [`setFeatureCompatibilityVersion`](https://docs.mongodb.com/v4.4/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion) reference page.

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



## Runtime Limit

## 运行时限制

By default, a transaction must have a runtime of less than one minute. You can modify this limit using [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds) for the [`mongod`](https://docs.mongodb.com/v4.4/reference/program/mongod/#mongodb-binary-bin.mongod) instances. For sharded clusters, the parameter must be modified for all shard replica set members. Transactions that exceeds this limit are considered expired and will be aborted by a periodic cleanup process.

For sharded clusters, you can also specify a `maxTimeMS` limit on `commitTransaction`. For more information, see [Sharded Clusters Transactions Time Limit](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-time-limit).

默认情况下，事务的运行时间必须少于一分钟。可以使用[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)修改[`mongod`](https:/ /docs.mongodb.com/v4.4/reference/program/mongod/#mongodb-binary-bin.mongod)实例的该限制。对于分片集群，必须为所有分片副本集成员修改该参数。超过此限制的事务将被视为已过期，并将被定期清理的进程中止掉。

对于分片集群，也可以在`commitTransaction`上指定一个`maxTimeMS`限制。更多信息参见[Sharded Clusters Transactions Time Limit](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-time-limit)。



## Oplog Size Limit

## Oplog大小限制

- Starting in version 4.2,

  MongoDB creates as many oplog entries as necessary to the encapsulate all write operations in a transaction, instead of a single entry for all write operations in the transaction. This removes the 16MB total size limit for a transaction imposed by the single oplog entry for all its write operations. Although the total size limit is removed, each oplog entry still must be within the BSON document size limit of 16MB.

- In version 4.0,

  MongoDB creates a single [oplog (operations log)](https://docs.mongodb.com/v4.4/core/replica-set-oplog/) entry at the time of commit if the transaction contains any write operations. That is, the individual operations in the transactions do not have a corresponding oplog entry. Instead, a single oplog entry contains all of the write operations within a transaction. The oplog entry for the transaction must be within the BSON document size limit of 16MB.
  
- 从4.2版本开始，

  MongoDB会根据需要创建尽可能多的oplog条目来封装事务中的所有写操作，而不是为事务中的所有写操作创建一个条目。这移除了单oplog条目对其所有写操作施加的事务总大小为16MB的限制。尽管删除了总大小限制，但每个oplog条目仍然必须满足BSON文档16MB大小的限制。

- 在4.0版本，

  如果事务包含任何写操作，MongoDB会在提交时创建一个[oplog（操作日志）](https://docs.mongodb.com/v4.4/core/replica-set-oplog/)条目。也就是说，事务中的各个操作没有对应的oplog条目。相反，由单个oplog条目包含事务中的所有写操作。事务的oplog条目必须满足BSON文档16MB大小的限制。

  

## WiredTiger Cache

## WiredTiger缓存

To prevent storage cache pressure from negatively impacting the performance:

- When you abandon a transaction, abort the transaction.
- When you encounter an error during individual operation in the transaction, abort and retry the transaction.

The [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds) also ensures that expired transactions are aborted periodically to relieve storage cache pressure.

为了防止存储缓存压力对性能产生负面影响：

- 当你放弃一个事务时，中止掉事务。
- 当你在事务中的单个操作过程中遇到错误时，中止并重试该事务。

参数[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)也可以确保过期的事务被定期中止掉，以减轻存储缓存的压力。

## Transactions and Security

## 事务和安全

- If running with [access control](https://docs.mongodb.com/v4.4/core/authorization/), you must have [privileges](https://docs.mongodb.com/v4.4/reference/built-in-roles/) for the [operations in the transaction](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-operations).
- If running with [auditing](https://docs.mongodb.com/v4.4/core/auditing/), operations in an aborted transaction are still audited. However, there is no audit event that indicates that the transaction aborted.

- 如果使用了[访问控制](https://docs.mongodb.com/v4.4/core/authorization/)，你必须具有用于[事务中操作](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-operations)的[权限](https://docs.mongodb.com/v4.4/reference/built-in-roles/)。
- 如果使用了[auditing](https://docs.mongodb.com/v4.4/core/auditing/)，被中止事务中的操作仍然会被审计到。但是，没有审计事件来表明事务已经中止了。



## Shard Configuration Restriction

## 分片配置限制

You cannot run transactions on a sharded cluster that has a shard with [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.4/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) set to `false` (such as a shard with a voting member that uses the [in-memory storage engine](https://docs.mongodb.com/v4.4/core/inmemory/)).

如果一个集群的某个分片上的参数[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.4/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)被设置为`false`，那么不能在该分片集群上运行事务（例如具有投票成员的分片使用了[内存存储引擎](https://docs.mongodb.com/v4.4/core/inmemory/)）。

## Sharded Clusters and Arbiters

## 分片集群和仲裁者

Transactions whose write operations span multiple shards will error and abort if any transaction operation reads from or writes to a shard that contains an arbiter.

See also [3-Member Primary-Secondary-Arbiter Architecture](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-transactions-psa) for transaction restrictions on shards that have disabled read concern majority.

如果任何事务操作从一个包含仲裁节点的分片中读取或写入，其写操作跨越多个分片的事务将出错并中止。

另请参阅[三成员主-从-仲裁架构](https://docs.mongodb.com/v4.4/core/transactions-production-thinking/#std-label-transactions-psa)了解在禁用了majority读关注分片上的事务限制 。

## 3-Member Primary-Secondary-Arbiter Architecture

## 三成员的主-从-仲裁架构

For a three-member replica set with a primary-secondary-arbiter (PSA) architecture or a sharded cluster with a three-member PSA shards, you may have [disabled read concern "majority"](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority) to avoid cache pressure.

- On sharded clusters,

  If a transaction involves a shard that has [disabled read concern "majority"](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority), you cannot use read concern [`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) for the transaction. You can only use read concern [`"local"`](https://docs.mongodb.com/v4.4/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) or [`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) for the transaction. If you use read concern [`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-), the transaction errors and aborts.`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false.`Transactions whose write operations span multiple shards will error and abort if any of the transaction's read or write operations involves a shard that has disabled read concern `"majority"`.

- On replica set,

  You can specify read concern [`"local"`](https://docs.mongodb.com/v4.4/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) or [`"majority"`](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) or [`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) even in the replica set has [disabled read concern "majority"](https://docs.mongodb.com/v4.4/reference/read-concern-majority/#std-label-disable-read-concern-majority).However, if you are planning to transition to a sharded cluster with disabled read concern majority shards, you may wish to avoid using read concern `"snapshot"`.

TIP

To check if read concern "majority" is disabled, You can run [`db.serverStatus()`](https://docs.mongodb.com/v4.4/reference/method/db.serverStatus/#mongodb-method-db.serverStatus) on the [`mongod`](https://docs.mongodb.com/v4.4/reference/program/mongod/#mongodb-binary-bin.mongod) instances and check the [`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/v4.4/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads) field. If `false`, read concern "majority" is disabled.

TIP

See also:

- [`--enableMajorityReadConcern false`](https://docs.mongodb.com/v4.4/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)
- [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/v4.4/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern).

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

## Acquiring Locks

## 获取锁

By default, transactions wait up to `5` milliseconds to acquire locks required by the operations in the transaction. If the transaction cannot acquire its required locks within the `5` milliseconds, the transaction aborts.

Transactions release all locks upon abort or commit.

TIP

When creating or dropping a collection immediately before starting a transaction, if the collection is accessed within the transaction, issue the create or drop operation with write concern [`"majority"`](https://docs.mongodb.com/v4.4/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) to ensure that the transaction can acquire the required locks.

默认情况下，事务最多等待5毫秒来获取事务中操作所需的锁。如果事务无法在5毫秒内获得所需的锁，事务将中止。

事务在中止或提交时释放所有锁。

> 提示
> 
> 在开始事务之前立即创建或删除集合时，如果需要在事务内访问该集合，则在进行创建或删除操作时使用写关注[`"majority"`](https://docs.mongodb.com/v4.4/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)可以保证事务能获取到请求的锁。


### Lock Request Timeout

### 锁请求超时

You can use the [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis) parameter to adjust how long transactions wait to acquire locks. Increasing [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis) allows operations in the transactions to wait the specified time to acquire the required locks. This can help obviate transaction aborts on momentary concurrent lock acquisitions, like fast-running metadata operations. However, this could possibly delay the abort of deadlocked transaction operations.

You can also use operation-specific timeout by setting [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis) to `-1`.

可以使用[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)参数来调整事务等待获取锁的时间。增加[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)允许事务中的操作等待指定的时间来获取所需的锁。这有助于避免在瞬时并发锁请求时事务发生中止，例如快速运行的元数据操作。但是，这可能会延迟死锁事务操作的中止。

还可以通过将[`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)设置为`-1`来使用特定于操作的超时。

## Pending DDL Operations and Transactions

## 待处理的DDL操作和事务

If a multi-document transaction is in progress, new DDL operations that affect the same database(s) or collection(s) wait behind the transaction. While these pending DDL operations exist, new transactions that access the same database(s) or collection(s) as the pending DDL operations cannot obtain the required locks and and will abort after waiting [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis). In addition, new non-transaction operations that access the same database(s) or collection(s) will block until they reach their `maxTimeMS` limit.

Consider the following scenarios:

- DDL Operation That Requires a Collection Lock

  While an in-progress transaction is performing various CRUD operations on the `employees` collection in the `hr` database, an administrator issues the [`db.collection.createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) DDL operation against the `employees` collection. [`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) requires an exclusive collection lock on the collection.Until the in-progress transaction completes, the [`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) operation must wait to obtain the lock. Any new transaction that affects the `employees` collection and starts while the [`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) is pending must wait until after [`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) completes.The pending [`createIndex()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) DDL operation does not affect transactions on other collections in the `hr` database. For example, a new transaction on the `contractors` collection in the `hr` database can start and complete as normal.

- DDL Operation That Requires a Database Lock

  While an in-progress transaction is performing various CRUD operations on the `employees` collection in the `hr` database, an administrator issues the [`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) DDL operation against the `contractors` collection in the same database. [`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) requires a database lock on the parent `hr` database.Until the in-progress transaction completes, the [`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) operation must wait to obtain the lock. Any new transaction that affects the `hr` database or *any* of its collections and starts while the [`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) is pending must wait until after [`collMod`](https://docs.mongodb.com/v4.4/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) completes.

In either scenario, if the DDL operation remains pending for more than [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis), pending transactions waiting behind that operation abort. That is, the value of [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/v4.4/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis) must at least cover the time required for the in-progress transaction *and* the pending DDL operation to complete.

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



## In-progress Transactions and Write Conflicts

## 正在进行的事务和写入冲突

If a transaction is in progress and a write outside the transaction modifies a document that an operation in the transaction later tries to modify, the transaction aborts because of a write conflict.

If a transaction is in progress and has taken a lock to modify a document, when a write outside the transaction tries to modify the same document, the write waits until the transaction ends.

TIP

See also:

- [Acquiring Locks](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txns-locks)
- [Pending DDL Operations and Transactions](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
- [`$currentOp output`](https://docs.mongodb.com/v4.4/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.prepareReadConflicts)

如果事务正在进行中，但事务外部的写入修改了该事务之后尝试修改的文档，则事务会因写入冲突而中止。

如果一个事务正在进行并且已经锁定修改文档，那么当事务外部的写操作试图修改同一个文档时，写操作会一直等到事务结束。

> 提示
> 
> 同样请参阅：
>
> - [获取锁](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txns-locks)
> - [待执行的DDL操作和事务](https://docs.mongodb.com/v4.4/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
> - [`$currentOp output`](https://docs.mongodb.com/v4.4/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.prepareReadConflicts)

## In-progress Transactions and Stale Reads

## 正在进行的事务和过时的读取

Read operations inside a transaction can return stale data. That is, read operations inside a transaction are not guaranteed to see writes performed by other committed transactions or non-transactional writes. For example, consider the following sequence: 1) a transaction is in-progress 2) a write outside the transaction deletes a document 3) a read operation inside the transaction is able to read the now-deleted document since the operation is using a snapshot from before the write.

To avoid stale reads inside transactions for a single document, you can use the [`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/v4.4/reference/method/db.collection.findOneAndUpdate/#mongodb-method-db.collection.findOneAndUpdate) method. For example:

```
session.startTransaction( { readConcern: { level: "snapshot" }, writeConcern: { w: "majority" } } );

employeesCollection = session.getDatabase("hr").employees;

employeeDoc = employeesCollection.findOneAndUpdate(
   { _id: 1, employee: 1, status: "Active" },
   { $set: { employee: 1 } },
   { returnNewDocument: true }
);
```

- If the employee document has changed outside the transaction, then the transaction aborts.
- If the employee document has not changed, the transaction returns the document and locks the document.

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



## In-progress Transactions and Chunk Migration

## 正在进行的事务和块迁移

[Chunk migration](https://docs.mongodb.com/v4.4/core/sharding-balancer-administration/#std-label-chunk-migration-procedure) acquires exclusive collection locks during certain stages.

If an ongoing transaction has a lock on a collection and a chunk migration that involves that collection starts, these migration stages must wait for the transaction to release the locks on the collection, thereby impacting the performance of chunk migrations.

If a chunk migration interleaves with a transaction (for instance, if a transaction starts while a chunk migration is already in progress and the migration completes before the transaction takes a lock on the collection), the transaction errors during the commit and aborts.

Depending on how the two operations interleave, some sample errors include (the error messages have been abbreviated):

- `an error from cluster data placement change ... migration commit in progress for <namespace>`
- `Cannot find shardId the chunk belonged to at cluster time ...`

TIP

See also:

[`shardingStatistics.countDonorMoveChunkLockTimeout`](https://docs.mongodb.com/v4.4/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.shardingStatistics.countDonorMoveChunkLockTimeout)

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


## Outside Reads During Commit

## 提交期间的外部读取

During the commit for a transaction, outside read operations may try to read the same documents that will be modified by the transaction. If the transaction writes to multiple shards, then during the commit attempt across the shards

- Outside reads that use read concern [`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) or [`"linearizable"`](https://docs.mongodb.com/v4.4/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-), or are part of causally consistent sessions (i.e. include [afterClusterTime](https://docs.mongodb.com/v4.4/reference/read-concern/#std-label-afterClusterTime)) wait for all writes of a transaction to be visible.
- Outside reads using other read concerns do not wait for all writes of a transaction to be visible but instead read the before-transaction version of the documents available.

在事务提交期间，外部的读操作可能会尝试读取将被事务修改的相同文档。如果事务写入多个分片，则在跨分片提交尝试期间

- 使用读关注为[`"snapshot"`](https://docs.mongodb.com/v4.4/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)或[`"linearizable"`](https://docs.mongodb.com/v4.4/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)的外部读操作，或者是因果一致性会话的一部分（即包括 [afterClusterTime](https://docs.mongodb.com/v4.4/reference/read-concern/#std-label-afterClusterTime)）会等待事务的所有写入可见。
- 使用其他读关注的外部读操作不会等待事务的所有写入可见，而是读取事务之前版本的可用文档。


## Errors

## 错误

### Use of MongoDB 4.0 Drivers

### 使用MongoDB 4.0驱动程序

To use transactions on MongoDB 4.2 deployments (replica sets and sharded clusters), clients **must** use MongoDB drivers updated for MongoDB 4.2.

On sharded clusters with multiple [`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos) instances, performing transactions with drivers updated for MongoDB 4.0 (instead of MongoDB 4.2) will fail and can result in errors, including:

NOTE

Your driver may return a different error. Refer to your driver's documentation for details.

| Error Code | Error Message                                           |
| :--------- | :------------------------------------------------------ |
| 251        | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940      | `cannot commit with no participants`                    |

要在MongoDB 4.2（副本集和分片集群）上使用事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。

在具有多个[`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos)实例的分片集群上，使用为MongoDB 4.0更新的驱动程序执行事务（而不是 MongoDB 4.2）将失败并可能导致错误，包括：

> 注意
> 
> 你的驱动程序可能会返回不同的错误。有关详细信息，请参阅驱动程序的文档。

| 错误码      | 错误信息                                                 |
| :--------- | :------------------------------------------------------ |
| 251        | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940      | `cannot commit with no participants`                    |



## Additional Information

## 附加信息

TIP

See also:

[Production Considerations (Sharded Clusters)](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/)

> 提示
>
> 同样请参阅：
>
> [生产注意事项（分片集群）](https://docs.mongodb.com/v4.4/core/transactions-sharded-clusters/)



原文链接：https://docs.mongodb.com/manual/core/transactions-production-consideration/

译者：李正洋
