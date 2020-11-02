# MongoDB分片

On this page **在本页面**

* [Sharded Cluster 分片集群](https://docs.mongodb.com/manual/sharding/#sharded-cluster)
* [Shard Keys 分片键](https://docs.mongodb.com/manual/sharding/#shard-keys)
* [Chunks 块](https://docs.mongodb.com/manual/sharding/#chunks)
* [Balancer and Even Chunk Distribution 均衡器和均匀分配](https://docs.mongodb.com/manual/sharding/#balancer-and-even-chunk-distribution)
* [Advantages of Sharding 分片的优点](https://docs.mongodb.com/manual/sharding/#advantages-of-sharding)
* [Considerations Before Sharding 分片前的注意事项](https://docs.mongodb.com/manual/sharding/#considerations-before-sharding)
* [Sharded and Non-Sharded Collections 分片和非分片集合](https://docs.mongodb.com/manual/sharding/#sharded-and-non-sharded-collections)
* [Connecting to a Sharded Cluster 连接到分片群集](https://docs.mongodb.com/manual/sharding/#connecting-to-a-sharded-cluster)
* [Sharding Strategy 分片策略](https://docs.mongodb.com/manual/sharding/#sharding-strategy)
* [Zones in Sharded Clusters 分片集群中的区域](https://docs.mongodb.com/manual/sharding/#zones-in-sharded-clusters)
* [Collations in Sharding 分片中的排序规则](https://docs.mongodb.com/manual/sharding/#collations-in-sharding)
* [Change Streams 变更流](https://docs.mongodb.com/manual/sharding/#change-streams)
* [Transactions 事务](https://docs.mongodb.com/manual/sharding/#transactions)

[Sharding](https://docs.mongodb.com/manual/reference/glossary/#term-sharding) is a method for distributing data across multiple machines. MongoDB uses sharding to support deployments with very large data sets and high throughput operations.

[分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)是一种将数据分配到多个机器上的方法。MongoDB通过分片技术来支持具有海量数据集和高吞吐量操作的部署方案。

Database systems with large data sets or high throughput applications can challenge the capacity of a single server. For example, high query rates can exhaust the CPU capacity of the server. Working set sizes larger than the system’s RAM stress the I/O capacity of disk drives.

数据库系统的数据集或应用的吞吐量比较大的情况下，会给单台服务器的处理能力带来极大的挑战。例如，高查询率会耗尽服务器的CPU资源。工作的数据集大于系统的内存压力、磁盘驱动器的I/O容量。

There are two methods for addressing system growth: vertical and horizontal scaling.

解决系统增长的方法有两种：垂直扩展和水平扩展。

_Vertical Scaling_ involves increasing the capacity of a single server, such as using a more powerful CPU, adding more RAM, or increasing the amount of storage space. Limitations in available technology may restrict a single machine from being sufficiently powerful for a given workload. Additionally, Cloud-based providers have hard ceilings based on available hardware configurations. As a result, there is a practical maximum for vertical scaling.

*垂直扩展* 通过增加单个服务器的能力来实现，例如使用更强大的CPU，增加更多的内存或存储空间量。由于现有技术的局限性，不能无限制地增加单个机器的配置。此外，云计算供应商提供可用的硬件配置具有严格的上限。其结果是，垂直扩展有一个实际的最大值。

_Horizontal Scaling_ involves dividing the system dataset and load over multiple servers, adding additional servers to increase capacity as required. While the overall speed or capacity of a single machine may not be high, each machine handles a subset of the overall workload, potentially providing better efficiency than a single high-speed high-capacity server. Expanding the capacity of the deployment only requires adding additional servers as needed, which can be a lower overall cost than high-end hardware for a single machine. The trade off is increased complexity in infrastructure and maintenance for the deployment.

*水平扩展*是通过将系统数据集划分至多台机器，并根据需要添加服务器来提升容量。虽然单个机器的总体速度或容量可能不高，但每台机器只需处理整个数据集的某个子集，所以可能会提供比单个高速大容量服务器更高的效率，而且机器的数量只需要根据数据集大小来进行扩展，与单个机器的高端硬件相比，这个方案可以降低总体成本。不过，这种方式会提高基础设施部署维护的复杂性。

MongoDB supports _horizontal scaling_ through [sharding](https://docs.mongodb.com/manual/reference/glossary/#term-sharding).

MongoDB通过[分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)来实现_水平扩展_。

## Sharded Cluster 分片集群

A MongoDB [sharded cluster](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster) consists of the following components:

* [shard](https://docs.mongodb.com/manual/core/sharded-cluster-shards/): Each shard contains a subset of the sharded data. Each shard can be deployed as a [replica set](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set).
* [mongos](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/): The `mongos` acts as a query router, providing an interface between client applications and the sharded cluster. Starting in MongoDB 4.4, `mongos` can support [hedged reads](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads) to minimize latencies.
* [config servers](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/): Config servers store metadata and configuration settings for the cluster.

The following graphic describes the interaction of components within a sharded cluster:

MongoDB分片[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)由以下组件组成：

* [分片](https://docs.mongodb.com/manual/core/sharded-cluster-shards/)：每个分片包含分片数据的子集。每个分片都可以部署为[副本集](https://docs.mongodb.com/manual/reference/glossary/#term-replica-set)。
* [mongos](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/)：`mongos`充当查询路由器，在客户端应用程序和分片群集之间提供接口。从MongoDB 4.4开始，`mongos`可以支持 [对冲读取（hedged reads）](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads)以最大程度地减少延迟。
* [config服务器](https://docs.mongodb.com/manual/core/sharded-cluster-config-servers/)：配置服务器存储集群的元数据和配置设置。

下图描述了分片群集中组件的交互：

![Diagram of a sample sharded cluster for production purposes. Contains exactly 3 config servers, 2 or more \`\`mongos\`\` query routers, and at least 2 shards. The shards are replica sets.](https://docs.mongodb.com/manual/_images/sharded-cluster-production-architecture.bakedsvg.svg)

MongoDB shards data at the [collection](https://docs.mongodb.com/manual/reference/glossary/#term-collection) level, distributing the collection data across the shards in the cluster.

MongoDB在[集合](https://docs.mongodb.com/manual/reference/glossary/#term-collection)级别分片数据，将收集数据分布在集群中的各个分片上。

## Shard Keys 分片键

MongoDB uses the [shard key](https://docs.mongodb.com/manual/core/sharding-shard-key/) to distribute the collection’s documents across shards. The shard key consists of a field or multiple fields in the documents.

* Starting in version 4.4, documents in sharded collections can be missing the shard key fields. Missing shard key fields are treated as having null values when distributing the documents across shards but not when routing queries. For more information, see [Missing Shard Key](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-missing).
* In version 4.2 and earlier, shard key fields must exist in every document for a sharded collection.

MongoDB使用分片[键](https://docs.mongodb.com/manual/core/sharding-shard-key/)在各个分片之间分发集合的文档。分片键由文档中的一个或多个字段组成。

* 从版本4.4开始，分片集合中的文档可能缺少分片键字段。在跨分片分布文档时，缺少分片键字段将被视为具有空值，但在路由查询时则不会。有关更多信息，请参见 [分片键缺失](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-missing)。
* 在4.2及更早版本中，分片键字段必须在每个文档中存在一个分片集合。

You select the shard key when [sharding a collection](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-creation).

* Starting in MongoDB 4.4, you can refine a collection’s shard key by adding a suffix field or fields to the existing key. See [`refineCollectionShardKey`](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#dbcmd.refineCollectionShardKey) for details.
* In MongoDB 4.2 and earlier, the choice of shard key cannot be changed after sharding.

在[分片集合](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-creation)时选择[分片](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-creation)键。

* 从MongoDB 4.4开始，您可以通过向现有键中添加一个或多个后缀字段来优化集合的分片键。有关详细信息，请参见[`refineCollectionShardKey`](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#dbcmd.refineCollectionShardKey)。
* 在MongoDB 4.2和更低版本中，无法在分片后更改分片键的选择。

A document’s shard key value determines its distribution across the shards.

* Starting in MongoDB 4.2, you can update a document’s shard key value unless your shard key field is the immutable `_id` field. See [Change a Document’s Shard Key Value](https://docs.mongodb.com/manual/core/sharding-shard-key/#update-shard-key) for more information.
* In MongoDB 4.0 and earlier, a document’s shard key field value is immutable.

文档的分片键值决定了其在各个分片中的分布。

* 从MongoDB 4.2开始，您可以更新文档的分片键值，除非您的分片键字段为不可变`_id`字段。有关更多信息，请参见 [更改文档的分片键值](https://docs.mongodb.com/manual/core/sharding-shard-key/#update-shard-key)。
* 在MongoDB 4.0和更早版本中，文档的分片键字段值是不可变的。

### Shard Key Index 分片键索引

To shard a populated collection, the collection must have an [index](https://docs.mongodb.com/manual/reference/glossary/#term-index) that starts with the shard key. When sharding an empty collection, MongoDB creates the supporting index if the collection does not already have an appropriate index for the specified shard key. See [Shard Key Indexes](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-indexes).

要对已填充的集合进行分片，该集合必须具有以分片键开头的[索引](https://docs.mongodb.com/manual/reference/glossary/#term-index)。分片一个空集合时，如果该集合还没有针对指定分片键的适当索引，则MongoDB会创建支持索引。请参阅分片[键索引](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-indexes)。

### Shard Key Strategy 分片键策略

The choice of shard key affects the performance, efficiency, and scalability of a sharded cluster. A cluster with the best possible hardware and infrastructure can be bottlenecked by the choice of shard key. The choice of shard key and its backing index can also affect the [sharding strategy](https://docs.mongodb.com/manual/sharding/#sharding-strategy) that your cluster can use.

See [Choosing a Shard Key](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection) documentation for more information.

分片键的选择会影响分片群集的性能，效率和可伸缩性。选择分片键可以使具有最佳硬件和基础结构的群集成为瓶颈。[分片](https://docs.mongodb.com/manual/sharding/#sharding-strategy)键及其后备索引的选择也会影响群集可以使用的[分片策略](https://docs.mongodb.com/manual/sharding/#sharding-strategy)。

有关更多信息，请参见[选择分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection)文档。

## Chunks 块

MongoDB partitions sharded data into [chunks](https://docs.mongodb.com/manual/reference/glossary/#term-chunk). Each chunk has an inclusive lower and exclusive upper range based on the [shard key](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key).

MongoDB将分片数据拆分成[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。每个分块都有一个基于分片[键的](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)上下限范围 。

## Balancer and Even Chunk Distribution 均衡器和均匀块分配

In an attempt to achieve an even distribution of chunks across all shards in the cluster, a [balancer](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) runs in the background to migrate [chunks](https://docs.mongodb.com/manual/reference/glossary/#term-chunk) across the [shards](https://docs.mongodb.com/manual/reference/glossary/#term-shard) .

See [Data Partitioning with Chunks](https://docs.mongodb.com/manual/core/sharding-data-partitioning/) for more information.

为了在整个集群中的所有分片上实现块的均匀分布，[平衡器](https://docs.mongodb.com/manual/core/sharding-balancer-administration/)在后台运行，以在各分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)上迁移[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。

有关更多信息，请参见[使用块](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)进行[数据分区](https://docs.mongodb.com/manual/core/sharding-data-partitioning/)。

## Advantages of Sharding 分片的优势

### Reads / Writes 读/写

MongoDB distributes the read and write workload across the [shards](https://docs.mongodb.com/manual/reference/glossary/#term-shard) in the [sharded cluster](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster), allowing each shard to process a subset of cluster operations. Both read and write workloads can be scaled horizontally across the cluster by adding more shards.

MongoDB将读写工作负载分布在分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)[集群](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)中的各个分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)上，从而允许每个分片处理集群操作的子集。通过添加更多分片，可以在集群中水平扩展读写工作负载。

For queries that include the shard key or the prefix of a [compound](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index) shard key, [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) can target the query at a specific shard or set of shards. These [targeted operations](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted) are generally more efficient than [broadcasting](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast) to every shard in the cluster.

对于包含分片键或[复合分片](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)键的前缀的查询，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以将查询定位到特定的分片或一组分片。这些[目标操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted)通常比[广播](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)到群集中的每个分片更有效 。

Starting in MongoDB 4.4, [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) can support [hedged reads](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads) to minimize latencies.

从MongoDB 4.4开始，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以支持[对冲读取（hedged reads）](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#mongos-hedged-reads)以最大程度地减少延迟。

### Storage Capacity 存储容量

[Sharding](https://docs.mongodb.com/manual/reference/glossary/#term-sharding) distributes data across the [shards](https://docs.mongodb.com/manual/reference/glossary/#term-shard) in the cluster, allowing each shard to contain a subset of the total cluster data. As the data set grows, additional shards increase the storage capacity of the cluster.

[分片](https://docs.mongodb.com/manual/reference/glossary/#term-sharding)横跨分发数据[碎片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)在集群中，允许每个碎片以包含总簇数据的子集。随着数据集的增长，其他分片将增加群集的存储容量。

### High Availability 高可用性

The deployment of config servers and shards as replica sets provide increased availability.

将配置服务器和分片作为副本集进行部署可提高可用性。

Even if one or more shard replica sets become completely unavailable, the sharded cluster can continue to perform partial reads and writes. That is, while data on the unavailable shard\(s\) cannot be accessed, reads or writes directed at the available shards can still succeed.

即使一个或多个分片副本集变得完全不可用，分片群集也可以继续执行部分读取和写入。也就是说，虽然无法访问不可用分片上的数据，但是针对可用分片的读取或写入仍然可以成功。

## Considerations Before Sharding 分片前的注意事项

Sharded cluster infrastructure requirements and complexity require careful planning, execution, and maintenance.

分片群集基础结构的要求和复杂性要求仔细计划，执行和维护。

Once a collection has been sharded, MongoDB provides no method to unshard a sharded collection.

分片后，MongoDB不会提供任何方法来对分片集群进行分片。

Careful consideration in choosing the shard key is necessary for ensuring cluster performance and efficiency. See [Choosing a Shard Key](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection).

为了确保群集的性能和效率，在选择分片键时需要仔细考虑。请参阅 [选择分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-shard-key-selection)。

Sharding has certain [operational requirements and restrictions](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/#sharding-operational-restrictions). See [Operational Restrictions in Sharded Clusters](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/) for more information.

分片有一定的[操作要求和限制](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/#sharding-operational-restrictions)。有关更多信息，请参见 [分片群集中的操作限制](https://docs.mongodb.com/manual/core/sharded-cluster-requirements/)。

If queries do _not_ include the shard key or the prefix of a [compound](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index) shard key, [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) performs a [broadcast operation](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast), querying _all_ shards in the sharded cluster. These scatter/gather queries can be long running operations.

如果查询_不_包含分片键或[复合分片](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)键的前缀 ，请[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)执行[广播操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)，查询分 片群集中的_所有分_片。这些分散/聚集查询可能是长时间运行的操作。

NOTE

If you have an active support contract with MongoDB, consider contacting your account representative for assistance with sharded cluster planning and deployment.

> 注意
>
> 如果您与MongoDB签订了有效的支持合同，请考虑与您的客户代表联系，以获取分片群集计划和部署方面的帮助。

## Sharded and Non-Sharded Collections 分片和非分片集合

A database can have a mixture of sharded and unsharded collections. Sharded collections are [partitioned](https://docs.mongodb.com/manual/reference/glossary/#term-data-partition) and distributed across the [shards](https://docs.mongodb.com/manual/reference/glossary/#term-shard) in the cluster. Unsharded collections are stored on a [primary shard](https://docs.mongodb.com/manual/reference/glossary/#term-primary-shard). Each database has its own primary shard.

数据库可以包含分片和未分片集合的混合。分片集合在群集中的分[片](https://docs.mongodb.com/manual/reference/glossary/#term-shard)上[分区](https://docs.mongodb.com/manual/reference/glossary/#term-data-partition)和分布 。未分片的集合存储在[主分片上](https://docs.mongodb.com/manual/reference/glossary/#term-primary-shard)。每个数据库都有其自己的主分片。

![Diagram of a primary shard. A primary shard contains non-sharded collections as well as chunks of documents from sharded collections. Shard A is the primary shard.](https://docs.mongodb.com/manual/_images/sharded-cluster-primary-shard.bakedsvg.svg)

## Connecting to a Sharded Cluster 连接到分片群集

You must connect to a [mongos](https://docs.mongodb.com/manual/reference/glossary/#term-mongos) router to interact with any collection in the [sharded cluster](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster). This includes sharded _and_ unsharded collections. Clients should _never_ connect to a single shard in order to perform read or write operations.

您必须连接到[mongos](https://docs.mongodb.com/manual/reference/glossary/#term-mongos)路由器才能与分片[群集中的](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)任何集合进行交互。这包括分片_和_未分片的集合。客户端_永远不要_连接到单个分片以执行读取或写入操作。

![Diagram of applications/drivers issuing queries to mongos for unsharded collection as well as sharded collection. Config servers not shown.](https://docs.mongodb.com/manual/_images/sharded-cluster-mixed.bakedsvg.svg)

You can connect to a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) the same way you connect to a [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod), such as via the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell or a MongoDB [driver](https://docs.mongodb.com/ecosystem/drivers?jump=docs).

您可以通过与[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)相同的方式连接[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) ，例如通过[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell 或MongoDB [驱动程序](https://docs.mongodb.com/ecosystem/drivers?jump=docs)。

## Sharding Strategy 分片策略

MongoDB supports two sharding strategies for distributing data across [sharded clusters](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster).

MongoDB支持两种分片策略，用于在分片[群集](https://docs.mongodb.com/manual/reference/glossary/#term-sharded-cluster)之间分布数据。

### Hashed Sharding 哈希分片

Hashed Sharding involves computing a hash of the shard key field’s value. Each [chunk](https://docs.mongodb.com/manual/reference/glossary/#term-chunk) is then assigned a range based on the hashed shard key values.

哈希分片涉及计算分片键字段值的哈希值。然后，根据散列的分片键值为每个[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)分配一个范围。

TIP 提示

MongoDB automatically computes the hashes when resolving queries using hashed indexes. Applications do **not** need to compute hashes.

使用哈希索引解析查询时，MongoDB自动计算哈希值。应用程序也**不会**需要计算哈希值。

![Diagram of the hashed based segmentation.](https://docs.mongodb.com/manual/_images/sharding-hash-based.bakedsvg.svg)

While a range of shard keys may be “close”, their hashed values are unlikely to be on the same [chunk](https://docs.mongodb.com/manual/reference/glossary/#term-chunk). Data distribution based on hashed values facilitates more even data distribution, especially in data sets where the shard key changes [monotonically](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-monotonic).

尽管一系列分片键可能是“接近”的，但它们的哈希值不太可能在同[一块上](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。基于哈希值的数据分发有助于更均匀的数据分发，尤其是在分片键[单调](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key-monotonic)更改的数据集中。

However, hashed distribution means that range-based queries on the shard key are less likely to target a single shard, resulting in more cluster wide [broadcast operations](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)

See [Hashed Sharding](https://docs.mongodb.com/manual/core/hashed-sharding/) for more information.

但是，哈希分布意味着对分片键的基于范围的查询不太可能针对单个分片，从而导致更多集群范围的[广播操作](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-broadcast)。

有关更多信息，请参见[哈希分片](https://docs.mongodb.com/manual/core/hashed-sharding/)。

### Ranged Sharding 范围分片

Ranged sharding involves dividing data into ranges based on the shard key values. Each [chunk](https://docs.mongodb.com/manual/reference/glossary/#term-chunk) is then assigned a range based on the shard key values.

范围分片涉及根据分片键值将数据划分为多个范围。然后，根据分片键值为每个[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)分配一个范围。

![Diagram of the shard key value space segmented into smaller ranges or chunks.](https://docs.mongodb.com/manual/_images/sharding-range-based.bakedsvg.svg)

A range of shard keys whose values are “close” are more likely to reside on the same [chunk](https://docs.mongodb.com/manual/reference/glossary/#term-chunk). This allows for [targeted operations](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted) as a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) can route the operations to only the shards that contain the required data.

值“接近”的一系列分片键更有可能驻留在同[一块上](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)。这允许有[针对性的操作，](https://docs.mongodb.com/manual/core/sharded-cluster-query-router/#sharding-mongos-targeted)因为[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)可以将操作仅路由到包含所需数据的分片。

The efficiency of ranged sharding depends on the shard key chosen. Poorly considered shard keys can result in uneven distribution of data, which can negate some benefits of sharding or can cause performance bottlenecks. See [shard key selection for range-based sharding](https://docs.mongodb.com/manual/core/ranged-sharding/#sharding-ranged-shard-key).

See [Ranged Sharding](https://docs.mongodb.com/manual/core/ranged-sharding/) for more information.

范围分片的效率取决于选择的分片键。分片键考虑不周全会导致数据分布不均，这可能会削弱分片的某些优势或导致性能瓶颈。有关[基于范围的分片，](https://docs.mongodb.com/manual/core/ranged-sharding/#sharding-ranged-shard-key)请参见 [分片键选择](https://docs.mongodb.com/manual/core/ranged-sharding/#sharding-ranged-shard-key)。

有关更多信息，请参见[范围分片](https://docs.mongodb.com/manual/core/ranged-sharding/)。

## Zones in Sharded Clusters 分片群集中的区域

Zones can help improve the locality of data for sharded clusters that span multiple data centers.

Zones区域可以帮助提高跨多个数据中心的分片群集的数据局部性。

In sharded clusters, you can create [zones](https://docs.mongodb.com/manual/reference/glossary/#term-zone) of sharded data based on the [shard key](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key). You can associate each zone with one or more shards in the cluster. A shard can associate with any number of zones. In a balanced cluster, MongoDB migrates [chunks](https://docs.mongodb.com/manual/reference/glossary/#term-chunk) covered by a zone only to those shards associated with the zone.

在分片群集中，您可以基于[分片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)创建分片数据[区域](https://docs.mongodb.com/manual/reference/glossary/#term-zone)。您可以将每个区域与集群中的一个或多个分片关联。分片可以与任意数量的区域关联。在平衡群集中，MongoDB仅将区域覆盖的[块](https://docs.mongodb.com/manual/reference/glossary/#term-chunk)迁移到与该区域关联的分片。

Each zone covers one or more ranges of [shard key](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key) values. Each range a zone covers is always inclusive of its lower boundary and exclusive of its upper boundary.

每个区域覆盖一个或多个分片[键值](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)范围。区域覆盖的每个范围始终包括其下边界和上边界。

![Diagram of data distribution based on zones in a sharded cluster](https://docs.mongodb.com/manual/_images/sharded-cluster-zones.bakedsvg.svg)

You must use fields contained in the [shard key](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key) when defining a new range for a zone to cover. If using a [compound](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index) shard key, the range must include the prefix of the shard key. See [shard keys in zones](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding-shard-key) for more information.

在定义要覆盖的区域的新范围时，必须使用分片[键中](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)包含的字段。如果使用[复合分片](https://docs.mongodb.com/manual/reference/glossary/#term-compound-index)键，则范围必须包含分片键的前缀。有关更多信息，请参见[区域中的分片键](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding-shard-key)。

The possible use of zones in the future should be taken into consideration when choosing a shard key.

选择分片键时，应考虑将来可能使用的区域。

> TIP 提示
>
> Starting in MongoDB 4.0.3, setting up zones and zone ranges _before_ you shard an empty or a non-existing collection allows for a faster setup of zoned sharding.
>
> 从MongoDB 4.0.3开始，在 对空集合或不存在的集合进行分片_之前_设置区域和区域范围可以更快地设置区域分片。

See [zones](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding) for more information.

有关更多信息，请参见[区域](https://docs.mongodb.com/manual/core/zone-sharding/#zone-sharding)。

## Collations in Sharding 分片中的排序规则

Use the [`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection) command with the `collation : { locale : "simple" }` option to shard a collection which has a [default collation](https://docs.mongodb.com/manual/reference/collation/). Successful sharding requires that:

* The collection must have an index whose prefix is the shard key
* The index must have the collation `{ locale: "simple" }`

When creating new collections with a collation, ensure these conditions are met prior to sharding the collection.

使用带有`collation : { locale : "simple" }`选项的[`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection)命令可以分片具有[默认排序规则](https://docs.mongodb.com/manual/reference/collation/)的集合 。成功的分片需要：

* 集合必须具有前缀为分片键的索引
* 索引必须具有`{ locale: "simple" }`排序规则 

使用排序规则创建新集合时，在分片集合之前，请确保满足这些条件。

> NOTE
>
> Queries on the sharded collection continue to use the default collation configured for the collection. To use the shard key index’s `simple` collation, specify `{locale : "simple"}` in the query’s [collation document](https://docs.mongodb.com/manual/reference/collation/).
>
> 注意
>
> 分片集合上的查询继续使用为集合配置的默认排序规则。要使用分片索引的`simple`归类，请在查询的[归类文档中](https://docs.mongodb.com/manual/reference/collation/)指定`{locale : "simple"}`。

See [`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection) for more information about sharding and collation.

请参阅[`shardCollection`](https://docs.mongodb.com/manual/reference/command/shardCollection/#dbcmd.shardCollection)以获取有关分片和整理的更多信息。

## Change Streams 变更流

Starting in MongoDB 3.6, [change streams](https://docs.mongodb.com/manual/changeStreams/) are available for replica sets and sharded clusters. Change streams allow applications to access real-time data changes without the complexity and risk of tailing the oplog. Applications can use change streams to subscribe to all data changes on a collection or collections.

从MongoDB 3.6开始，[变更流](https://docs.mongodb.com/manual/changeStreams/)可用于副本集和分片群集。更改流允许应用程序访问实时数据更改，而不会带来复杂性和拖延操作日志的风险。应用程序可以使用变更流来订阅一个或多个集合上的所有数据更改。

## Transactions 事务

Starting in MongoDB 4.2, with the introduction of [distributed transactions](https://docs.mongodb.com/manual/core/transactions/), multi-document transactions are available on sharded clusters.

从MongoDB 4.2开始，随着[分布式事务](https://docs.mongodb.com/manual/core/transactions/)的引入，[分片](https://docs.mongodb.com/manual/core/transactions/)群集上可以使用多文档事务。

Until a transaction commits, the data changes made in the transaction are not visible outside the transaction.

在提交事务之前，在事务外部看不到在事务中进行的数据更改。

However, when a transaction writes to multiple shards, not all outside read operations need to wait for the result of the committed transaction to be visible across the shards. For example, if a transaction is committed and write 1 is visible on shard A but write 2 is not yet visible on shard B, an outside read at read concern \[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local)"\) can read the results of write 1 without seeing write 2.

但是，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在所有分片上可见。例如，如果提交了一个事务，并且在分片A上可以看到写1，但是在分片B上仍然看不到写2，则在读问题上进行的外部读取 \[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local"\)可以读取写1的结果而看不到写2。](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local"%29可以读取写1的结果而看不到写2。)

For details, see:

* [Transactions](https://docs.mongodb.com/manual/core/transactions/)
* [Production Considerations](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
* [Production Considerations \(Sharded Clusters\)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)

← [Replica Set Member States](https://docs.mongodb.com/manual/reference/replica-states/)  
[Sharded Cluster Components](https://docs.mongodb.com/manual/core/sharded-cluster-components/) →

有关详细信息，请参见：

* [事务](https://docs.mongodb.com/manual/core/transactions/)
* [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
* [生产注意事项（分片群集）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)

← [副本集成员状态](https://docs.mongodb.com/manual/reference/replica-states/)  
[分片集群组件](https://docs.mongodb.com/manual/core/sharded-cluster-components/) →

原文链接：[https://docs.mongodb.com/manual/sharding/](https://docs.mongodb.com/manual/sharding/)

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

