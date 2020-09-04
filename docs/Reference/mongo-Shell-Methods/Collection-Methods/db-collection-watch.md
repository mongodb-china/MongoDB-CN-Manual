# [ ](#)db.collection.watch（）

[]()

在本页面

*   [定义](#definition)
*   [可用性](#availability)
    *   [部署方式](#deployment)
    *   [存储引擎](#storage-engine)
    *   [阅读关注`majority`支持](#read-concern-majority-support)
*   [行为](#behaviors)
    *   [可恢复](#resumability)
    
    *   [完整文档查找和更新操作](#full-document-lookup-of-update-operations)
*   [访问控制](#access-control)
*   [例子](#examples)
    *   [打开更改流](#open-a-change-stream)
    
    *   [使用完整文档更新查找更改流](#change-stream-with-full-document-update-lookup)
    
    *   [使用聚合管道过滤器更改流](#change-stream-with-aggregation-pipeline-filter)
    
    *   [恢复变更流](#resuming-a-change-stream)

## <span id="definition">定义</span>

* `db.collection.` `watch`(管道，选项)

  *仅适用于副本集和分片群集*

  在集合上打开改变流游标。

| 参数       | 类型     | 描述                                                         |
| ---------- | -------- | ------------------------------------------------------------ |
| `pipeline` | array    | 以下一个或多个聚合阶段的序列：<br/> `$match` <br/> `$project` <br/> `$addFields` <br/> `$replaceRoot` <br/>`$replaceWith`（从MongoDB 4.2开始可用）<br /> `$redact `<br/>`$set`（从MongoDB 4.2开始可用）<br />`$unset`（从MongoDB 4.2开始可用）<br />指定管道以过滤/修改变更事件输出。<br />从MongoDB 4.2开始，如果更改流聚合管道修改了事件的_id字段，则更改流将引发异常。 |
| `options`  | document | 可选的。修改watch()行为的其他选项。 <br/>如果未指定管道但传递`options`文档，则必须将空 array `[]`传递给`pipeline`参数。 |

`options`文档可以包含以下字段和值：

| 字段                   | 类型      | 描述                                                         |
| ---------------------- | --------- | ------------------------------------------------------------ |
| `resumeAfter`          | document  | 可选的。指示watch尝试在恢复令牌中指定的操作之后重新开始通知。 <br/>每个更改流 event 文档都包含一个恢复标记作为`_id`字段。传递 change event 文档的整个`_id`字段，该字段表示您要在之后恢复的操作。<br />`resumeAfter`与`startAfter`和 互斥`startAtOperationTime`。 |
| `startAfter`           | document  | 可选的。指示`watch`在恢复令牌中指定的操作之后尝试启动新的更改流。允许在事件无效后恢复通知。<br />每个变更流事件文档都包括一个恢复令牌作为 `_id`字段。传递更改事件文档的*整个* `_id`字段，该字段代表您要恢复的操作。<br />`startAfter`与`resumeAfter`和 互斥`startAtOperationTime`。<br />*4.2版中的新功能。* |
| `fullDocument`         | string    | 可选的。默认情况下，watch()返回由更新操作修改的字段的增量，而不是整个更新的文档。 <br/>设置`fullDocument`为“ `"updateLookup"`直接” `watch()`以查找更新文档的最新多数批准版本。 `watch()`返回一个`fullDocument`字段，其中包含除了`updateDescription`增量以外的文档查找。 |
| `batchSize`            | int       | 可选的。指定 MongoDB集群的每批响应中返回的最大更改次数。 <br/>具有与cursor.batchSize()的功能相同。 |
| `maxAwaitTimeMS`       | int       | 可选的。服务器等待新数据更改以在返回空批处理之前向更改流游标报告的最大时间(以毫秒为单位)。 <br/>默认为`1000`毫秒。 |
| `collation`            | document  | 可选的。传递排序规则文件为更改流游标指定排序。<br />从MongoDB 4.2开始，`simple`如果省略，默认为二进制比较。在早期版本中，在单个集合上打开的更改流将继承该集合的默认排序规则。 |
| `startAtOperationTime` | Timestamp | 可选的。变更流的起点。如果指定的起点是过去的时间，则必须在操作日志的时间范围内。要查看操作日志的时间范围，请参阅 `rs.printReplicationInfo()`。<br />`startAtOperationTime`与`resumeAfter` 和互斥`startAfter`。<br />*版本4.0中的新功能。* |

| <br />   |                                                              |
| -------- | ------------------------------------------------------------ |
| 返回值： | 一个游标是保持被打开，以MongoDB的部署的连接保持打开状态，并收集存在。有关变更事件文档的示例，请参见变更事件。 |

> **也可以看看**
> 
> `db.watch()` 和 `Mongo.watch()`

## <span id="availability">可用性</span>

### 部署

`db.collection.watch()`可用于副本集和分片群集部署：

- 对于副本集，您可以`db.collection.watch()`在任何数据承载成员上发行。
- 对于分片群集，必须`db.collection.watch()`在`mongos`实例上发出。

### 存储引擎

您只能`db.collection.watch()`与Wired Tiger存储引擎一起使用。

### 阅读关注`majority`支持

从MongoDB 4.2开始，无论是否支持读关注，更改流都可用`"majority"`。也就是说，`majority`可以启用（默认）读取关注支持或禁用 以使用更改流。

在MongoDB 4.0和更早版本中，更改流仅在`"majority"`启用了阅读关注支持后才可用（默认）。

## <span id="behaviors">行为</span>

*   db.collection.watch()仅通知持续存在于大多数 data-bearing 成员的数据更改。

*   改变流游标保持打开状态，直到出现以下情况之一：

    *   游标显式关闭。

    *   发生无效事件；例如：集合删除或重命名。

    *   与 MongoDB 部署的连接已关闭。

    *   如果部署是分片集群，则删除分片可能会导致打开更改流游标关闭，并且关闭的更改流游标可能无法完全恢复。


### <span id="resumability">可恢复</span>

与 MongoDB 驱动程序不同，mongo shell 在发生错误后不会自动尝试恢复更改流游标。 MongoDB 驱动程序尝试在某些错误后自动恢复更改流游标。

db.collection.watch()使用存储在 oplog 中的信息来生成更改 event 描述并生成与该操作关联的恢复标记。如果由传递给`resumeAfter`or `startAfter`选项的恢复令牌标识的操作已经从oplog中删除，`db.collection.watch()`则无法恢复更改流。

有关恢复更改流的更多信息，请参阅恢复变更流。

> **注意**
>
>*   `resumeAfter`在无效事件（例如，集合删除或重命名）关闭流之后，您不能用来恢复更改 流。从MongoDB 4.2开始，您可以使用 startAfter在invalidate事件之后启动新的更改流。
>*   如果部署是分片集群，则分片删除可能会导致打开的更改流游标关闭，并且关闭的更改流游标可能无法完全恢复。

**恢复令牌**

恢复令牌`_data`类型取决于MongoDB版本，在某些情况下，取决于更改流打开/恢复时的功能兼容性版本（fcv）（即，fcv值的更改不会影响已打开的更改流的恢复令牌。 ）：

| MongoDB版本             | 功能兼容版本   | 恢复令牌`_data`类型          |
| ----------------------- | -------------- | ---------------------------- |
| MongoDB 4.2及更高版本   | “ 4.2”或“ 4.0” | 十六进制编码的字符串（`v1`） |
| MongoDB 4.0.7及更高版本 | “ 4.0”或“ 3.6” | 十六进制编码的字符串（`v1`） |
| MongoDB 4.0.6及更早版本 | “ 4.0”         | 十六进制编码的字符串（`v0`） |
| MongoDB 4.0.6及更早版本 | “ 3.6”         | BinData                      |
| MongoDB 3.6             | “ 3.6”         | BinData                      |

使用十六进制编码的字符串恢复令牌，您可以对恢复令牌进行比较和排序。

无论fcv值如何，4.0部署都可以使用BinData恢复令牌或十六进制字符串恢复令牌来恢复更改流。这样，4.0部署可以使用在3.6部署的集合中打开的更改流中的恢复令牌。

MongoDB版本中引入的新的恢复令牌格式不能被早期MongoDB版本使用。

### <span id="full-document-lookup-of-update-operations">完整文档查找和更新操作</span>

默认情况下，更改流游标返回用于更新操作的特定字段更改/增量。您还可以配置更改流以查找并返回更改文档的当前多数提交版本。根据更新和查找之间可能发生的其他写入操作，返回的文档可能与更新时的文档有很大不同。

根据更新操作期间应用的更改数量和整个文档的大小，存在更新操作的更改事件文档的大小大于16MB BSON文档限制的风险。如果发生这种情况，服务器将关闭更改流游标并返回错误。

## <span id="access-control">访问控制</span>

使用访问控制运行时，用户必须对集合资源具有 `find`和`changeStream`特权操作。也就是说，用户必须具有授予以下特权的角色：

```powershell
{ resource: { db: <dbname>, collection: <collection> }, actions: [ "find", "changeStream" ] }
```

内置`read`角色提供适当的特权。

## <span id="examples">例子</span>

### <span id="open-a-change-stream">打开更改流</span>

以下操作将针对`data.sensors`集合打开更改流游标：

```powershell
watchCursor = db.getSiblingDB("data").sensors.watch()
```

迭代光标以检查新的 events。使用`cursor.isExhausted()`方法确保循环仅在更改流游标关闭且最新批次中没有 objects 时退出：

```powershell
while (!watchCursor.isExhausted()){
    if (watchCursor.hasNext()){
        watchCursor.next();
    }
}
```

有关更改流输出的完整文档，请参阅变更事件。

### <span id="change-stream-with-full-document-update-lookup">使用完整文档更新查找更改流</span>

设置`fullDocument`选项以`"updateLookup"`指示更改流游标查找与更新更改流事件相关联的文档的最新的多数提交版本。

以下操作使用`fullDocument : "updateLookup"`该选项针对集合 `data.sensors`打开更改流游标。

```powershell
watchCursor = db.getSiblingDB("data").sensors.watch(
    [],
    { fullDocument : "updateLookup" }
)
```

迭代光标以检查新的 events。使用`cursor.isExhausted()`方法确保循环仅在更改流游标关闭且最新批次中没有 objects 时退出：

```powershell
while (!watchCursor.isExhausted()){
    if (watchCursor.hasNext()){
        watchCursor.next();
    }
}
```

对于任何更新操作，change事件都会在`fullDocument`字段中返回文档查找的结果。

有关完整文档更新输出的示例，请参阅更改流更新事件。

有关更改流输出的完整文档，请参阅改变事件。

### <span id="change-stream-with-aggregation-pipeline-filter">使用聚合管道过滤器更改流</span>

> **注意**
>
> 从MongoDB 4.2开始，如果更改流聚合管道修改了事件的_id字段，则更改流将引发异常。

以下操作使用聚合管道打开针对`data.sensors`集合的更改流游标：

```powershell
watchCursor = db.getSiblingDB("data").sensors.watch(
    [
        { $match : {"operationType" : "insert" } }
    ]
)
```

迭代光标以检查新的事件。使用`cursor.isExhausted()`方法确保循环仅在更改流游标关闭且最新批次中没有 objects 时退出：

```powershell
while (!watchCursor.isExhausted()){
    if (watchCursor.hasNext()){
        watchCursor.next();
    }
}
```

更改流游标仅返回为`insert`的 change events。有关更改流输出的完整文档，请参阅变更事件。

### <span id="resuming-a-change-stream">恢复变更流</span>

更改流游标返回的每个文档都包含一个恢复标记作为`_id`字段。要恢复更改流，请将要恢复的更改事件的整个`_id`文档传递给watch()的`resumeAfter`或`startAfter`选项。

以下操作`data.sensors`使用恢复令牌恢复针对集合的更改流游标 。假设生成恢复令牌的操作尚未脱离集群的操作日志。

```powershell
let watchCursor = db.getSiblingDB("data").sensors.watch();
let firstChange;

while (!watchCursor.isExhausted()) {
    if (watchCursor.hasNext()) {
        firstChange = watchCursor.next();
        break;
    }
}

watchCursor.close();

let resumeToken = firstChange._id;

resumedWatchCursor = db.getSiblingDB("data").sensors.watch(
    [],
    { resumeAfter : resumeToken }
)
```

迭代光标以检查新的事件。使用`cursor.isExhausted()`方法确保循环仅在更改流游标关闭且最新批次中没有 objects 时退出：

```powershell
while (!resumedWatchCursor.isExhausted()){
    if (resumedWatchCursor.hasNext()){
        resumedWatchCursor.next();
    }
}
```

有关恢复更改流的完整文档，请参阅恢复变更流。



译者：李冠飞

校对：