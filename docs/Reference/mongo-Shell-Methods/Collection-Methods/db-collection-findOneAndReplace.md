# [ ](#)db.collection.findOneAndReplace（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `findOneAndReplace`(过滤，替换，选项)
   *   version 3.2 中的新内容。

根据`filter`和`sort`条件修改和替换单个文档。

findOneAndReplace()方法具有以下形式：

```powershell
db.collection.findOneAndReplace(
    <filter>,
    <replacement>,
    {
        projection: <document>,
        sort: <document>,
        maxTimeMS: <number>,
        upsert: <boolean>,
        returnNewDocument: <boolean>,
        collation: <document>
    }
)
```

findOneAndReplace()方法采用以下参数：

| 参数                | 类型     | 描述                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| `filter`            | document | 更新的选择标准。可以使用与find()方法相同的query selectors。 <br/>指定一个空文档`{ }`以替换集合中返回的第一个文档。 <br/>如果未指定，则默认为空文档。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果查询参数不是文档，则操作错误。 |
| `replacement`       | document | 替换文件。 <br/>不能包含更新 operators。 <br/> `<replacement>`文档无法指定与替换文档不同的`_id` value。 |
| `projection`        | document | 可选的。 return 的字段子集。 <br/>要_return 匹配文档中的所有字段，请省略此参数。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果投影参数不是文档，则操作错误。 |
| `sort`              | document | 可选的。为`filter`匹配的文档指定排序 order。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果 sort 参数不是文档，则操作错误。 <br/>见cursor.sort()。 |
| `maxTimeMS`         | number   | 可选的。指定操作必须在其中完成的 time 限制(以毫秒为单位)。如果超出限制则引发错误。 |
| `upsert`            | boolean  | 可选的。当`true`，findOneAndReplace()时：<br/>如果没有文档与`filter`匹配，则从`replacement`参数插入文档。插入新文档后返回`null`，除非`returnNewDocument`是`true`。 <br/>用`replacement`文档替换与`filter`匹配的文档。 <br/> MongoDB 将`_id`字段添加到替换文档中，如果未在`filter`或`replacement`文档中指定。如果两者都存在`_id`，则值必须相等。 <br/>要避免多次 upsert，请确保`query`字段为唯一索引。 <br/>默认为`false`。 |
| `returnNewDocument` | boolean  | 可选的。当`true`时，返回替换文档而不是原始文档。 <br/>默认为`false`。 |
| `collation`         | document | 可选的。 <br/>指定要用于操作的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 返回原始文档，如果是`returnNewDocument: true`，则返回替换文档。 |

## <span id="behavior">行为</span>

findOneAndReplace()替换集合中与`filter`匹配的第一个匹配文档。 `sort`参数可用于影响修改哪个文档。

### 投影

`projection`参数采用以下形式的文档：

```powershell
{ field1 : < boolean >, field2 : < boolean > ... }
```

`<boolean>` value 可以是以下任何一种：

*   `1`或`true`包括该字段。即使未在投影参数中明确说明，该方法也会返回`_id`字段。
*   `0`或`false`排除该字段。这可以在任何字段上使用，包括`_id`。

### 分片集合

要`db.collection.findOneAndReplace()`在分片集合上使用，查询过滤器必须在分片键上包含相等条件。

#### 碎片键修改

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

### 事务

`db.collection.findOneAndReplace()`可以在多文档交易中使用。

如果该操作导致upsert，则该集合必须已经存在。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

## <span id="examples">例子</span>

### 替换文档

`scores`集合包含类似于以下内容的文档：

```powershell
{ "_id" : 1521, "team" : "Fearful Mallards", "score" : 25000 },
{ "_id" : 2231, "team" : "Tactful Mooses", "score" : 23500 },
{ "_id" : 4511, "team" : "Aquatic Ponies", "score" : 19250 },
{ "_id" : 5331, "team" : "Cuddly Zebras", "score" : 15235 },
{ "_id" : 3412, "team" : "Garrulous Bears", "score" : 22300 }
```

以下操作查找`score`小于`20000`的第一个文档并替换它：

