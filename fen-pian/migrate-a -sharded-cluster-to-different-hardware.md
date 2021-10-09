

# Migrate a Sharded Cluster to Different Hardware

# 把一个分片集群迁移到不同的硬件

The tutorial is specific to MongoDB 5.0. For earlier versions of MongoDB, refer to the corresponding version of the MongoDB Manual.

本教程专用于MongoDB 5.0。对于MongoDB的早期版本，请参考相应版本的手册。

Starting in MongoDB 3.2, config servers for sharded clusters can be deployed as a [replica set](https://docs.mongodb.com/manual/replication/). The replica set config servers must run the [WiredTiger storage engine](https://docs.mongodb.com/manual/core/wiredtiger/). MongoDB 3.2 deprecates the use of three mirrored [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) instances for config servers.

在MongoDB 3.2中，对于分片集群的配置服务器可以配置为一个复制集合。这个复制集合配置服务器要运行在 [WiredTiger存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/)上。MongoDB 3.2 不可以在配置服务器时使用三个镜像 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)实例。

This procedure moves the components of the [sharded cluster](https://docs.mongodb.com/manual/reference/glossary/#std-term-sharded-cluster) to a new hardware system without downtime for reads and writes.

这个过程在不间断读写的情况下把分片服务器的组件移动到一个硬件系统。

**<!--IMPORTANT**-->

<!--重要-->

<!--While the migration is in progress, do not attempt to change to the [Sharded Cluster Metadata](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/#std-label-sharding-internals-config-database). Do not use any operation that modifies the cluster metadata *in any way*. For example, do not create or drop databases, create or drop collections, or use any sharding commands.-->

<!--当迁移过程中，不要尝试将数据改变为 [分片集群元数据](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/#std-label-sharding-internals-config-database)。不要使用任何操作将集群元数据修改为任何形式。例如，不要创建或删除数据库，创建或删除数据集合，或者使用任何分片命令。-->

## Disable the Balancer

## 关闭平衡器

Disable the balancer to stop [chunk migration](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) and do not perform any metadata write operations until the process finishes. If a migration is in progress, the balancer will complete the in-progress migration before stopping.

关闭平衡器以 [停止块迁移](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) ，在进程完成之前不要做任何元数据的写操作。如果是在迁移过程中，平衡器会在迁移进程停止之前完成。

## To disable the balancer, connect to one of the cluster's [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances and issue the following method: [[1\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-stop)

要关闭平衡器，连接到集群的 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例和发布以下方法： [[1\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-stop)

`sh.stopBalancer()`

To check the balancer state, issue the [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) method.

使用 [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 方法检查平衡器状态。

For more information, see [Disable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily).

欲知详情，请查看 [关闭平衡器](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily)

| [[1](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#ref-autosplit-stop-id1)] | Starting in MongoDB 4.2, [`sh.stopBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.stopBalancer/#mongodb-method-sh.stopBalancer) also disables auto-splitting for the sharded cluster. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [1]                                                          | 在MongoDB 4.2中启动，sh.stopBalancer()也可以关闭分片集群的自动分割 |

## Migrate Each Config Server Separately

## 分布迁移每个配置服务器

*Changed in version 3.4*

3.4版发生的变化

Starting in MongoDB 3.2, config servers for sharded clusters can be deployed as a [replica set](https://docs.mongodb.com/manual/replication/) (CSRS) instead of three mirrored config servers (SCCC). Using a replica set for the config servers improves consistency across the config servers, since MongoDB can take advantage of the standard replica set read and write protocols for the config data. In addition, using a replica set for config servers allows a sharded cluster to have more than 3 config servers since a replica set can have up to 50 members. To deploy config servers as a replica set, the config servers must run the [WiredTiger storage engine](https://docs.mongodb.com/manual/core/wiredtiger/).

在MongoDB 3.2中，用于分片集群的配置服务器可以部署为一个 [复制集合](https://docs.mongodb.com/manual/replication/) (CSRS) ，从而替代三个镜像配置服务器(SCCC)。使用一个用于配置服务器的复制集合提高一致性，MongoDB可以利用标准复制集合读写配置数据的协议。此外，由于一个复制集合可以拥有50个以上的成员，使用用于配置服务器的复制集合可以让一个分片集群拥有3个以上的配置服务器。配置服务器必须运行 [WiredTiger 存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/)才能把配置服务器部署为一个复制集合。

In version 3.4, MongoDB [removes support for SCCC config servers](https://docs.mongodb.com/manual/release-notes/3.4-compatibility/#std-label-3.4-compat-remove-sccc).

在3.4版本中，MongoDB[删除了对SCCC配置服务器的支持](https://docs.mongodb.com/manual/release-notes/3.4-compatibility/#std-label-3.4-compat-remove-sccc)。

The following restrictions apply to a replica set configuration when used for config servers:

党使用配置服务器时，以下限制应用于一个复制集合配置：

- Must have zero [arbiters](https://docs.mongodb.com/manual/core/replica-set-arbiter/).

- Must have no [delayed members](https://docs.mongodb.com/manual/core/replica-set-delayed-member/).

- Must build indexes (i.e. no member should have [`members[n\].buildIndexes`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.buildIndexes) setting set to false).

- 必须有0的判断。

- 必须没有延迟成员。

- 必须建立索引（ 例如，没有成员有[`members[n\].buildIndexes`]设置设为false）。

  For each member of the config server replica set:

  对每个配置服务器复制集合的成员：

  **IMPORTANT**

  **重要**

  **Replace the secondary members before replacing the primary.**

  **在替换第一个成员之前替换第二个成员。**

  ### 1.Start the replacement config server.

  ### 1.启动替换配置服务器。

  Start a [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) instance, specifying the `--configsvr`, `--replSet`, `--bind_ip` options, and other options as appropriate to your deployment.

  启动一个mongod实例，指定`--configsvr`, `--replSet`, `--bind_ip`设置，以及其他设置用于你的部署。

  **WARNING**

  **警告**

  Before binding to a non-localhost (e.g. publicly accessible) IP address, ensure you have secured your cluster from unauthorized access. For a complete list of security recommendations, see [Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/). At minimum, consider [enabling authentication](https://docs.mongodb.com/manual/administration/security-checklist/#std-label-checklist-auth) and [hardening network infrastructure](https://docs.mongodb.com/manual/core/security-hardening/).

  在绑定到一个非本地（例如公共访问）IP地址之前，确保你的集群没有非法访问。查看 [Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/)了解完整的安全推荐列表。最少，考虑 [开启认证](https://docs.mongodb.com/manual/administration/security-checklist/#std-label-checklist-auth) 并[加固网络基础设施](https://docs.mongodb.com/manual/core/security-hardening/)。

  `mongod --configsvr --replSet <replicaSetName> --bind_ip localhost,<hostname(s)|ip address(es)>`

  ### 2.Add the new config server to the replica set.

  ### 2.给复制集合添加新的配置服务器。

  Connect [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) to the primary of the config server replica set and use [`rs.add()`](https://docs.mongodb.com/manual/reference/method/rs.add/#mongodb-method-rs.add) to add the new member.

  把 [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) 连接到配置服务器复制集合的主模块，使用 [`rs.add()`](https://docs.mongodb.com/manual/reference/method/rs.add/#mongodb-method-rs.add) 添加新的成员。

  **WARNING**

  **警告**

  Before MongoDB 5.0, a newly added secondary still counts as a voting member even though it can neither serve reads nor become primary until its data is consistent. If you are running a MongoDB version earlier than 5.0 and add a secondary with its [`votes`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes) and [`priority`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) settings greater than zero, this can lead to a case where a majority of the voting members are online but no primary can be elected. To avoid such situations, consider adding the new secondary initially with [`priority :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) and [`votes :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes). Then, run [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status) to ensure the member has transitioned into [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#mongodb-replstate-replstate.SECONDARY) state. Finally, use [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) to update its priority and votes.

  **在**MongoDB 5.0之前，作为一个投票成员，一个新添加的二级静态计数节点，尽管在数据是连续的之前，其既不能读也不能成为主模块。如果你正在运行一个MongoDB 5.0的早期版本，并使用其 [`votes`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes) 和 [`priority`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) 大于零的设置添加一个二级节点，这可能导致出现的情况是，大多数投票成员在线，但主要成员落选。为避免这种情况，可以使用 [`priority :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) 和[`votes :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes)添加新的二级节点。然后，运行 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)以确保成员过渡到二级节点状态。最后，使用 [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 更新其票数和权重。

  `rs.add( { host: "<hostnameNew>:<portNew>", priority: 0, votes: 0 } )`

  The initial sync process copies all the data from one member of the config server replica set to the new member without restarting.

  最初的同步进程无需重启，从配置服务器复制集合的一个成员复制所有的数据到新成员中。

  [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances automatically recognize the change in the config server replica set members without restarting.

  无需重启，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例自动识别配置服务器复制集合成员的变化。

  ### 3.Update the newly added config server's votes and priority settings.

  ### 3.更新新添加的配置服务器的票数和权重设置。

  a.Ensure that the new member has reached [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#mongodb-replstate-replstate.SECONDARY) state. To check the state of the replica set members, run [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status):

  a.确保新成员达到 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#mongodb-replstate-replstate.SECONDARY) 状态。运行 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)检查复制集合成员的状态：

  `rs.status()`

  b.Reconfigure the replica set to update the votes and priority of the new member:

  b.再次配置复制集合以更新新成员的票数和权重：

  `var cfg = rs.conf();`

  `cfg.members[n].priority = 1;  // Substitute the correct array index for the new member`
  `cfg.members[n].votes = 1;     // Substitute the correct array index for the new member`

  `rs.reconfig(cfg)`

  where `n` is the array index of the new member in the [`members`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members) array.

  n是成员数组中新成员的数组索引。

  **WARNING**

  **警告**

  - The [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) shell method can force the current primary to step down, which causes an [election](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-elections). When the primary steps down, the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) closes all client connections. While this typically takes 10-20 seconds, try to make these changes during scheduled maintenance periods.

  -  [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 壳方法可以强制当前的主节点让位，从而引起一次[election](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-elections)。当主让位后， [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 关闭所有的客户端连接。在计划的维护周期尝试做这些变化一般会用10-20秒时间。

  - Avoid reconfiguring replica sets that contain members of different MongoDB versions as validation rules may differ across MongoDB versions.

  - 避免重新配置包含不同MongoDB版本成员的复制集合作为校验规则，可能不同于其他MongoDB版本。

    ### 4.Shut down the member to replace.

    ### 4.关闭成员以进行替换。

    If replacing the primary member, step down the primary first before shutting down.

    如果替换主要成员，在关闭之前先要停下主节点。

    ### 5.Remove the member to replace from the config server replica set.

    ### 5.移除成员以从配置服务器复制集合进行替换。

    Upon completion of initial sync of the replacement config server, from a [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) session that is connected to the primary, use [`rs.remove()`](https://docs.mongodb.com/manual/reference/method/rs.remove/#mongodb-method-rs.remove) to remove the old member.

    在完成替换配置服务器的初始化同步之上，从一个连接到主节点的 [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) session，使用 [`rs.remove()`](https://docs.mongodb.com/manual/reference/method/rs.remove/#mongodb-method-rs.remove) 移除老的成员。

    ```
    rs.remove("<hostnameOld>:<portOld>")
    ```

    [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances automatically recognize the change in the config server replica set members without restarting.

    [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例无需重启即可自动识别配置服务器复制集合成员的变化。

    ## Restart the `mongos` Instances

    ### 重启mongos实例

    *Changed in version 3.2*: With replica set config servers, the [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances specify in the [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) or [`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB) setting the config server replica set name and at least one of the replica set members. The [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances for the sharded cluster must specify the same config server replica set name but can specify different members of the replica set.

    版本3.3的变化：使用复制集合配置服务器， [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例指定 [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) 或者 [`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB)设置配置服务器复制集合名称和至少一个复制集合成员。用于分片集群的 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例必须指定同样的配置服务器集合名称，但可以指定不同的复制集合成员。

    If a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instance specifies a migrated replica set member in the [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) or [`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB) setting, update the config server setting for the next time you restart the [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instance.

    如果 一个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例指定了一个 [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) 或者[`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB) 设置中的迁移的复制集合成员，下次重启[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例时要更新配置服务器。

    For more information, see [Start a `mongos` for the Sharded Cluster](https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/#std-label-sharding-setup-start-mongos).

    详情查阅[Start a `mongos` for the Sharded Cluster](https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/#std-label-sharding-setup-start-mongos)。

    ## Migrate the Shards

    ## 迁移分片

    Migrate the shards one at a time. For each shard, follow the appropriate procedure in this section.

    一次迁移一个分片。对于每个分片，伴随本部分的适当过程。

    ### Migrate a Replica Set Shard

    ### 迁移一个复制集合分片

    To migrate a sharded cluster, migrate each member separately. First migrate the non-primary members, and then migrate the [primary](https://docs.mongodb.com/manual/reference/glossary/#std-term-primary) last.

    要迁移一个分片集合，应分别迁移每个成员。首先迁移非主要成员，最后迁移主成员。

    If the replica set has two voting members, add an [arbiter](https://docs.mongodb.com/manual/core/replica-set-arbiter/) to the replica set to ensure the set keeps a majority of its votes available during the migration. You can remove the arbiter after completing the migration.

    如果复制集合有两个投票成员，给复制集合添加一个仲裁器，确保该集合在迁移期间拥有大多数投票。你可以在完成迁移之后移除仲裁器。

    #### Migrate a Member of a Replica Set Shard

    #### 迁移一个复制集合分片的成员

    1.Shut down the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) process. To ensure a clean shutdown, use the [`shutdown`](https://docs.mongodb.com/manual/reference/command/shutdown/#mongodb-dbcommand-dbcmd.shutdown) command.

    1.关闭 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程，使用 [`shutdown`](https://docs.mongodb.com/manual/reference/command/shutdown/#mongodb-dbcommand-dbcmd.shutdown) 命令确保一个干净的关闭。

    2.Move the data directory (i.e., the [`dbPath`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-storage.dbPath)) to the new machine.

    2.移动数据目录（例如 [`dbPath`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-storage.dbPath)）到新的机器。

    3.Restart the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) process at the new location.

    3.重启 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程至新的位置。

    4.Connect to the replica set's current primary.

    4.连接到复制集合的当前主节点。

    5.If the hostname of the member has changed, use [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) to update the [replica set configuration document](https://docs.mongodb.com/manual/reference/replica-configuration/) with the new hostname.

    For example, the following sequence of commands updates the hostname for the instance at position `2` in the `members` array:

    5.如果成员的主机名变化了，使用 [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 以更新新的主机名到复制集合配置文档。

    例如，以下一些命令在成员数组的位置2的实例更新主机名：

    `cfg = rs.conf()`
    `cfg.members[2].host = "pocatello.example.net:27018"`
    `rs.reconfig(cfg)`

    For more information on updating the configuration document, see [Examples](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#std-label-replica-set-reconfiguration-usage).

    想要了解更新配置文档的更多信息，查看 [Examples](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#std-label-replica-set-reconfiguration-usage)。

    6.To confirm the new configuration, issue [`rs.conf()`](https://docs.mongodb.com/manual/reference/method/rs.conf/#mongodb-method-rs.conf).

    6.使用[`rs.conf()`](https://docs.mongodb.com/manual/reference/method/rs.conf/#mongodb-method-rs.conf)确认新的配置。

    7.Wait for the member to recover. To check the member's state, issue [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status).

    7.等待成员回复。使用 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)检查成员状态。

    #### Migrate the Primary in a Replica Set Shard

    #### 在复制集合分片中迁移主节点

    While migrating the replica set's primary, the set must elect a new primary. This failover process which renders the replica set unavailable to perform reads or accept writes for the duration of the election, which typically completes quickly. If possible, plan the migration during a maintenance window.

    当迁移复制集合的主节点时，集合必须选择一个新的主节点。这种提交复制集合的故障转移过程设置选举过程期间的读操作或者接收写操作为无效，这通常很快就可以完成。如果可能的话，在维护窗口期间计划迁移。

    1.Step down the primary to allow the normal [failover](https://docs.mongodb.com/manual/core/replica-set-high-availability/#std-label-replica-set-failover) process. To step down the primary, connect to the primary and issue the either the [`replSetStepDown`](https://docs.mongodb.com/manual/reference/command/replSetStepDown/#mongodb-dbcommand-dbcmd.replSetStepDown) command or the [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) method. The following example shows the [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) method:

    1.关闭主节点以启动正常的故障转移进程。为了关闭主节点，应连接到主节点或者使用 [`replSetStepDown`](https://docs.mongodb.com/manual/reference/command/replSetStepDown/#mongodb-dbcommand-dbcmd.replSetStepDown) 命令或者 [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) 方法。以下的例子展示了 [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) 方法：

    `rs.stepDown()`

    2.Once the primary has stepped down and another member has become PRIMARY state. To migrate the stepped-down primary, follow the Migrate a Member of a Replica Set Shard procedure  You can check the output of rs.status() to confirm the change in status.

    2.一旦主节点关闭，其他的成员成为主节点状态。为了迁移关闭的主节点，随着迁移一个复制集合分片进程的成员，你可以检查rs.status()的输出以确认状态的变化。

    ## Re-Enable the Balancer

    ### 重新打开平衡器

    To complete the migration, re-enable the balancer to resume [chunk migrations](https://docs.mongodb.com/manual/core/sharding-balancer-administration/).

    要完成迁移，重新打开平衡器并恢复 [chunk migrations](https://docs.mongodb.com/manual/core/sharding-balancer-administration/)。

    Connect to one of the cluster's [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances and pass `true` to the [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) method: [[2\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-start)

    连接一个集群的[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例，并传递true值给 [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 方法：[[2\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-start)

    `sh.startBalancer()`

    To check the balancer state, issue the [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) method.

    要检查平衡器状态，使用 [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 方法。

    For more information, see [Enable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable).

    详情查看 [打开平衡器](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable)。

    | [[2](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#ref-autosplit-start-id2)] | Starting in MongoDB 4.2, [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) also enables auto-splitting for the sharded cluster. |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | [2]                                                          | 从 MongoDB 4.2开始， [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 也可以用于分片集群的自动分割。 |



译者：张冲

原文链接：https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/