# 查询计划

**在本页面**

*   [计划缓存条目状态](#计划)
*   [`queryHash`](#queryHash)
*   [`planCacheKey`](#planCacheKey)
*   [可用性](#可用性)

对于查询，MongoDB查询优化器在给定可用索引的情况下选择并缓存效率最高的查询计划。最有效的查询计划的评估是基于查询执行计划在查询计划评估候选计划时执行的“工作单元”(**works**)的数量。

关联的计划缓存条目用于具有相同查询形状的后续查询。


## <span id="计划">计划缓存条目状态</span>

从MongoDB 4.2开始，缓存条目与状态关联：

| State                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [失踪](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-missing) | 缓存中不存在此形状的条目。<br />对于查询，如果形状的缓存条目状态为 [Missing](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-missing)：<br />1.对候选计划进行评估并选出一个获胜的计划。<br />2.所选计划将以非活动状态及其工作值添加到缓存中。。 |
| [不活跃](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive) | 缓存中的条目是此形状的占位符条目。也就是说，计划者已经看到了形状并计算了其成本（`works`价值）并存储为占位符条目，但查询形状**不**用于生成查询计划。<br />对于查询，如果形状的缓存条目状态非[活动](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)：<br />1.对候选计划进行评估并选出一个获胜的计划。<br />2.所选计划的工作值与非活动条目的工作值进行比较。如果所选计划的works值为：小于或等于[非活动](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)条目的，<br />所选计划将替换占位符“不 [活动”](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)条目，并具有“ [活动”](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-active)状态。<br />如果在替换发生之前，“ [非活动”](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)条目变为“ [活动”](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-active)(例如，由于其他查询操作)，则仅当新活动条目的`works`值大于所选计划时，才会替换该新活动条目。<br />大于非[活动](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)条目的数量，<br />不[活动](https://docs.mongodb.com/master/core/query-plans/#cache-entry-inactive)的条目仍然存在，但其工作值增加。 |
| [活性](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-active) | 缓存中的条目用于中奖计划。计划者可以使用该条目来生成查询计划。<br />对于查询，如果形状的缓存条目状态为 [Active](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-active)：<br />活动条目用于生成查询计划。<br />计划者还评估条目的性能，如果条目的 `works`值不再符合选择标准，它将转换为非[活动](https://docs.mongodb.com/manual/core/query-plans/#cache-entry-inactive)状态。 |

有关触发对计划缓存进行更改的其他方案，请参阅[计划缓存刷新](https://docs.mongodb.com/manual/core/query-plans/#query-plans-plan-cache-flushes)。

#### 查询计划和高速缓存信息

要查看给定查询的查询计划信息，可以使用 [`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)或[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)。

从MongoDB 4.2开始，您可以使用[`$planCacheStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/planCacheStats/#pipe._S_planCacheStats) 聚合阶段来查看集合的计划缓存信息。

#### 计划缓存刷新

如果[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 重新启动或关闭，查询计划缓存将不会保留。此外：

- 索引或收集删除之类的目录操作会清除计划缓存。
- 最近最少使用（LRU）高速缓存替换机制将清除最近最少访问的高速缓存条目，而不管其状态如何。

用户还可以：

- 使用[`PlanCache.clear()`](https://docs.mongodb.com/manual/reference/method/PlanCache.clear/#PlanCache.clear)方法手动清除整个计划缓存 。

- 使用[`PlanCache.clearPlansByQuery()`](https://docs.mongodb.com/manual/reference/method/PlanCache.clearPlansByQuery/#PlanCache.clearPlansByQuery)方法手动清除特定的计划缓存条目 。

  也可以看看

  [queryHash和planCacheKey](https://docs.mongodb.com/manual/core/query-plans/#query-hash-plan-cache-key)

##### queryHash和planCacheKey

## <span id="queryHash">queryHash</span>

为了帮助识别具有相同[查询形状](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)的慢速查询，从MongoDB 4.2开始，每个查询形状都与一个[queryHash](https://docs.mongodb.com/manual/release-notes/4.2/#query-hash)相关联。**queryHash**是一个十六进制字符串，表示查询形状的散列，并且只依赖于查询形状。

> **[success] 注意**
>
> 与任何hash函数一样，两个不同的查询形状可能会导致相同的hash值。但是，不同查询形状之间不会发生哈希冲突。

## <span id="planCacheKey">planCacheKey</span>

为了更深入地了解[缓存查询计划](https://docs.mongodb.com/master/core/query-plans/#)，MongoDB 4.2引入了 [planCacheKey](https://docs.mongodb.com/master/release-notes/4.2/#plan-cache-key).

`planCacheKey` 是与查询关联的计划缓存条目的键的hash值。

> **[success] 注意**
>
> 与**queryHash**不同，**planCacheKey**是查询形状和当前可用的形状索引的函数。也就是说，如果添加/删除了支持查询形状的索引，**planCacheKey**值可能会改变，而**queryHash**值不会改变。

例如，考虑一个具有以下索引的**foo**集合:

```shell
db.foo.createIndex( { x: 1 } )
db.foo.createIndex( { x: 1, y: 1 } )
db.foo.createIndex( { x: 1, z: 1 }, { partialFilterExpression: { x: { $gt: 10 } } } )
```

集合上的以下查询具有相同的形状:

```shell
db.foo.explain().find( { x: { $gt: 5 } } )  // Query Operation 1
db.foo.explain().find( { x: { $gt: 20 } } ) // Query Operation 2
```

对于这些查询，带有[部分过滤表达式](https://docs.mongodb.com/master/core/index-partial/#partial-index-query-coverage) 的索引可以支持查询操作2，但不支持查询操作1。由于支持查询操作1的索引与查询操作2不同，这两个查询具有不同的**planCacheKey**。

如果删除了其中一个索引，或者添加了一个新的索引**{x: 1, a: 1}**，那么用于这两个查询操作的**planCacheKey**将会改变。

## <span id="可用性">可用性</span>

**queryHash**和**planCacheKey**是可用的在:

- [explain() output](https://docs.mongodb.com/manual/reference/explain-results/)字段： [`queryPlanner.queryHash`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.queryHash)和 [`queryPlanner.planCacheKey`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.planCacheKey)
- 记录慢查询时，[探查器日志消息](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) 和[诊断日志消息（即mongod / mongos日志消息）](https://docs.mongodb.com/manual/reference/log-messages/)。
- [`$planCacheStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/planCacheStats/#pipe._S_planCacheStats)聚合阶段（*MongoDB 4.2中的新增功能*）
-  **PlanCache.listQueryShapes()**方法/**planCacheListQueryShapes**命令
- **PlanCache.getPlansByQuery()**方法/**planCacheListPlans**命令

##### 索引筛选器

索引筛选器确定优化器为查询形状评估哪些索引。查询形状由查询、排序和投影规范的组合组成。如果存在针对给定查询形状的索引筛选器，则优化器仅考虑筛选器中指定的那些索引。

当存在查询形状的索引过滤器时，MongoDB会忽略[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)。要查看MongoDB是否为查询形状应用了索引筛选器，请检查[`db.collection.explain()`](https://docs.mongodb.com/master/reference/method/db.collection.explain/#db.collection.explain)或[`cursor.explain()`](https://docs.mongodb.com/master/reference/method/cursor.explain/#cursor.explain) 方法的[`indexFilterSet`](https://docs.mongodb.com/master/reference/explain-results/#explain.queryPlanner.indexFilterSet)字段。

索引过滤器仅影响优化器评估的索引；对于给定的查询形状，优化器仍然可以选择将集合扫描作为获胜计划。

索引过滤器在服务器进程的持续时间内存在，并且在关闭后不会持续存在。MongoDB还提供了手动删除过滤器的命令。

因为索引过滤器会覆盖优化器和[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法的预期行为，所以请谨慎使用索引过滤器。

见[`planCacheListFilters`](https://docs.mongodb.com/manual/reference/command/planCacheListFilters/#dbcmd.planCacheListFilters)， [`planCacheClearFilters`](https://docs.mongodb.com/manual/reference/command/planCacheClearFilters/#dbcmd.planCacheClearFilters)和[`planCacheSetFilter`](https://docs.mongodb.com/manual/reference/command/planCacheSetFilter/#dbcmd.planCacheSetFilter)。

​	也可以看看：

​	[索引策略](https://docs.mongodb.com/manual/applications/indexes/)



译者：杨帅

校对：杨帅