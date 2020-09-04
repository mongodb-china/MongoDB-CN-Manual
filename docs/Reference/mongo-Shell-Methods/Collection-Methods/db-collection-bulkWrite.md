# [ ](#)db.collection.bulkWrite（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.`  `bulkWrite` ()

       *   version 3.2 中的新内容。

使用 order 执行控件执行多个写操作。

bulkWrite()具有以下语法：

```powershell
db.collection.bulkWrite(
       [ <operation 1>, <operation 2>, ... ],
       {
          writeConcern : <document>,
          ordered : <boolean>
       }
    )
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `operations`   | array    | bulkWrite()写操作的 array。 <br/>    有效操作为：<br/>     insertOne <br/>     updateOne <br/>     updateMany <br/>     deleteOne <br/>     deleteMany <br/>     replaceOne <br/>    有关每个操作的使用情况，请参阅写操作。 |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。             |
| `ordered`      | boolean  | 可选的。一个 boolean，指定mongod实例是否应执行有序或无序操作执行。默认为`true`。 <br/>    参阅执行操作 |

| <br />   |                                                              |
| -------- | ------------------------------------------------------------ |
| 返回值： | 一个布尔值`acknowledged`，`true`好像该操作在运行时带有 写关注点，或者`false`禁用了写关注点。<br /> 每个写入操作的计数。<br /> 一个数组，其中包含`_id`每个成功插入或插入的文档的。 |

## <span id="behavior">行为</span>

bulkWrite()采用 array 写操作并执行每个操作。默认情况下，操作在 order 中执行。请参阅执行操作以控制写操作执行的 order。

### 写操作

#### insertOne

将单个文档插入集合中。

见db.collection.insertOne()。

```powershell
db.collection.bulkWrite( [
       { insertOne : { "document" : <document> } }
    ] )
```

#### updateOne 和 updateMany

更改 version 3.6：`updateOne`和`updateMany`操作添加了对`arrayFilters`参数的支持，该参数确定要在 array 字段中修改哪些元素。有关详细信息，请参阅db.collection.updateOne()和db.collection.updateMany()。

已更改 version 3.4：添加对整理的支持。有关详细信息，请参阅db.collection.updateOne()和db.collection.updateMany()

`updateOne`更新集合中与过滤器匹配的单个文档。如果多个文档 match，`updateOne`将仅更新第一个匹配的文档。见db.collection.updateOne()。

```powershell
db.collection.bulkWrite( [
       { updateOne :
          {
             "filter" : <document>,
             "update" : <document>,
             "upsert" : <boolean>,
             "collation": <document>,
             "arrayFilters": [ <filterdocument1>, ... ]
          }
       }
    ] )
```

`updateMany`更新集合中匹配过滤器的所有文档。见db.collection.updateMany()。

```powershell
db.collection.bulkWrite( [
       { updateMany :
          {
             "filter" : <document>,
             "update" : <document>,
             "upsert" : <boolean>,
             "collation": <document>,
             "arrayFilters": [ <filterdocument1>, ... ]
          }
       }
    ] )
```

| 字段           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| `filter`       | 更新的选择标准。提供与 方法中相同的查询选择器`db.collection.find()`。 |
| `update`       | 要执行的更新操作。可以指定：<br />仅包含更新运算符表达式的文档。<br />一个聚合管道 `[ <stage1>, <stage2>, ... ]`，指定要执行的修改。 |
| `upsert`       | 可选的。一个布尔值，指示是否执行upsert。<br />默认情况下`upsert`为`false`。 |
| `arrayFilters` | 可选的。筛选器文档数组，用于确定要对数组字段进行更新操作要修改的数组元素。 |
| `collation`    | 可选的。指定用于操作的排序规则。                             |
| `hint`         | 可选的。用于支持更新的索引`filter`。如果指定的索引不存在，则操作错误。<br />*4.2.1版中的新功能。* |

有关详细信息，请参见`db.collection.updateOne()`和 `db.collection.updateMany()`。

#### replaceOne

`replaceOne`替换与过滤器匹配的集合中的单个文档。如果多个文档 match，`replaceOne`将仅替换第一个匹配的文档。

```powershell
db.collection.bulkWrite([
       { replaceOne :
          {
             "filter" : <document>,
             "replacement" : <document>,
             "upsert" : <boolean>
          }
       }
    ] )