```powershell
db.scores.findOneAndReplace(
    { "score" : { $lt : 20000 } },
    { "team" : "Observant Badgers", "score" : 20000 }
)
```

该操作返回已替换的原始文档：

```powershell
{ "_id" : 2512, "team" : "Aquatic Ponies", "score" : 19250 }
```

如果`returnNewDocument`是 true，则操作将_return 替换文档。

### 排序和替换文档

`scores`集合包含类似于以下内容的文档：

```powershell
{ "_id" : 1521, "team" : "Fearful Mallards", "score" : 25000 },
{ "_id" : 2231, "team" : "Tactful Mooses", "score" : 23500 },
{ "_id" : 4511, "team" : "Aquatic Ponies", "score" : 19250 },
{ "_id" : 5331, "team" : "Cuddly Zebras", "score" : 15235 },
{ "_id" : 3412, "team" : "Garrulous Bears", "score" : 22300 }
```

按`score`排序会更改操作的结果。以下操作按`score`升序对`filter`的结果进行排序，并替换最低得分文档：

```powershell
db.scores.findOneAndReplace(
    { "score" : { $lt : 20000 } },
    { "team" : "Observant Badgers", "score" : 20000 },
    { sort: { "score" : 1 } }
)
```

该操作返回已替换的原始文档：

```powershell
{ "_id" : 5112, "team" : "Cuddly Zebras", "score" : 15235 }
```

有关此命令的 non-sorted 结果，请参见替换文档。

### 投射退回文件

`scores`集合包含类似于以下内容的文档：

```powershell
{ "_id" : 1521, "team" : "Fearful Mallards", "score" : 25000 },
{ "_id" : 2231, "team" : "Tactful Mooses", "score" : 23500 },
{ "_id" : 4511, "team" : "Aquatic Ponies", "score" : 19250 },
{ "_id" : 5331, "team" : "Cuddly Zebras", "score" : 15235 },
{ "_id" : 3412, "team" : "Garrulous Bears", "score" : 22300 }
```

以下操作使用 projection 仅显示返回文档中的`team`字段：

```powershell
db.scores.findOneAndReplace(
    { "score" : { $lt : 22250 } },
    { "team" : "Therapeutic Hamsters", "score" : 22250 },
    { sort : { "score" : 1 }, project: { "_id" : 0, "team" : 1 } }
)
```

该操作返回仅包含`team`字段的原始文档：

```powershell
{ "team" : "Aquatic Ponies"}
```

### 用 Time Limit 替换 Document

以下操作设置完成的 5ms time 限制：

```powershell
try {
    db.scores.findOneAndReplace(
        { "score" : { $gt : 25000 } },
        { "team" : "Emphatic Rhinos", "score" : 25010 },
        { maxTimeMS: 5 }
    );
} catch(e){
    print(e);
}
```

如果操作超过 time 限制，则返回：

```powershell
Error: findAndModifyFailed failed: { "ok" : 0, "errmsg" : "operation exceeded time limit", "code" : 50 }
```

### 用 Upsert 替换文档

如果没有匹配的`filter`，则以下操作使用`upsert`字段来插入替换文档：

```powershell
try {
    db.scores.findOneAndReplace(
        { "team" : "Fortified Lobsters" },
        { "_id" : 6019, "team" : "Fortified Lobsters" , "score" : 32000},
        { upsert : true, returnNewDocument: true }
    );
} catch (e){
    print(e);
}
```

该操作返回以下内容：

```powershell
{
    "_id" : 6019,
    "team" : "Fortified Lobsters",
    "score" : 32000
}
```

如果`returnNewDocument`是 false，则操作将返回`null`，因为 return 没有原始文档。

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
db.myColl.findOneAndReplace(
    { category: "cafe", status: "a" },
    { category: "cafÉ", status: "Replaced" },
    { collation: { locale: "fr", strength: 1 } }
);
```

该操作返回以下文档：

```powershell
{ "_id" : 1, "category" : "café", "status" : "A" }
```



译者：李冠飞

校对：