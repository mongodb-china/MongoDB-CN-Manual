# [产品说明](https://docs.mongodb.com/manual/administration/production-notes/ "Permalink to this headline")



- [MongoDB 二进制文件](https://docs.mongodb.com/manual/administration/production-notes/#mongodb-binaries)

- [MongoDB 文件存储路径](https://docs.mongodb.com/manual/administration/production-notes/#mongodb-dbpath)

- [并发](https://docs.mongodb.com/manual/administration/production-notes/#concurrency)

- [数据一致性](https://docs.mongodb.com/manual/administration/production-notes/#data-consistency)

- [联网](https://docs.mongodb.com/manual/administration/production-notes/#networking)

- [硬件注意事项](https://docs.mongodb.com/manual/administration/production-notes/#hardware-considerations)

- [架构](https://docs.mongodb.com/manual/administration/production-notes/#architecture)

- [压缩](https://docs.mongodb.com/manual/administration/production-notes/#compression)

- [时钟同步](https://docs.mongodb.com/manual/administration/production-notes/#clock-synchronization)

- [平台特定注意事项](https://docs.mongodb.com/manual/administration/production-notes/#platform-specific-considerations)

- [性能监控](https://docs.mongodb.com/manual/administration/production-notes/#performance-monitoring)

- [备份](https://docs.mongodb.com/manual/administration/production-notes/#backups)

  
本文详细描述了影响MongoDB，特别是在生产环境中运行时的系统配置。

移除 MMAPV1：

MongoDB 4.2 移除了已弃用的 MMAPv1 存储引擎。要将 MMAPv1 存储引擎部署更改为  [WiredTiger存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/) ，请参见：

- [将单节点更改为WiredTiger](https://docs.mongodb.com/manual/tutorial/change-standalone-wiredtiger/)
- [将复制集群更改为WiredTiger](https://docs.mongodb.com/manual/tutorial/change-replica-set-wiredtiger/)
- [将分片集群更改为WiredTiger](https://docs.mongodb.com/manual/tutorial/change-sharded-cluster-wiredtiger/)


注意：


[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是云端的数据库服务。[MongoDB Cloud Manager：官方推出的运维自动化管理系统](https://www.mongodb.com/cloud/cloud-manager/?jmp=docs)， 是一个托管服务； [Ops Manager：用于监控和备份MongoDB的基础设施服务](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)，是一个本地解决方案, 提供 MongoDB 实例的监视，备份和自动化。 有关文档，请参阅 [Atlas 文档](https://docs.atlas.mongodb.com/?jmp=docs), [MongoDB Cloud Manager 文档](https://docs.cloudmanager.mongodb.com/) 和 [Ops Manager 文档](https://docs.opsmanager.mongodb.com/?jmp=docs)。



## MongoDB 二进制文件


### 支持的平台

**在生产环境中**运行时，请参阅[推荐的平台](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-recommended-platforms)以获取推荐使用的操作系统。

注意：

MongoDB 4.0 在 macOS 10.12.x 和 10.13.x 系统上当硬盘未正常关机时可能丢失数据。

对于更多的细节，参见[WT-4018](https://jira.mongodb.org/browse/WT-4018)。


#### x86_64


**平台支持的产品生命期结束通知**

|              |                              |
| :----------: | :--------------------------: |
| Ubuntu 14.04 | 在MongoDB 4.2+中移除了支持。 |
|   Debian 8   | 在MongoDB 4.2+中移除了支持。 |
| macOS 10.11  | 在MongoDB 4.2+中移除了支持。 |


*即将到来的的产品生命期结束通知*

|                    |                                   |
| :----------------: | :-------------------------------: |
| Windows 8.1/2012R2 | MongoDB在接下来的版本中不再支持。 |
|   Windows 8/2012   | MongoDB在接下来的版本中不再支持。 |
|  Windows 7/2008R2  | MongoDB在接下来的版本中不再支持。 |



|                             平台                             | 4.2 社区版和企业版 | 4.0 社区版和企业版 | 3.6 社区版和企业版 | 3.4 社区版和企业版 |
| :----------------------------------------------------------: | :----------------: | :----------------: | :----------------: | :----------------: |
|                        亚马逊 Linux 2                        |         ✓          |         ✓          |                    |                    |
|               亚马逊 Linux 2013.03 和更高版本                |         ✓          |         ✓          |         ✓          |         ✓          |
|                          Debian 10                           |       4.2.1+       |                    |                    |                    |
|                           Debian 9                           |         ✓          |         ✓          |       3.6.5+       |                    |
|                           Debian 8                           |                    |         ✓          |         ✓          |         ✓          |
| RHEL/CentOS/Oracle Linux [[1\]](https://docs.mongodb.com/manual/administration/production-notes/#oracle-linux) 8.0 and later |       4.2.1+       |      4.0.14+       |      3.6.17+       |                    |
| RHEL/CentOS/Oracle Linux [[1\]](https://docs.mongodb.com/manual/administration/production-notes/#oracle-linux) 7.0 和更高版本 |         ✓          |         ✓          |         ✓          |         ✓          |
| RHEL/CentOS/Oracle Linux [[1\]](https://docs.mongodb.com/manual/administration/production-notes/#oracle-linux) 6.2 和更高版本 |         ✓          |         ✓          |         ✓          |         ✓          |
|                           SLES 15                            |       4.2.1+       |                    |                    |                    |
|                           SLES 12                            |         ✓          |         ✓          |         ✓          |         ✓          |
|                      Solaris 11 64-bit                       |                    |                    |                    |     仅社区版本     |
|                         Ubuntu 18.04                         |         ✓          |       4.0.1+       |                    |                    |
|                         Ubuntu 16.04                         |         ✓          |         ✓          |         ✓          |         ✓          |
|                         Ubuntu 14.04                         |                    |         ✓          |         ✓          |         ✓          |
|                     Windows Server 2019                      |         ✓          |                    |                    |                    |
|                   Windows 10 / Server 2016                   |         ✓          |         ✓          |         ✓          |         ✓          |
|                 Windows 8.1 / Server 2012 R2                 |         ✓          |         ✓          |         ✓          |         ✓          |
|                   Windows 8 / Server 2012                    |         ✓          |         ✓          |         ✓          |         ✓          |
|                  Windows 7 / Server 2008 R2                  |         ✓          |         ✓          |         ✓          |         ✓          |
|                        Windows Vista                         |                    |                    |                    |         ✓          |
|                    macOS 10.13 和更高版本                    |         ✓          |         ✓          |                    |                    |
|                         macOS 10.12                          |         ✓          |         ✓          |         ✓          |         ✓          |
|                         macOS 10.11                          |                    |         ✓          |         ✓          |         ✓          |
|                         macOS 10.10                          |                    |                    |         ✓          |         ✓          |


[1]	*([1](https://docs.mongodb.com/manual/administration/production-notes/#id1), [2](https://docs.mongodb.com/manual/administration/production-notes/#id2), [3](https://docs.mongodb.com/manual/administration/production-notes/#id3))* MongoDB 仅支持 Oracle Linux 运行 Red Hat Compatible Kernel (RHCK). MongoDB 不支持Unbreakable Enterprise Kernel (UEK)。


#### ARM64


| 平台支持的 产品生命期结束通知 |                                   |
| :---------------------------: | :-------------------------------: |
|      Ubuntu 16.04 ARM64       | 在MongoDB 社区版 4.2+中不再支持。 |



| 平台         | 4.2 社区版和企业版 | 4.0 社区版和企业版 | 3.6 社区版和企业版 | 3.4 社区版和企业版 |
| :----------- | :----------------: | :----------------: | :----------------: | :----------------: |
| Ubuntu 18.04 |      仅社区版      |                    |                    |                    |
| Ubuntu 16.04 |      仅企业版      |         ✓          |         ✓          |         ✓          |

#### PPC64LE (MongoDB 企业版) 


| 平台支持的 产品生命期结束通知 |                                   |
| :---------------------------: | :-------------------------------: |
|     Ubuntu 16.04 PPC64LE      | 在MongoDB 社区版 4.2+中不再支持。 |



|     平台      | 4.2 企业版 | 4.0 企业版 |       3.6 企业版       |       3.4 企业版       |
| :-----------: | :--------: | :--------: | :--------------------: | :--------------------: |
| RHEL/CentOS 7 |     ✓      |     ✓      |           ✓            |           ✓            |
| Ubuntu 18.04  |     ✓      |            |                        |                        |
| Ubuntu 16.04  |            |     ✓      | 在3.6.13版本中开始移除 | 在3.4.21版本中开始移除 |


#### s390x


| 平台          | 4.2 社区版和企业版 | 4.0 企业版 |       3.6 企业版       |       3.4 企业版       |
| :------------ | :----------------: | :--------: | :--------------------: | :--------------------: |
| RHEL/CentOS 7 |         ✓          |   4.0.6+   | 在3.6.2版本中开始移除  | 在3.4.14版本中开始移除 |
| RHEL/CentOS 6 |         ✓          |     ✓      | 在3.6.14版本中开始移除 | 在3.4.22版本中开始移除 |
| SLES12        |         ✓          |   4.0.6+   | 在3.6.2版本中开始移除  | 在3.4.15版本中开始移除 |
| Ubuntu 18.04  |       4.2.1+       |   4.0.6+   |                        |                        |


### 推荐的平台


虽然 MongoDB 支持各种平台，但建议使用以下操作系统使用产品：

- Amazon Linux 2
- Debian 9 and 10                                                                                                                                                               
- RHEL / CentOS 6, 7, and 8
- SLES 12 and 15
- Ubuntu LTS 16.04 and 18.04
- Windows Server 2016 and 2019

另见：

[平台特定注意事项](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-platform-considerations)


### 使用最新的稳定包

确保您拥有最新的稳定版本。

所有 MongoDB 版本都可在 [MongoDB 下载中心](https://www.mongodb.com/download-center/community?jmp=docs) 页面获取.  [MongoDB 下载中心](https://www.mongodb.com/download-center/community?jmp=docs) 页面可以找到当前稳定版本，即使您通过包管理进行安装。

对于其他 MongoDB 产品，请参阅 [MongoDB 下载中心](https://www.mongodb.com/download-center/community?jmp=docs) 页面或者 [各自对应文档](https://docs.mongodb.com/?jmp=docs)。


## MongoDB 文件存储路径

[dbPath](https://www.docs4dev.com/docs/zh/mongodb/v3.6/reference/reference-configuration-options.html#storage.dbPath)目录中的文件必须与配置的[存储引擎](https://docs.mongodb.com/manual/reference/glossary/#term-storage-engine)对应。如果[文件存储路径](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath) 包含由 [--storageEngine](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-storageengine) 指定的存储引擎以外的存储引擎创建的数据文件，[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 将不会启动。

[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)必须对指定的[文件存储路径](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath)拥有读写权限


## 并发

### WiredTiger

[WiredTiger](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger)支持读写器对对集合中的文档进行并发访问。 客户端可以可以在进行写操作时读取文档，多个线程可以同时修改集合中的不同文档。

也可以看看

[分配足够的 RAM 和 CPU](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-ram) 提供有关WiredTiger如何利用多个CPU核以及如何提高操作吞吐量的信息。


## 数据一致性

### 日志

MongoDB 使用预写式日志方式写入到磁盘[日志](https://docs.mongodb.com/manual/reference/glossary/#term-journal)。日志记录保证MongoDB可以快速从崩溃或其他严重错误中恢复写入日志但未写入数据文件的 [写操作](https://docs.mongodb.com/manual/crud/)。

从MongoDB 4.0开始，不能为使用WiredTiger存储引擎的副本集成员 [--nojournal](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-nojournal) 选项或者[storage.journal.enabled: false](https://docs.mongodb.com/manual/reference/configuration-options/#storage.journal.enabled) 


### 读操作安全机制

*在 version 3.2 中的新内容*  

从 MongoDB 3.6 开始，如果写请求确认，则可以使用[因果一致性会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)来读取您自己的写入。

在 MongoDB 3.6 之前，您必须确保写操作使用了 [{ w: "majority}" ](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写入安全机制，然后对读取操作使用 ["majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")或 ["linearizable"](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern."linearizable")读取安全机制，以确保单个线程可以读取自己的写入。

要使用 ["majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")的级别的 [读安全机制](https://docs.mongodb.com/manual/reference/glossary/#term-read-concern) ，副本集必须使用[WiredTiger 存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger)。

您可以禁用具有三个成员的主-副-仲裁(PSA)体系结构部署的读安全机制 ["majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority");但是，这对更改流(仅在MongoDB 4.0和更早的版本中)和分片集群上的事务有影响。有关更多信息，请参见[Disable Read Concern Majority](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority)。

### 写操作安全机制

写操作安全机制](https://docs.mongodb.com/manual/reference/write-concern/) 描述 MongoDB 写操作时确认请求写入的安全机制级别。写操作安全机制的级别会影响写操作返回的速度。当写操作具有较弱的写入安全机制时，它们会快速返回。对于更强的写入安全机制，客户端必须在发送写入操作后等待，直到 MongoDB 在请求的写入安全机制级别上确认写入操作。由于写入安全机制级别不够，写操作可能会显示客户端成功，但在某些服务器故障情况下可能不会缓存。

有关选择适当的写操作安全机制级别的详细信息，请参阅 [写操作安全机制](https://docs.mongodb.com/manual/reference/write-concern/)文档。
                                                                          

## 联网

### 使用可信网络环境

始终在可信环境中运行 MongoDB，其网络规则阻止从所有未知计算机，系统和网络中进行访问。与依赖于网络访问的任何敏感系统一样，只有需要访问的特定系统才能访问 MongoDB 部署，例如应用服务器，监视服务和其他 MongoDB 组件。

重要

默认情况下，[授权](https://docs.mongodb.com/manual/core/authorization/)未启用， [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)默认为受信任的环境。根据需要启用[authorization](https://docs.mongodb.com/manual/reference/configuration-options/#security.authorization) 模式。有关 MongoDB 中支持的身份验证机制以及 MongoDB 中的授权的详细信息，请参阅 [授权](https://docs.mongodb.com/manual/core/authentication/)和 [基于角色的访问控制](https://docs.mongodb.com/manual/core/authorization/)。

有关安全性的其他信息和注意事项，请参阅[安全部分](https://docs.mongodb.com/manual/security/)中的文档，具体如下：

- [安全检查列表](https://docs.mongodb.com/manual/administration/security-checklist/)
- [网络和配置强化](https://docs.mongodb.com/manual/core/security-hardening/)

对于 Windows 用户，在 Windows 上部署 MongoDB 时请考虑 [有关TCP配置的Windows Server Technet文章 ](http://technet.microsoft.com/en-us/library/dd349797.aspx)。

### 禁用 HTTP 接口

3.6版本中的变化： MongoDB 3.6 移除了 HTTP 接口和 REST API 。

早期版本的 MongoDB 提供了一个 HTTP 接口来检查服务器的状态，还可以选择运行查询。默认情况下禁用 HTTP 接口。不要在生产环境中启用 HTTP 接口。


### 管理连接池大小

通过调整连接池大小以适合您的用例，避免重载 [mongod ](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和 [mongos ](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例的连接资源。从当前数据库请求的典型数量的 110-115％开始，并根据需要修改连接池大小。请参阅[连接池选项](https://docs.mongodb.com/manual/reference/connection-string/#connection-pool-options)以调整连接池大小。

[connPoolStats](https://docs.mongodb.com/manual/reference/command/connPoolStats/#dbcmd.connPoolStats) 命令返回有关分片集群中[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 和 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例的当前数据库打开连接数的信息。 

另见 [分配足够的 RAM 和 CPU](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-ram).

### 硬件考虑因素

MongoDB 专为商用硬件而设计，几乎没有硬件要求或限制。 MongoDB 的核心组件运行在小端硬件上，主要是 x86/x86_64 处理器。客户端库（例如驱动程序）可以在大端或小端系统上运行。

### 分配足够的 RAM 和 CPU

至少，确保每个 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 或者 [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例可以访问两个实核或一个多核物理CPU。

#### WiredTiger

[WiredTiger](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger) 存储引擎是多线程的，可以利用额外的 CPU 内核。具体而言，相对于可用CPU的数量，活动线程（即并发操作）的总数会影响性能：

- 随着并发活动操作数量增加到 CPU 数量，吞吐量会增加。
- 当并发活动操作的数量超过CPU数量的某个阈值时，吞吐量会降低。

阈值取决于您的应用程序。您可以通过实验和测量吞吐量来确定应用程序的最佳并发活动操作数。 [mongostat](https://docs.mongodb.com/manual/reference/program/mongostat/#bin.mongostat) 的输出提供（ar | aw）列中活动读/写次数的统计信息。

使用 WiredTiger，MongoDB同时使用WiredTiger内部缓存和文件系统缓存。

从MongoDB 3.4开始，默认的WiredTiger内部缓存大小为以下两者中的较大者：

- 50% 的 (RAM - 1 GB), 或者
- 256 MB。
              

例如，在总共有4GB RAM的系统上，WiredTiger缓存将使用1.5GB RAM(0.5 * (4 GB - 1 GB) = 1.5 GB)。相反，RAM总量为1.25GB的系统将为WiredTiger缓存分配256MB，因为这超过了RAM总量减去1GB（0.5*（1.25GB-1GB）=128MB<256MB) 的一半。

                                                                                                                                                                        
注意


在某些情况下，例如在容器中运行时，数据库可能具有低于总系统内存的内存约束。在这种情况下，这个内存限制，而不是整个系统内存，被用作可用的最大RAM。                             

要查看内存限制，请参阅 [hostInfo.system.memLimitMB](https://docs.mongodb.com/manual/reference/command/hostInfo/#hostInfo.system.memLimitMB)。                                   

默认情况下，WiredTiger对所有集合使用snapy块压缩，对所有索引使用前缀压缩。压缩默认值在全局级别上是可配置的，也可以在集合和索引创建期间根据每个集合和每个索引进行设置。

WiredTiger内部缓存中的数据与磁盘上的格式相比使用了不同的表示：

- 文件系统缓存中的数据与磁盘上的格式相同，包括对数据文件进行任何压缩的好处。操作系统使用文件系统缓存来减少磁盘I/O。

- 加载在WiredTiger内部缓存中的索引与磁盘上的格式具有不同的数据表示形式，但仍然可以利用索引前缀压缩来减少RAM的使用。索引前缀压缩从索引字段中删除常用前缀。

- WiredTiger内部缓存中的收集数据是未压缩的，使用与磁盘格式不同的表示形式。块压缩可以显著节省磁盘存储空间，但数据必须解压缩才能由服务器操作。

MongoDB通过文件系统缓存自动使用WiredTiger缓存或其他进程未使用的所有可用内存。


要调整WiredTiger内部缓存的大小，请参见 [storage.wiredTiger.engineConfig.cacheSizeGB](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB) 和 [--wiredTigerCacheSizeGB](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-wiredtigercachesizegb)。避免将WiredTiger内部缓存大小增加到其默认值以上。

注意

[storage.wiredTiger.engineConfig.cacheSizeGB](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB) 限制了wiredTiger内部缓存的大小。操作系统将使用可用的空闲内存进行文件系统缓存，这将允许压缩的MongoDB数据文件保留在内存中。此外，操作系统将使用任何空闲RAM缓冲文件系统块和文件系统缓存。

为了适应RAM的其他使用者，您可能必须减小WiredTiger内部缓存的大小。

默认的WiredTiger内部缓存大小值假定每台计算机有一个[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。如果一台机器包含多个MongoDB实例，那么您应该减少设置以适应其他[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。

如果在无法访问系统中所有可用RAM的容器（例如lxc、cgroups、Docker等）中运行[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)，则必须将 [storage.wiredTiger.engineConfig.cacheSizeGB](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)设置为小于容器中可用RAM的值。具体数量取决于容器中运行的其他进程。参见 [memLimitMB](https://docs.mongodb.com/manual/reference/command/hostInfo/#hostInfo.system.memLimitMB)。

要查看缓存和逐出率的统计信息，请参阅从[serverStatus](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) 命令返回的 [wiredTiger.cache](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.wiredTiger.cache) 字段。

#### 压缩和加密

当使用加密时，配备AES-NI指令集扩展的CPU可以显示出显著的性能优势。如果将MongoDB 企业版与 [加密存储引擎](https://docs.mongodb.com/manual/core/security-encryption-at-rest/#encrypted-storage-engine)一起使用，请选择支持AES-N指令集的CPU以获得更好的性能。

也可以看看                                                                    

[并发](https://docs.mongodb.com/manual/administration/production-notes/#prod-notes-concurrency)


### 使用固态硬盘（SSD）

MongoDB使用SATA SSD能得到很好的效果和很好的性价比。

在可用且经济的情况下请使用SSD。                                                                                        

传统硬盘通常也是个好的选择，因为使用更昂贵的硬盘来提高随机IO性能并不是那么有效（只能是每次2倍）。使用SSD或增加RAM的容量可能对于提升IO更有效率。

### MongoDB和NUMA硬件

在运行NUMA的系统中运行MongoDB可能造成一系列问题，包括一段时间内的效率低下和高系统进程使用率。

当在NUMA硬件上运行MongoDB服务器和客户端时，应配置内存交错策略，以便主机以非NUMA方式运行。MongoDB在Linux（2.0版以后）和Windows（2.6版以后）机器上部署时，会在启动时检查NUMA设置。如果NUMA配置可能会降低性能，MongoDB会打印一个警告。
                                                                                                                                                                        
也可以看看

- [MySQL的 “疯狂交换” 问题和 NUMA的影响](http://jcole.us/blog/archives/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/) 报告, ，它描述了NUMA对数据库造成的影响。这篇文章介绍了NUMA和它的目标，并指出了为什么这些目标和生产环境数据库的需求是不相容的。尽管这篇博文讨论了NUMA对于 MySQL的影响，但是MongoDB的问题是相似的。
- [NUMA: 综述](https://queue.acm.org/detail.cfm?id=2513149)。


#### 在 Windows 上配置 NUMA
                                                  

在 Windows 上，必须通过机器的 BIOS 启用内存交叉存取。有关详细信息，请参阅系统文档

#### 在 Linux 上配置 NUMA                                                                                                  


在 Linux上，您必须禁用内存区域回收，并确保您的 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) and [mongos ](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例由 numactl命令启动，numactl 通常是通过平台的 init 系统配置的。您必须执行这两个操作才能正确禁用 NUMA 以便与 MongoDB 一起使用。

1. 使用以下命令之一禁用内存区域回收:

   ```
echo 0 | sudo tee /proc/sys/vm/zone_reclaim_mod
   ```

   ```
sudo sysctl -w vm.zone_reclaim_mode=0
   ```

2.然后，您应该使用 numactl 来启动 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) and [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) ，这通常是通过平台的 init 系统配置的。运行以下命令以确定平台上正在使用的init系统：

   ```
   ps --no-headers -o comm 1
   ```

   - 如果是systemd，则您的平台使用 systemd init 系统，您必须按照下面 systemd 选项卡中的步骤来编辑MongoDB服务文件。
   - 如果是init，则平台使用SysV init系统，不需要执行此步骤。SysV init 的默认MongoDB init 脚本默认包含通过numactl 启动 MongoDB 实例的必要步骤。
   - 如果您管理自己的 init 脚本（例如没有使用这两个 init 系统中的任何一个），则必须按照下面自定义 init 脚本选项卡中的步骤编辑自定义 init 脚本。

##### systemd

你必须使用 numactl 启动每个 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例,包括所有 [配置服务器](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/), [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例,和客户端.。如下所示编辑每个系统的默认 systemd 服务文件：

   1. 复制默认MongoDB服务文件：

      ```
   sudo cp /lib/systemd/system/mongod.service /etc/systemd/system/
      ```


 2. 编辑 /etc/systemd/system/mongod.service 文件，首先要更新 ExecStart 语句：
   
      ```
   /usr/bin/numactl --interleave=all
      ```
      

例如

如果现有的 ExecStart 语句为：

      ```
   ExecStart=/usr/bin/mongod --config /etc/mongod.conf
      ```


将该语句更新为：


      ```
   ExecStart=/usr/bin/numactl --interleave=all /usr/bin/mongod --config /etc/mongod.conf
      ```

   3. 将更改应用于 systemd：

   ```
      sudo systemctl daemon-reload
   ```
   
   4. 重新启动任何正在运行的 mongod 实例：

      ```
      sudo systemctl stop mongod
      sudo systemctl start mongod
      ```

   5. 如果适用，对任何[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 重复这些步骤。
                                                       

有关更多信息，请参见 [Documentation for /proc/sys/vm/*](http://www.kernel.org/doc/Documentation/sysctl/vm.txt)。

##### 自定义初始化脚本

你必须使用 numactl 启动每个 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例,包括所有 [配置服务器](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/), [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例和客户端。


1.如果尚未安装numactl，请为您的平台安装 numactl。有关安装 numactl 包的信息，请参阅操作系统的文档。

2. 配置每个自定义init脚本以通过numactl启动每个MongoDB实例：

   ```
   numactl --interleave=all <path> <options>
   ```

其中是 <path> 是要启动的程序的路径，也是要传递给该程序的任何可选参数。

例如：

   ```
   numactl --interleave=all /usr/local/bin/mongod -f /etc/mongod.conf
   ```

有关更多信息，请参见 [Documentation for /proc/sys/vm/*](http://www.kernel.org/doc/Documentation/sysctl/vm.txt)。


### 磁盘和存储系统

#### 交换


MongoDB在可以避免交换或将交换保持在最低限度的地方表现最好，因为从交换中检索数据总是比访问RAM中的数据慢。但是，如果托管 MongoDB 的系统没有RAM，交换可以防止 Linux OOM Killer 终止 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 进程。

通常，您应该选择以下交换策略之一：

1. 在系统上分配交换空间，并将内核配置为只允许在高内存负载下进行交换，或者
2. 不要在系统上分配交换空间，并将内核配置为完全禁用交换

请参阅 [Set vm.swappiness](https://docs.mongodb.com/manual/administration/production-notes/#set-swappiness) 以获取有关在Linux系统上按照这些指导原则配置swap的说明。

注意

如果MongoDB实例托管在同时运行其他软件（如Web服务器）的系统上，则应选择第一个交换策略。在这种情况下不要禁用交换。如果可能，强烈建议您在MongoDB自己的专用系统上运行MongoDB。

#### 阵列

为了在存储层方面实现最佳性能，请使用 RAID-10 支持的磁盘。 RAID-5 和 RAID-6 通常不提供足够的 性能来支持 MongoDB 部署。

#### 远程文件系统

使用 WiredTiger 存储引擎，如果远程文件系统符合 ISO/IEC 9945-1:1996(POSIX.1)，则 WiredTiger 对象 可以存储在远程文件系统上。由于远程文件系统通常比本地文件系统慢，因此使用 远程文件系统进行存储可能会降低性能。

如果决定使用网络文件系统，请在 /etc/fstab 文件中添加以下NFS选项：bg、nolock 和 noatime。

#### 将组件分离到不同的存储设备上


为了提高性能，请考虑根据应用程序的访问和写入模式，将数据库的数据、logs 和 journal 分离到不同的存储设备上。将组件作为单独的文件系统挂载，并使用符号链接将每个组件的路径映射到存储它的设备。

对于WiredTiger存储引擎，还可以将索引存储在不同的存储设备上。见[storage.wiredTiger.engineConfig.directoryForIndexes](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.directoryForIndexes)。  

注意              

使用不同的存储设备将影响您创建数据快照式备份的能力，因为文件将位于不同的设备和卷上。


#### 调度

##### 虚拟或云主机设备的调度

对于通过虚拟机监视器连接到虚拟机实例或由云托管提供商托管的本地块设备，客户操作系统应使用 noop 调度器以获得最佳性能。noop 调度器允许操作系统将 I/O 调度延缓到底层管理程序。

##### 物理服务器的调度

对于物理服务器，操作系统应使用 *deadline*调度器。*deadline*调度器限制每个请求的最大延迟，并保持良好的磁盘吞吐量，这对于磁盘密集型数据库应用程序来说是最好的。

## 架构

### 副本集

有关副本集部署的体系结构注意事项的概述，请参阅 [副本集体系结构文档](https://docs.mongodb.com/manual/core/replica-set-architectures/)。

### 分片集群

有关建议的用于生产部署的分片集群体系结构的概述，请参阅[分片集群生产体系结构 ](https://docs.mongodb.com/manual/core/sharded-cluster-components/)。

也可以参阅                                                                 

[开发清单列表](https://docs.mongodb.com/manual/administration/production-checklist-development/)


## 压缩


WiredTiger可以使用以下压缩库之一压缩收集数据：

- [snappy](https://docs.mongodb.com/manual/reference/glossary/#term-snappy)

  提供比zlib或zstd更低的压缩率，但比任何一种的CPU成本都低。

- [zlib](https://docs.mongodb.com/manual/reference/glossary/#term-zlib)

  提供了比 snappy 更好的压缩率，但比 snappy 和 zstd 的CPU成本都要高。

- [zstd](https://docs.mongodb.com/manual/reference/glossary/#term-zstd) (从 MongoDB 4.2 开始可以使用)

  提供比 snappy 和 zlib 更好的压缩率，并且比 zlib 具有更低的CPU成本。
 

默认情况下，WiredTiger使用 [snappy](https://docs.mongodb.com/manual/reference/glossary/#term-snappy) 压缩库。要更改压缩设置，请参见[storage.wiredTiger.collectionConfig.blockCompressor](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.collectionConfig.blockCompressor)。


默认情况下，WiredTiger对所有索引使用 [前缀压缩](https://docs.mongodb.com/manual/reference/glossary/#term-prefix-compression) 。


## 时钟同步 
                                                                                                                               

MongoDB [组件](https://docs.mongodb.com/manual/reference/program/)保留逻辑时钟以支持与时间相关的操作。使用[网络时间协议](http://www.ntp.org/)同步主机时钟来降低组件之间时钟漂移的风险。组件之间的时钟漂移增加了时间相关操作不正确或异常行为的可能性，如下所示：

- 如果任何给定 MongoDB 组件的底层系统时钟偏离同一部署中的其他组件一年或更长时间，则这些成员之间的通信可能变得不可靠或完全停止。

  [maxAcceptableLogicalClockDriftSecs](https://docs.mongodb.com/manual/reference/parameters/#param.maxAcceptableLogicalClockDriftSecs) 参数控制组件之间可接受的时钟偏移量。MaxAcceptableLogicalClockDiftSecs值较低的集群对时钟漂移的容忍度相应较低。

- 对于返回当前集群或系统时间的操作，具有不同系统时钟的两个集群成员可能返回不同的值，例如 [Date()](https://docs.mongodb.com/manual/reference/method/Date/#Date), [NOW](https://docs.mongodb.com/manual/reference/aggregation-variables/#variable.NOW), 和 [CLUSTER_TIME](https://docs.mongodb.com/manual/reference/aggregation-variables/#variable.CLUSTER_TIME)。

- 在MongoDB组件之间存在时钟漂移的集群中，依赖于计时的特性可能会有不一致或不可预测的行为。


例如，[TTL索引](https://docs.mongodb.com/manual/core/index-ttl/#index-feature-ttl)依赖于系统时钟来计算何时删除给定文档。如果两个成员有不同的系统时钟时间，则每个成员可以在不同的时间删除TTL索引覆盖的给定文档。由于[客户端会话和因果一致性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)保证使用TTL索引来控制它们的寿命，时钟漂移可能导致不一致或不可预测的会话超时行为。
                                       

运行 MongoDB 低于 3.4.6 或 3.2.17 的部署需要 NTP 同步，使用 WiredTiger 存储引擎，时钟漂移可能导致[检查点挂起](https://jira.mongodb.org/browse/WT-3227)。该问题在 MongoDB [3.4.6+](https://docs.mongodb.com/manual/release-notes/3.4-changelog/#id148) 和 MongoDB [3.2.17+](https://docs.mongodb.com/manual/release-notes/3.2/#id5) 中得到了修复，并在 MongoDB 3.6、4.0 和 4.2 版本中所有点得到了解决。


## 平台特定注意事项


#### Kernel and File Systems 内核和文件系统
                                                                                                                                                 

在Linux上的生产环境中运行MongoDB时，应该使用 Linux 内核版本 2.6.36 或更高版本，并使用 XFS或 EXT4 文件系统。如果可能的话，使用 XFS，因为它通常在 MongoDB 中执行得更好。


对于 [WiredTiger 存储引擎](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger)，强烈建议使用 XFS，以避免将 EXT4 与 WiredTiger 一起使用时可能出现的性能问题。

- 一般来说，如果您使用的是 XFS 文件系统，那么至少要使用 2.6.25 版本的Linux内核。
- 如果使用 EXT4 文件系统，请至少使用 2.6.28 版本的 Linux 内核。
- 在Red Hat 企业版 Linux 和 CentOS 上，至少使用 2.6.18-194 版 Linux 内核。

#### 系统C库

MongoDB在Linux上使用 [GNU C 库](http://www.gnu.org/software/libc/) (glibc)。一般来说，每个Linux发行版都提供了自己经过审查的版本。为了获得最佳结果，请使用此系统提供版本的最新更新。您可以使用系统的包管理器检查是否安装了最新版本。例如：

- 在 RHEL/CentOS 上，以下命令更新系统提供的 GNU C 库：

  ```
  sudo yum update glibc
  ```

- 在Ubuntu/Debian上，以下命令更新系统提供的 GNU C 库：

  ```
  sudo apt-get install libc6
  ```

#### 目录中的 fsync()

 重要

MongoDB要求文件系统对目录支持 fsync()。例如 HGFS 和 Virtual Box 的共享目录不支持这个操作。


#### 将 vm.swappiness 设置为 1 或者 0


“Swappiness” 是一种影响虚拟内存管理器的 Linux 内核设置，vm.swappiness 设置的范围从0到100：该值越高，它越倾向于将内存页交换到磁盘，而不是从RAM中删除页。

- 设置为0将完全禁用交换 [[2\]](https://docs.mongodb.com/manual/administration/production-notes/#swappiness-kernel-version)。
- 设置为1只允许内核交换以避免内存不足问题。
- 设置60告诉内核经常交换到磁盘，这是许多Linux发行版的默认值。
- 设置为100将告诉内核尽可能交换到磁盘。

MongoDB 在可以避免或保持最小交换的地方表现最好。因此，您应该根据应用程序需要和集群配置将 vm.swappiness 设置为1或0。

[2] 对于3.5之前的Linux内核版本，或 2.6.32-303 之前的 RHEL/CentOS 内核版本，vm.swappiness 设置为0仍然允许内核在某些紧急情况下进行交换。


注意

如果 MongoDB 实例托管在同时运行其他软件（如Web服务器）的系统上，则应将 vm.swappiness 设置为1。如果可能，强烈建议您在MongoDB自己的专用系统上运行MongoDB。

- 要检查系统上的当前交换设置，请运行：

  ```
  cat /proc/sys/vm/swappiness
  ```

- 要更改系统上的交换设置：

  1. 编辑 /etc/sysctl.conf 文件并添加以下行：

     ```
     vm.swappiness = 1
     ```

  2. 运行以下命令以应用设置：

     ```
     sudo sysctl -p
     ```


 注意

如果您正在运行 RHEL/CentOS 并使用优化的性能配置文件，则还必须编辑所选配置文件以将vm.swappiness 设置为1或0。



#### 推荐配置


对于所有MongoDB部署：

- 在主机之间同步时间使用网络时间协议（NTP）。这在分片集群中尤为重要。


对于 WiredTiger 存储引擎，请考虑以下建议：

- 在包含[数据库文件](https://docs.mongodb.com/manual/reference/glossary/#term-dbpath)的存储卷关闭 atime 配置。 

- 按照 [ulimit](https://docs.mongodb.com/manual/reference/ulimit/) 设置的推荐，设置描述符限制，-n 和用户进程限制（ulimit），-u 设置为20000以上。当大量使用时，低 ulimit 将影响 MongoDB，并可能产生错误，导致与MongoDB进程的连接失败和服务丢失。

- 不要使用透明大页，因为MongoDB在标准页中表现更好。参见 [透明大页设置](https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/).     

- 在BIOS中禁用NUMA。如果做不到，请参考 [MongoDB 和 NUMA 硬件](https://docs.mongodb.com/manual/administration/production-notes/#production-numa)章节。

- 如果不使用默认的 MongoDB 目录路径或 [端口](https://docs.mongodb.com/manual/reference/default-mongodb-port/)，请为 MongoDB 配置 SELinux。

请参阅[为 MongoDB 配置 SELinux](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/#install-rhel-configure-selinux) 和 [为 MongoDB 企业版配置 SELinux](https://docs.mongodb.com/manual/tutorial/install-mongodb-enterprise-on-red-hat/#install-enterprise-rhel-configure-selinux) 以获得所需的配置。


  注意

如果您使用的是 SELinux，任何需要 [服务器端 javaScript](https://docs.mongodb.com/manual/core/server-side-javascript/) 的 MongoDB 操作都会导致段错误。 [禁用服务器端执行JavaScript](https://docs.mongodb.com/manual/core/server-side-javascript/#disable-server-side-js) 描述如何禁用服务器端 JavaScript 执行。


对于WiredTiger存储引擎：

- 无论存储介质类型（旋转磁盘、SSD等）如何，将 文件预读的值设置为8到32。

较高的预读通常有利于顺序 I/O 操作。由于MongoDB 磁盘访问模式通常是随机的，因此使用更高的文件预读设置提供的好处有限，或者可能会降低性能。因此，为了获得最佳的 MongoDB 性能，请将文件预读的值设置在8到32之间，除非测试在更高的文件预读值中显示出可测量、可重复和可靠的好处。 [MongoDB 商业支持](https://support.mongodb.com/welcome?jmp=docs)可以提供关于备用文件预读配置的建议和指导。

  

#### MongoDB 和 TLS/SSL 库
 

在 Linux 平台上，您可以在 MongoDB 日志中看到以下语句之一：

```
<path to TLS/SSL libs>/libssl.so.<version>: no version information available (required by /usr/bin/mongod)
<path to TLS/SSL libs>/libcrypto.so.<version>: no version information available (required by /usr/bin/mongod)
```                              

这些警告表示系统的 TLS/SSL 库与 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)  编译时所依据的 TLS/SSL 库不同。通常这些消息不需要干预；但是，您可以使用以下操作来确定 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)  期望的符号版本：

```
objdump -T <path to mongod>/mongod | grep " SSL_"
objdump -T <path to mongod>/mongod | grep " CRYPTO_"
```

这些操作将返回类似于以下行之一的输出：

```
0000000000000000      DF *UND*       0000000000000000  libssl.so.10 SSL_write
0000000000000000      DF *UND*       0000000000000000  OPENSSL_1.0.0 SSL_write
```

此输出中的最后两个字符串是符号版本和符号名。将这些值与以下操作返回的值进行比较，以检测符号版本是否匹配：


```
objdump -T <path to TLS/SSL libs>/libssl.so.1*
objdump -T <path to TLS/SSL libs>/libcrypto.so.1*
```


这个过程既不精确也不详尽： [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 从 libcrypto 库中使用的许多符号不是以 CRYPTO_ 开头的。

### Windows 上的 MongoDB
                                                    

对于使用 WiredTiger 存储引擎的 MongoDB 实例，Windows 上的性能与 Linux 上的性能相当。



### 虚拟环境中的MongoDB
                                                                  

本章节描述了在常用虚拟环境中运行MongoDB需要考虑的问题。

对于所有平台，请考虑 [调度](https://docs.mongodb.com/manual/administration/production-notes/#virtualized-disks-scheduling).


#### AWS EC2（亚马逊弹性计算云）



有两种性能配置需要考虑：

- 性能测试或基准测试的可复制性能，以及
- 原始最大性能
 

 要为任一配置优化弹性计算云上的性能，应：
- 为您的实例启用亚马逊[增强的网络](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html#enabling_enhanced_networking)。并非所有实例类型都支持增强的网络。


要了解有关增强联网的更多信息，请参阅[AWS 文档](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html#enabling_enhanced_networking)。


如果您更关心弹性计算云的可重复性能，您还应该：

- 为存储使用配置的 IOPS，日志和数据使用单独的设备。不要使用大多数实例类型上可用的临时（SSD）存储，因为它们的性能会随时发生变化。（i 系列是一个显著的例外，但非常昂贵。）

- 禁用 DVFS 和 CPU 节能模式。


  也可以看看

  [关于处理器状态控制的Amazon文档](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/processor_state_control.html)                                                                              

- 禁用超线程。


  也可以看看

   [亚马逊关于禁用超线程的博客文章](https://aws.amazon.com/blogs/compute/disabling-intel-hyper-threading-technology-on-amazon-linux/).

- 使用 numactl 将内存局部性绑定到单个套接字。


#### Azure

使用[高级存储](https://azure.microsoft.com/en-us/documentation/articles/storage-premium-storage/)。微软Azure提供了两种常见的存储类型：标准存储和高级存储。与标准存储相比，Azure上的MongoDB 在使用高级存储时具有更好的性能。

默认情况下，Azure 负载平衡器上的 TCP 空闲超时默认为240秒，如果 Azure 系统上的 TCP 长连接大于此值，则会导致它自动断开连接。您应该将 TCP 长连接时间设置为120以改善此问题。


注意

要使新的系统范围长连接设置生效，您需要重新启动 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 和[mongos ](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)进程。

- 要在 Linux 上查看长连接设置，请使用以下命令之一：

  ```
  sysctl net.ipv4.tcp_keepalive_time
  ``` 

  或者：

  ```
  cat /proc/sys/net/ipv4/tcp_keepalive_time
  ```

  该值以秒为单位。


  注意                                                                        
  
  尽管设置名称包括IPv4 ，但 TCP 长连接时间值同时适用于 IPv4 和 IPv6。

- 要更改 TCP 长连接时间值，可以使用以下命令之一，以秒为单位提供<value>：：

  ```
  sudo sysctl -w net.ipv4.tcp_keepalive_time=<value>
  ```

  或者：

  ```
  echo <value> | sudo tee /proc/sys/net/ipv4/tcp_keepalive_time
  ```

这些操作不会在系统重新启动时保持。要保持设置，请将以下行添加到 /etc/sysctl.conf，以秒为单位提供 <value>，然后重新启动计算机：

  ```
  net.ipv4.tcp_keepalive_time = <value>
  ```

长连接值大于300秒（5分钟）将在 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) and [mongos ](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)套接字上重写，并设置为300秒。

- 要在 Windows 上查看长连接设置，请发出以下命令：

  ```
  reg query HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters /v KeepAliveTime
  ```

  默认情况下不存在注册表值。如果该值不存在，则使用系统默认值7200000毫秒或0x6ddd00（十六进制）。

- 要更改长连接时间值，请在管理员命令提示符中使用以下命令，该命令以十六进制表示（例如120000是0x1d4c0）：

  ```
  reg add HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\ /t REG_DWORD /v KeepAliveTime /d <value>
  ```

Windows用户应考虑 [Windows服务器 Technet 关于长连接时间值的文章](https://technet.microsoft.com/en-us/library/cc957549.aspx) 以获取有关在 Windows系统上设置 MongoDB 部署长连接的详细信息。 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) and [mongos](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 将忽略大于或等于*600000*毫秒（10分钟）的长连接值。

  

#### VMware


MongoDB 与 VMware 兼容。                       

VMware支持内存过量使用，在这里，您可以为虚拟机分配比物理机可用更多的内存。当内存被过度使用时，管理程序会在虚拟机之间重新分配内存。VMware 的气球驱动（vmmemctl）回收那些被认为价值最低的页面。

气球驱动程序位于客户操作系统中。当气球驱动程序扩展时，可能导致客户操作系统从客户端应用程序中回收内存，从而干扰 MongoDB 的内存管理，影响 MongoDB 的性能。

不要禁用气球驱动程序和内存过载使用功能。这会导致虚拟机监控程序使用其交换，从而影响性能。相反，映射并保留运行 MongoDB 的虚拟机的全部内存。这可以确保，如果管理程序中存在由于过度提交配置而导致的内存压力，则气球不会在本地操作系统中膨胀。

通过设置VMware的[关联规则](https://kb.vmware.com/selfservice/microsites/search.do?cmd=displayKC&docType=kc&externalId=1005508&sliceId=1&docTypeID=DT_KB_1_1&dialogID=549881455&stateId=0 0 549889513)，确保虚拟机留在特定的 ESX/ESXi 主机上。如果必须手动将虚拟机迁移到另一个主机，并且虚拟机上的 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例是[最重要的](https://docs.mongodb.com/manual/reference/glossary/#term-primary)，则必须先[逐步关闭](https://docs.mongodb.com/manual/reference/method/rs.stepDown/#rs.stepDown)最重要的实例，然后[关闭实例](https://docs.mongodb.com/manual/reference/method/db.shutdownServer/#db.shutdownServer)。

遵循 [vMotion的网络最佳实践](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.vcenterhost.doc/GUID-7DAD15D4-7F41-4913-9F16-567289E22977.html)和 [VMKernel](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2054994)。未能遵循最佳实践可能会导致性能问题，并影响[副本集](https://docs.mongodb.com/manual/core/replica-set-high-availability/)和[分片集群](https://docs.mongodb.com/manual/tutorial/troubleshoot-sharded-clusters/)的高可用性机制。

可以克隆运行 MongoDB 的虚拟机。您可以使用此函数启动新的虚拟主机，将其添加为副本集的成员。如果克隆启用了日志记录的虚拟机，则克隆快照将有效。如果不使用日志记录，首先停止[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)，然后克隆虚拟机，最后重新启动 [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)。

#### KVM


MongoDB 与 KVM 兼容。
                                      

KVM支持内存超载使用，在这里，您可以为虚拟机分配比物理机可用更多的内存。当内存被过度提交时，管理程序会在虚拟机之间重新分配内存。KVM的气球驱动程序回收被认为价值最低的页面。

气球驱动程序位于客户操作系统中。当气球驱动程序扩展时，可能导致客户操作系统从客户端应用程序中回收内存，从而干扰 MongoDB 的内存管理，影响 MongoDB 的性能。     

不要禁用气球驱动程序和内存过载使用功能。这会导致虚拟机监控程序使用其交换，从而影响性能。相反，映射并保留运行 MongoDB 的虚拟机的全部内存。这可以确保，如果管理程序中存在由于过度提交配置而导致的内存压力，则气球不会在本地操作系统中膨胀。


## 性能监控

注意

从4.0版开始，MongoDB为标准和副本集提供[免费云监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。有关更多信息，请参阅[免费监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。



### iostat

在 Linux 上，使用 iostat 命令检查磁盘 I/O 是否是数据库的瓶颈。指定运行 iostat 时的秒数，以避免显示信息为自服务器启动以来的统计信息。


例如，以下命令将每隔一秒显示扩展统计信息和每个显示报告的时间，流量单位为MB/s：

```
iostat -xmt 1
```

iostat中的关键字段：

- %util: 这是快速检查最有用的字段，它表示设备/驱动器使用时间的百分比。
- avgrq-sz:平均请求大小。此值的较小数字反映了更多的随机IO操作。

### bwm-ng

[bwm-ng](http://www.gropp.org/?id=projects&sub=bwm-ng) 是用于监视网络使用的命令行工具。如果怀疑是基于网络的瓶颈，可以使用 bwm-ng 开始诊断进程。


## 备份

                                                                                                                                                                        
要备份 MongoDB 数据库，请参阅 [MongoDB 备份方法概述](http://docs.mongodb.com/manual/core/backups/)。



## 附录


原文链接：https://docs.mongodb.com/manual/administration/production-notes/

译者：孔令升
