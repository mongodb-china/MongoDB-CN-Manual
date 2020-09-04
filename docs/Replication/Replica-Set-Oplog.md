# 副本集日志

在本页

- [日志大小](https://docs.mongodb.com/manual/core/replica-set-oplog/#oplog-size) 
- [可能需要更大日志的工作负载](https://docs.mongodb.com/manual/core/replica-set-oplog/#workloads-that-might-require-a-larger-oplog-size)
- [日志状态](https://docs.mongodb.com/manual/core/replica-set-oplog/#oplog-status)
- [慢日志应用程序](https://docs.mongodb.com/manual/core/replica-set-oplog/#slow-oplog-application)
- [日志集合的特性](https://docs.mongodb.com/manual/core/replica-set-oplog/#oplog-collection-behavior)


oplog(操作日志)是一个特殊的[有限集合](https://docs.mongodb.com/manual/reference/glossary/#term-capped-collection)，它对数据库中所存储数据的所有修改操作进行滚动记录。

说明

从MongoDB 4.0开始，与其他有限集合不同，oplog集合可以超过其配置的大小限制，以避免[大多数提交点](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime)被删除。

MongoDB在[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)上应用数据库操作，然后将这些操作记录到主节点的oplog上。然后[从节点](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)成员会以异步的方式复制并应用这些操作。所有副本集成员都包含一个oplog的副本，其位于[local.oplog.rs ](https://docs.mongodb.com/manual/reference/local-database/#local.oplog.rs)集合中，该集合可以让副本集成员维护数据库的当前状态。

为了便于复制，所有副本集成员将心跳(ping)发送给所有其他成员。任何[从节点](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)成员都可以从任何其他成员导入oplog条目。

oplog中的每个操作都是[幂等的](https://docs.mongodb.com/manual/reference/glossary/#term-idempotent)。也就是说，对目标数据集应用一次或多次oplog操作都会产生相同的结果。


## 日志大小


当您第一次启动一个副本集成员时，如果您没有指定oplog大小，MongoDB将创建一个默认大小的oplog。[[1\]](https://docs.mongodb.com/manual/core/replica-set-oplog/#oplog)


- 对于Unix和Windows系统

  oplog大小依赖于存储引擎：

  | 存储引擎           | 默认oplog大小    | 下限  | 上限 |
  | ------------------ | ---------------- | ----- | ---- |
  | In-Memory存储引擎  | 物理内存的5%     | 50MB  | 50GB |
  | WiredTiger存储引擎 | 空闲磁盘空间的5% | 990MB | 50GB |

- 对于64-bit macOS系统

  默认的oplog大小是192MB物理内存或空闲磁盘空间，具体取决于存储引擎:

  | 存储引擎           | 默认oplog大小     |
  | ------------------ | ----------------- |
  | In-Memory存储引擎  | 192MB物理内存     |
  | WiredTiger存储引擎 | 192MB空闲磁盘空间 |


在大多数情况下，默认的oplog大小就足够了。例如，如果一个oplog是空闲磁盘空间的5%，并且可容纳24小时的操作记录，那么从节点从oplog停止复制条目的时间可以长达24小时，并且不会因oplog条目变得太陈旧而无法继续复制。但是，大多数副本集的操作容量要小得多，它们的oplog可以容纳更多的操作。

在 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 创建一个oplog前，您可以使用 [`oplogSizeMB`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.oplogSizeMB) 选项来定义oplog的大小。一旦您第一次启动副本集成员后，可使用 [`replSetResizeOplog`](https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) 管理命令去改变oplog的大小。 [`replSetResizeOplog`](https://docs.mongodb.com/manual/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) 命令允许您动态调整oplog大小而无需重新启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 进程。

[[1\]](https://docs.mongodb.com/manual/core/replica-set-oplog/#id2) | 从MongoDB 4.0开始，oplog可以超过其配置的大小限制，来避免删除[大多数提交点](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime)。 


## 可能需要更大日志大小的工作负载


如果您可以预测您的副本集的工作负载与以下模式之一相似，那么您可能希望创建一个比默认值更大的oplog。相反，如果您的应用程序主要执行读操作，而写操作很少，那么更小的oplog可能就足够了。

以下工作负载可能需要大容量的oplog。


### 一次更新多个文档

为了保持幂等性，oplog必须将多次更新转换为单个操作。这会使用大量的oplog空间，而不会相应增加数据大小或磁盘使用。


### 删除与插入的数据量相等

如果删除的数据量与插入的数据量大致相同，则数据库在磁盘使用方面不会显著增长，但操作日志的大小可能相当大。


### 大量的就地更新


如果工作负载中很大一部分是不增加文档大小的更新，那么数据库会记录大量操作，但不会更改磁盘上的数据量。


## Oplog状态


为了查看oplog的状态，包括oplog的大小和操作的时间范围，可使用[`rs.printReplicationInfo()`](https://docs.mongodb.com/manual/reference/method/rs.printReplicationInfo/#rs.printReplicationInfo) 方法。有关oplog状态的更多内容，请参见[检查Oplog大小](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#replica-set-troubleshooting-check-oplog-size)。


### 复制延迟和流控制


在各种异常情况下，对从节点oplog的更新可能会滞后于预期的性能时间。在从节点上使用 [`db.getReplicationInfo()`](https://docs.mongodb.com/manual/reference/method/db.getReplicationInfo/#db.getReplicationInfo)命令，以及根据复制状态输出结果来评估复制的当前状态，并确定是否存在任何意外的复制延迟。


从MongoDB 4.2开始，管理员可以限制主节点应用其写操作的速度，目的是将大多数提交延迟保持在可配置参数[`flowControlTargetLagSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.flowControlTargetLagSeconds)最大值之下。

默认情况下，流控制是启用的。

说明

为了进行流控制，副本集/分片集群必须满足：参数[featureCompatibilityVersion (FCV)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) 设置为4.2，并启用majority读关注。也就是说，如果FCV不是4.2，或者读关注majority被禁用，那么启用流控制将不起作用。

更多信息请参见[流控制](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#flow-control)。


## 慢Oplog应用程序


从4.2版本开始（从4.0.6开始也是可行的），副本集的副本成员会记录oplog中应用时间超过慢操作阈值的慢操作条目。这些慢oplog信息被记录在从节点[`REPL`](https://docs.mongodb.com/manual/reference/log-messages/#REPL) 组件的文本`applied op: took ms`中。

```
2018-11-16T12:31:35.886-0500 I REPL   [repl writer worker 13] applied op: command { ... }, took 112ms
```

记录在从节点上的慢操作应用程序有：

- 不受 [`slowOpSampleRate`](https://docs.mongodb.com/manual/reference/configuration-options/#operationProfiling.slowOpSampleRate)的影响；例如，所有的慢oplog条目被记录在从节点上。
- 不受 [`logLevel`](https://docs.mongodb.com/manual/reference/parameters/#param.logLevel)/[`systemLog.verbosity`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.verbosity) 级别的影响（或者[`systemLog.component.replication.verbosity`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.component.replication.verbosity) 的级别）；例如，对于oplog条目，从节点仅记录慢oplog条目。增加日志的冗余级别不会导致记录所有的oplog条目。
- 不会被捕获器抓取到，并且不受捕获级别的影响。

更多有关慢操作阈值设置的信息，请参见：

- [`mongod --slowms`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-slowms)
- [`slowOpThresholdMs`](https://docs.mongodb.com/manual/reference/configuration-options/#operationProfiling.slowOpThresholdMs)
-  [`profile`](https://docs.mongodb.com/manual/reference/command/profile/#dbcmd.profile) 命令或者 [`db.setProfilingLevel()`](https://docs.mongodb.com/manual/reference/method/db.setProfilingLevel/#db.setProfilingLevel) shell帮助命令


## Oplog集合的特性

如果你的MongoDB部署使用的是WiredTiger存储引擎，你无法从副本集任何成员中删除 `local.oplog.rs` 集合。这个限制适用于单成员和多成员的副本集。如果一个节点临时宕机并试图在重启过程中重新应用oplog，那么删除oplog可能会导致副本集中的数据不一致。


原文链接：https://docs.mongodb.com/manual/core/replica-set-oplog/

译者：李正洋
