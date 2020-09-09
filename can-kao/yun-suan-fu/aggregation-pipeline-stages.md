# 聚合管道阶段

在`db.collection.aggregate`和`db.aggregate`方法中 ，管道阶段出现在列表中。文档按顺序通过各个阶段。

## 阶段

### db.collection.aggregate\(\)阶段

除了[`$out`](aggregation-pipeline-stages.md)、[`$merge`](aggregation-pipeline-stages.md)和[`$geoNear`](aggregation-pipeline-stages.md)阶段之外，所有阶段都可以在管道中出现多次。

> **注意**
>
> 有关特定运算符的详细信息，包括语法和示例，请单击特定运算符以转到其参考页。

```text
db.collection.aggregate( [ { <stage> }, ... ] )
```

| 阶段 | 描述 |
| :--- | :--- |
| [$addFields](aggregation-pipeline-stages.md) | 向文档添加新字段。与$project类似，$addFields重塑了流中的每个文档;具体而言，通过向输出文档添加新字段，该文档包含输入文档和新添加字段中的现有字段。 `$set`是的别名`$addFields`。 |
| [$bucket](aggregation-pipeline-stages.md) | 根据指定的表达式和存储段边界将传入文档分类为称为存储段的组。 |
| [$bucketAuto](aggregation-pipeline-stages.md) | 根据指定的表达式将传入的文档分类为特定数量的组\(称为存储桶\)。自动确定存储桶边界，以尝试将文档均匀地分配到指定数量的存储桶中。 |
| [$collStats](aggregation-pipeline-stages.md) | 返回有关集合或视图的统计信息。 |
| [$count](aggregation-pipeline-stages.md) | 返回聚合管道此阶段的文档数量计数。 |
| [$facet](aggregation-pipeline-stages.md) | 在同一阶段的同一组输入文档上处理多个聚合管道。支持在一个阶段中创建能够表征多维或多面数据的多面聚合。 |
| [$geoNear](aggregation-pipeline-stages.md) | 基于与地理空间点的接近度返回有序的文档流。将`$match`，`$sort`和`$limit`的功能合并到地理空间数据中。输出文档包括附加距离字段，并且可以包括位置标识符字段。 |
| [$graphLookup](aggregation-pipeline-stages.md) | 对集合执行递归搜索。对于每个输出文档，添加一个新的 array 字段，该字段包含该文档的递归搜索的遍历结果。 |
| [$group](aggregation-pipeline-stages.md) | 按指定的标识符表达式对文档进行分组，并将累加器表达式\(如果指定\)应用于每个 group。消耗所有输入文档，并为每个不同的 group 输出一个文档。输出文档仅包含标识符字段，如果指定，则包含累积字段。 |
| [$indexStats](aggregation-pipeline-stages.md) | 返回有关集合的每个索引的使用的统计信息。 |
| [$limit](aggregation-pipeline-stages.md) | 将未修改的前 n 个文档传递给管道，其中 n 是指定的限制。对于每个输入文档，输出一个文档\(对于前 n 个文档\)或零文档\(在前 n 个文档之后\)。 |
| [$listSessions](aggregation-pipeline-stages.md) | 列出所有活动时间已足够长以传播到`system.sessions`集合的会话。 |
| [$lookup](aggregation-pipeline-stages.md) | 对同一数据库中的另一个集合执行左外连接，以从“已连接”集合中过滤文档以进行处理。 |
| [$match](aggregation-pipeline-stages.md) | 过滤文档流以仅允许匹配的文档未经修改地传递到下一个管道阶段。 `$match`使用标准的 MongoDB 查询。对于每个输入文档，输出一个文档\(匹配\)或零文档\(不匹配\)。 |
| [$merge](aggregation-pipeline-stages.md) | 将聚合管道的结果文档写入集合。该阶段可以将结果合并（插入新文档，合并文档，替换文档，保留现有文档，使操作失败，使用自定义更新管道处理文档）将结果合并到输出集合中。要使用该`$merge`阶段，它必须是管道中的最后一个阶段。 _4.2版中的新功能。_ |
| [$out](aggregation-pipeline-stages.md) | 将聚合管道的结果文档写入集合。要使用$out阶段，它必须是管道中的最后一个阶段。 |
| [$planCacheStats](aggregation-pipeline-stages.md) | 返回集合的计划缓存信息。 |
| [$project](aggregation-pipeline-stages.md) | 重塑流中的每个文档，例如通过添加新字段或删除现有字段。对于每个输入文档，输出一个文档。 另请参阅`$unset`删除现有字段。 |
| [$redact](aggregation-pipeline-stages.md) | 通过基于文档本身中存储的信息限制每个文档的内容来重塑流中的每个文档。包含`$project`和`$match`的功能。可用于实现字段级编辑。对于每个输入文档，输出一个或零个文档。 |
| [$replaceRoot](aggregation-pipeline-stages.md) | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶部级别。 `$replaceWith`是`$replaceRoot`阶段的别名 。 |
| [$replaceWith](aggregation-pipeline-stages.md) | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶部级别。 `$replaceWith`是`$replaceRoot`阶段的别名 。 |
| [$sample](aggregation-pipeline-stages.md) | 从输入中随机选择指定数量的文档。 |
| [$set](aggregation-pipeline-stages.md) | 将新字段添加到文档。与`$project`相似，`$set`重塑流中的每个文档；具体而言，通过向输出文档添加新字段，该输出文档既包含输入文档中的现有字段，又包含新添加的字段。 `$set`是`$addFields`阶段的别名。 |
| [$skip](aggregation-pipeline-stages.md) | 跳过前 n 个文档，其中 n 是指定的跳过编号，并将未修改的其余文档传递给管道。对于每个输入文档，输出零文档\(对于前 n 个文档\)或一个文档\(如果在前 n 个文档之后\)。 |
| [$sort](aggregation-pipeline-stages.md) | 按指定的排序 key 重新排序文档流。只有顺序改变;文档保持不变。对于每个输入文档，输出一个文档。 |
| [sortByCount](aggregation-pipeline-stages.md) | 根据指定表达式的 value 对传入文档进行分组，然后计算每个不同 group 中的文档计数。 |
| [$unset](aggregation-pipeline-stages.md) | 从文档中删除/排除字段。 `$unset`是`$project`删除字段的阶段的别名。 |
| [$unwind](aggregation-pipeline-stages.md) | 从输入文档解构 array 字段以输出每个元素的文档。每个输出文档都使用元素 value 替换 array。对于每个输入文档，输出 n 个文档，其中 n 是 array 元素的数量，对于空 array 可以为零。 |

