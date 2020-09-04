## 词汇表

- **$cmd**

  一个特殊的虚拟[集合](https://docs.mongodb.com/master/reference/glossary/#term-collection)，它公开MongoDB的[数据库命令](https://docs.mongodb.com/master/reference/glossary/#term- databs-command)。要使用数据库命令，请参见[Issue commands](https://docs.mongodb.com/master/tutorial/use-database-commands/#issue-commands)。

- **_id**

  每个MongoDB[文档](https://docs.mongodb.com/master/reference/glossary/#term-document)中都需要的字段。[_id](https://docs.mongodb.com/master/core/document/#document-id-field)字段必须有一个唯一的值。您可以将 **_id** 字段看作文档的[主键](https://docs.mongodb.com/master/reference/glossary/#term-primary-key)。如果您创建一个没有`_id`字段的新文档，MongoDB将自动创建该字段并分配一个唯一的BSON [ObjectId](https://docs.mongodb.com/master/reference/glossary/#term-objectid)。

- **accumulator**

  [聚合框架](https://docs.mongodb.com/master/reference/glossary/#term-aggregation-framework)中的一种[表达式](https://docs.mongodb.com/master/reference/glossary/#term-expression)，用于维护聚合[管道](https://docs.mongodb.com/master/reference/glossary/#term-pipeline)中文档之间的状态 。有关accumulator操作的列表，请参见 。[`$group`](https://docs.mongodb.com/master/reference/operator/aggregation/group/#pipe._S_group)

- **action**

  用户可以对资源执行的操作。Actions和[资源](https://docs.mongodb.com/master/reference/glossary/#term-resource)组合创建[特权](https://docs.mongodb.com/master/reference/glossary/#term-privilege)。看[行动](https://docs.mongodb.com/master/reference/privilege-actions/)。

- **admin database**

  一个数据库特权。用户必须能够访问 **admin** 数据库才能运行某些管理命令。有关管理命令的列表，请参见[管理命令](https://docs.mongodb.com/master/reference/command/#admin-commands)。

- **aggregation**

  减少和汇总大量数据的各种操作中的任何一种。MongoDB [`aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate)和 [`mapReduce()`](https://docs.mongodb.com/master/reference/method/db.collection.mapReduce/#db.collection.mapReduce)方法是聚合操作的两个示例。有关更多信息，请参见 [聚合](https://docs.mongodb.com/master/aggregation/)。

- **aggregation framework**

  一组MongoDB操作符，让您不必使用[map-reduce](https://docs.mongodb.com/master/reference/glossary/#term-map-reduce)就可以计算聚合值。有关操作符的列表，请参见[Aggregation Reference](https://docs.mongodb.com/master/reference/aggregation/)。

- **arbiter**

  一个[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的成员，该成员仅存在于[elections](https://docs.mongodb.com/master/reference/glossary/#term-election)中投票。仲裁器不复制数据。查看[Replica Set仲裁者](https://docs.mongodb.com/master/core/replica-set仲裁者/# replica-set仲裁者-配置)。

- **Atlas**

  [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)是云托管的数据库即服务。

- **authentication**

  验证用户身份。请看[authentication](https://docs.mongodb.com/master/core/authentication/)。

- **authorization**

  提供对数据库和操作的访问。参见[基于角色的访问控制](https://docs.mongodb.com/master/core/authorization/)。

- **B-tree**

  数据库管理系统通常用于存储索引的数据结构。MongoDB使用B-trees为其索引。

- **balancer**

  一个内部的MongoDB进程，运行在一个[分片集群](https://docs.mongodb.com/master/reference/glossary/#term-shard-cluster)的上下文中，并管理[chunk](https://docs.mongodb.com/master/reference/glossary/#term-chunk)的迁移。管理员必须为分片集群上的所有维护操作禁用平衡器。参见[Sharded Cluster Balancer](https://docs.mongodb.com/master/core/sharding- balancer-admination/# sharding-balancing)。

- **BSON**

  一种用于在MongoDB中存储[文档](https://docs.mongodb.com/master/reference/glossary/#term-document)和进行远程过程调用的序列化格式。“BSON”是“二进制”和“JSON”的合成词。可以将BSON视为JSON（JavaScript对象表示法）文档的二进制表示形式。请参阅 [BSON类型](https://docs.mongodb.com/master/reference/bson-types/)和 [MongoDB扩展JSON(v2)](https://docs.mongodb.com/master/reference/mongodb-extended-json/)。

- **BSON types**

  [BSON](https://docs.mongodb.com/master/reference/glossary/#term-bson)序列化格式支持的类型集。有关BSON类型的列表，请参见[BSON types](https://docs.mongodb.com/master/reference/bson-types/)。

- **CAP Theorem**

  给定计算系统的三个属性，一致性，可用性和分区容限，分布式计算系统可以提供这些功能中的任何两个，但不能提供全部三个。

- **capped collection**

  一个固定大小的[集合](https://docs.mongodb.com/master/reference/glossary/#term-collection)，当其达到最大大小时会自动覆盖其最早的条目。在[复制](https://docs.mongodb.com/master/reference/glossary/#term-replication)中使用的MongoDB  [oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)是一个有上限的集合。。请参阅[限制集合](https://docs.mongodb.com/master/core/capped-collections/)。

- **cardinality**

  对一组值中元素数量的度量。例如，集合 **A ={2,4,6}** 包含3个元素，基数为3。参见[分片键基数](https://docs.mongodb.com/master/core/sharing-shard-key/#shard-keycardinality)。

- **checksum**

  用于确保数据完整性的计算值。有时使用[md5](https://docs.mongodb.com/master/reference/glossary/#term-md5)算法作为checksum。

- **chunk**

  一个连续范围的[分片键](https://docs.mongodb.com/master/reference/glossary/#term-shard-key)的特定内的值[分片](https://docs.mongodb.com/master/reference/glossary/#term-shard)。块范围包括下边界，不包括上边界。当MongoDB超出配置的块大小（默认为64兆字节）时，MongoDB将对其进行拆分。当一个分片相对于其他分片包含一个集合的太多分块时，MongoDB会迁移这些分块。请参见 [使用块](https://docs.mongodb.com/master/core/sharding-data-partitioning/#sharding-data-partitioning)和分[片群集平衡器进行](https://docs.mongodb.com/master/core/sharding-balancer-administration/#sharding-balancing)[数据分区](https://docs.mongodb.com/master/core/sharding-data-partitioning/#sharding-data-partitioning)。

- **client**

  使用数据库进行数据持久性和存储的应用层。[Drivers](https://docs.mongodb.com/master/reference/glossary/#term-driver)提供了应用程序层和数据库服务器之间的接口级别。客户端也可以引用单个线程或进程。

- **cluster**

  请看 [sharded cluster](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster).

- **collection**

  MongoDB [文档](https://docs.mongodb.com/master/reference/glossary/#term-document)的分组。集合等效于[RDBMS](https://docs.mongodb.com/master/reference/glossary/#term-rdbms)表。集合存在于单个[数据库](https://docs.mongodb.com/master/reference/glossary/#term-database)中。集合不强制执行架构。集合中的文档可以具有不同的字段。通常，集合中的所有文档都具有相似或相关的目的。请参阅[命名空间](https://docs.mongodb.com/master/reference/limits/#faq-dev-namespace)。

- **collection scan**

  集合扫描是一种查询执行策略，MongoDB必须检查集合中的每个文档，以确定它是否符合查询条件。这些查询效率非常低，并且不使用索引。有关查询执行策略的详细信息，请参阅[查询优化](https://docs.mongodb.com/master/core/query-optimization/)。

- **compound index**

  由两个或多个键组成的[索引](https://docs.mongodb.com/master/reference/glossary/#term-index)。请看[复合索引](https://docs.mongodb.com/master/core/index-compound/ # index-type-compound)。

- **concurrency control**

  并发控制可确保数据库操作可以并发执行而不会影响正确性。悲观并发控制，例如在带[锁](https://docs.mongodb.com/master/reference/glossary/#term-lock)的系统中使用的，将阻止任何可能发生冲突的操作，即使它们可能最终并未真正冲突。乐观并发控制，即WiredTiger使用的方法将延迟检查，直到可能发生冲突之后，终止并重试任何出现 [写冲突](https://docs.mongodb.com/master/reference/glossary/#term-write-conflict)的操作。

- **config database**

  一个内部数据库，保存与[分片集群](https://docs.mongodb.com/master/reference/glossary/#term-shard-cluster)相关联的元数据。应用程序和管理员不应该在正常操作过程中修改config数据库。请看[配置数据库](https://docs.mongodb.com/master/reference/config-database/)。

- **config server**

  一个[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例，存储与[分片集群](https://docs.mongodb.com/master/reference/glossary/#term-shard-cluster)相关联的所有元数据。看到[配置服务器](https://docs.mongodb.com/master/core/sharded-cluster-config-servers/ # sharding-config-server)。

- **container**

  打包在一起的一组软件及其从属库可以简化在计算环境之间的传输。容器在您的操作系统上作为分隔的进程运行，并且可以赋予它们自己的资源限制。常见的容器技术是Docker和Kubernetes。

- **CRUD**

  数据库基本操作的缩写:创建、读取、更新和删除。查看[MongoDB CRUD操作](https://docs.mongodb.com/master/crud/)。

- **CSV**

  一种基于文本的数据格式，由逗号分隔的值组成。由于该格式非常适合表格数据，因此通常用于在关系数据库之间交换数据。您可以使用导入CSV文件[`mongoimport`](https://docs.mongodb.com/database-tools/mongoimport/#bin.mongoimport)。

- **cursor**

  一个指向[查询](https://docs.mongodb.com/master/reference/glossary/#term-query)结果集的指针。客户端可以遍历游标来检索结果。默认情况下，游标在不活动10分钟后超时。参见[在mongo Shell中迭代游标](https://docs.mongodb.com/master/tutorial/iterate-a-cursor/#read-operations-cursors)。

- **daemon**

  后台、非交互进程的传统名称。

- **data directory**

  [` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)存储数据文件的文件系统位置。[`dbPath`](https://docs.mongodb.com/master/reference/configuring-options/#storage.dbpath)选项指定数据目录。

- **data partition**

  将数据划分为范围的分布式系统体系结构。 [分片](https://docs.mongodb.com/master/reference/glossary/#term-sharding)使用分区。请参见 [使用块进行数据分区](https://docs.mongodb.com/master/core/sharding-data-partitioning/#sharding-data-partitioning)。

- **data-center awareness**

  一种属性，允许客户端根据其位置来寻址系统中的成员。[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set) 使用[标签](https://docs.mongodb.com/master/reference/glossary/#term-tag)实现数据中心感知。请参阅 [数据中心意识](https://docs.mongodb.com/master/data-center-awareness/)。

- **database**

  [集合](https://docs.mongodb.com/master/reference/glossary/#term-collection)的物理容器。每个数据库在文件系统上有自己的一组文件。一个MongoDB服务器通常有多个数据库。

- **database command**

  MongoDB操作，而不是插入、更新、删除或查询。有关数据库命令的列表，请参见[数据库命令](https://docs.mongodb.com/master/reference/command/)。要使用数据库命令，请参见[Issue commands](https://docs.mongodb.com/master/tutorial/use-database-commands/#issue-commands)。

- **database profiler**

  一种工具，当它被启用时，它在数据库的“系统”中保存所有长时间运行的操作的记录。概要文件的集合。分析器最常用来诊断慢速查询。请看[数据库分析](https://docs.mongodb.com/master/administration/analyzing-mongodb-performance/ #数据库明细)。

- **dbpath**

  MongoDB的数据文件存储位置。请看[` dbPath `](https://docs.mongodb.com/master/reference/configuration-options/ # storage.dbPath)。

- **delayed member**

  一个[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)成员，该成员不能成为主成员并在指定的延迟下应用操作。延迟对于保护数据不受人为错误(即无意中删除的数据库)或对生产数据库有不可预见影响的更新的影响非常有用。参见[Delayed Replica Set Members](https://docs.mongodb.com/master/core/replica-setdelayed-member/ # replica-setdelayed-members)。

- **document**

  MongoDB[集合](https://docs.mongodb.com/master/reference/glossary/#term-collection)中的一条记录和MongoDB中的基本数据单元。文档类似于[JSON](https://docs.mongodb.com/master/reference/glossary/#term-json)对象，但是以一种更丰富类型的格式存在于数据库中，称为[BSON](https://docs.mongodb.com/master/reference/glossary/#term-bson)。请看[document](https://docs.mongodb.com/master/core/document/)。

- **dot notation**

  MongoDB使用点表示法来访问数组的元素和访问嵌入文档的字段。看到[Dot Notation](https://docs.mongodb.com/master/core/document/ # document-dot-notation)。

- **draining**

  从一个[分片](https://docs.mongodb.com/master/reference/glossary/#term-shard)到另一个分片的移除或“shedding”[chunks](https://docs.mongodb.com/master/reference/glossary/#term-chunk)的过程。管理员必须在将分片从集群中删除之前将其排干。参见[从现有分片集群中删除分片](https://docs.mongodb.com/master/tutorial/remove-shars-from-cluster/)。

- **driver**

  用特定语言与MongoDB交互的客户端库。见 [/drivers](https://docs.mongodb.com/ecosystem/drivers).

- **durable**

  当一个或多个服务器进程关闭(或崩溃)和重新启动时，写操作是持久的。对于单个[' mongod '](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)服务器，当写入服务器的[journal](https://docs.mongodb.com/master/reference/glossary/#term-journal)文件时，写操作被认为是持久的。对于[复制集](https://docs.mongodb.com/master/replication/)，一旦写入操作在大多数投票节点上是持久的，那么写入操作就被认为是持久的;即写给大多数投票节点的日志。

- **election**

  在启动和失败时，[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的成员选择一个[primary](https://docs.mongodb.com/master/reference/glossary/#term-primary)的进程。查看[Replica Set Elections](https://docs.mongodb.com/master/core/replica-setelections/ # replica-setelections)。

- **eventual consistency**

  分布式系统的一种属性，允许对系统的更改逐渐传播。在数据库系统中，这意味着可读成员不需要随时反映最新的写操作。

- **expression**

  在[聚合框架](https://docs.mongodb.com/master/reference/glossary/#term-aggreging-framework)的上下文中，表达式是对通过[管道](https://docs.mongodb.com/master/reference/glossary/#term-pipeline)的数据进行操作的无状态转换。请看[聚合管道](https://docs.mongodb.com/master/core/aggregation-pipeline/)。

- **failover**

  在发生故障时允许[副本集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的[辅助](https://docs.mongodb.com/master/reference/glossary/#term-secondary)成员成为[主要](https://docs.mongodb.com/master/reference/glossary/#term-primary)成员 的过程。请参阅[自动故障转移](https://docs.mongodb.com/master/replication/#replication-auto-failover)。

- **field**

  A name-value pair in a [document](https://docs.mongodb.com/master/reference/glossary/#term-document). A document has zero or more fields. Fields are analogous to columns in relational databases. See [Document Structure](https://docs.mongodb.com/master/core/document/#document-structure).

  [文档](https://docs.mongodb.com/master/reference/glossary/#term-document)中的名称-值对。一个文档有零个或多个字段。字段类似于关系数据库中的列。请看[文档结构](https://docs.mongodb.com/master/core/document/#document-structure)。

- **field path**

  文档中某个字段的路径。要指定字段路径，请使用一个字符串在字段名前加上美元符号(' **$** ')。

- **firewall**

  一种基于IP地址限制访问的系统级网络过滤器。防火墙是有效网络安全策略的一部分。请看[防火墙](https://docs.mongodb.com/master/core/security-hardening/#security-firewalls).

- **fsync**

  将内存中所有脏页面刷新到磁盘的系统调用。MongoDB至少每60秒对其数据库文件调用 **fsync()** 。请看[` fsync `](https://docs.mongodb.com/master/reference/command/fsync/ # dbcmd.fsync)。

- **geohash**

  geohash值是坐标网格中位置的二进制表示。参见[计算2d索引的Geohash值](https://docs.mongodb.com/master/core/geospatial-indexes/#geospatial-indexes-geohash)。

- **GeoJSON**

  基于JavaScript对象符号的数据交换格式([JSON](https://docs.mongodb.com/master/reference/glossary/#term-geospatial))。GeoJSON用于[地理空间查询](https://docs.mongodb.com/master/geospatial-queries/)。有关受支持的GeoJSON对象，请参见[地理空间数据](https://docs.mongodb.com/master/geospatial-queries/#geo-overview-location-data)。有关GeoJSON格式规范，请参见https://tools.ietf.org/html/rfc7946#section-3.1。

- **geospatial**

  与地理位置有关的。看到[地理空间查询](https://docs.mongodb.com/master/geospatial-queries/)。

- **GridFS**

  在MongoDB数据库中存储大文件的约定。所有官方的MongoDB驱动程序都支持这个约定，就像[` mongofiles `](https://docs.mongodb.com/databe-tools/mongofiles/ #bin.mongofiles)程序一样。参见[GridFS](https://docs.mongodb.com/master/core/gridfs/)。

- **hashed shard key**

  一种特殊类型的[分片键](https://docs.mongodb.com/master/reference/glossary/ # term-shard-key),使用一个hash值的分片键字段成员之间分发文件的[分片集群](https://docs.mongodb.com/master/reference/glossary/ # term-sharded-cluster)。请看[Hashed索引](https://docs.mongodb.com/master/core/index-hashed/ # index-type-hashed)。

- **haystack index**

  一个[geospatial](https://docs.mongodb.com/master/reference/glossary/#term-geospatial)索引，该索引通过创建根据第二个标准分组的对象的“**buckets**”来增强搜索。看到[geoHaystack索引](https://docs.mongodb.com/master/core/geohaystack/)。

- **hidden member**

  一个[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)成员，不能成为[primary](https://docs.mongodb.com/master/reference/glossary/#term-primary)并且对客户端应用程序不可见。参见[Hidden Replica Set Members](https://docs.mongodb.com/master/core/replica-sethidden -member/# replica-sethidden - Members)。

- **high availability**

  高可用性指的是为持久性、冗余和自动故障转移而设计的系统，这样系统所支持的应用程序就可以连续运行，而不会在很长一段时间内停机。MongoDB[复制集](https://docs.mongodb.com/master/replication/)复制支持高可用性部署时根据我们的记录[最佳实践](https://docs.mongodb.com/master/administration/production-checklist-operations/) 。有关复制集部署架构的指导，请参阅[副本集部署架构](https://docs.mongodb.com/master/core/replica-setarchitectures/# replica-setarchitecture)。

- **idempotent**

  在相同的输入下产生相同结果的操作的质量，无论运行一次还是多次。

- **index**

  优化查询的数据结构。请看[索引](https://docs.mongodb.com/master/indexes/)。

- **init script**

  Linux平台的[init系统](https://docs.mongodb.com/master/reference/glossary/#term-initsystem)使用的一个简单的shell脚本，用于启动、重启或停止一个[daemon](https://docs.mongodb.com/master/reference/glossary/#term-daemon)进程。如果您通过包管理器安装了MongoDB，那么作为安装的一部分，会为您的系统提供一个init脚本。请参阅相应的[安装指南](https://docs.mongodb.com/master/installation/)来了解您的操作系统。

- **init system**

  init系统是内核启动后在Linux平台上启动的第一个进程，它管理系统上的所有其他进程。init系统使用一个[init脚本](https://docs.mongodb.com/master/reference/glossary/ term-init-script)开始,重新启动,或停止一个[守护进程](https://docs.mongodb.com/master/reference/glossary/ term-daemon)过程,如[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)。Linux的最新版本倾向于使用**systemd** init系统，它使用`systemctl`命令，而旧版本倾向于使用**system V** init系统，它使用`service`命令。请参阅相应的[安装指南](https://docs.mongodb.com/master/installation/)来了解您的操作系统。

- **initial sync**

  [复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)操作，该操作将数据从现有的复制集成员复制到新的复制集成员。请看[初始同步](https://docs.mongodb.com/master/core/replica-set-sync/ # replica-set-initial-sync)。

- **intent lock**

  [lock](https://docs.mongodb.com/master/reference/glossary/ # term-lock)资源,表明锁的持有人将读(intent shared)或写(intent exclusive)资源使用[并发控制](https://docs.mongodb.com/master/reference/glossary/#term-concurrency-control) 比资源更细粒度的概念与意图锁。意图锁允许并发读取和写入资源。查看[MongoDB使用什么类型的锁?](https://docs.mongodb.com/master/faq/concurrency/#faq-concurrency-locking)。

- **interrupt point**

  操作生命周期中可以安全中止的点。MongoDB只在指定的中断点终止操作。参见[终止运行操作](https://docs.mongodb.com/master/tutorial/terminate-runing-operations/)。

- **IPv6**

  对IP(Internet协议)标准的修订，提供更大的地址空间，以更有效地支持当代Internet上的主机数量。

- **ISODate**

  [` mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)使用的国际日期格式来显示日期。格式是:` YYYY-MM-DD HH:MM.SS.millis `。

- **JavaScript**

  一种最初为web浏览器设计的流行脚本语言。MongoDB shell和某些服务器端函数使用JavaScript解释器。更多信息请参见[服务器端JavaScript](https://docs.mongodb.com/master/core/server-sidejavascript/)。

- **journal**

  一种顺序的二进制事务日志，用于在发生硬关闭时使数据库进入有效状态。日志记录首先将数据写入日志，然后写入核心数据文件。MongoDB 2.0及更新版本的64位版本默认允许日志记录。日志文件是预先分配的，并作为文件存在于[data目录](https://docs.mongodb.com/master/reference/glossary/#term-data-directory)中。请看[日志](https://docs.mongodb.com/master/core/journaling/)。

- **JSON**

  JavaScript对象表示法。一种人类可读的纯文本格式，用于表示结构化数据，支持多种编程语言。更多信息，请参见[http://www.json.org](http://www.json.org/)。某些MongoDB工具以JSON格式呈现MongoDB [BSON](https://docs.mongodb.com/master/reference/glossary/#term-bson)文档的近似值。参见[MongoDB Extended JSON (v2)](https://docs.mongodb.com/master/reference/mongodb- extendedjson/)。

- **JSON document**

  一个[JSON](https://docs.mongodb.com/master/reference/glossary/#term-json)文档是结构化格式的字段和值的集合。对于示例JSON文档，请参见http://json.org/example.html。

- **JSONP**

  [JSON](https://docs.mongodb.com/master/reference/glossary/ # term-json)填充。引用一种将JSON注入应用程序的方法。**表示潜在的安全问题**。

- **least privilege**

  一种授权策略，只向用户提供对该用户的工作至关重要的访问权限，而不提供其他权限。

- **legacy coordinate pairs**

  该格式用于MongoDB 2.4版本之前的[geospatial](https://docs.mongodb.com/master/reference/glossary/#term-geospatial)数据。这种格式将地理空间数据存储为平面坐标系统上的点(例如。`[x, y] `)。参见[地理空间查询](https://docs.mongodb.com/master/geospatial-queries/)。

- **LineString**

  LineString是由两个或多个位置组成的数组定义的。具有四个或更多位置的封闭LineString称为线性环，如GeoJSON LineString规范所述:https://tools.ietf.org/html/rfc7946#section-3.1.4。要在MongoDB中使用LineString，请参见[GeoJSON Objects](https://docs.mongodb.com/master/reference/geojson/#geospatial-indexes-store-geojson)。

- **lock**

  MongoDB使用锁来确保[并发](https://docs.mongodb.com/master/faq/concurrency/)不会影响正确性。MongoDB使用[read locks](https://docs.mongodb.com/master/reference/glossary/#term-read-lock)、[write locks](https://docs.mongodb.com/master/reference/glossary/# term-inting-lock)和[intent locks](https://docs.mongodb.com/master/reference/glossary/# term-inting-lock)。更多信息，请参见[MongoDB使用什么类型的锁定?](https://docs.mongodb.com/master/faq/concurrency/#faq-concurrency-locking)。

- **LVM**

  逻辑卷管理器。LVM是一个从物理设备提取磁盘映像的程序，它提供了许多对系统管理有用的原始磁盘操作和快照功能。有关LVM和MongoDB的信息，请参见[在Linux上使用LVM进行备份和恢复](https://docs.mongodb.com/master/tutorial/backupwith-filesystem-snapshots/ #lvm-backup-and-restore)。

- **map-reduce**

  数据处理和聚合范例由选择数据的“映射”阶段和转换数据的“减少”阶段组成。在MongoDB中，您可以使用map-reduce在数据上运行任意的聚合。对于map-reduce实现，请参见[map-reduce](https://docs.mongodb.com/master/core/map-reduce/)。对于所有的聚合方法，请参见[aggregation](https://docs.mongodb.com/master/aggregation/)。

- **mapping type**

  一种将键与值相关联的编程语言结构，其中键可以嵌套其他键和值对(例如字典、hash表、映射和关联数组)。这些结构的属性取决于语言规范和实现。通常，映射类型中的键的顺序是任意的，不能保证。

- **md5**

  一种hashing算法，用于有效地提供可重现的惟一字符串来识别和[校验和](https://docs.mongodb.com/master/reference/glossary/#term-checksum)数据。MongoDB使用md5为[GridFS](https://docs.mongodb.com/master/reference/glossary/#term-gridfs)识别数据块。参见[filemd5](https://docs.mongodb.com/master/reference/command/filemd5/)。

- **MIB**

  管理信息基础。MongoDB在MongoDB企业版中使用MIB文件定义SNMP跟踪的数据类型。

- **MIME**

  多用途因特网邮件扩展。一组标准的类型和编码定义，用于在多个数据存储、传输和电子邮件上下文中声明数据的编码和类型。[`mongofiles `](https://docs.mongodb.com/databe-tools/mongofiles/# bin.mongofiles)工具提供了一个选项来指定MIME类型来描述插入到[GridFS](https://docs.mongodb.com/master/reference/glossary/#term-gridfs)存储中的文件。

- **mongo**

  MongoDB shell。[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/ bin.mongo)流程启动MongoDB shell[守护进程](https://docs.mongodb.com/master/reference/glossary/ # term-daemon)连接到一个[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)实例。shell有一个JavaScript接口。参见[mongo](https://docs.mongodb.com/master/reference/program/mongo/)和[mongo Shell方法](https://docs.mongodb.com/master/reference/method/)。

- **mongod**

  MongoDB数据库服务器。[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)进程启动MongoDB服务器作为一个[守护进程](https://docs.mongodb.com/master/reference/glossary/#term-daemon)。MongoDB服务器管理数据请求和格式，并管理后台操作。参见[mongod](https://docs.mongodb.com/master/reference/program/mongod/)。

- **mongos**

  MongoDB分片集群查询路由器。[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)进程启动MongoDB路由器作为一个[daemon](https://docs.mongodb.com/master/reference/glossary/#term-daemon)。MongoDB路由器充当应用程序和MongoDB [sharded集群](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster)之间的接口，并在集群中处理所有路由和负载平衡。参见[mongos](https://docs.mongodb.com/master/reference/program/mongos/)。

- **namespace**

  MongoDB中集合或索引的规范名称。命名空间是数据库名称和集合或索引名称的组合，如`[database-name].[collection-or-index] `。所有文档都属于一个名称空间。参见[名称空间](https://docs.mongodb.com/master/reference/limits/ # faq-dev-namespace)。

- **natural order**

  数据库引用磁盘上文档的顺序。这是默认的排序顺序。查看[`$natural`](https://docs.mongodb.com/master/reference/operator/meta/natural/#metaOp._S_natural)和[以自然顺序返回](https://docs.mongodb.com/master/reference/method/cursor.sort/# Return -natural- Order)。

- **network partition**

  一种网络故障，它将分布式系统分割为多个分区，使得一个分区中的节点无法与另一个分区中的节点通信。有时，分区是部分的或不对称的。部分分区的一个例子将是一个网络的节点分成三组,第一组内的成员不能与第二组的成员,反之亦然,但所有节点可以与第三组的成员交流。在一个不对称的分区,沟通可能只有当它源自某些节点。例如，分区一端的节点只有在它们启动通信通道时才能与另一端通信。

- **ObjectId**

  一个特殊的12字节[BSON](https://docs.mongodb.com/master/reference/glossary/#term-bson)类型，它保证了[集合](https://docs.mongodb.com/master/reference/glossary/#term-collection)中的唯一性。ObjectId是基于时间戳、机器ID、进程ID和进程本地增量计数器生成的。MongoDB使用ObjectId值作为[_id](https://docs.mongodb.com/master/reference/glossary/#term-id)字段的默认值。

- **operator**

  以`$ `开头的关键字，用于表示更新、复杂查询或数据转换。例如，` $gt `是查询语言的" greater than "操作符。有关可用的操作符，请参见[operators](https://docs.mongodb.com/master/reference/operator/)。

- **oplog**

  一个[capped collection](https://docs.mongodb.com/master/reference/glossary/#term-capped-collection)，它将逻辑写入的有序历史存储到MongoDB数据库中。oplog是在MongoDB中启用[复制](https://docs.mongodb.com/master/reference/glossary/#term-replication)的基本机制。参见[Replica Set Oplog](https://docs.mongodb.com/master/core/replica-set-oplog/)。

- **optime**

  *以下描述了MongoDB 3.2：*中引入的[` protocolVersion: 1 `](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.protocolVersion)使用的optime格式。对复制[oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)中位置的引用。optime值是一个文档，其中包含:`ts `、操作的[时间戳](https://docs.mongodb.com/master/reference/bson-types/# documenting-bson -type-timestamp)。`t `， [`term`](https://docs.mongodb.com/master/reference/command/replSetGetStatus/#replSetGetStatus.term)，该操作最初在主服务器上生成。

- **ordered query plan**

  一个查询计划，它以与[`sort()`](https://docs.mongodb.com/master/reference/method/cursor.sort/#cursor.sort)顺序一致的顺序返回结果。[查询计划](https://docs.mongodb.com/master/core/query-plans/ # read-operations-query-optimization)。

- **orphaned document**

  在分片集群中，孤立文档是指某个分片上的文档，由于迁移失败或由于异常关机而导致迁移清理不完整，这些文档也存在于其他分片上的块中。从MongoDB 4.4开始，在块迁移完成后，孤立的文档会被自动清理。删除孤立文档不再需要运行[`cleanuporphaned`](https://docs.mongodb.com/master/reference/command/cleanupOrphaned/#dbcmd.cleanupOrphaned)。

- **passive member**

  一个[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的成员不能成为主元素，因为它的[`members[n].priority`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.members[n].priority)是`0`。参见[Priority 0 Replica Set Members](https://docs.mongodb.com/master/core/replica-setprior-0 -member/)。

- **PID**

  一个进程标识符。类unix系统为每个正在运行的进程分配一个唯一的整数PID。可以使用PID检查正在运行的进程并向其发送信号。参见[/proc文件系统](https://docs.mongodb.com/master/reference/ulimit/# proc-filessystem)。

- **pipe**

  类unix系统中的一种通信通道，允许独立进程发送和接收数据。在UNIX shell中，管道操作允许用户将一个命令的输出定向到另一个命令的输入。

- **pipeline**

  一个[聚合](https://docs.mongodb.com/master/reference/glossary/#term-aggregation)流程中的一系列操作。看到[聚合管道](https://docs.mongodb.com/master/core/aggregation-pipeline/)。

- **Point**

  GeoJSON点规范中描述的单个坐标对:https://tools.ietf.org/html/rfc7946#section-3.1.2。要在MongoDB中使用一个点，请参见[GeoJSON Objects](https://docs.mongodb.com/master/reference/geojson/#geospatial-indexes-store-geojson)。

- **Polygon**

  一个[LinearRing](https://docs.mongodb.com/master/reference/glossary/#term-linestring)坐标数组，正如在GeoJSON多边形规范中描述的:https://tools.ietf.org/html/rfc7946#section-3.1.6。对于有多个环的多边形，第一个必须是外环，其他必须是内环或孔。MongoDB不允许外环自交。内环必须完全包含在外环内，不能相互交叉或重叠。参见[GeoJSON对象](https://docs.mongodb.com/master/reference/geojson/ # geospatial-indexes-store-geojson)。

- **powerOf2Sizes**

  每个集合设置改变和规范MongoDB为每个[文档](https://docs.mongodb.com/master/reference/glossary/#term-document)分配空间的方式，以最大化存储重用和减少碎片。这是[TTL集合](https://docs.mongodb.com/master/tutorial/expire-data/)的默认值。查看[collMod](https://docs.mongodb.com/master/reference/command/collMod/)和`usepowerof2size`。

- **pre-splitting**

  在插入数据之前执行的一种操作，它将可能的切分键值范围划分为块，以方便插入和高写吞吐量。在某些情况下预加速文件的初始分布[分片集群](https://docs.mongodb.com/master/reference/glossary/ # term-sharded-cluster)通过手动划分集而不是等待MongoDB[均衡器](https://docs.mongodb.com/master/reference/glossary/ # term-balancer)。参见[在分片集群中创建块](https://docs.mongodb.com/master/tutorial/create-chunks-in-shard-cluster/)。

- **prefix compression**

  通过在每一页内存中只存储一次相同的索引键前缀，减少内存和磁盘消耗。参见:[压缩](https://docs.mongodb.com/master/core/wiredtiger/# storagewiredtiger - Compression)了解更多关于WiredTiger的压缩行为。

- **primary**

  在[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)中，主元素是接收所有写操作的成员。参见[Primary](https://docs.mongodb.com/master/core/replica-set-members/ # replica-set-primary-member)。

- **primary key**

  记录的唯一不可变标识符。在[RDBMS](https://docs.mongodb.com/master/reference/glossary/#term-rdbms)中，主键通常是存储在每行' id '字段中的整数。在MongoDB中，[_id](https://docs.mongodb.com/master/reference/glossary/#term-id)字段持有文档的主键，通常是BSON [ObjectId](https://docs.mongodb.com/master/reference/glossary/#term-objectid)。

- **primary shard**

  [shard](https://docs.mongodb.com/master/reference/glossary/#term-shard)，它包含所有未分片的集合。参见[Primary Shard](https://docs.mongodb.com/master/core/sharded-cluster-shards/ #主碎片)。

- **priority**

  一个可配置的值，帮助确定[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)中的哪些成员最有可能成为[primary](https://docs.mongodb.com/master/reference/glossary/#term-primary)。参见 [`members[n].priority`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.members[n].priority).

- **privilege**

  资源上允许的指定的[资源](https://docs.mongodb.com/master/reference/glossary/#term-resource)和[actions](https://docs.mongodb.com/master/reference/glossary/#term-action)的组合。参见[privilege](https://docs.mongodb.com/master/core/authorization/特权)。

- **projection**

  一个给[查询](https://docs.mongodb.com/master/reference/glossary/#term-query)的文档，它指定MongoDB在结果集中返回哪些字段。有关投影操作符的列表，请参见[投影操作符](https://docs.mongodb.com/master/reference/operator/projection/)。

- **query**

  读请求。MongoDB使用[JSON](https://docs.mongodb.com/master/reference/glossary/#term-json)类似的查询语言，包括各种各样的[查询操作符](https://docs.mongodb.com/master/reference/glossary/#term-operator)，名称以“$”字符开头。[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/ bin.mongo)shell,你可以发出查询使用[`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/ db.collection.find)和[`db.collection.findOne()`](https://docs.mongodb.com/master/reference/method/db.collection.findOne/ # db.collection.findOne)方法。参见[查询文件](https://docs.mongodb.com/master/tutorial/query-documents/ # read-operations-queries)。

- **query optimizer**

  生成查询计划的流程。对于每个查询，优化器都会生成一个计划，将查询与尽可能高效地返回结果的索引相匹配。优化器在每次运行查询时重用查询计划。如果一个集合发生重大变化，优化器将创建一个新的查询计划。参见[查询计划](https://docs.mongodb.com/master/core/query-plans/ # read-operations-query-optimization)。

- **query shape**

  查询谓词、排序和投影的组合。对于查询谓词，只有谓词的结构(包括字段名)是重要的;查询谓词中的值不重要。因此，查询谓词`{type: 'food'} `等价于查询形状的查询谓词`{type: 'utensil'} `。来帮助识别相同的慢速查询[查询形状](https://docs.mongodb.com/master/reference/glossary/ # term-query-shape),开始在MongoDB 4.2中,每个[查询形状](https://docs.mongodb.com/master/reference/glossary/ # term-query-shape)是与[queryHash](https://docs.mongodb.com/master/release-notes/4.2/ # query-hash)。`queryHash`是一个十六进制字符串，表示查询形状的散列，并且只依赖于查询形状。对于任何散列函数，两个不同的查询形状可能会导致相同的散列值。但是，不同查询形状之间不太可能出现哈希冲突。

- **RDBMS**

  关系数据库管理系统。基于关系模型的数据库管理系统，通常使用[SQL](https://docs.mongodb.com/master/reference/glossary/#term-sql)作为查询语言。

- **read concern**

  指定读操作的隔离级别。例如，您可以使用read concern来只读已经传播到[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)中的大多数节点的数据。参见[读问题](https://docs.mongodb.com/master/reference/read-concern/)。

- **read lock**

  资源上的一个共享[锁](https://docs.mongodb.com/master/reference/glossary/#term-lock)，该资源(比如集合或数据库)在持有时允许并发读取但不允许写入。查看[MongoDB使用什么类型的锁?](https://docs.mongodb.com/master/faq/concurrency/#faq-concurrency-locking)。

- **read preference**

  决定客户端如何直接读取操作的设置。读取首选项影响所有副本集，包括分片副本集。默认情况下，MongoDB将读取定向到[初选](https://docs.mongodb.com/master/reference/glossary/#term-primary)。但是，您也可以为[最终一致](https://docs.mongodb.com/master/reference/glossary/#term-eventual-consistency)读取直接将读取指向二级。参见[阅读偏好](https://docs.mongodb.com/master/core/read-preference/)。

- **recovering**

  [replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)成员状态，表示成员还没有准备好开始辅助或主成员的正常活动。正在恢复的成员不可用于读取。

- **replica pairs**

  MongoDB的前身[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set).*自1.6版本以来已被弃用*.

- **replica set**

  实现复制和自动故障转移的MongoDB服务器集群。MongoDB推荐的复制策略。参见[复制](https://docs.mongodb.com/master/replication/)。

- **replication**

  允许多个数据库服务器共享相同数据的特性，从而确保冗余和促进负载平衡。参见[复制](https://docs.mongodb.com/master/replication/)。

- **replication lag**

  最后一个操作之间的时间长度[primary’s](https://docs.mongodb.com/master/reference/glossary/ # term-primary) [oplog](https://docs.mongodb.com/master/reference/glossary/ term-oplog)和最后一个操作应用于一个特定的[二级](https://docs.mongodb.com/master/reference/glossary/ # term-secondary)。通常，您希望将复制延迟保持得尽可能小。参见[复制延迟](https://docs.mongodb.com/master/tutorial/troubleshoot-replica-sets/ # replica-set-replication-lag)。

- **resident memory**

  当前存储在物理RAM中的应用程序内存的子集。常驻内存是[虚拟内存](https://docs.mongodb.com/master/reference/glossary/#term-virtual-memory)的一个子集，其中包括映射到物理RAM和磁盘的内存。

- **resource**

  数据库、集合、集合集或集群。一个[特权](https://docs.mongodb.com/master/reference/glossary/#term-action)允许在指定的资源上执行[动作](https://docs.mongodb.com/master/reference/glossary/#term-action)。参见[资源](https://docs.mongodb.com/master/reference/resource-document/ resource-document)。

- **role**

  在指定的[资源](https://docs.mongodb.com/master/reference/glossary/#term-action)上允许[操作](https://docs.mongodb.com/master/reference/glossary/#term-resource)的一组特权。分配给用户的角色决定了用户对资源和操作的访问。参见[安全](https://docs.mongodb.com/master/security/)。

- **rollback**

  恢复写操作以确保所有复制集成员的一致性的进程。参见[复制集故障转移期间回滚](https://docs.mongodb.com/master/core/replica-set-rollbacks/#replica-set-rollback)。

- **secondary**

  复制主数据库内容的[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)成员。辅助成员可以处理读请求，但是只有[主](https://docs.mongodb.com/master/reference/glossary/#term-primary)成员可以处理写操作。参见[Secondaries](https://docs.mongodb.com/master/core/replica-set-members/ replica-set-secondary-members)。

- **secondary index**

  一个数据库[索引](https://docs.mongodb.com/master/reference/glossary/#term-index)，通过最小化查询引擎执行查询时必须执行的工作来提高查询性能。参见[索引](https://docs.mongodb.com/master/indexes/)。

- **set name**

  任意的名字给一个复制集。复制集的所有成员必须具有相同的名称指定的[`replSetName`](https://docs.mongodb.com/master/reference/configuration-options/ # replication.replSetName)设置或[`——replSet `](https://docs.mongodb.com/master/reference/program/mongod/ # cmdoption-mongod-replset)选项。

- **shard**

  一个[` mongod`](https://docs.mongodb.com/master/reference/program/mongod/ # bin.mongod)实例或[复制集](https://docs.mongodb.com/master/reference/glossary/ # term-replica-set)存储的[分片集群的](https://docs.mongodb.com/master/reference/glossary/ # term-sharded-cluster)一部分数据集。在生产中,所有分片都应该复制集。参见[分片](https://docs.mongodb.com/master/core/sharded-cluster-shards/)。

- **shard key**

  MongoDB用于在[分片集群](https://docs.mongodb.com/master/reference/glossary/#term-shard-cluster)的成员之间分发文档的字段。参见[分片键](https://docs.mongodb.com/master/core/sharding-shard-key/ #分片键)。

- **sharded cluster**

  包含[sharded](https://docs.mongodb.com/master/reference/glossary/#term-sharding) MongoDB部署的节点集。分片集群由配置服务器、分片和一个或多个[`mongos `](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)路由进程组成。参见[分片集群组件](https://docs.mongodb.com/master/core/shar-cluster-components/)。

- **sharding**

  按键范围划分数据并将数据分布在两个或多个数据库实例之间的数据库体系结构。切分允许水平伸缩。参见[分片](https://docs.mongodb.com/master/sharding/)。

- **shell helper**

  `mongo`shell中的一个方法，它为[数据库命令](https://docs.mongodb.com/master/reference/command/)提供了更简洁的语法。Shell helper改善了一般的交互体验。参见[mongo Shell方法](https://docs.mongodb.com/master/reference/method/)。

- **single-master replication**

  一个[replication](https://docs.mongodb.com/master/reference/glossary/#term-replication) topology ，其中只有一个数据库实例接受写操作。单主复制确保了一致性，是MongoDB使用的复制topology 。参见[Replica Set Primary](https://docs.mongodb.com/master/core/replica-setprimary/)。

- **snappy**

  一个压缩/解压缩库，设计来平衡有效的计算需求与合理的压缩率。Snappy是MongoDB使用[WiredTiger](https://docs.mongodb.com/master/core/wiredtiger/# storagewiredtiger)的默认压缩库。更多信息，请参见[Snappy](https://google.github.io/snappy/)和[WiredTiger压缩文档](https://source.wiredtiger.com/mongodb-3.4/compression.html)。

- **split**

  [分片集群](https://docs.mongodb.com/master/reference/glossary/#term-chunk)中的[chunks](https://docs.mongodb.com/master/reference/glossary/#term- shard-cluster)的划分。参见[使用块进行数据分区](https://docs.mongodb.com/master/core/sharding-data-partitioning/)。

- **SQL**

  结构化查询语言(Structured Query Language, SQL)是一种通用的特殊用途编程语言，用于与关系数据库进行交互，包括访问控制、插入、更新、查询和删除。不同数据库供应商支持的基本SQL语法中有一些类似的元素，但是大多数实现都有自己的方言、数据类型和对提议的SQL标准的解释。复杂的SQL通常不能在主要的[RDBMS](https://docs.mongodb.com/master/reference/glossary/#term-rdbms)产品之间直接移植。“SQL”经常被用作关系数据库的转喻。

- **SSD**

  固态磁盘。一种高性能的磁盘驱动器，使用固态电子器件来保持性能，与传统机械硬盘驱动器所使用的旋转磁盘和可移动读写磁头不同。

- **standalone**

  一个[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)的实例，它作为一个单独的服务器运行，而不是作为[replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的一部分。要将独立转换为复制集，请参见[将独立转换为复制集](https://docs.mongodb.com/master/tutorial/convert-standal1-replica-set/)。

- **storage engine**

  数据库中负责管理如何在内存和磁盘中存储和访问数据的部分。对于特定的工作负载，不同的存储引擎执行得更好。请参阅[Storage Engines](https://docs.mongodb.com/master/core/storageengines/)了解MongoDB中内置存储引擎的具体细节。

- **storage order**

  参见[natural order](https://docs.mongodb.com/master/reference/glossary/#term-natural-order).

- **strict consistency**

  分布式系统的一种属性，要求所有成员始终反映系统的最新更改。在数据库系统中，这意味着任何能够提供数据的系统都必须始终反映最新的写操作。

- **sync**

  [replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)操作，其中成员从[primary](https://docs.mongodb.com/master/reference/glossary/#term-primary)复制数据。同步首先发生在MongoDB创建或恢复一个成员时，该成员被称为[initial Sync](https://docs.mongodb.com/master/reference/glossary/#term-initial-sync)。然后持续进行同步，以通过复制集数据的更改更新成员。查看[Replica Set Data Synchronization](https://docs.mongodb.com/master/core/replica-set-sync/)。

- **syslog**

  在类unix系统上，为服务器和进程提供提交日志信息的统一标准的日志过程。MongoDB提供了一个将输出发送到主机的syslog系统的选项。参见[' syslogFacility '](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.syslogFacility)。

- **tag**

  应用于复制集成员的标签，由客户端用于发出感知数据中心的操作。使用标签复制集的更多信息,参见本手册的以下部分:[阅读偏好标记集](https://docs.mongodb.com/master/core/read-preference-tags/ # replica-set-read-preference-tag-sets)。

  *3.4版本中改变:*在MongoDB 3.4中,分片集群[`zones`](https://docs.mongodb.com/master/reference/glossary/) term-zone取代[`tags`](https://docs.mongodb.com/master/reference/glossary/ # term-tag)。

- **tag set**

  包含零个或多个[标签](https://docs.mongodb.com/master/reference/glossary/#term-tag)的文档。

- **tailable cursor**

  对于一个[capped集合](https://docs.mongodb.com/master/reference/glossary/#term-capped-collection)，一个可tailable游标是一个在客户端在初始游标中查看完结果后保持打开的游标。当客户端向有上限的集合插入新文档时，可定制游标将继续检索文档。

- **term**

  对于一个复制集的成员，一种单调递增的数目，对应于一次选举尝试。

- **topology**

  部署的MongoDB实例的状态,包括部署的类型(即独立、复制集,或分片集群),以及服务器的可用性,和每个服务器的角色(例如[主要](https://docs.mongodb.com/master/reference/glossary/ # term-primary),[二级](https://docs.mongodb.com/master/reference/glossary/ # term-secondary),[配置服务器](https://docs.mongodb.com/master/reference/glossary/ # term-config-server),或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ bin.mongos))。

- TSV

  一种基于文本的数据格式，由制表符分隔的值组成。这种格式通常用于在关系数据库之间交换数据，因为这种格式非常适合表格数据。您可以使用[` mongoimport `](https://docs.mongodb.com/databe-tools/mongoimport/ #bin.mongoimport)导入TSV文件。

- TTL

  表示“生存时间”，表示给定信息在缓存或其他临时存储中保留的过期时间或期间，然后系统将其删除或老化。MongoDB有一个TTL集合特性。查看[通过设置TTL从集合过期数据](https://docs.mongodb.com/master/tutorial/expire-data/)。

- **unique index**

  一种索引，强制跨单个集合的特定字段具有唯一性。参见[独特的索引](https://docs.mongodb.com/master/core/index-unique/ # index-type-unique)。

- **unix epoch**

  1970年1月1日00时。通常用于表示时间，其中从这个点开始计算的秒数或毫秒数。

- **unordered query plan**

  返回的查询计划的顺序与[` sort() `](https://docs.mongodb.com/master/reference/method/cursor.sort/#cursor.sort)顺序不一致。参见[查询计划](https://docs.mongodb.com/master/core/query-plans/ # read-operations-query-optimization)。

- **upsert**

  更新操作的选项;例如[` db.collection.update () `](https://docs.mongodb.com/master/reference/method/db.collection.update/ # db.collection.update), [`db.collection.findAndModify ()`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/ # db.collection.findAndModify)。如果设置为true，更新操作将更新指定查询匹配的文档，如果没有文档匹配，则插入一个新文档。新文档将在操作中指示字段。参见[如果不存在匹配，插入新文档(Upsert)](https://docs.mongodb.com/master/reference/method/db.collection.update/#upsert-parameter)。

- **virtual memory**

  应用程序的工作内存，通常驻留在磁盘和物理RAM中。

- **WGS84**

  默认的参考系统和大地基准，MongoDB使用它来计算类似地球的球体上的几何图形，用于在[GeoJSON](https://docs.mongodb.com/master/reference/glossary/#term-geojson)对象上的地理空间查询。请参阅“**EPSG:4326: WGS 84**”规范:http://spatialreference.org/ref/epsg/4326/。

- **working set**

  MongoDB最常用的数据。

- **write concern**

  指定写操作是否成功。Write concern允许您的应用程序检测插入错误或不可用[`mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例。对于[replica sets](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)，您可以配置write concern来确认复制到指定数量的成员。请看[写问题](https://docs.mongodb.com/master/reference/write-concern/)。

- **write conflict**

  在这种情况下，两个并发操作(其中至少一个是写操作)试图以违反使用乐观[并发控制](https://docs.mongodb.com/master/reference/glossary/#term-concurrening-control)的存储引擎施加的约束的方式使用资源。MongoDB将透明地中止并重试其中一个冲突的操作。

- **write lock**

  资源(比如集合或数据库)上的独占[锁](https://docs.mongodb.com/master/reference/glossary/#term-lock)。当一个进程写入一个资源时，它采用独占写锁来防止其他进程写入或读取该资源。有关锁的更多信息，请参见[FAQ: Concurrency](https://docs.mongodb.com/master/faq/concurrency/)。

- **writeBacks**

  切分系统内的进程确保向[shard](https://docs.mongodb.com/master/reference/glossary/#term-shard)发出的不负责相关块的写被应用到适当的切分。有关信息，请参见[writebacklisten在日志中的意思是什么?](https://docs.mongodb.com/master/faq/sharding/#faq-writebacklisten)和[writeBacksQueued](https://docs.mongodb.com/master/reference/command/serverStatus/# server-stat-writebacksqueued)。

- **zlib**

  与MongoDB使用的[snappy](https://docs.mongodb.com/master/reference/glossary/#term-snappy)相比，这个数据压缩库提供了更高的压缩率，但占用了更多的CPU。您可以配置[WiredTiger](https://docs.mongodb.com/master/core/wiredtiger/# storagewiredtiger)来使用zlib作为其压缩库。更多信息请参见[http://www.zlib.net](http://www.zlib.net/)和[WiredTiger压缩文档](https://source.wiredtiger.com/mongodb-3.4/compression.html)。

- **zone**

  *3.4版本中的新特性:*给定分片集合的基于范围[分片键](https://docs.mongodb.com/master/reference/glossary/#term-shard-key)值的文档分组。分片集群中的每个碎片可以与一个或多个区域关联。在一个平衡的集群中，MongoDB只将一个区域覆盖的读和写定向到该区域内的那些碎片。有关更多信息，请参阅[zone](https://docs.mongodb.com/master/core/zone-sharding/#zone-sharding)手册页。

  在MongoDB 3.2中，区域取代了[标签](https://docs.mongodb.com/master/reference/glossary/#term-tag)所描述的功能。

- **zstd**

  *4.2版中的新功能。*
  
  与[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib)相比，该数据压缩库提供更高的压缩率和更低的CPU使用率。

