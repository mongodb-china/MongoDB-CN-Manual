## 衡量索引使用
**在本页面**

- [使用$indexStats度量索引使用](#id1)
- [使用 `explain()`返回查询计划](#id2)
- [使用`hint()`控制索引使用](#id3)
- [索引指标](#id4)
### <span id="id1">使用$indexStats度量索引使用</span>
使用[`$indexStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#pipe._S_indexStats) 聚合阶段获取关于集合中每个索引的使用情况的统计信息。例如，以下聚合操作返回关于`orders`集合中索引使用情况的统计信息:

```powershell
db.orders.aggregate( [ { $indexStats: { } } ] )
```
也可参考：

[`$indexStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#pipe._S_indexStats)

### <span id="id2">使用 `explain()`返回查询计划</span>
在[executionStats](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-executionstats) 模式中使用[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) 或[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法返回关于查询过程的统计信息，包括使用的索引、扫描的文档数量以及查询处理所用的时间(以毫秒为单位)。

在[allPlansExecution](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-allplansexecution) 模式下使用 [`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) 或[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法查看计划选择期间收集的部分执行统计信息。

也可参考：

[planCacheKey](https://docs.mongodb.com/manual/core/query-plans/#plan-cache-key)

### <span id="id3">使用`hint()`控制索引使用</span>
要强制MongoDB为[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)操作使用特定的索引，请使用hint()方法指定该索引。将[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法附加到[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法。考虑下面的例子:

代码示例如下：

```powershell
db.people.find(
   { name: "John Doe", zipcode: { $gt: "63000" } }
).hint( { zipcode: 1 } )
```
查看使用特定索引的执行统计信息，在[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)语句追加的[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法后跟随[`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法，代码示例如下：

```powershell
db.people.find(
   { name: "John Doe", zipcode: { $gt: "63000" } }
).hint( { zipcode: 1 } ).explain("executionStats")
```
或者在[`db.collection.explain().find()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)方法后追加[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法。
```powershell
db.people.explain("executionStats").find(
   { name: "John Doe", zipcode: { $gt: "63000" } }
).hint( { zipcode: 1 } )
```
在[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法中声明`$natural`参数，避免MongoDB在查询过程中使用任何索引。

```powershell
db.people.find(
   { name: "John Doe", zipcode: { $gt: "63000" } }
).hint( { $natural: 1 } )
```
### <span id="id4">索引指标</span>
除了[`$indexStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#pipe._S_indexStats)聚合阶段，MongoDB提供了各种索引统计数据，您可能想要考虑分析索引使用您的数据库:

|  |                                                              |
| --- | --- |
| 在[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus)方法的输出结果中： | [`metrics.queryExecutor.scanned`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.metrics.queryExecutor.scanned)和[`metrics.operation.scanAndOrder`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.metrics.operation.scanAndOrder) |
| 在[`collStats`](https://docs.mongodb.com/manual/reference/command/collStats/#dbcmd.collStats)输出结果中 | [`totalIndexSize`](https://docs.mongodb.com/manual/reference/command/collStats/#collStats.totalIndexSize)和[`indexSizes`](https://docs.mongodb.com/manual/reference/command/collStats/#collStats.indexSizes) |
| 在[`dbStats`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbcmd.dbStats)输出结果中 | [`dbStats.indexes`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbStats.indexes)和[`dbStats.indexSize`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbStats.indexSize) |

译者：程哲欣
