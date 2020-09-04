# [ ](#)db.collection.deleteOne（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.`  `deleteOne` ()
   *   从集合中删除单个文档。

```powershell
db.collection.deleteOne(
    <filter>,
    {
        writeConcern: <document>,
        collation: <document>
    }
)
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `filter`       | document | 使用query operators指定删除条件。 <br/>    指定空文档`{ }`以删除集合中返回的第一个文档。 |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。             |
| `collation`    | document | 可选的。 <br/>    指定要用于操作的整理。 <br/>     整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>    排序规则选项具有以下语法：<br/>    排序规则：{<br/>     locale：&lt;string&gt;，<br/>     caseLevel：&lt;boolean&gt;，<br/>     caseFirst：&lt;string&gt;，<br/>     strength：&lt;int&gt;，<br/>     numericOrdering：&lt;boolean&gt;，<br/>     alternate：&lt;string&gt;，<br/>     maxVariable：&lt;string&gt;，<br/>     backwards ：&lt;boolean&gt; <br/>    } <br/>    指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>    如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>    如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>    您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/>     version 3.4 中的新内容。 |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 包含以下内容的文档：<br/>     boolean `acknowledged` as `true`如果操作使用写关注或`false`运行，如果写入关注被禁用<br/>     `deletedCount`包含已删除文档的数量 |

## <span id="behavior">行为</span>

### 删除 Order

deleteOne删除与过滤器匹配的第一个文档。使用属于独特的指数的字段(如`_id`)进行精确删除。

### 上限收藏

如果在上限集合上使用，deleteOne()会抛出`WriteError` exception。要从上限集合中删除文档，请改用db.collection.drop()。

### 事务

`db.collection.deleteOne()`可以在多文档事务中使用。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

### 删除单个文档

`orders`集合包含具有以下结构的文档：

```powershell
{
    _id: ObjectId("563237a41a4d68582c2509da"),
    stock: "Brent Crude Futures",
    qty: 250,
    type: "buy-limit",
    limit: 48.90,
    creationts: ISODate("2015-11-01T12:30:15Z"),
    expiryts: ISODate("2015-11-01T12:35:15Z"),
    client: "Crude Traders Inc."
}
```

以下操作使用`_id: ObjectId("563237a41a4d68582c2509da")`删除 order：

```powershell
try {
   db.orders.deleteOne( { "_id" : ObjectId("563237a41a4d68582c2509da") } );
} catch (e) {
   print(e);
}
```

操作返回：

```powershell
{ "acknowledged" : true, "deletedCount" : 1 }
```

以下操作删除`expiryts`大于`ISODate("2015-11-01T12:40:15Z")`的第一个文档

```powershell
try {
    db.orders.deleteOne( { "expiryts" : { $lt: ISODate("2015-11-01T12:40:15Z") } } );
} catch (e) {
    print(e);
}
```

操作返回：

```powershell
{ "acknowledged" : true, "deletedCount" : 1 }
```

### 写作关注 deleteOne()

给定三个成员副本集，以下操作指定`majority` `wtimeout`，`wtimeout` `100`：

```powershell
try {
    db.orders.deleteOne(
        { "_id" : ObjectId("563237a41a4d68582c2509da") },
        { w : "majority", wtimeout : 100 }
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



### 指定排序规则

version 3.4 中的新内容。

整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。

集合`myColl`具有以下文档：

```powershell
{ _id: 1, category: "café", status: "A" }
{ _id: 2, category: "cafe", status: "a" }
{ _id: 3, category: "cafE", status: "a" }
```

以下操作包括整理选项：

```powershell
db.myColl.deleteOne(
    { category: "cafe", status: "A" },
    { collation: { locale: "fr", strength: 1 } }
)
```

> **也可以看看**
>
> 要删除多个文档，请参阅db.collection.deleteMany()



译者：李冠飞

校对：