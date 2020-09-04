# [ ](#)db.collection.countDocuments（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

`db.collection.countDocuments`(query, options)

*版本4.0.3中的新功能。*

| 参数      | 类型     | 描述                                                         |
| --------- | -------- | ------------------------------------------------------------ |
| `query`   | document | 查询选择条件。要计算所有文档，请指定一个空文档。另请参阅查询限制。 |
| `options` | document | 可选的。影响计数行为的其他选项。                             |

该`options`文档可以包含以下内容：

| 字段        | 类型               | 描述                                   |
| ----------- | ------------------ | -------------------------------------- |
| `limit`     | integer            | 可选的。要计算的最大文件数。           |
| `skip`      | integer            | 可选的。计数前要跳过的文档数。         |
| `hint`      | string or document | 可选的。用于查询的索引名称或索引规范。 |
| `maxTimeMS` | integer            | 可选的。允许计数运行的最长时间。       |

## <span id="behavior">行为</span>

### 结构

与`db.collection.count()`， `db.collection.countDocuments()`不使用元数据返回计数不同。相反，它会执行文档的聚合以返回准确的计数，即使是在异常关闭后或分片群集中存在孤立的文档之后。

`db.collection.countDocuments()`包装以下聚合操作并仅返回的值`n`：

```powershell
db.collection.aggregate([
    { $match: <query> },
    { $group: { _id: null, n: { $sum: 1 } } }
])
```

### 空或不存在的集合和视图

从版本4.2.1（和版本4.0.13中的4.0系列）开始， `db.collection.countDocuments()`返回`0`在一个空的或不存在的集合或视图。

在MongoDB的早期版本中，`db.collection.countDocuments()`查询空或不存在的集合或视图会报错。

### 查询限制

您不能在`db.collection.countDocuments()`中将以下查询运算符用作以下查询表达式的一部分：

| 限制操作符    | 替代                                                         |
| ------------- | ------------------------------------------------------------ |
| `$where`      | 使用`$expr`代替                                              |
| `$near`       | `$geoWithin`与`$center`一起使用。即：<br />{ $geoWithin: { $center: [ [ &lt;x&gt;, &lt;y&gt; ], &lt;radius&gt; ] } } |
| `$nearSphere` | `$geoWithin`与`$centerSphere`一起使用。即：<br />{ $geoWithin: { $centerSphere: [ [ &lt;x&gt;, &lt;y&gt; ], &lt;radius&gt; ] } } |

### 事务

`db.collection.countDocuments()`可以在多文档事务中使用。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档事务的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

### 客户端断开

从MongoDB 4.2开始，如果发出`db.collection.countDocuments()`断开连接的客户端 在操作完成之前断开连接，则MongoDB将标记为终止`db.collection.countDocuments()`（即在操作上`killOp`）。

## <span id="examples">例子</span>

### 计算集合中的所有文档

要计算`orders`集合中所有文档的数量，请使用以下操作：

```powershell
db.orders.countDocuments({})
```

### 计算与查询匹配的所有文档

计算`orders` 集合中具有`ord_dt`大于的字段的文档数：`new Date('01/01/2012')`：

```powershell
db.orders.countDocuments( { ord_dt: { $gt: new Date('01/01/2012') } }, { limit: 100 } )
```

>也可以看看
>
>* `db.collection.estimatedDocumentCount()`
>* `$group` 和 `$sum`
>* `count`
>* 带有count选项的collStats pipeline stage。



译者：李冠飞

校对：