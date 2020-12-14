# 常见问题解答:MongoDB 存储

>  在本页
>
> - [存储引擎基础](https://docs.mongodb.com/v4.2/faq/storage/#storage-engine-fundamentals)
> - [是否支持在副本集中混用存储引擎？](https://docs.mongodb.com/v4.2/faq/storage/#can-you-mix-storage-engines-in-a-replica-set)
> - [WiredTiger存储引擎](https://docs.mongodb.com/v4.2/faq/storage/#wiredtiger-storage-engine)
> - [数据存储诊断](https://docs.mongodb.com/v4.2/faq/storage/#data-storage-diagnostics)

本文档回答了有关MongoDB存储系统的常见问题。



## 存储引擎基础

### 什么是存储引擎？

存储引擎是数据库的一部分，负责管理如何在内存和磁盘上存储数据。许多数据库支持多个存储引擎，其中不同的引擎对于特定的工作负载可能性能更好。例如，一个存储引擎可能为包含大量读取操作的工作负载提供更好的性能，而另一个引擎则可能为写入操作提供更高的吞吐量。

**另请参阅**

[存储引擎](https://docs.mongodb.com/v4.2/core/storage-engines/)



## 是否可以在副本集中混用不同的存储引擎？

可以。您可以让副本集成员使用不同的存储引擎（比如WiredTiger和内存引擎）

> **注意：**
>
> 从4.2版本以后，MongoDB彻底移除了废弃的MMAPv1存储引擎。



## WiredTiger存储引擎

### 是否可以将现有的部署升级到WiredTiger引擎?

可以，请参考：

- [将单机节点转为WiredTiger](https://docs.mongodb.com/v4.2/tutorial/change-standalone-wiredtiger/)
- [将副本集转为WiredTiger](https://docs.mongodb.com/v4.2/tutorial/change-replica-set-wiredtiger/)
- [将分片集群转为WiredTiger](https://docs.mongodb.com/v4.2/tutorial/change-sharded-cluster-wiredtiger/)



### WiredTiger引擎能提供多少的压缩（比例）？

压缩数据与未压缩数据的比率取决于您的数据和所使用的压缩库。默认情况下，WiredTiger中的集合数据使用的是[Snappy块压缩](https://docs.mongodb.com/v4.2/reference/glossary/#term-snappy), 也可以使用[zlib](https://docs.mongodb.com/v4.2/reference/glossary/#term-zlib)和[zstd](https://docs.mongodb.com/v4.2/reference/glossary/#term-zstd)压缩。索引数据默认情况下使用[prefix压缩](https://docs.mongodb.com/v4.2/reference/glossary/#term-prefix-compression)。



### 我应该将WiredTiger内部缓存设置为多大？

对于WiredTiger，MongoDB可以利用WiredTiger内部缓存和文件系统缓存。

从MongoDB 3.4开始，默认的WiredTiger内部缓存大小是以下两者中的较大者：

- （RAM - 1 GB）/2
- 256 MB.

例如，在总共有4GB RAM的系统上，WiredTiger缓存将使用1.5GB RAM**`（0.5 *（4 GB-1 GB）= 1.5 GB）`**。 相反，总内存为1.25GB的系统将为WiredTiger缓存分配256MB，因为256MB大于总RAM减去1GB的一半**`（0.5 *（1.25 GB-1 GB）= 128 MB <256 MB）`**。

> 注意：
>
> 在某些情况下，例如在容器中运行时，数据库的内存限制可能低于系统总内存。在这种情况下，内存限制不再是系统总内存，而是最大可用RAM。
>
> 关于内存限制，请参考[`hostInfo.system.memLimitMB`](https://docs.mongodb.com/v4.2/reference/command/hostInfo/#hostInfo.system.memLimitMB)。

默认情况下，WiredTiger对所有集合使用Snappy块压缩，对所有索引使用prefix压缩。压缩默认值是可以在全局级别配置的，也可以在集合和索引创建期间基于每个集合和每个索引进行设置。

WiredTiger内部缓存中的数据与磁盘格式使用不同的表示形式：

- 文件系统缓存中的数据与磁盘上的格式相同，包括所有对数据文件进行压缩的收益。操作系统使用文件系统缓存来减少磁盘I/O。
- 加载到WiredTiger内部缓存中的索引具有与磁盘格式不同的数据表示形式，但仍可以利用索引prefix压缩来减少RAM使用量。索引prefix压缩从索引字段中删除通用前缀。
- WiredTiger内部缓存中的集合数据未经压缩，并使用与磁盘格式不同的表示形式。块压缩可以节省大量的磁盘存储空间，但是必须对数据进行解压缩才能由服务器端进行处理。

通过文件系统缓存，MongoDB自动使用WiredTiger缓存或其他进程未使用的所有可用内存。

要调整WiredTiger引擎的内部缓存大小，请参考`storage.wiredTiger.engineConfig.cacheSizeGB`以及`--wiredTigerCacheSizeGB`参数。应避免将WiredTiger内部缓存的大小增加到其默认值以上。

> **注意：**
>
> `storage.wiredTiger.engineConfig.cacheSizeGB`限制WiredTiger内部缓存的大小。操作系统将使用可用的空闲内存进行文件系统缓存，从而允许压缩的MongoDB数据文件保留在内存中。此外，操作系统将使用任何可用的RAM来缓冲文件系统块和文件系统缓存。
>
> 为了容纳更多的RAM使用者，您可能需要降低WiredTiger内部缓存的大小。

默认的WiredTiger内部缓存大小值假定每台计算机有且仅有一个 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 实例。如果一台机器包含多个MongoDB实例，则应减小设置以容纳其他 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 实例。

如果您在**无法**访问系统中所有可用RAM的容器（例如`lxc`，`cgroups`，`Docker`等）中运行 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) ，则必须将`storage.wiredTiger.engineConfig.cacheSizeGB`设置为小于该容器中可用的RAM数量。确切的数量取决于容器中运行的其他进程。请参考[`memLimitMB`](https://docs.mongodb.com/v4.2/reference/command/hostInfo/#hostInfo.system.memLimitMB)。

要查看有关缓存和淘汰率的统计信息，请参阅`serverStatus`命令返回的`wiredTiger.cache`字段的相关内容。



### WiredTiger引擎多久写一次磁盘？

- **检查点** 

  从3.6版开始，MongoDB将WiredTiger配置为以60秒的间隔创建检查点（即将快照数据写入磁盘）。在早期的版本中，MongoDB将在WiredTiger中以60秒的间隔或在写满2GB日志数据时对用户数据进行创建检查点操作，两个条件中任意一个满足即可。

- **日志数据**

  WiredTiger在以下任一情况满足时会将缓存的日记记录同步到磁盘：
  
  - 对于副本集成员（主或者从节点成员），
  
    - 是否有等待oplog条目的操作。需要等待oplog条目的操作包括：
      - 针对oplog转发扫描查询
      - 读取操作作为[因果一致性会话](https://docs.mongodb.com/v4.2/core/read-isolation-consistency-recency/#causal-consistency)的一部分
    - 另外，对于从节点成员，在每批oplog条目被应用之后。
  
  - 如果写操作包括或隐含了 [`j: true`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.j)的写关注。
  
    >  注意：
    >
    > 当 [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault) 为`true`时，设置为[`"majority"`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern."majority")的写关注隐含了`j:true`。
  
  - 每隔100ms（参考 [`storage.journal.commitIntervalMs`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.journal.commitIntervalMs)）
  
  - WiredTiger创建新的日志文件时。由于MongoDB使用的日志文件大小限制为100MB，因此WiredTiger大约每100MB数据创建一个新的日志文件。
  
  

### 如何在WiredTiger引擎中回收磁盘空间？

WiredTiger存储引擎在删除文档时会维护数据文件中的空记录列表。WiredTiger可以重用此空间，但是并不会将其返回给操作系统，除非是在非常特殊的情况下。

WiredTiger引擎的可用于重用的空闲空间反映在 [`db.collection.stats()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.stats/#db.collection.stats) 的结果的`wiredTiger.block-manager.file bytes availablefor reuse`字段中。

为了使WiredTiger存储引擎可以将这些空闲空间归还给操作系统，可以对数据文件进行碎片整理。这可以通过`compact`命令实现。更多相关信息及其他考虑，请参考 [`compact`](https://docs.mongodb.com/v4.2/reference/command/compact/#dbcmd.compact)。



## 数据存储诊断

### 如何确认一个集合的大小？

要查看集合的统计信息，包括数据大小，请使用[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell中的`db.collection.stats()`方法。以下示例为在`order`集合上执行`db.collection.stats()`：

```
db.orders.stats();
```

MongoDB同样提供了以下方法来返回集合的具体大小：

- [`db.collection.dataSize()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.dataSize/#db.collection.dataSize) 会返回集合中未压缩的数据大小，以字节为单位。
- [`db.collection.storageSize()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.storageSize/#db.collection.storageSize) 会返回集合在磁盘上占用的大小，以字节为单位。如果集合的数据是压缩过的（WiredTiger引擎默认带数据压缩），则存储大小反映的是压缩后的大小，可能会比[`db.collection.storageSize()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.storageSize/#db.collection.storageSize) 返回的结果小一些。
- `db.collection.totalIndexSize`会返回集合中所有索引的大小，以字节为单位。如果一个索引使用了前缀压缩（WiredTiger引擎默认索引使用前缀压缩），则返回的大小反映的是压缩后的大小。

以下的脚本会输出每个数据库的统计信息：

```
db.adminCommand("listDatabases").databases.forEach(function (d) {
   mdb = db.getSiblingDB(d.name);
   printjson(mdb.stats());
})
```

以下的脚本会输出每个数据库中每个集合的统计信息：

```
db.adminCommand("listDatabases").databases.forEach(function (d) {
   mdb = db.getSiblingDB(d.name);
   mdb.getCollectionNames().forEach(function(c) {
      s = mdb[c].stats();
      printjson(s);
   })
})
```



### 如何确认集合中每个索引的大小？

要查看为每个索引分配的数据大小，使用 [`db.collection.stats()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.stats/#db.collection.stats) 方法，然后查看返回结果文档中的`indexSizes`字段。

如果索引使用前缀压缩（这也是WiredTiger引擎的默认设置），返回的大小代表了压缩后的大小。



### 如何获得有关数据库存储使用的相关信息？

mongo shell中的 [`db.stats()`](https://docs.mongodb.com/v4.2/reference/method/db.stats/#db.stats) 方法返回“激活”数据库的当前状态。有关返回字段的描述，请参考[dbStats输出](https://docs.mongodb.com/v4.2/reference/command/dbStats/#dbstats-output)。



译者：刘翔

校对：牟天垒
