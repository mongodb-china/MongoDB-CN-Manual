# [ ](#)db.collection.insertMany（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.`  `insertMany` ()
   *   version 3.2 中的新内容。

将多个文档插入集合中。

insertMany()方法具有以下语法：

```powershell
db.collection.insertMany(
    [ <document 1> , <document 2>, ... ],
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `document`     | document | 要插入集合的 array 文档。                                    |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。<br />如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |
| `ordered`      | boolean  | 可选的。一个 boolean，指定mongod实例是否应该执行有序或无序的 insert。默认为`true`。 |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 一个文档包含：<br/> boolean `acknowledged` as `true`如果操作使用写关注或`false`运行如果写入关注被禁用<br/> 为`_id`每个成功插入的文档 |

## <span id="behaviors">行为</span>

给定 array 文档，insertMany()将 array 中的每个文档插入到集合中。

### 执行操作

默认情况下，文档插入 order。

如果`ordered`设置为 false，则文档以无序格式插入，并且可以通过mongod重新排序以增加 performance。如果使用无序insertMany()，Applications 不应该依赖于插入的排序。

每个 group 中的操作数不能超过数据库maxWriteBatchSize的 value。从 MongoDB 3.6 开始，这个 value 是`100,000`。此值显示在isMaster.maxWriteBatchSize字段中。

此限制可防止出现超大错误消息的问题。如果 group 超过此`limit`，则 client 驱动程序将 group 分成较小的组，其计数小于或等于限制的 value。例如，对于`100,000`的`maxWriteBatchSize` value，如果队列包含`200,000`操作，则驱动程序将创建 2 个组，每个组具有`100,000`个操作。

> **注意**
>
> 使用 high-level API 时，驱动程序仅将 group 分为较小的组。如果直接使用db.runCommand()(对于 example，在编写驱动程序时)，MongoDB 在尝试执行超出限制的写入批处理时会抛出错误。

从 MongoDB 3.6 开始，一旦单个批处理的错误报告变得太大，MongoDB 会将所有剩余的错误消息截断为空的 string。目前，一旦至少有 2 个错误消息，总大小大于`1MB`，则开始。

尺寸和分组机制是内部性能细节，在将来的版本中可能会有所变化。

在分片集合上执行有序操作列表通常比执行无序列表慢，因为对于有序列表，每个操作必须等待上一个操作完成。

### 集合创建

如果集合不存在，则insertMany()在成功写入时创建集合。

### _id 字段

如果文档未指定\_id字段，则mongod添加`_id`字段并为文档指定唯一的ObjectId。大多数驱动程序创建一个 ObjectId 并插入`_id`字段，但如果驱动程序或 application 没有创建，mongod将创建并填充`_id`。

如果文档包含`_id`字段，则`_id` value 在集合中必须是唯一的，以避免重复的 key 错误。

### Explainability

insertMany()与db.collection.explain()不兼容。

请改用insert()。

### 错误处理

插入抛出`BulkWriteError` exception。

排除写关注错误，有序操作在发生错误后停止，而无序操作继续处理队列中任何剩余的写操作。

写入关注错误显示在`writeConcernErrors`字段中，而所有其他错误显示在`writeErrors`字段中。如果遇到错误，则显示成功写入操作的数量，而不是插入的_id 列表。有序操作显示遇到的单个错误，而无序操作显示 array 中的每个错误。

### 事务

`db.collection.insertMany()`可以在多文档交易中使用。

集合必须已经存在。事务中不允许执行会导致创建新集合的插入操作。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

以下示例将文档插入`products`集合。

### 插入多个文档而不指定_id 字段

以下 example 使用db.collection.insertMany()来插入不包含`_id`字段的文档：

```powershell
try {
    db.products.insertMany( [
        { item: "card", qty: 15 },
        { item: "envelope", qty: 20 },
        { item: "stamps" , qty: 30 }
    ] );
} catch (e) {
    print (e);
}
```

该操作返回以下文档：

```powershell
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("562a94d381cb9f1cd6eb0e1a"),
        ObjectId("562a94d381cb9f1cd6eb0e1b"),
        ObjectId("562a94d381cb9f1cd6eb0e1c")
    ]
}
```

由于文档不包含`_id`，mongod为每个文档创建并添加`_id`字段，并为其分配唯一的ObjectId value。

当操作为 run 时，`ObjectId`值特定于机器和 time。因此，您的值可能与 example 中的值不同。

### Insert 若干文档指定_id 字段

以下 example/operation 使用insertMany()来插入包含`_id`字段的文档。 `_id`的 value 在集合中必须是唯一的，以避免重复的 key 错误。

```powershell
try {
    db.products.insertMany( [
        { _id: 10, item: "large box", qty: 20 },
        { _id: 11, item: "small box", qty: 55 },
        { _id: 12, item: "medium box", qty: 30 }
    ] );
} catch (e) {
    print (e);
}
```

该操作返回以下文档：

```powershell
{ "acknowledged" : true, "insertedIds" : [ 10, 11, 12 ] }
```

为的任何 key(例如`_id`)插入重复的 value 会抛出 exception。以下尝试使用已存在的`_id` value 插入文档：

```powershell
try {
    db.products.insertMany( [
        { _id: 13, item: "envelopes", qty: 60 },
        { _id: 13, item: "stamps", qty: 110 },
        { _id: 14, item: "packing tape", qty: 38 }
    ] );
} catch (e) {
    print (e);
}
```

由于`_id: 13`已存在，因此抛出以下 exception：

```powershell
BulkWriteError({
    "writeErrors" : [
        {
            "index" : 0,
            "code" : 11000,
            "errmsg" : "E11000 duplicate key error collection: inventory.products index: _id_ dup key: { : 13.0 }",
            "op" : {
                "_id" : 13,
                "item" : "stamps",
                "qty" : 110
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

请注意，插入了一个文档：`_id: 13`的第一个文档将成功插入，但第二个 insert 将失败。这也将阻止插入队列中剩余的其他文档。

使用`ordered`到`false`时，insert 操作将继续使用任何剩余文档。

### 无序插入

以下尝试使用`_id`字段和`ordered: false`插入多个文档。 array 文档包含两个具有重复`_id`字段的文档。

```powershell
try {
    db.products.insertMany( [
        { _id: 10, item: "large box", qty: 20 },
        { _id: 11, item: "small box", qty: 55 },
        { _id: 11, item: "medium box", qty: 30 },
        { _id: 12, item: "envelope", qty: 100},
        { _id: 13, item: "stamps", qty: 125 },
        { _id: 13, item: "tape", qty: 20},
        { _id: 14, item: "bubble wrap", qty: 30}
    ], { ordered: false } );
} catch (e) {
    print (e);
}
```

该操作抛出以下 exception：

```powershell
BulkWriteError({
    "writeErrors" : [
        {
            "index" : 2,
            "code" : 11000,
            "errmsg" : "E11000 duplicate key error collection: inventory.products index: _id_ dup key: { : 11.0 }",
            "op" : {
                "_id" : 11,
                "item" : "medium box",
                "qty" : 30
            }
        },
        {
            "index" : 5,
            "code" : 11000,
            "errmsg" : "E11000 duplicate key error collection: inventory.products index: _id_ dup key: { : 13.0 }",
            "op" : {
                "_id" : 13,
                "item" : "tape",
                "qty" : 20
            }
        }
    ],
    "writeConcernErrors" : [ ],
    "nInserted" : 5,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
```

由于重复的`_id`值，`item: "medium box"`和`item: "tape"`的文档无法插入`nInserted`表示插入了剩余的 5 个文档。

### 使用写关注

给定三个成员副本集，以下操作指定`majority` `majority`和`wtimeout` `100`：

```powershell
try {
    db.products.insertMany(
        [
            { _id: 10, item: "large box", qty: 20 },
            { _id: 11, item: "small box", qty: 55 },
            { _id: 12, item: "medium box", qty: 30 }
        ],
        { w: "majority", wtimeout: 100 }
    );
} catch (e) {
    print (e);
}
```

如果主要和至少一个辅助设备在 100 毫秒内确认每个写入操作，则返回：

```powershell
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("562a94d381cb9f1cd6eb0e1a"),
        ObjectId("562a94d381cb9f1cd6eb0e1b"),
        ObjectId("562a94d381cb9f1cd6eb0e1c")
    ]
}
```

如果副本集中所有必需节点确认写入操作所需的总 time 大于`wtimeout`，则在`wtimeout`期间过后将显示以下`writeConcernError`。

此操作返回：

```powershell
WriteConcernError({
    "code" : 64,
    "errInfo" : {
        "wtimeout" : true
    },
    "errmsg" : "waiting for replication timed out"
})
```



译者：李冠飞

校对：