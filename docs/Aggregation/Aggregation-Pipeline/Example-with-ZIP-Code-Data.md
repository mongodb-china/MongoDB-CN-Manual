# [ ](#)使用 Zip Code 数据集进行聚合

[]()

在本页面

*   [数据模型 Data Model](#data-model)

*   [aggregate()方法](#aggregate-method)

*   [返回人口超过 1000 万的国家](#return-states-with-populations-above-10-million)

*   [按 State 返回平均城市人口](#return-average-city-population-by-state)

*   [按 State 返回最大和最小城市](#return-largest-and-smallest-cities-by-state)

本文档中的示例使用`zipcodes`集合。该系列可在以下网址获得：[media.mongodb.org/zips.json](http://media.mongodb.org/zips.json)。使用[mongoimport](../../docs/Reference/MongoDB-Package-Components/mongoimport.md)将此数据集加载到[mongod](../../docs/Reference/MongoDB-Package-Components/mongod.md)实例中。

[]()

## <span id="data-model">数据模型</span>

`zipcodes`集合中的每个文档都具有以下形式：

```powershell
{
  "_id": "10280",
  "city": "NEW YORK",
  "state": "NY",
  "pop": 5574,
  "loc": [
    -74.016323,
    40.710537
  ]
}
```

*   `_id`字段将 zip code 保存为 string。
*   `city`字段包含 city name。一个城市可以有多个与之关联的 zip code，因为城市的不同部分可以各自具有不同的 zip code。
*   `state`字段包含两个字母 state 缩写。
*   `pop`字段包含人口。
*   `loc`字段将位置保存为纬度经度对。

[]()

## <span id="aggregate-method">aggregate()方法</span>

以下所有示例都使用[mongo](reference-program-mongo.html#db.collection.aggregate) shell 中的[aggregate()](reference-method-db.collection.aggregate.html#bin.mongo)帮助程序。

[aggregate()](reference-method-db.collection.aggregate.html#db.collection.aggregate)方法使用[聚合管道](core-aggregation-pipeline.html#id1)将文档处理为聚合结果。 [聚合管道](core-aggregation-pipeline.html#id1)由[多个阶段](reference-operator-aggregation-pipeline.html#aggregation-pipeline-operator-reference)组成，每个阶段在文档沿着管道传递时都会对其进行处理。文档按顺序通过各个阶段。

[mongo](reference-program-mongo.html#db.collection.aggregate) shell 中的[aggregate()](reference-method-db.collection.aggregate.html#bin.mongo)方法在[聚合](reference-command-aggregate.html#dbcmd.aggregate)数据库命令提供了一个包装器。有关用于数据聚合操作的更惯用的界面，请参阅[驱动](https://docs.mongodb.com/ecosystem/drivers)的文档。

[]()

[]()

## <span id="return-states-with-populations-above-10-million">返回人口超过 1000 万的国家</span>

以下聚合操作将返回总人口超过 1000 万的所有州：

```powershell
db.zipcodes.aggregate( [
    { $group: { _id: “$state“, totalPop: { $sum: “$pop“ } } },
    { $match: { totalPop: { $gte: 10*1000*1000 } } }
] )
```

在此 example 中，[聚合管道](core-aggregation-pipeline.html#id1)包含[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段，后跟[$match](reference-operator-aggregation-match.html#pipe._S_match)阶段：

* 阶段按`state`字段对`zipcode`集合的文档进行分组，为每个 state 计算`totalPop`字段，并为每个唯一的 state 输出文档。

  新的 per-state 文档有两个字段：`_id`字段和`totalPop`字段。 `_id`字段包含`state`的 value即： group by field。 `totalPop`字段是一个计算字段，包含每个 state 的总人口。要计算 value，[$group](reference-operator-aggregation-group.html#pipe._S_group)使用[$sum](reference-operator-aggregation-sum.html#grp._S_sum) operator 为每个 state 添加填充字段(`pop`)。

  在[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段之后，管道中的文档类似于以下内容：

  ```powershell
  {
      “_id“ : “AK“,
      “totalPop“ : 550043
  }
  ```

*   [$match](reference-operator-aggregation-match.html#pipe._S_match)阶段过滤这些分组文档，仅输出`totalPop` value 大于或等于 1000 万的文档。 [$match](reference-operator-aggregation-match.html#pipe._S_match)阶段不会更改匹配的文档，但会不加修改地输出匹配的文档。

此聚合操作的等效[SQL](reference-glossary.html#term-sql)是：

```sql
SELECT state, SUM(pop) AS totalPop
    FROM zipcodes
    GROUP BY state
    HAVING totalPop >= (10*1000*1000)
```

> **[success] 也可以看看**
>
> [$group](reference-operator-aggregation-group.html#pipe._S_group)，[$match](reference-operator-aggregation-match.html#pipe._S_match)，[$sum](reference-operator-aggregation-sum.html#grp._S_sum)

[]()

## <span id="return-average-city-population-by-state">按 State 返回平均城市人口</span>

以下聚合操作返回每个 state 中城市的平均人口数：

```powershell
db.zipcodes.aggregate( [
    { $group: { _id: { state: “$state“, city: “$city“ }, pop: { $sum: “$pop“ } } },
    { $group: { _id: “$_id.state“, avgCityPop: { $avg: “$pop“ } } }
] )
```

在这个 example 中，[聚合管道](core-aggregation-pipeline.html#id1)包含[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段，后跟另一个[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段：

* 第一个阶段通过`city`和`state`的组合对文档进行分组，使用[$sum](reference-operator-aggregation-sum.html#grp._S_sum)表达式计算每个组合的总体，并为每个`city`和`state`组合输出一个文档。 [[1]](#multiple-zips-per-city)

  在管道中的这个阶段之后，文档类似于以下内容：

  ```powershell
  {
      “_id“ : {
          “state“ : “CO“,
          “city“ : “EDGEWATER“
      },
      “pop“ : 13154
  }
  ```

*   第二个[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段通过`_id.state`字段(i.e.`_id`文档中的`state`字段)对管道中的文档进行分组，使用[$avg](reference-operator-aggregation-avg.html#grp._S_avg)表达式计算每个 state 的平均城市人口(`avgCityPop`)，并为每个 state 输出一个文档。

此聚合操作产生的文档类似于以下内容：

```powershell
{
    “_id“ : “MN“,
    “avgCityPop“ : 5335
}
```

> **[success] 也可以看看**
>
> [$group](reference-operator-aggregation-group.html#pipe._S_group)，[$sum](reference-operator-aggregation-sum.html#grp._S_sum)，[$avg](reference-operator-aggregation-avg.html#grp._S_avg)

[]()

## <span id="return-largest-and-smallest-cities-by-state">按 State 返回最大和最小城市</span>

以下聚合操作按每个 state 的填充返回最小和最大的城市：

```powershell
db.zipcodes.aggregate( [
    { 
        $group:{
            _id: { state: “$state“, city: “$city“ },
            pop: { $sum: “$pop“ }
        }
    },
    { $sort: { pop: 1 } },
    { 
        $group:{
            _id : “$_id.state“,
            biggestCity:  { $last: “$_id.city“ },
            biggestPop:   { $last: “$pop“ },
            smallestCity: { $first: “$_id.city“ },
            smallestPop:  { $first: “$pop“ }
        }
    },
    // the following $project is optional, and
    // modifies the output format.
    { 
        $project:{ 
            _id: 0,
            state: “$_id“,
            biggestCity:  { name: “$biggestCity“,  pop: “$biggestPop“ },
            smallestCity: { name: “$smallestCity“, pop: “$smallestPop“ }
        }
    }
] )
```

在此 example 中，[聚合管道](core-aggregation-pipeline.html#id1)包含[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段，`$sort`阶段，另一个[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段和`$project`阶段：

* 第一个[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段通过`city`和`state`的组合对文档进行分组，计算每个组合的`pop`值的[和](reference-operator-aggregation-sum.html#grp._S_sum)，并为每个`city`和`state`组合输出一个文档。

  在管道的这个阶段，文档类似于以下内容：

  ```powershell
  {
      “_id“ : {
          “state“ : “CO“,
          “city“ : “EDGEWATER“
      },
      “pop“ : 13154
  }
  ```

* [$sort](reference-operator-aggregation-sort.html#pipe._S_sort)阶段通过`pop` field value 对管道中的文档进行排序，从最小到最大; 即：通过增加 order。此操作不会更改文档。

* 下一个[$group](reference-operator-aggregation-group.html#pipe._S_group)阶段按`_id.state`字段(即：`_id`文档中的`state`字段)对 now-sorted 文档进行分组，并为每个 state 输出一个文档。

  该阶段还为每个 state 计算以下四个字段。使用[$last](reference-operator-aggregation-last.html#grp._S_last)表达式，[$group](reference-operator-aggregation-group.html#pipe._S_group) operator 创建`biggestCity`和`biggestPop`字段，用于存储人口和人口最多的城市。使用[$first](reference-operator-aggregation-first.html#grp._S_first)表达式，[$group](reference-operator-aggregation-group.html#pipe._S_group) operator 创建`smallestCity`和`smallestPop`字段，用于存储人口和人口最少的城市。

  在管道的这个阶段，文件类似于以下内容：

  ```powershell
  {
      “_id“ : “WA“,
      “biggestCity“ : “SEATTLE“,
      “biggestPop“ : 520096,
      “smallestCity“ : “BENGE“,
      “smallestPop“ : 2
  }
  ```

*   最后的[$project](reference-operator-aggregation-project.html#pipe._S_project)阶段将`_id`字段重命名为`state`，并将`biggestCity`，`biggestPop`，`smallestCity`和`smallestPop`移动到`biggestCity`和`smallestCity`嵌入文档中。

此聚合操作的输出文档类似于以下内容：

```powershell
{
    “state“ : “RI“,
    “biggestCity“ : {
        “name“ : “CRANSTON“,
        “pop“ : 176404
    },
    “smallestCity“ : {
        “name“ : “CLAYVILLE“,
        “pop“ : 45
    }
}
```

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| [1]    | 一个城市可以有多个与之关联的 zip code，因为城市的不同部分可以各自具有不同的 zip code。 |



译者：李冠飞

校对：李冠飞