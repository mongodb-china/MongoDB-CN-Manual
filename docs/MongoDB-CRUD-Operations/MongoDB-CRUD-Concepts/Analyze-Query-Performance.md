# 分析查询性能

**在本页面**

* [评估查询的性能](#评估)

该 [`cursor.explain("executionStats")`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) 和[`db.collection.explain("executionStats")`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)方法提供了有关查询的性能统计信息。这些统计信息可用于衡量查询是否以及如何使用索引。

[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) 提供关于其他操作(如[`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update))执行的信息。详细信息请参见[`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain) 

## <span id="评估">评估查询的性能</span>

考虑一个包含以下文件的收集清单:

```shell
{ "_id" : 1, "item" : "f1", type: "food", quantity: 500 }
{ "_id" : 2, "item" : "f2", type: "food", quantity: 100 }
{ "_id" : 3, "item" : "p1", type: "paper", quantity: 200 }
{ "_id" : 4, "item" : "p2", type: "paper", quantity: 150 }
{ "_id" : 5, "item" : "f3", type: "food", quantity: 300 }
{ "_id" : 6, "item" : "t1", type: "toys", quantity: 500 }
{ "_id" : 7, "item" : "a1", type: "apparel", quantity: 250 }
{ "_id" : 8, "item" : "a2", type: "apparel", quantity: 400 }
{ "_id" : 9, "item" : "t2", type: "toys", quantity: 50 }
{ "_id" : 10, "item" : "f4", type: "food", quantity: 75 }
```

### 没有索引的查询

以下查询检索的文档中，**quantity**字段的值在**100**到**200**之间，包括:

```shell
db.inventory.find( { quantity: { $gte: 100, $lte: 200 } } )
```

查询返回以下文档:

```shell
{ "_id" : 2, "item" : "f2", "type" : "food", "quantity" : 100 }
{ "_id" : 3, "item" : "p1", "type" : "paper", "quantity" : 200 }
{ "_id" : 4, "item" : "p2", "type" : "paper", "quantity" : 150 }
```

要查看所选的查询计划，请将[`cursor.explain("executionStats")`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)游标方法链接到**find**命令的末尾:

```shell
db.inventory.find(
   { quantity: { $gte: 100, $lte: 200 } }
).explain("executionStats")
```

[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) 返回以下结果：

```shell
{
   "queryPlanner" : {
         "plannerVersion" : 1,
         ...
         "winningPlan" : {
            "stage" : "COLLSCAN",
            ...
         }
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 3,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 0,
      "totalDocsExamined" : 10,
      "executionStages" : {
         "stage" : "COLLSCAN",
         ...
      },
      ...
   },
   ...
}
```

- [`queryPlanner.winningPlan.stage`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.winningPlan.stage)显示 `COLLSCAN`以指示收集扫描。

  收集扫描表明， [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)必须逐个文档扫描整个收集文档以识别结果。这通常是昂贵的操作，并且可能导致查询缓慢。

- [`executionStats.nReturned`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.nReturned)显示`3`表示查询匹配并返回三个文档。

- [`executionStats.totalKeysExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalKeysExamined)显示`0` 以指示这是查询未使用索引。

- [`executionStats.totalDocsExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalDocsExamined)屏幕显示`10` MongoDB必须扫描十个文档（即集合中的所有文档）才能找到三个匹配的文档。

匹配文档的数量和检查文档的数量之间的差异可能表明，为了提高效率，查询可能会受益于索引的使用。

### 查询与索引

为了支持对**quantity**字段的查询，请在**quantity**字段上添加索引:

```shell
db.inventory.createIndex( { quantity: 1 } )
```

要查看查询计划统计信息，请使用**explain(“executionStats”)**方法:

```shell
db.inventory.find(
   { quantity: { $gte: 100, $lte: 200 } }
).explain("executionStats")
```

该[explain()](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法返回以下结果:

```shell
{
   "queryPlanner" : {
         "plannerVersion" : 1,
         ...
         "winningPlan" : {
               "stage" : "FETCH",
               "inputStage" : {
                  "stage" : "IXSCAN",
                  "keyPattern" : {
                     "quantity" : 1
                  },
                  ...
               }
         },
         "rejectedPlans" : [ ]
   },
   "executionStats" : {
         "executionSuccess" : true,
         "nReturned" : 3,
         "executionTimeMillis" : 0,
         "totalKeysExamined" : 3,
         "totalDocsExamined" : 3,
         "executionStages" : {
            ...
         },
         ...
   },
   ...
}
```

- [`queryPlanner.winningPlan.inputStage.stage`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.winningPlan.inputStage)显示 `IXSCAN`以指示索引的使用。
- [`executionStats.nReturned`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.nReturned) 显示`3`表示查询匹配并返回三个文档。
- [`executionStats.totalKeysExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalKeysExamined)显示`3` 以指示MongoDB扫描了三个索引条目。检查的键数与返回的文档数匹配，这意味着[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)只需检查索引键即可返回结果。在 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)没有扫描所有的文件，只有三个匹配文档不得不被拉入内存中。这导致非常有效的查询。
- [`executionStats.totalDocsExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalDocsExamined)屏幕显示`3` MongoDB扫描了三个文档。

如果没有索引，查询将扫描包含**10**个文档的整个集合，以返回**3**个匹配的文档。查询还必须扫描每个文档的全部内容，可能会将它们拉到内存中。这将导致昂贵的查询操作，并且可能会很慢。

当使用索引运行时，查询扫描了**3**个索引项和**3**个文档，以返回**3**个匹配的文档，从而产生一个非常高效的查询。

### 比较索引的性能

要手动比较使用多个索引的查询的性能，可以将 [`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法与[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法结合使用。

考虑以下查询:

```shell
db.inventory.find( {
   quantity: {
      $gte: 100, $lte: 300
   },
   type: "food"
} )
```

查询返回以下文档:

```shell
{ "_id" : 2, "item" : "f2", "type" : "food", "quantity" : 100 }
{ "_id" : 5, "item" : "f3", "type" : "food", "quantity" : 300 }
```

要支持查询，添加[复合索引](https://docs.mongodb.com/manual/core/index-compound/)。对于[复合索引](https://docs.mongodb.com/manual/core/index-compound/)，字段的顺序很重要。

例如，添加以下两个复合索引。第一个索引首先按数量字段排序，然后按类型字段排序。第二个索引首先按类型排序，然后是**quantity**字段。

```shell
db.inventory.createIndex( { quantity: 1, type: 1 } )
db.inventory.createIndex( { type: 1, quantity: 1 } )
```

评估第一个索引对查询的影响:

```shell
db.inventory.find(
   { quantity: { $gte: 100, $lte: 300 }, type: "food" }
).hint({ quantity: 1, type: 1 }).explain("executionStats")
```

[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法返回如下输出:

```shell
{
   "queryPlanner" : {
      ...
      "winningPlan" : {
         "stage" : "FETCH",
         "inputStage" : {
            "stage" : "IXSCAN",
            "keyPattern" : {
               "quantity" : 1,
               "type" : 1
            },
            ...
            }
         }
      },
      "rejectedPlans" : [ ]
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 2,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 5,
      "totalDocsExamined" : 2,
      "executionStages" : {
      ...
      }
   },
   ...
}
```

MongoDB扫描了5个索引键([`executionStats.totalKeysExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalKeysExamined))以返回2个匹配的文档([`executionStats.nReturned`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.nReturned))。

评估第二个索引对查询的影响:

```shell
db.inventory.find(
   { quantity: { $gte: 100, $lte: 300 }, type: "food" }
).hint({ type: 1, quantity: 1 }).explain("executionStats")
```

[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法返回如下输出:

```shell
{
   "queryPlanner" : {
      ...
      "winningPlan" : {
         "stage" : "FETCH",
         "inputStage" : {
            "stage" : "IXSCAN",
            "keyPattern" : {
               "type" : 1,
               "quantity" : 1
            },
            ...
         }
      },
      "rejectedPlans" : [ ]
   },
   "executionStats" : {
      "executionSuccess" : true,
      "nReturned" : 2,
      "executionTimeMillis" : 0,
      "totalKeysExamined" : 2,
      "totalDocsExamined" : 2,
      "executionStages" : {
         ...
      }
   },
   ...
}
```

MongoDB扫描了2个索引键([`executionStats.totalKeysExamined`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.totalKeysExamined))以返回2个匹配的文档([`executionStats.nReturned`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.nReturned))。

对于这个示例查询，复合索引**{type: 1, quantity: 1}**比复合索引**{quantity: 1, type: 1}**更有效。

​	也可以看看

​	[查询优化](https://docs.mongodb.com/manual/core/query-optimization/)，[查询计划](https://docs.mongodb.com/manual/core/query-plans/)， [优化查询性能](https://docs.mongodb.com/manual/tutorial/optimize-query-performance-with-indexes-and-projections/)， [索引策略](https://docs.mongodb.com/manual/applications/indexes/)



译者：杨帅

校对：杨帅