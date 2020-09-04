# [ ](#)db.collection.aggregate（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `aggregate`(管道，选项)

       *   计算集合中数据的聚合值或视图。

| 参数       | 类型     | 描述                                                         |
| ---------- | -------- | ------------------------------------------------------------ |
| `pipeline` | array    | 一系列数据聚合操作或阶段。有关详细信息，请参阅聚合管道运算符。 在 version 2.6 中更改：该方法仍然可以接受管道阶段作为单独的 arguments 而不是 array 中的元素;但是，如果未将`pipeline`指定为 array，则无法指定`options`参数。 |
| `options`  | document | 可选的。 aggregate()传递给aggregate命令的其他选项。  version 2.6 中的新内容：仅当您将`pipeline`指定为 array 时才可用。 |

`options`文档可以包含以下字段和值：

| 字段                       | 类型                 | 描述                                                         |
| -------------------------- | -------------------- | ------------------------------------------------------------ |
| `explain`                  | boolean              | 可选的。指定 return 有关管道处理的信息。有关 example，请参见返回有关聚合管道操作的信息。<br/>version 2.6 中的新内容。<br />在多文档交易中不可用。 |
| `allowDiskUse`             | boolean              | 可选的。允许写入临时文件。设置为时 `true`，大多数聚合操作可以将数据写入`_tmp`目录中的 `dbPath`子目录，但以下情况除外：<br />`$graphLookup`]阶段<br />`$addToSet`该`$group`阶段中使用的累加器表达式 （从4.2.3、4.0.14、3.6.17版本开始）<br />`$push`该`$group`阶段中使用的累加器表达式 （从4.2.3、4.0.14、3.6.17版本开始）<br />有关allowDiskUse的示例，请参见 使用外部排序执行大型排序操作。<br />从MongoDB 4.2开始，事件探查器日志消息和诊断日志消息包括一个`usedDisk` 指示符，指示是否有任何聚合阶段由于内存限制而将数据写入临时文件。 |
| `cursor`                   | document             | 可选的。指定游标的初始批处理大小。 `cursor`字段的 value 是一个带有`batchSize`字段的文档。有关语法和 example，请参阅指定初始批量大小。 <br/> version 2.6 中的新内容。 |
| `maxTimeMS`                | non-negative integer | 可选的。指定处理游标操作的 time 限制(以毫秒为单位)。如果没有为 maxTimeMS 指定 value，则操作不会 timeout。 `0`的 value 显式指定默认的无界行为。 <br/> MongoDB 使用与db.killOp()相同的机制终止超出其分配的 time 限制的操作。 MongoDB 仅在其指定的中断点之一处终止操作。 |
| `bypassDocumentValidation` | boolean              | 可选的。仅在指定$out或$merge]聚合阶段时可用。 <br/>在操作期间启用db.collection.aggregate以绕过文档验证。这使您可以插入不符合验证要求的文档。 <br/> version 3.2 中的新内容。 |
| `readConcern`              | document             | 可选的。指定读关注。 <br/> readConcern 选项具有以下语法：<br/>在 version 3.6 中更改。 <br/> `readConcern: { level: <value> }` <br/>可能的阅读关注级别为：<br/> “local”。这是 level 的默认读取问题。 <br/> “available”。当阅读操作和 Causally Consistent Sessions和“level”未指定时，这是对二级的读取的默认值。查询返回实例的最新数据。 <br/> “manority”。适用于使用WiredTiger 存储引擎的副本集。 <br/> “linerizable”。仅适用于主的读取操作。 <br/>有关读取关注级别的更多信息，请参阅读关注级别。 <br/>从MongoDB 4.2开始，该`$out`阶段不能与读取关注一起使用`"linearizable"`。也就是说，如果您为指定了`"linearizable"`读取关注 `db.collection.aggregate()`，则不能将`$out`阶段包括 在管道中。<br />该`$merge`阶段不能与已关注的内容一起使用`"linearizable"`。也就是说，如果您为指定了 `"linearizable"`读取关注 `db.collection.aggregate()`，则不能将`$merge`阶段包括 在管道中。 |
| `collation`                | document             | 可选的。 <br/>指定要用于操作的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection()，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |
| `hint`                     | string or document   | 可选的。用于聚合的索引。索引位于初始 collection/view，聚合为 run。 <br/>通过索引 name 或索引规范文档指定索引。 <br/> **注意** <br/> `hint`不适用于[$lookup]()和[$graphLookup]()阶段。 <br/> version 3.6 中的新内容。 |
| `comment`                  | string               | 可选的。用户可以指定任意 string 以帮助通过数据库探查器，currentOp 和日志跟踪操作。 <br/> version 3.6 中的新内容。 |
| `writeConcern`             | document             | 可选的。表示 与or 阶段一起使用的[写关注点](的文档。`$out` `$merge`<br />忽略对`$out`or `$merge`阶段使用默认的写关注。 |

| <br />   |                                                              |
| -------- | ------------------------------------------------------------ |
| 返回值： | 一个游标通过聚合管道操作的最后阶段产生的文件，或者包括 `explain`选项，提供了聚合操作的处理细节的文件。<br />如果管道包含`$out`运算符，则 `aggregate()`返回一个空游标。请参阅 `$out`以获取更多信息。 |

## <span id="behavior">行为</span>

### 错误处理

如果发生错误，aggregate()帮助程序将抛出 exception。

### 游标行为

在mongo shell 中，如果从db.collection.aggregate()返回的游标未使用`var`关键字分配给变量，则mongo shell 会自动迭代光标 20 次。请参阅在 mongo Shell 中迭代一个 Cursor以处理mongo shell 中的游标。

从聚合返回的游标仅支持对已评估的游标(已检索其第一批的，即：游标)进行操作的游标方法，例如以下方法：

| <br />                                                       |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| cursor.hasNext()<br/>cursor.next()<br/>cursor.toArray()<br/>cursor.forEach() | cursor.map()<br/>cursor.objsLeftInBatch()<br/>cursor.itcount()<br/>cursor.pretty() |

> **也可以看看**
>
> 有关更多信息，请参阅聚合管道，聚合参考，聚合管道限制和聚合。

### 会话

*版本4.0中的新功能。*

对于在会话内创建的游标，不能在`getMore`会话外调用 。

同样，对于在会话外部创建的游标，不能在`getMore`会话内部调用 。

#### 会话空闲超时

从MongoDB 3.6开始，MongoDB驱动程序和`mongo`shell程序将所有操作与服务器会话相关联，但未确认的写操作除外。对于未与会话明确关联的操作（即使用`Mongo.startSession()`），MongoDB驱动程序和`mongo`shell程序会创建一个隐式会话并将其与该操作相关联。

如果会话空闲时间超过30分钟，则MongoDB服务器会将会话标记为已过期，并可以随时关闭它。当MongoDB服务器关闭会话时，它还会终止所有正在进行的操作并打开与该会话关联的游标。这包括配置了30分钟`noCursorTimeout`或`maxTimeMS`30分钟以上的光标。

对于返回游标的操作，如果游标可能闲置了30分钟以上，请在显式会话中使用发出操作，`Session.startSession()`并使用`refreshSessions`命令定期刷新该会话。请参阅 以获取更多信息。`Session Idle Timeout`

### 事务

`db.collection.aggregate()`可以在多文档事务中使用。

但是，事务中不允许以下阶段：

- `$collStats`
- `$currentOp`
- `$indexStats`
- `$listLocalSessions`
- `$listSessions`
- `$out`
- `$merge`
- `$planCacheStats`

您也不能指定该`explain`选项。

- 对于在事务外部创建的游标，不能`getMore`在事务内部调用 。
- 对于在事务中创建的游标，不能`getMore`在事务外部调用 。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

### 客户端断开

对于`db.collection.aggregate()`不包含`$out`或`$merge`阶段的操作：

从MongoDB 4.2开始，如果发出`db.collection.aggregate()`断开连接的客户端在操作完成之前断开连接，则MongoDB将标记`db.collection.aggregate()`为终止（即在操作上`killOp`）。

## <span id="examples">例子</span>

以下示例使用包含以下文档的集合`orders`：

```powershell
{ _id: 1, cust_id: "abc1", ord_date: ISODate("2012-11-02T17:04:11.102Z"), status: "A", amount: 50 }
{ _id: 2, cust_id: "xyz1", ord_date: ISODate("2013-10-01T17:04:11.102Z"), status: "A", amount: 100 }
{ _id: 3, cust_id: "xyz1", ord_date: ISODate("2013-10-12T17:04:11.102Z"), status: "D", amount: 25 }
{ _id: 4, cust_id: "xyz1", ord_date: ISODate("2013-10-11T17:04:11.102Z"), status: "D", amount: 125 }
{ _id: 5, cust_id: "abc1", ord_date: ISODate("2013-11-12T17:04:11.102Z"), status: "A", amount: 25 }
```

### 分组和计算总和

以下聚合操作选择状态等于`"A"`的文档，按`cust_id`字段对匹配文档进行分组，并从`amount`字段的总和计算每个`cust_id`字段的`total`，并按降序 order 中的`total`字段对结果进行排序：

```powershell
db.orders.aggregate([
    { $match: { status: "A" } },
    { $group: { _id: "$cust_id", total: { $sum: "$amount" } } },
    { $sort: { total: -1 } }
])
```


该操作返回带有以下文档的游标：

```powershell
{ "_id" : "xyz1", "total" : 100 }
{ "_id" : "abc1", "total" : 75 }
```


mongo shell 自动迭代返回的光标以打印结果。有关在mongo shell 中手动处理游标的信息，请参阅在 mongo Shell 中迭代一个 Cursor。

### <span id="example-aggregate-method-explain-option">返回有关聚合管道操作的信息</span>

以下聚合操作将选项`explain`设置为`true`以_return 有关聚合操作的信息。

```powershell
db.orders.aggregate(
    [
        { $match: { status: "A" } },
        { $group: { _id: "$cust_id", total: { $sum: "$amount" } } },
        { $sort: { total: -1 } }
    ],
    {
        explain: true
    }
)
```


该操作返回带有文档的游标，该文档包含有关聚合管道处理的详细信息。例如，除了其他细节之外，文档可以显示所使用的操作的索引(如果有的话)。 [1]如果`orders`集合是分片集合，则文档还将显示分片和合并操作之间的分工，以及目标查询，目标分片。

> **注意**
>
> `explain`输出文档的预期 readers 是人类，而不是机器，输出格式可能会在不同版本之间发生变化。

mongo shell 自动迭代返回的光标以打印结果。有关在mongo shell 中手动处理游标的信息，请参阅在 mongo Shell 中迭代一个 Cursor。

> [1]索引过滤器会影响所用索引的选择。有关详细信息，请参见索引过滤器。

### 使用外部排序执行大型排序操作

聚合管道阶段有最大 memory 使用限制。要处理大型数据集，请将`allowDiskUse`选项设置为`true`以启用将数据写入临时 files，如下面的示例所示：

```powershell
var results = db.stocks.aggregate(
    [
        { $project : { cusip: 1, date: 1, price: 1, _id: 0 } },
        { $sort : { cusip : 1, date: 1 } }
    ],
    {
        allowDiskUse: true
    }
)
```

从MongoDB 4.2开始，事件profiler log massages和diagnostic log massages包括一个`usedDisk` 指示符，指示是否有任何聚合阶段由于内存限制而将数据写入临时文件。

### 指定初始批量大小

要指定游标的初始批处理大小，请对`cursor`选项使用以下语法：

```powershell
cursor: { batchSize: <int> }
```


对于 example，以下聚合操作指定游标的初始批处理大小`0`：

```powershell
db.orders.aggregate(
    [
        { $match: { status: "A" } },
        { $group: { _id: "$cust_id", total: { $sum: "$amount" } } },
        { $sort: { total: -1 } },
        { $limit: 2 }
    ],
    {
        cursor: { batchSize: 0 }
    }
)
```

`A batchSize` `0`表示空的第一批，对于快速返回游标或失败消息而不执行重要的 server-side 工作非常有用。与其他 MongoDB 游标一样，将后续批量大小指定为OP_GET_MORE操作。

mongo shell 自动迭代返回的光标以打印结果。有关在mongo shell 中手动处理游标的信息，请参阅在 mongo Shell 中迭代一个 Cursor。

### 指定排序规则

version 3.4 中的新内容。

整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。

集合`myColl`具有以下文档：

```powershell
{ _id: 1, category: "café", status: "A" }
{ _id: 2, category: "cafe", status: "a" }
{ _id: 3, category: "cafE", status: "a" }
```


以下聚合操作包括整理选项：

```powershell
db.myColl.aggregate(
    [ { $match: { status: "A" } }, { $group: { _id: "$category", count: { $sum: 1 } } } ],
    { collation: { locale: "fr", strength: 1 } }
);
```

> **注意**
>
> 如果执行涉及多个视图的聚合(例如$lookup或$graphLookup)，则视图必须具有相同的整理。

有关归类字段的说明，请参阅整理文件。

### 提示索引

version 3.6 中的新内容。

使用以下文档创建集合`foodColl`：

```powershell
db.foodColl.insert([
    { _id: 1, category: "cake", type: "chocolate", qty: 10 },
    { _id: 2, category: "cake", type: "ice cream", qty: 25 },
    { _id: 3, category: "pie", type: "boston cream", qty: 20 },
    { _id: 4, category: "pie", type: "blueberry", qty: 15 }
])
```

创建以下索引：

```powershell
db.foodColl.createIndex( { qty: 1, type: 1 } );
db.foodColl.createIndex( { qty: 1, category: 1 } );
```


以下聚合操作包括强制使用指定索引的`hint`选项：

```powershell
db.foodColl.aggregate(
    [ { $sort: { qty: 1 }}, { $match: { category: "cake", qty: 10  } }, { $sort: { type: -1 } } ],
    { hint: { qty: 1, category: 1 } }
)
```


### 覆盖 readConcern

使用该`readConcern`选项可以指定操作的读取关注点。

您不能将`$out`或`$merge`阶段与阅读关注结合使用`"linearizable"`。也就是说，如果您为指定了`"linearizable"`读取关注 `db.collection.aggregate()`，则不能在管道中包括任何一个阶段。

对副本集的以下操作指定“ 读取关注点”，`"majority"`以读取已确认已写入大多数节点的数据的最新副本。

> **注意**
> 
> * 要使用“多数”的阅读关注 level，replica sets 必须使用WiredTiger 存储引擎并选举protocol version 1。从 MongoDB 3.6 开始，默认情况下启用对读取问题“多数”的支持。对于 MongoDB 3.6.1 - 3.6.x，您可以禁用读取关注“多数”。有关更多信息，请参阅禁用阅读关注多数。
>
> * 要确保单个线程可以读取自己的写入，请对副本集的主要使用“多数”读取关注和“多数”写入问题。
>
> * 要使用“多数”的阅读关注 level，您不能包含$out阶段。
>
> * 无论阅读关注 level 如何，节点上的最新数据可能无法反映系统中数据的最新 version。

```powershell
db.restaurants.aggregate(
    [ { $match: { rating: { $lt: 5 } } } ],
    { readConcern: { level: "majority" } }
)
```

### 指定 Comment

名为`movies`的集合包含格式如下的文档：

```powershell
{
    "_id" : ObjectId("599b3b54b8ffff5d1cd323d8"),
    "title" : "Jaws",
    "year" : 1975,
    "imdb" : "tt0073195"
}
```


以下聚合操作查找在 1995 年创建的影片，并包含`comment`选项以在`logs`，`db.system.profile`集合和`db.currentOp`中提供跟踪信息。

```powershell
db.movies.aggregate( [ { $match: { year : 1995 } } ], { comment : "match_all_movies_from_1995" } ).pretty()
```


在启用了性能分析的系统上，您可以查询`system.profile`集合以查看所有最近的类似聚合，如下所示：

```powershell
db.system.profile.find( { "command.aggregate": "movies", "command.comment" : "match_all_movies_from_1995" } ).sort( { ts : -1 } ).pretty()
```

这将以下列格式返回一组探查器结果：

```powershell
{
    "op" : "command",
    "ns" : "video.movies",
    "command" : {
        "aggregate" : "movies",
        "pipeline" : [
            {
                "$match" : {
                    "year" : 1995
                }
            }
        ],
        "comment" : "match_all_movies_from_1995",
        "cursor" : {
        },
        "$db" : "video"
    },
    ...
}
```

应用程序可以编码 order 中的任意信息，以便更轻松地跟踪或识别系统中的特定操作。例如，application 可能附加 string comment，其中包含 process ID，线程 ID，client 主机名和发出命令的用户。



译者：李冠飞

校对：