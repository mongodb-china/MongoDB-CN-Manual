## 在分片群集上建立滚动索引

**在本页面**

- [注意事项](#注意)
- [前提条件](#条件)
- [程序](#程序)
- [附加信息](#附加)

索引构建会影响分片集群的性能。默认情况下，MongoDB 4.4及以后版本在所有承载数据的复制集成员上同时构建索引。基于分片集群的索引仅发生在那些包含被索引的集合数据的分片上。对于不能容忍由于索引构建而导致性能下降的工作负载，可以考虑使用以下过程以滚动方式构建索引。

滚动索引构建一次最多取出一个碎片复制集成员(从辅助成员开始)，并在该成员上作为一个独立的成员构建索引。构建滚动索引需要每个碎片至少进行一次复制集选择。

### <span id="注意">注意事项</span>

#### 唯一索引

要使用以下过程创建[唯一索引](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)，必须在此过程中停止对集合的所有写操作。

如果在此过程中无法停止对集合的所有写操作，请不要使用此页面上的过程。相反，可以通过[`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)在分片[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)群集上发出来在集合上构建唯一索引。

#### Oplog大小

确保您的[oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)足够大，以允许索引或重新索引操作完成，而不会落后太多而无法跟上。参见[oplog sizing](https://docs.mongodb.com/master/core/replica-set-oplog/#replica-set-oplog-sizing)文档了解更多信息。

### <span id="条件">前提条件</span>

- 用于构建唯一索引

  1.要使用以下过程创建唯一索引，必须在索引生成期间停止对集合的所有写操作。否则，复制集成员之间的数据可能会不一致。如果	不能停止对集合的所有写操作，请不要使用以下过程创建唯一索引。
  
  > <p color="red">警告</p>
  >
  > 如果不能停止对集合的所有写操作，请不要使用以下过程创建唯一索引。
  
  2.在创建索引之前，验证集合中没有文档违反索引约束。如果一个集合分布在多个切片上，而一个切片中包含有重复文档的块，那么	创建索引操作可能在没有重复的切片上成功，但在有重复的切片上失败。为了避免在多个碎片之间留下不一致的索引，可以从			**mongos**中发出[`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)来从集合中删除索引。

### 程序

> 重要
>
> 以下以滚动方式构建索引的过程适用于分片集群部署，而不适用于复制集部署。关于复制集的过程，请参见复制集上的滚动索引构建。

#### A. 停止平衡器

将[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell连接到分片[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 群集中的实例，然后运行[`sh.stopBalancer()`](https://docs.mongodb.com/master/reference/method/sh.stopBalancer/#sh.stopBalancer)以禁用平衡器:

```powershell
sh.stopBalancer()
```

> 注意
>
> 如果迁移正在进行中，系统将在停止平衡器之前完成迁移。

要验证均衡器被禁用，运行[sh.getBalancerState()](https://docs.mongodb.com/master/reference/method/sh.getbalancstate/ #sh.getBalancerState)，如果均衡器被禁用，将返回**false**:

```powershell
sh.getBalancerState()
```

#### B. 确定集合的分布

在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell连接程序 [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)，刷新缓存的路由表， [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)以免返回该集合的陈旧分发信息。刷新后，运行 [`db.collection.getShardDistribution()`](https://docs.mongodb.com/master/reference/method/db.collection.getShardDistribution/#db.collection.getShardDistribution)要构建索引的集合。

例如，如果您想在`test` 数据库的`records`集合上使用升序索引:

```powershell
db.adminCommand( { flushRouterConfig: "test.records" } );
db.records.getShardDistribution();
```

该方法输出切分分布。例如，考虑一个分片集群，有3个分片**' shardA '**、**' shardB '**和**' shardC '**， [' db.collection.getShardDistribution() '](https://docs.mongodb.com/master/reference/method/db.collection.getShardDistribution/#db.collection.getShardDistribution)返回以下结果:

```powershell
Shard shardA at shardA/s1-mongo1.example.net:27018,s1-mongo2.example.net:27018,s1-mongo3.example.net:27018
 data : 1KiB docs : 50 chunks : 1
 estimated data per chunk : 1KiB
 estimated docs per chunk : 50

Shard shardC at shardC/s3-mongo1.example.net:27018,s3-mongo2.example.net:27018,s3-mongo3.example.net:27018
 data : 1KiB docs : 50 chunks : 1
 estimated data per chunk : 1KiB
 estimated docs per chunk : 50

Totals
 data : 3KiB docs : 100 chunks : 2
 Shard shardA contains 50% data, 50% docs in cluster, avg obj size on shard : 40B
 Shard shardC contains 50% data, 50% docs in cluster, avg obj size on shard : 40B
```

从输出中，您只为`test`构建索引。记录在`shardA `和` shardC `。

#### C. 在包含集合块的碎片上构建索引

对于包含集合块的每个分片，遵循以下过程在分片上构建索引。

##### C1. 停止一个辅助设备并独立重启

对于受影响的分片，停止[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)与其辅助节点之一相关联的过程。在进行以下配置更新后重新启动:

- 配置文件

如果您正在使用配置文件，请进行以下配置更新:

- 将[`net.port`](https://docs.mongodb.com/master/reference/configuration-options/#net.port)更改为其他端口。记下原始端口设置作为注释。
- 注释掉该[`replication.replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName)选项。
- 注释掉该[`sharding.clusterRole`](https://docs.mongodb.com/master/reference/configuration-options/#sharding.clusterRole)选项。
- 在部分[`skipShardingConfigurationChecks`](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks) 中将参数设置（也适用于MongoDB 3.6.3 +，3.4.11 +，3.2.19 +） `true`[`setParameter`](https://docs.mongodb.com/master/reference/privilege-actions/#setParameter)
- 在设置参数部分将参数**disableLogicalSessionCacheRefresh**设置为**true**。

例如，对于一个分片复制集成员，更新后的配置文件将包括如下示例所示的内容:

```powershell
net:
   bindIp: localhost,<hostname(s)|ip address(es)>
   port: 27218
#   port: 27018
#replication:
#   replSetName: shardA
#sharding:
#   clusterRole: shardsvr
setParameter:
   skipShardingConfigurationChecks: true
   disableLogicalSessionCacheRefresh: true
```

并重新启动:

```powershell
mongod --config <path/To/ConfigFile>
```

其他设置(例如[' storage.dbPath '](https://docs.mongodb.com/master/reference/configuring-options/#storage.dbpath)等)保持不变。



* 命令行选项

如果使用命令行选项，请进行以下配置更新:

- 修改[`--port`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-port)为其他端口。
- 删除[`--replSet`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-replset)。
- [`--shardsvr`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-shardsvr)如果分片成员和[`--configsvr`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configsvr)配置服务器成员则删除。
- 在[`--setParameter`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-setparameter)选项中将参数[`skipShardingConfigurationChecks`](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks) (也可用于MongoDB 3.6.3+、3.4.11+、3.2.19+)设置为**true**。
- 在 [`--setParameter`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-setparameter)选项中设置参数**disableLogicalSessionCacheRefresh**为**true**。

例如，重新启动不带[`--replSet`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-replset)和 [`--shardsvr`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-shardsvr)选项的分片副本集成员。指定新的端口号，并将[`skipShardingConfigurationChecks`](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks)和 `disableLogicalSessionCacheRefresh`参数都设置 为true：

```powershell
mongod --port 27218 --setParameter skipShardingConfigurationChecks=true --setParameter disableLogicalSessionCacheRefresh=true
```

其他设置（例如[`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-dbpath)等）保持不变。

##### C2. 建立索引

直接连接到[' mongod '](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例作为一个独立的运行在新的端口上，并为这个实例创建新的索引。

例如，将[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell连接到实例，并使用[`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法`username`在`records` 集合的字段上创建升序索引：

```powershell
db.records.createIndex( { username: 1 } )
```

##### C3. 重新启动程序 mongod 作为复制集成员

当索引构建完成时，关闭[' mongod '](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例。撤销作为独立启动时所做的配置更改，以返回原始配置并重新启动。

> 重要
>
> 一定要删除[' skipShardingConfigurationChecks '](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks)参数和' **disableLogicalSessionCacheRefresh '**参数。

例如，重新启动你的复制集分片成员:

- 配置文件

  

如果您正在使用配置文件，请进行以下配置更新:

- 恢复为原始端口号。
- 取消注释[`replication.replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName)。
- 取消注释[`sharding.clusterRole`](https://docs.mongodb.com/master/reference/configuration-options/#sharding.clusterRole)。
- [`skipShardingConfigurationChecks`](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks) 在该[`setParameter`](https://docs.mongodb.com/master/reference/privilege-actions/#setParameter)部分中删除参数。
- `disableLogicalSessionCacheRefresh` 在该[`setParameter`](https://docs.mongodb.com/master/reference/privilege-actions/#setParameter)部分中删除参数。

```powershell
net:
   bindIp: localhost,<hostname(s)|ip address(es)>
   port: 27018
replication:
   replSetName: shardA
sharding:
   clusterRole: shardsvr
```

其他设置(例如[' storage.dbPath '](https://docs.mongodb.com/master/reference/configuring-options/#storage.dbpath)等)保持不变。

并重新启动：

```powershell
mongod --config <path/To/ConfigFile>
```



* 命令行选项

如果使用命令行选项，请进行以下配置更新:

- 恢复为原始端口号。
- 包括[`--replSet`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-replset)。
- 包括分片[`--shardsvr`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-shardsvr)成员或[`--configsvr`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configsvr)配置服务器成员。
- 删除参数 [`skipShardingConfigurationChecks`](https://docs.mongodb.com/master/reference/parameters/#param.skipShardingConfigurationChecks)。
- 删除参数`disableLogicalSessionCacheRefresh`。

例如：

```powershell
mongod --port 27018 --replSet shardA --shardsvr
```

其他设置（例如[`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-dbpath)等）保持不变。

##### C4. 对分片的其他次要数据重复此过程

一旦该成员赶上了集合中的其他成员，就对分片中剩余的次要成员一次重复这个过程:

1. [C1.停止一台备用服务器并独立启动](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-stop-one-member)
2. [C2.建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-build-index)
3. [C3.重新启动程序mongod作为副本集成员](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-restart-mongod)

##### C5. 在主服务器上构建索引

当分片的所有辅助数据库都具有新索引时，请降低分片的主数据库，使用上述步骤以独立方式重新启动它，然后在前一个主数据库上建立索引:

1. 使用外壳程序中的[`rs.stepDown()`](https://docs.mongodb.com/master/reference/method/rs.stepDown/#rs.stepDown)方法[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)降低主数据库的性能。成功降级后，当前的主节点将成为辅助节点，复制集成员将选择新的主节点。
2. [C1.停止一台备用服务器并独立启动](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-stop-one-member)
3. [C2.建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-build-index)
4. [C3.重新启动程序mongod作为副本集成员](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-sharded-clusters-restart-mongod)

#### D. 对其他受影响的分片重复此操作

在为切分构建完索引之后，重复[C]。在包含集合块的切分上为其他受影响的切分构建索引](https://docs.mongodb.com/master/tutorial/build-indexes-on- shard-clusters/ #tutorial-index-on-affected-shards)。

一旦完成了为分片[建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-affected-shards)，请重复步骤 [C .在包含集合块的碎片上构建索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-affected-shards)为其他受影响的分片[建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/#tutorial-index-on-affected-shards)。

#### E.重新启动平衡器

为受影响的分片完成滚动索引构建后，重新启动平衡器。

将[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell连接到分片[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 群集中的实例，然后运行[`sh.startBalancer()`](https://docs.mongodb.com/master/reference/method/sh.startBalancer/#sh.startBalancer)：

```powershell
sh.startBalancer()
```

### <span id="附加">附加信息</span>

如果在包含集合块的每个分片上没有完全相同的索引(包括索引选项)，则分片集合具有不一致的索引。虽然在正常操作中不应该出现索引不一致的情况，但也会出现索引不一致的情况，例如:

- 当用户正在创建一个索引，一个“唯一”的关键约束和一个分片包含块与重复的文档。在这种情况下，创建索引操作可能在没有重复的分片上成功，但在有重复的切分上失败。
- 当用户以滚动方式在多个切片之间创建索引，但要么未能为关联的切片建立索引，要么不正确地建立了不同规格的索引。

从MongoDB 4.4（和4.2.6）开始，[配置服务器](https://docs.mongodb.com/master/core/sharded-cluster-config-servers/)主[服务器会](https://docs.mongodb.com/master/core/sharded-cluster-config-servers/)定期检查分片集合中各分片之间的索引不一致。要配置这些定期检查，请参阅 [`enableShardedIndexConsistencyCheck`](https://docs.mongodb.com/master/reference/parameters/#param.enableShardedIndexConsistencyCheck)和 [`shardedIndexConsistencyCheckIntervalMS`](https://docs.mongodb.com/master/reference/parameters/#param.shardedIndexConsistencyCheckIntervalMS)。

当在配置服务器主服务器上运行时，该命令[`serverStatus`](https://docs.mongodb.com/master/reference/command/serverStatus/#dbcmd.serverStatus)返回该字段 [`shardedIndexConsistency`](https://docs.mongodb.com/master/reference/command/serverStatus/#serverstatus.shardedIndexConsistency)以报告索引不一致情况。

要检查分片集合是否具有不一致的索引，请参阅 [查找分片中的不一致索引](https://docs.mongodb.com/master/tutorial/manage-indexes/#manage-indexes-find-inconsistent-indexes)。