# [ ](#)Map-Reduce 和分片集合
[]()

在本页面

*   [分片集合作为输入](#sharded-collection-as-input)

*   [分片集合作为输出](#sharded-collection-as-output)

Map-reduce 支持对分片集合的操作，既可以作为输入也可以作为输出。本节介绍[MapReduce]()特定于分片集合的操作。

[]()

[]()

## <span id="sharded-collection-as-input">分片集合作为输入</span>

当使用分片集合作为 map-reduce 操作的输入时，[mongos]()将自动将 map-reduce job 分派给 parallel 中的每个分片。不需要特殊选项。 [mongos]()将等待所有分片上的作业完成。

[]()

## <span id="sharded-collection-as-output">分片集合作为输出</span>

如果[MapReduce]()的`out`字段具有`sharded` 值，则 MongoDB 使用`_id`字段将输出集合分片为分片键。

要输出到分片集合：

* 如果输出集合不存在，请首先创建分片集合

  从版本4.2开始，MongoDB弃用map-reduce选项以 *创建*新的分片集合，并将该`sharded` 选项用于map-reduce。因此，要输出到分片集合，请首先创建分片集合。

  如果您没有首先创建分片集合，则MongoDB会在`_id`字段上创建和分片集合。但是，建议您首先创建分片集合。

*   从4.2版开始，MongoDB不赞成替换现有的分片集合。

*   从版本4.0开始，如果输出集合已存在但未分片，则map-reduce失败。

*   对于新的或空的分片集合，MongoDB使用map-reduce操作的第一阶段的结果来创建在分片之间分布的初始块。

*   [`mongos`]()并行地将映射减少后处理作业分派给拥有块的每个分片。在后处理期间，每个分片将从其他分片中提取其自身块的结果，运行最终的reduce / finalize，然后本地写入输出集合。
> **注意**
>
> * 在以后的 map-reduce 作业中，MongoDB 根据需要拆分块。
>
> * 在 post-processing 期间会自动阻止输出集合的块平衡，以避免并发问题。



译者：李冠飞

校对：小芒果
