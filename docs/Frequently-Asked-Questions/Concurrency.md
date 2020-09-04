# 常见问题解答：并发


在本页面

- [MongoDB使用哪种类型的锁定？](https://docs.mongodb.com/manual/faq/concurrency/#what-type-of-locking-does-mongodb-use)
- [MongoDB中中锁的粒度有多细？](https://docs.mongodb.com/manual/faq/concurrency/#how-granular-are-locks-in-mongodb)
- [如何查看](https://docs.mongodb.com/manual/faq/concurrency/#how-do-i-see-the-status-of-locks-on-my-mongod-instances)[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例[上锁的状态](https://docs.mongodb.com/manual/faq/concurrency/#how-do-i-see-the-status-of-locks-on-my-mongod-instances)？
- [读或写操作是否会产生锁定？](https://docs.mongodb.com/manual/faq/concurrency/#does-a-read-or-write-operation-ever-yield-the-lock)
- [一些常见的客户端操作会采取什么锁定？](https://docs.mongodb.com/manual/faq/concurrency/#what-locks-are-taken-by-some-common-client-operations)
- [哪些管理命令可以锁定数据库？](https://docs.mongodb.com/manual/faq/concurrency/#which-administrative-commands-lock-the-database)
- [哪些管理命令可以锁定集合？](https://docs.mongodb.com/manual/faq/concurrency/#which-administrative-commands-lock-a-collection)
- [MongoDB操作是否可以锁定多个数据库？](https://docs.mongodb.com/manual/faq/concurrency/#does-a-mongodb-operation-ever-lock-more-than-one-database)
- [分片如何影响并发？](https://docs.mongodb.com/manual/faq/concurrency/#how-does-sharding-affect-concurrency)
- [并发性如何影响副本集的主节点？](https://docs.mongodb.com/manual/faq/concurrency/#how-does-concurrency-affect-a-replica-set-primary)
- [并发如何影响副本集的从节点？](https://docs.mongodb.com/manual/faq/concurrency/#how-does-concurrency-affect-secondaries)
- [MongoDB是否支持事务？](https://docs.mongodb.com/manual/faq/concurrency/#does-mongodb-support-transactions)
- [MongoDB提供什么隔离保证？](https://docs.mongodb.com/manual/faq/concurrency/#what-isolation-guarantees-does-mongodb-provide)



MongoDB允许多个客户端读取和写入相同的数据。为了确保一致性，它使用锁定和其他 [并发控制](https://docs.mongodb.com/manual/reference/glossary/#term-concurrency-control)措施来防止多个客户端同时修改同一数据。这些机制共同保证了对单个文档的所有写入全部发生或完全不发生，并且客户端永远不会看到不一致的数据视图。



## MongoDB使用哪种类型的锁定？



MongoDB使用多粒度锁定[[1\]](https://docs.mongodb.com/manual/faq/concurrency/#mgl-ref)，它允许操作锁定在全局，数据库或集合级别，并允许各个存储引擎在集合级别以下实现自己的并发控制（例如，在WiredTiger中的文档级别）。 


MongoDB使用读-写锁，允许并发的读写操作以共享的方式访问资源（例如数据库或集合）。


除了用于读取的共享（S）锁定模式和用于写操作的排他（X）锁定模式之外，意向共享（IS）和意向排他（IX）模式还表明使用更精细的锁定粒度来读取或写入资源的意图 。当以一定的粒度锁定时，所有更高级别的锁定都使用[意向锁来锁定](https://docs.mongodb.com/manual/reference/glossary/#term-intent-lock)。


例如，当锁定用于写的集合（使用排它X锁定模式）时，必须同时在意向排他（IX）锁定模式下锁定相应的数据库锁和全局锁。单个数据库可以同时在IS和IX模式下锁定，但是独占（X）锁不能与任何其他模式共存，共享（S）锁只能与意向共享（IS）锁共存。


锁是公平的，读取和写入按顺序排队。但是，为了优化吞吐量，当一个请求被授予时，所有其他兼容请求将同时被授予，从而有可能在冲突请求之前释放它们。例如，考虑X锁（排它锁）被释放的情况，并且冲突队列包含以下各项：

IS → IS → X → X → S → IS


在严格的先进先出（FIFO）顺序中，将仅授予前两个IS模式。相反，MongoDB实际上将授予所有IS和S模式，一旦它们全部完成，它将授予X，即使在此期间新的IS或S请求已进入排队。由于授予将始终将所有其他请求移到队列中，因此任何请求都不会存在饿死等待问题。


在[`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus)和[`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp)输出中，锁定模式表示如下：



| 锁模式 | Description         |
| :----- | :------------------ |
| `R`    | 代表共享锁(S)       |
| `W`    | 代表排它锁(X)       |
| `r`    | 代表意向共享锁 (IS) |
| `w`    | 代表意向排它锁 (IX) |


| [[1\]](https://docs.mongodb.com/manual/faq/concurrency/#id1) | 有关更多信息，请参见Wikipedia页面上的 [多粒度锁定](http://en.wikipedia.org/wiki/Multiple_granularity_locking)。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |



## MongoDB中锁的粒度有多细？[¶](https://docs.mongodb.com/manual/faq/concurrency/#how-granular-are-locks-in-mongodb)


对于大多数读取和写入操作，WiredTiger使用乐观并发控制。WiredTiger仅在全局，数据库和集合级别使用意向锁。当存储引擎检测到两个操作之间存在冲突时，将引发写冲突，从而导致MongoDB对用户而言透明地重试该操作。


一些全局操作（通常是涉及多个数据库的短暂操作）仍然需要全局“实例范围”锁定。其他一些操作（如[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)）仍需要排他数据库锁。



## 如何查看[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例上的锁状态？[¶](https://docs.mongodb.com/manual/faq/concurrency/#how-do-i-see-the-status-of-locks-on-my-mongod-instances)


要报告有关锁的锁使用率信息，请使用以下任何一种方法：


- [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus)，
- [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp)，
- [mongotop](https://docs.mongodb.com/manual/reference/program/mongotop/)，
- [mongostat](https://docs.mongodb.com/manual/reference/program/mongostat/)，和/或
- 在[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?jmp=docs)或 [Ops Manager，MongoDB企业版提供的先进的解决方案](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)


具体而言，[serverStatus输出中](https://docs.mongodb.com/manual/reference/command/serverStatus/)的[`locks`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.locks)文档或[当前操作报告中](https://docs.mongodb.com/manual/reference/method/db.currentOp/)的字段可提供有关 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例中锁的类型和锁争用的数量。


在[`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus)和[`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp)输出中，锁定模式表示如下：


| 锁定模式 | 描述                   |
| :------- | :--------------------- |
| `R`      | 表示共享（S）锁。      |
| `W`      | 表示排他（X）锁。      |
| `r`      | 表示共享意图（IS）锁。 |
| `w`      | 表示意向排他（IX）锁。 |

要终止操作，请使用[`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/#db.killOp)。



## 读或写操作是否会产生锁定？[¶](https://docs.mongodb.com/manual/faq/concurrency/#does-a-read-or-write-operation-ever-yield-the-lock)


在某些情况下，读和写操作可以产生它持有的锁。


长时间运行的读写操作（例如查询，更新和删除）在许多情况下都会产生。MongoDB如果单个修改文档的操作，影响带有`multi`参数修改多个文档 [`update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update)操作，MongoDB也会产生锁定。


对于支持文档级[并发控制的](https://docs.mongodb.com/manual/reference/glossary/#term-concurrency-control)存储引擎（例如[WiredTiger）](https://docs.mongodb.com/manual/core/wiredtiger/)，当使用意向锁访问存储时不需要锁定，因为该锁是数据库和集合级别的全局锁定，不会阻塞其他读取和写入操作。但是，操作将定期产生锁定，例如：

- 避免长时间执行的存储性事务，因为这些事务可能需要在内存中保存大量数据；

- 作为中断响应点（interruption points），以便您可以取消长时间运行的操作；

- 允许需要排他访问集合的操作，例如索引/集合删除和创建。

  

## 一些常见的客户端操作会采取什么锁定？


下表列出了一些操作以及它们用于文档级锁定存储引擎的锁定类型：


| 操作             | 数据库                              | 集合级别锁                       |
| :--------------- | :---------------------------------- | :------------------------------- |
| 发出查询         | `r` （意向共享）                    | `r` （意向共享）                 |
| 插入资料         | `w` （意向排他）                    | `w` （意向排他）                 |
| 删除资料         | `w` （意向排他）                    | `w` （意向排他）                 |
| 更新数据         | `w` （意向排他）                    | `w` （意向排他）                 |
| 执行聚合操作     | `r` （意向共享）                    | `r` （意向共享）                 |
| 创建索引（前台） | `W` （排他）                        |                                  |
| 创建索引（后台） | `w` （意向排他）                    | `w` （意向排他）                 |
| 列出集合列表     | `r` （意向共享）*在版本4.0中更改。* |                                  |
| 映射减少操作     | `W`（排他）和`R`（共享）            | `w`（意向排他）和`r`（意向共享） |


## 哪些管理命令可以锁定数据库？


某些管理命令可以较长时间排他锁定数据库。在某些部署中，对于大型数据库，您可以考虑使[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例脱机，以便客户端不受影响。例如，如果 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)是[副本集的](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)一部分，请执行[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)脱机操作，让集合服务的其他成员请求负载。


以下管理操作需要在数据库级别上长时间进行排他锁定：


| 运作方式                                                     | 方法                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`cloneCollectionAsCapped`](https://docs.mongodb.com/manual/reference/command/cloneCollectionAsCapped/#dbcmd.cloneCollectionAsCapped) |                                                              |
| [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod) |                                                              |
| [`compact`](https://docs.mongodb.com/manual/reference/command/compact/#dbcmd.compact) |                                                              |
| [`convertToCapped`](https://docs.mongodb.com/manual/reference/command/convertToCapped/#dbcmd.convertToCapped) |                                                              |
| [`renameCollection`](https://docs.mongodb.com/manual/reference/command/renameCollection/#dbcmd.renameCollection)[`db.collection.renameCollection()`](https://docs.mongodb.com/manual/reference/method/db.collection.renameCollection/#db.collection.renameCollection) | *在版本4.2中进行了更改。*如果在同一数据库中重命名集合，则该操作将对源集合和目标集合进行排他（W）锁定。在MongoDB 4.2之前的版本中，在同一数据库中重命名时，该操作将对数据库使用排他（W）锁。（*仅*）如果目标名称空间与源集合位于不同的数据库中，则锁定行为取决于版本：[`renameCollection`](https://docs.mongodb.com/manual/reference/command/renameCollection/#dbcmd.renameCollection)*（MongoDB 4.2.2和更高版本）*在跨数据库重命名集合时，该操作在目标数据库上获得排他（W）锁，并阻塞该数据库上的其他操作，直到操作完成。*（MongoDB 4.2.1和更早版本）*在跨数据库重命名集合时，该操作采用全局排他（W）锁，并阻止其他操作，直到操作完成。 |


以下管理操作将锁定数据库，但仅在很短的时间内保持锁定：


| 运作方式                                                     | 方法 |
| :----------------------------------------------------------- | :--- |
| [`authenticate`](https://docs.mongodb.com/manual/reference/command/authenticate/#dbcmd.authenticate)[`db.auth()`](https://docs.mongodb.com/manual/reference/method/db.auth/#db.auth) |      |
| [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#dbcmd.createUser)[`db.createUser()`](https://docs.mongodb.com/manual/reference/method/db.createUser/#db.createUser) |      |

也可以看看

[MongoDB操作是否可以锁定多个数据库？](https://docs.mongodb.com/manual/faq/concurrency/#faq-concurrency-lock-multiple-dbs)


## 哪些管理命令可以锁定集合？[¶](https://docs.mongodb.com/manual/faq/concurrency/#which-administrative-commands-lock-a-collection)

*在版本4.2中进行了更改。*

以下管理操作需要在集合级别具有排他锁定：


| 运作方式                                                     | 方法                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`create`](https://docs.mongodb.com/manual/reference/command/create/#dbcmd.create)[`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection)[`db.createView()`](https://docs.mongodb.com/manual/reference/method/db.createView/#db.createView) |                                                              |
| [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#dbcmd.createIndexes)[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#db.collection.createIndexes) |                                                              |
| [`drop`](https://docs.mongodb.com/manual/reference/command/drop/#dbcmd.drop)[`db.collection.drop()`](https://docs.mongodb.com/manual/reference/method/db.collection.drop/#db.collection.drop) |                                                              |
| [`dropIndexes`](https://docs.mongodb.com/manual/reference/command/dropIndexes/#dbcmd.dropIndexes)[`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#db.collection.dropIndex)[`db.collection.dropIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndexes/#db.collection.dropIndexes) |                                                              |
| [`renameCollection`](https://docs.mongodb.com/manual/reference/command/renameCollection/#dbcmd.renameCollection)[`db.collection.renameCollection()`](https://docs.mongodb.com/manual/reference/method/db.collection.renameCollection/#db.collection.renameCollection) | *在版本4.2中进行了更改。*如果在同一数据库中重命名集合，则该操作将对源集合和目标集合进行排他（W）锁定。在MongoDB 4.2之前的版本中，在同一数据库中重命名时，该操作将对数据库使用排他（W）锁。（*仅*）如果目标名称空间与源集合位于不同的数据库中，则锁定行为取决于版本：[`renameCollection`](https://docs.mongodb.com/manual/reference/command/renameCollection/#dbcmd.renameCollection)*MongoDB 4.2.2及更高版本*在跨数据库重命名集合时*，*该操作将对目标数据库进行排他（W）锁定，并阻塞该数据库上的其他操作，直到操作完成。*MongoDB 4.2.1和更早版本*在跨数据库重命名集合时，该操作将使用全局排他（W）锁，并阻止其他操作，直到操作完成。 |
| [`reIndex`](https://docs.mongodb.com/manual/reference/command/reIndex/#dbcmd.reIndex)[`db.collection.reIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.reIndex/#db.collection.reIndex) | *在版本4.2中进行了更改。*对于MongoDB 4.2.2和更高版本，这些操作在集合上获得排他（W）锁，并在集合上阻止其他操作，直到完成。对于MongoDB 4.0.0到4.2.1，这些操作采用全局排他（W）锁定并阻止其他操作，直到完成。 |
| [`replSetResizeOplog`](https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) | *在版本4.2中进行了更改。*对于MongoDB 4.2.2及更高版本，此操作对[`oplog`](https://docs.mongodb.com/manual/reference/local-database/#local.oplog.rs)集合进行排他（W）锁定，并阻止对集合的其他操作，直到完成。对于MongoDB 4.2.1和更早版本，此操作采用全局排他（W）锁定并阻止其他操作，直到完成。 |


在MongoDB 4.2之前的版本，以上命令操作了对数据库的排他锁，阻止所有数据库操作*和*它的汇集，直到操作完成。



## MongoDB操作是否可以锁定多个数据库？



以下MongoDB操作可能会在多个数据库上获得并持有锁：


| 运作方式                                                     | 方法                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.copyDatabase()`](https://docs.mongodb.com/manual/reference/method/db.copyDatabase/#db.copyDatabase) | 此操作获得全局（W）排他锁，并阻止其他操作，直到完成为止。    |
| [`reIndex`](https://docs.mongodb.com/manual/reference/command/reIndex/#dbcmd.reIndex)[`db.collection.reIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.reIndex/#db.collection.reIndex) | *在版本4.2中进行了更改。*对于MongoDB 4.0.0到4.2.1，这些操作采用全局排他（W）锁定并阻止其他操作，直到完成。从MongoDB 4.2.2开始，这些操作仅获得排他（W）集合锁，而不是全局排他锁。在MongoDB 4.0之前，这些操作获得了排他（W）数据库锁。 |
| [`renameCollection`](https://docs.mongodb.com/manual/reference/command/renameCollection/#dbcmd.renameCollection) | *在版本4.2中进行了更改。*对于MongoDB 4.2.1和更早版本，此操作在重命名数据库之间的集合时会获得全局排他（W）锁，并阻止其他操作直到完成。从MongoDB 4.2.2开始，此操作仅在目标数据库上获得互斥（W）锁定，在源数据库上获得意向共享（r）锁定，并在源集合上获得共享（S）锁定，而不是全局排他锁。 |
| [`replSetResizeOplog`](https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) | *在版本4.2中进行了更改。*对于MongoDB 4.2.1和更早版本，此操作获得全局排他（W）锁并阻止其他操作，直到完成。从MongoDB 4.2.2开始，此操作仅在[`oplog`](https://docs.mongodb.com/manual/reference/local-database/#local.oplog.rs) 集合上获得排他（W）锁，而不是全局排他锁。 |


## 分片如何影响并发性？[¶](https://docs.mongodb.com/manual/faq/concurrency/#how-does-sharding-affect-concurrency)


[分片通过将集合分布在多个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例，提高并发的能力，允许分片服务器（即[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)进程）来并发地执行针对下游[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例的任意数量的操作。


在分片群集中，锁适用于每个单独的分片，而不适用于整个群集。也就是说，每个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例都独立于分片群集中的其他实例，并使用自己的 [锁](https://docs.mongodb.com/manual/faq/concurrency/#faq-concurrency-locking)。一个 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例上的操作不会阻止任何其他实例上的操作。



## 并发性如何影响副本集上的主节点？[¶](https://docs.mongodb.com/manual/faq/concurrency/#how-does-concurrency-affect-a-replica-set-primary)


对于副本集，当MongoDB写入[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)上的集合时 ，MongoDB还将写入主节点上的[oplog](https://docs.mongodb.com/manual/reference/glossary/#term-oplog)-`local`数据库中的特殊集合。因此，MongoDB必须同时锁定集合所在的数据库和`local` 数据库。[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)必须同时锁定这两个库保持数据库一致，并确保写入操作，甚至包括复制，是“全有或全无”的操作。


写入[副本集时](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)，锁的范围适用于[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)。



## 并发如何影响副本集的从节点？[¶](https://docs.mongodb.com/manual/faq/concurrency/#how-does-concurrency-affect-secondaries)


在进行副本集[复制同步](https://docs.mongodb.com/manual/reference/glossary/#term-replication)时，MongoDB不会将写入连续的应用到 [从节点](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)。从节点批量收集oplog记录，然后并行应用这些批处理。从节点将按照出现在操作日志中的顺序应用写入操作。


从MongoDB 4.0开始，如果从节点正在复制，则读取从数据的[WiredTiger](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger)快照读取的 [目标从节点](https://docs.mongodb.com/manual/core/read-preference/#replica-set-read-preference)数据。这允许读取与复制同时进行，同时仍保证数据的一致视图。在MongoDB 4.0之前的版本中，将在所有正在进行的复制完成之前阻止对从节点的读取操作。有关更多信息，请参见[多线程复制](https://docs.mongodb.com/manual/core/replica-set-sync/#replica-set-internals-multi-threaded-replication)。

## MongoDB是否支持事务？


由于单个文档可以包含关联数据（译者注：通过内嵌文档或数组的方式），否则它们将在关系模式中的各个父子表之间建模，因此MongoDB的单文档原子操作已经提供了满足大多数应用程序数据完整性需求的事务语义。可以在单个操作中写入一个或多个字段，包括对多个子文档和数组元素的更新。MongoDB提供的保证可确保在文档更新时完全隔离。任何错误都会导致操作回滚，以使客户端获得一致的文档视图。


但是，对于需要对多个文档（在单个或多个集合中）进行读写原子性的情况，MongoDB支持多文档事务：


- **在版本4.0中**，MongoDB支持副本集上的多文档事务。

- **在版本4.2中**，MongoDB引入了分布式事务，它增加了对分片群集上多文档事务的支持，并合并了对副本集上多文档事务的现有支持。

  有关MongoDB中事务的详细信息，请参阅 [事务](https://docs.mongodb.com/manual/core/transactions/)页面。


重要

在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应代替高效的模式设计。在许多情况下， [非规范化数据模型（嵌入式文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，对数据进行适当的建模将最大程度地减少对多文档事务的需求。


有关其他事务使用方面的注意事项（例如运行时限制和oplog大小限制），另请参见 [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)。


## MongoDB提供什么隔离保证？

根据读取的关注点，客户端可以在[持久](https://docs.mongodb.com/manual/reference/glossary/#term-durable)写入之前看到写入结果。要控制是否可以回滚读取的数据，客户端可以使用该`readConcern`选项。


有关信息，请参阅：

- [读取隔离，一致性和因近原则](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/)
- [原子性和事务](https://docs.mongodb.com/manual/core/write-operations-atomicity/)
- [读关注](https://docs.mongodb.com/manual/reference/read-concern/)



原文链接：https://docs.mongodb.com/manual/faq/concurrency/

译者：钟秋

update:小芒果
