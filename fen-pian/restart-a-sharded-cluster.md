# 重启一个分片集群 Restart a Sharded Cluster

The tutorial is specific to MongoDB 5.0. For earlier versions of MongoDB, refer to the corresponding version of the MongoDB Manual.

本教程对应的是MongoDB 5.0。对早期版本，可参考相关版本的手册。

This procedure demonstrates the shutdown and startup sequence for restarting a sharded cluster. Stopping or starting the components of a sharded cluster in a different order may cause communication errors between members. For example, [shard](https://docs.mongodb.com/manual/core/sharded-cluster-shards/) servers may appear to hang if there are no [config servers](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/) available.

这一部分演示重启一个分片集群时关闭和启动的顺序。使用不同的顺序停止和开启一个分片集群，可能会导致成员间的通信错误。例如，如果没有配置有效的服务器会导致分片服务器挂起。

<!--**IMPORTANT**-->

<!--重要提示-->

<!--This procedure should only be performed during a planned maintenance period. During this period, applications should stop all reads and writes to the cluster in order to prevent potential data loss or reading stale data.-->

<!--只有在一个计划好的维护周期中才可以实施本操作。在此期间，为避免重要数据的丢失或者读取脏数据，应用程序需要停止所有对集群的读写。-->

## Disable the Balancer

## 禁用平衡器

Disable the balancer to stop [chunk migration](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) and do not perform any metadata write operations until the process finishes. If a migration is in progress, the balancer will complete the in-progress migration before stopping.

禁用平衡器以停止块迁移，在操作完成之前不要进行任何大数据写操作。如果迁移正在进行，平衡器会在停止前完成正在进行的迁移。

To disable the balancer, connect to one of the cluster's [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances and issue the following command: [[1\]](https://docs.mongodb.com/manual/tutorial/restart-sharded-cluster/#footnote-autosplit-stop)

如果要关闭平衡器，连接一个mongos的实例，使用如下命令：

`sh.stopBalancer()`

To check the balancer state, issue the [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) command.

检查平衡器状态，使用[`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 命令。

For more information, see [Disable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily).

要了解更多信息，查看[Disable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-disable-temporarily)。

| [[1](https://docs.mongodb.com/manual/tutorial/restart-sharded-cluster/#ref-autosplit-stop-id1)] | Starting in MongoDB 4.2, [`sh.stopBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.stopBalancer/#mongodb-method-sh.stopBalancer) also disables auto-splitting for the sharded cluster. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [1]                                                          | 自MongoDB 4.2开始，sh.stopBalancer()也可以禁用分片集群的自动分割功能。 |

## Stop Sharded Cluster

## 停止分片集群

### 1.Stop [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) routers

### 1.停止[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 路由

Run [`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) from the `admin` database on each [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) router:

在每个 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 路由上，从admin数据库中运行 [`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) ：

`use admin`
`db.shutdownServer()`

### 2.Stop each shard replica set

## 2.停止每个分片复本集

Run [`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) from the `admin` database on each [shard](https://docs.mongodb.com/manual/core/sharded-cluster-shards/) replica set member to shutdown its [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) process. Shutdown all secondary members before shutting down the primary in each replica set.

在每个分片复制集合成员的admin数据库中运行[`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) ，以关闭其 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程。在每个复制集合关闭主模块之前，关闭所有的二级成员。

### 3.Stop config servers

## 3.停止配置服务器

Run [`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) from the `admin` database on each of the [config servers](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/) to shutdown its [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) process. Shutdown all secondary members before shutting down the primary.

在每个分片复制集合的admin数据库中运行[`db.shutdownServer()`](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#mongodb-method-db.shutdownServer) ，以关闭其 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 进程。在关闭主模块之前，关闭所有的二级成员。

## Start Sharded Cluster

# 启动分片集群

### 1.Start config servers

## 1.启动配置服务器

When starting each [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod), specify the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) settings using either a configuration file or the command line. For more information on startup parameters, see the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) reference page.

启动每个 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)时，使用一个配置文件或者命令行指定 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)的设置。要了解更多的启动参数，查看 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)参考页。

**Configuration File**

##### 配置文件

If using a configuration file, start the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) with the `--config` option set to the configuration file path.

如果使用配置文件，使用 `--config` 选项启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 以配置文件路径。

`mongod --config <path-to-config-file>`

**Command Line**

##### 命令行

If using the command line options, start the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) with the `--configsvr`, `--replSet`, `--bind_ip`, and other options as appropriate to your deployment. For example:

如果使用命令行设置，用 `--configsvr`, `--replSet`, `--bind_ip`选项和其他适合配置的选项启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) ，例如：

`mongod --configsvr --replSet <replica set name> --dbpath <path> --bind_ip localhost,<hostname(s)|ip address(es)>`

After starting all config servers, connect to the primary [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) and run [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status) to confirm the health and availability of each CSRS member.

启动所有的配置服务器之后，连接主mongod，并运行[`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status) 以确认每个CSRS成员的健康和有效。

### 2.Start each shard replica set

## 2.启动每个分片复本集

When starting each [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod), specify the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) settings using either a configuration file or the command line.

启动每个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)时，使用配置文件或命令行指定[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)设置。

**Configuration File**

##### 配置文件

If using a configuration file, start the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) with the `--config` option set to the configuration file path.

如果使用配置文件，使用--config选项配置文件路径并启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)。

`mongod --config <path-to-config-file>`

