# 哈希分片

哈希分片使用[哈希索引](https://docs.mongodb.com/v4.2/core/index-hashed/#index-hashed-index)来在分片集群中对数据进行划分。哈希索引计算某一个字段的哈希值作为索引值，这个值被用作片键。

![Diagram of the hashed based segmentation.](https://docs.mongodb.com/v4.2/_images/sharding-hash-based.bakedsvg.svg)

哈希分片以减少[定向操作和增加广播操作](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/#sharding-query-isolation)作为代价，分片集群内的数据分布更加均衡。在哈希之后，拥有比较“接近”的片键的文档将不太可能会分布在相同的数据库或者分片上。mongos更有可能执行广播操作来完成一个给定的范围查询。相对的，mongos可以将等值匹配的查询直接定位到单个分片上。

> 注意：
>
> 当使用哈希索引来解析查询时，MongoDB会自动计算哈希值。应用程序**不需要**计算哈希。

>  警告
>
> MongoDB哈希索引在哈希计算之前会将浮点数截断为64位整数。 例如，哈希索引会将为具有`2.3`、`2.2`和`2.9`的值的字段存储为相同的值。 为了避免冲突，请勿对不能可靠地转换为64位整数（然后再返回到浮点）的浮点数使用哈希索引。 MongoDB哈希索引不支持大于2^53的浮点值。
>
> 如果想查看一个键的哈希值是什么，请参考 [`convertShardKeyToHashed()`](https://docs.mongodb.com/v4.2/reference/method/convertShardKeyToHashed/#convertShardKeyToHashed)。

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [[1\]](https://docs.mongodb.com/v4.2/core/hashed-sharding/#id1) | 从4.0版开始，mongo shell提供了`convertShardKeyToHashed（）`方法。 此方法使用与哈希索引相同的哈希函数，可用于查看键的哈希值。 |



## 哈希分片的片键

您选择作为哈希片键的字段应具有良好的[基数](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#shard-key-range)或者该字段包含大量不同的值。 哈希分片非常适合选取具有像`ObjectId`值或时间戳那样[单调](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#shard-key-monotonic)更改的字段作为片键。 一个很好的例子是默认的`_id`字段，假设它仅包含`ObjectID`值（而非用户自定义的`_id`）。

要使用哈希片键对集合进行分片，请参阅 [对集合进行分片](https://docs.mongodb.com/v4.2/tutorial/deploy-shard-cluster/#deploy-hashed-sharded-cluster-shard-collection)。



## 哈希分片 VS 范围分片

给定一个使用单调递增的值`X`作为片键的集合，使用范围分片会导致插入数据的分布类似于下面这样：

![Diagram of poor shard key distribution due to monotonically increasing or decreasing shard key](https://docs.mongodb.com/v4.2/_images/sharded-cluster-monotonic-distribution.bakedsvg.svg)



由于`X`的值始终在增加，因此具有`maxKey`(上限)的数据块将接收大多数传入的写操作。 这将插入操作限制在只能定向到包含此块的单个分片，从而减少或消除了分片集群中分布式写入的优势。

通过在`X`上使用哈希索引，插入的分布将类似于下面这样：

![Diagram of hashed shard key distribution](https://docs.mongodb.com/v4.2/_images/sharded-cluster-hashed-distribution.bakedsvg.svg)

由于现在数据分布更加均匀，因此可以在整个集群中更高效地分布式插入数据。



## 对一个集合进行分片

使用 [`sh.shardCollection()`](https://docs.mongodb.com/v4.2/reference/method/sh.shardCollection/#sh.shardCollection) 方法，指定集合的完整命名空间以及作为[片键](https://docs.mongodb.com/v4.2/reference/glossary/#term-shard-key)的目标[哈希索引](https://docs.mongodb.com/v4.2/core/index-hashed/)。

```
sh.shardCollection( "database.collection", { <field> : "hashed" } )
```

> 重要
>
> - 一旦对某个集合进行分片后，片键的选择是不可变的。 也就是说，您不能再为该集合选择其他的片键。
> - 从MongoDB 4.2开始，除非片键字段是不可变的`_id`字段，否则您可以更新文档的片键值。 有关更新片键的详细信息，请参阅[更改文档的片键值](https://docs.mongodb.com/v4.2/core/sharding-shard-key/#update-shard-key)。在MongoDB 4.2以前的版本，片键是不可变的。



### 对一个已有数据的集合进行分片

如果您使用哈希片键对一个已经包含数据的集合进行分片操作：

- 分片操作将创建初始数据块，以覆盖片键值的整个范围。 创建的数据块数取决于[配置的数据块大小](https://docs.mongodb.com/v4.2/core/sharding-data-partitioning/#sharding-chunk-size)。
- 在初始数据块创建之后，均衡器会在分片上适当地迁移这些初始数据块，并管理后续的数据块分配。



### 对一个空集合进行分片

如果您使用哈希片键对一个空集合进行分片操作：

- 如果没有为空集合或不存在的集合指定区域和区域范围：

  - 分片操作将创建空数据块，以覆盖片键值的整个范围，并执行初始数据块分配。默认情况下，该操作为每个分片创建2个数据块，并在整个集群中迁移。您可以使用`numInitialChunks`选项指定不同数量的初始块。数据块的这种初始创建和分配可以使分片设置更加快速。
- 初始分配之后，均衡器将管理后续的数据块分配。
  
- 如果已经为空集合或不存在的集合指定区域和区域范围（从MongoDB4.0.3版本起可用）：

  - 分片操作会为定义的区域范围以及所有其他分片创建空数据块，以覆盖片键值的整个范围，并根据区域范围执行初始数据块分配。数据块的这种初始创建和分配可以使分片设置更加快速。
- 初始分配之后，均衡器将管理后续的数据块分配。



另请参考：

要了解如何部署分片集群和实现哈希分片，请参阅[部署分片集群](https://docs.mongodb.com/v4.2/tutorial/deploy-shard-cluster/#sharding-procedure-setup)。



原文链接：https://docs.mongodb.com/v4.2/core/hashed-sharding/

译者：刘翔

校对：牟天垒