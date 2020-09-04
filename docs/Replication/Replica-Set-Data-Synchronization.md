# 副本集数据同步

在本页

- [初始化同步](https://docs.mongodb.com/manual/core/replica-set-sync/#initial-sync)
- [复制](https://docs.mongodb.com/manual/core/replica-set-sync/#replication)

为了维护共享数据集的最新副本，副本集中的从节点成员可以从其他成员同步或复制数据。MongoDB中有两种形式的数据同步：初始化同步将完整的数据集填充至新成员；而复制会持续将变更应用到整个数据集上。



## 初始化同步


初始化同步会从副本集成员中的一个节点复制所有的数据到另外一个成员。有关初始化同步源选择条件的更多信息请参见[初始化同步源选择](https://docs.mongodb.com/manual/core/replica-set-sync/#replica-set-initial-sync-source-selection)。

从MongoDB 4.2.7开始，你可以使用参数[`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 指定优先的初始化同步源。这只能在启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)时配置。


### 过程


当执行一个初始化同步时，MongoDB会：

1. 克隆除[local](https://docs.mongodb.com/manual/reference/local-database/#replica-set-local-database)数据库之外的所有数据库。为了进行克隆，[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 扫描每个源数据库中的各个集合，并将所有数据插入到这些集合各自的副本中。

   3.4版本的变化：初始化同步在为每个集合复制文档时会建立集合的所有索引。在MongoDB的早期版本中，在这个阶段只建立_id索引。

   3.4版本的变化：初始化同步会获取在数据复制期间新增的oplog记录。请确保目标成员的local 数据库中有足够的磁盘空间，以便可以在数据复制阶段期间内临时存储这些oplog记录。

2. 对数据集应用所有的更改。使用来自源库的oplog，[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 更新其数据集以反映副本集的当前状态。当初始化同步完成后，目标成员会从 [`STARTUP2`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.STARTUP2)状态转为[`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.SECONDARY)状态。

若要执行初始化同步，请参见[重新同步副本集成员](https://docs.mongodb.com/manual/tutorial/resync-replica-set-member/)。



### 容错


为了从短暂网络或操作故障中恢复，初始化同步具有内置的重试逻辑。

版本3.4的变化：MongoDB 3.4改进了初始化同步的重试逻辑，对网络上的间歇性故障更有弹性。


#### 初始化同步源的选择


初始化同步源的选择取决于[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 的启动参数[`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) （版本4.2.7中的新参数）：

- 若参数[`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 设置为 [`primary`](https://docs.mongodb.com/manual/core/read-preference/#primary) （禁用级联后的默认值），则选择主节点作为同步源。如果主服务器不可用或无法访问，则记录错误并定期检查主服务器的可用性。
- 若参数[`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 设置为[`primaryPreferred`](https://docs.mongodb.com/manual/core/read-preference/#primaryPreferred)，则优先尝试选择主节点作为同步源。如果主节点不可用或者无法访问，则将从剩余可用的副本集成员中选择同步源。
- 若参数[`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 设置为[`nearest`](https://docs.mongodb.com/manual/core/read-preference/#nearest) （启用级联后的默认值），则从副本集成员中选择网络时延最小的节点最为同步源。
- 对于所有其他受支持的读偏好类型，则将从这些副本集成员中选择同步源。


执行初始化同步源选择的成员将会遍历所有副本集成员的列表两次：


**同步源选择（第一次遍历）**


当选择初始同步源进行第一次遍历时，执行同步源选择的成员将检查每个副本集成员是否满足如下条件：

- 同步源必须处于 [`PRIMARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.PRIMARY) 或者 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.SECONDARY) 的复制状态。
- 同步源必须是在线且可访问的。
- 如果参数 [`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 设置为 [`secondary`](https://docs.mongodb.com/manual/core/read-preference/#secondary) 或者 [`secondaryPreferred`](https://docs.mongodb.com/manual/core/read-preference/#secondaryPreferred)，则同步源必须是一个从节点。
- 同步源必须和主节点最新的oplog条目同步时间相差在30s之内。
- 如果该成员是可创建索引的，则同步源也必须可创建索引。
- 如果该成员可参与副本集选举投票，则同步源也必须具有投票权。
- 如果该成员不是一个延迟成员，则同步源也不能是延迟成员。
- 如果该成员是一个延迟成员，则同步源必须配置一个更短的延迟时间。
- 同步源必须比当前最好的同步源更快(即更低的时延)。


如果第一次遍历没有产生候选的同步源，则该成员会用更宽松的条件进行第二次遍历。请参考**同步源选择（第二次遍历）**。


**同步源选择（第二次遍历）**


当选择初始同步源进行第二次遍历时，执行同步源选择的成员将检查每个副本集成员是否满足如下条件：

- 同步源必须处于 [`PRIMARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.PRIMARY) 或者 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.SECONDARY) 的复制状态。
- 同步源必须是在线且可访问的。
- 如果参数 [`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 设置为 [`secondary`](https://docs.mongodb.com/manual/core/read-preference/#secondary) ，则同步源必须是一个从节点。
- 如果该成员是可创建索引的，则同步源也必须可创建索引。
- 同步源必须比当前最好的同步源更快(即更低的时延)。

如果该成员在两次遍历后依然无法选择出初始同步源，它会记录报错并在等待1s后重新发起选择的过程。从节点的Mongod进程在出现报错退出之前，最多会重试10次初始同步源选择的过程。


## 复制


从节点成员在初始化同步之后会不断地复制数据。从节点成员从同步源复制[oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/) ，并以异步的方式应用这些操作 [[1\]](https://docs.mongodb.com/manual/core/replica-set-sync/#slow-oplogs)。

从节点可以根据ping时间和其他成员复制状态的变化，按需来自动调整它们的同步源。

3.2版本的变化：MongoDB 3.2中投票权为1的副本集成员无法从投票权为0的成员那里同步数据。

从节点应避免从[延迟成员](https://docs.mongodb.com/manual/core/replica-set-delayed-member/#replica-set-delayed-members)和[隐藏成员](https://docs.mongodb.com/manual/core/replica-set-hidden-member/#replica-set-hidden-members)中同步数据。


如果一个从节点成员的参数[`members[n].buildIndexes`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].buildIndexes) 设置为true，它只能从其他参数[buildIndexes](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].buildIndexes)设置为true的成员同步数据。参数[buildIndexes](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].buildIndexes)设置为false的成员可以从任何其他节点同步数据，除非有其他的同步限制。参数[buildIndexes](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].buildIndexes)默认为true。


 [[1\]](https://docs.mongodb.com/manual/core/replica-set-sync/#id1) | 从4.2版本开始（从4.0.6开始也是可行的），副本集的副本成员会记录oplog中应用时间超过慢操作阈值的慢操作条目。这些慢oplog信息被记录在从节点的诊断日志中，其路径位于REPL 组件的文本`applied op: took ms`中。这些慢日志条目仅仅依赖于慢操作阈值。它们不依赖于日志级别（无论是系统还是组件级别）、过滤级别，或者慢操作采样比例。过滤器不会捕获慢日志条目。 


### 多线程复制

MongoDB通过使用多线程批量应用写操作来提高并发。MongoDB根据文档id （[WiredTiger](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger)）进行分批，同时使用不同的线程应用每组操作。MongoDB总是按照原始的写顺序对给定的文档应用写操作。

*4.0版本的变化。*

从MongoDB 4.0开始，如果读取发生在正在应用批量复制的从节点上，那么针对从节点且读关注级别设置为“local”或“majority”的读取操作，现在将从WiredTiger数据快照中读取数据。从快照中读取数据可以保证数据的一致性视图，并且允许在进行复制的同时进行读取，而不需要使用锁。因此，这些读关注级别的从节点读取操作不再需要等待批量复制应用完成，并且可以在接收它们的同时进行处理。


### 流控制


从MongoDB 4.2开始，管理员可以限制主节点应用其写操作的速度，目的是将大多数提交延迟保持在可配置参数[`flowControlTargetLagSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.flowControlTargetLagSeconds)最大值之下。

默认情况下，流控制是启用的。

说明

为了进行流控制，副本集/分片集群必须满足：参数[featureCompatibilityVersion (FCV)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#view-fcv) 设置为4.2，并启用majority级别的读关注。也就是说，如果FCV不是4.2，或者读关注majority被禁用，那么流控制的启用将不会生效。

更多信息请参见[流控制](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#flow-control)。


### 复制同步源的选择


复制同步源的选择取决于副本集参数[`chaining`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.chainingAllowed) 的设置：

- 启用级联(默认)后，从副本集成员间执行同步源选择。
- 禁用级联后，选择主节点作为复制源。如果主服务器不可用或无法访问，则记录错误并定期检查主服务器的可用性。

Members performing replication sync source selection make two passes through the list of all replica set members:

执行复制同步源选择的成员将会遍历所有副本集成员的列表两次：


**同步源选择（第一次）**


当为选择复制同步源进行第一次遍历时，执行同步源选择的成员将检查每个副本集成员是否满足如下条件：

- 同步源必须处于 [`PRIMARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.PRIMARY) 或者 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.SECONDARY) 的复制状态。
- 同步源必须是在线且可访问的。
- 同步源必须比该成员具有更新的oplog条目（即同步源数据同步领先于该成员）。
- 同步源必须是[可见的](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].hidden)。
- 同步源必须和主节点最新的oplog条目同步时间相差在30s之内。
- 如果该成员是可创建索引的，则同步源也必须可创建索引。
- 如果该成员可参与副本集选举投票，则同步源也必须具有投票权。
- 如果该成员不是一个延迟成员，则同步源也不能是延迟成员。
- 如果该成员是一个延迟成员，则同步源必须配置一个更短的延迟时间。
- 同步源必须比当前最好的同步源更快(即更低的时延)。

如果第一次遍历没有产生候选的同步源，则该成员会用更宽松的条件进行第二次遍历。请参考**同步源选择（第二次遍历）**。


**同步源选择（第二次遍历）**

当为选择复制同步源进行第二次遍历时，执行同步源选择的成员将检查每个副本集成员是否满足如下条件：

- 同步源必须处于 [`PRIMARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.PRIMARY) 或者 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#replstate.SECONDARY) 的复制状态。
- 同步源必须是在线且可访问的。
- 如果该成员是可创建索引的，则同步源也必须可创建索引。
- 同步源必须比当前最好的同步源更快(即更低的时延)。

如果该成员在两次遍历后依然无法选择出初始同步源，它会记录报错并在等待1s后重新发起选择的过程。

说明

从MongoDB 4.2.7开始，当选择一个初始化同步源时，启动参数 [`initialSyncSourceReadPreference`](https://docs.mongodb.com/manual/reference/parameters/#param.initialSyncSourceReadPreference) 是优先级高于副本集参数 [`settings.chainingAllowed`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.chainingAllowed)。在副本集成员成功执行初始化同步之后，选择复制同步源时则取决于参数 [`chainingAllowed`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.chainingAllowed)的值。

有关初始化同步源的选择的更多信息请参考[初始化同步源的选择](https://docs.mongodb.com/manual/core/replica-set-sync/#replica-set-initial-sync-source-selection) 。



原文链接：https://docs.mongodb.com/manual/core/replica-set-sync/

译者：李正洋
