# 产品注意事项（分片集群）

[Transactions](https://docs.mongodb.com/v4.2/core/transactions/) > Production Considerations (Sharded Clusters)

On this page

- [Sharded Transactions and MongoDB Drivers](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#sharded-transactions-and-mongodb-drivers)
- [Performance](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#performance)
- [Read Concerns](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#read-concerns)
- [Write Concerns](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#write-concerns)
- [Arbiters](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#arbiters)
- [Three Member Primary-Secondary-Arbiter Shards](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#three-member-primary-secondary-arbiter-shards)
- [Backups and Restores](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#backups-and-restores)
- [Chunk Migrations](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#chunk-migrations)
- [Outside Reads During Commit](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#outside-reads-during-commit)
- [Interaction with Replicated Index Builds](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#interaction-with-replicated-index-builds)
- [Additional Information](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#additional-information)

> 本页面中
>
> - [分片事务及MongoDB驱动](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#sharded-transactions-and-mongodb-drivers)
> - [性能](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#performance)
> - [Read Concerns读关注](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#read-concerns)
> - [Write Concerns写关注](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#write-concerns)
> - [仲裁者](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#arbiters)
> - [三成员 主-从-仲裁 分片](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#three-member-primary-secondary-arbiter-shards)
> - [备份回档](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#backups-and-restores)
> - [块迁移](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#chunk-migrations)
> - [提交期间的读请求](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#outside-reads-during-commit)
> - [与副本索引的交互](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#interaction-with-replicated-index-builds)
> - [其他信息](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#additional-information)

Starting in version 4.2, MongoDB provides the ability to perform multi-document transactions for sharded clusters.

The following page lists concerns specific to running transactions on a sharded cluster. These concerns are in addition to those listed in [Production Considerations](https://docs.mongodb.com/v4.2/core/transactions-production-consideration/).

> 从版本4.2开始，MongoDB提供了对分片群集执行多文档事务的功能。
>
> 下一页列出了特定于分片群集上运行事务的问题。除了[产品注意事项](https://docs.mongodb.com/v4.2/core/transactions-production-consideration/)中列出的那些关注点之外，还存在这些关注点。



## Sharded Transactions and MongoDB Drivers 分片事务及MongoDB驱动

*For transactions on MongoDB 4.2 deployments (replica sets and sharded clusters)*, clients **must** use MongoDB drivers updated for MongoDB 4.2.

On sharded clusters with multiple [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instances, performing transactions with drivers updated for MongoDB 4.0 (instead of MongoDB 4.2) will fail and can result in errors, including:

NOTE

Your driver may return a different error. Refer to your driver’s documentation for details.

> *对于MongoDB 4.2部署（副本集和分片群集）上的事务，*客户必须使用为MongoDB 4.2更新的MongoDB驱动程序。
>
> 在具有多个[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例的分片群集上，用MongoDB 4.0（而非MongoDB 4.2）执行事务更将会失败并可能导致错误，包括：
>
> 注意
>
> 您的驱动程序可能会返回其他错误。 有关详细信息，请参阅驱动程序的文档。

| Error Code / 错误码 | Error Message / 错误信息                                |
| ------------------- | ------------------------------------------------------- |
| 251                 | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940               | `cannot commit with no participants`                    |



## Performance 性能

### Single Shard

Transactions that target a single shard should have the same performance as replica-set transactions.

> ###单分片
>
> 针对单个分片的事务应具有与副本集事务相同的性能。



### Multiple Shards

Transactions that affect multiple shards incur a greater performance cost.

NOTE

On a sharded cluster, transactions that span multiple shards will error and abort if any involved shard contains an arbiter.

> ###多分片
>
> 影响多个分片的事务会产生更高的性能成本。
>
> 注意
>
> 在分片群集上，如果任何涉及的分片包含仲裁者，那么跨越多个分片的事务将出错并中止。



### Time Limit

To specify a time limit, specify a `maxTimeMS` limit on `commitTransaction`.

If `maxTimeMS` is unspecified, MongoDB will use the [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds).

If `maxTimeMS` is specified but would result in transaction that exceeds [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds), MongoDB will use the [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds).

To modify [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds) for a sharded cluster, the parameter must be modified for all shard replica set members.

> ###时间限制
>
> 要指定时间限制，请在`commitTransaction`上指定`maxTimeMS`限制。
>
> 如果未指定`maxTimeMS`，则MongoDB将使用[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds)。
>
> 如果指定`maxTimeMS`，但会导致事务超过[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds)，则MongoDB将使用[`transactionLifetimeLimitSeconds `](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds)。
>
> 要为分片群集修改[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/v4.2/reference/parameters/#param.transactionLifetimeLimitSeconds)，必须为所有分片副本集成员修改参数。



## Read Concerns 读关注

Multi-document transactions support [`"local"`](https://docs.mongodb.com/v4.2/reference/read-concern-local/#readconcern.%22local%22), [`"majority"`](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#readconcern.%22majority%22), and [`"snapshot"`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22) read concern levels.

For transactions on a sharded cluster, only the [`"snapshot"`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22) read concern provides a consistent snapshot across multiple shards.

For more information on read concern and transactions, see [Transactions and Read Concern](https://docs.mongodb.com/v4.2/core/transactions/#transactions-read-concern).



> 多文档事务支持[`local`](https://docs.mongodb.com/v4.2/reference/read-concern-local/#readconcern.%22local%22)，[`majority` ](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#readconcern.%22majority%22)和[`snapshot`](https：//docs.mongodb。 com / v4.2 / reference / read-concern-snapshot /＃readconcern。％22snapshot％22)读关注级别。
>
> 对于分片群集上的事务，只有[`snapshot`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22)提供了读关注跨多个分片的一致性快照。
>
> 有关读关注和事务的更多信息，请参阅[事务和读关注](https://docs.mongodb.com/v4.2/core/transactions/#transactions-read-concern)。



## Write Concerns 写关注

You cannot run transactions on a sharded cluster that has a shard with [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault) set to `false` (such as a shard with a voting member that uses the [in-memory storage engine](https://docs.mongodb.com/v4.2/core/inmemory/)).

NOTE

Regardless of the [write concern specified for the transaction](https://docs.mongodb.com/v4.2/core/transactions/#transactions-write-concern), the commit operation for a sharded cluster transaction includes some parts that use `{w: "majority", j: true}` write concern.



> 您无法在[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`时使用事务（ 例如使用[内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)的带有投票成员的分片）。
>
> 注意
>
> 不管[为事务指定的写关注](https://docs.mongodb.com/v4.2/core/transactions/#transactions-write-concern)，分片群集事务的提交操作都包含一些部分，这些部分包括： 用`{w：“ majority”，j：true}`写关注。



## Arbiters 仲裁者

Transactions whose write operations span multiple shards will error and abort if any transaction operation reads from or writes to a shard that contains an arbiter.

See also [Three Member Primary-Secondary-Arbiter Shards](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#transactions-sharded-clusters-psa) for transaction restrictions on shards that have disabled read concern majority.



> 如果任何事务操作读取或写入包含仲裁程序的分片，则其写操作跨越多个分片的事务将出错并中止。
>
> 另请参阅[三成员 主-从-仲裁 分片](https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/#transactions-sharded-clusters-psa)，了解对分片禁用阅读大多数策略的事务限制。



## Three Member Primary-Secondary-Arbiter Shards 三成员 主-从-仲裁 分片

For a sharded cluster with three-member PSA shards, you may have [disabled read concern “majority”](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#disable-read-concern-majority) (i.e. [`--enableMajorityReadConcern false`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-enablemajorityreadconcern) or [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/v4.2/reference/configuration-options/#replication.enableMajorityReadConcern)) to avoid cache pressure.

- On sharded clusters,

  If a transaction involves a shard that has [disabled read concern “majority”](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#disable-read-concern-majority), you cannot use read concern [`"snapshot"`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22) for the transaction. You can only use read concern [`"local"`](https://docs.mongodb.com/v4.2/reference/read-concern-local/#readconcern.%22local%22) or [`"majority"`](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#readconcern.%22majority%22) for the transaction. If you use read concern [`"snapshot"`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22), the transaction errors and aborts.`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false. `Transactions whose write operations span multiple shards will error and abort if any of the transaction’s read or write operations involves a shard that has disabled read concern `"majority"`.

- To check if read concern “majority” is disabled,

  You can run [`db.serverStatus()`](https://docs.mongodb.com/v4.2/reference/method/db.serverStatus/#db.serverStatus) and check the [`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/v4.2/reference/command/serverStatus/#serverstatus.storageEngine.supportsCommittedReads) field. If `false`, read concern “majority” is disabled.



> 对于具有三成员PSA的分片群集，您可以[`禁用读策略“majority”`](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#disable-read-关注多数)（如[`--enableMajorityReadConcern false`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-enablemajorityreadconcern)或[`replication.enableMajorityReadConcern：false `](https://docs.mongodb.com/v4.2/reference/configuration-options/#replication.enableMajorityReadConcern)）以避免缓存压力。
>
> - 在分片集群上，
>
>   如果事务涉及的分片[禁用读策略“majority”](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#disable-read-concern-majority)，则您不能设置读策略[`“snapshot”`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22)用于事务。对于该事务您只能使用读策略的[`“ local”`](https://docs.mongodb.com/v4.2/reference/read-concern-local/#readconcern.%22local%22)或[`“majority”`](https://docs.mongodb.com/v4.2/reference/read-concern-majority/#readconcern.%22majority%22)。如果使用读策略[`“ snapshot”`](https://docs.mongodb.com/v4.2/reference/read-concern-snapshot/#readconcern.%22snapshot%22)，则事务错误并中止。当enableMajorityReadConcern = false时，分片群集不支持`readConcern level'snapshot'。如果事务的任何读或写操作涉及到禁用读策略“majority”的分片，则其写操作跨越多个分片的事务将出错并中止。
>
> - 要检查读策略“majority”是否被禁用，
>
> 您可以运行[`db.serverStatus（）`](https://docs.mongodb.com/v4.2/reference/method/db.serverStatus/#db.serverStatus)并检查[`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/v4.2/reference/command/serverStatus/#serverstatus.storageEngine.supportsCommittedReads)字段。如果为“ false”，则禁用阅读关注“多数”。



## Backups and Restores 备份和回档

WARNING

[`mongodump`](https://docs.mongodb.com/v4.2/reference/program/mongodump/#bin.mongodump) and [`mongorestore`](https://docs.mongodb.com/v4.2/reference/program/mongorestore/#bin.mongorestore) **cannot** be part of a backup strategy for 4.2+ sharded clusters that have sharded transactions in progress, as backups created with [`mongodump`](https://docs.mongodb.com/v4.2/reference/program/mongodump/#bin.mongodump) *do not maintain* the atomicity guarantees of transactions across shards.

For 4.2+ sharded clusters with in-progress sharded transactions, use one of the following coordinated backup and restore processes which *do maintain* the atomicity guarantees of transactions across shards:

- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server),
- [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager?tck=docs_server), or
- [MongoDB Ops Manager](https://www.mongodb.com/products/ops-manager?tck=docs_server).



> 警告
>
> [`mongodump`](https://docs.mongodb.com/v4.2/reference/program/mongodump/#bin.mongodump)和[`mongorestore`](https://docs.mongodb.com/v4.2/reference/program/mongorestore/#bin.mongorestore)**不能**作为4.2+ 正在进行分片事务的分片群集的备份策略的一部分，因为使用[`mongodump`](https：// docs.mongodb.com/v4.2/reference/program/mongodump/#bin.mongodump)不维护跨分片事务的原子性保证。
>
> 对于具有正在进行中的分片事务的4.2+分片群集，请使用以下协调的备份和回档方案，这些方案确实维护了各个分片事务的原子性保证：
>
> -[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)，
>-[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager?tck=docs_server)，或
> -[MongoDB Ops Manager](https://www.mongodb.com/products/ops-manager?tck=docs_server)。



## Chunk Migrations 块迁移

[Chunk migration](https://docs.mongodb.com/v4.2/core/sharding-balancer-administration/#chunk-migration-procedure) acquires exclusive collection locks during certain stages.

If an ongoing transaction has a lock on a collection and a chunk migration that involves that collection starts, these migration stages must wait for the transaction to release the locks on the collection, thereby impacting the performance of chunk migrations.

If a chunk migration interleaves with a transaction (for instance, if a transaction starts while a chunk migration is already in progress and the migration completes before the transaction takes a lock on the collection), the transaction errors during the commit and aborts.

Depending on how the two operations interleave, some sample errors include (the error messages have been abbreviated):

- `an error from cluster data placement change ... migration commit in progress for <namespace>`
- `Cannot find shardId the chunk belonged to at cluster time ...`

SEE ALSO

[`shardingStatistics.countDonorMoveChunkLockTimeout`](https://docs.mongodb.com/v4.2/reference/command/serverStatus/#serverstatus.shardingStatistics.countDonorMoveChunkLockTimeout)



> [块迁移](https://docs.mongodb.com/v4.2/core/sharding-balancer-administration/#chunk-migration-procedure)在某些阶段获得会排他的集合锁。
>
> 如果正在进行的事务锁定了集合，并且涉及该集合的块迁移开始，则这些迁移阶段必须等待事务释放对集合的锁定，从而影响块迁移的性能。
>
> 如果大块迁移与事务交织（例如，如果在大块迁移已在进行的同时启动事务，并且迁移在事务锁定集合之前完成），则提交期间的事务错误并中止。
>
> 根据两个操作的交错方式，包括一些示例错误（错误消息已被缩写）：
>
> - `an error from cluster data placement change ... migration commit in progress for <namespace>`
>- `Cannot find shardId the chunk belonged to at cluster time ...`
> 
> 也可以看看
>
> [`shardingStatistics.countDonorMoveChunkLockTimeout`](https://docs.mongodb.com/v4.2/reference/command/serverStatus/#serverstatus.shardingStatistics.countDonorMoveChunkLockTimeout)



## Outside Reads During Commit 在提交期间进行外部读取

During the commit for a transaction, outside read operations may try to read the same documents that will be modified by the transaction. If the transaction writes to multiple shards, then during the commit attempt across the shards

- Outside reads that use read concern `snapshot` or [`"linearizable"`](https://docs.mongodb.com/v4.2/reference/read-concern-linearizable/#readconcern.%22linearizable%22), or are part of causally consistent sessions (i.e. include [afterClusterTime](https://docs.mongodb.com/v4.2/reference/read-concern/#afterclustertime)) wait for all writes of a transaction to be visible.
- Outside reads using other read concerns do not wait for all writes of a transaction to be visible but instead read the before-transaction version of the documents available.

SEE ALSO

[Transactions and Atomicity](https://docs.mongodb.com/v4.2/core/transactions/#transactions-atomicity)



> 在提交事务期间，外部读取操作可能会尝试读取将由事务修改的相同文档。 如果事务写入多个分片，则在尝试对各个分片进行提交时
>
> - 外部读取时使用读策略`snapshot`或[`“ linearizable”`](https://docs.mongodb.com/v4.2/reference/read-concern-linearizable/#readconcern.%22linearizable%22)， 或者是一致性的会话的一部分（例如，包括[afterClusterTime](https://docs.mongodb.com/v4.2/reference/read-concern/#afterclustertime)）等待事务的所有写入均可见。
>
> - 外部读取使用其他读策略时，不会等待事务的所有写入都可见，而是会读取可用集合的事务前版本。
>
> 也可以看看
>
> [事务及原子性](https://docs.mongodb.com/v4.2/core/transactions/#transactions-atomicity)



## Interaction with Replicated Index Builds 与副本索引的交互

For [replicated index builds](https://docs.mongodb.com/v4.2/core/index-creation/#index-operations-replicated-build) on a collection (as opposed to a [rolling index build](https://docs.mongodb.com/v4.2/tutorial/build-indexes-on-replica-sets/#index-build-on-replica-sets)), once an index build issued against the primary replica set member completes, the secondary members apply the associated oplog entry and start the index build. While building the index, the secondary waits to apply any later oplog entries, including a distributed transaction that commits during the build. If replication stalls for longer than the [oplog window](https://docs.mongodb.com/v4.2/core/replica-set-oplog/#replica-set-oplog-sizing), the secondary falls out of sync and requires [resynchronization](https://docs.mongodb.com/v4.2/core/replica-set-sync/#replica-set-initial-sync) to recover.

To minimize potential interactions between sharded transactions and indexes, consider one of the following strategies for building indexes on sharded clusters:

- Build indexes during a maintenance window in which applications cease issuing distributed transactions against the collections being indexed.
- Build indexes using a rolling index build procedure as described in [Build Indexes on Sharded Clusters](https://docs.mongodb.com/v4.2/tutorial/build-indexes-on-sharded-clusters/).
- Increase the [oplog size](https://docs.mongodb.com/v4.2/tutorial/change-oplog-size/) on each replica set member to mitigate the likelihood of falling out of sync due to replicated index builds.



> 对于集合上的[复制索引版本](https://docs.mongodb.com/v4.2/core/index-creation/#index-operations-replicated-build)（而不是[滚动索引版本]( https://docs.mongodb.com/v4.2/tutorial/build-indexes-on-replica-sets/#index-build-on-replica-sets)），一旦针对主副本集成员发布了索引构建完成后，从成员将应用关联的oplog条目并开始索引构建。在构建索引时，从数据库等待应用以后的任何oplog条目，包括在构建期间提交的分布式事务。如果复制停顿的时间长于[oplog窗口](https://docs.mongodb.com/v4.2/core/replica-set-oplog/#replica-set-oplog-sizing)，则c从副本将失去同步并需要[重新同步](https://docs.mongodb.com/v4.2/core/replica-set-sync/#replica-set-initial-sync)才能恢复。
>
> 为了最大程度地减少分片事务和索引之间的潜在交互，请考虑以下在分片集群上构建索引的策略之一：
>
> - 在维护窗口期间建立索引，在该窗口中，应用程序停止针对要建立索引的集合发出分布式事务。
>- 使用滚动索引构建过程构建索引，如[在分片群集上构建索引](https://docs.mongodb.com/v4.2/tutorial/build-indexes-on-sharded-clusters/)中所述。
> - 增加每个副本集成员上的[oplog大小](https://docs.mongodb.com/v4.2/tutorial/change-oplog-size/)，以减轻由于复制索引构建而导致不同步的可能性。



## Additional Information 其他信息

See also [Production Considerations](https://docs.mongodb.com/v4.2/core/transactions-production-consideration/).

> 参考 [产品说明](https://docs.mongodb.com/v4.2/core/transactions-production-consideration/).



原文链接：https://docs.mongodb.com/v4.2/core/transactions-sharded-clusters/
 
 
译者：王金铷
