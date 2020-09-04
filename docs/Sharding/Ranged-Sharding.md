# 范围分片

基于范围的分片会将数据划分为由片键值确定的连续范围。 在此模型中，具有“接近”片键值的文档可能位于相同的[块](https://docs.mongodb.com/v4.2/reference/glossary/#term-chunk)或[分片](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard)中。 这允许在连续范围内读取目标文档的高效查询。 但是，如果片键选择不佳，则读取和写入性能均可能降低。 请参阅[片键的选择](https://docs.mongodb.com/v4.2/core/ranged-sharding/#sharding-ranged-shard-key)。

![Diagram of the shard key value space segmented into smaller ranges or chunks.](https://docs.mongodb.com/v4.2/_images/sharding-range-based.bakedsvg.svg)

如果未配置任何其他选项（例如`哈希分片`或`区域`所需的其他选项），则基于范围的分片是默认的分片方式。



## 片键的选择

当片键呈现出以下特征时，范围分片更高效：

- [基数](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#shard-key-range) 大
- [频率](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#shard-key-frequency) 低
- 非[单调变更](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#shard-key-monotonic) 

下图说明了使用字段`X`作为片键的分片群集。 如果`X`的值具有大取值范围，低频率以及非单调变化的特征，则插入的分布可能类似于下面这样：

![Diagram of good shard key distribution](https://docs.mongodb.com/v4.2/_images/sharded-cluster-ranged-distribution-good.bakedsvg.svg)



## 对一个集合进行分片

使用`sh.shardCollection()`方法，指定集合的完整命名空间以及作为[片键](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key)的目标[索引](https://docs.mongodb.com/v4.2/reference/glossary/#term-index)或复合索引。

```
sh.shardCollection( "database.collection", { <shard key> } )
```

> 重要
>
> - 一旦对某个集合进行分片后，片键的选择是不可变的。 也就是说，您不能再为该集合选择其他片键。
> - 从MongoDB 4.2开始，除非片键字段是不可变的`_id`字段，否则您可以更新文档的片键值。 有关更新片键的详细信息，请参阅[更改文档的片键值](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#update-shard-key)。在MongoDB 4.2以前的版本，片键是不可变的.



### 对一个已有数据的集合进行分片

如果您对一个已经包含数据的集合进行分片操作：

- 分片操作将创建初始数据块，以覆盖片键值的整个范围。 创建的数据块数取决于[配置的数据块大小](https://docs.mongodb.com/v4.2/core/sharding-data-partitioning/#sharding-chunk-size)。
- 在初始数据块创建之后，均衡器会在分片上适当地迁移这些初始数据块，并管理后续的数据块分配。



### 对一个空集合进行分片

如果您对一个空集合进行分片操作：

- 如果没有为空集合或不存在的集合指定区域和区域范围：

  - 分片操作将创建一个空块，以覆盖片键值的整个范围。
- 在创建初始块之后，平衡器将在块之间适当地迁移初始块，并管理后续的块分配。
  
- 如果已经为空集合或不存在的集合指定区域和区域范围（从MongoDB4.0.3版本起可用）：

  - 分片操作会为定义的区域范围以及覆盖该片键值的整个范围的任何其他块创建空数据块，并根据区域范围执行初始数据块分配。数据块的这种初始创建和分配可以使分片设置更加快速。
- 在初始分配之后，均衡器将管理后续的数据块分配。



另请参阅

要了解如何部署分片集群和实现范围分片，请参阅[部署分片集群](https://docs.mongodb.com/v4.2/tutorial/deploy-shard-cluster/#sharding-procedure-setup)。



原文链接：https://docs.mongodb.com/v4.2/core/ranged-sharding/

译者：刘翔

校对：牟天垒