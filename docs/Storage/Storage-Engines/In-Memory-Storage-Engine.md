# 内存存储引擎

从MongoDB Enterprise 3.2.6开始，In-Memory内存存储引擎是64位版本中通用可用性（GA）的一部分。 除某些元数据和诊断数据外，In-Memory内存存储引擎不维护任何磁盘上的数据，包括配置数据、索引、用户凭据等。<br />

通过避免磁盘I / O，内存中存储引擎使数据库操作的延迟更可预测。<br />


## **指定In-Memory存储引擎**

要选择in-memory内存存储引擎，配置启动参数即可：<br />

用于--storageEngine选项设置inMemory；或者如果使用配置文件方式，则为storage.engine设置。<br />

--dbpath，如果使用配置文件，则为storage.dbPath。 尽管内存存储引擎不会将数据写入文件系统，但它会在--dbpath中维护小型元数据文件和诊断数据以及用于构建大型索引的临时文件。<br />

例如，从命令行输入参数命令：
> mongod --storageEngine inMemory --dbpath

或者，如果使用[YAML配置文件格式]（[https://docs.mongodb.com/manual/reference/configuration-options/](https://docs.mongodb.com/manual/reference/configuration-options/)）：
> storage:
> engine: inMemory
> dbPath:

请参阅内存选项中有关此存储引擎的配置选项。 除与数据持久性相关的那些选项参数（例如日志记录或静态配置加密）外，大多数mongod配置选项均可用于in-memory内存存储引擎。

> 警告
> 进程关闭后，内存中存储引擎不会保留数据。


## 并发

in-memory内存存储引擎将文档级并发控制用于写入操作。 因此，多个客户端可以同时修改集合的不同文档。


## 内存使用


内存存储引擎要求其所有数据（包括索引，oplog（如果mongod实例是副本集的一部分）等）必须适合指定的--inMemorySizeGB命令行选项或中的storage.inMemory.engineConfig.inMemorySizeGB设置。 YAML配置文件。<br />

默认情况下，in-memory 内存存储引擎使用50％的（物理RAM减去1GB）。<br />

如果写操作将导致数据超过指定的内存大小，则MongoDB返回错误：
> “ WT_CACHE_FULL：操作将溢出缓存”

要指定新大小，请使用YAML配置文件格式的storage.inMemory.engineConfig.inMemorySizeGB设置：<br />
或使用命令行选项--inMemorySizeGB启动服务：
> mongod --storageEngine inMemory --dbpath  --inMemorySizeGB


## Durability 持久性


内存中存储引擎是非持久性的，不会将数据写入持久性存储。 非持久数据包括应用程序数据和系统数据，例如用户，权限，索引，副本集配置，分片群集配置等。<br />

因此，日志或等待数据变得持久的概念不适用于内存中的存储引擎。<br />

如果副本集的任何有投票权的成员使用内存存储引擎，则必须将writeConcernMajorityJournalDefault设置为false。

> 注意
> 从版本4.2（以及4.0.13和3.6.14）开始，如果副本集成员使用内存中的存储引擎（投票或不投票），但是副本集的writeConcernMajorityJournalDefault设置为true，则副本集成员记录a 启动警告。
> 将writeConcernMajorityJournalDefault设置为false时，MongoDB不会等待w：在确认写入之前，“多数”写入将写入磁盘日志。 这样，如果给定副本集中大多数节点的瞬时丢失（例如崩溃和重新启动），多数写入操作可能会回滚。

立即记录指定日记记录的写关注点的写操作。 当mongod实例由于shutdown命令或由于系统错误而关闭时，无法恢复内存中的数据。


## 事务


从MongoDB 4.2开始，副本集和分片群集上支持事务，其中：<br />

主要成员使用WiredTiger存储引擎，辅助成员使用WiredTiger存储引擎或内存中存储引擎。<br />

在MongoDB 4.0中，仅使用WiredTiger存储引擎的副本集支持事务。

> 注意
> 您无法在具有将writeConcernMajorityJournalDefault设置为false的分片的分片群集上运行事务，例如，具有使用in-memory 内存存储引擎的投票成员的分片集群。


## 部署架构


除了独立运行外，使用in-memory内存存储引擎的mongod实例还可以作为副本集的一部分或分片群集的一部分运行。



### 复制集


可以部署将in-memory内存存储引擎用作副本集一部分的mongod实例。 例如，作为三副本集的一部分，您可能需要修改配置：

- 两个mongod实例与内存存储引擎一起运行。
- 一个使用WiredTiger存储引擎运行的mongod实例。 将WiredTiger成员配置为隐藏成员（即hidden：true和优先级：0）。

使用此部署模型，只有与in-memory内存存储引擎一起运行的mongod实例才能成为主要实例。 客户端仅连接到内存存储引擎mongod实例。 即使两个运行内存存储引擎的mongod实例都崩溃并重新启动，它们也可以从运行WiredTiger的成员进行同步。 与WiredTiger一起运行的隐藏mongod实例会将数据持久保存到磁盘，包括用户数据，索引和复制配置信息。


> 注意
> In-memory内存存储引擎要求其所有数据（如果mongod是副本集的一部分，则包括oplog等）都应适合指定的--inMemorySizeGB命令行选项或storage.inMemory.engineConfig.inMemorySizeGB设置。 请参阅内存使用。


### 分片集群


可以将使用内存存储引擎的mongod实例部署为分片群集的一部分。 例如，在分片群集中，您可以拥有一个由以下副本集组成的分片：

- 两个mongod实例与内存存储引擎一起运行
- 一个WiredTiger存储引擎运行的mongod实例。 将WiredTiger成员配置为隐藏成员（即hidden：true和优先级：0）。

在此分片节点上，添加标记inmem。 例如，如果此分片的名称为shardC，请连接到mongos并运行sh.addShardTag（）命令，添加标签。<br />

例如，<br />
向其他分片添加一个单独的标签persisted。

```
sh.addShardTag("shardA", "persisted")
sh.addShardTag("shardB", "persisted")
```

对于应驻留在inmem分片上的每个分片集合，将标签inmem分配给整个块范围：

```
`sh.addTagRange("test.analytics", { shardKey: MinKey }, { shardKey: MaxKey }, "inmem")`
```

对于应该驻留在持久化分片上的每个分片集合，将标签持久化分配给整个块范围：

```
`sh.addTagRange("salesdb.orders", { shardKey: MinKey }, { shardKey: MaxKey }, "persisted")`
```

<br />


译者：徐雷


