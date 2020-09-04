
# 改变流

在本页面  
- 可用性
- 监视集合/数据库/部署
- 打开变更流
- 修改变更流输出
- 查找完整文档以进行更新操作
- 恢复变更流
- 用例
- 访问控制
- 活动通知
- 比较 

3.6版的新功能。

变更流允许应用程序访问实时数据更改，而不会带来复杂性和拖延oplog的风险。应用程序可以使用变更流来订阅单个集合，数据库或整个部署中的所有数据变更，并立即对其做出反应。因为变更流使用聚合框架，所以应用程序还可以过滤特定的变更或随意转换通知。

## 可用性
变更流可用于副本集和分片群集。

### 存储引擎。

副本集和分片群集必须使用WiredTiger存储引擎。变更流还可用于采用MongoDB静态加密功能的部署 。

### 副本集协议版本。

副本集和分片群集必须使用副本集协议版本1（pv1）。

### 阅读关注“多数”启用。

从MongoDB 4.2开始，无论是否支持读关注，更改流都可用"majority"。也就是说，majority可以启用（默认）读取关注支持或禁用以使用更改流。

在MongoDB 4.0和更早版本中，更改流仅在"majority"启用了阅读关注支持后才可用（默认）。

## 监视集合/数据库/部署

您可以针对以下情况打开变更流：

|目标	|描述|
| ----| ----| 
|集合	|你可以打开一个变换流光标单个集合（除system收藏，或任何集合在 admin，local和config数据库）。  本页上的示例使用MongoDB驱动程序打开并使用单个集合的变更流游标。另请参见mongoshell方法 db.collection.watch()。| 

| 数据库	|在MongoDB中4.0开始，你可以打开一个变换流光标单个数据库（不包括admin，local和 config数据库）来监视更改其所有非系统集合。  有关MongoDB驱动程序方法，请参阅驱动程序文档。另请参见mongoshell方法 db.watch()。|

|部署	| 在MongoDB中4.0开始，你可以打开一个变换流光标部署（无论是副本集中或分片集群）来监视除了所有数据库中更改所有非系统集合admin，local和config。  有关MongoDB驱动程序方法，请参阅驱动程序文档。另请参见mongoshell方法 Mongo.watch()。|
变更流示例

此页面上的示例使用MongoDB驱动程序来说明如何打开集合的更改流游标以及如何使用变更流游标。

##打开变更流

要打开变更流：

对于副本集，您可以从任何数据承载成员发出开放变更流操作。
对于分片群集，必须从中发出开放更改流操作mongos。
以下示例打开一个集合的变更流，并在光标上进行迭代以检索变更流文档。 [1]

下面的Python示例假定您已连接到MongoDB副本集并访问了包含inventory集合的数据库。

```
cursor = db.inventory.watch()
document = next(cursor)
```

要从游标检索数据更改事件，请迭代更改流游标。有关变更流事件的信息，请参见变更事件。

与MongoDB部署的连接保持打开状态时，游标保持打开状态，直到发生以下情况之一：

- 游标已显式关闭。
- 发生无效事件。
- 如果部署是分片群集，则分片删除可能会导致打开的更改流游标关闭，并且关闭的更改流游标可能无法完全恢复。

注意

未封闭游标的生命周期取决于语言。

[1]	从MongoDB 4.0开始，您可以指定a startAtOperationTime 在特定时间点打开游标。如果指定的起点是过去的时间，则必须在操作日志的时间范围内。
修改变更流输出；

在配置变更流时，可以通过提供以下一个或多个以下管道阶段的数组来控制变更流的输出：

$addFields  
$match  
$project  
$replaceRoot  
$replaceWith （从MongoDB 4.2开始可用）  
$redact  
$set （从MongoDB 4.2开始可用）  
$unset （从MongoDB 4.2开始可用） 

```
pipeline = [
    {'$match': {'fullDocument.username': 'alice'}},
    {'$addFields': {'newField': 'this is an added field!'}}
]
cursor = db.inventory.watch(pipeline=pipeline)
document = next(cursor)
```

提示:

变更流事件文档的_id字段用作恢复令牌。不要使用管道来修改或删除更改流事件的_id字段。

从MongoDB 4.2开始，如果更改流聚合管道修改了事件的_id字段，则更改流将引发异常。

有关更改流响应文档格式的更多信息，请参见更改事件。

## 查找完整文档以进行更新操作

默认情况下，更改流仅在更新操作期间返回字段增量。但是，您可以配置变更流以返回更新文档的最新的多数提交版本。

要返回更新文档的最新多数批准版本，请传递full_document='updateLookup'到 db.collection.watch()方法。

在下面的示例中，所有更新操作通知都包含一个full_document字段，该字段表示 受更新操作影响的文档的当前版本。

```
cursor = db.inventory.watch(full_document='updateLookup')
document = next(cursor)
```

注意

如果有一个或多个修改更新的文件多数提交的操作之后更新操作，但之前的查找，返回完整的文档可以显著从文档的更新操作的时间有所不同。

