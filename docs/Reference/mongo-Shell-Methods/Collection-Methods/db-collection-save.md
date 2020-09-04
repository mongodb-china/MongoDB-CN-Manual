# [ ](#)db.collection.save（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

*   [写结果](#writeresult)

## <span id="definition">定义</span>

*   `db.collection.`  `save` ()


更新现有的文献或插入新文档，具体取决于其`document`参数。

> **注意**
>
> MongoDB弃用该`db.collection.save()`方法。而是使用`db.collection.insertOne()`或 `db.collection.replaceOne()`代替。

save()方法具有以下形式：

```powershell
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `document`     | document | 要保存到集合的文档。                                         |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。见写关注。 <br/>如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |

该`save()`返回包含操作的状态的对象。

| <br /> |                               |
| ------ | ----------------------------- |
| 返回： | 包含操作状态的写结果 object。 |

## <span id="behaviors">行为</span>

### 写关注

save()方法使用插入或更新命令，它使用默认的写关注。要指定其他写关注点，请在 options 参数中包含写入关注点。

### 插入

如果文档不包含\_id字段，则save()方法 calls insert()方法。在操作期间，mongo shell 将创建ObjectId并将其分配给`_id`字段。

> **注意**
>
> 大多数 MongoDB 驱动程序 clients 将包含`_id`字段并在将 insert 操作发送到 MongoDB 之前生成`ObjectId`;但是，如果 client 发送没有`_id`字段的文档，则mongod将添加`_id`字段并生成`ObjectId`。

### 更新

如果文档包含\_id字段，则save()方法等效于upsert 选项设置为`true`且`_id`字段上的查询谓词的更新。

### 事务

`db.collection.save()`可以在多文档交易中使用。

如果该操作导致插入，则该集合必须已经存在。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

### 保存新文档而不指定_id 字段

在下面的示例中，save()方法执行 insert，因为传递给该方法的文档不包含`_id`字段：

```powershell
db.products.save( { item: "book", qty: 40 } )
```

在 insert 期间，shell 将创建具有唯一ObjectId value 的`_id`字段，由插入的文档验证：

```powershell
{ "_id" : ObjectId("50691737d386d8fadbd6b01d"), "item" : "book", "qty" : 40 }
```

当操作为 run 时，`ObjectId`值特定于机器和 time。因此，您的值可能与 example 中的值不同。

### 保存新文档指定_id 字段

在下面的示例中，save()使用`upsert:true`执行更新，因为文档包含`_id`字段：

```powershell
db.products.save( { _id: 100, item: "water", qty: 30 } )
```

由于`_id`字段包含集合中不存在的 value，因此更新操作会导致插入文档。这些操作的结果与带有 upsert 选项的 update()方法设置为`true`相同。

该操作导致`products`集合中的以下新文档：

```powershell
{ "_id" : 100, "item" : "water", "qty" : 30 }
```

### 替换现有文档

`products`集合包含以下文档：

```powershell
{ "_id" : 100, "item" : "water", "qty" : 30 }
```

save()方法使用`upsert:true`执行更新，因为文档包含`_id`字段：

```powershell
db.products.save( { _id : 100, item : "juice" } )
```

由于`_id`字段包含集合中存在的 value，因此操作会执行更新以替换文档并生成以下文档：

```powershell
{ "_id" : 100, "item" : "juice" }
```

### 覆盖默认写入关注

对副本集的以下操作指定`"w: majority"`的`"w: majority"`，其`wtimeout`为 5000 毫秒，以便该方法在写入传播到大多数表决副本集成员后返回，或者该方法在 5 秒后超时。

```powershell
db.products.save(
    { item: "envelopes", qty : 100, type: "Clasp" },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

## <span id="writeresult">写结果</span>

将`save()`返回一个`WriteResult`包含插入或更新操作的状态对象。有关详细信息，请参见WriteResult以获得插入信息，并 参见 WriteResult以获得更新信息。



译者：李冠飞

校对：