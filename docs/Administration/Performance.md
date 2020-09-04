# MongoDB性能

在本页

- [锁性能](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#locking-performance) 
- [连接数](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#number-of-connections) 
- [数据库性能](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#database-profiling) 
- [全时诊断数据采集](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#full-time-diagnostic-data-capture) 

当您开发和操作基于MongoDB的应用时，您或许需要分析应用和数据库的性能表现。应用的性能降级通常是因为数据库访问策略、硬件可用性和数据库连接数设置不正确导致的。

一些用户可能因为采用不合适的索引策略，或使用糟糕的表设计模式，而遭遇应用或数据库性能瓶颈。[锁性能章节](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#analyzing-performance-locks) 探讨这些因素如何对MongoDB内部的死锁产生影响。

性能问题可能说明数据库正在按容量临界值执行，是时候为数据库添加额外的服务器资源。通常应用程序的工作集需与服务器可用物理内存相匹配。

某些性能问题可能是暂时的，与不正常负载有关。[连接数](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#number-of-connections)章节，讨论了一些通过数量缩放释放过度负载的措施。

[Database Profiling](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#database-profiling) can help you to understand what operations are causing degradation. [数据库性能](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#database-profiling)章节或许可以帮助您了解什么类型的操作会造成性能降级。

## 锁性能

MongoDB使用一套锁机制确保数据集的一致性。如果某个操作执行时间较长或是一个队列表单，下一操作请求由于要等待当前操作释放锁而出现性能降级。

与锁相关的慢查询可能是间歇性的，确定是否由于死锁影响了应用性能，请参考[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) 输出内容中的 [锁](https://docs.mongodb.com/manual/reference/command/serverStatus/#server-status-locks) 部分和 [全局锁](https://docs.mongodb.com/manual/reference/command/serverStatus/#globallock) 部分。

 `locks.timeAcquiringMicros`除以`locks.acquireWaitCount`能计算出特定锁模式的平均等待时间。

 `locks.deadlockCount`获取死锁次数。

如果 [`globalLock.currentQueue.total`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.globalLock.currentQueue.total) 值持续较高，有可能有大量的请求在等待锁释放。说明可能有影响性能的并发问题。

如果 [`globalLock.totalTime`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.globalLock.totalTime) 相对于 [`uptime`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.uptime) 较高，说明数据库的死锁已经维持一段时间了。

慢查询可能的原因：索引的无效使用；非最优表设计模式；糟糕的查询结构；系统架构问题；内存不足触发磁盘读取。

## 连接数

某些情形下，应用和数据之间的连接数可能超出了服务器能处理的请求数，[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) json文档中的一些属性可以提供一些洞察。


是如下两个字段的容器：

  - [`connections.current`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.connections.current) 是连接到当前数据库实例的客户端连接总数。
  - [`connections.available`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.connections.available) 是可以给新客户端使用的剩余连接总数。

如果有大量的并发应用请求，数据库可能无法满足需求。您可能需要对数据库进行扩容。

对于“读”较频繁的应用，您需要增加 [复制集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set) 的大小并将读操作路由到 [secondary](https://docs.mongodb.com/manual/reference/glossary/#term-secondary) 节点

对于“写”较频繁的应用，部署 [分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding) 并添加多个 [分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard) 到 [分片集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster) 分散 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例之间的负载。

连接数峰值也可能是应用程序或驱动程序错误的结果。所有官方MongoDB驱动均实现了连接池，支持客户端更高效的使用和复用连接对象。高连接数却未发现相匹配的负载，可能说明驱动或者其他配置发生错误。

通过设置[`maxIncomingConnections`](https://docs.mongodb.com/manual/reference/configuration-options/#net.maxIncomingConnections) 配置指定mongoDB支持的最大传入连接数，该值不可超过操作系统最大范围限制。Unix类操作系统中，系统最大范围限制可以通过 `ulimit` 命令修改，或通过编辑 `/etc/sysctl` 文件修改。更多详情参见 [UNIX ulimit 设置](https://docs.mongodb.com/manual/reference/ulimit/) 章节。

## 数据库性能

[数据库分析器](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) 收集MongoDB实例上执行操作的详细信息。“分析器”的输出能帮助用户识别无效查询和操作。

您可以给一个 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例的单个或全部数据库开启和配置数据库分析器。分析器的配置仅作用于单个 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例，并不会在[复制集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set) 或 [分片集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster) 上传播。

开启和配置分析器参见 [数据库分析器](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) 章节。

以下分析级别可用：

| 级别 | 描述                                                  |
| ----- | ------------------------------------------------------------ |
| `0`   | 分析器关闭且不收集数据，默认配置0。 |
| `1`   | 分析器对执行时间超过 `slowms` 阈值的操作进行数据收集 |
| `2`   | 分析器对所有操作进行数据收集 |

> 重要
>
> 分析器会影响性能且与系统日志共享配置。生产环境开启或设置分析器前请认证考虑性能和安全性影响。
>
> 分析器可能造成的潜在性能降级参见 [分析器开销](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#database-profiling-overhead) 章节

> 注意
>
> 当[`logLevel`](https://docs.mongodb.com/manual/reference/parameters/#param.logLevel)设置成0时，MongoDB慢查询将以[`slowOpSampleRate`](https://docs.mongodb.com/manual/reference/configuration-options/#operationProfiling.slowOpSampleRate)确定的采样速率发送到诊断日志。从MongoDB 4.2开始，复制集Secondaries节点的[所有超过慢查询阈值的oplog条目信息都将输出](https://docs.mongodb.com/manual/release-notes/4.2/#slow-oplog) ，并不遵从这一采样速率。更高级别的 [`logLevel`](https://docs.mongodb.com/manual/reference/parameters/#param.logLevel) 配置下，所有操作都将显示在诊断日志中，无论其延迟时间如何，除了：[secondaries节点的慢oplog条目消息的记录](https://docs.mongodb.com/manual/release-notes/4.2/#slow-oplog)的日志。secondaries节点日志只记录慢的oplog条目；增加 [`logLevel`](https://docs.mongodb.com/manual/reference/parameters/#param.logLevel) 不会记录所有oplog条目。

从MongoDB 4.2开始， [profiler entries(分析器实体)](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) 和 读/写操作的[diagnostic log messages (i.e. mongod/mongos log messages)(诊断日志消息，例如：mongod/mongos 日志消息)](https://docs.mongodb.com/manual/reference/log-messages/#log-message-slow-ops)包括：

- `queryHash` 帮助判别有相同 [query shape](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape) 的慢查询。
- `planCacheKey` 对慢查询的 [query plan cache 查询计划缓存](https://docs.mongodb.com/manual/core/query-plans/) 提供更多详情。

## 诠释诊断数据采集


“mongod”和“mongos”进程包括一个全时诊断数据收集（FTDC）机制，以便于MongoDB公司工程师对MongoDB服务器运行情况进行分析。FTDC数据文件是不可读压缩格式，并且继承与MongoDB数据文件相同的文件访问权限。只有能够访问FTDC数据文件的用户才能传输FTDC数据。工程师不能独立于系统所有者或运营人员访问FTDC数据。MongoDB进程默认运行FTDC。更多MongoDB支持选项请查看 [Getting Started With MongoDB Support](https://www.mongodb.com/support/get-started?jmp=docs)。

FTDC隐私

FTDC数据文件是的不可读压缩格式。MongoDB公司工程师没有系统所有者或运营者的明确许可和帮助，不能访问FTDC数据。

FTDC数据 **从不** 包含如下类型信息：

- 查询、查询谓词或查询结果的示例
- 从任何最终用户集合或索引中采样的数据
- 系统或MongoDB用户凭据或安全证书

FTDC data包含某些主机信息，例如：主机名称，操作系统信息，和用于启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 或 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)的配置。这些信息可能被某些组织或监管机构视为受保护或机密，但通常不被视为个人身份信息（PII）。对于这些字段配置了受保护、机密或PII数据的集群，请在发送FTDC数据之前通知MongoDB公司工程师，以便采取适当的措施。

FTDC定期收集由以下命令生成的统计信息：

- [`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus)
- [`replSetGetStatus`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#dbcmd.replSetGetStatus)(仅mongod)
- 对 [`local.oplog.rs`](https://docs.mongodb.com/manual/reference/local-database/#local.oplog.rs) 表的 [`collStats`](https://docs.mongodb.com/manual/reference/command/collStats/#dbcmd.collStats) 命令。(仅mongod)
- [`connPoolStats`](https://docs.mongodb.com/manual/reference/command/connPoolStats/#dbcmd.connPoolStats)(仅mongos)

依赖于主机操作系统，诊断数据可能包括如下统计：

- CPU使用率
- 内存使用率
- 与性能相关的磁盘利用率。FTDC不包括与存储容量相关的数据。
- 网络性能统计。FTDC只捕获元数据，不捕获或检查任何网络数据包。

FTDC收集在文件交换或启动时以下命令生成的统计信息

- [`getCmdLineOpts`](https://docs.mongodb.com/manual/reference/command/getCmdLineOpts/#dbcmd.getCmdLineOpts)
- [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#dbcmd.buildInfo)
- [`hostInfo`](https://docs.mongodb.com/manual/reference/command/hostInfo/#dbcmd.hostInfo)

[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)进程将FTDC数据文件存储在mongoDB实例 [`storage.dbPath`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath) 下的 `diagnostic.data` 目录中。所有诊断数据文件被存储在这个路径下。举例：[`dbPath`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath) 设置成 `/data/db` ，诊断数据路径则是 `/data/db/diagnostic.data`。

[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 进程将FTDC数据文件存储在相对于 [`systemLog.path`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path) 日志路径设置的诊断目录中。MongoDB截断日志文件扩展名，并将 `diagnostic.data` 连接到剩余的名称。举例： [`path`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path) 设置`/var/log/mongodb/mongos.log`，诊断数据路径为`/var/log/mongodb/mongos.diagnostic.data`。

FTDC默认按如下执行：

- 每秒进行数据采集
- 最大200MB“diagnostic.data”文件夹大小。

这些默认设置旨在向MongoDB公司工程师提供有用的数据，对性能或存储大小的影响最小。这些值仅在MongoDB公司工程师出于特定诊断目的需求时才需修改。

您能在[MongoDB Github Repository](https://github.com/mongodb/mongo/tree/master/src/mongo/db/ftdc)查看FTDC源代码。`ftdc_system_stats_*.ccp` 文件具体定义捕获的任何特定于系统的诊断数据。

以 `diagnosticDataCollectionEnabled: false` 启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 或 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)，或者在配置文件 [`setParameter`](https://docs.mongodb.com/manual/reference/privilege-actions/#setParameter) 中设置该选项，可关闭FTDC。

```
setParameter:
  diagnosticDataCollectionEnabled: false
```

关闭FTDC可能增加MongDB公司工程师分析和调试问题的时间和资源。

原文链接：https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#mongodb-performance

译者：程哲欣