```

| 字段          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `filter`      | 替换操作的选择标准。提供与 方法中相同的 查询选择器`db.collection.find()`。 |
| `replacement` | 替换文件。该文档不能包含 更新运算符。                        |
| `upsert`      | 可选的。一个布尔值，指示是否执行upsert。默认情况下`upsert`为`false`。 |
| `collation`   | 可选的。指定用于操作的排序规则。                             |
| `hint`        | 可选的。用于支持更新的索引`filter`。如果指定的索引不存在，则操作错误。<br />*4.2.1版中的新功能。* |

有关详细信息，请参见`db.collection.replaceOne()`。

#### deleteOne 和 deleteMany

`deleteOne`删除集合中的一个文件 match 过滤器。如果多个文档 match，`deleteOne`将仅删除第一个匹配的文档。见`db.collection.deleteOne()`。

```powershell
db.collection.bulkWrite([
       { deleteOne :  { "filter" : <document> } }
    ] )
```

`deleteMany`删除集合中匹配过滤器的所有文档。见db.collection.deleteMany()。

```powershell
db.collection.bulkWrite([
       { deleteMany :  { "filter" : <document> } }
    ] )
```

| 字段        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| `filter`    | 删除操作的选择标准。提供与 方法中相同的 查询选择器`db.collection.find()`。 |
| `collation` | 可选的。指定用于操作的排序规则。                             |

有关详细信息，请参见`db.collection.deleteOne()`和 `db.collection.deleteMany()`。	

### _id 字段

如果文档未指定\_id字段，则mongod添加`_id`字段并在插入或插入文档之前为文档指定唯一的ObjectId。大多数驱动程序创建一个 ObjectId 并插入`_id`字段，但如果驱动程序或 application 没有，mongod将创建并填充`_id`。

如果文档包含`_id`字段，则`_id` value 在集合中必须是唯一的，以避免重复的 key 错误。

更新或替换操作不能指定与原始文档不同的`_id` value。

### 执行操作

`ordered`参数指定bulkWrite()是否将在 order 中执行操作。默认情况下，操作在 order 中执行。

以下 code 表示带有五个操作的bulkWrite()。

```powershell
db.collection.bulkWrite(
       [
          { insertOne : <document> },
          { updateOne : <document> },
          { updateMany : <document> },
          { replaceOne : <document> },
          { deleteOne : <document> },
          { deleteMany : <document> }
       ]
    )
```

在默认的`ordered : true` state 中，每个操作都将在 order 中执行，从第一个操作`insertOne`到最后一个操作`deleteMany`。

如果`ordered`设置为 false，则mongod可以重新排序操作以增加 performance。 Applications 不应该依赖于 order 操作执行。

以下 code 表示无序bulkWrite()，包含六个操作：

```powershell
db.collection.bulkWrite(
       [
          { insertOne : <document> },
          { updateOne : <document> },
          { updateMany : <document> },
          { replaceOne : <document> },
          { deleteOne : <document> },
          { deleteMany : <document> }
       ],
       { ordered : false }
    )
