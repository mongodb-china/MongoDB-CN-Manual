# [ ](#)db.collection.insert（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

*   [写结果](#writeresult)

*   [BulkWriteResult](#bulkwriteresult)

## <span id="definition">定义</span>

*   `db.collection.`  `insert` ()

       *   将一个或多个文档插入集合中。

insert()方法具有以下语法：

```powershell
db.collection.insert(
       <document or array of documents>,
       {
         writeConcern: <document>,
         ordered: <boolean>
       }
    )
```

| 参数           | 类型           | 描述                                                         |
| -------------- | -------------- | ------------------------------------------------------------ |
| `document`     | 文件或 array   | 要插入集合的文档或 array 文档。                              |
| `writeConcern` | `writeConcern` | 可选的。表示写关注的文件。省略使用默认写入问题。见写关注。 <br/> version 2.6 中的新内容。 |
| `ordered`      | boolean        | 可选的。如果`true`，则在 array 中执行文档的有序插入，如果其中一个文档发生错误，MongoDB 将 return 而不处理 array 中的其余文档。 <br/>如果`false`，执行无序的 insert，如果其中一个文档发生错误，继续处理 array 中的其余文档。 <br/>默认为`true`。 |

更改 version 2.6：insert()返回包含操作状态的 object。

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 单个插入的写结果 object。 <br/> ABulkWriteResult object 用于批量插入。 |

## <span id="behaviors">行为</span>

### 写关注

insert()方法使用插入命令，该命令使用默认的写关注。要指定其他写入问题，请在 options 参数中包含写入关注点。

### 创建集合

如果集合不存在，则insert()方法将创建集合。
### _id 字段

如果文档未指定\_id字段，则 MongoDB 将添加`_id`字段，并在插入之前为文档指定唯一的ObjectId。大多数驱动程序创建一个 ObjectId 并插入`_id`字段，但如果驱动程序或 application 没有，则mongod将创建并填充`_id`。

如果文档包含`_id`字段，则`_id` value 在集合中必须是唯一的，以避免重复的 key 错误。

### 事务

`db.collection.insert()`可以在多文档交易中使用。

集合必须已经存在。事务中不允许执行会导致创建新集合的插入操作。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

以下示例将文档插入`products`集合。如果集合不存在，insert()方法将创建集合。
### 插入文档而不指定_id 字段

在以下 example 中，传递给insert()方法的文档不包含`_id`字段：

```powershell
db.products.insert( { item: "card", qty: 15 } )
```

在 insert 期间，mongod将创建`_id`字段并为其分配唯一的ObjectId value，由插入的文档验证：

```powershell
{ "_id" : ObjectId("5063114bd386d8fadbd6b004"), "item" : "card", "qty" : 15 }
```

当操作为 run 时，`ObjectId`值特定于机器和 time。因此，您的值可能与 example 中的值不同。

### 插入指定_id 字段的文档

在下面的示例中，传递给insert()方法的文档包含`_id`字段。 `_id`的 value 在集合中必须是唯一的，以避免重复的 key 错误。

```powershell
db.products.insert( { _id: 10, item: "box", qty: 20 } )
```

该操作在`products`集合中插入以下文档：

```powershell
{ "_id" : 10, "item" : "box", "qty" : 20 }
```

### 插入多个文档

以下 example 通过将 array 文档传递给insert()方法来执行三个文档的批量插入。默认情况下，MongoDB 执行有序的 insert。对于有序插入，如果在其中一个文档的 insert 期间发生错误，MongoDB 将返回错误而不处理 array 中的其余文档。

array 中的文档不需要具有相同的字段。例如，array 中的第一个文档有一个`_id`字段和一个`type`字段。由于第二个和第三个文档不包含`_id`字段，mongod将在 insert 期间为第二个和第三个文档创建`_id`字段：

```powershell
db.products.insert(
    [
        { _id: 11, item: "pencil", qty: 50, type: "no.2" },
        { item: "pen", qty: 20 },
        { item: "eraser", qty: 25 }
    ]
)
```

该操作插入了以下三个文件：

```powershell
{ "_id" : 11, "item" : "pencil", "qty" : 50, "type" : "no.2" }
{ "_id" : ObjectId("51e0373c6f35bd826f47e9a0"), "item" : "pen", "qty" : 20 }
{ "_id" : ObjectId("51e0373c6f35bd826f47e9a1"), "item" : "eraser", "qty" : 25 }
```

### 执行无序 Insert

以下 example 执行三个文档的无序插入。对于无序插入，如果在其中一个文档的 insert 期间发生错误，MongoDB 将继续插入 array 中的其余文档。

```powershell
db.products.insert(
    [
        { _id: 20, item: "lamp", qty: 50, type: "desk" },
        { _id: 21, item: "lamp", qty: 20, type: "floor" },
        { _id: 22, item: "bulk", qty: 100 }
    ],
    { ordered: false }
)
```

### 覆盖默认写入关注

对副本集的以下操作指定`"w: majority"`的`"w: majority"`，其`wtimeout`为 5000 毫秒，以便该方法在写入传播到大多数表决副本集成员后返回，或者该方法在 5 秒后超时。

在 version 3.0 中更改：在以前的版本中，`majority`指的是副本集的大多数成员而不是大多数投票成员。

```powershell
db.products.insert(
    { item: "envelopes", qty : 100, type: "Clasp" },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

## <span id="writeresult">写结果</span>

传递单个文档时，insert()返回`WriteResult` object。

### 成功的结果

insert()返回包含操作状态的写结果 object。成功后，写结果 object 包含有关插入文档数量的信息：

```powershell
WriteResult({ "nInserted" : 1 })
```

### 写关注错误

如果insert()方法遇到写入关注错误，则结果包括WriteResult.writeConcernError字段：

```powershell
WriteResult({
    "nInserted" : 1,
    "writeConcernError" : {
        "code" : 64,
        "errmsg" : "waiting for replication timed out at shard-a"
    }
})
```

### 与写关注无关的错误

如果insert()方法遇到 non-write 关注错误，则结果包括WriteResult.writeError字段：

```powershell
WriteResult({
    "nInserted" : 0,
    "writeError" : {
        "code" : 11000,
        "errmsg" : "insertDocument :: caused by :: 11000 E11000 duplicate key error index: test.foo.$_id_  dup key: { : 1.0 }"
    }
})
```

## <span id="bulkwriteresult">BulkWriteResult</span>

传递 array 文档时，insert()返回BulkWriteResult() object。有关详细信息，请参阅BulkWriteResult()。



译者：李冠飞

校对：