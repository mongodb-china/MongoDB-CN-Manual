# [ ](#)数据库命令

[]()

在本页面

*   [用户命令](#user-commands)
*   [数据库操作](#database-operations)
*   [审核命令](#auditing-commands)

下文概述的所有命令文档均描述了命令及其可用参数，并提供了每个命令的文档模板或原型。一些命令文档还包括相关的 `mongo`Shell帮助器。

要针对当前数据库运行命令，请使用`db.runCommand()`：

```powershell
db.runCommand( { <command> } )
```

要对`admin`数据库运行管理命令，请使用`db.adminCommand()`：

```powershell
db.adminCommand( { <command> } )
```

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

## <span id="user-commands">用户命令</span>

### 聚合命令

| 名称            | 描述                                     |
| --------------- | ---------------------------------------- |
| [`aggregate`]() | 使用聚合框架执行聚合任务，例如分组。     |
| [`count`]()     | 计算集合或视图中的文档数。               |
| [`distinct`]()  | 显示在集合或视图中为指定键找到的不同值。 |
| [`mapReduce`]() | 对大型数据集执行map-reduce聚合。         |

### 地理空间命令

| 名称            | 描述                                              |
| --------------- | ------------------------------------------------- |
| [`geoSearch`]() | 执行使用MongoDB的haystack索引功能的地理空间查询。 |

### 查询和写操作命令

| 名称                | 描述                               |
| ------------------- | ---------------------------------- |
| [`delete`]()        | 删除一个或多个文档。               |
| [`find`]()          | 返回集合或视图中的文档。           |
| [`findAndModify`]() | 返回并修改单个文档。               |
| [`getLastError`]()  | 返回上一个操作的成功状态。         |
| [`getMore`]()       | 返回当前由游标指向的批处理文档。   |
| [`insert`]()        | 插入一个或多个文档。               |
| [`resetError`]()    | *不推荐使用*。重置上一个错误状态。 |
| [`update`]()        | 更新一个或多个文档。               |

### 查询计划缓存命令

| 名称                           | 描述                                 |
| ------------------------------ | ------------------------------------ |
| [`planCacheClear`]()           | 删除集合的缓存查询计划。             |
| [`planCacheClearFilters`]()    | 清除集合的索引过滤器。               |
| [`planCacheListFilters`]()     | 列出集合的索引过滤器。               |
| [`planCacheListPlans`]()       | 显示指定查询模型的缓存查询计划。     |
| [`planCacheListQueryShapes`]() | 显示存在其缓存的查询计划的查询模型。 |
| [`planCacheSetFilter`]()       | 为集合设置索引过滤器。               |

## <span id="database-operations">数据库操作</span>

### 认证命令

| 名称               | 描述                                                 |
| ------------------ | ---------------------------------------------------- |
| [`authenticate`]() | 使用用户名和密码启动经过身份验证的会话。             |
| [`getnonce`]()     | 这是一个内部命令，用于生成用于身份验证的一次性密码。 |
| [`logout`]()       | 终止当前已认证的会话。                               |

### 用户管理命令

| 名称                           | 描述                         |
| ------------------------------ | ---------------------------- |
| [`createUser`]()               | 创建一个新用户。             |
| [`dropAllUsersFromDatabase`]() | 删除与数据库关联的所有用户。 |
| [`dropUser`]()                 | 删除一个用户。               |
| [`grantRolesToUser`]()         | 向用户授予角色及其特权。     |
| [`revokeRolesFromUser`]()      | 从用户删除角色。             |
| [`updateUser`]()               | 更新用户的数据。             |
| [`usersInfo`]()                | 返回有关指定用户的信息。     |

### 角色管理命令

| 名称                           | 描述                                           |
| ------------------------------ | ---------------------------------------------- |
| [`createRole`]()               | 创建一个角色并指定其特权。                     |
| [`dropRole`]()                 | 删除用户定义的角色。                           |
| [`dropAllRolesFromDatabase`]() | 从数据库中删除所有用户定义的角色。             |
| [`grantPrivilegesToRole`]()    | 将特权分配给用户定义的角色。                   |
| [`grantRolesToRole`]()         | 指定角色，用户定义的角色将从这些角色继承特权。 |
| [`invalidateUserCache`]()      | 刷新用户信息的内存缓存，包括凭据和角色。       |
| [`revokePrivilegesFromRole`]() | 从用户定义的角色中删除指定的特权。             |
| [`revokeRolesFromRole`]()      | 从用户定义的角色中删除指定的继承角色。         |
| [`rolesInfo`]()                | 返回指定角色的信息。                           |
| [`updateRole`]()               | 更新用户定义的角色。                           |

### 复制命令

| 名称                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| [`applyOps`]()                   | 应用于内部命令OPLOG条目到当前数据集。                        |
| [`isMaster`]()                   | 显示有关此成员在副本集中的角色的信息，包括它是否为主角色。   |
| [`replSetAbortPrimaryCatchUp`]() | 强制选择的主数据库中止同步（追赶），然后完成到主数据库的过渡。 |
| [`replSetFreeze`]()              | 防止当前成员在一段时间内寻求选举为主。                       |
| [`replSetGetConfig`]()           | 返回副本集的配置对象。                                       |
| [`replSetGetStatus`]()           | 返回报告副本集状态的文档。                                   |
| [`replSetInitiate`]()            | 初始化新的副本集。                                           |
| [`replSetMaintenance`]()         | 启用或禁用维护模式，该模式将辅助节点置于一种`RECOVERING`状态。 |
| [`replSetReconfig`]()            | 将新配置应用于现有副本集。                                   |
| [`replSetResizeOplog`]()         | 动态调整副本集成员的操作日志的大小。仅适用于WiredTiger存储引擎。 |
| [`replSetStepDown`]()            | 当前`primary`下台,成为一个`secondary`，迫使选举。            |
| [`replSetSyncFrom`]()            | 显式覆盖用于选择要复制的成员的默认逻辑。                     |

> **也可以看看**
>
> 有关复制的更多信息。

### 分片命令

| 名称                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| [`addShard`]()            | 添加一个分片到分片集群。                                     |
| [`addShardToZone`]()      | 将分片与`zone`关联。支持在分片群集中配置`zone`。             |
| [`balancerStart`]()       | 启动平衡器线程。                                             |
| [`balancerStatus`]()      | 返回有关平衡器状态的信息。                                   |
| [`balancerStop`]()        | 停止平衡器线程。                                             |
| [`checkShardingIndex`]()  | 验证分片键索引的内部命令。                                   |
| [`clearJumboFlag`]()      | 清除`jumbo`数据块的标志。                                    |
| [`cleanupOrphaned`]()     | 删除分片键值超出分片拥有的数据块范围之外的孤立数据。         |
| [`enableSharding`]()      | 在特定数据库上启用分片。                                     |
| [`flushRouterConfig`]()   | 强制`mongod`/ `mongos`实例更新其缓存的路由元数据。           |
| [`getShardMap`]()         | 报告分片群集状态的内部命令。                                 |
| [`getShardVersion`]()     | 返回配置服务器版本的内部命令。                               |
| [`isdbgrid`]()            | 验证进程是否是`mongos`。                                     |
| [`listShards`]()          | 返回已配置的分片列表。                                       |
| [`medianKey`]()           | 不推荐使用的内部命令。请参阅`splitVector`。                  |
| [`moveChunk`]()           | 在分片之间迁移块的内部命令。                                 |
| [`movePrimary`]()         | 从分片集群中删除分片时，重新分配主分片。                     |
| [`mergeChunks`]()         | 提供在单个分片上组合块的功能。                               |
| [`removeShard`]()         | 启动从分片群集中删除分片的进程。                             |
| [`removeShardFromZone`]() | 删除分片和`zone`之间的关联。支持在分片群集中配置`zone`。     |
| [`setShardVersion`]()     | 内部命令，用于设置配置服务器版本。                           |
| [`shardCollection`]()     | 启用集合的分片功能，从而可以对集合进行分片。                 |
| [`shardingState`]()       | 报告`mongod` 是否为分片群集的成员。                          |
| [`split`]()               | 创建一个新的块。                                             |
| [`splitChunk`]()          | 拆分块的内部命令。而是使用方法`sh.splitFind()`和`sh.splitAt()`。 |
| [`splitVector`]()         | 确定分割点的内部命令。                                       |
| [`unsetSharding`]()       | 影响MongoDB部署中实例之间的连接的内部命令。                  |
| [`updateZoneKeyRange`]()  | 添加或删除范围内的分片数据与`zone`之间的关联。支持在分片群集中配置`zone`。 |

> **也可以看看**
>
> 有关MongoDB的分片功能的更多信息。

### 会话命令

| 指令                           | 描述                                                |
| ------------------------------ | --------------------------------------------------- |
| [`abortTransaction`]()         | 中止事务<br />*版本4.0中的新功能。*                 |
| [`commitTransaction`]()        | 提交事务<br />*版本4.0中的新功能。*                 |
| [`endSessions`]()              | 在会话超时期限之前终止会话。<br />*3.6版的新功能。* |
| [`killAllSessions`]()          | 杀死所有会话。<br />*3.6版的新功能。*               |
| [`killAllSessionsByPattern`]() | 杀死所有与指定模式匹配的会话<br />*3.6版的新功能。* |
| [`killSessions`]()             | 杀死指定的会话。<br />*3.6版的新功能。*             |
| [`refreshSessions`]()          | 刷新空闲会话。<br />*3.6版的新功能。*               |
| [`startSession`]()             | 开始新的会话。<br />*3.6版的新功能。*               |

### 管理命令

| 名称                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| [`clean`]()                          | 内部名称空间管理命令。                                       |
| [`cloneCollection`]()                | 将集合从远程主机复制到当前主机。                             |
| [`cloneCollectionAsCapped`]()        | 将未设置上限的集合复制为新的设置上限的集合。                 |
| [`collMod`]()                        | 向集合添加选项或修改视图定义。                               |
| [`compact`]()                        | 对集合进行分片整理并重建索引。                               |
| [`connPoolSync`]()                   | 用于刷新连接池的内部命令。                                   |
| [`convertToCapped`]()                | 将无上限的集合转换为有上限的集合。                           |
| [`create`]()                         | 创建一个集合或视图。                                         |
| [`createIndexes`]()                  | 为一个集合构建一个或多个索引。                               |
| [`currentOp`]()                      | 返回一个文档，该文档包含有关数据库实例正在进行的操作的信息。 |
| [`drop`]()                           | 从数据库中删除指定的集合。                                   |
| [`dropDatabase`]()                   | 删除当前数据库。                                             |
| [`dropConnections`]()                | 将外向连接删除到指定的主机列表。                             |
| [`dropIndexes`]()                    | 从集合中删除索引。                                           |
| [`filemd5`]()                        | 返回使用GridFS存储的文件的md5哈希值。                        |
| [`fsync`]()                          | 将挂起的写入刷新到存储层，并锁定数据库以允许备份。           |
| [`fsyncUnlock`]()                    | 解锁一个fsync锁。                                            |
| [`getParameter`]()                   | 检索配置选项。                                               |
| [`killCursors`]()                    | 杀死集合的指定游标。                                         |
| [`killOp`]()                         | 终止操作ID指定的操作。                                       |
| [`listCollections`]()                | 返回当前数据库中的集合列表。                                 |
| [`listDatabases`]()                  | 返回列出所有数据库的文档，并返回基本数据库统计信息。         |
| [`listIndexes`]()                    | 列出集合的所有索引。                                         |
| [`logRotate`]()                      | 循环MongoDB日志，以防止单个文件占用过多空间。                |
| [`reIndex`]()                        | 重建集合上的所有索引。                                       |
| [`renameCollection`]()               | 更改现有集合的名称。                                         |
| [`setFeatureCompatibilityVersion`]() | 启用或禁用保留向后不兼容的数据的功能。                       |
| [`setParameter`]()                   | 修改配置选项。                                               |
| [`shutdown`]()                       | 关闭`mongod`或`mongos`进程。                                 |

### 诊断命令

| 名称                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| [`availableQueryOptions`]() | 内部命令，报告当前MongoDB实例的功能。                        |
| [`buildInfo`]()             | 显示有关MongoDB构建的统计信息。                              |
| [`collStats`]()             | 报告指定集合的存储利用率静态信息。                           |
| [`connPoolStats`]()         | 报告从此MongoDB实例到部署中其他MongoDB实例的传出连接的统计信息。 |
| [`connectionStatus`]()      | 报告当前连接的身份验证状态。                                 |
| [`cursorInfo`]()            | 在MongoDB 3.2中已删除。替换为`metrics.cursor`。              |
| [`dataSize`]()              | 返回数据范围的数据大小。供内部使用。                         |
| [`dbHash`]()                | 返回数据库及其集合的哈希值。                                 |
| [`dbStats`]()               | 报告指定数据库的存储利用率统计信息。                         |
| [`diagLogging`]()           | 在MongoDB 3.6中已删除。要捕获，重放和分析发送到您的MongoDB部署的命令，请使用`mongoreplay`。 |
| [`driverOIDTest`]()         | 将ObjectId转换为字符串以支持测试的内部命令。                 |
| [`explain`]()               | 返回有关各种操作执行的信息。                                 |
| [`features`]()              | 报告当前MongoDB实例中可用的功能。                            |
| [`getCmdLineOpts`]()        | 返回带有MongoDB实例及其解析选项的运行时参数的文档。          |
| [`getLog`]()                | 返回最近的日志消息。                                         |
| [`hostInfo`]()              | 返回反映基础主机系统的数据。                                 |
| [isSelf]()                  | 内部命令支持测试。                                           |
| [`listCommands`]()          | 列出当前`mongod`实例提供的所有数据库命令。                   |
| [`lockInfo`]()              | 内部命令，返回有关当前正在保留或挂起的锁的信息。仅适用于 `mongod`实例。 |
| [`netstat`]()               | 报告部署内连接性的内部命令。仅适用于`mongos`实例。           |
| [`ping`]()                  | 测试部署内连接性的内部命令。                                 |
| [`profile`]()               | 数据库事件探查器的接口。                                     |
| [`serverStatus`]()          | 返回有关实例范围的资源利用率和状态的集合指标。               |
| [`shardConnPoolStats`]()    | 报告`mongos`连接池上的统计信息，以供客户端针对分片进行操作。 |
| [`top`]()                   | 返回`mongod`实例中每个数据库的原始使用情况统计信息。         |
| [`validate`]()              | 内部命令，用于扫描集合的数据并为正确性编制索引。             |
| [`whatsmyuri`]()            | 返回有关当前客户端信息的内部命令。                           |

### 免费监控命令

| 名称                    | 描述                        |
| ----------------------- | --------------------------- |
| [`setFreeMonitoring`]() | 在运行时启用/禁用免费监视。 |

## <span id="auditing-commands">审核命令</span>

| 名称                        | 描述                         |
| --------------------------- | ---------------------------- |
| [`logApplicationMessage`]() | 将自定义消息发布到审核日志。 |



译者：李冠飞

校对：