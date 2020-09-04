## 在复制集上建立索引

**在本页面**

- [注意事项](#注意)
- [前提条件](#条件)
- [程序](#程序)

索引构建会影响复制集的性能。默认情况下，MongoDB 4.4及以后版本在所有承载数据的复制集成员上同时构建索引。对于不能容忍由于索引构建而导致性能下降的工作负载，可以考虑使用以下过程以滚动方式构建索引。

滚动索引构建一次最多抽取一个复制集成员(从辅助成员开始)，并在该成员上作为独立的索引构建。构建滚动索引至少需要一次复制集的选择。

### <span id="注意">注意事项</span>

#### 唯一索引

要使用以下过程创建[唯一索引](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)，必须在此过程中停止对集合的所有写操作。

如果在此过程中不能停止对集合的所有写操作，请不要使用此页面上的过程。相反，通过在主节点上为一个副本集发出[' db.collection.createIndex() '](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)来在该集合上构建你的唯一索引。

#### Oplog大小

确保您的[oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)足够大，以允许索引或重新索引操作完成，而不会落后太多而无法跟上。参见[oplog sizing](https://docs.mongodb.com/master/core/replica-set-oplog/#replica-set-oplog-sizing)文档了解更多信息。

## <span id="条件">前提条件</span>

* 用于构建唯一索引

要使用以下过程创建[唯一索引](https://docs.mongodb.com/master/core/index-unique/#index-type-unique)，必须在索引构建期间停止对集合的所有写操作。否则，复制集成员之间的数据可能会不一致。如果不能停止对集合的所有写操作，请不要使用以下过程创建唯一索引。

### <span id="程序">程序</span>

> <p color="red">重要</p>
>
> 以下以滚动方式构建索引的过程适用于复制集部署，而不适用分片集群。有关分片集群的过程，请参阅[在分片集群上构建滚动索引](https://docs.mongodb.com/master/tutorial/build-indexes-onshard-clusters/)。



#### A. 停止一个辅助节点并作为独立节点重新启动

停止与辅助节点关联的mongod进程。进行以下配置更新后重新启动：

- 配置文件

如果您正在使用配置文件，请进行以下配置更新:

- 注释掉[replication.replSetName](https://docs.mongodb.com/master/reference/configuring-options/#replication.replsetname)选项。
- 更改[net.port](https://docs.mongodb.com/master/reference/configuring-options/#net.port)到一个不同的端口。[[1]](https://docs.mongodb.com/master/tutorial/build-indexes-onreplica-sets/#differing-port)记录原始的端口设置作为注释。
- 在[setParameter](https://docs.mongodb.com/master/reference/privilege-actions/#setParameter)部分设置参数' disablelogicalicalsessioncacherefresh '为' true '。

例如，更新后的副本集成员配置文件将包括如下示例所示的内容:

```powershell
net:
   bindIp: localhost,<hostname(s)|ip address(es)>
   port: 27217
#   port: 27017
#replication:
#   replSetName: myRepl
setParameter:
   disableLogicalSessionCacheRefresh: true
```

其他设置（例如[`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath)等）保持不变。

并重新启动:

```powershell
mongod --config <path/To/ConfigFile>
```

* 命令行选项

如果使用命令行选项，请进行以下配置更新:

* 删除---复制集。
* 修改---端口到另一个端口。
* 在---setParameter选项中设置参数**disableLogicalSessionCacheRefresh**为**true**

例如，如果你的复制集成员通常运行在默认端口**27017**和----replSet选项，你应该指定一个不同的端口，省略----replSet选项，并设置**disableLogicalSessionCacheRefresh**参数为**true**:

```powershell
mongod --port 27217 --setParameter disableLogicalSessionCacheRefresh=true
```

其他设置（例如[`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-dbpath)等）保持不变。

#### B. 建立索引

直接连接到[mongod](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例作为一个独立的运行在新的端口上，并为这个实例创建新的索引。

例如，将[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)连接到实例，然后使用[`createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)来`username`在`records`集合的字段上创建升序索引：

```powershell
db.records.createIndex( { username: 1 } )
```

### C. 重新启动程序mongod作为复制集成员

索引构建完成后，关闭[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 实例。撤消以独立版本启动时所做的配置更改，以返回其原始配置并以复制集的成员身份重新启动。

> 重要
>
> 一定要删除**' disableLogicalSessionCacheRefresh '**参数。

例如，重新启动复制集成员:

- 配置文件

如果您正在使用配置文件:

- 恢复到原始端口号。
- 取消[replication.replSetName](https://docs.mongodb.com/master/reference/configuring-options/#replication.replsetname)的注释。
- 删除[setParameter](https://docs.mongodb.com/master/reference/privilege-actions/#setParameter)中的参数**' disableLogicalSessionCacheRefresh '**。

例如：

copycopied

```powershell
net:
   bindIp: localhost,<hostname(s)|ip address(es)>
   port: 27017
replication:
   replSetName: myRepl
```

Other settings (e.g. [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath), etc.) remain the same.

并重新启动

```powershell
mongod --config <path/To/ConfigFile>
```

* 命令行选项

如果您正在使用配置文件:

* 恢复到原始端口号
* 包括----**replSet**选项。
* 删除参数**disableLogicalSessionCacheRefresh**。

例如：

```powershell
mongod --port 27017 --replSet myRepl
```

其他设置（例如[`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-dbpath)等）保持不变。

#### D.重复其余的步骤

一旦该成员赶上集合中的其他成员，请对其余的次要成员一次重复一个成员的过程：

1. [A.停止一个辅助节点并以独立方式重新启动](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-stop-one-member)
2. [B.建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-build-index)
3. [C.重新启动程序mongod作为副本集成员](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-restart-mongod)

#### E. 在主服务器上构建索引

当所有的辅助服务器都有了新的索引时，从主服务器下走一步，使用上面描述的过程作为一个独立的程序重新启动它，并在前主服务器上构建索引:

1. 使用`mongo shell`中的[`rs.stepDown()`](https://docs.mongodb.com/master/reference/method/rs.stepDown/#rs.stepDown)方法[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)降低主数据库的性能。成功降级后，当前的主节点将成为辅助节点，复制集成员将选择新的主节点。
2. [A.停止一个辅助节点并以独立方式重新启动](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-stop-one-member)
3. [B.建立索引](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-build-index)
4. [C.重新启动程序mongod作为副本集成员](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/#tutorial-index-on-replica-sets-restart-mongod)