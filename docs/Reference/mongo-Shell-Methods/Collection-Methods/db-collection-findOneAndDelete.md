# [ ](#)db.collection.findOneAndDelete（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

## 定义

*   `db.collection.` `findOneAndDelete`(过滤器，选项)

       *   version 3.2 中的新内容。

根据`filter`和`sort`条件删除单个文档，返回已删除的文档。

findOneAndDelete()方法具有以下形式：

```powershell
db.collection.findOneAndDelete(
    <filter>,
    {
        projection: <document>,
        sort: <document>,
        maxTimeMS: <number>,
        collation: <document>
    }
)
```

findOneAndDelete()方法采用以下参数：

| 参数         | 类型     | 描述                                                         |
| ------------ | -------- | ------------------------------------------------------------ |
| `filter`     | document | 更新的选择标准。可以使用与find()方法相同的query selectors。 <br/>指定空文档`{ }`以删除集合中返回的第一个文档。 <br/>如果未指定，则默认为空文档。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果查询参数不是文档，则操作错误。 |
| `projection` | document | 可选的。 return 的字段子集。 <br/>要_返回返回文档中的所有字段，请省略此参数。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果投影参数不是文档，则操作错误。 |
| `sort`       | document | 可选的。为`filter`匹配的文档指定排序 order。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果 sort 参数不是文档，则操作错误。 <br/>见cursor.sort()。 |
| `maxTimeMS`  | number   | 可选的。指定操作必须在其中完成的 time 限制(以毫秒为单位)。如果超出限制则引发错误。 |
| `collation`  | document | 可选的。 <br/>指定要用于操作的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |

| <br /> |                    |
| ------ | ------------------ |
| 返回： | 返回已删除的文档。 |

## 行为

findOneAndDelete()删除集合中与`filter`匹配的第一个匹配文档。 `sort`参数可用于影响更新的文档。

### 投影

`projection`参数采用以下形式的文档：

```powershell
{ field1 : < boolean >, field2 : < boolean> ... }
```

`<boolean>` value 可以是以下任何一种：

*   `1`或`true`包括该字段。即使未在投影参数中明确说明，该方法也会返回`_id`字段。
*   `0`或`false`排除该字段。这可以在任何字段上使用，包括`_id`。

### 事务

`db.collection.findOneAndDelete()`可以在多文档交易中使用。

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

### 例子

### 删除文档

`grades`集合包含类似于以下内容的文档：

```powershell
{ _id: 6305, name : "A. MacDyver", "assignment" : 5, "points" : 24 },
{ _id: 6308, name : "B. Batlock", "assignment" : 3, "points" : 22 },
{ _id: 6312, name : "M. Tagnum", "assignment" : 5, "points" : 30 },
{ _id: 6319, name : "R. Stiles", "assignment" : 2, "points" : 12 },
{ _id: 6322, name : "A. MacDyver", "assignment" : 2, "points" : 14 },
{ _id: 6234, name : "R. Stiles", "assignment" : 1, "points" : 10 }
```

以下操作查找`name : M. Tagnum`并删除它的第一个文档：

```powershell
db.scores.findOneAndDelete(
    { "name" : "M. Tagnum" }
)
```

该操作返回已删除的原始文档：

```powershell
{ _id: 6312, name: "M. Tagnum", "assignment" : 5, "points" : 30 }
```

### 排序和删除文档

`grades`集合包含类似于以下内容的文档：

```powershell
{ _id: 6305, name : "A. MacDyver", "assignment" : 5, "points" : 24 },
{ _id: 6308, name : "B. Batlock", "assignment" : 3, "points" : 22 },
{ _id: 6312, name : "M. Tagnum", "assignment" : 5, "points" : 30 },
{ _id: 6319, name : "R. Stiles", "assignment" : 2, "points" : 12 },
{ _id: 6322, name : "A. MacDyver", "assignment" : 2, "points" : 14 },
{ _id: 6234, name : "R. Stiles", "assignment" : 1, "points" : 10 }
```

以下操作首先查找`name : "A. MacDyver"`所有文档。然后在删除具有最低点 value 的文档之前按`points`升序排序：

```powershell
db.scores.findOneAndDelete(
    { "name" : "A. MacDyver" },
    { sort : { "points" : 1 } }
)
```

该操作返回已删除的原始文档：

```powershell
{ _id: 6322, name: "A. MacDyver", "assignment" : 2, "points" : 14 }
```

### 投影已删除的文档

以下操作使用 projection 仅返回返回文档中的`_id`和`assignment`字段：

```powershell
db.scores.findOneAndDelete(
    { "name" : "A. MacDyver" },
    { sort : { "points" : 1 }, projection: { "assignment" : 1 } }
)
```

该操作返回包含`assignment`和`_id`字段的原始文档：

```powershell
{ _id: 6322, "assignment" : 2 }
```

### 使用 Time 限制更新文档

以下操作设置 5ms time 限制以完成删除：

```powershell
try {
    db.scores.findOneAndDelete(
        { "name" : "A. MacDyver" },
        { sort : { "points" : 1 }, maxTimeMS : 5 };
    );
} catch(e){
    print(e);
}
```

如果操作超过 time 限制，则返回：

```powershell
Error: findAndModifyFailed failed: { "ok" : 0, "errmsg" : "operation exceeded time limit", "code" : 50 }
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
db.myColl.findOneAndDelete(
    { category: "cafe", status: "a" },
    { collation: { locale: "fr", strength: 1 } }
);
```

该操作返回以下文档：

```powershell
{ "_id" : 1, "category" : "café", "status" : "A" }
```



译者：李冠飞

校对：