**Command Line**

命令行

If using the command line option, start the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) with the `--replSet`, `--shardsvr`, and `--bind_ip` options, and other options as appropriate to your deployment. For example:

如果使用命令行，使用`--replSet`, `--shardsvr`和--bind_ip选项和其他相应选项启动mongod，例如：

`mongod --shardsvr --replSet <replSetname> --dbpath <path> --bind_ip localhost,<hostname(s)|ip address(es)>`

After starting all members of each shard, connect to each primary [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) and run [`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status) to confirm the health and availability of each member.

在启动每个分片的所有成员之后，连接到每个主 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)并运行[`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#mongodb-method-rs.status) 以确认每个成员的健康和有效。

### 3.Start [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) routers

## 3.启动[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 路由

Start [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) routers using either a configuration file or a command line parameter to specify the config servers.

使用一个配置文件或者命令行参数指定配置服务器，启动 [`mongos`]路由。

**Configuration File**

##### 配置文件

If using a configuration file, start the [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) specifying the `--config` option and the path to the configuration file.

如果使用配置文件，指定--config选项配置文件路径并启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)。

`mongos --config <path-to-config>`

For more information on the configuration file, see [configuration options](https://docs.mongodb.com/manual/reference/configuration-options/).

要了解更多配置文件信息，查看 [configuration options](https://docs.mongodb.com/manual/reference/configuration-options/)。

**Command Line**

##### 命令行

If using command line parameters, start the [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) and specify the `--configdb`, `--bind_ip`, and other options as appropriate to your deployment. For example:

如果使用命令行参数，指定`--configdb`, `--bind_ip`和其他相应选项启动 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)。例如：

**WARNING**

**警告**

Before binding to a non-localhost (e.g. publicly accessible) IP address, ensure you have secured your cluster from unauthorized access. For a complete list of security recommendations, see [Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/). At minimum, consider [enabling authentication](https://docs.mongodb.com/manual/administration/security-checklist/#std-label-checklist-auth) and [hardening network infrastructure](https://docs.mongodb.com/manual/core/security-hardening/).

绑定到一个非本地IP地址之前，要确保你的集群如果遇到未授权访问的安全性。如果想查看安全建议的完整列表，查看Security Checklist。最少，考虑 [enabling authentication](https://docs.mongodb.com/manual/administration/security-checklist/#std-label-checklist-auth) 和 [hardening network infrastructure](https://docs.mongodb.com/manual/core/security-hardening/)。

`mongos --configdb <configReplSetName>/cfg1.example.net:27019,cfg2.example.net:27019 --bind_ip localhost,<hostname(s)|ip address(es)>`

## Re-Enable the Balancer

# 重新开启平衡器

Re-enable the balancer to resume [chunk migrations](https://docs.mongodb.com/manual/core/sharding-balancer-administration/).

重新开启平衡器以恢复块迁移。

Connect to one of the cluster's [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) instances and run the [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) command: [[2\]](https://docs.mongodb.com/manual/tutorial/restart-sharded-cluster/#footnote-autosplit-start)

连接一个集群的[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 实例并运行[`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 命令：

`sh.startBalancer()`

To check the balancer state, issue the [`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) command.

使用[`sh.getBalancerState()`](https://docs.mongodb.com/manual/reference/method/sh.getBalancerState/#mongodb-method-sh.getBalancerState) 命令检查平衡器状态。

For more information, see [Enable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable).

要了解更多信息，查看[Enable the Balancer](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-balancing-enable)。

| [[2](https://docs.mongodb.com/manual/tutorial/restart-sharded-cluster/#ref-autosplit-start-id3)] | Starting in MongoDB 4.2, [`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) also enables auto-splitting for the sharded cluster. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [2]                                                          | 自MongoDB 4.2开始，[`sh.startBalancer()`](https://docs.mongodb.com/manual/reference/method/sh.startBalancer/#mongodb-method-sh.startBalancer) 也可以开启分片集群的自动分割功能。 |

## Validate Cluster Accessibility

# 验证集群可访问性

Connect a [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell to one of the cluster's [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) processes. Use [`sh.status()`](https://docs.mongodb.com/manual/reference/method/sh.status/#mongodb-method-sh.status) to check the overall cluster status.

连接一个 [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) 命令行到一个集群的[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 进程。使用[`sh.status()`](https://docs.mongodb.com/manual/reference/method/sh.status/#mongodb-method-sh.status) 检查整体的集群状态。

To confirm that all shards are accessible and communicating, insert test data into a temporary sharded collection. Confirm that data is being split and migrated between each shard in your cluster. You can connect a [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell to each shard primary and use [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) to validate that the data was sharded as expected.

要确认所有的分片是可访问的和可通信的，把测试数据插入到一个临时的分片集合当中。在你的集合的每个分片间确认数据正在被分割和迁移。你可以把一个[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) 命令行连接到每个分片的主块区，使用db.collection.find()验证数据是否按预期进行了分片。

**IMPORTANT**

**重要**

To prevent potential data loss or reading stale data, do not start application reads and writes to the cluster until after confirming the cluster is healthy and accessible.

为了避免潜在的数据丢失或读取垃圾数据，在确认集群是健康的和可访问之前，不要开始对集群的应用程序读写。



译者：张冲

时间：2021年8月20日

原文链接：https://docs.mongodb.com/manual/tutorial/restart-sharded-cluster/#stop-each-shard-replica-set