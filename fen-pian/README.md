# MongoDB分片

**在本页面**

* [分片集群](https://docs.mongodb.com/manual/sharding/#sharded-cluster)
* [分片键](https://docs.mongodb.com/manual/sharding/#shard-keys)
* [Chunks块](https://docs.mongodb.com/manual/sharding/#chunks)
* [均衡器和均匀分配](https://docs.mongodb.com/manual/sharding/#balancer-and-even-chunk-distribution)
* [分片的优点](https://docs.mongodb.com/manual/sharding/#advantages-of-sharding)
* [分片前的注意事项](https://docs.mongodb.com/manual/sharding/#considerations-before-sharding)
* [分片和非分片集合](https://docs.mongodb.com/manual/sharding/#sharded-and-non-sharded-collections)
* [连接到分片群集](https://docs.mongodb.com/manual/sharding/#connecting-to-a-sharded-cluster)
* [分片策略](https://docs.mongodb.com/manual/sharding/#sharding-strategy)
* [分片集群中的区域](https://docs.mongodb.com/manual/sharding/#zones-in-sharded-clusters)
* [分片中的排序规则](https://docs.mongodb.com/manual/sharding/#collations-in-sharding)
* [变更流](https://docs.mongodb.com/manual/sharding/#change-streams)
* [事务](https://docs.mongodb.com/manual/sharding/#transactions)

[分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)是一种将数据分配到多个机器上的方法。MongoDB通过分片技术来支持具有海量数据集和高吞吐量操作的部署方案。

数据库系统的数据集或应用的吞吐量比较大的情况下，会给单台服务器的处理能力带来极大的挑战。例如，高查询率会耗尽服务器的CPU资源。工作的数据集大于系统的内存压力、磁盘驱动器的I/O容量。

解决系统增长的方法有两种：垂直扩展和水平扩展。

_垂直扩展_ 通过增加单个服务器的能力来实现，例如使用更强大的CPU，增加更多的内存或存储空间量。由于现有技术的局限性，不能无限制地增加单个机器的配置。此外，云计算供应商提供可用的硬件配置具有严格的上限。其结果是，垂直扩展有一个实际的最大值。

_水平扩展_是通过将系统数据集划分至多台机器，并根据需要添加服务器来提升容量。虽然单个机器的总体速度或容量可能不高，但每台机器只需处理整个数据集的某个子集，所以可能会提供比单个高速大容量服务器更高的效率，而且机器的数量只需要根据数据集大小来进行扩展，与单个机器的高端硬件相比，这个方案可以降低总体成本。不过，这种方式会提高基础设施部署维护的复杂性。

MongoDB通过[分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)来实现水平扩展。

## 分片集群

MongoDB分片[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)包括以下组件：

* [分片](https://docs.mongodb.com/manual/core/sharded-cluster-shards/)：每个shard（分片）包含被分片的数据集中的一个子集。每个分片可以被部署为[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)架构。
* [mongos](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/)：`mongos`充当查询路由器，在客户端应用程序和分片集群之间提供接口。从MongoDB 4.4开始，`mongos`可以支持 [对冲读取（hedged reads）](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads)以最大程度地减少延迟。
* [config服务器](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/)：config servers存储了分片集群的元数据和配置信息。

下图描述了分片集群中各组件的交互：

![Diagram of a sample sharded cluster for production purposes. Contains exactly 3 config servers, 2 or more \`\`mongos\`\` query routers, and at least 2 shards. The shards are replica sets.](https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg)

MongoDB在[集合](https://docs.mongodb.com/manual/reference/glossary/#term-collection)级别分片数据，将收集数据分布在集群中的各个分片上。

## 分片键

MongoDB使用分片[键](https://docs.mongodb.com/manual/core/sharding-shard-key/)在各个分片之间分发集合中的文档。分片键由文档中的一个或多个字段组成。

* 从版本4.4开始，分片集合中的文档可能缺少分片键字段。在跨分片分布文档时，缺少分片键字段将被视为具有空值，但在路由查询时则不会。有关更多信息，请参见 [分片键缺失](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-missing)。
* 在4.2及更早版本中，分片键字段必须在每个文档中存在一个分片集合。

在[分片集合](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-creation)时选择[分片](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-creation)键。

* 从MongoDB 4.4开始，您可以通过向现有键中添加一个或多个后缀字段来优化集合的分片键。有关详细信息，请参见[`refineCollectionShardKey`](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#dbcmd.refineCollectionShardKey)。
* 在MongoDB 4.2和更低版本中，无法在分片后更改分片键的选择。

文档的分片键值决定了其在各个分片中的分布。

* 从MongoDB 4.2开始，您可以更新文档的分片键值，除非您的分片键字段为不可变`_id`字段。有关更多信息，请参见 [更改文档的分片键值](https://docs.mongodb.com/manual/core/sharding-shard-key/#update-shard-key)。
* 在MongoDB 4.0和更早版本中，文档的分片键字段值是不可变的。

### 分片键索引

要对已填充的集合进行分片，该集合必须具有以分片键开头的[索引](https://docs.mongodb.com/manual/reference/glossary/#term-index)。分片一个空集合时，如果该集合还没有针对指定分片键的适当索引，则MongoDB会创建支持索引。请参阅分片[键索引](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-indexes)。

### 分片键策略


分片键的选择会影响分片群集的性能，效率和可伸缩性。选择分片键可以使具有最佳硬件和基础结构的群集成为瓶颈。[分片](https://docs.mongodb.com/manual/sharding/#sharding-strategy)键及其后备索引的选择也会影响群集可以使用的[分片策略](https://docs.mongodb.com/manual/sharding/#sharding-strategy)。

有关更多信息，请参见[选择分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection)文档。

## Chunks块


MongoDB将分片数据拆分成[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。每个分块都有一个基于分片[键的](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)上下限范围 。

##  Bncer and Even Chunk Distribution 均衡器和均匀分配


[均衡器](https://docs.mongodb.com/manual/core/sharding-balancer-administration/)通过在后台迁移各个[分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)上的[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)，来实现集群的所有分片中块的均匀分布。

有关更多信息，请参见[使用块](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)进行[数据分区](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)。

## 分片的优势

### 读写负载


MongoDB将读写工作负载分布在分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)中的各个分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)上，从而允许每个分片处理集群操作的子集。通过添加更多分片，可以在集群中水平扩展读写工作负载。

对于包含分片键或[复合分片](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)键的前缀的查询，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以将查询定位到特定的分片或一组分片。这些[目标操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted)通常比[广播](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)到集群中的每个分片更有效。


从MongoDB 4.4开始，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以支持[对冲读取（hedged reads）](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads)以最大程度地减少延迟。

### 存储容量

通过[分片技术](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)将数据分布到分片集群中的各个[分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)中，每个分片只需存储数据集中的部分子集。随着数据集的增长， 通过增加分片的数量即可增加整个集群的存储容量。

### 高可用性


将配置服务器和分片作为副本集进行部署可提高可用性。


即使一个或多个分片副本集变得完全不可用，分片群集也可以继续提供部分的读取或写入服务。也就是说，虽然停机期间无法访问不可用分片中的数据子集，但是针对可用分片执行读取或写入操作仍然可以成功。

## 分片前的注意事项


鉴于分片集群基础架构的要求和复杂性，您需要仔细地制定计划、执行和维护。

分片后，MongoDB不会提供任何方法来对分片集群进行分片。

为了确保集群的性能和效率，在选择分片键时需要仔细考虑。请参阅 [选择分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection)。

分片有一定的[操作要求和限制](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/#sharding-operational-restrictions)。有关更多信息，请参见 [分片集群中的操作限制](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/)。

如果查询条件中没有包含拆分键或[复合](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)拆分键的前缀，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)节点将执行[广播操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)，即查询分片集群中的所有分片。这些分散/聚集查询可能会变成长耗时的操作。

> 注意
>
> 如果您与MongoDB签订了有效的支持合同，请考虑与您的客户代表联系，以获取分片集群计划和部署方面的帮助。

## 分片和未分片集合

数据库可以混合使用分片和未分片集合。分片集合被[分区](https://docs.mongodb.com/manual/reference/glossary/#term-data-partition)并分布在集群中的各个[分片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)中。而未分片集合仅存储在[主分片](https://docs.mongodb.com/manual/reference/glossary/#term-primary-shard)中。每个数据库都有自己的主分片。

![Diagram of a primary shard. A primary shard contains non-sharded collections as well as chunks of documents from sharded collections. Shard A is the primary shard.](https://docs.mongodb.com/manual/_images/sharded-cluster-primary-shard.bakedsvg.svg)

## 连接到分片集群

您必须连接到[mongos](https://docs.mongodb.com/manual/reference/glossary/#term-mongos)路由器才能与分片[集群中的](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)任何集合进行交互。这包括分片_和_未分片的集合。客户端永远不要直接连到单个分片来执行读或写操作。

![Diagram of applications/drivers issuing queries to mongos for unsharded collection as well as sharded collection. Config servers not shown.](https://docs.mongodb.com/manual/_images/sharded-cluster-mixed.bakedsvg.svg)

您可以像连接[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)一样来连接[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)，例如通过[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell或MongoDB[驱动程序](https://docs.mongodb.com/ecosystem/drivers?jump=docs)。

## 分片策略

MongoDB支持如下两种分片策略来实现[分片集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)中的分布数据。

### 哈希分片


哈希分片涉及计算分片键字段值的哈希值。然后，根据散列的分片键值为每个[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)分配一个范围。

提示

使用哈希索引解析查询时，MongoDB会自动计算哈希值，**不需要**应用程序来计算。

![Diagram of the hashed based segmentation.](https://docs.mongodb.com/manual/_images/sharding-hash-based.bakedsvg.svg)

尽管一系列分片键可能是“接近”的，但它们的哈希值不太可能在同[一块上](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。基于哈希值的数据分发有助于更均匀的数据分发，尤其是在分片键[单调](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-monotonic)更改的数据集中。

但是，哈希分布意味着对分片键的基于范围的查询不太可能针对单个分片，从而导致更多集群范围的[广播操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)。

有关更多信息，请参见[哈希分片](https://docs.mongodb.com/manual/core/hashed-sharding/)。

### 范围分片

范围分片根据分片键的值将数据划分为多个范围，然后基于分片键的值分配每个[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)的范围。

![Diagram of the shard key value space segmented into smaller ranges or chunks.](https://docs.mongodb.com/manual/_images/sharding-range-based.bakedsvg.svg)

值“接近”的一系列分片键更有可能分布在同[一块上](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。好处是便于[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)执行[针对性的操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted)，可以仅将操作路由到包含所需数据的分片上。

范围分片的效率取决于选择的分片键。分片键考虑不周全会导致数据分布不均，这可能会削弱分片的某些优势或导致性能瓶颈。有关[基于范围的分片，](https://docs.mongodb.com/manual/core/ranged-sharding/#sharding-ranged-shard-key)请参见 [分片键选择](https://docs.mongodb.com/manual/core/ranged-sharding/#sharding-ranged-shard-key)。

有关更多信息，请参见[范围分片](https://docs.mongodb.com/manual/core/ranged-sharding/)。

## 分片集群中的区域

Zones区域可以改善跨多个数据中心的分片集群的数据局部性。

在分片集群中，您可以根据分片键创建分片数据的[区域](https://docs.mongodb.com/manual/reference/glossary/#term-zone)。您可以将每个区域与集群中的一个或多个分片相关联。分片节点可以与任何数量的区域相关联。在分片集群中（已开启均衡器），MongoDB仅迁移到与区域相关的分片节点覆盖的块。

每个区域都覆盖了一个或多个分片[键值](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)的范围。区域覆盖的每个范围始终包括其下边界和上边界。

![Diagram of data distribution based on zones in a sharded cluster](https://docs.mongodb.com/manual/_images/sharded-cluster-zones.bakedsvg.svg)

在为区域定义区域的新范围时，必须使用分片[键中](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)包含的字段。如果使用[复合分片](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)键，则范围必须包含分片键的前缀。有关更多信息，请参见[区域中的分片键](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding-shard-key)。

选择分片键时，应考虑将来可能使用的区域。

> 提示
>
> 从MongoDB 4.0.3开始，在对空集合或不存在的集合进行分片_之前_设置区域和区域范围可以更快地设置区域分片。

有关更多信息，请参见[区域](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding)。


## 分片中的排序规则


使用带有`collation : { locale : "simple" }`选项的[`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection)命令可以对具备[默认排序规则](https://docs.mongodb.com/manual/reference/collation/)的集合进行分片。分片成功需具备下述条件：

* 集合必须具有索引，且该索引的前缀是分片键
* 索引必须具有`{ locale: "simple" }`排序规则 

使用排序规则创建新集合时，请在分片集合之前确保满足这些条件。

> 注意
>
> 分片集合上的查询继续使用为集合配置的默认排序规则。要使用分片索引的简单排序规则，请在查询的[排序文档](https://docs.mongodb.com/manual/reference/collation/)中指定`{locale：“ simple”}`。

请参阅[`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection)以获取有关分片和集群的更多信息。

## Change Streams变更流

从MongoDB 3.6开始，[变更流](https://docs.mongodb.com/manual/changeStreams/)可用于副本集和分片集群。更改流允许应用程序访问实时的数据更改，避免使用tail oplog带来的复杂性和风险。应用程序可以使用变更流来订阅一个或多个集合上的所有数据变更信息。

## 事务

从MongoDB 4.2开始，随着[分布式事务](https://docs.mongodb.com/manual/core/transactions/)功能的引入，[分片](https://docs.mongodb.com/manual/core/transactions/)集群上可以支持多文档事务。

在提交事务之前，在事务外部看不到在事务中进行的数据变更。

但是，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在所有分片上可见。例如，如果提交了一个事务，并且在分片A上可以看到写1，但是在分片B上仍然看不到写2，在外部读取时设置读关注为\[`"local"`\]\(https://docs.mongodb.com/v4.2/reference/read-concern-local/\#readconcern."local"\)，则可以读取写1的结果而看不到写2的结果。


有关详细信息，请参见：

* [事务](https://docs.mongodb.com/manual/core/transactions/)
* [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
* [生产注意事项（分片群集）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)

← [副本集成员状态](https://docs.mongodb.com/manual/reference/replica-states/)  
[分片集群组件](https://docs.mongodb.com/manual/core/sharded-cluster-components/) →

原文链接：[https://docs.mongodb.com/manual/sharding/](https://docs.mongodb.com/manual/sharding/)

译者：桂陈

## MongoDB中文社区

![MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x2014;MongoDB&#x7231;&#x597D;&#x8005;&#x6280;&#x672F;&#x4EA4;&#x6D41;&#x5E73;&#x53F0;](https://mongoing.com/wp-content/uploads/2020/09/6de8a4680ef684d-2.png)

| 资源列表推荐 | 资源入口 |
| :--- | :--- |
| MongoDB中文社区官网 | [https://mongoing.com/](https://mongoing.com/) |
| 微信服务号 ——最新资讯和优质文章 | Mongoing中文社区（mongoing-mongoing） |
| 微信订阅号 ——发布文档翻译内容 | MongoDB中文用户组（mongoing123） |
| 官方微信号 —— 官方最新资讯 | MongoDB数据库（MongoDB-China） |
| MongoDB中文社区组委会成员介绍 | [https://mongoing.com/core-team-members](https://mongoing.com/core-team-members) |
| MongoDB中文社区翻译小组介绍 | [https://mongoing.com/translators](https://mongoing.com/translators) |
| MongoDB中文社区微信技术交流群 | 添加社区助理小芒果微信（ID:mongoingcom），并备注 mongo |
| MongoDB中文社区会议及文档资源 | [https://mongoing.com/resources](https://mongoing.com/resources) |
| MongoDB中文社区大咖博客 | [基础知识](https://mongoing.com/basic-knowledge)  [性能优化](https://mongoing.com/performance-optimization)  [原理解读](https://mongoing.com/interpretation-of-principles)  [运维监控](https://mongoing.com/operation-and-maintenance-monitoring)  [最佳实践](https://mongoing.com/best-practices) |
| MongoDB白皮书 | [https://mongoing.com/mongodb-download-white-paper](https://mongoing.com/mongodb-download-white-paper) |
| MongoDB初学者教程-7天入门 | [https://mongoing.com/mongodb-beginner-tutorial](https://mongoing.com/mongodb-beginner-tutorial) |
| 社区活动邮件订阅 | [https://sourl.cn/spszjN](https://sourl.cn/spszjN) |

