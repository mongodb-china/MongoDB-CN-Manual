# 解释结果

**在本页面**

- [解释输出](#1)
  - [`queryPlanner`](#11)
  - [`executionStats`](#12)
  - [`serverInfo`](#13)
- [3.0格式变更](#2)
  - [集合扫描与索引使用](#21)
  - [覆盖查询](#22)
  - [索引交集](#23)
  - [`$or` 表达](#24)

为了返回查询计划的信息和查询计划的执行统计信息，MongoDB提供:

- [`db.collection.explain()`](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#db.collection.explain)方法，
- [`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)方法，
- 该[`explain`](https://docs.mongodb.com/manual/reference/command/explain/#dbcmd.explain)命令。

`explain`结果将查询计划呈现为一个阶段树。

```shell
"winningPlan" : {
   "stage" : <STAGE1>,
   ...
   "inputStage" : {
      "stage" : <STAGE2>,
      ...
      "inputStage" : {
         "stage" : <STAGE3>,
         ...
      }
   }
},
```

每个阶段将其结果(即文档或索引键)传递给父节点。叶节点访问集合或索引。内部节点操作子节点产生的文档或索引键。根节点是MongoDB派生结果集的最后一个阶段。

阶段描述了操作；例如

- `COLLSCAN` 用于收集扫描
- `IXSCAN` 用于扫描索引键
- `FETCH` 用于检索文件
- `SHARD_MERGE` 用于合并分片的结果
- `SHARDING_FILTER` 用于从分片中筛选出孤立文档

## <span id="1">解释输出</span>

以下各节列出了该`explain`操作返回的一些关键字段。

> 注意
>
> - 字段列表并不意味着详尽无遗，而只是强调了早期解释版本中的一些关键字段更改。
> - 输出格式在各个发行版之间可能有所更改。

### <span id="11">`queryPlanner`</span>

[`queryPlanner`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner)信息详细说明了[查询优化器](https://docs.mongodb.com/manual/core/query-plans/)选择的计划。

- 未分片集合
- 分片集合

**explain.queryPlanner**

包含有关[查询优化器](https://docs.mongodb.com/manual/core/query-plans/)选择查询计划的信息 。

- `explain.queryPlanner.``namespace`

  一个字符串，它指定`<database>.<collection>`要对其运行查询的名称空间（即 ）。

- `explain.queryPlanner.``indexFilterSet`

  一个布尔值，指定MongoDB是否对[查询形状](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)应用了[索引过滤器](https://docs.mongodb.com/manual/core/query-plans/#index-filters)。

- `explain.queryPlanner.``queryHash`

  一个十六进制字符串，代表[查询形状](https://docs.mongodb.com/manual/reference/glossary/#term-query-shape)的哈希， 并且仅取决于查询形状。 `queryHash`可以帮助识别具有相同查询形状的慢查询（包括写操作的查询过滤器）。

> 注意
>
> 与任何散列函数一样，两个不同的查询形状可能导致相同的散列值。但是，不同查询形状之间不太可能出现哈希冲突。

只有当值为**true**且仅应用于聚合管道操作中的explain时，该字段才会出现。当为**true**时，由于管道已被优化，所以在输出中不会出现聚合阶段信息。

*新版本4.2*

**explain.queryPlanner.winningPlan**

​	详细说明查询优化器选择的计划的文档。MongoDB将计划呈现为一个阶段树;例如，一个阶段可以有一个**inputStage**，如果该阶段有	多个子阶段，则可以有**inputStage**。



​	**explain.queryPlanner.winningPlan.stage**

​		表示舞台名称的字符串。

​		每个阶段由特定于该阶段的信息组成。例如，**IXSCAN**阶段将包括索引边界以及特定于索引扫描的其他数据。如果一个阶段有一个子		阶段或多个子阶段，那么这个阶段将有一个inputStage或inputStage。

​	**explain.queryPlanner.winningPlan.inputStage**

​		描述子阶段的文档，它向父阶段提供文档或索引键。如果父阶段只有一个子阶段，则会显示该字段。

​	**explain.queryPlanner.winningPlan.inputStages**

​		一系列描述子阶段的文档。子阶段将文档或索引键提供给父阶段。*如果*父级具有多个子节点，*则*该字段存在。例如，[$或表达式的](https://docs.mongodb.com/manual/reference/explain-results/#explain-output-or-expression)阶		段或[索引交集](https://docs.mongodb.com/manual/reference/explain-results/#explain-output-index-intersection)会消耗来自多个源的输入。

​	**explain.queryPlanner.rejectedPlans**

​		查询优化器考虑和拒绝的候选计划的数组。如果没有其他候选计划，则该数组可以为空。

### <span id="12">`executionStats`</span>

返回的[`executionStats`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats)信息详细说明了获胜计划的执行情况。为了包括 `executionStats`在结果中，您必须在以下任一位置运行解释：

- [执行状态](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-executionstats)
- [allPlansExecution](https://docs.mongodb.com/manual/reference/method/db.collection.explain/#explain-method-allplansexecution) 详细模式。使用`allPlansExecution`模式包括在[计划选择](https://docs.mongodb.com/manual/core/query-plans/#query-plans-query-optimization)期间捕获的部分执行数据。

- 未分片集合
- 分片集合

**explain.executionStats.executionStages**

​	以阶段树的形式详细说明获奖计划的完成执行情况；即一个阶段可以有一个`inputStage`或多个 `inputStages`。

​	**explain.executionStats.executionStages.works**

​		指定查询执行阶段执行的“工作单位”的数量。查询执行将其工作分为几个小单元。“工作单元”可能包括检查单个索引键，从集合中获		取单个文档，对单个文档应用投影或进行内部簿记。

​	**explain.executionStats.executionStages.advanced**

​		在此阶段返回到其父阶段的中间结果数，或将其*前进*。

​	**explain.executionStats.executionStages.needTime**

​		没有将中间结果提前到其父阶段的工作周期数（请参阅参考资料 [`explain.executionStats.executionStages.advanced`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.executionStages.advanced)）。例		如，索引扫描阶段可能会花费一个工作周期来寻找索引中的新位置，而不是返回索引键。

​		这个工作周期将计入[`explain.executionStats.executionStages.needTime`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.executionStages.needTime)而非计入 	

​		[`explain.executionStats.executionStages.advanced`](https://docs.mongodb.com/manual/reference/explain-results/#explain.executionStats.executionStages.advanced)。

​	**explain.executionStats.executionStages.needYield**

​		存储层请求查询阶段挂起处理并产生其锁的次数。

​	**explain.executionStats.executionStages.saveState**

​		查询阶段挂起处理并保存其当前执行状态的次数，例如，为准备产生锁而做的准备。

​	**explain.executionStats.executionStages.restoreState**

​		查询阶段恢复保存的执行状态的次数，例如，在恢复之前已产生的锁之后。

​	**explain.executionStats.executionStages.isEOF**

​		指定执行阶段是否已到达流的末尾：

​			如果`true`或`1`，则执行阶段已到达流的末尾。

​			如果`false`或`0`，则阶段可能仍会返回结果。例如，考虑一个具有限制的查询，其执行阶段由查询`LIMIT`的输入阶段组

​			成`IXSCAN`。如果查询返回的值超过指定的限制，则该`LIMIT`阶段将报告，但其基础阶段将报告。**isEOF: 1IXSCANisEOF: 0**

​	**explain.executionStats.executionStages.inputStage.keysExamined**

​			对于扫描索引的查询执行阶段（例如IXSCAN）， `keysExamined`是在索引扫描过程中检查的入站和出站键的总数。如果索引扫描			由单个连续范围的键组成，则仅需要检查入站键。如果索引范围由几个键范围组成，则索引扫描执行过程可能会检查越界键，以便			从一个范围的末尾跳到下一个范围的末尾。

考虑以下示例，其中有一个字段索引， `x`并且集合包含100个文档，其`x`值从1到100：

```shell
db.keys.find( { x : { $in : [ 3, 4, 50, 74, 75, 90 ] } } ).explain( "executionStats" )
```

​		查询将扫描键**3**和**4**。然后它将扫描键**5**，检测它是否超出范围，并跳到下一个键**50**。

​		继续这个过程，查询扫描键3、4、5、50、51、74、75、76、90和91。键5,51,76和91是仍在检查的超出范围的	

​		键。**keysExamined**的值为10。

​	**explain.executionStats.executionStages.inputStage.docsExamined**

​			指定在查询执行阶段扫描的文档数量。

​			用于**COLLSCAN**阶段，以及从集合检索文档的阶段(例如**FETCH**)

​	**explain.executionStats.executionStages.inputStage.seeks**

​			版本3.4中的新特性:仅用于索引扫描**(IXSCAN)**阶段。

​			为了完成索引扫描，我们必须将索引游标查找到新位置的次数。

   **explain.executionStats.allPlansExecution**

​			包含在计划选择阶段捕获的胜出计划和被否决计划的部分执行信息。只有当explain在所有计划执行冗长模式下运行时，该字段才			会出现。

### <span id="13">`serverInfo`</span>

- 未分片集合
- 分片集合

对于未分片的集合，`explain`返回`serverInfo`MongoDB实例的以下 信息：

```shell
“ serverInfo”：{ 
   “ host”：<string>，
   “ port”：<int>，
   “ version”：<string>，
   “ gitVersion”：<string> 
}
```

对于分片集合，`explain`返回`serverInfo`每个访问的分片的，并返回的 顶级 `serverInfo`对象[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)。

```shell
"queryPlanner" : {
   ...
   "winningPlan" : {
      "stage" : <STAGE1>,
      "shards" : [
         {
            "shardName" : <string>,
            "connectionString" : <string>,
            "serverInfo" : {
               "host" : <string>,
               "port" : <int>,
               "version" : <string>,
               "gitVersion" : <string>
            },
            ...
         }
         ...
      ]
   }
},
"serverInfo" : {      // serverInfo for mongos
  "host" : <string>,
  "port" : <int>,
  "version" : <string>,
  "gitVersion" : <string>
}
```

## <span id="2">3.0格式变更</span>

从MongoDB 3.0开始，结果的格式和字段`explain` 与以前的版本已更改。以下列出了一些主要区别。

### <span id="21">集合扫描与索引使用</span>

如果查询计划者选择了集合扫描，则解释结果将包括一个`COLLSCAN`阶段。

如果查询计划者选择了索引，则说明结果包括一个 `IXSCAN`阶段。该阶段包括诸如索引键样式，遍历方向和索引边界之类的信息。

在以前的MongoDB版本中，`cursor.explain()`返回的 `cursor`字段值为：

- `BasicCursor` 用于收集扫描，
- `BtreeCursor <index name> [<direction>]` 用于索引扫描。

有关收集扫描和索引扫描的执行统计信息的更多信息，请参见[分析查询性能](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)。

### <span id="22">覆盖查询</span>

当索引涵盖查询时，MongoDB既可以匹配查询条件**，也**可以仅使用索引键返回结果；即MongoDB无需检查集合中的文档即可返回结果。

当索引覆盖查询时，解释结果的`IXSCAN` 阶段**不是**该阶段的后代`FETCH`，而在 [executionStats中](https://docs.mongodb.com/manual/reference/explain-results/#executionstats)，`totalDocsExamined`is是`0`。

在MongoDB的早期版本中，`cursor.explain()`返回该 `indexOnly`字段以指示索引是否覆盖查询。

### <span id="23">索引交集</span>

对于[索引交叉计划](https://docs.mongodb.com/manual/core/index-intersection/)，结果将包括一个`AND_SORTED`阶段或一个`AND_HASH` 包含[`inputStages`](https://docs.mongodb.com/manual/reference/explain-results/#explain.queryPlanner.winningPlan.inputStages)详细描述索引的数组的阶段。例如：

```shell
{ 
   “ stage”  ： “ AND_SORTED” ，
   “ inputStages”  ： [ 
      { 
         “ stage”  ： “ IXSCAN” ，
         ... 
      }，
      { 
         “ stage”  ： “ IXSCAN” ，
         ... 
      } 
   ] 
}
```

在以前的MongoDB版本中，`cursor.explain()`返回`cursor`值为index交集的 字段。`Complex Plan`

### <span id="24">`$or`表达式</span>

如果MongoDB对[`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or)表达式使用索引，则结果将包括`OR`带有`inputStages`详细索引的数组的阶段 ；例如：

复制复制的

```shell
{ 
   “ stage”  ： “ OR” ，
   “ inputStages”  ： [ 
      { 
         “ stage”  ： “ IXSCAN” ，
         ... 
      }，
      { 
         “ stage”  ： “ IXSCAN” ，
         ... 
      }，
      ... 
   ] 
}
```

在MongoDB的早期版本中，`cursor.explain()`返回`clauses`详细说明索引的 数组。

#### 分类阶段

如果MongoDB可以使用索引扫描来获取请求的排序顺序，则结果将**不**包含`SORT`阶段。否则，如果MongoDB无法使用索引进行排序，则`explain`结果将包括一个 `SORT`阶段。

在MongoDB 3.0之前，`cursor.explain()`返回此 `scanAndOrder`字段以指定MongoDB是否可以使用索引顺序返回排序的结果。



译者：杨帅

校对：杨帅