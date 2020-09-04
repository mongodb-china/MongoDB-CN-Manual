# 评估当前运营的绩效

**在本页面**

- [使用数据库分析器来计算针对数据库的操作](#操作)
- [使用`db.currentOp()`来评估`mongod`业务](#业务)
- [使用`explain`来评估查询性能](#性能)

以下各节介绍了用于评估操作性能的技术。

## <span id="操作">使用数据库分析器来计算针对数据库的操作</span>

MongoDB提供了一个[数据库分析器](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)，它显示针对数据库的每个操作的性能特征。使用分析器定位任何运行缓慢的查询或写操作。例如，您可以使用此信息来确定要创建什么索引。

从MongoDB 4.2开始，用于读写操作的[profiler条目](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)和诊断日志消息(即**mongod/mongos日志消息**)包括:

- `queryHash`帮助识别具有相同[查询形状的](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)慢速查询 。
- `planCacheKey`为深入了解[查询计划缓存](https://docs.mongodb.com/manual/core/query-plans/)提供慢速查询。

从版本4.2(也可以从4.0.6开始使用)开始，复制集的次要成员现在会记录花费超过慢操作阈值的**oplog条目**。这些缓慢的**oplog**消息被记录在REPL组件下的诊断日志中，并应用文本**op: <oplog条目>取<`num`>ms**。这些较慢的oplog条目仅依赖于较慢的操作阈值。它们不依赖于日志级别(系统或组件级别)、分析级别或较慢的操作采样率。分析器不会捕获很慢的**oplog**条目。

有关更多信息，请参见[Database Profiler](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)。

## <span id="业务">使用`db.currentOp()`到评估`mongod`业务</span>

该[`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#db.currentOp)方法报告[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例上正在运行的当前操作。

## <span id="性能">使用`explain`来评估查询性能</span>

在[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)与[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) 方法返回关于查询执行的信息，如MongoDB的选择以满足查询和执行统计数据的指标。您可以在[queryPlanner](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-queryplanner) 模式，[executionStats](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-executionstats)模式或 [allPlansExecution](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-allplansexecution)模式下运行这些方法，以控制返回的信息量。

https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)

**例子**

要在名为**records**的集合中查询与表达式**{a: 1}**匹配的文档时使用[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)，在mongo shell中使用类似于下面的操作:

```shell
db.records.find( { a: 1 } ).explain("executionStats")
```

从MongoDB 4.2开始，**explain**输出包括:

- [`queryHash`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.queryHash)帮助识别具有相同[查询形状的](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)慢速查询。
- [`planCacheKey`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.planCacheKey)为深入了解[查询计划缓存](https://docs.mongodb.com/manual/core/query-plans/)提供慢速查询。

欲了解更多信息，请参阅[解释结果](https://docs.mongodb.com/manual/reference/explain-results/)， [`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)，[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)，和 [分析查询性能](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)。



译者：杨帅

校对：杨帅