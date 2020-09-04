
# 聚合
在本页面

*   [聚合管道](#aggregation-pipeline)

*   [Map-Reduce](#map-reduce)

*   [单用途聚合操作](#single-purpose-aggregation-operations)

*   [附加功能和行为](#additional-features-and-behaviors)

聚合操作处理数据记录和 return 计算结果。聚合操作将来自多个文档的值组合在一起，并且可以对分组数据执行各种操作以返回单个结果。 MongoDB 提供了三种执行聚合的方法：[聚合管道](#聚合管道)，[map-reduce function](#map-reduce)和[单一目的聚合方法](#单用途聚合操作)。

[]()

[]()

## <span id="aggregation-pipeline">聚合管道</span>

MongoDB 的[Aggregation framework](Aggregation/Aggregation-Pipeline.md)是以数据处理管道的概念为蓝本的。文档进入多阶段管道，将文档转换为聚合结果。例如：


<iframe 
    height=450 
    width=800 
    src="../img/docs/Aggregation/agg-pipeline.mp4" 
    frameborder=0 
    allowfullscreen>
</iframe>
在这个例子中：

```powershell
db.orders.aggregate([
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

**第一阶段**：[`$match`]()阶段按`status`字段过滤文档，并将`status`等于`"A"`的文档传递到下一阶段。

**第二阶段**：[`$group`]()阶段按`cust_id`字段将文档分组，以计算每个唯一值`cust_id`的金额总和。

最基本的管道阶段提供*过滤器*，其操作类似于查询和修改输出文档格式的*文档转换*。

其他管道操作提供了用于按特定字段对文档进行分组和排序的工具，以及用于汇总包括文档数组在内的数组内容的工具。另外，管道阶段可以将[运算符]()用于诸如计算平均值或连接字符串之类的任务。

管道使用MongoDB中的原生操作提供有效的数据聚合，并且是MongoDB中数据聚合的首选方法。

聚合管道可以在[分片集合  sharded collection]()上运行。

聚合管道可以使用索引来改善其某些阶段的性能。此外，聚合管道具有内部优化阶段。有关详细信息，请参阅[管道操作和索引](Aggregation/Aggregation-Pipeline.md)和[聚合管道优化](Aggregation/Aggregation-Pipeline/Aggregation-Pipeline-Optimization.md)。

[]()

[]()

## <span id="map-reduce">Map-Reduce</span>

MongoDB 还提供[map-reduce](Aggregation/Map-Reduce.md)操作来执行聚合。通常，map-reduce 操作有两个阶段：一个 map 阶段，它处理每个文档并为每个输入文档发出一个或多个对象，以及将map操作的输出组合在一起的*reduce*阶段。可选地，map-reduce 可以具有最终化阶段以对结果进行最终修改。与其他聚合操作一样，map-reduce 可以指定查询条件以选择输入文档以及对结果排序和限制。

Map-reduce 使用自定义 JavaScript 函数来执行 map 和 reduce操作，以及可选的 finalize 操作。与聚合管道相比，自定义JavaScript提供了很大的灵活性，但通常情况下，map-reduce比聚合管道效率低，而且更复杂。

Map-reduce 可以在[分片集合 sharded collection]()上运行。 Map-reduce 操作也可以输出到分片集合。有关详细信息，请参阅[聚合管道和分片集合](Aggregation/Aggregation-Pipeline/Aggregation-Pipeline-and-Sharded-Collections.md)和[Map-Reduce 和 Sharded Collections](Aggregation/Map-Reduce/Map-Reduce-and-Sharded-Collections.md)。

> **[success] 注意**
>
> 从 MongoDB 2.4 开始，在 map-reduce 操作中无法访问某些mongoshell 函数和属性。 MongoDB 2.4 还支持多个 JavaScript 操作以在同一时间运行。在 MongoDB 2.4 之前，JavaScript code 在单个线程中执行，引发了 map-reduce 的并发问题。

![带注释的 map-reduce 操作图](../img/docs/Aggregation/map-reduce.bakedsvg.svg)



## <span id="single-purpose-aggregation-operations">单用途聚合操作</span>

MongoDB 还提供  [db.collection.estimatedDocumentCount()](), [db.collection.count()](Reference/mongo-Shell-Methods/Collection-Methods/db-collection-count.md)和[db.collection.distinct()](Reference/mongo-Shell-Methods/Collection-Methods/db-collection-distinct.md)。

所有这些操作都聚合来自单个集合的文档。虽然这些操作提供了对常见聚合过程的简单访问，但它们缺乏聚合管道和 map-reduce 的灵活性和功能。

![带注释的不同操作的图表](../img/docs/Aggregation/distinct.bakedsvg.svg)



## <span id="additional-features-and-behaviors">附加功能和行为</span>

有关聚合管道 map-reduce 和特殊组功能的特性比较，请参阅[聚合命令比较](Aggregation/Aggregation-Reference/Aggregation-Commands-Commparison.md)。



译者：李冠飞

校对：李冠飞