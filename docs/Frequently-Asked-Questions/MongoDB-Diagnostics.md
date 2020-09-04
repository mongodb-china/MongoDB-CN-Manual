# 常见问题解答：MongoDB诊断


在本页面

- [在哪里可以找到有关`mongod`进程意外停止运行的信息？](https://docs.mongodb.com/manual/faq/diagnostics/#where-can-i-find-information-about-a-mongod-process-that-stopped-running-unexpectedly)
- [TCP `keepalive`时间会影响MongoDB部署吗？](https://docs.mongodb.com/manual/faq/diagnostics/#does-tcp-keepalive-time-affect-mongodb-deployments)
- [为什么MongoDB会记录这么多“Connection Accepted”事件？](https://docs.mongodb.com/manual/faq/diagnostics/#why-does-mongodb-log-so-many-connection-accepted-events)
- [哪些工具可用于监视MongoDB？](https://docs.mongodb.com/manual/faq/diagnostics/#what-tools-are-available-for-monitoring-mongodb)
- [WiredTiger存储引擎的内存诊断](https://docs.mongodb.com/manual/faq/diagnostics/#memory-diagnostics-for-the-wiredtiger-storage-engine)
- [分片集群诊断](https://docs.mongodb.com/manual/faq/diagnostics/#sharded-cluster-diagnostics)


本文档提供了常见诊断问题的答案。

如果找不到所需的答案，请查看[常见问题解答](https://docs.mongodb.com/manual/faq/)的[完整列表](https://docs.mongodb.com/manual/faq/)或将问题发布到 [MongoDB社区](https://community.mongodb.com/?tck=docs_server)。



## 在哪里可以找到有关`mongod`进程意外停止运行的信息？


如果[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)在UNIX或基于UNIX的平台上意外关闭，并且[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)无法记录关闭或错误消息，请检查系统日志中与MongoDB有关的消息。例如，使用以下命令查看位于`/var/log/messages`中的日志：

复制

```
sudo grep mongod /var/log/messages
sudo grep score /var/log/messages
```



## TCP `keepalive`时间会影响MongoDB部署吗？[¶](https://docs.mongodb.com/manual/faq/diagnostics/#does-tcp-keepalive-time-affect-mongodb-deployments)

（译者注：tcp keepalive时间设置，主要用来探测连接对端是否还存活。当你建立一个TCP连接的时候，便有一组定时器与之绑定在一起。其中的一些定时器就用于处理keepalive过程。当keepalive定时器到0的时候，我们便会给对端发送一个不包含数据部分的keepalive探测包。如果我们收到了keepalive探测包的回复消息，那么我们就可以断定连接依然是OK的。如果我们没有收到对端keepalive探测包的回复消息，我们便可以断定连接已经不可用，进而可以采取一些措施。）

在客户端和服务器之间或者分片集群或副本集的成员之间，如果通信中遇到网络超时或套接字错误，请检查受影响系统的*TCP keepalive值*。

许多操作系统默认将*TCP keepalive值*设置为`7200`秒（两个小时）。对于MongoDB，通常情况下，保持较短时间值（`120`几秒钟（两分钟）），您将获得更好的结果。

如果您的MongoDB部署遇到与keepalive相关的问题，则必须更改*所有*受影响系统上的keepalive价值。这些系统包括所有正在运行[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)或[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 的计算机以及连接到MongoDB的客户端进程的所有计算机。



### 调整TCP keepalive值：[¶](https://docs.mongodb.com/manual/faq/diagnostics/#adjusting-the-tcp-keepalive-value)

- Linux

- Windows

- macOS

- 要在Linux上查看keepalive设置，请使用以下命令之一：

  复制

  ```
  sysctl net.ipv4.tcp_keepalive_time
  ```

  Or:

复制

  ```
  cat /proc/sys/net/ipv4/tcp_keepalive_time
  ```


  该值以秒为单位测量。


  注意

  尽管设置名称包括`ipv4`，但该 `tcp_keepalive_time`值同时适用于IPv4和IPv6。

  

- 要更改该`tcp_keepalive_time`值，可以使用以下命令之一，以秒为单位提供一个<value>：

复制
  
  ```
sudo sysctl -w net.ipv4.tcp_keepalive_time=<value>
  ```

  或者:

复制
  
  ```
echo <value> | sudo tee /proc/sys/net/ipv4/tcp_keepalive_time
  ```
  

  这些操作不会在系统重新启动后持续存在。要保留设置，请将以下行添加到中`/etc/sysctl.conf`，在几秒钟内提供一个<value>，然后重新启动计算机：
  
  
  复制
  
  ```
  net.ipv4.tcp_keepalive_time = <value>
  ```
  
  
  [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)进程将设置Keepalive值为`300`秒（5分钟），来重写覆盖大于5分钟的设置。



- 要在Windows上查看keepalive设置，请使用以下命令之一：

复制

  ```
  reg query HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters /v KeepAliveTime
  ```

  注册表值默认情况下不存在。如果没有该值，则使用系统默认值，以`7200000`毫秒为单位或 `0x6ddd00`以十六进制表示。

- 要改变`KeepAliveTime`值，在管理员使用以下**命令提示符**，其中以十六进制（例如，表示`120000`为`0x1d4c0`）：

复制

  ```
  reg add HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\ /t REG_DWORD /v KeepAliveTime /d <value>
  ```


  Windows用户应该[查看KeepAliveTime上](https://technet.microsoft.com/en-us/library/cc957549.aspx)的[Windows Server Technet章节，](https://technet.microsoft.com/en-us/library/cc957549.aspx)以获取有关在Windows系统上为MongoDB部署设置keepalive的更多信息。[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)会忽略大于或等于*600000*毫秒（10分钟）的 keepalive值 。



- 要在macOS上查看keepalive设置，请使用以下命令之一：

复制

  ```
  sysctl net.inet.tcp.keepidle
  ```

  该值以秒为单位测量。

- 要更改该`net.inet.tcp.keepidle`值，可以使用以下命令，以毫秒为单位提供一个<value> ：

复制

  ```
  sudo sysctl net.inet.tcp.keepidle=<value>
  ```

  此操作不会在系统重新启动后持续存在，必须在每次系统重新启动时进行设置。请参阅操作系统的文档，以获取有关持久设置此值的说明。[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 会忽略大于或等于`600000`毫秒（10分钟）的keepalive值。


  注意

  在macOS 10.15 Catalina中，Apple不再允许配置该`net.inet.tcp.keepidle`选项。


您将需要重新启动[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)并[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 进程，新的Keepalive系统设置才能生效。



## 为什么MongoDB记录这么多“Connection Accepted” 事件？[¶](https://docs.mongodb.com/manual/faq/diagnostics/#why-does-mongodb-log-so-many-connection-accepted-events)


如果您在MongoDB日志中看到大量的连接和重新连接消息，则说明客户端与MongoDB服务器经常连接和断开。对于不使用请求池的应用程序（例如CGI），这是正常的行为。考虑使用FastCGI，Apache模块或其他某种持久性应用程序服务器来减少连接开销。


如果这些连接不影响性能，则可以使用运行时[`quiet`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.quiet)选项或命令行选项 [`--quiet`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-quiet)从日志中禁止显示这些消息。



## 哪些工具可用于监视MongoDB？[¶](https://docs.mongodb.com/manual/faq/diagnostics/#what-tools-are-available-for-monitoring-mongodb)


从版本4.0开始，MongoDB 为单机部署和副本集提供[免费的云监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。免费监控可提供如下部署的信息：

- 操作执行时间
- 内存使用情况
- CPU使用率
- 操作计数

有关更多信息，请参见[免费监控](https://docs.mongodb.com/manual/administration/free-monitoring/)。

在[MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?jmp=docs)和 [在MongoDB的企业提供先进的内部部署解决方案的Ops Manager](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)包括监控功能，其收集运行的MongoDB部署数据，并提供基于数据的可视化和警报。

有关更多信息，另请参阅[MongoDB Cloud Manager文档](https://docs.cloudmanager.mongodb.com/)和 [Ops Manager文档](https://docs.opsmanager.mongodb.com/current/application)。

[Monitoring for MongoDB](https://docs.mongodb.com/manual/administration/monitoring/)文档中提供了完整的第三方工具列表。



## WiredTiger存储引擎的内存诊断

### 我的working set必须适合内存吗？

不需要。

如果缓存没有足够的空间来加载其他数据，则WiredTiger会将页面从缓存中逐出以释放空间。

注意

该[`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)限制WiredTiger内部缓存的大小。操作系统将使用可用的空闲内存进行文件系统缓存，从而允许压缩的MongoDB数据文件保留在内存中。此外，操作系统将使用任何可用的RAM来缓冲文件系统块和文件系统缓存。

为了容纳更多的内存使用者，您可能必须减小WiredTiger内部缓存的大小。


默认的WiredTiger内部缓存大小值假定每台计算机有一个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。如果一台机器包含多个MongoDB实例，则您应减小设置以容纳其他[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例。


如果你在一个容器（例如`lxc`， `cgroups`，Docker，等等）运行[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)，它*没有*访问所有系统中可用的RAM，您必须设置[`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)的值小于容器使用的内存量。确切的数量取决于容器中运行的其他进程。请参阅 [`memLimitMB`](https://docs.mongodb.com/manual/reference/command/hostInfo/#hostInfo.system.memLimitMB)。


要查看有关缓存和逐出的统计信息，请使用 [`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus)命令。该 [`wiredTiger.cache`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.wiredTiger.cache)字段保存有关缓存和逐出的信息。

复制

```
...
"wiredTiger" : {
   ...
   "cache" : {
      "tracked dirty bytes in the cache" : <num>,
      "bytes currently in the cache" : <num>,
      "maximum bytes configured" : <num>,
      "bytes read into cache" :<num>,
      "bytes written from cache" : <num>,
      "pages evicted by application threads" : <num>,
      "checkpoint blocked page eviction" : <num>,
      "unmodified pages evicted" : <num>,
      "page split during eviction deepened the tree" : <num>,
      "modified pages evicted" : <num>,
      "pages selected for eviction unable to be evicted" : <num>,
      "pages evicted because they exceeded the in-memory maximum" : <num>,,
      "pages evicted because they had chains of deleted items" : <num>,
      "failed eviction of pages that exceeded the in-memory maximum" : <num>,
      "hazard pointer blocked page eviction" : <num>,
      "internal pages evicted" : <num>,
      "maximum page size at eviction" : <num>,
      "eviction server candidate queue empty when topping up" : <num>,
      "eviction server candidate queue not empty when topping up" : <num>,
      "eviction server evicting pages" : <num>,
      "eviction server populating queue, but not evicting pages" : <num>,
      "eviction server unable to reach eviction goal" : <num>,
      "pages split during eviction" : <num>,
      "pages walked for eviction" : <num>,
      "eviction worker thread evicting pages" : <num>,
      "in-memory page splits" : <num>,
      "percentage overhead" : <num>,
      "tracked dirty pages in the cache" : <num>,
      "pages currently held in the cache" : <num>,
      "pages read into cache" : <num>,
      "pages written from cache" : <num>,
   },
   ...
```


有关某些关键高速缓存和逐出统计信息（例如`wiredTiger.cache.bytes currently in the cache`和`wiredTiger.cache.tracked dirty bytes in the cache`）的说明，请参见[`wiredTiger.cache`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.wiredTiger.cache)。


要调整WiredTiger内部缓存的大小，请参阅 [`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)和 [`--wiredTigerCacheSizeGB`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-wiredtigercachesizegb)。避免将WiredTiger内部缓存的大小增加到其默认值以上。



### 如何计算应用程序需要多少内存？[¶](https://docs.mongodb.com/manual/faq/diagnostics/#how-do-i-calculate-how-much-ram-i-need-for-my-application)


通过WiredTiger，MongoDB可以利用WiredTiger内部缓存和文件系统缓存。


从MongoDB 3.4开始，默认的WiredTiger内部缓存大小是以下两者中的较大者：

- 50% of (RAM - 1 GB), or
- 256 MB.


例如，在总共有4GB内存的系统上，WiredTiger缓存将使用1.5GB 内存（`0.5 * (4 GB - 1 GB) = 1.5 `）。相反，总内存为1.25 GB的系统将为WiredTiger高速缓存分配256 MB，因为这是总内存的一半以上减去1 GB 。(`0.5 * (1.25 GB - 1 GB) = 128 MB < 256 MB`).


注意

在某些情况下，例如在容器中运行时，数据库的内存限制可能低于系统总内存。在这种情况下，此内存限制而不是系统总内存将用作最大可用内存。


要查看内存限制，请参阅[`hostInfo.system.memLimitMB`](https://docs.mongodb.com/manual/reference/command/hostInfo/#hostInfo.system.memLimitMB)。


默认情况下，WiredTiger对所有集合使用Snappy块压缩，对所有索引使用前缀压缩。压缩默认设置可在全局级别配置，也可以在每个集合和每个索引创建期间单独进行设置。


WiredTiger内部缓存中的数据与磁盘上的格式使用不同的形式的数据格式：


- 文件系统缓存中的数据与磁盘格式相同，包括对数据文件进行任何压缩的好处。操作系统使用文件系统缓存来减少磁盘I / O。
- 加载到WiredTiger内部缓存中的索引的数据表示形式与磁盘格式不同，但是仍可以利用索引前缀压缩来减少n内存使用量。索引前缀压缩可从索引字段中删除通用前缀。
- WiredTiger内部缓存中的集合数据是未压缩的，并使用与磁盘格式不同的表示形式。块压缩可以节省大量的磁盘存储空间，但数据解压缩才能由服务器操作。


通过文件系统缓存，MongoDB自动使用WiredTiger缓存或其他进程未使用的所有可用内存。


要调整WiredTiger内部缓存的大小，请参阅 [`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)和 [`--wiredTigerCacheSizeGB`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-wiredtigercachesizegb)。避免将WiredTiger内部缓存的大小增加到其默认值以上。


注意

该[`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)限制WiredTiger内部缓存的大小。操作系统将使用可用的空闲内存进行文件系统缓存，从而允许压缩的MongoDB数据文件保留在内存中。此外，操作系统将使用任何可用的内存来缓冲文件系统块和文件系统缓存。


为了容纳更多的内存使用者，您可能必须减小WiredTiger内部缓存的大小。


默认的WiredTiger内部缓存大小值假定每台计算机有一个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。如果单个机器包含多个MongoDB实例，则应减小设置以容纳其他[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例。


如果你在一个容器（例如`lxc`， `cgroups`，Docker，等等）运行[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)，它*没有*访问所有系统中可用的内存，您必须将[storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)的值设置为小于容器可用内存大小的值。确切的大小取决于容器中运行的其他进程。参见 [`memLimitMB`](https://docs.mongodb.com/manual/reference/command/hostInfo/#hostInfo.system.memLimitMB)。


要查看有关缓存和缓存淘汰率的统计信息，请参阅[`wiredTiger.cache`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.wiredTiger.cache)命令返回的[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) 字段。


## 分片群集诊断


成功维护分片集群的两个最重要的因素是：

- [选择适当的片键，](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-shard-keys)
- [足以支持当前和将来的运营的容量](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/#sharding-capacity-planning)。


通过确保为部署选择最佳的[片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)，并确保始终在当前资源饱和之前为集群添加额外的容量，可以避免分片遇到的大多数问题。继续阅读，查看您在生产环境中可能遇到的特定问题。



### 在新的分片集群中，为什么所有数据都保留在一个分片上？


您的集群必须具有足够的数据才能进行分片。分片的工作原理是在分片之间迁移数据块，直到每个分片具有大致相同数量的分块。

默认的块大小为64 MB。在集群中的大块不平衡量超过[迁移阈值](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#sharding-migration-thresholds)之前，MongoDB不会开始迁移 。此行为有助于防止不必要的块迁移降低整个集群的性能。

如果您刚刚部署了分片集群，请确保您有足够的数据使分片有效。如果没有足够的数据来创建八个以上的64 MB数据块，则所有数据将保留在一个分片上。降低[块大小](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#sharding-chunk-size)设置，或向集群添加更多数据。

作为一个相关问题，系统将仅在插入或更新时拆分块，这意味着，如果您配置了分片并且不继续执行插入和更新操作，则数据库将不会创建任何块。您可以等待，直到应用程序插入数据*或* [手动拆分块](https://docs.mongodb.com/manual/tutorial/split-chunks-in-sharded-cluster/)。

最后，如果片键的[基数](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-cardinality)较低，则MongoDB可能无法在数据之间创建足够的拆分（chunk将无法继续分裂）。



### 为什么一个分片会在分片集群中会收到不均衡的流量？


在某些情况下，单个分片或集群的子集将接收不均衡的流量和工作负载。在几乎所有情况下，这都是因为片键无法高效地允许[写缩放](https://docs.mongodb.com/manual/sharding/#sharding-shard-key-write-scaling)。

您也可能有“hot chunks”。在这种情况下，您可以通过拆分然后迁移部分块来解决问题。

在最坏的情况下，您可能必须考虑重新分拆数据并[选择其他](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-choose-shard-key)[片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-choose-shard-key) 来更正此模式。


### 什么可以阻止分片集群均衡？[¶](https://docs.mongodb.com/manual/faq/diagnostics/#what-can-prevent-a-sharded-cluster-from-balancing)


如果您刚刚部署了分片群集，则可能需要考虑[针对新集群数据仍保留在单个分片上](https://docs.mongodb.com/manual/faq/diagnostics/#sharding-troubleshooting-not-splitting)的[故障排除建议](https://docs.mongodb.com/manual/faq/diagnostics/#sharding-troubleshooting-not-splitting)。

如果集群最初是均衡的，但后来却出现了数据分布不均的情况，请考虑以下可能的原因：


- 您已从集群中删除或移除了大量数据。如果添加了其他数据，则其片键的分配可能会有所不同。
- 您的[片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)具有较低的[基数，](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-cardinality) 并且MongoDB无法进一步拆分块。
- 您的数据集增长速度超过了均衡器可以在集群中分发数据的速度。这种情况并不常见，通常是由于以下原因造成的：
  - 相对于数据增长的速度，这个[均衡窗口](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#sharding-schedule-balancing-window)太短。
  
  - [写操作](https://docs.mongodb.com/manual/sharding/#sharding-shard-key-write-scaling)分布不均，需要更多的数据迁移。您可能必须选择其他分片键才能解决此问题。
  
  - 分片之间的网络连接不良，这可能会导致需要很长时间才能完成数据块迁移。检查您的网络配置和分片之间的互连。
  
    

### 为什么块迁移会影响分片集群的性能？


如果迁移会影响您的集群或应用程序的性能，请根据影响的性质考虑以下选项：
   
   1.如果迁移仅偶尔中断集群，则可以限制均衡窗口以防止高峰时段进行均衡活动。确保有足够的时间来防止数据再次失去均衡。
   
   2.如果均衡器始终在迁移块而不利于整体集群性能：
   
   - 您可能想要尝试[减小块大小](https://docs.mongodb.com/manual/tutorial/modify-chunk-size-in-sharded-cluster/) 以限制迁移的大小。
   - 您的集群可能超载，您可能想尝试向该集群[添加一个或两个分片](https://docs.mongodb.com/manual/tutorial/add-shards-to-shard-cluster/#sharding-procedure-add-shard)以分摊负载。

您的片键还可能导致应用程序将所有写入指向单个分片。这种活动模式可能要求均衡器在写入数据后立即迁移大部分数据。请考虑使用提供更好[写入扩展的片键](https://docs.mongodb.com/manual/sharding/#sharding-shard-key-write-scaling)重新部署集群。



原文链接：https://docs.mongodb.com/manual/faq/diagnostics/

译者：钟秋

update:小芒果
