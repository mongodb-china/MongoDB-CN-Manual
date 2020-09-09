# 聚合命令对比

在本页面

* [聚合命令比较表](aggregation-commands-commparison.md#aggregation-commands-comparison-table)

> **\[success\] 建议**
>
> 从4.4版开始，MongoDB添加[`$accumulator`](aggregation-commands-commparison.md)和 [`$function`](aggregation-commands-commparison.md)运算符。使用 [`$accumulator`](aggregation-commands-commparison.md)和[`$function`](aggregation-commands-commparison.md)， [`mapReduce`](aggregation-commands-commparison.md)可以使用聚合运算符重写表达式。
>
> 即使是4.4版本之前，一些map-reduce表达式也可以使用改写[其他聚合管道运算符](aggregation-commands-commparison.md)，如[`$group`](aggregation-commands-commparison.md)， [`$merge`](aggregation-commands-commparison.md)等。

## 聚合命令比较表

以下表格简要概述了 MongoDB 聚合命令的特点。

|  | [Aggregate](aggregation-commands-commparison.md) / [db.collection.aggregate\(\)](aggregation-commands-commparison.md) | [MapReduce](aggregation-commands-commparison.md) / [db.collection.mapReduce\(\)](aggregation-commands-commparison.md) |
| :--- | :--- | :--- |
| **描述** | 旨在提高聚合任务的性能和可用性的具体目标。  使用“管道”方法，其中 objects 在通过一系列管道操作符\(如[$group](aggregation-commands-commparison.md)，[$match](aggregation-commands-commparison.md)和[$sort](aggregation-commands-commparison.md)\)时进行转换。  有关管道运算符的更多信息，请参见[聚合管道操作符](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/docs/Reference/Operators/Aggregation-Pipline-Operators.md)。 | 实现 Map-Reduce 聚合以处理大型数据集。 |
| **主要特点** | 可以根据需要重复管道操作符。  管道运算符不需要为每个输入文档生成一个输出文档。  还可以生成新文档或过滤掉文档。 通过在版本4.2中添加`$merge`，可以创建按需的物化视图，在该视图中可以逐步运行管道来更新输出集合的内容。`$merge`可以将结果\(插入新文档、合并文档、替换文档、保留现有文档、使操作失败、使用自定义更新管道处理文档\)合并到现有集合中。 | 除了分组操作之外，还可以执行复杂的聚合任务以及对不断增长的数据集执行增量聚合。  见[Map-Reduce 例子](aggregation-commands-commparison.md)和[执行增量 Map-Reduce](aggregation-commands-commparison.md)。 |
| **灵活性** | 从4.4版开始，可以使用[`$accumulator`](aggregation-commands-commparison.md)和[`$function`](aggregation-commands-commparison.md)定义自定义聚合表达式。 在以前的版本中，只能使用聚合管道支持的运算符和表达式。  但是，可以使用[`$project`](aggregation-commands-commparison.md) 管道运算符添加计算字段，创建新的虚拟子对象以及将子字段提取到结果的顶层。 有关更多信息，请参阅[`$project`](aggregation-commands-commparison.md)以及[聚合管道操作符](aggregation-commands-commparison.md)，以了解有关所有可用管道操作符的更多信息。 | 自定义`map`，`reduce`和`finalize JavaScript` 函数为聚合逻辑提供了灵活性。  有关功能的详细信息和限制，请参阅[MapReduce](aggregation-commands-commparison.md)。 |
| **输出结果** | 以游标的形式返回结果。如果管道包含[`$out`](aggregation-commands-commparison.md)阶段或[`$merge`](aggregation-commands-commparison.md)阶段，则游标为空。 使用[`$out`](aggregation-commands-commparison.md)，您可以完全替换现有的输出集合或输出到新的集合。详情见[`$out`](aggregation-commands-commparison.md)。 使用[`$merge`](aggregation-commands-commparison.md)，您可以输出到新的或现有的集合。对于现有的cllections，可以指定如何将结果合并到输出集合中\(插入新文档、合并文档、替换文档、保留现有文档、使操作失败、使用自定义更新管道处理文档\)。有关详细信息，请参见[`$merge`](aggregation-commands-commparison.md)。 | 返回各种选项的结果\(内联，新集合，合并，替换，减少\)。有关输出选项的详细信息，请参阅[MapReduce](aggregation-commands-commparison.md)。 |
| **分片** | 支持非分片和分片输入集合。 [`$merge`](aggregation-commands-commparison.md)可以输出到非分片或分片集合。 | 支持非分片和分片输入集合。 |
| **更多信息** | [聚合管道](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/Aggregation/Aggregation-Pipline.md) [db.collection.aggregate\(\)](aggregation-commands-commparison.md) [aggregate](aggregation-commands-commparison.md) | [Map-Reduce](../map-reduce/) [db.collection.mapReduce\(\)](aggregation-commands-commparison.md) [mapReduce](aggregation-commands-commparison.md) |

也可以看看

* [Map-Reduce to Aggregate](aggregation-commands-commparison.md)

译者：李冠飞

校对：李冠飞

