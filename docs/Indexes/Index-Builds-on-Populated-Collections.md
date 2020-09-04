## 在填充的集合上建立索引

**在本页面**

- [行为](#行为)
- [索引构建对数据库性能的影响](#影响)
- [复制集中的索引构建](#复制)
- [构建失败和恢复](#构建)
- [监视进行中的索引构建](#监视)
- [终止进行中的索引构建](#终止)
- [索引建立过程](#过程)

*MongoDB 4.2版本新变化*

针对已填充的集合的MongoDB索引构建需要对该集合的排他性读写锁定。需要对集合进行读取或写入锁定的操作必须等待，直到[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)释放锁定为止 。MongoDB 4.2使用了优化的构建过程，该过程仅在索引构建的开始和结束时持有排他锁。其余的构建过程将产生交错的读写操作。

索引构建过程总结如下:

1. **初始化**

   [mongod](../reference/program/mongod.html#bin.mongod) 进程对正在编制索引的集合使用独占锁。所有对该集合的读写操作将阻塞直到[`mongod`](../reference/program/mongod.html#bin.mongod) 进程释放锁。在此期间，应用程序无法访问集合。

1. **数据摄取和加工**
  [`mongod`](../reference/program/mongod.html#bin.mongod)进程释放上一过程中获取的所有锁，然后针对被索引的集合获取一系列意向锁。在此期间，应用程序可以对集合发出读写操作。

1. **清理**
  [`mongod`](../reference/program/mongod.html#bin.mongod)进程释放上一过程中获取的所有锁，然后针对被索引集合获取独占锁。这时将阻塞对该集合所有读写操作直到[`mongod`](../reference/program/mongod.html#bin.mongod)进程释放锁。应用程序此时无法访问该集合。

1. **完成**
[`mongod`](../reference/program/mongod.html#bin.mongod)进程标记索引状态为已可用，然后释放索引构建过程中的所有锁。

索引构建过程中的加锁描述细节参见[Index Build Process](#index-build-process)章节。更深入了解MongoDB的加锁行为参见[FAQ: Concurrency](_home_admin_logs_confluencetemp_import_2020_05_28_0a506c1d1590656636262555511694_faq_concurrency)。
### <span id="行为">行为</span>
MongoDB 4.2索引构建完全替代了以前MongoDB版本中支持的索引构建过程。如果指定为[`createIndexes`](../reference/command/createIndexes.html#dbcmd.createIndexes) 或它的shell助手[`createIndexes()`](../reference/method/db.collection.createIndexes.html#db.collection.createIndexes)和[`createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#db.collection.createIndexes)， MongoDB会忽略后台索引构建选项。

#### 与前景和背景构建进行比较

MongoDB的早期版本支持在前台或后台构建索引。前台索引构建速度很快，能够生成更高效的索引数据结构，但是需要在构建期间阻塞对被索引的集合的父数据库的所有读写访问。后台索引构建速度较慢，效率较低，但允许在构建过程中对数据库及其集合进行读写访问。

*MongoDB 4.2版本变化*

MongoDB 4.2索引构建只在构建过程的开始和结束时对被索引的集合获得独占锁，以保护元数据的更改。构建过程的其余部分使用后台索引构建的生成行为来最大化构建期间对集合的读写访问。4.2索引构建仍然产生高效的索引数据结构，尽管有更宽松的锁定行为。

MongoDB 4.2的索引构建性能至少与后台索引构建相当。对于在构建过程中很少或没有收到更新的工作负载，4.2索引构建构建的速度可以与基于相同数据的前台索引构建的速度一样快。

使用[`db.currentOp()`](../reference/method/db.currentOp.html#db.currentOp) 命令监视正在进行的索引构建的进度。

####  索引构建期间的冲突约束
对集合具有强制约束作用的索引，例如：[unique](index-unique) 索引，  [`mongod`](../reference/program/mongod.html#bin.mongod)进程会在索引构建完成后对所有预先存在的和并发写入的文档进行约束性检查。如果任何文档违反了索引约束条件，[`mongod`](../reference/program/mongod.html#bin.mongod)进程将终止构建并抛出错误。

举例：对`inventory`集合的`product_sku`属性构建unique特性索引。如果任意文档的`product_sku`属性有重复值，索引构建过程仍可成功开始，如果在构建结束时仍然存在任何冲突，[`mongod`](../reference/program/mongod.html#bin.mongod)进程会终止构建并抛出错误。

类似的，在索引构建的过程中，应用程序可成功对`inventory`集合写入`product_sku`属性值重复的文档。如果在构建结束时存在任何索引约束冲突，[`mongod`](../reference/program/mongod.html#bin.mongod)进程会终止构建并抛出错误。

降低因违反约束而导致索引生成失败的风险：

- 校验集合中没有违反索引约束的文档。
- 停止所有可能违反该集合索引约束条件的应用程序写入操作。

###  <span id="影响">索引构建对数据库性能的影响</span>
**高负载写入时的索引构建**

在目标集合处于高负载写入状态时执行索引构建操作，会造成写入性能下降和更长的索引构建时间。

考虑指定一个维护窗口用于构建索引，在此期间停止或减少对目标集合的写入操作。以减少索引生成过程对性能的潜在负面影响。

**可用系统内存不足时的索引构建**

命令支持在集合上构建多个索引。[`createIndexes`](../reference/command/createIndexes.html#dbcmd.createIndexes)命令同时使用内存和磁盘上的临时文件空间完成索引构建。[`createIndexes`](../reference/command/createIndexes.html#dbcmd.createIndexes)命令默认的内存使用限制是500MB，该空间在所有使用[`createIndexes`](../reference/command/createIndexes.html#dbcmd.createIndexes)命令生成的索引之间共享。一旦在构建索引时到达该空间限制，[`createIndexes`](../reference/command/createIndexes.html#dbcmd.createIndexes)命令将使用[`--dbpath`](../reference/program/mongod.html#cmdoption-mongod-dbpath)目录下`_tmp`文件夹下的临时磁盘文件空间，用于完成索引构建。

你可以自定义[`maxIndexBuildMemoryUsageMegabytes`](../reference/parameters.html#param.maxIndexBuildMemoryUsageMegabytes)参数，更改该空间大小限制。设置更高的内存空间可以在索引大小超500MB时更快完成索引构建。当然，该参数设置过高也会导致占用太多无用内存造成系统内存错误。

如果主机内存有限，你可能需要安排一个维护期，增加整个系统内存，再更改[`mongod`](../reference/program/mongod.html#bin.mongod)进程中的内存参数设置。

###  <span id="复制">复制集中的索引构建</span>
尽量减少建立索引对以下方面的影响:

- 复制集中，使用滚动索引生成策略，如[Build Indexes on Replica Sets](_home_admin_logs_confluencetemp_import_2020_05_28_0a506c1d1590656636262555511694_tutorial_build-indexes-on-replica-sets)章节所述。
- 拥有分片复制集的分片集群中，使用滚动索引生成策略，如[Build Indexes on Sharded Clusters](_home_admin_logs_confluencetemp_import_2020_05_28_0a506c1d1590656636262555511694_tutorial_build-indexes-on-sharded-clusters)章节所述。

你可以在[primary](../reference/glossary.html#term-primary)节点执行索引构建。索引生成完成后，secondaries节点进行复制并开始索引构建。复制集中在开始构建索引之前请注意如下风险：

**Secondaries节点可能不进行同步**

Secondary节点的索引构建将阻塞应用正在执行的对构建索引集合的事务操作。 [`mongod`](../reference/program/mongod.html#bin.mongod)进程在构建索引完成前将无法使用任何oplog。

如果索引生成在执行操作或命令时持有独占锁，则对被索引集合的复制写操作也可能被延迟到索引构建完成之后。[`mongod`](../reference/program/mongod.html#bin.mongod) 进程无法使用任何oplog直到锁释放。如果复制延迟时间超过secondary节点的[oplog window](replica-set-oplog.html#replica-set-oplog-sizing)，secondary将不进行同步，需要通过[resynchronization](replica-set-sync.html#replica-set-initial-sync)来恢复。

在构建索引之前，使用[`rs.printReplicationInfo()`](../reference/method/rs.printReplicationInfo.html#rs.printReplicationInfo)命令鉴别每个副本集成员配置的时间区间中可处理的oplog大小。你可以增大oplog大小[increase the oplog size](_home_admin_logs_confluencetemp_import_2020_05_28_0a506c1d1590656636262555511694_tutorial_change-oplog-size)，降低不同步的概率。例如，设置oplog的窗口可以覆盖72个小时的操作记录，只要确保secondary节点可以容忍这么久的复制延迟。

或者在维护时间窗口中执行索引构建，应用程序停止对该索引集合的所有事物操作、写操作和元数据操作。

**Secondary节点的索引构建可能造成读写操作延迟**

MongoDB 4.2索引生成过程的开始和结束时获取正在索引的集合的独占锁。当secondary节点索引生成持有独占锁时，该secondary节点将暂停任何读写操作，直到索引生成释放该锁为止。

**索引生成完成后，辅助进程索引将删除**

在secondary节点完成索引构建的复制之前，在primary节点删除索引，将不会中断secondary节点的构建。当secondary节点执行索引删除的复制操作时，其须等待之前的索引生成操作执行复制完毕。此外，由于索引删除是集合上的元数据操作，因此索引删除会暂停该secondary节点上的复制。

### <span id="构建">构建失败和恢复</span>
#### 单个`mongod`进程的索引构建中断
如果[`mongod`](../reference/program/mongod.html#bin.mongod)进程在索引构建时终止了，索引构建任务和所有进程将丢失。重启[`mongod`](../reference/program/mongod.html#bin.mongod)进程不会重新执行索引构建，你必须重新运行[`createIndex()`](../reference/method/db.collection.createIndex.html#db.collection.createIndex) 操作来重启索引构建。

#### Primary节点`mongod`进程的索引构建中断
如果primary节点在索引构建时停止了，索引构建任务和所有进程将丢失。你必须重新运行[`createIndex()`](../reference/method/db.collection.createIndex.html#db.collection.createIndex) 操作来重启索引构建。
#### Secondary节点`mongod`进程的索引构建中断
如果secondary节点在索引构建时停止了，索引构建任务将保留。重启 [`mongod`](../reference/program/mongod.html#bin.mongod)进程将恢复索引构建并从头重新开始。

启动进程会在任意已恢复的索引生成之后暂停。所有操作，包括复制将进入等待直到索引构建完成。如果secondary节点的oplog未在时间窗口区间完成缩影索引构建的复制，secondary不再进行这部分oplog的复制集的同步，需要[resynchronization](replica-set-sync.html#replica-set-initial-sync)恢复。

如果你重启了[`mongod`](../reference/program/mongod.html#bin.mongod)进程作为独立的复制集节点实例或删除[`--replSetName`](../reference/program/mongod.html#cmdoption-mongod-replset))。`mongod`进程仍将从头恢复索引构建。你可以使用[`storage.indexBuildRetry`](../reference/configuration-options.html#storage.indexBuildRetry)配置文件设置或写入命令行参数[`--noIndexBuildRetry`](../reference/program/mongod.html#cmdoption-mongod-noindexbuildretry)。

> MONGODB 4.0以上版本
>
> 你不能对副本集中的一个[`mongod`](../reference/program/mongod.html#bin.mongod)进程实例，设置[`storage.indexBuildRetry`](../reference/configuration-options.html#storage.indexBuildRetry)选项  或  [`--noIndexBuildRetry`](../reference/program/mongod.html#cmdoption-mongod-noindexbuildretry)选项。

#### 构建过程中的回滚
从4.0版本开始，MongoDB在任意正在执行索引构建的进程完成后进行[rollback](replica-set-rollbacks)。
### <span id="监视">监视进行中的索引构建</span>
你可以在[`mongo`](../reference/program/mongo.html#bin.mongo)  shell中执行[`db.currentOp()`](../reference/method/db.currentOp.html#db.currentOp)命令，查看索引构建操作的状态。筛选当前操作中的索引创建操作，参见[Active Indexing Operations](../reference/method/db.currentOp.html#currentop-index-creation)操作。

[`msg`](../reference/command/currentOp.html#currentOp.msg)包含了索引构建当前阶段的完成百分比。

### <span id="终止">终止进行中的索引构建</span>
终止primary节点或单个  `mongod` 进程中正在执行的索引构建命令，请在[`mongo`](../reference/program/mongo.html#bin.mongo)进程中使用[`db.killOp()`](../reference/method/db.killOp.html#db.killOp)命令。当终止索引生成时，[`db.killOp()`](../reference/method/db.killOp.html#db.killOp)命令可能不会立即执行，并可能在大部分索引生成操作完成后执行。

你无法终止一个已经在复制集secondary节点进行复制的索引构建操作。你必须首先在primary节点[`drop`](../reference/method/db.collection.dropIndex.html#db.collection.dropIndex)删除索引。secondary节点将复制删除操作并在索引构建完成后删除索引。所用复制操作将阻塞直到索引构建完成并执行完删除操作。

尽量降低复制集和拥有复制集的分片集群的构建索引影响，参见：

- [在复制集上建立索引](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/)
- [在分片群集上建立索引](https://docs.mongodb.com/manual/tutorial/build-indexes-on-sharded-clusters/)
### <span id="过程">索引建立过程</span>
如下表格描述索引构建过程中的每个阶段：

| 阶段 | 描述 |
| --- | --- |
| Lock | [`mongod`](../reference/program/mongod.html#bin.mongod) 进程获得索引集合的独占`X`锁。该集合的所有读写操作被阻塞，包括应用程序对该集合所有复制写操作或者元数据指令。[`mongod`](../reference/program/mongod.html#bin.mongod)进程不会释放独占锁。 |
| 初始化 | [`mongod`](../reference/program/mongod.html#bin.mongod)进程在该初始状态创建三个数据结构：初始索引元数据项。一张临时表（"side writes table"），用来存储构建过程中对索引集合进行写入时生成的key。一张临时表(“constraint violation table”)，用于存储可能导致重复键约束冲突的所有文档。 |
| Lock | [`mongod`](../reference/program/mongod.html#bin.mongod) 进程将之前获取的独占锁`X`降级为意图独占锁`IX`，[`mongod`]进程周期性的释放该锁，允许进行交错读写操作。 |
| 扫描集合 | [`mongod`](../reference/program/mongod.html#bin.mongod)进程对集合中每个文档生成一个key，并将该key存储到外部分类器中，如果 [`mongod`](../reference/program/mongod.html#bin.mongod)进程在集合扫描期间发现重复key，它会将该key存储在constraint violation表中以备后续处理。如果mongod进程在生成key时遇到任何其他错误，构建将失败并出现错误。一旦mongod进程完成集合扫描，它将分类的key转储到索引中。 |
| 进程端写入表 | [`mongod`](../reference/program/mongod.html#bin.mongod) 使用先进先出方式处理Side Writes Table表中数据，如果遇到重复key，将该key写入constraint violation table表以备后续处理。如果[`mongod`](../reference/program/mongod.html#bin.mongod)进程在处理键时出现任何异常错误，构建将失败。对于每个在索引构建过程中写入集合的文档，mongod进程都会给该文档生成一个key，并将其存储到side write table表中。[`mongod`](../reference/program/mongod.html#bin.mongod)使用快照系统设置要处理的key的数量限制。 |
| Lock | [`mongod`](../reference/program/mongod.html#bin.mongod)进程将索引集合上的意图独占锁`IX`升级为共享`S`锁。这会阻塞该集合的所有写操作，包括应用程序对该集合的任何复制写操作或元数据操作。 |
| 处理完临时端写表 | [`mongod`](../reference/program/mongod.html#bin.mongod)进程继续处理Side Writes Table表中的存量数据。[`mongod`](../reference/program/mongod.html#bin.mongod)进程在该阶段可能会暂停复制。如果遇到重复key，将该key写入constraint violation table表以备后续处理。如果[`mongod`](../reference/program/mongod.html#bin.mongod)进程在处理键时出现任何异常错误，构建将失败。 |
| Lock | [`mongod`](../reference/program/mongod.html#bin.mongod)进程将索引集合的共享`S`锁升级成独占`X`锁。该集合的所有读写操作被阻塞，包括应用程序对该集合所有复制写操作或者元数据指令操作。[`mongod`](../reference/program/mongod.html#bin.mongod)进程不会释放独占锁。 |
| 下侧写表 | [`mongod`](../reference/program/mongod.html#bin.mongod)在处理完Side Write Table表中所有数据后将其删除。如果遇到重复key，将该key写入constraint violation table表以备后续处理。如果[`mongod`](../reference/program/mongod.html#bin.mongod)进程在处理键时出现任何异常错误，构建将失败。 |
| 流程约束违规表 | [`mongod`](../reference/program/mongod.html#bin.mongod)进程使用先入先出的方式处理Constraint Violation Table表中数据，之后删除该表。如果其中任何键仍然出现重复键错误，[`mongod`](../reference/program/mongod.html#bin.mongod)将终止构建并抛出错误。[`mongod`](../reference/program/mongod.html#bin.mongod)进程在处理完Constraint Violation Table表后或出现重复键约束时将删除该表。 |
| 将索引标记为就绪 | [`mongod`](../reference/program/mongod.html#bin.mongod)将索引元数据更新为可使用状态。 |
| Lock | [`mongod`](../reference/program/mongod.html#bin.mongod)进程释放该索引集合的独占`X`锁。 |

译者：程哲欣
