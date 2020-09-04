# [ ](#)mongo Shell 方法

[]()

在本页面

*   [集合](#collection)
*   [游标](#cursor)
*   [数据库](#database)
*   [查询计划缓存](#query-plan-cache)
*   [批量写入操作](#bulk-write-operation)
*   [用户管理](#user-management)
*   [角色管理](#role-management)
*   [复制](#replication)
*   [分片](#sharding)
*   [Free监控](#free-monitoring)
*   [构造函数](#constructors)
*   [连接](#connection)
*   [本机](#native)
*   [客户端字段级加密](#client-side-field-level-encryption)
> **MONGODB 中的 JAVASCRIPT**
> 
> 虽然这些方法使用 JavaScript，但大多数与 MongoDB 的交互都不使用 JavaScript，而是在交互 application 的语言中使用惯用的司机。



> **注意**
> 
> 有关特定方法(包括语法和示例)的详细信息，请单击特定方法以转到其 reference 页面。

## <span id="collection">集合</span>

| 名称                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| db.collection.aggregate()            | 提供对聚合管道的访问。                                       |
| db.collection.bulkWrite()            | 提供批量写入操作功能。                                       |
| db.collection.copyTo()               | 已过时。包装EVAL以在单个 MongoDB 实例中的集合之间复制数据。  |
| db.collection.count()                | 包装计数以_return 计算集合或视图中的文档数。                 |
| db.collection.createIndex()          | 在集合上构建索引。                                           |
| db.collection.createIndexes()        | 在集合上构建一个或多个索引。                                 |
| db.collection.dataSize()             | 返回集合的大小。包装collStats输出中的尺寸字段。              |
| db.collection.deleteOne()            | 删除集合中的单个文档。                                       |
| db.collection.deleteMany()           | 删除集合中的多个文档。                                       |
| db.collection.distinct()             | 返回具有指定字段的不同值的文档的 array。                     |
| db.collection.drop()                 | 从数据库中删除指定的集合。                                   |
| db.collection.dropIndex()            | 删除集合上的指定索引。                                       |
| db.collection.dropIndexes()          | 删除集合上的所有索引。                                       |
| db.collection.ensureIndex()          | 已过时。使用db.collection.createIndex()。                    |
| db.collection.explain()              | 返回有关各种方法的查询执行的信息。                           |
| db.collection.find()                 | 对集合或视图执行查询并返回游标 object。                      |
| db.collection.findAndModify()        | 以原子方式修改并返回单个文档。                               |
| db.collection.findOne()              | 执行查询并返回单个文档。                                     |
| db.collection.findOneAndDelete()     | 查找单个文档并将其删除。                                     |
| db.collection.findOneAndReplace()    | 查找单个文档并替换它。                                       |
| db.collection.findOneAndUpdate()     | 查找单个文档并进行更新。                                     |
| db.collection.getIndexes()           | 返回描述集合上现有索引的文档的 array。                       |
| db.collection.getShardDistribution() | 对于分片群集中的集合，db.collection.getShardDistribution()报告块分布的数据。 |
| db.collection.getShardVersion()      | 分片 cluster 的内部诊断方法。                                |
| db.collection.group()                | 已过时。提供简单的数据聚合 function。通过 key 对集合中的文档进行分组，并处理结果。使用aggregate()进行更复杂的数据聚合。 |
| db.collection.insert()               | 在集合中创建新文档。                                         |
| db.collection.insertOne()            | 在集合中插入新文档。                                         |
| db.collection.insertMany()           | 在集合中插入几个新文档。                                     |
| db.collection.isCapped()             | 报告集合是否为上限集合。                                     |
| db.collection.latencyStats()         | 返回集合的延迟统计信息。                                     |
| db.collection.mapReduce()            | 执行 map-reduce 样式数据聚合。                               |
| db.collection.reIndex()              | 重建集合上的所有现有索引。                                   |
| db.collection.remove()               | 从集合中删除文档。                                           |
| db.collection.renameCollection()     | 更改集合的 name。                                            |
| db.collection.replaceOne()           | 替换集合中的单个文档。                                       |
| db.collection.save()                 | 在insert()和update()周围提供 wrapper 以插入新文档。          |
| db.collection.stats()                | 报告集合的 state。在collStats周围提供 wrapper。              |
| db.collection.storageSize()          | 报告集合使用的总大小(以字节为单位)。在collStats输出的storageSize字段周围提供 wrapper。 |
| db.collection.totalIndexSize()       | 报告集合上索引使用的总大小。在collStats输出的totalIndexSize字段周围提供 wrapper。 |
| db.collection.totalSize()            | 报告集合的总大小，包括所有文档的大小和集合上的所有索引。     |
| db.collection.update()               | 修改集合中的文档。                                           |
| db.collection.updateOne()            | 修改集合中的单个文档。                                       |
| db.collection.updateMany()           | 修改集合中的多个文档。                                       |
| db.collection.watch()                | 在集合上建立变更流。                                         |
| db.collection.validate()             | 对集合执行诊断操作。                                         |

## <span id="cursor">游标</span>

| 名称                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| cursor.addOption()       | 添加特殊的线程协议标志，用于修改查询的行为。                 |
| cursor.batchSize()       | 控制 MongoDB 将在单个网络消息中 return 到 client 的文档数。  |
| cursor.close()           | 关闭游标并释放相关的服务器资源。                             |
| cursor.isClosed()        | 如果光标关闭，则返回`true`。                                 |
| cursor.collation()       | 指定db.collection.find()返回的游标的排序规则。               |
| cursor.comment()         | 将 comment 附加到查询以允许日志和 system.profile 集合中的可跟踪性。 |
| cursor.count()           | 修改光标以_return 结果集中的文档数而不是文档本身。           |
| cursor.explain()         | 报告游标的查询执行计划。                                     |
| cursor.forEach()         | 对游标中的每个文档应用 JavaScript function。                 |
| cursor.hasNext()         | 如果游标有文档并且可以迭代，则返回 true。                    |
| cursor.hint()            | 强制 MongoDB 为查询使用特定索引。                            |
| cursor.isExhausted()     | 如果光标关闭且批处理中没有剩余 object，则返回`true`。        |
| cursor.itcount()         | 通过获取和迭代结果集来计算游标 client-side 中的文档总数。    |
| cursor.limit()           | 约束游标结果集的大小。                                       |
| cursor.map()             | 对函数中的每个文档应用 function，并在 array 中收集 return 值。 |
| cursor.max()             | 指定游标的独占上限索引。用于cursor.hint()。                  |
| cursor.maxScan()         | 指定要扫描的最大项目数;收集扫描的文档，索引扫描的键。        |
| cursor.maxTimeMS()       | 指定用于处理游标操作的累积 time 限制(以毫秒为单位)。         |
| cursor.min()             | 指定游标的包含性较低索引范围。用于cursor.hint()              |
| cursor.next()            | 返回游标中的下一个文档。                                     |
| cursor.noCursorTimeout() | 指示服务器在一段时间不活动后自动关闭光标。                   |
| cursor.objsLeftInBatch() | 返回当前游标批处理中剩余的文档数。                           |
| cursor.pretty()          | 配置光标以 easy-to-read 格式显示结果。                       |
| cursor.readConcern()     | 为find()操作指定阅读关注。                                   |
| cursor.readPref()        | 指定阅读偏好到游标以控制 client 如何将查询定向到副本集。     |
| cursor.returnKey()       | 将光标修改为 return 索引键而不是文档。                       |
| cursor.showRecordId()    | 向光标返回的每个文档添加内部存储引擎 ID 字段。               |
| cursor.size()            | 应用skip()和limit()方法后，返回游标中文档的计数。            |
| cursor.skip()            | 返回仅在传递或跳过多个文档后才开始返回结果的游标。           |
| cursor.sort()            | 返回根据排序规范排序的结果。                                 |
| cursor.tailable()        | 将光标标记为 tailable。仅适用于超过上限集合的游标。          |
| cursor.toArray()         | 返回包含游标返回的所有文档的 array。                         |

## <span id="database">数据库</span>

| 名称                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| db.adminCommand()              | 对`admin`数据库运行命令。                                    |
| db.aggregate()                 | 运行不需要底层集合的 admin/diagnostic 管道。                 |
| db.cloneCollection()           | 在 MongoDB 实例之间直接复制数据。包裹cloneCollection。       |
| db.cloneDatabase()             | 将数据库从 remote host 复制到当前 host。包裹克隆。           |
| db.commandHelp()               | 返回数据库命令的帮助信息。                                   |
| db.copyDatabase()              | 将数据库复制到当前 host 上的另一个数据库。包裹COPYDB。       |
| db.createCollection()          | 创建新集合或视图。常用于创建上限集合。                       |
| db.createView()                | 创建一个视图。                                               |
| db.currentOp()                 | 报告当前的 in-progress 操作。                                |
| db.dropDatabase()              | 删除当前数据库。                                             |
| db.eval()                      | 已过时。将 JavaScript function 传递给mongod实例 server-side JavaScript evaluation。 |
| db.fsyncLock()                 | 刷新写入磁盘并锁定数据库以防止写入操作并协助备份操作。包裹FSYNC。 |
| db.fsyncUnlock()               | 允许在使用db.fsyncLock()锁定的数据库上写入 continue。        |
| db.getCollection()             | 返回集合或视图 object。用于访问名称在mongo shell 中无效的集合。 |
| db.getCollectionInfos()        | 返回当前数据库中所有集合和视图的集合信息。                   |
| db.getCollectionNames()        | 列出当前数据库中的所有集合和视图。                           |
| db.getLastError()              | 检查并返回上一次操作的状态。包裹GetLastError 函数。          |
| db.getLastErrorObj()           | 返回上一个操作的状态文档。包裹GetLastError 函数。            |
| db.getLogComponents()          | 返回 log 消息详细级别。                                      |
| db.getMongo()                  | 返回当前连接的Mongo()连接 object。                           |
| db.getName()                   | 返回当前数据库的 name。                                      |
| db.getPrevError()              | 返回包含自上次错误重置以来的所有错误的状态文档。包裹getPrevError。 |
| db.getProfilingLevel()         | 返回数据库操作的当前分析 level。                             |
| db.getProfilingStatus()        | 返回反映当前性能分析 level 和性能分析阈值的文档。            |
| db.getReplicationInfo()        | 返回包含复制统计信息的文档。                                 |
| db.getSiblingDB()              | 提供对指定数据库的访问。                                     |
| db.help()                      | 显示 common `db` object 方法的说明。                         |
| db.hostInfo()                  | 返回一个文档，其中包含有关运行 MongoDB 的系统的信息。包裹Hostinfo 中。 |
| db.isMaster()                  | 返回报告副本集的 state 的文档。                              |
| db.killOp()                    | 终止指定的操作。                                             |
| db.listCommands()              | 显示 common 数据库命令的列表。                               |
| db.logout()                    | 结束经过身份验证的 session。                                 |
| db.printCollectionStats()      | 打印每个集合的统计信息。包裹db.collection.stats()。          |
| db.printReplicationInfo()      | 从主数据库的角度打印副本集状态的报告。                       |
| db.printShardingStatus()       | 打印分片配置和块范围的报告。                                 |
| db.printSlaveReplicationInfo() | 从辅助节点的角度打印副本集状态的报告。                       |
| db.repairDatabase()            | 在当前数据库上运行修复例程。                                 |
| db.resetError()                | 重置db.getPrevError()和getPrevError返回的错误消息。          |
| db.runCommand()                | 运行数据库命令。                                             |
| db.serverBuildInfo()           | 返回显示mongod实例的编译参数的文档。包装`buildinfo`。        |
| db.serverCmdLineOpts()         | 返回一个文档，其中包含有关用于启动 MongoDB 实例的运行时的信息。包裹getCmdLineOpts。 |
| db.serverStatus()              | 返回一个文档，该文档提供数据库 process 的 state 的概述。     |
| db.setLogLevel()               | 设置单个 log 消息详细程度 level。                            |
| db.setProfilingLevel()         | 修改数据库概要分析的当前 level。                             |
| db.shutdownServer()            | 干净安全地关闭当前的mongod或mongos process。                 |
| db.stats()                     | 返回报告当前数据库的 state 的文档。                          |
| db.version()                   | 返回mongod实例的 version。                                   |

## <span id="query-plan-cache">查询计划缓存</span>

| 名称                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| db.collection.getPlanCache()  | 返回一个接口，用于访问集合的查询计划缓存 object 和关联的 PlanCache 方法。 |
| PlanCache.clear()             | 清除集合的所有缓存查询计划。可通过特定集合的计划缓存 object 访问，即：`db.collection.getPlanCache().clear()`。 |
| PlanCache.clearPlansByQuery() | 清除指定查询形状的缓存查询计划。可通过特定集合的计划缓存 object 访问，即：`db.collection.getPlanCache().clearPlansByQuery()` |
| PlanCache.getPlansByQuery()   | 显示指定查询形状的缓存查询计划。可通过特定集合的计划缓存 object 访问，即：`db.collection.getPlanCache().getPlansByQuery()`。 |
| PlanCache.help()              | 显示集合的查询计划缓存可用的方法。可通过特定集合的计划缓存 object 访问，即：`db.collection.getPlanCache().help()`。 |
| PlanCache.listQueryShapes()   | 显示存在缓存查询计划的查询形状。可通过特定集合的计划缓存 object 访问，即：`db.collection.getPlanCache().listQueryShapes()`。 |

## <span id="bulk-write-operation">批量写入操作</span>

| 名称                                      | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| db.collection.initializeOrderedBulkOp()   | 为有序的操作列表初始化Bulk()操作构建器。                     |
| db.collection.initializeUnorderedBulkOp() | 为无序的操作列表初始化Bulk()操作构建器。                     |
| Bulk()                                    | 批量运营建设者。                                             |
| Bulk.execute()                            | 批量执行操作列表。                                           |
| Bulk.find()                               | 指定更新或删除操作的查询条件。                               |
| Bulk.find.arrayFilters()                  | 指定用于确定要为`update`或`updateOne`操作更新 array 的哪些元素的过滤器。 |
| Bulk.find.collation()                     | 指定查询条件的整理。                                         |
| Bulk.find.remove()                        | 将多个文档删除操作添加到操作列表中。                         |
| Bulk.find.removeOne()                     | 将单个文档删除操作添加到操作列表。                           |
| Bulk.find.replaceOne()                    | 将单个文档替换操作添加到操作列表中。                         |
| Bulk.find.updateOne()                     | 将单个文档更新操作添加到操作列表。                           |
| Bulk.find.update()                        | 将`multi`更新操作添加到操作列表中。                          |
| Bulk.find.upsert()                        | 为更新操作指定`upsert: true`。                               |
| Bulk.getOperations()                      | 返回Bulk() operations object 中执行的写操作的 array。        |
| Bulk.insert()                             | 将 Insert 操作添加到操作列表中。                             |
| Bulk.tojson()                             | 返回一个 JSON 文档，其中包含Bulk() operations object 中的操作数和批处理数。 |
| Bulk.toString()                           | 将Bulk.tojson()结果作为 string 返回。                        |

## <span id="user-management">用户管理</span>

| 名称                     | 描述                                   |
| ------------------------ | -------------------------------------- |
| db.auth()                | 将用户验证到数据库。                   |
| db.changeUserPassword()  | 更改现有用户的密码。                   |
| db.createUser()          | 创建一个新用户。                       |
| db.dropUser()            | 删除单个用户。                         |
| db.dropAllUsers()        | 删除与数据库关联的所有用户。           |
| db.getUser()             | 返回有关指定用户的信息。               |
| db.getUsers()            | 返回有关与数据库关联的所有用户的信息。 |
| db.grantRolesToUser()    | 向用户授予角色及其权限。               |
| db.removeUser()          | 已过时。从数据库中删除用户。           |
| db.revokeRolesFromUser() | 从用户中删除角色。                     |
| db.updateUser()          | 更新用户数据。                         |

## <span id="role-management">角色管理</span>

| 名称                          | 描述                                       |
| ----------------------------- | ------------------------------------------ |
| db.createRole()               | 创建角色并指定其权限。                     |
| db.dropRole()                 | 删除 user-defined 角色。                   |
| db.dropAllRoles()             | 删除与数据库关联的所有 user-defined 角色。 |
| db.getRole()                  | 返回指定角色的信息。                       |
| db.getRoles()                 | 返回数据库中所有 user-defined 角色的信息。 |
| db.grantPrivilegesToRole()    | 为 user-defined 角色分配权限。             |
| db.revokePrivilegesFromRole() | 从 user-defined 角色中删除指定的权限。     |
| db.grantRolesToRole()         | 指定 user-defined 角色从中继承权限的角色。 |
| db.revokeRolesFromRole()      | 从角色中删除继承的角色。                   |
| db.updateRole()               | 更新 user-defined 角色。                   |

## <span id="replication">复制</span>

| 名称                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| rs.add()                       | 将成员添加到副本集。                                         |
| rs.addArb()                    | 将仲裁者添加到副本集。                                       |
| rs.conf()                      | 返回副本集 configuration 文档。                              |
| rs.freeze()                    | 阻止当前成员在 time 期间寻求选举。                           |
| rs.help()                      | 返回副本集函数的基本帮助文本。                               |
| rs.initiate()                  | 初始化新的副本集。                                           |
| rs.printReplicationInfo()      | 从主数据库的角度打印副本集状态的报告。                       |
| rs.printSlaveReplicationInfo() | 从辅助节点的角度打印副本集状态的报告。                       |
| rs.reconfig()                  | Re-configures 通过应用新副本集 configuration object 设置副本。 |
| rs.remove()                    | 从副本集中删除成员。                                         |
| rs.slaveOk()                   | 为当前连接设置`slaveOk` property。已过时。使用readPref()和Mongo.setReadPref()设置阅读偏好。 |
| rs.status()                    | 返回包含有关副本集的 state 的信息的文档。                    |
| rs.stepDown()                  | 导致当前主成为强制选举的辅助。                               |
| rs.syncFrom()                  | 设置此副本集成员将同步的成员，覆盖默认同步目标选择逻辑。     |

## <span id="sharding">分片</span>

| 名称                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| sh.addShard()            | 将碎片添加到分片 cluster。                                   |
| sh.addShardTag()         | 在 MongoDB 3.4 中，此方法别名为sh.addShardToZone()。         |
| sh.addShardToZone()      | 将碎片与 zone 关联。支持在分片群集中配置zones。              |
| sh.addTagRange()         | 在 MongoDB 3.4 中，此方法别名为sh.updateZoneKeyRange()。     |
| sh.disableBalancing()    | 禁用分片数据库中单个集合的平衡。不影响分片 cluster 中其他集合的平衡。 |
| sh.enableBalancing()     | 如果以前使用sh.disableBalancing()禁用，则激活分片收集平衡器 process。 |
| sh.disableAutoSplit()    | 禁用分片 cluster 的 auto-splitting。                         |
| sh.enableAutoSplit()     | 为分片 cluster 启用 auto-splitting。                         |
| sh.enableSharding()      | 在特定数据库上启用分片。                                     |
| sh.getBalancerHost()     | 从 MongoDB 3.4 开始不推荐使用                                |
| sh.getBalancerState()    | 返回 boolean 以报告当前是否启用了平衡器。                    |
| sh.removeTagRange()      | 在 MongoDB 3.4 中，此方法别名为sh.removeRangeFromZone()。    |
| sh.removeRangeFromZone() | 删除一系列分片键和 zone 之间的关联。支持在分片群集中配置zones。 |
| sh.help()                | 返回`sh`方法的帮助文本。                                     |
| sh.isBalancerRunning()   | 返回 boolean 以报告 balancer process 当前是否正在迁移块。    |
| sh.moveChunk()           | 在分片 cluster中迁移块。                                     |
| sh.removeShardTag()      | 在 MongoDB 3.4 中，此方法别名为sh.removeShardFromZone()。    |
| sh.removeShardFromZone() | 删除分片和 zone 之间的关联。用于管理zone 分片。              |
| sh.setBalancerState()    | 启用或禁用在碎片之间迁移块的平衡器。                         |
| sh.shardCollection()     | 为集合启用分片。                                             |
| sh.splitAt()             | 使用碎片 key的特定值作为分割点将现有的块分成两个块。         |
| sh.splitFind()           | 将包含与查询匹配的文档的现有块划分为两个近似相等的块。       |
| sh.startBalancer()       | 启用平衡器并等待平衡启动。                                   |
| sh.status()              | 报告分片 cluster的状态，如db.printShardingStatus()。         |
| sh.stopBalancer()        | 禁用平衡器并等待任何正在进行的平衡轮完成。                   |
| sh.waitForBalancer()     | 内部。等待平衡器 state 改变。                                |
| sh.waitForBalancerOff()  | 内部。等到平衡器停止运行。                                   |
| sh.waitForPingChange()   | 内部。等待从分片 cluster 中的mongos之一更改 ping state。     |
| sh.updateZoneKeyRange()  | 将一系列分片键与 zone 关联。支持在分片群集中配置zones。      |

## <span id="free-monitoring">Free监控</span>

| 名称                         | 描述                   |
| ---------------------------- | ---------------------- |
| db.enableFreeMonitoring()    | 在运行时启用Free监控。 |
| db.disableFreeMonitoring()   | 在运行时禁用Free监控。 |
| db.getFreeMonitoringStatus() | 返回空闲监视状态。     |

## <span id="constructors">构造函数</span>

| 名称                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| BulkWriteResult()                  | Wrapper 来自Bulk.execute()的结果集。                         |
| Date()                             | 创建 date object。默认情况下，创建包含当前 date 的 date object。 |
| ObjectId()                         | 返回ObjectId。                                               |
| ObjectId.getTimestamp()            | 返回ObjectId的时间戳部分。                                   |
| ObjectId.toString()                | 显示ObjectId的 string 表示。                                 |
| ObjectId.valueOf()                 | 将 ObjectId 的`str`属性显示为十六进制 string。               |
| UUID()                             | 将 32-byte 十六进制 string 转换为 UUID BSON 子类型。         |
| WriteResult()                      | Wrapper 来自 write 方法的结果集。                            |
| WriteResult.hasWriteError()        | 返回一个 boolean，指定结果是否包含WriteResult.writeError。   |
| WriteResult.hasWriteConcernError() | 返回一个 boolean，指定结果是否包含WriteResult.writeConcernError。 |

## <span id="connection">连接</span>

| 名称                         | 描述                                        |
| ---------------------------- | ------------------------------------------- |
| connect()                    | 连接到 MongoDB 实例和该实例上的指定数据库。 |
| Mongo()                      | 创建一个新连接 object。                     |
| Mongo.getDB()                | 返回数据库 object。                         |
| Mongo.getReadPrefMode()      | 返回 MongoDB 连接的当前读取首选项模式。     |
| Mongo.getReadPrefTagSet()    | 返回 MongoDB 连接的读取首选项标记集。       |
| Mongo.isCausalConsistency()  | 指示是否在连接 object 上启用了因果一致性。  |
| Mongo.setCausalConsistency() | 启用或禁用连接 object 上的因果一致性。      |
| Mongo.setReadPref()          | 为 MongoDB 连接设置阅读偏好。               |
| Mongo.setSlaveOk()           | 允许当前连接上的操作从次要成员读取。        |
| Mongo.startSession()         | 在连接 object 上启动 session。              |
| session                      | session object。                            |
| SessionOptions               | 选项 object 为 session。                    |

## <span id="native">本机</span>

| 名称              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| cat()             | 返回指定文件的内容。                                         |
| cd()              | 将当前工作目录更改为指定的路径。                             |
| copyDbpath()      | 复制本地DBPATH。供内部使用。                                 |
| fuzzFile()        | 供内部使用以支持测试。                                       |
| getHostName()     | 返回系统的主机名运行mongo shell。                            |
| getMemInfo()      | 返回报告 shell 使用的 memory 数量的文档。                    |
| hostname()        | 返回系统的主机名运行 shell。                                 |
| listFiles()       | 返回给出目录中每个 object 的 name 和大小的文档的 array。     |
| load()            | 在 shell 中加载并运行 JavaScript 文件。                      |
| ls()              | 返回当前目录中 files 的列表。                                |
| md5sumFile()      | 指定文件的MD5哈希值。                                        |
| mkdir()           | 在指定的路径创建目录。                                       |
| pwd()             | 返回当前目录。                                               |
| quit()            | 退出当前的 shell session。                                   |
| removeFile()      | 从本地文件系统中删除指定的文件。                             |
| resetDbpath()     | 删除本地DBPATH。供内部使用。                                 |
| sleep()           | 在给定的 time 期间暂停mongo shell。                          |
| setVerboseShell() | 配置mongo shell 以报告操作时间。                             |
| version()         | 返回mongo shell 实例的当前 version。                         |
| _isWindows()      | 如果 shell 在 Windows 系统上运行，则返回`true`; `false`如果是 Unix 或 Linux 系统。 |
| _rand()           | 返回`0`和`1`之间的随机数。                                   |
| _srand()          | 供内部使用。                                                 |

## <span id="client-side-field-level-encryption">客户端字段级加密</span>

> **注意**
>
> [`mongo`]()客户端的字段级的加密方法需要与客户端的字段级加密的数据库连接启用。如果当前数据库连接不是在启用客户端字段级加密的情况下启动的，则可以：
>
> - 使用shell程序中的[`Mongo()`]()构造函数[`mongo`]()与所需的客户端字段级加密选项建立连接。该[`Mongo()`]()方法同时支持Amazon Web Services和本地密钥管理服务（KMS）提供程序以进行客户主密钥（CMK）管理。
>
>   *要么*
>
> - 使用`mongo`shell [命令行选项]()与所需选项建立连接。命令行选项仅支持AWS KMS提供程序进行CMK管理。

| 名称                                    | 描述                                              |
| --------------------------------------- | ------------------------------------------------- |
| [`getKeyVault()`]()                     | 返回当前MongoDB连接的密钥保险库对象。             |
| [`KeyVault.createKey()`]()              | 创建用于客户端字段级加密的数据加密密钥。          |
| [`KeyVault.deleteKey()`]()              | 从密钥库中删除指定的数据加密密钥。                |
| [`KeyVault.getKey()`]()                 | 从密钥库中检索指定的数据加密密钥。                |
| [`KeyVault.getKeys()`]()                | 检索密钥库中的所有密钥。                          |
| [`KeyVault.addKeyAlternateName()`]()    | 将密钥替代名称与指定的数据加密密钥相关联。        |
| [`KeyVault.removeKeyAlternateName()`]() | 从指定的数据加密密钥中删除密钥替代名称。          |
| [`KeyVault.getKeyByAltName()`]()        | 检索具有指定键替代名称的键。                      |
| [`getClientEncryption()`]()             | 返回用于支持字段的显式加密/解密的客户端加密对象。 |
| [`ClientEncryption.encrypt()`]()        | 使用指定的数据加密密钥和加密算法对字段进行加密。  |
| [`ClientEncryption.decrypt()`]()        | 使用关联的数据加密密钥和加密算法解密字段。        |

