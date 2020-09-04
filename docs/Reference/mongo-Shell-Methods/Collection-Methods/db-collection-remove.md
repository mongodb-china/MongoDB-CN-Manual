# [ ](#)db.collection.remove（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

*   [写结果](#writeresult)

## <span id="definition">定义</span>

*   `db.collection.`  `remove` ()

从集合中删除文档。

db.collection.remove()方法可以具有两种语法之一。 remove()方法可以采用查询文档和可选的`justOne` boolean：

```powershell
db.collection.remove(
    <query>,
    <justOne>
)
```

或者该方法可以采用查询文档和可选的删除选项文档：

version 2.6 中的新内容。

```powershell
db.collection.remove(
    <query>,
    {
        justOne: <boolean>,
        writeConcern: <document>,
        collation: <document>
    }
)    
```

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `query`        | document | 使用query operators指定删除条件。要删除集合中的所有文档，请传递空文档(`{}`)。 |
| `justOne`      | boolean  | 可选的。要将删除限制为仅一个文档，请设置为`true`。省略使用`false`的默认 value 并删除符合删除条件的所有文档。 |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。见写关注。 <br/>如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |
| `collation`    | document | 可选的。 <br/>指定要用于操作的排序规则。 <br/>排序规则允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>collation：{<br/>     locale：&lt;string&gt;，<br/>     caseLevel：&lt;boolean&gt;，<br/>     caseFirst：&lt;string&gt;，<br/>     strength：&lt;int&gt;，<br/>     numericOrdering：&lt;boolean&gt;，<br/>     alternate：&lt;string&gt;，<br/>     maxVariable：&lt;string&gt;，<br/>     backwards ：&lt;boolean&gt; <br/>    } <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/>version 3.4 中的新内容。 |

remove()返回包含操作状态的 object。

| <br /> |                               |
| ------ | ----------------------------- |
| 返回： | 包含操作状态的写结果 object。 |

## <span id="behaviors">行为</span>

### 写关注

remove()方法使用删除命令，该命令使用默认的写关注。要指定其他写入问题，请在 options 参数中包含写入关注点。

### 查询注意事项

默认情况下，remove()删除 match `query`表达式的所有文档。指定`justOne`选项以限制删除单个文档的操作。要删除按指定 order 排序的单个文档，请使用findAndModify()方法。

删除多个文档时，删除操作可能与对集合的其他读 and/or 写操作交错。

### 上限集合

您不能将remove()方法与上限集合一起使用。

### 分片集合

指定`justOne`选项的分片集合的所有remove()操作必须包含查询规范中的碎片 key或`_id`字段。 remove()操作在分片集合中指定`justOne`，不包含碎片 key或`_id`字段返回错误。

### 事务

`db.collection.remove()`可以在多文档事务中使用。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

以下是remove()方法的示例。

### 从集合中删除所有文档

要删除集合中的所有文档，请使用空查询文档`{}`调用去掉方法。以下操作将删除bios 系列中的所有文档：

```
db.bios.remove( { } )
```

此操作不等同于drop()方法。

要从集合中删除所有文档，使用drop()方法删除整个集合(包括索引)，然后重新创建集合并重建索引可能更有效。

### 删除符合的所有文档

要删除匹配删除条件的文档，请使用`<query>`参数调用remove()方法：

以下操作将从集合`products`中删除`qty`大于`20`的所有文档：

```powershell
db.products.remove( { qty: { $gt: 20 } } )
```

### 覆盖默认写入关注

对副本集的以下操作将删除集合`products`中`qty`大于`20`的所有文档，并指定`"w: majority"`的`"w: majority"`，其`wtimeout`为 5000 毫秒，以便该方法在写入传播到大多数表决副本集后返回成员或方法在 5 秒后超时。

```powershell
db.products.remove(
    { qty: { $gt: 20 } },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
)
```

### 删除匹配条件的单个文档

要删除匹配删除条件的第一个文档，请使用`query`条件调用去掉方法，并将`justOne`参数设置为`true`或`1`。

以下操作从集合`products`中删除第一个文档，其中`qty`大于`20`：

```powershell
db.products.remove( { qty: { $gt: 20 } }, true )
```

### 指定排序规则

version 3.4 中的新内容。

整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。

集合`myColl`具有以下文档：

```powershell
{ _id: 1, category: "cafe", status: "A" }
{ _id: 2, category: "cafe", status: "a" }
{ _id: 3, category: "cafE", status: "a" }
```

以下操作包括整理选项：

```powershell
db.myColl.remove(
    { category: "cafe", status: "A" },
    { collation: { locale: "fr", strength: 1 } }
)
```

## <span id="writeresult">写结果</span>

更改了 version 2.6.

### 成功的结果

remove()返回包含操作状态的写结果 object。成功后，写结果 object 包含有关删除的文档数量的信息：

```powershell
WriteResult({ "nRemoved" : 4 })
```

> **也可以看看**
>
> WriteResult.nRemoved

### 写下关注错误

如果remove()方法遇到写入关注错误，则结果包括WriteResult.writeConcernError字段：

```powershell
WriteResult({
    "nRemoved" : 21,
    "writeConcernError" : {
        "code" : 64,
        "errInfo" : {
            "wtimeout" : true
        },
        "errmsg" : "waiting for replication timed out"
    }
})
```

> **也可以看看**
>
> WriteResult.hasWriteConcernError()

### 与写关注无关的错误

如果remove()方法遇到 non-write 关注错误，则结果包括WriteResult.writeError字段：

```powershell
WriteResult({
    "nRemoved" : 0,
    "writeError" : {
        "code" : 2,
        "errmsg" : "unknown top level operator: $invalidFieldName"
    }
})
```

> **也可以看看**
>
> WriteResult.hasWriteError()



译者：李冠飞

校对：