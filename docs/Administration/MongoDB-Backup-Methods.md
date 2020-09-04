# MogoDB 备份方法

在本页

- [使用 Atlas 备份](https://docs.mongodb.com/manual/core/backups/#back-up-with-atlas)
- [使用 MongoDB Cloud 或者 Ops Manager 备份](https://docs.mongodb.com/manual/core/backups/#back-up-with-mms-or-ops-manager)
- [通过拷贝基础数据文件备份](https://docs.mongodb.com/manual/core/backups/#back-up-by-copying-underlying-data-files)
- [使用 `mongodump` 备份](https://docs.mongodb.com/manual/core/backups/#back-up-with-mongodump)

当在生产环境中部署 MongoDB 时，应该制定一种策略，以备在发生数据丢失事件时捕获和还原备份。


## 使用 Atlas 进行备份

MongoDB 官方云服务 MongoDB Atlas 提供2种完全托管的备份方法

1. [连续备份](https://docs.atlas.mongodb.com/backup/continuous-backups)，它对群集中的数据进行增量备份，从而确保备份通常仅比操作系统落后几秒钟。利用 Atlas 连续备份，您可以从存储的快照或最近24小时内的选定时间点还原。您还可以查询连续备份快照。

2. [云提供商快照](https://docs.atlas.mongodb.com/backup/cloud-provider-snapshots),使用集群的云服务提供商的原生快照功能提供的本地化的备份存储。


## 使用 MongoDB Cloud Manage 或者 Ops Manager

MongoDB Cloud Manager 是针对 MongoDB 的托管备份，监控和自动化服务。[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server) 支持用户在图形化界面操作备份和还原 MongoDB [副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)和[分片集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster).


### MongoDB Cloud Manager

[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server) 支持 MongoDB 部署的备份和恢复

通过从 MongoDB 部署中读取[操作日志](https://docs.mongodb.com/manual/reference/glossary/#term-oplog)数据，MongoDB Cloud Manager 持续备份 MongoDB [副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)和[分片群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)。MongoDB Cloud Manager 会按设置的时间间隔创建数据快照，还可以提供 MongoDB 副本集和分片群集的时间点恢复。

提示

使用其他 MongoDB 备份方法很难实现分片群集快照。

要开始使用 MongoDB Cloud Manager 备份，请注册 [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server)。有关 MongoDB Cloud Manager 的文档，请参阅 [MongoDB Cloud Manager 的文档](https://docs.cloudmanager.mongodb.com/)。


### Ops Manager


借助 Ops Manager，MongoDB 用户可以在自己的基础架构上安装和运行驱动 [MongoDB Cloud Manager](https://docs.mongodb.com/manual/core/backups/#backup-with-mms) 的相同核心软件。Ops Manager 是一种本地解决方案，具有与 MongoDB Cloud Manager 相似的功能，可与订阅的企业版高级功能一起使用。

For more information about Ops Manager, see the [MongoDB Enterprise Advanced](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) page and the [Ops Manager Manual](https://docs.opsmanager.mongodb.com/current/).

有关更多 Ops Manager，请看[MongoDB 企业版高级高级功能](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) 和 [Ops Manager 操作手册](https://docs.opsmanager.mongodb.com/current/).


## 通过复制基础数据文件进行备份


使用 AES256-GCM 的加密存储引擎的注意事项

对于使用 `AES256-GCM` 加密模式的[加密存储引擎](https://docs.mongodb.com/manual/core/security-encryption-at-rest/#encrypted-storage-engine)，AES256-GCM 要求每个进程都使用唯一的计数器块值和密钥。

对于配置了 `AES256-GCM` 密码[加密存储引擎](https://docs.mongodb.com/manual/core/security-encryption-at-rest/#encrypted-storage-engine):

- 从热备份还原

  从 4.2 开始，如果您通过“热”备份(即 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 正在运行)获取的文件进行还原，MongoDB 可以在启动时检测“脏”密钥并自动翻转数据库密钥以避免IV（初始化向量）重用。

- 从冷备份还原

  但是, 如果您通过“冷”备份获取的文件恢复(即 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 没有在运行),则MongoDB无法在启动时检测到“脏”密钥，并且IV的重用会使机密性和完整性保证无效。

  从4.2开始, 为了避免从冷的文件系统快照还原后重新使用密钥，MongoDB 添加了一个新的命令行选项 [`--eseDatabaseKeyRollover`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-esedatabasekeyrollover). 使用[`--eseDatabaseKeyRollover`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-esedatabasekeyrollover) 选项启动, [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例将回滚使用 `AES256-GCM` 密码配置的数据库密钥，然后退出。


提示

- In general, if using filesystem based backups for MongoDB Enterprise 4.2+, use the “hot” backup feature, if possible.
- 通常，如果对 MongoDB Enterprise 4.2+ 使用基于文件系统的备份，情尽可能使用“热”备份功能。
- For MongoDB Enterprise versions 4.0 and earlier, if you use `AES256-GCM` encryption mode, do **not** make copies of your data files or restore from filesystem snapshots (“hot” or “cold”).
- 对于 MongoDB Enterprise 4.0 及更早版本，如果您使用 `AES256-GCM` 加密模式，请**不要**复制数据文件或从文件系统快照（“热”或“冷”）还原。


### 使用文件系统快照备份


您可以通过复制MongoDB的基础数据文件来创建MongoDB部署的备份。

如果 MongoDB 存储数据文件的卷支持时间点快照，则可以使用这些快照在确切的时间创建 MongoDB 系统的备份。文件系统快照是操作系统卷管理器功能，并非特定于 MongoDB。借助文件系统快照，操作系统可以获取卷的快照以用作数据备份的基准。快照的机制取决于基础存储系统。例如，在 Linux 上，逻辑卷管理器（LVM）可以创建快照。同样，Amazon 的 EC2 EBS 存储系统支持快照。

要获取正在运行的 mongod 进程的正确快照，您必须启用日记功能，并且日记必须与其他MongoDB 数据文件位于同一逻辑卷上。如果未启用日记功能，则无法保证快照将保持一致或有效。

要获得[分片群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)的一致快照，必须禁用平衡器并在大约同一时间从每个分片以及配置服务器捕获快照。

欲了解更多信息，请参阅[使用文件系统快照备份和恢复](https://docs.mongodb.com/manual/tutorial/backup-with-filesystem-snapshots/) 和 [使用文件系统快照备份分片集群](https://docs.mongodb.com/manual/tutorial/backup-sharded-cluster-with-filesystem-snapshots/)使用 LVM 创建快照的完整说明。


### 使用 `cp` 或者 `rsync` 备份


如果存储系统不支持快照，可以直接使用 cp，rsync 或类似的工具拷贝文件。由于复制多个文件不是原子操作，因此必须[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)在复制文件之前停止对的所有写操作。否则，您将以无效状态复制文件。

通过复制基础数据生成的备份不支持[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)的时间点恢复，并且对于较大的[分片群集](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump)很难管理。此外，这些备份更大，因为它们包括索引以及重复的基础存储填充和碎片。mongodump 相反，创建的备份较小。


## 使用 `mongodump` 备份


[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 从 MongoDB 数据库读取数据，并创建高保真 BSON 文件，该 [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore) 工具可用于填充 MongoDB 数据库。 [`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 和 [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore) 是用于备份和还原小型 MongoDB 部署的简单高效的工具，但是对于捕获大型系统的备份而言并不是理想的选择。

[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 和 [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore)针对正在运行的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 进程进行操作，并且可以直接操作基础数据文件。默认情况下，mongodump 不捕获本地数据库的内容。

[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 仅捕获数据库中的文档。生成的备份是节省空间的，但是[`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore) 或 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)恢复数据后，必须重建索引。

当连接一个 MongoDB 实例时，[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump)可能会对[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)的性能产生不利影响。如果您的数据大于系统内存，则查询会将工作集推出内存，从而导致页面错误。


应用程序可以在 [`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 捕获输出的同时继续修改数据，对于副本集，当进行[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 操作时，[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 提供 [`--oplog`](https://docs.mongodb.com/database-tools/mongodump/#cmdoption-mongodump-oplog) 选项来包括它输出的[oplog](https://docs.mongodb.com/manual/reference/glossary/#term-oplog) 实体。这允许响应的[`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore)恢复捕获的 oplog。要恢复创建时带了[`--oplog`](https://docs.mongodb.com/database-tools/mongodump/#cmdoption-mongodump-oplog)选项的备份，进行[`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore)操作是需要有 [`--oplogReplay`](https://docs.mongodb.com/database-tools/mongorestore/#cmdoption-mongorestore-oplogreplay)选项。


但是对于副本集，请考虑使用 [MongoDB Cloud Manager](https://docs.mongodb.com/manual/core/backups/#backup-with-mms) 或 [Ops Manager](https://docs.mongodb.com/manual/core/backups/#backup-with-mms-onprem)。

注意

[`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) 和 [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore)**不能**作为正在进行分片事务的4.2+版本分片群集的备份策略的一部分，因为使用创建的备份*不会保持*跨分片事务的原子性保证。

对于具有正在进行的分片事务的 4.2+ 版本分片集群，请使用以下一个协调的备份和还原过程，这些过程*确实维护*了跨分片事务的原子性保证：

- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server),
- [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager?tck=docs_server), or 或
- [MongoDB Ops Manager](https://www.mongodb.com/products/ops-manager?tck=docs_server).

有关更多信息请参阅[Back Up and Restore with MongoDB Tools](https://docs.mongodb.com/manual/tutorial/backup-and-restore-tools/) 和 [Back Up a Sharded Cluster with Database Dumps](https://docs.mongodb.com/manual/tutorial/backup-sharded-cluster-with-database-dumps/)

译者：谢伟成