```

使用`ordered : false`时，操作结果可能会有所不同。对于 example，`deleteOne`或`deleteMany`可能会删除更多或更少的文档，具体取决于`insertOne`，`updateOne`，`updateMany`或`replaceOne`操作之前或之后的 run。

每个 group 中的操作数不能超过数据库maxWriteBatchSize的 value。从 MongoDB 3.6 开始，这个 value 是`100,000`。此值显示在isMaster.maxWriteBatchSize字段中。

此限制可防止出现超大错误消息的问题。如果 group 超过此`limit`，则 client 驱动程序将 group 分成较小的组，其计数小于或等于限制的 value。例如，对于`100,000`的`maxWriteBatchSize` value，如果队列包含`200,000`操作，则驱动程序将创建 2 个组，每个组具有`100,000`个操作。

> **注意**
>
> 使用 high-level API 时，驱动程序仅将 group 分为较小的组。如果直接使用db.runCommand()(对于 example，在编写驱动程序时)，MongoDB 在尝试执行超出限制的写入批处理时会抛出错误。

从 MongoDB 3.6 开始，一旦单个批处理的错误报告变得太大，MongoDB 会将所有剩余的错误消息截断为空的 string。目前，一旦至少有 2 个错误消息，总大小大于`1MB`，则开始。

尺寸和分组机械是内部性能细节，在将来的版本中可能会有所变化。

在分片集合上执行有序操作列表通常比执行无序列表慢，因为对于有序列表，每个操作必须等待上一个操作完成。

### 上限收藏

bulkWrite()写操作在上限集合上使用时有限制。

如果`update`条件增加了要修改的文档的大小，则`updateOne`和`updateMany`抛出`WriteError`。

如果`replacement`文档的大小比原始文档大，则`replaceOne`抛出`WriteError`。

如果在上限集合中使用`deleteOne`和`deleteMany`则抛出`WriteError`。

### 错误处理

bulkWrite()会在错误上抛出`BulkWriteError` exception。

排除写关注错误，有序操作在发生错误后停止，而无序操作继续处理队列中任何剩余的写操作。

写入关注错误显示在`writeConcernErrors`字段中，而所有其他错误显示在`writeErrors`字段中。如果遇到错误，则显示成功写入操作的数量而不是插入的`_id`值。有序操作显示遇到的单个错误，而无序操作显示 array 中的每个错误。

### 事务

`db.collection.bulkWrite()`可以在多文档事务中使用。

如果在事务中运行，则集合必须已经存在才能进行插入和操作。`upsert: true`

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

#### 事务里面的错误处理

从MongoDB 4.2开始，如果`db.collection.bulkWrite()`操作在事务内部遇到错误，则该方法将引发BulkWriteException（与事务外部相同）。

在4.0中，如果`bulkWrite`操作在事务内部遇到错误，则抛出的错误不会包装为 `BulkWriteException`。

在事务内部，即使批量写入是无序的，批量写入中的第一个错误也会导致整个批量写入失败并中止事务。

## <span id="examples">例子</span>

### 批量写操作

`guidebook`数据库中的`characters`集合包含以下文档：

```powershell
{ "_id" : 1, "char" : "Brisbane", "class" : "monk", "lvl" : 4 },
{ "_id" : 2, "char" : "Eldon", "class" : "alchemist", "lvl" : 3 },
{ "_id" : 3, "char" : "Meldane", "class" : "ranger", "lvl" : 3 }
```

以下bulkWrite()对集合执行多个操作：

```powershell
try {
       db.characters.bulkWrite([
          { insertOne: { "document": { "_id": 4, "char": "Dithras", "class": "barbarian", "lvl": 4 } } },
          { insertOne: { "document": { "_id": 5, "char": "Taeln", "class": "fighter", "lvl": 3 } } },
          { updateOne : {
             "filter" : { "char" : "Eldon" },
             "update" : { $set : { "status" : "Critical Injury" } }
          } },
          { deleteOne : { "filter" : { "char" : "Brisbane"} } },
          { replaceOne : {
             "filter" : { "char" : "Meldane" },
             "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl": 4 }
          } }
       ]);
    } catch (e) {
       print(e);
    }
```

该操作返回以下内容：

```powershell
{
       "acknowledged" : true,
       "deletedCount" : 1,
       "insertedCount" : 2,
       "matchedCount" : 2,
       "upsertedCount" : 0,
       "insertedIds" : {
          "0" : 4,
          "1" : 5
       },
       "upsertedIds" : {
          }
}
```

如果集合在执行批量写入之前包含带有`"_id" : 5"`的文档，则在执行批量写入时，将为第二个 insertOne 抛出以下重复的 key exception：

```powershell
BulkWriteError({
       "writeErrors" : [
          {
             "index" : 1,
             "code" : 11000,
             "errmsg" : "E11000 duplicate key error collection: guidebook.characters index: _id_ dup key: { : 5.0 }",
             "op" : {
                "_id" : 5,
                "char" : "Taeln",
                "class" : "fighter",
                "lvl" : 3
             }
          }
       ],
       "writeConcernErrors" : [ ],
       "nInserted" : 1,
       "nUpserted" : 0,
       "nMatched" : 0,
       "nModified" : 0,
       "nRemoved" : 0,
       "upserted" : [ ]
    })
```

由于`ordered`默认为 true，因此只有第一个操作成功完成。 rest 未执行。 尽管出现错误，使用`ordered : false`运行bulkWrite()将允许剩余的操作完成。

### 无序批量写入

`guidebook`数据库中的`characters`集合包含以下文档：

