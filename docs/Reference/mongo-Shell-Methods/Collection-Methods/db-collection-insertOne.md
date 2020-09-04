# [ ](#)db.collection.insertOne（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.`  `insertOne` ()
   *   version 3.2 中的新内容。

将文档插入集合中。

insertOne()方法具有以下语法：

```powershell
db.collection.insertOne(
    <document>,
    {
        writeConcern: <document>
    }
)
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `document`     | document | 要插入集合的文档。                                           |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。<br />如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 包含以下内容的文档：<br />一个布尔值`acknowledged`，`true`好像该操作在运行时带有 写关注点，或者`false`禁用了写关注点。<br />`insertedId`具有`_id`插入文档的值的字段。 |

## <span id="behaviors">行为</span>

### 集合创建

如果集合不存在，则insertOne()方法将创建集合。

### _id 字段

如果文档未指定\_id字段，则mongod将添加`_id`字段，并在插入之前为文档指定唯一的ObjectId。大多数驱动程序创建一个 ObjectId 并插入`_id`字段，但如果驱动程序或 application 没有，mongod将创建并填充`_id`。

如果文档包含`_id`字段，则`_id` value 在集合中必须是唯一的，以避免重复的 key 错误。

### Explainability

insertOne()与db.collection.explain()不兼容。

请改用insert()。

### 错误处理

出错时，insertOne()抛出`writeError`或`writeConcernError` exception。

### 事务

`db.collection.insertOne()`可以在多文档交易中使用。

集合必须已经存在。事务中不允许执行会导致创建新集合的插入操作。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

### 插入文档而不指定_id 字段

在以下 example 中，传递给insertOne()方法的文档不包含`_id`字段：

```powershell
try {
    db.products.insertOne( { item: "card", qty: 15 } );
} catch (e) {
    print (e);
};
```

该操作返回以下文档：

```powershell
{
    "acknowledged" : true,
    "insertedId" : ObjectId("56fc40f9d735c28df206d078")
}
```

由于文档不包含`_id`，mongod创建并添加`_id`字段并为其分配唯一的ObjectId value。

当操作为 run 时，`ObjectId`值特定于机器和 time。因此，您的值可能与 example 中的值不同。

### 插入指定_id 字段的文档

在下面的示例中，传递给insertOne()方法的文档包含`_id`字段。 `_id`的 value 在集合中必须是唯一的，以避免重复的 key 错误。

```powershell
try {
    db.products.insertOne( { _id: 10, item: "box", qty: 20 } );
} catch (e) {
    print (e);
}
```

该操作返回以下内容：

```powershell
{ "acknowledged" : true, "insertedId" : 10 }
```

为的任何 key 插入重复的 value，例如`_id`，会抛出 exception。以下尝试使用已存在的`_id` value 插入文档：

```powershell
try {
    db.products.insertOne( { _id: 10, "item" : "packing peanuts", "qty" : 200 } );
} catch (e) {
    print (e);
}
```

由于`_id: 10`已存在，因此抛出以下 exception：

```powershell
WriteError({
    "index" : 0,
    "code" : 11000,
    "errmsg" : "E11000 duplicate key error collection: inventory.products index: _id_ dup key: { : 10.0 }",
    "op" : {
        "_id" : 10,
        "item" : "packing peanuts",
        "qty" : 200
    }
})
```

### 增加写作关注

给定三个成员副本集，以下操作指定`majority` `wtimeout`，`wtimeout` `100`：

```powershell
try {
    db.products.insertOne(
        { "item": "envelopes", "qty": 100, type: "Self-Sealing" },
        { writeConcern: { w : "majority", wtimeout : 100 } }
    );
} catch (e) {
    print (e);
}
```

如果确认时间超过`wtimeout`限制，则抛出以下 exception：

```powershell
WriteConcernError({
    "code" : 64,
    "errInfo" : {
        "wtimeout" : true
    },
    "errmsg" : "waiting for replication timed out"
})
```

> **也可以看看**
>
> 要插入多个文档，请参阅db.collection.insertMany()



译者：李冠飞

校对：