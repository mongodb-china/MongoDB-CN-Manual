# 聚合管道优化

在本页面

* [投影优化](aggregation-pipeline-optimization.md#projection-optimization)
* [管道序列优化](aggregation-pipeline-optimization.md#pipeline-sequence-optimization)
* [管道聚结优化](aggregation-pipeline-optimization.md#pipeline-coalescence-optimization)
* [例子](aggregation-pipeline-optimization.md#example)

聚合管道操作具有优化阶段，该阶段试图重塑管道以改善性能。

要查看优化程序如何转换特定聚合管道，请在[db.collection.aggregate\(\)](aggregation-pipeline-optimization.md)方法中包含[explain](aggregation-pipeline-optimization.md)选项。

优化可能会在不同版本之间发生变化。

## 投影优化

聚合管道可以确定它是否仅需要文档中的字段的子集来获得结果。如果是这样，管道将只使用那些必需的字段，减少通过管道的数据量。

## 管道序列优化

### \($project or $unset or $addFields or $set\) + $match 序列优化

对于包含投影阶段\([$project](aggregation-pipeline-optimization.md)或[$unset](aggregation-pipeline-optimization.md)或[$addFields](aggregation-pipeline-optimization.md)或[$set](aggregation-pipeline-optimization.md)\)后跟[$match](aggregation-pipeline-optimization.md)阶段的聚合管道，MongoDB 将[$match](aggregation-pipeline-optimization.md)阶段中不需要在投影阶段计算的值的任何过滤器移动到投影前的新[$match](aggregation-pipeline-optimization.md)阶段。

如果聚合管道包含多个投影 and/or [$match](aggregation-pipeline-optimization.md)阶段，MongoDB 会为每个[$match](aggregation-pipeline-optimization.md)阶段执行此优化，将每个[$match](aggregation-pipeline-optimization.md)过滤器移动到过滤器不依赖的所有投影阶段之前。

考虑以下阶段的管道：

```text
{ $addFields: {
    maxTime: { $max: "$times" },
    minTime: { $min: "$times" }
} },
{ $project: {
    _id: 1, name: 1, times: 1, maxTime: 1, minTime: 1,
    avgTime: { $avg: ["$maxTime", "$minTime"] }
} },
{ $match: {
    name: "Joe Schmoe",
    maxTime: { $lt: 20 },
    minTime: { $gt: 5 },
    avgTime: { $gt: 7 }
} }
```

优化器将[$match](aggregation-pipeline-optimization.md)阶段分成四个单独的过滤器，一个用于[$match](aggregation-pipeline-optimization.md)查询文档中的每个键。然后优化器将每个筛选器移动到尽可能多的投影阶段之前，根据需要创建新的[$match](aggregation-pipeline-optimization.md)阶段。鉴于此示例，优化程序生成以下优化管道：

```text
{ $match: { name: "Joe Schmoe" } },
{ $addFields: {
                maxTime: { $max: "$times" },
        minTime: { $min: "$times" }
} },
{ $match: { maxTime: { $lt: 20 }, minTime: { $gt: 5 } } },
{ $project: {
        _id: 1, name: 1, times: 1, maxTime: 1, minTime: 1,
        avgTime: { $avg: ["$maxTime", "$minTime"] }
} },
{ $match: { avgTime: { $gt: 7 } } }
```

[$match](aggregation-pipeline-optimization.md)过滤器`{ avgTime: { $gt: 7 } }`取决于[$project](aggregation-pipeline-optimization.md)阶段来计算`avgTime`字段。 [$project](aggregation-pipeline-optimization.md)阶段是此管道中的最后一个投影阶段，因此`avgTime`上的[$match](aggregation-pipeline-optimization.md)过滤器无法移动。

`maxTime`和`minTime`字段在[$addFields](aggregation-pipeline-optimization.md)阶段计算，但不依赖于[$project](aggregation-pipeline-optimization.md)阶段。优化器为这些字段上的过滤器创建了一个新的[$match](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/Aggregation/Aggregation-Pipeline/reference-operator-aggregation-match.html#pipe._S_match)阶段，并将其放在[$project](aggregation-pipeline-optimization.md)阶段之前。

[$match](aggregation-pipeline-optimization.md)过滤器`{ name: "Joe Schmoe" }`不使用在[$project](aggregation-pipeline-optimization.md)或[$addFields](aggregation-pipeline-optimization.md)阶段计算的任何值，因此它在两个投影阶段之前被移动到新的[$match](aggregation-pipeline-optimization.md)阶段。

> **\[success\] 注意**
>
> 优化后，过滤器`{ name: "Joe Schmoe" }`位于管道开头的[$match](aggregation-pipeline-optimization.md)阶段。这具有额外的好处，即允许聚合在最初查询集合时在`name`字段上使用索引。有关更多信息，请参见[管道操作符和索引](aggregation-pipeline-optimization.md)。

### $sort + $match 序列优化

如果序列中带有[$sort](aggregation-pipeline-optimization.md)后跟[$match](aggregation-pipeline-optimization.md)，则[$match](aggregation-pipeline-optimization.md)会移动到[$sort](aggregation-pipeline-optimization.md)之前，以最大程度的减少要排序的对象的数量。例如，如果管道包含以下阶段：

```text
{ $sort: { age : -1 } }, 
{ $match: { status: 'A' } }
```

在优化阶段，优化程序将序列转换为以下内容：

```text
{ $match: { status: 'A' } }, 
{ $sort: { age : -1 } }
```

### $redact + $match 序列优化

如果可能，当管道的[$redact](aggregation-pipeline-optimization.md)阶段紧在[$match](aggregation-pipeline-optimization.md)阶段之后时，聚合有时可以在[$redact](aggregation-pipeline-optimization.md)阶段之前添加[$match](aggregation-pipeline-optimization.md)阶段的一部分。如果添加的[$match](aggregation-pipeline-optimization.md)阶段位于管道的开头，则聚合可以使用索引以及查询集合来限制进入管道的文档数。有关更多信息，请参见[管道操作符和索引](aggregation-pipeline-optimization.md)。 例如，如果管道包含以下阶段：

```text
{ $redact: { $cond: { if: { $eq: [ "$level", 5 ] }, then: "$$PRUNE", else: "$$DESCEND" } } },
{ $match: { year: 2014, category: { $ne: "Z" } } }
```

优化器可以在[$redact](aggregation-pipeline-optimization.md)阶段之前添加相同的[$match](aggregation-pipeline-optimization.md)阶段：

```text
{ $match: { year: 2014 } },
{ $redact: { $cond: { if: { $eq: [ "$level", 5 ] }, then: "$$PRUNE", else: "$$DESCEND" } } },
{ $match: { year: 2014, category: { $ne: "Z" } } }
```

### `$project`/ `$unset` + `$skip`序列优化

_3.2版本中的新功能。_

当有一个[`$project`](aggregation-pipeline-optimization.md)或[`$unset`](aggregation-pipeline-optimization.md)之后跟有[`$skip`](aggregation-pipeline-optimization.md)序列时，[`$skip`](aggregation-pipeline-optimization.md) 会移至[`$project`](aggregation-pipeline-optimization.md)之前。例如，如果管道包括以下阶段：

```text
{ $sort: { age : -1 } },
{ $project: { status: 1, name: 1 } },
{ $skip: 5 }
```

在优化阶段，优化器将序列转换为以下内容：

```text
{ $sort: { age : -1 } },
{ $skip: 5 },
{ $project: { status: 1, name: 1 } }
```

## 管道聚合优化

如果可能，优化阶段将一个管道阶段合并到其前身。通常，合并发生在任何序列重新排序优化之后。

### `$sort` + `$limit`合并

_Mongodb 4.0版本的改变。_

当一个[`$sort`](aggregation-pipeline-optimization.md)先于[`$limit`](aggregation-pipeline-optimization.md)，优化器可以聚结[`$limit`](aggregation-pipeline-optimization.md)到[`$sort`](aggregation-pipeline-optimization.md)，如果没有中间阶段的修改文件（例如，使用数[`$unwind`](aggregation-pipeline-optimization.md)，[`$group`](aggregation-pipeline-optimization.md)）。如果有管道阶段会更改和阶段之间的文档数，则MongoDB将不会合并[`$limit`](aggregation-pipeline-optimization.md)到 。[`$sort`](aggregation-pipeline-optimization.md)[`$sort`](aggregation-pipeline-optimization.md)[`$limit`](aggregation-pipeline-optimization.md)

例如，如果管道包括以下阶段：

```text
{ $sort : { age : -1 } },
{ $project : { age : 1, status : 1, name : 1 } },
{ $limit: 5 }
```

在优化阶段，优化器将序列合并为以下内容：

```text
{
    "$sort" : {
       "sortKey" : {
          "age" : -1
       },
       "limit" : NumberLong(5)
    }
},
{ "$project" : {
         "age" : 1,
         "status" : 1,
         "name" : 1
  }
}
```

这样，排序操作就可以仅在执行过程中保持最高`n`结果，这`n`是指定的限制，MongoDB仅需要将`n`项目存储在内存中 [\[1\]](aggregation-pipeline-optimization.md)。有关更多信息，请参见[$ sort运算符和内存](aggregation-pipeline-optimization.md)。

> 用`$skip`进行序列优化
>
> 如果[`$skip`](aggregation-pipeline-optimization.md)在[`$sort`](aggregation-pipeline-optimization.md) 和[`$limit`](aggregation-pipeline-optimization.md)阶段之间有一个阶段，MongoDB将合并 [`$limit`](aggregation-pipeline-optimization.md)到该[`$sort`](aggregation-pipeline-optimization.md)阶段并增加该 [`$limit`](aggregation-pipeline-optimization.md)值[`$skip`](aggregation-pipeline-optimization.md)。有关示例，请参见 [$ sort + $ skip + $ limit序列](aggregation-pipeline-optimization.md)。
>
> [\[1\]](https://docs.mongodb.com/manual/core/aggregation-pipeline-optimization/#id1)当优化仍将适用 `allowDiskUse`是`true`与`n`项目超过 [聚集内存限制](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#agg-memory-restrictions)。

### `$limit`+ `$limit`合并

当[`$limit`](aggregation-pipeline-optimization.md)紧接着另一个时 [`$limit`](aggregation-pipeline-optimization.md)，两个阶段可以合并为一个阶段 [`$limit`](aggregation-pipeline-optimization.md)，其中限制量为两个初始限制量中的较小者。例如，管道包含以下序列：

```text
{ $limit: 100 },
{ $limit: 10 }
```

然后，第二[`$limit`](aggregation-pipeline-optimization.md)级可以聚结到第一 [`$limit`](aggregation-pipeline-optimization.md)阶段，并导致在单个[`$limit`](aggregation-pipeline-optimization.md) 阶段，即限制量`10`是两个初始极限的最小`100`和`10`。

```text
{ $limit: 10 }
```

### `$skip`+ `$skip`合并

当[`$skip`](aggregation-pipeline-optimization.md)紧跟另一个[`$skip`](aggregation-pipeline-optimization.md)，这两个阶段可合并成一个单一的[`$skip`](aggregation-pipeline-optimization.md)，其中跳过量为总和的两个初始跳过量。例如，管道包含以下序列：

```text
{ $skip: 5 },
{ $skip: 2 }
```

然后，第二[`$skip`](aggregation-pipeline-optimization.md)阶段可以合并到第一 [`$skip`](aggregation-pipeline-optimization.md)阶段，并导致单个[`$skip`](aggregation-pipeline-optimization.md) 阶段，其中跳过量`7`是两个初始限制`5`和的总和`2`。

```text
{ $skip: 7 }
```

### `$match`+ `$match`合并

当一个[`$match`](aggregation-pipeline-optimization.md)紧随另一个紧随其后时 [`$match`](aggregation-pipeline-optimization.md)，这两个阶段可以合并为一个单独 [`$match`](aggregation-pipeline-optimization.md)的条件 [`$and`](aggregation-pipeline-optimization.md)。例如，管道包含以下序列：

```text
{ $match: { year: 2014 } },
{ $match: { status: "A" } }
```

然后，第二[`$match`](aggregation-pipeline-optimization.md)阶段可以合并到第一 [`$match`](aggregation-pipeline-optimization.md)阶段，从而形成一个[`$match`](aggregation-pipeline-optimization.md) 阶段

```text
{ $match: { $and: [ { "year" : 2014 }, { "status" : "A" } ] } }
```

### `$lookup` + `$unwind` 合并

_3.2版中的新功能。_

当a [`$unwind`](aggregation-pipeline-optimization.md)立即紧随其后 [`$lookup`](aggregation-pipeline-optimization.md)，并且在 领域[`$unwind`](aggregation-pipeline-optimization.md)运行时，优化程序可以将其合并 到阶段中。这样可以避免创建较大的中间文档。`as`[`$lookup`](aggregation-pipeline-optimization.md)[`$unwind`](aggregation-pipeline-optimization.md)[`$lookup`](aggregation-pipeline-optimization.md)

例如，管道包含以下序列：

```text
{
  $lookup: {
    from: "otherCollection",
    as: "resultingArray",
    localField: "x",
    foreignField: "y"
  }
},
{ $unwind: "$resultingArray"}
```

优化器可以将[`$unwind`](aggregation-pipeline-optimization.md)阶段合并为 [`$lookup`](aggregation-pipeline-optimization.md)阶段。如果使用`explain` 选项运行聚合，则`explain`输出将显示合并阶段：

```text
{
  $lookup: {
    from: "otherCollection",
    as: "resultingArray",
    localField: "x",
    foreignField: "y",
    unwinding: { preserveNullAndEmptyArrays: false }
  }
}
```

## 例子

### $limit $skip $limit $skip 序列

止于Mongodb4.0

管道包含一系列交替的[$limit](aggregation-pipeline-optimization.md)和[$skip](aggregation-pipeline-optimization.md)阶段：

```text
{ $limit: 100 },
{ $skip: 5 },
{ $limit: 10 },
{ $skip: 2 }
```

[$skip $limit 序列优化](aggregation-pipeline-optimization.md)反转`{ $skip: 5 }`和`{ $limit: 10 }`阶段的位置并增加限制量：

```text
{ $limit: 100 },
{ $limit: 15},
{ $skip: 5 },
{ $skip: 2 }
```

然后，优化器将两个[$limit](aggregation-pipeline-optimization.md)阶段合并为一个[$limit](aggregation-pipeline-optimization.md)阶段，将两个[$skip](aggregation-pipeline-optimization.md)阶段合并为一个[$skip](aggregation-pipeline-optimization.md)阶段。结果序列如下：

```text
{ $limit: 15 },
{ $skip: 7 }
```

有关详细信息，请参阅[$limit $limit 合并](aggregation-pipeline-optimization.md)和[$skip $skip 合并](aggregation-pipeline-optimization.md)。

> **\[success\] 可以看看**
>
> [db.collection.aggregate\(\)](aggregation-pipeline-optimization.md)中的[说明](aggregation-pipeline-optimization.md)选项

译者：李冠飞

校对：李冠飞