```powershell
{ "_id" : 1, "char" : "Brisbane", "class" : "monk", "lvl" : 4 },
{ "_id" : 2, "char" : "Eldon", "class" : "alchemist", "lvl" : 3 },
{ "_id" : 3, "char" : "Meldane", "class" : "ranger", "lvl" : 3 }
```

以下bulkWrite()对`characters`集合执行多个`unordered`操作。请注意，其中一个`insertOne`阶段具有重复的`_id` value：

```powershell
try {
       db.characters.bulkWrite([
          { insertOne: { "document": { "_id": 4, "char": "Dithras", "class": "barbarian", "lvl": 4 } } },
          { insertOne: { "document": { "_id": 4, "char": "Taeln", "class": "fighter", "lvl": 3 } } },
          { updateOne : {
             "filter" : { "char" : "Eldon" },
             "update" : { $set : { "status" : "Critical Injury" } }
          } },
          { deleteOne : { "filter" : { "char" : "Brisbane"} } },
          { replaceOne : {
             "filter" : { "char" : "Meldane" },
             "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl": 4 }
          } }
       ], { ordered : false } );
    } catch (e) {
       print(e);
    }
```

该操作返回以下内容：

```powershell
BulkWriteError({
       "writeErrors" : [
          {
             "index" : 1,
             "code" : 11000,
             "errmsg" : "E11000 duplicate key error collection: guidebook.characters index: _id_ dup key: { : 4.0 }",
             "op" : {
                "_id" : 4,
                "char" : "Taeln",
                "class" : "fighter",
                "lvl" : 3
             }
          }
       ],
       "writeConcernErrors" : [ ],
       "nInserted" : 1,
       "nUpserted" : 0,
       "nMatched" : 2,
       "nModified" : 2,
       "nRemoved" : 1,
       "upserted" : [ ]
    })
```

由于这是`unordered`操作，因此尽管存在 exception，仍会处理队列中剩余的写入。

### 批量写与写关注

`enemies`集合包含以下文档：

```powershell
{ "_id" : 1, "char" : "goblin", "rating" : 1, "encounter" : 0.24 },
{ "_id" : 2, "char" : "hobgoblin", "rating" : 1.5, "encounter" : 0.30 },
{ "_id" : 3, "char" : "ogre", "rating" : 3, "encounter" : 0.2 },
{ "_id" : 4, "char" : "ogre berserker" , "rating" : 3.5, "encounter" : 0.12}
```

以下bulkWrite()使用100 毫秒写入关注值`"majority"`和超时值为对集合执行多个操作：

```powershell
try {
       db.enemies.bulkWrite(
          [
             { updateMany :
                {
                   "filter" : { "rating" : { $gte : 3} },
                   "update" : { $inc : { "encounter" : 0.1 } }
                },
         },
         { updateMany :
            {
               "filter" : { "rating" : { $lt : 2} },
               "update" : { $inc : { "encounter" : -0.25 } }
            },
         },
         { deleteMany : { "filter" : { "encounter": { $lt : 0 } } } },
         { insertOne :
            {
               "document" :
                  {
                     "_id" :5, "char" : "ogrekin" , "rating" : 2, "encounter" : 0.31
                  }
            }
         }
      ],
      { writeConcern : { w : "majority", wtimeout : 100 } }
   );
} catch (e) {
   print(e);
}
```

如果副本集中所有必需节点确认写入操作所需的总 time 大于`wtimeout`，则在`wtimeout`期间过后将显示以下`writeConcernError`。

```powershell
BulkWriteError({
       "writeErrors" : [ ],
       "writeConcernErrors" : [
          {
             "code" : 64,
             "codeName" : "WriteConcernFailed",
             "errInfo" : {
                "wtimeout" : true
             },
             "errmsg" : "waiting for replication timed out"
          },
          {
             "code" : 64,
             "codeName" : "WriteConcernFailed",
             "errInfo" : {
                "wtimeout" : true
             },
             "errmsg" : "waiting for replication timed out"
          },
          {
             "code" : 64,
             "codeName" : "WriteConcernFailed",
             "errInfo" : {
                "wtimeout" : true
             },
             "errmsg" : "waiting for replication timed out"
          }
       ],
       "nInserted" : 1,
       "nUpserted" : 0,
       "nMatched" : 4,
       "nModified" : 4,
       "nRemoved" : 1,
       "upserted" : [ ]
    })
```

结果集显示执行的操作，因为`writeConcernErrors`错误不是任何写操作失败的指示。



译者：李冠飞

校对：