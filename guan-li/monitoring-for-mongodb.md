# MongoDB监控

MongoDB Manual (Version 4.2）> Administration > Monitoring for MongoDB

在本页面

* [监控策略](https://docs.mongodb.com/manual/administration/monitoring/index.html#monitoring-strategies)
* [MongoDB 报告工具](https://docs.mongodb.com/manual/administration/monitoring/index.html#mongodb-reporting-tools)
* [进程日志](https://docs.mongodb.com/manual/administration/monitoring/index.html#process-logging)
* [诊断性能问题](https://docs.mongodb.com/manual/administration/monitoring/index.html#diagnosing-performance-issues)
* [复制集和监控](https://docs.mongodb.com/manual/administration/monitoring/index.html#replication-and-monitoring)
* [分片和监控](https://docs.mongodb.com/manual/administration/monitoring/index.html#sharding-and-monitoring)
* [存储节点看门狗](https://docs.mongodb.com/manual/administration/monitoring/index.html#storage-node-watchdog)



监控是所有数据库管理的重要组成部分。牢牢掌握 MongoDB 的报告，将使您能够评估数据库的状态并维持部署不会出现危险。此外，MongoDB 的正常运行参数使您能够在问题升级为故障之前进行诊断。

本文档概述了 MongoDB 中可用的监控实用程序和报告统计信息。它还介绍了用于监视副本集和分片群集的诊断策略和建议。



## 监控策略

MongoDB 提供了各种方法来收集正在运行的 MongoDB 实例的状态数据：

* 从版本 4.0 开始，MongoDB 为单机和副本集提供[免费的云监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。
* MongoDB 分发了一组实用程序，这些实用程序提供了数据库活动的实时报告。
* MongoDB 提供了各种[数据库命令](https://docs.mongodb.com/manual/reference/command/)，这些[命令](https://docs.mongodb.com/manual/reference/command/)以更高的保真度返回有关当前数据库状态的统计信息。
* [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是一种云托管的数据库即服务，用于运行，监控和维护 MongoDB 部署。
* [MongoDB 云管理器](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server)是一项托管服务，可监控正在运行的 MongoDB 部署以收集数据并基于该数据提供可视化和警报。
* MongoDB Ops Manager 是 [MongoDB 企业高级版](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) 中提供的本地解决方案，它监视正在运行的 MongoDB 部署以收集数据并提供基于该数据的可视化和警报。

每种策略都可以帮助回应不同的问题，并且在不同的情况下很有用。这些方法是互补的。



## MongoDB 报告工具

本节概述了用MongoDB 分发的报告方法。它还提供了每种方法最适合您解决的各种问题的示例。



### 免费监控

_4.0 版本中的新功能_

MongoDB 为单机或副本集提供[免费的云监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。

默认情况下，您可以在运行时使用[`db.enableFreeMonitoring()`](https://docs.mongodb.com/manual/reference/method/db.enableFreeMonitoring/#db.enableFreeMonitoring) 和 [`db.disableFreeMonitoring()`](https://docs.mongodb.com/manual/reference/method/db.disableFreeMonitoring/#db.disableFreeMonitoring)开启/关闭免费监控。

免费监控可提供长达 24 小时的数据。有关更多详细信息，请参见[免费监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。





### 实用工具

MongoDB 发行版包含许多实用程序，可快速返回有关实例性能和活动的统计信息。通常，这些对于诊断问题和评估正常操作最有用。



#### `mongostat`

[`mongostat`](https://docs.mongodb.com/database-tools/mongostat/#bin.mongostat) 根据数据库操作类型（例如插入，查询，更新，删除等）捕获并返回计数。这些计数报告服务器上的负载分布。

使用[`mongostat`](https://docs.mongodb.com/database-tools/mongostat/#bin.mongostat)来了解操作类型的分布情况，并通知容量规划。有关详细信息，请参见 [mongostat manual](https://docs.mongodb.com/manual/reference/program/mongostat/) 手册。



#### `mongotop`

[`mongotop`](https://docs.mongodb.com/database-tools/mongotop/#bin.mongotop)跟踪并报告 MongoDB 实例当前的读写活动，并基于每个集合报告这些统计信息。

使用[`mongotop`](https://docs.mongodb.com/database-tools/mongotop/#bin.mongotop)来检查数据库活动和使用是否符合您的期望。有关详细信息，请参见[mongotop manual](https://docs.mongodb.com/manual/reference/program/mongotop/)手册。



#### HTTP 控制台

_在 3.6 版本中做的更改：_ MongoDB 3.6 删除了 MongoDB 弃用的 HTTP 接口和 REST API。





### 命令

MongoDB 包含许多报告数据库状态的命令。

这些数据可以提供比上面讨论的实用程序更好的粒度级别。您可以考虑在脚本和程序中使用它们的输出来开发自定义警报，或根据实例的活动来修改应用程序的行为。 [`db.currentOp`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp) 方法是用于识别数据库实例正在进行操作的另一有用工具。



#### `serverStatus`

使用 [`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) 命令，或shell 程序的[`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus) ，可以返回数据库状态的一般概述，包含磁盘使用，内存使用，连接，日志和索引访问。该命令将快速返回，不会影响 MongoDB 的性能。

[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) 输出一个 MongoDB 实例状态的帐户。此命令很少直接运行。在大多数情况下，聚合后的数据更有意义，就像使用监控工具（包括 [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server) 和 [Ops Manager](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server)）所看到的那样。尽管如此，所有管理员都应该熟悉[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus)所提供的数据 。



#### `dbStats`

使用 [`dbStats`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbcmd.dbStats) 命令，或shell 程序的 [`db.stats()`](https://docs.mongodb.com/manual/reference/method/db.stats/#db.stats) ，可以返回一个介绍存储使用和数据量的文档。 [`dbStats`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbcmd.dbStats) 反映存储的使用量，包含在数据库中的数据的数量，对象集合和索引计数器。

使用此数据监视指定数据库的状态和存储容量。此输出还允许您比较数据库之间的使用情况，并确定数据库中[文档](https://docs.mongodb.com/manual/reference/glossary/#term-document)的平均大小。



#### `collStats`

shell 程序的 [`collStats`](https://docs.mongodb.com/manual/reference/command/collStats/#dbcmd.collStats) 或 [`db.collection.stats()`](https://docs.mongodb.com/manual/reference/method/db.collection.stats/#db.collection.stats)提供类似于 [`dbStats`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbcmd.dbStats) 集合级别的统计信息，包括集合中对象的数量，集合的大小，集合使用的磁盘空间量以及有关其索引的信息。



#### `replSetGetStatus`

 [`replSetGetStatus`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#dbcmd.replSetGetStatus) 命令（来自内核程序的[`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#rs.status)）可以返回副本集状态的概述。 [replSetGetStatus](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/) 文档详细介绍了副本集和统计信息及其成员的状态和配置。

使用此数据可确保正确配置了复制，并检查了当前主机与副本集的其他成员之间的连接。



#### 托管 \(SaaS\) 监控工具

这些作为托管服务提供的监视工具，通常通过付费订阅提供。

| **名称** | **说明** |
| :--- | :--- |
| [MongoDB 云管理器](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server) | MongoDB Cloud Manager 是基于云的用于管理 MongoDB 部署的服务套件。MongoDB Cloud Manager 提供监控，备份和自动化功能。有关本地解决方案，另请参阅 [MongoDB 企业高级版中提供的 Ops Manager](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server). |
| [VividCortex](https://www.vividcortex.com/) | VividCortex 提供了能在一秒钟里对 MongoDB 的[生产工作负载和查询性能](https://www.vividcortex.com/product/how-it-works)进行深入观测的能力，跟踪延迟，吞吐量，错误等，以确保您的应用程序在 MongoDB 上具有可伸缩性和出色的性能。 |
| [Scout](http://scoutapp.com/) | 一些插件, 包括[MongoDB 监控，](https://scoutapp.com/plugin_urls/391-mongodb-monitoring)[MongoDB 慢查询 ](http://scoutapp.com/plugin_urls/291-mongodb-slow-queries)和 [MongoDB 复制集监控](http://scoutapp.com/plugin_urls/2251-mongodb-replica-set-monitoring)。 |
| [Server Density](http://www.serverdensity.com/) | [适用于 MongoDB 的仪表盘](http://www.serverdensity.com/mongodb-monitoring/)，针对 MongoDB 的报警，复制故障转移时间表和 iPhone， iPad 和安卓的移动应用程序。 |
| [应用性能管理](http://ibmserviceengage.com/) | IBM 有一个提供了一个应用性能管理 SaaS 产品，其中包括用于 MongoDB 以及其他应用程序和中间件的监视器。 |
| [New Relic](http://newrelic.com/) | New Relic 为应用程序性能管理提供全面支持。另外，New Relic 的插件和深入观察能力使您能够从 New Relic 中的 Cloud Manager 查看监控指标。 |
| [Datadog](https://www.datadoghq.com/) | [基础架构监视](http://docs.datadoghq.com/integrations/mongodb/)，以可视化 MongoDB 部署的性能。 |
| [SPM 性能监控](https://sematext.com/spm) | [监视，异常检测和警报](https://sematext.com/spm/integrations/mongodb-monitoring/)，SPM 监视所有主要的 MongoDB 指标以及基础设施。对于Docker 和其他应用程序指标，例如 Node.js，Java，NGINX，Apache，HAProxy 或 Elasticsearch，SPM 提供指标和日志的关联。 |
| [Pandora FMS](http://www.pandorafms.com/) | 潘多拉 FMS 提供 [PandoraFMS-mongodb-monitoring](http://blog.pandorafms.org/how-to-monitor-mongodb-or-how-to-keep-your-users-happy/) 插件用来监控 MongoDB。 |



## 进程记录

在正常操作期间， [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 和 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例报告一个真实账号的所有服务器活动和操作，要么是标准输出，要么输出到日志文件。以下运行时设置控制这些选项。

* [`quiet`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.quiet)限制写入日志或输出的信息量。
* [`verbosity`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.verbosity)增加写入日志或标准输出的信息量。您还可以在运行时使用 shell 程序中的 [`logLevel`](https://docs.mongodb.com/manual/reference/parameters/#param.logLevel) 参数或 [`db.setLogLevel()`](https://docs.mongodb.com/manual/reference/method/db.setLogLevel/#db.setLogLevel) 方法来修改日志记录的详细程度。
* [`path`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path)启用日志记录到文件，而不是标准输出。调整此设置时，必须指定日志文件的完整路径。
* [`logAppend`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.logAppend)将信息添加到日志文件，而不是覆盖文件。



注意

您可以将这些配置操作指定为 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/) 或 [mongos](https://docs.mongodb.com/manual/reference/program/mongos/) 的命令行参数，例如：

```text
mongod -v --logpath /var/log/mongodb/server1.log --logappend
```

[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例以 [`verbose`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.verbosity) 模式启动，追加数据到日志文件 `/var/log/mongodb/server1.log/`。



以下[数据库命令](https://docs.mongodb.com/manual/reference/glossary/#term-database-command)也会影响日志记录：

* [`getLog`](https://docs.mongodb.com/manual/reference/command/getLog/#dbcmd.getLog)显示来自[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)进程日志的最新日志。

* [`logRotate`](https://docs.mongodb.com/manual/reference/command/logRotate/#dbcmd.logRotate)只为[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 进程进行滚动日志文件。请参阅[滚动日志文件](https://docs.mongodb.com/manual/tutorial/rotate-log-files/)。

  

### 日志编辑

_3.4 版中的新功能：_仅在 MongoDB 企业版中可用

运行有 [`security.redactClientLogData`](https://docs.mongodb.com/manual/reference/configuration-options/#security.redactClientLogData) 的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 在打印日志之前，编辑与日志事件相关联的信息，只留下的元数据，源文件，或与该事件有关的行号。[`security.redactClientLogData`](https://docs.mongodb.com/manual/reference/configuration-options/#security.redactClientLogData)以牺牲详细诊断信息为代价防止潜在的敏感信息进入系统日志。

例如，以下操作会插入一个文档到没有日志编辑的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 中。[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)已设置 [`systemLog.component.command.verbosity`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.component.command.verbosity) 为 `1`:

```text
db.clients.insertOne( { "name" : "Joe", "PII" : "Sensitive Information" } )
```

此操作将产生以下日志事件：

```text
2017-06-09T13:35:23.446-04:00 I COMMAND  [conn1] command internal.clients
   appName: "MongoDB Shell"
   command: insert {
      insert: "clients",
      documents: [ {
            _id: ObjectId('593adc5b99001b7d119d0c97'),
            name: "Joe",
            PII: " Sensitive Information"
         } ],
      ordered: true
   }
   ...
```

运行有 [`security.redactClientLogData`](https://docs.mongodb.com/manual/reference/configuration-options/#security.redactClientLogData) 的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 执行相同的插入操作生成以下日志事件：

```text
2017-06-09T13:45:18.599-04:00 I COMMAND  [conn1] command internal.clients
   appName: "MongoDB Shell"
   command: insert {
      insert: "###", documents: [ {
         _id: "###", name: "###", PII: "###"
      } ],
      ordered: "###"
   }
```

[`redactClientLogData`](https://docs.mongodb.com/manual/reference/configuration-options/#security.redactClientLogData)同 [静态加密](https://docs.mongodb.com/manual/core/security-encryption-at-rest/)和 [TLS/SSL\(传输加密\)](https://docs.mongodb.com/manual/core/security-transport-encryption/)结合使用，以符合监管要求。



## 诊断性能问题

使用 MongoDB 开发和操作应用程序时，您可能需要分析数据库性能作为应用程序的性能。 [MongoDB性能](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/)讨论了一些可能影响性能的操作因素。



## 复制和监控

除了对任何 MongoDB 实例的基本监视要求之外，对于副本集，管理员还必须监视复制滞后。“复制滞后”是指将[主](https://docs.mongodb.com/manual/reference/glossary/#term-primary)磁盘上的写操作复制（即复制）到 [辅助](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)磁盘上所花费的时间。可以接受一些小的延迟时间，但是随着复制滞后的增加，会出现严重的问题，包括：

* 主数据库上的缓存压力越来越大。

* 滞后期间发生的操作不会复制到一个或多个次级。如果您使用复制来确保数据的持久性，那么特别长的延迟可能会影响数据集的完整性。

* 如果复制滞后超过操作日志 \([oplog](https://docs.mongodb.com/manual/reference/glossary/#term-oplog)\) 的长度，则 MongoDB 将必须在辅助数据库上执行初始同步，从[主](https://docs.mongodb.com/manual/reference/glossary/#term-primary)数据库复制所有数据并重建所有索引。在通常情况下，这种情况并不常见，但是如果您将 oplog 配置为小于默认值，则可能会出现问题。

  

  注意

  oplog 的大小只能在第一次运行时使用 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 命令的[`--oplogSize`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-oplogsize)参数进行配置，或者最好是在 MongoDB 配置文件中设置 [`oplogSizeMB`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.oplogSizeMB) 。如果您在使用[`--replSet`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-replset)选项运行之前未在命令行上指定此选项，则[ `mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)将创建一个默认大小的操作日志。

  默认情况下，操作日志是 64 位系统上总可用磁盘空间的 5%。有关更改 oplog 大小的更多信息，请参阅[“更改 Oplog 的大小”](https://docs.mongodb.com/manual/tutorial/change-oplog-size/)。

  

### 流量控制

从 MongoDB 4.2 开始，管理员可以限制主数据库应用其写入的速率，以将[`多数承诺`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime)的延迟保持在可配置的最大值[`flowControlTargetLagSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.flowControlTargetLagSeconds)以下。

默认情况下，流量控制是[`开启的`](https://docs.mongodb.com/manual/reference/parameters/#param.enableFlowControl)



注意

为了启用流量控制，副本集/分片集群必须具有: [featureCompatibilityVersion \(FCV\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) 4.2 以及[`开启大多数读`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern)。也就是说，如果 FCV 不是 `4.2` 或者禁用了大多数读，则启用的流量控制无效。

另请参阅：[检查复制延迟](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#replica-set-replication-lag)。



### 副本集状态

复制问题通常是由成员之间的网络连接问题引起的，或者是由于[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)没有资源来支持应用程序和复制通信而导致的。要检查副本的状态，使用[`replSetGetStatus`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#dbcmd.replSetGetStatus)或在shell程序中使用以下帮助程序：

```text
rs.status()
```

[`replSetGetStatus`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#dbcmd.replSetGetStatus)参考提供了此输出的更深入的概述视图。通常监听 `optimeDate` 的值，并特别注意[primary](https://docs.mongodb.com/manual/reference/glossary/#term-primary)和[secondary](https://docs.mongodb.com/manual/reference/glossary/#term-secondary)之间的时间差。

从 MongoDB 4.0 开始，操作日志可以超出其配置的大小限制，以避免删除 [`majority commit point`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime)。



### 免费监控

注意

从 4.0 版本开始，MongoDB 为独立和副本集提供[免费监控](https://docs.mongodb.com/manual/administration/free-monitoring/) 。有关更多信息，请参见[免费监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。



### Oplog 条目的慢应用

从版本 4.2 开始（版本 4.0.6 开始可用）,副本集的辅助成员现在 [记录操作日志条目](https://docs.mongodb.com/manual/release-notes/4.2/#slow-oplog)所花费的时间比应用慢操作阈值长。这些慢日志消息记录在 [`REPL`](https://docs.mongodb.com/manual/reference/log-messages/#REPL) 组件下的[诊断日志](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-logpath)中的辅助日志中，使用了格式为 `applied op: <oplog entry> took <num>ms`的文本文件。这些慢操作日志条目仅取决于慢操作阈值。它们不依赖于日志级别（在系统级别或组件级别），配置级别或运行缓慢的采样率。探查器不会捕获缓慢的操作日志条目。



## 分片和监控

在大多数情况下，[分片群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)的组件与所有其他 MongoDB 实例一样，都将从相同的监视和分析中受益。此外，群集需要进一步监视以确保数据在节点之间有效分布，并且分片操作正常运行。

请参阅[分片](https://docs.mongodb.com/manual/sharding/)以获取更多信息的文档。



### 配置服务器

[配置数据库](https://docs.mongodb.com/manual/reference/glossary/#term-config-database)保留一个地图识别哪些文件是哪个分片。集群在分片之间移动[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)时会更新此映射 。当无法访问配置服务器时，某些分片操作将变得不可用，例如移动块和启动 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例。但是，仍然可以从已运行的 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例访问群集 。

由于无法访问的配置服务器会严重影响分片群集的可用性，因此您应该监视配置服务器，以确保群集保持良好的平衡并且 mongos 实例可以重新启动。

[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server) 和 [Ops Manager](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) 监视配置服务器，并且在无法访问配置服务器时可以创建通知。有关更多信息，请参阅 [MongoDB Cloud Manager 文档](https://docs.cloudmanager.mongodb.com/) and [Ops Manager 文档](https://docs.opsmanager.mongodb.com/current/application)。



### 平衡和块分布

最有效的[分片群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)部署会均衡分片之间的[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。为了实现这一点，MongoDB 具有一个后台[平衡器](https://docs.mongodb.com/manual/reference/glossary/#term-balancer)进程，该进程用于分配数据，以确保始终在各个[分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)之间最佳地分配块。

通过 [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell 发出 [`db.printShardingStatus()`](https://docs.mongodb.com/manual/reference/method/db.printShardingStatus/#db.printShardingStatus) 或 [`sh.status()`](https://docs.mongodb.com/manual/reference/method/sh.status/#sh.status) 命令将返回整个集群的概述，包括数据库名称和块列表。



### 过期的锁

要检查数据库的锁定状态，请使用[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell 连接到[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例。发出以下命令序列以切换到 `config` 数据库并显示分片数据库上的所有未完成锁：

```text
use config
db.locks.find()
```

平衡过程采用特殊的“平衡器”锁，以防止发生其他平衡活动。在 `config` 数据库中，使用以下命令查看“balancer”锁：

```text
db.locks.find( { _id : "balancer" } )
```

在 3.4 版本中做了更改_: 从 3.4 版本开始，CSRS 配置服务器的主服务器使用进程 ID 为“ConfigServer” 的进程持有“平衡器”锁。此锁永远不会释放。要确定平衡器是否正在运行，请参阅[检查平衡器是否正在运行](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#sharding-balancing-is-running)。



## 存储节点看门狗程序

注意

* 从 MongoDB 4.2 开始，MongoDB社区版和企业版均提供了[Storage Node Watchdog存储节点看门狗](https://docs.mongodb.com/manual/administration/monitoring/index.html#storage-node-watchdog)。
* 在早期版本（3.2.16 +，3.4.7 +，3.6.0 +，4.0.0 +）中，存储节点看门狗仅在 MongoDB企业版中可用。



存储节点看门狗监视以下 MongoDB 目录以检测文件系统无响应：

* [`--dbpath`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-dbpath)目录
* 如果启用了[`journaling`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-journal) ，则在 [`--dbpath`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-dbpath) 目录内的`journal`目录
* [`--logpath`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-logpath) 文件
* [`--auditPath`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-auditpath) 文件

默认情况下，存储节点看门狗是禁用的。你可以在启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)时，通过将[`watchdogPeriodSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.watchdogPeriodSeconds)参数设置为大于或等于 60 的整数。 但是，一旦启用，您可以暂停[存储节点看门狗](https://docs.mongodb.com/manual/administration/monitoring/index.html#storage-node-watchdog)程序并在运行时重新启动。有关详细信息，请参见[`watchdogPeriodSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.watchdogPeriodSeconds) 参数。

如果包含受监视目录的任何文件系统都没有响应，则存储节点监视程序将终止[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)并退出，并以状态码 61 退出。 如果是副本集[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)的[主节点](https://docs.mongodb.com/manual/reference/glossary/#term-primary)，则终止会启动[故障转移](https://docs.mongodb.com/manual/reference/glossary/#term-failover)，从而允许另一个成员成为主节点。

一旦 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 终止，在同一机器，可能无法干净地重新启动它。

> **符号链接**
>
>  如果其任何受监视目录是到其他卷的符号链接，则存储节点监视程序将不监视该符号链接目标。
>
> 例如，如果[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)使用[`storage.directoryPerDB: true`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.directoryPerDB) \(或 [`--directoryperdb`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-directoryperdb)\)链接数据库目录到另一个数据卷，则存储节点看门狗程序将不遵循符号链接来监视目标。

存储节点看门狗检测无响应的文件系统并终止的最长时间几乎是[`watchdogPeriodSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.watchdogPeriodSeconds)的值的两倍。



原文链接：[https://docs.mongodb.com/v4.2/administration/monitoring/](https://docs.mongodb.com/v4.2/administration/monitoring/)

译者：谢伟成
