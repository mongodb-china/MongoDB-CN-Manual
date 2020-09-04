# 操作检查表


- [文件系統](https://docs.mongodb.com/manual/administration/production-checklist-operations/#filesystem)

- [复制](https://docs.mongodb.com/manual/administration/production-checklist-operations/#replication)

- [分片](https://docs.mongodb.com/manual/administration/production-checklist-operations/#sharding)

- [日志：WiredTiger存储引擎](https://docs.mongodb.com/manual/administration/production-checklist-operations/#journaling-wiredtiger-storage-engine)

- [硬件](https://docs.mongodb.com/manual/administration/production-checklist-operations/#hardware)

- [部署到云硬件](https://docs.mongodb.com/manual/administration/production-checklist-operations/#deployments-to-cloud-hardware)

- [操作系统配置](https://docs.mongodb.com/manual/administration/production-checklist-operations/#operating-system-configuration)

- [备份](https://docs.mongodb.com/manual/administration/production-checklist-operations/#backups)

- [监控](https://docs.mongodb.com/manual/administration/production-checklist-operations/#monitoring)

- [负载均衡](https://docs.mongodb.com/manual/administration/production-checklist-operations/#load-balancing)


下面的清单和[开发清单](https://docs.mongodb.com/manual/administration/production-checklist-development/)列表一同提供了一些建议，帮助您避免生产环境下MongoDB部署中的问题。
    

## 文件系统

- 将磁盘分区与RAID配置对齐。

- 避免对  [dbPath](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath) 使用 NFS 驱动器。使用 NFS 驱动器可能导致性能下降和不稳定。

 有关详细信息，请参阅：[远程文件系统](https://docs.mongodb.com/manual/administration/production-notes/#production-nfs) 。                        

  - VMware 用户应该通过 NFS 使用 VMware 虚拟驱动器。

- Linux/Unix：将驱动器格式化为 XFS 或 EXT4。如果可能的话，使用 XFS，因为它通常在MongoDB 中运行得更好。

  - 对于 WiredTiger 存储引擎，强烈建议使用XFS，以避免在将 EXT4 与 WiredTiger 一起使用时产生性能问题。
  - 如果使用 RAID，可能需要使用 RAID 几何阵列配置 XFS。

- Windows：使用 NTFS 文件系统。不要使用任何 FAT 文件系统（例如 FAT 16/32/exFAT）。


## 复制

- 验证所有非隐藏副本集成员在 RAM，CPU，磁盘，网络设置等方面的配置是否相同。

- [配置 oplog 的大小](https://docs.mongodb.com/manual/tutorial/change-oplog-size/) 以适合您的使用案例：    

  - 复制 oplog 窗口包括正常维护和停机时间窗口，以避免需要完全重新同步。

  - 复制 oplog 窗口应涵盖从上次备份还原副本集成员所需的时间。

*在 3.4 版本中更改*：复制 oplog 窗口不再需要覆盖通过初始同步还原副本集成员所需的时间，因为在数据复制期间会提取 oplog 记录。但是，正在还原的成员必须在[本地](https://docs.mongodb.com/manual/reference/local-database/#replica-set-local-database)数据库中具有足够的磁盘空间，以便在此数据复制阶段的持续时间内临时存储这些 oplog 记录。

对于早期版本的 MongoDB，复制 oplog 窗口应涵盖通过初始同步还原副本集成员所需的时间。

- 确保您的副本集至少包含三个数据承载节点，这些节点与日志记录一起运行，并且为了可用性和持久性，您使用 w:"majority" [写策略](https://www.docs4dev.com/docs/zh/mongodb/v3.6/reference/reference-write-concern.html)发出写操作。

- 配置副本集成员时使用主机名，而不是IP地址。
- 确保所有 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例之间的完全双向网络连接。
- 确保每个主机都可以自行解决。
- 确保副本集包含奇数个投票成员。
- 确保 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例有0票或1票。
- 对[高可用性](https://docs.mongodb.com/manual/reference/glossary/#term-high-availability)，将副本集部署到至少三个数据中心。


## 分片

- 将 [配置服务器](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/)放在专用硬件上，以便在大型集群中获得最佳性能。确保硬件有足够的 RAM 将数据文件完全保存在内存中，并且有专用的存储器。
- 根据[生产配置](https://docs.mongodb.com/manual/core/sharded-cluster-components/#sc-production-configuration)指南部署 [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 前端路由。
- 使用NTP来同步切分集群所有组件上的时钟。
- 确保 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod), [mongos ](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)和配置服务器之间的完全双向网络连接。
- 使用 CNAMEs 将配置服务器标识到集群，以便可以在不停机的情况下重命名和重新编号配置服务器。


## 日志：WiredTiger存储引擎

- 确保所有实例都使用[日志](https://docs.mongodb.com/manual/core/journaling/)。
- 将日志放在其自己的低延迟磁盘上，以适应写密集型的工作负载。请注意，这将影响快照样式备份，因为构成数据库状态的文件将位于单独的卷上。


## 硬件

- 使用 RAID10 和 SSD 驱动器可获得最佳性能。
- SAN 和虚拟化：
  - 确保每个[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 已为其 [数据库文件存储路径](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath)配置了 IOPS，或者具有自己的物理驱动器或 LUN。
  - 在虚拟环境中运行时，请避免使用动态内存特性，如内存膨胀。
  - 避免将所有副本集成员放在同一个 SAN 上，因为 SAN 可能是单点故障。


## 部署到云硬件

- Windows Azure：将 TCP 长连接（TCP长连接时间）调整为100-120。Azure 负载均衡器上的 TCP 空闲超时对于 MongoDB 的连接池行为太慢。有关详细信息，请参阅 [Azure产品说明](https://docs.mongodb.com/manual/administration/production-notes/#windows-azure-production-notes)。
- 在具有高延迟存储的系统（如Microsoft Azure）上使用 MongoDB 版本 2.6.4 或更高版本，因为这些版本包括这些系统性能的改进。


## 操作系统配置

### Linux

- 关闭透明大页。有关更多信息，请参见[透明大页设置](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/)。

- 在存储数据库文件的设备上[调整文件预读设置](https://docs.mongodb.com/manual/administration/production-notes/#readahead) 。

  - 对于 WiredTiger 存储引擎，无论存储介质类型（旋转磁盘、固态硬盘等）如何，请将文件预读设置在8到32之间，除非测试显示在较高的文件预读值中有可测量、可重复和可靠的好处。

    [MongoDB专业支持](https://support.mongodb.com/welcome?jmp=docs) 可以提供关于交替文件预读配置的建议和指导。

- 如果在 RHEL/CentOS 上使用 tuned（动态内核调优工具），则必须自定义您的 tuned 配置文件。RHEL/CentOS 附带的许多 tuned 文件可能会对其默认设置的性能产生负面影响。将您选择的 tuned 文件自定义为：                                                                   

  - 禁用透明大页。有关说明，请参见[使用 tuned 和 ktune](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/#configure-thp-tuned)。
  - 无论存储介质类型如何，都将文件预读设置为8到32之间。有关详细信息，请参阅[预读设置](https://docs.mongodb.com/manual/administration/production-notes/#readahead)。

- 对SSD驱动器使用 noop 或 deadline 磁盘调度程序。

- 对来宾虚拟机中的虚拟化驱动器使用 noop 磁盘调度程序。

- 禁用 NUMA 或将 vm.zone_reclaim_mode 设置为0并运行具有节点交错的 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例。请参阅：[MongoDB和NUMA硬件](https://docs.mongodb.com/manual/administration/production-notes/#production-numa)了解更多信息。

- 调整硬件上的 ulimit 值以适合您的用例。如果多个 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 或者 [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例在同一用户下运行，请相应地缩放 ulimit 值。有关详细信息，请参见：[UNIX ulimit 设置](https://docs.mongodb.com/manual/reference/ulimit/)。

- 使用noatime作为 [dbPath](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath) 挂载点。

- 为部署配置足够的文件句柄（fs.file max）、内核 pid 限制（kernel.pid_max）、每个进程的最大线程数（kernel.threads max）和每个进程的最大内存映射区域数（vm.max_map_count）。对于大型系统，以下值提供了一个良好的起点：

  - fs.file-max 值为98000,
  - kernel.pid_max 值为64000,
  - kernel.threads-max 值为64000, 和
  - vm.max_map_count 值为128000

- 确保系统已配置交换空间。有关适当大小的详细信息，请参阅操作系统的文档。

- 确保系统默认的 TCP 长连接设置正确。TCP 长连接时间值300通常为副本集和分片集群提供更好的性能。有关详细信息，请参阅常见问题中的 [TCP 保持时间是否影响MongoDB部署?](https://docs.mongodb.com/manual/faq/diagnostics/#faq-keepalive) 。

### Window

- 考虑禁用 NTFS “最后访问时间”更新。这类似于在 Unix-like 系统上禁用atime。
- 使用默认分配单元大小的[4096 字节](https://support.microsoft.com/en-us/help/140365/default-cluster-size-for-ntfs-fat-and-exfat)格式化NTFS磁盘。


## 备份

- 安排定期测试备份和恢复过程，以便手头有时间估计，并验证其功能。


## 监控

- 使用 [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?jmp=docs)或者[MongoDB 企业高级版中提供的本地解决方案- Ops Manager ](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs) 或者另一个监控系统来监控关键数据库指标并为它们设置警报。包括以下指标的警报:

  - 复制滞后
  - 复制 oplog 窗口
  - 断言
  - 队列
  - 页面错误

- 监视服务器的硬件统计信息。尤其要注意磁盘使用、CPU 和可用磁盘空间。

  在没有磁盘空间监视的情况下，以下方案作为预防措施：

  - 在storage.dbPath驱动器上创建一个4 GB的虚拟文件，以确保磁盘满时有可用空间。
  - 如果没有其他监视工具可用，cron+df 的组合可以在磁盘空间达到高水位时发出警报。


## 负载均衡

- 将负载平衡器配置为启用“粘滞会话”或“客户端亲和性”，并为现有连接提供足够的延时。
- 避免在 MongoDB 集群或副本集组件之间放置负载平衡器。




原文链接：https://docs.mongodb.com/manual/administration/production-checklist-operations/

译者：孔令升
