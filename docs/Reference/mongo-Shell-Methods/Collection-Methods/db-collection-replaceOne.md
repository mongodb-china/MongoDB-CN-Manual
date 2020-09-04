# [ ](#)db.collection.replaceOne（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `replaceOne`(过滤，替换，选项)


*version 3.2 中的新内容。*

根据过滤器替换集合中的单个文档。

replaceOne()方法具有以下形式：

```powershell
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
```

replaceOne()方法采用以下参数：

| 参数           | 类型     | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| `filter`       | document | 更新的选择标准。可以使用与find()方法相同的query selectors。 <br/>指定一个空文档`{ }`以替换集合中返回的第一个文档。 |
| `replacement`  | document | 替换文件。 <br/>不能包含更新 operators。                     |
| `upsert`       | boolean  | 可选的。当`true`，replaceOne()时：<br/>如果没有文档与`filter`匹配，则从`replacement`参数插入文档。 <br/>将与`filter`匹配的文档替换为`replacement`文档。如果未在`filter`或`replacement`文档中指定<br/>`_id`如果`filter`或`replacement`文档中未指定。 MongoDB，则会将`_id`字段添加到替换文档中。如果两者都存在`_id`，则值必须相等。 <br/>要避免多次 upsert，请确保`query`字段为唯一索引。 <br/>默认为`false`。 |
| `writeConcern` | document | 可选的。表示写关注的文件。省略使用默认写入问题。<br />如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |
| `collation`    | document | 可选的。 <br/>指定要用于操作的排序规则。 <br/>排序规则允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>collation：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |
| `hint`         | document | 可选的。指定用于支持过滤器的索引的文档或字符串。<br />该选项可以采用索引规范文档或索引名称字符串。<br />如果指定的索引不存在，则操作错误。<br />有关示例，请参阅为replaceOne指定提示。<br />*4.2.1版中的新功能。* |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 包含以下内容的文档：<br/>一个布尔值`acknowledged`，就`true`好像该操作在运行时带有 写关注关系或`false`是否禁用了写关注关系<br/> `matchedCount`包含匹配文档数<br/> `modifiedCount`包含已修改文档数<br/> `upsertedId`包含`_id` for upserted 文档 |

## <span id="behaviors">行为</span>

replaceOne()使用`replacement`文档替换集合中与`filter`匹配的第一个匹配文档。

### `upsert`

如果`upsert: true`和`filter`没有文档匹配，则 `db.collection.replaceOne()`根据`replacement`文档创建一个新文档。   

如果在分片集合上指定`upsert: true`，则必须在`filter` 中包含完整的分片键。有关分片集合的其他 `db.collection.replaceOne()`行为，请参见分片集合。 

请参阅用Upsert替换。

### 上限收藏

如果替换操作更改了文档大小，则操作将失败。

### 分片集合

从MongoDB 4.2开始，首先`db.collection.replaceOne()`尝试使用查询过滤器定位单个分片。如果该操作无法通过查询过滤器定位到单个分片，则它将尝试以替换文档定位。

在早期版本中，该操作尝试使用替换文档作为目标。

如果替换分片集合中的文档，则替换文档必须包含分片键。附加要求适用于分片集合和分片 密钥修改上的更新。

#### `upsert`在分片集合上

从MongoDB 4.2开始，对于`db.collection.replaceOne()` 包含`upsert: true`分片集合并在分片集合上的操作，您必须在`filter`中包含完整的分片键。 

#### 碎片键修改

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

要用于`db.collection.replaceOne()`更新分片键：

- 您**必须**在运行`mongos`无论是在 事务或作为重试写。千万**不能**直接在碎片颁发运行。
- 您**必须**在查询过滤器的完整分片键上包含相等条件。例如，如果一个集合`messages` 使用`{ country : 1, userid : 1 }`的片键，更新为一个文件的碎片关键，你必须包括`country: <value>, userid: <value>`在查询过滤器。您可以根据需要在查询中包括其他字段。 

### 事务

`db.collection.replaceOne()`可以在多文档交易中使用。