对于要在管道阶段使用的聚合表达式运算符，请参阅聚合管道操作符。

### db.aggregate\(\)阶段

从 version 3.6 开始，MongoDB 还提供了db.aggregate方法：

```text
db.aggregate( [ { <stage> }, ... ] )
```

以下阶段使用`db.aggregate()`方法而不是`db.collection.aggregate()`方法。

| 阶段 | 描述 |
| :--- | :--- |
| [$currentOp](aggregation-pipeline-stages.md) | 返回有关 MongoDB 部署的活动 and/or 休眠操作的信息。 |
| [$listLocalSessions](aggregation-pipeline-stages.md) | 列出最近在当前连接的mongos或mongod实例上使用的所有 active 会话。这些会话可能尚未传播到`system.sessions`集合。 |

### 阶段可用于更新

从MongoDB 4.2开始，您可以使用聚合管道在以下位置进行更新：

| 命令 | mongo shell方法 |
| :--- | :--- |
| [findAndModify](aggregation-pipeline-stages.md) | [db.collection.findOneAndUpdate（）](aggregation-pipeline-stages.md) [db.collection.findAndModify（）](aggregation-pipeline-stages.md) |
| [update](aggregation-pipeline-stages.md) | [db.collection.updateOne（）](aggregation-pipeline-stages.md) [db.collection.updateMany（）](aggregation-pipeline-stages.md) [db.collection.update（）](aggregation-pipeline-stages.md)  [Bulk.find.update（）](aggregation-pipeline-stages.md) [Bulk.find.updateOne（）](aggregation-pipeline-stages.md) [Bulk.find.upsert（）](aggregation-pipeline-stages.md) |

对于更新，管道可以包括以下阶段：

* [`$addFields`](aggregation-pipeline-stages.md) 及其别名 [`$set`](aggregation-pipeline-stages.md)
* [`$project`](aggregation-pipeline-stages.md) 及其别名 [`$unset`](aggregation-pipeline-stages.md)
* [`$replaceRoot`](aggregation-pipeline-stages.md)及其别名[`$replaceWith`](aggregation-pipeline-stages.md)。

## 按字母顺序排列的阶段列表

