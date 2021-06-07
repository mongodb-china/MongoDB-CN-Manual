# MongoDB复制参考

## Replication Reference

## 复制参考

On this page

内容概览

* [Replication Methods in the `mongo` Shell](https://docs.mongodb.com/v4.2/reference/replication/#replication-methods-in-the-mongo-shell)
* [`mongo` Shell中的复制方法](https://docs.mongodb.com/v4.2/reference/replication/#replication-methods-in-the-mongo-shell)
* [Replication Database Commands](https://docs.mongodb.com/v4.2/reference/replication/#replication-database-commands)
* [关于复制的数据库命令](https://docs.mongodb.com/v4.2/reference/replication/#replication-database-commands)
* [Replica Set Reference Documentation](https://docs.mongodb.com/v4.2/reference/replication/#replica-set-reference-documentation)
* [副本集参考文档](https://docs.mongodb.com/v4.2/reference/replication/#replica-set-reference-documentation)

### Replication Methods in the `mongo` Shell

| Name | Description |
| :--- | :--- |
| [`rs.add()`](https://docs.mongodb.com/v4.2/reference/method/rs.add/#rs.add) | Adds a member to a replica set. 将成员添加到副本集。 |
| [`rs.addArb()`](https://docs.mongodb.com/v4.2/reference/method/rs.addArb/#rs.addArb) | Adds an [arbiter](https://docs.mongodb.com/v4.2/reference/glossary/#term-arbiter) to a replica set. 将仲裁节点添加到副本集。 |
| [`rs.conf()`](https://docs.mongodb.com/v4.2/reference/method/rs.conf/#rs.conf) | Returns the replica set configuration document. 返回副本集的配置内容。 |
| [`rs.freeze()`](https://docs.mongodb.com/v4.2/reference/method/rs.freeze/#rs.freeze) | Prevents the current member from seeking election as primary for a period of time. 阻止当前成员在一段时间内寻求选举为主节点。 |
| [`rs.help()`](https://docs.mongodb.com/v4.2/reference/method/rs.help/#rs.help) | Returns basic help text for [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set) functions. 返回[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)功能的基本帮助文本。 |
| [`rs.initiate()`](https://docs.mongodb.com/v4.2/reference/method/rs.initiate/#rs.initiate) | Initializes a new replica set. 初始化新的副本集。 |
| [`rs.printReplicationInfo()`](https://docs.mongodb.com/v4.2/reference/method/rs.printReplicationInfo/#rs.printReplicationInfo) | Prints a report of the status of the replica set from the perspective of the primary. 以主节点的角度来打印副本集状态的报告。 |
| [`rs.printSlaveReplicationInfo()`](https://docs.mongodb.com/v4.2/reference/method/rs.printSlaveReplicationInfo/#rs.printSlaveReplicationInfo) | Prints a report of the status of the replica set from the perspective of the secondaries. 以从节点的角度来打印副本集状态的报告。 |
| [`rs.reconfig()`](https://docs.mongodb.com/v4.2/reference/method/rs.reconfig/#rs.reconfig) | Re-configures a replica set by applying a new replica set configuration object. 通过应用新的副本集配置对象来重新配置副本集。 |
| [`rs.remove()`](https://docs.mongodb.com/v4.2/reference/method/rs.remove/#rs.remove) | Remove a member from a replica set. 将成员从副本集中移除。 |
| [`rs.status()`](https://docs.mongodb.com/v4.2/reference/method/rs.status/#rs.status) | Returns a document with information about the state of the replica set. 返回包含关于副本集状态信息的文档。 |
| [`rs.stepDown()`](https://docs.mongodb.com/v4.2/reference/method/rs.stepDown/#rs.stepDown) | Causes the current [primary](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary) to become a secondary which forces an [election](https://docs.mongodb.com/v4.2/reference/glossary/#term-election). 使当前的[主节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary)转变为从节点，同时触发[选举](https://docs.mongodb.com/v4.2/reference/glossary/#term-election)。 |
| [`rs.syncFrom()`](https://docs.mongodb.com/v4.2/reference/method/rs.syncFrom/#rs.syncFrom) | Sets the member that this replica set member will sync from, overriding the default sync target selection logic. 设置复制集成员从哪个成员中同步数据，同时覆盖默认的同步目标选择逻辑。 |

### Replication Database Commands

| Name | Description |
| :--- | :--- |
| [`applyOps`](https://docs.mongodb.com/v4.2/reference/command/applyOps/#dbcmd.applyOps) | Internal command that applies [oplog](https://docs.mongodb.com/v4.2/reference/glossary/#term-oplog) entries to the current data set. 内部命令，可将[oplog](https://docs.mongodb.com/v4.2/reference/glossary/#term-oplog)条目应用于当前数据集。 |
| [`isMaster`](https://docs.mongodb.com/v4.2/reference/command/isMaster/#dbcmd.isMaster) | Displays information about this member’s role in the replica set, including whether it is the master. 显示关于此成员在副本集中的角色信息，包括它是否为主角色。 |
| [`replSetAbortPrimaryCatchUp`](https://docs.mongodb.com/v4.2/reference/command/replSetAbortPrimaryCatchUp/#dbcmd.replSetAbortPrimaryCatchUp) | Forces the elected [primary](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary) to abort sync \(catch up\) then complete the transition to primary. 对所选的主节点强行中止同步（即追平数据），然后完成到主节点的转换。 |
| [`replSetFreeze`](https://docs.mongodb.com/v4.2/reference/command/replSetFreeze/#dbcmd.replSetFreeze) | Prevents the current member from seeking election as [primary](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary) for a period of time. 阻止当前成员在一段时间内寻求选举为[主节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary)。 |
| [`replSetGetConfig`](https://docs.mongodb.com/v4.2/reference/command/replSetGetConfig/#dbcmd.replSetGetConfig) | Returns the replica set’s configuration object. 返回副本集的配置对象。 |
| [`replSetGetStatus`](https://docs.mongodb.com/v4.2/reference/command/replSetGetStatus/#dbcmd.replSetGetStatus) | Returns a document that reports on the status of the replica set. 返回报告副本集状态的文档。 |
| [`replSetInitiate`](https://docs.mongodb.com/v4.2/reference/command/replSetInitiate/#dbcmd.replSetInitiate) | Initializes a new replica set. 初始化新的副本集。 |
| [`replSetMaintenance`](https://docs.mongodb.com/v4.2/reference/command/replSetMaintenance/#dbcmd.replSetMaintenance) | Enables or disables a maintenance mode, which puts a [secondary](https://docs.mongodb.com/v4.2/reference/glossary/#term-secondary) node in a `RECOVERING` state. 启用或禁用维护模式，该模式会将[从节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-secondary)置于`RECOVERING`状态。 |
| [`replSetReconfig`](https://docs.mongodb.com/v4.2/reference/command/replSetReconfig/#dbcmd.replSetReconfig) | Applies a new configuration to an existing replica set. 将新的配置应用于现有副本集。 |
| [`replSetResizeOplog`](https://docs.mongodb.com/v4.2/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) | Dynamically resizes the oplog for a replica set member. Available for WiredTiger storage engine only. 动态调整副本集成员oplog的大小。该功能仅适用于WiredTiger存储引擎。 |
| [`replSetStepDown`](https://docs.mongodb.com/v4.2/reference/command/replSetStepDown/#dbcmd.replSetStepDown) | Forces the current [primary](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary) to _step down_ and become a [secondary](https://docs.mongodb.com/v4.2/reference/glossary/#term-secondary), forcing an election. 使当前的[主节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary)转变为[从节点](https://docs.mongodb.com/v4.2/reference/glossary/#term-secondary),，同时触发[选举](https://docs.mongodb.com/v4.2/reference/glossary/#term-election)。 |
| [`replSetSyncFrom`](https://docs.mongodb.com/v4.2/reference/command/replSetSyncFrom/#dbcmd.replSetSyncFrom) | Explicitly override the default logic for selecting a member to replicate from. 显式重写用于选择要复制的成员的默认逻辑。 |

### Replica Set Reference Documentation

### 副本集参考文档

* [Replica Set Configuration](https://docs.mongodb.com/v4.2/reference/replica-configuration/)
* [副本集配置](https://docs.mongodb.com/v4.2/reference/replica-configuration/)

  Complete documentation of the [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set) configuration object returned by [`rs.conf()`](https://docs.mongodb.com/v4.2/reference/method/rs.conf/#rs.conf).

  [`rs.conf()`](https://docs.mongodb.com/v4.2/reference/method/rs.conf/#rs.conf)命令返回的副本集配置对象的完整文档。

* [副本集协议版本](https://docs.mongodb.com/v4.2/reference/replica-set-protocol-versions/)
* [Replica Set Protocol Version](https://docs.mongodb.com/v4.2/reference/replica-set-protocol-versions/)

  参考副本集协议版本。

* [Troubleshoot Replica Sets](https://docs.mongodb.com/v4.2/tutorial/troubleshoot-replica-sets/)
* [副本集的故障排查](https://docs.mongodb.com/v4.2/tutorial/troubleshoot-replica-sets/)

  副本集故障排查指南。

* [The local Database](https://docs.mongodb.com/v4.2/reference/local-database/)
* [local数据库](https://docs.mongodb.com/v4.2/reference/local-database/)

  Complete documentation of the content of the `local` database that [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) instances use to support replication.

  关于`local`数据库内容介绍的完整文档，该数据库在[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)实例中用于支持复制功能。

* [Replica Set Member States](https://docs.mongodb.com/v4.2/reference/replica-states/)
* [副本集成员状态](https://docs.mongodb.com/v4.2/reference/replica-states/)

  副本集成员状态的参考。

  原文链接：[https://docs.mongodb.com/v4.2/reference/replication/](https://docs.mongodb.com/v4.2/reference/replication/)

  译者：桂陈