如果该操作导致upsert，则该集合必须已经存在。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

### 替换

`restaurant`集合包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan" },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 0 }
```

以下操作替换`name: "Central Perk Cafe"`所在的单个文档：

```powershell
try {
    db.restaurant.replaceOne(
        { "name" : "Central Perk Cafe" },
        { "name" : "Central Pork Cafe", "Borough" : "Manhattan" }
    );
} catch (e){
    print(e);
}
```

操作返回：

```powershell
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

如果未找到匹配项，则操作将返回：

```powershell
{ "acknowledged" : true, "matchedCount" : 0, "modifiedCount" : 0 }
```

如果未找到 match，则设置`upsert: true`将插入文档。见替换为 Upsert

### 替换为 Upsert

`restaurant`集合包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan",  "violations" : 3 },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 0 }
```

以下操作尝试使用`upsert : true`替换文档，使用`upsert : true`：

```powershell
try {
    db.restaurant.replaceOne(
        { "name" : "Pizza Rat's Pizzaria" },
        { "_id": 4, "name" : "Pizza Rat's Pizzaria", "Borough" : "Manhattan", "violations" : 8 },
        { upsert: true }
    );
} catch (e){
    print(e);
}
```

从`upsert : true`开始，文档是根据`replacement`文档插入的。操作返回：

```powershell
{
    "acknowledged" : true,
    "matchedCount" : 0,
    "modifiedCount" : 0,
    "upsertedId" : 4
}
```

该集合现在包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan", "violations" : 3 },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 0 },
{ "_id" : 4, "name" : "Pizza Rat's Pizzaria", "Borough" : "Manhattan", "violations" : 8 }
```

### 替换为写关注

给定三个成员副本集，以下操作指定`majority` `majority`和`wtimeout` `100`：

```powershell
try {
    db.restaurant.replaceOne(
        { "name" : "Pizza Rat's Pizzaria" },
        { "name" : "Pizza Rat's Pub", "Borough" : "Manhattan", "violations" : 3 },
        { w: "majority", wtimeout: 100 }
    );
} catch (e) {
    print(e);
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
db.myColl.replaceOne(
    { category: "cafe", status: "a" },
    { category: "cafÉ", status: "Replaced" },
    { collation: { locale: "fr", strength: 1 } }
);
```

### 指定`hint`用于`replaceOne`

*4.2.1版中的新功能。*

`members`使用以下文档创建样本集合：

```powershell
db.members.insertMany([
   { "_id" : 1, "member" : "abc123", "status" : "P", "points" :  0,  "misc1" : null, "misc2" : null },
   { "_id" : 2, "member" : "xyz123", "status" : "A", "points" : 60,  "misc1" : "reminder: ping me at 100pts", "misc2" : "Some random comment" },
   { "_id" : 3, "member" : "lmn123", "status" : "P", "points" :  0,  "misc1" : null, "misc2" : null },
   { "_id" : 4, "member" : "pqr123", "status" : "D", "points" : 20,  "misc1" : "Deactivated", "misc2" : null },
   { "_id" : 5, "member" : "ijk123", "status" : "P", "points" :  0,  "misc1" : null, "misc2" : null },
   { "_id" : 6, "member" : "cde123", "status" : "A", "points" : 86,  "misc1" : "reminder: ping me at 100pts", "misc2" : "Some random comment" }
])
```

在集合上创建以下索引：

```powershell
db.members.createIndex( { status: 1 } )
db.members.createIndex( { points: 1 } )
```

以下更新操作明确暗示要使用索引：`{ status: 1 }`

> **注意**
>
> 如果指定的索引不存在，则操作错误。

```powershell
db.members.replaceOne(
   { "points": { $lte: 20 }, "status": "P" },
   { "misc1": "using index on status", status: "P", member: "replacement", points: "20"},
   { hint: { status: 1 } }
)
```

该操作返回以下内容：

```powershell
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

要查看使用的索引，可以使用`$indexStats`管道：

```powershell
db.members.aggregate( [ { $indexStats: { } }, { $sort: { name: 1 } } ] )
```



译者：李冠飞

校对：