| 阶段 | 描述 |
| :--- | :--- |
| [$addFields](aggregation-pipeline-stages.md) | 向文档添加新字段。输出包含输入文档和新添加字段中所有现有字段的文档。 |
| [$bucket](aggregation-pipeline-stages.md) | 根据指定的表达式和存储段边界将传入文档分类为称为存储段的组。 |
| [$bucketAuto](aggregation-pipeline-stages.md) | 根据指定的表达式将传入的文档分类为特定数量的组\(称为存储桶\)。自动确定存储桶边界，以尝试将文档均匀地分配到指定数量的存储区中。 |
| [$collStats](aggregation-pipeline-stages.md) | 返回有关集合或视图的统计信息。 |
| [$count](aggregation-pipeline-stages.md) | 返回聚合管道此阶段的文档数量计数。 |
| [$currentOp](aggregation-pipeline-stages.md) | 返回有关 MongoDB 部署的活动 and/or 休眠操作的信息。要运行，请使用`db.aggregate()`方法。 |
| [$facet](aggregation-pipeline-stages.md) | 在同一组输入文档的单个阶段内处理多个聚合管道。允许创建能够在单个阶段中跨多个维度或方面表征数据的 多面聚合。 |
| [$geoNear](aggregation-pipeline-stages.md) | 基于与地理空间点的接近度返回有序的文档流。将`$match`，`$sort`和`$limit`的功能合并到地理空间数据中。输出文档包括附加距离字段，并且可以包括位置标识符字段。 |
| [$graphLookup](aggregation-pipeline-stages.md) | 对集合执行递归搜索。对于每个输出文档，添加一个新的 array 字段，该字段包含该文档的递归搜索的遍历结果。 |
| [$group](aggregation-pipeline-stages.md) | 按指定的标识符表达式对文档进行分组，并将累加器表达式\(如果指定\)应用于每个 group。消耗所有输入文档，并为每个不同的 group 输出一个文档。输出文档仅包含标识符字段，如果指定，则包含累积字段。 |
| [$indexStats](aggregation-pipeline-stages.md) | 返回有关集合的每个索引的使用的统计信息。 |
| [$limit](aggregation-pipeline-stages.md) | 将未修改的前 n 个文档传递给管道，其中 n 是指定的限制。对于每个输入文档，输出一个文档\(对于前 n 个文档\)或零文档\(在前 n 个文档之后\)。 |
| [$listLocalSessions](aggregation-pipeline-stages.md) | 列出最近在当前连接的mongos或mongod实例上使用的所有 active 会话。这些会话可能尚未传播到`system.sessions`集合。 |
| [$listSessions](aggregation-pipeline-stages.md) | 列出所有活动时间已足够长以传播到`system.sessions`集合的会话。 |
| [$lookup](aggregation-pipeline-stages.md) | 对同一数据库中的另一个集合执行左外连接，以从“已连接”集合中过滤文档以进行处理。 |
| [$match](aggregation-pipeline-stages.md) | 过滤文档流以仅允许匹配的文档未经修改地传递到下一个管道阶段。 `$match`使用标准的 MongoDB 查询。对于每个输入文档，输出一个文档\(匹配\)或零文档\(不匹配\)。 |
| [$merge](aggregation-pipeline-stages.md) | 将聚合管道的结果文档写入集合。该阶段可以将结果合并（插入新文档，合并文档，替换文档，保留现有文档，使操作失败，使用自定义更新管道处理文档）将结果合并到输出集合中。要使用该`$merge`阶段，它必须是管道中的最后一个阶段。 _4.2版中的新功能。_ |
| [$out](aggregation-pipeline-stages.md) | 将聚合管道的结果文档写入集合。要使用`$out`阶段，它必须是管道中的最后一个阶段。 |
| [$planCacheStats](aggregation-pipeline-stages.md) | 返回集合的计划缓存信息。 |
| [$project](aggregation-pipeline-stages.md) | 重新整形流中的每个文档，例如添加新字段或删除现有字段。对于每个输入文档，输出一个文档。 |
| [$redact](aggregation-pipeline-stages.md) | 通过基于文档本身中存储的信息限制每个文档的内容来重塑流中的每个文档。包含`$project`和`$match`的功能。可用于实现字段级编辑。对于每个输入文档，输出一个或零个文档。 |
| [$replaceRoot](aggregation-pipeline-stages.md) | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶部级别。 |
| [$replaceWith](aggregation-pipeline-stages.md) | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶部级别。 别名`$replaceRoot`。 |
| [$sample](aggregation-pipeline-stages.md) | 从输入中随机选择指定数量的文档。 |
| [$set](aggregation-pipeline-stages.md) | 将新字段添加到文档。输出包含输入文档中所有现有字段和新添加的字段的文档。 别名`$addFields`。 |
| [$skip](aggregation-pipeline-stages.md) | 跳过前 n 个文档，其中 n 是指定的跳过编号，并将未修改的其余文档传递给管道。对于每个输入文档，输出零文档\(对于前 n 个文档\)或一个文档\(如果在前 n 个文档之后\)。 |
| [$sort](aggregation-pipeline-stages.md) | 按指定的排序 key 重新排序文档流。只有顺序改变;文件保持不变。对于每个输入文档，输出一个文档。 |
| [$sortByCount](aggregation-pipeline-stages.md) | 根据指定表达式的值对传入文档进行分组，然后计算每个不同组中的文档数。 |
| [$unwind](aggregation-pipeline-stages.md) | 从输入文档解构 array 字段以输出每个元素的文档。每个输出文档都使用元素 value 替换 array。对于每个输入文档，输出 n 个文档，其中 n 是 array 元素的数量，对于空 array 可以为零。 |

译者：李冠飞

校对：

