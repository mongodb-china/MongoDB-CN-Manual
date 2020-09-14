# 诊断命令

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

| 名称 | 描述 |
| :--- | :--- |
| [`availableQueryOptions`](diagnostic-commands.md) | 内部命令，报告当前MongoDB实例的功能。 |
| [`buildInfo`](diagnostic-commands.md) | 显示有关MongoDB构建的统计信息。 |
| [`collStats`](diagnostic-commands.md) | 报告指定集合的存储利用率静态信息。 |
| [`connPoolStats`](diagnostic-commands.md) | 报告从此MongoDB实例到部署中其他MongoDB实例的传出连接的统计信息。 |
| [`connectionStatus`](diagnostic-commands.md) | 报告当前连接的身份验证状态。 |
| [`cursorInfo`](diagnostic-commands.md) | 在MongoDB 3.2中已删除。替换为`metrics.cursor`。 |
| [`dataSize`](diagnostic-commands.md) | 返回数据范围的数据大小。供内部使用。 |
| [`dbHash`](diagnostic-commands.md) | 返回数据库及其集合的哈希值。 |
| [`dbStats`](diagnostic-commands.md) | 报告指定数据库的存储利用率统计信息。 |
| [`diagLogging`](diagnostic-commands.md) | 在MongoDB 3.6中已删除。要捕获，重放和分析发送到您的MongoDB部署的命令，请使用`mongoreplay`。 |
| [`driverOIDTest`](diagnostic-commands.md) | 将ObjectId转换为字符串以支持测试的内部命令。 |
| [`explain`](diagnostic-commands.md) | 返回有关各种操作执行的信息。 |
| [`features`](diagnostic-commands.md) | 报告当前MongoDB实例中可用的功能。 |
| [`getCmdLineOpts`](diagnostic-commands.md) | 返回带有MongoDB实例及其解析选项的运行时参数的文档。 |
| [`getLog`](diagnostic-commands.md) | 返回最近的日志消息。 |
| [`hostInfo`](diagnostic-commands.md) | 返回反映基础主机系统的数据。 |
| [isSelf](diagnostic-commands.md) | 内部命令支持测试。 |
| [`listCommands`](diagnostic-commands.md) | 列出当前`mongod`实例提供的所有数据库命令。 |
| [`lockInfo`](diagnostic-commands.md) | 内部命令，返回有关当前正在保留或挂起的锁的信息。仅适用于 `mongod`实例。 |
| [`netstat`](diagnostic-commands.md) | 报告部署内连接性的内部命令。仅适用于`mongos`实例。 |
| [`ping`](diagnostic-commands.md) | 测试部署内连接性的内部命令。 |
| [`profile`](diagnostic-commands.md) | 数据库事件探查器的接口。 |
| [`serverStatus`](diagnostic-commands.md) | 返回有关实例范围的资源利用率和状态的集合指标。 |
| [`shardConnPoolStats`](diagnostic-commands.md) | 报告`mongos`连接池上的统计信息，以供客户端针对分片进行操作。 |
| [`top`](diagnostic-commands.md) | 返回`mongod`实例中每个数据库的原始使用情况统计信息。 |
| [`validate`](diagnostic-commands.md) | 内部命令，用于扫描集合的数据并为正确性编制索引。 |
| [`whatsmyuri`](diagnostic-commands.md) | 返回有关当前客户端信息的内部命令。 |

译者：李冠飞

校对：

