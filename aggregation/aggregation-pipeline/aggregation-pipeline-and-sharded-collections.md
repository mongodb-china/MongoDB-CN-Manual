# 聚合管道和分片集合

在本页面

* [行为](aggregation-pipeline-and-sharded-collections.md#behavior)
* [优化](aggregation-pipeline-and-sharded-collections.md#optimization)

聚合管道支持对[分片](aggregation-pipeline-and-sharded-collections.md)集合的操作。本节介绍特定于[聚合管道](./)和分片集合的行为。

## 行为

Mongodb 3.2 版本的改变

如果管道以 shard key 上的精确[$match](aggregation-pipeline-and-sharded-collections.md)开头，则整个管道仅在匹配的分片上运行。以前，管道将被拆分，合并它的工作必须在主分片上完成。

对于必须在多个分片上运行的聚合操作，如果操作不需要在数据库的主分片上运行，则这些操作将会将结果路由到随机分片以合并结果，以避免该数据库的主分片超载。 [$out](aggregation-pipeline-and-sharded-collections.md)阶段和[$lookup](aggregation-pipeline-and-sharded-collections.md)阶段需要在数据库的主分片上运行。

## 优化

在将聚合管道分成两部分时，管道被拆分以确保分片在考虑优化的情况下执行尽可能多的阶段。

要查看管道是如何拆分的，请在[db.collection.aggregate\(\)](aggregation-pipeline-and-sharded-collections.md)方法中包含[explain](aggregation-pipeline-and-sharded-collections.md)选项。

优化可能会在不同版本之间发生变化。

译者：李冠飞

校对：李冠飞