但是，变更流文档中包含的增量始终正确描述了应用于该更改流事件的监视集合更改。

有关变更流响应文档格式的更多信息，请参见更改事件。

## 恢复变更流

通过在打开游标时将resume令牌指定为resumeAfter或 startAfter，可以恢复变更流 。

### resumeAfter用于变更流

通过resumeAfter在打开游标时将恢复令牌传递给特定事件，您可以在特定事件之后恢复更改流。对于恢复令牌，请使用更改流事件文档的 _id值。有关恢复令牌的更多信息，请参见恢复令牌。

重要

如果时间戳记是过去的，则oplog必须具有足够的历史记录来定位与令牌或时间戳记关联的操作。

resumeAfter在无效事件（例如，集合删除或重命名）关闭流之后，您不能用来恢复变更流。从MongoDB 4.2开始，您可以使用 startAfter在invalidate事件之后启动新的变更流。

您可以使用resume_after修饰符在恢复令牌中指定的操作之后恢复通知。该resume_after调节器将是必须解决的简历令牌的值，例如resume_token在下面的例子。

```
resume_token = cursor.resume_token
cursor = db.inventory.watch(resume_after=resume_token)
document = next(cursor)
```

### startAfter用于变更流

4.2版中的新功能。

您可以通过startAfter在打开游标时将恢复令牌传递到特定事件之后，开始新的变更流。与resumeAfter不同 ，startAfter可以 通过创建新的更改流在无效事件之后恢复通知。对于恢复令牌，请使用更改流事件文档的_id值。有关恢复令牌的更多信息，请参见恢复令牌。

重要

如果时间戳记是过去的，则oplog必须具有足够的历史记录来定位与令牌或时间戳记关联的操作。

恢复令牌

变更流事件文档的_id值用作恢复令牌：

```
{
   "_data" : <BinData|hex string>
}
```

恢复令牌_data类型取决于MongoDB版本，在某些情况下，取决于更改流打开/恢复时的功能兼容性版本（fcv）（即，fcv值的更改不会影响已打开的更改流的恢复令牌。 ）：

MongoDB版本	功能兼容版本	恢复令牌_data类型
MongoDB 4.2及更高版本	“ 4.2”或“ 4.0”	十六进制编码的字符串（v1）
MongoDB 4.0.7及更高版本	“ 4.0”或“ 3.6”	十六进制编码的字符串（v1）
MongoDB 4.0.6及更早版本	“ 4.0”	十六进制编码的字符串（v0）
MongoDB 4.0.6及更早版本	“ 3.6”	BinData
MongoDB 3.6	“ 3.6”	BinData

使用十六进制编码的字符串恢复令牌，您可以对恢复令牌进行比较和排序。

无论fcv值如何，4.0部署都可以使用BinData恢复令牌或十六进制字符串恢复令牌来恢复变更流。这样，4.0部署可以使用在3.6部署的集合中打开的变更流中的恢复令牌。

MongoDB版本中引入的新的恢复令牌格式不能被早期MongoDB版本使用。

提示

从MongoDB 4.2开始，如果变更流聚合管道修改了事件的_id字段，则变更流将引发异常。


## 用例


变更流可以使具有相关业务系统的体系结构受益，一旦数据变更能够持久，就可以通知下游系统。例如，更改流可以在实现提取，转换和加载（ETL）服务，跨平台同步，协作功能和通知服务时为开发人员节省时间。

## 访问控制


对于执行身份验证和授权的部署：

要针对特定集合打开变更流，应用程序必须具有对相应集合进行授予changeStream和 find操作的特权。

```
{  resource ： {  db ： < dbname > ， collection ： < collection >  }， actions ： [  “ find” ， “ changeStream”  ]  }
```

要在单个数据库上打开变更流，应用程序必须具有对数据库中所有非集合进行授予changeStream和 find操作的特权system。

```
{  resource ： {  db ： < dbname > ， collection ： “”  }， actions ： [  “ find” ， “ changeStream”  ]  }
```

要在整个部署上打开变更流，应用程序必须具有为部署中所有数据库的所有非集合授予权限changeStream并对其 find执行操作的特权system。

```
{  resource ： {  db ： “” ， collection ： “”  }， actions ： [  “ find” ， “ changeStream”  ]  }
```


## 事件通知


变更流仅通知已保留到副本集中大多数数据承载成员的数据更改。这样可以确保仅由多数情况下提交的更改触发通知，这些更改在故障情况下是持久的。

例如，考虑一个3成员副本集，该副本集具有针对primary打开的变更流游标。如果客户端发出插入操作，则该更改流仅在该插入一直存在于大多数承载数据的成员之后，才将数据更改通知应用程序。

如果操作与事务相关联，则更改事件文档包括 txnNumber和lsid。

## 比较

从MongoDB 4.2开始，更改流将使用simple二进制比较，除非提供了明确的排序规则。在早期版本中，在单个集合（db.collection.watch()）上打开的变更流将继承该集合的默认排序规则。

译者： wh
