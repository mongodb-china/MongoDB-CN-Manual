# 日志记录

[Storage](https://docs.mongodb.com/v4.2/storage/) &gt; Journaling

## 日志记录

在本页面

* [日志记录和WiredTiger存储引擎](https://docs.mongodb.com/v4.2/core/journaling/#journaling-and-the-wiredtiger-storage-engine)
* [日志记录和内存存储引擎](https://docs.mongodb.com/v4.2/core/journaling/#journaling-and-the-in-memory-storage-engine)

  为了在发生故障时提供持久性，MongoDB使用_预写日志记录_到磁盘[journal](https://docs.mongodb.com/v4.2/reference/glossary/#term-journal)文件中。

### 日志记录和WiredTiger存储引擎

**重要**

本节中提到的_log_是指WiredTiger预写日志（即日志），而不是MongoDB日志文件。

[WiredTiger](https://docs.mongodb.com/v4.2/core/wiredtiger/)使用[检查点](https://docs.mongodb.com/v4.2/core/wiredtiger/#storage-wiredtiger-checkpoints)以提供磁盘上数据的一致视图，并允许MongoDB从最后一个检查点恢复。但是，如果MongoDB在检查点之间意外退出，则需要日记来恢复上一个检查点之后发生的信息。

**注意**

从MongoDB 4.0开始，对于使用WiredTiger存储引擎的副本集成员，不能指定[`--nojournal`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-nojournal)选项或[`storage.journal.enabled:false`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.journal.enabled)。

使用日志记录的恢复过程：

1.在数据文件中查找最后一个检查点的标识符。 2.在日记文件中搜索与最后一个检查点的标识符匹配的记录。 3.从上一个检查点开始，将操作应用于日志文件。

#### 日志记录过程

_于3.2版本中变更_

使用日志功能，WiredTiger为每个客户端发起的写操作创建一个日记记录。日志记录包括由初始写入引起的任何内部写入操作。例如，对集合中文档的更新可能会导致对索引的修改；WiredTiger创建单个日志记录，其中包含更新操作及其关联的索引修改。

MongoDB将WiredTiger配置为使用内存缓冲来存储日记记录。线程进行协调以分配并复制到其缓冲区中。全部日记记录最多可缓存128kB。

WiredTiger在以下任一情况下将缓冲的日记记录同步到磁盘：

* 对于副本集成员（主节点和从节点成员），
  * 如果有操作在等待操作日志条目。可以等待操作日志条目的操作包括：
    * 针对oplog转发扫描查询
    * 读取操作作为[因果一致会话](https://docs.mongodb.com/v4.2/core/read-isolation-consistency-recency/#causal-consistency)的一部分执行
  * 另外，对于从节点成员，在每次批量处理oplog条目之后。
* 如果写操作包含或隐含了[`j:true`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.j)的写关注。

  ```text
  注意
  ```

  如果[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为true，则写关注[`“majority”`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.%22majority%22)会隐含`j: true`。

* 每100毫秒一次（请参阅[`storage.journal.commitIntervalMs`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.journal.commitIntervalMs)）。
* WiredTiger创建新的日记文件时。由于MongoDB使用的日志文件大小限制为100 MB，因此WiredTiger大约每100 MB数据创建一个新的日志文件。

  重要

  在两次写操作之间，虽然日记记录保留在WiredTiger缓冲区中，但是强制关闭[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)可能会导致更新丢失。

  同样请参阅

[`serverStatus`](https://docs.mongodb.com/v4.2/reference/command/serverStatus/#dbcmd.serverStatus)命令在“wiredTiger.log”字段中返回有关WiredTiger日记统计信息的信息。

#### 日志文件

对于日志文件，MongoDB在[`dbPath`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.dbPath)目录下会创建一个名为“journal”的子目录。 WiredTiger日志文件的名称具有以下格式`WiredTigerLog.<sequence>`，其中`<sequence>`是从`0000000001`开始的零填充数字。

**日志记录**

日志文件包含每个客户端的初始写操作记录

* 日记记录包括由初始写入引起的任何内部写入操作。例如，对集合中文档的更新可能会导致对索引的修改；WiredTiger创建单个日志记录，其中包含更新操作及其关联的索引修改。
* 每个记录都有一个唯一的标识符。
* WiredTiger的最小日志记录大小为128字节。

**压缩**

默认情况下，MongoDB将WiredTiger配置为对其日记数据使用snappy压缩。要指定其他压缩算法或不进行压缩，请使用[`storage.wiredTiger.engineConfig.journalCompressor`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.wiredTiger.engineConfig%20.journalCompressor)设置。详细信息请参阅[Change WiredTiger Journal Compressor](https://docs.mongodb.com/v4.2/tutorial/manage-journaling/#manage-journaling-change-wt-journal-compressor)。

注意

如果日志记录小于或等于128字节（WiredTiger的最小值[日志记录大小](https://docs.mongodb.com/v4.2/core/journaling/#wt-jouraling-record)），则WiredTiger不会压缩该记录。

**日志文件大小限制**

MongoDB的WiredTiger日志文件的大小限制为最大大约为100 MB。

* 文件超过该限制后，WiredTiger将创建一个新的日记文件。
* WiredTiger会自动删除旧的日志文件，仅保留从上一个检查点恢复所需的文件。

**预分配**

WiredTiger预分配日志文件。

### 日志和内存存储引擎

从MongoDB Enterprise的3.2.6版本开始，[内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)就成为MongoDB常规可用性（GA）的一部分。因为其数据保留在内存中，所以没有单独的日志。使用写关注[`j:true`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.j)的写操作会被立即确认。

如果副本集中任何有投票权的成员使用[内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/#storage-inmemory)，则必须将[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`。

注意

从版本4.2（以及4.0.13和3.6.14）开始，如果副本集成员使用[内存存储引擎](https://docs.mongodb.com/v4.2/core/inmemory/)（投票或非投票），但副本集的[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为true，则副本集成员会记录一条启动警告。

将[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/v4.2/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`，MongoDB在确认写入之前不会等待[`w：“majority”`](https://docs.mongodb.com/v4.2/reference/write-concern/#writeconcern.%22majority%22)的写操作被写入磁盘日志。因此，`majority`写操作可能会在给定副本集中的大多数节点瞬时丢失（比如崩溃和重启）时回滚。

同样请参阅 [内存存储引擎：持久性](https://docs.mongodb.com/v4.2/core/inmemory/#inmemory-durability)

原文链接：[https://docs.mongodb.com/v4.2/core/journaling/](https://docs.mongodb.com/v4.2/core/journaling/)

译者：王金铷

