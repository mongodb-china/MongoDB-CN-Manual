



# 把一个分片集群迁移到不同的硬件


本教程专用于MongoDB 5.0。对于MongoDB的早期版本，请参考相应版本的手册。


在MongoDB 3.2中，对于分片集群的配置服务器可以配置为一个复制集合。这个复制集合配置服务器要运行在 [WiredTiger存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/)上。MongoDB 3.2 不可以在配置服务器时使用三个镜像 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)实例。


这个过程在不间断读写的情况下把分片服务器的组件移动到一个硬件系统。


<!--重要-->


<!--当迁移过程中，不要尝试将数据改变为 [分片集群元数据](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/#std-label-sharding-internals-config-database)。不要使用任何操作将集群元数据修改为任何形式。例如，不要创建或删除数据库，创建或删除数据集合，或者使用任何分片命令。-->



## 关闭平衡器



关闭平衡器以 [停止块迁移](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) ，在进程完成之前不要做任何元数据的写操作。如果是在迁移过程中，平衡器会在迁移进程停止之前完成。


要关闭平衡器，连接到集群的 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例和发布以下方法： [[1\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-stop)

`sh.stopBalancer()`



使用 [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 方法检查平衡器状态。


欲知详情，请查看 [关闭平衡器](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily)

| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [1]                                                          | 在MongoDB 4.2中启动，sh.stopBalancer()也可以关闭分片集群的自动分割 |



## 分布迁移每个配置服务器



3.4版发生的变化



在MongoDB 3.2中，用于分片集群的配置服务器可以部署为一个 [复制集合](https://docs.mongodb.com/manual/replication/) (CSRS) ，从而替代三个镜像配置服务器(SCCC)。使用一个用于配置服务器的复制集合提高一致性，MongoDB可以利用标准复制集合读写配置数据的协议。此外，由于一个复制集合可以拥有50个以上的成员，使用用于配置服务器的复制集合可以让一个分片集群拥有3个以上的配置服务器。配置服务器必须运行 [WiredTiger 存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/)才能把配置服务器部署为一个复制集合。


在3.4版本中，MongoDB[删除了对SCCC配置服务器的支持](https://docs.mongodb.com/manual/release-notes/3.4-compatibility/#std-label-3.4-compat-remove-sccc)。

党使用配置服务器时，以下限制应用于一个复制集合配置：


- 必须有0的判断。

- 必须没有延迟成员。

- 必须建立索引（ 例如，没有成员有[`members[n\].buildIndexes`]设置设为false）。


  对每个配置服务器复制集合的成员：



  **重要**


  **在替换第一个成员之前替换第二个成员。**



  ### 1.启动替换配置服务器。

  启动一个mongod实例，指定`--configsvr`, `--replSet`, `--bind_ip`设置，以及其他设置用于你的部署。

 

  **警告**



  在绑定到一个非本地（例如公共访问）IP地址之前，确保你的集群没有非法访问。查看 [Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/)了解完整的安全推荐列表。最少，考虑 [开启认证](https://docs.mongodb.com/manual/administration/security-checklist/#std-label-checklist-auth) 并[加固网络基础设施](https://docs.mongodb.com/manual/core/security-hardening/)。

  `mongod --configsvr --replSet <replicaSetName> --bind_ip localhost,<hostname(s)|ip address(es)>`


  ### 2.给复制集合添加新的配置服务器。



  把 [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) 连接到配置服务器复制集合的主模块，使用 [`rs.add()`](https://docs.mongodb.com/manual/reference/method/rs.add/#mongodb-method-rs.add) 添加新的成员。


  **警告**



  **在**MongoDB 5.0之前，作为一个投票成员，一个新添加的二级静态计数节点，尽管在数据是连续的之前，其既不能读也不能成为主模块。如果你正在运行一个MongoDB 5.0的早期版本，并使用其 [`votes`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes) 和 [`priority`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) 大于零的设置添加一个二级节点，这可能导致出现的情况是，大多数投票成员在线，但主要成员落选。为避免这种情况，可以使用 [`priority :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.priority) 和[`votes :0`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.members-n-.votes)添加新的二级节点。然后，运行 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)以确保成员过渡到二级节点状态。最后，使用 [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 更新其票数和权重。

  `rs.add( { host: "<hostnameNew>:<portNew>", priority: 0, votes: 0 } )`



  最初的同步进程无需重启，从配置服务器复制集合的一个成员复制所有的数据到新成员中。



  无需重启，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例自动识别配置服务器复制集合成员的变化。


  ### 3.更新新添加的配置服务器的票数和权重设置。



  a.确保新成员达到 [`SECONDARY`](https://docs.mongodb.com/manual/reference/replica-states/#mongodb-replstate-replstate.SECONDARY) 状态。运行 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)检查复制集合成员的状态：

  `rs.status()`


  b.再次配置复制集合以更新新成员的票数和权重：

  `var cfg = rs.conf();`

  `cfg.members[n].priority = 1;  // Substitute the correct array index for the new member`
  `cfg.members[n].votes = 1;     // Substitute the correct array index for the new member`

  `rs.reconfig(cfg)`



  n是成员数组中新成员的数组索引。



  **警告**



  -  [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 壳方法可以强制当前的主节点让位，从而引起一次[election](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-elections)。当主让位后， [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 关闭所有的客户端连接。在计划的维护周期尝试做这些变化一般会用10-20秒时间。



  - 避免重新配置包含不同MongoDB版本成员的复制集合作为校验规则，可能不同于其他MongoDB版本。

  
    ### 4.关闭成员以进行替换。


    如果替换主要成员，在关闭之前先要停下主节点。


    ### 5.移除成员以从配置服务器复制集合进行替换。


    在完成替换配置服务器的初始化同步之上，从一个连接到主节点的 [`mongosh`](https://docs.mongodb.com/mongodb-shell/#mongodb-binary-bin.mongosh) session，使用 [`rs.remove()`](https://docs.mongodb.com/manual/reference/method/rs.remove/#mongodb-method-rs.remove) 移除老的成员。

    ```
    rs.remove("<hostnameOld>:<portOld>")
    ```


    [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例无需重启即可自动识别配置服务器复制集合成员的变化。


    ### 重启mongos实例



    版本3.3的变化：使用复制集合配置服务器， [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例指定 [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) 或者 [`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB)设置配置服务器复制集合名称和至少一个复制集合成员。用于分片集群的 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例必须指定同样的配置服务器集合名称，但可以指定不同的复制集合成员。



    如果 一个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例指定了一个 [`--configdb`](https://docs.mongodb.com/manual/reference/program/mongos/#std-option-mongos.--configdb) 或者[`sharding.configDB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-sharding.configDB) 设置中的迁移的复制集合成员，下次重启[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例时要更新配置服务器。


    详情查阅[Start a `mongos` for the Sharded Cluster](https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/#std-label-sharding-setup-start-mongos)。


    ## 迁移分片



    一次迁移一个分片。对于每个分片，伴随本部分的适当过程。



    ### 迁移一个复制集合分片

    要迁移一个分片集合，应分别迁移每个成员。首先迁移非主要成员，最后迁移主成员。


    如果复制集合有两个投票成员，给复制集合添加一个仲裁器，确保该集合在迁移期间拥有大多数投票。你可以在完成迁移之后移除仲裁器。



    #### 迁移一个复制集合分片的成员

    1.关闭 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程，使用 [`shutdown`](https://docs.mongodb.com/manual/reference/command/shutdown/#mongodb-dbcommand-dbcmd.shutdown) 命令确保一个干净的关闭。


    2.移动数据目录（例如 [`dbPath`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-storage.dbPath)）到新的机器。


    3.重启 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程至新的位置。


    4.连接到复制集合的当前主节点。


    5.如果成员的主机名变化了，使用 [`rs.reconfig()`](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#mongodb-method-rs.reconfig) 以更新新的主机名到复制集合配置文档。

    例如，以下一些命令在成员数组的位置2的实例更新主机名：

    `cfg = rs.conf()`
    `cfg.members[2].host = "pocatello.example.net:27018"`
    `rs.reconfig(cfg)`


    想要了解更新配置文档的更多信息，查看 [Examples](https://docs.mongodb.com/manual/reference/method/rs.reconfig/#std-label-replica-set-reconfiguration-usage)。


    6.使用[`rs.conf()`](https://docs.mongodb.com/manual/reference/method/rs.conf/#mongodb-method-rs.conf)确认新的配置。


    7.等待成员回复。使用 [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status)检查成员状态。


    #### 在复制集合分片中迁移主节点



    当迁移复制集合的主节点时，集合必须选择一个新的主节点。这种提交复制集合的故障转移过程设置选举过程期间的读操作或者接收写操作为无效，这通常很快就可以完成。如果可能的话，在维护窗口期间计划迁移。



    1.关闭主节点以启动正常的故障转移进程。为了关闭主节点，应连接到主节点或者使用 [`replSetStepDown`](https://docs.mongodb.com/manual/reference/command/replSetStepDown/#mongodb-dbcommand-dbcmd.replSetStepDown) 命令或者 [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) 方法。以下的例子展示了 [`rs.stepDown()`](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#mongodb-method-rs.stepDown) 方法：

    `rs.stepDown()`


    2.一旦主节点关闭，其他的成员成为主节点状态。为了迁移关闭的主节点，随着迁移一个复制集合分片进程的成员，你可以检查rs.status()的输出以确认状态的变化。

 

    ### 重新打开平衡器


    要完成迁移，重新打开平衡器并恢复 [chunk migrations](https://docs.mongodb.com/manual/core/sharding-balancer-administration/)。


    连接一个集群的[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例，并传递true值给 [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 方法：[[2\]](https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/#footnote-autosplit-start)

    `sh.startBalancer()`


    要检查平衡器状态，使用 [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 方法。


    详情查看 [打开平衡器](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable)。

    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | [2]                                                          | 从 MongoDB 4.2开始， [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 也可以用于分片集群的自动分割。 |



译者：张冲

原文链接：https://docs.mongodb.com/manual/tutorial/migrate-sharded-cluster-to-new-hardware/
