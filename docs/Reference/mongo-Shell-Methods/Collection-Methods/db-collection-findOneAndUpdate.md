# [ ](#)db.collection.findOneAndUpdate（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)


## <span id="definition">定义</span>

*   `db.collection.` `findOneAndUpdate`(过滤，更新，选项)

       *   version 3.2 中的新内容。

根据`filter`和`sort`条件更新单个文档。

findOneAndUpdate()方法具有以下形式：

更改了 version 3.6.

```powershell
db.collection.findOneAndUpdate(
   <filter>,
   <update>,
   {
     projection: <document>,
     sort: <document>,
     maxTimeMS: <number>,
     upsert: <boolean>,
     returnNewDocument: <boolean>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

findOneAndUpdate()方法采用以下参数：

| 参数                | 类型     | 描述                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| `filter`            | document | 更新的选择标准。可以使用与find()方法相同的query selectors。 <br/>指定一个空文档`{ }`以更新集合中返回的第一个文档。 <br/>如果未指定，则默认为空文档。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果查询参数不是文档，则操作错误。 |
| `update`            | document | 更新文件。 <br/>必须仅包含更新 operators。                   |
| `projection`        | document | 可选的。 return 的字段子集。 <br/>要_返回返回文档中的所有字段，请省略此参数。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果投影参数不是文档，则操作错误。 |
| `sort`              | document | 可选的。为`filter`匹配的文档指定排序 order。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果 sort 参数不是文档，则操作错误。 <br/>见cursor.sort()。 |
| `maxTimeMS`         | number   | 可选的。指定操作必须在其中完成的 time 限制(以毫秒为单位)。如果超出限制则引发错误。 |
| `upsert`            | boolean  | 可选的。当`true`，findOneAndUpdate()时：<br/>如果没有文件匹配`filter`，则创建一个新文档。有关详细信息，请参阅upsert 行为。插入新文档后返回`null`，除非`returnNewDocument`是`true`。 <br/>更新与`filter`匹配的单个文档。 <br/>要避免多次 upsert，请确保`filter`字段为唯一索引。 <br/>默认为`false`。 |
| `returnNewDocument` | boolean  | 可选的。当`true`时，返回更新的文档而不是原始文档。 <br/>默认为`false`。 |
| `collation`         | document | 可选的。 <br/>指定要用于操作的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |
| `arrayFilters`      | array    | 可选的。过滤器文档的 array，用于确定要在 array 字段上为更新操作修改哪些 array 元素。 <br/>在更新文档中，使用$ [&lt;identifier&gt;]过滤后的位置 operator 来定义标识符，然后在 array 过滤器文档中进行 reference。如果标识符未包含在更新文档中，则不能为标识符提供 array 过滤器文档。 <br/> **注意**<br/> `<identifier>`必须以小写字母开头，并且只包含字母数字字符。 <br/>您可以在更新文档中多次包含相同的标识符;但是，对于更新文档中的每个不同标识符(`$[identifier]`)，您必须指定**恰好一个**对应的 array 过滤器文档。也就是说，您不能为同一标识符指定多个 array 过滤器文档。对于 example，如果 update 语句包含标识符`x`(可能多次)，则不能为`arrayFilters`指定以下内容，其中包含 2 个单独的`x`过滤器文档：<br/> // INVALID<br/>  [<br/>   { "x.a": { $gt: 85 } },<br/>   { "x.b": { $gt: 80 } }<br/> ] <br/>但是，您可以在同一标识符上指定复合条件单个过滤器文档，例如以下示例：<br/> // Example 1<br/> [<br/>   { $or: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }<br/> ]<br/> // Example 2<br/> [<br/>   { $and: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }<br/> ]<br/> // Example 3<br/> [<br/>   { "x.a": { $gt: 85 }, "x.b": { $gt: 80 } }<br/> ]<br/>例如，请参阅为 Array Update Operations 指定 arrayFilters。 <br/> version 3.6 中的新内容。 |

| <br/>  |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 返回原始文档，如果是`returnNewDocument: true`，则返回更新的文档。 |

## <span id="behavior">行为</span>

findOneAndUpdate()更新集合中与`filter`匹配的第一个匹配文档。 `sort`参数可用于影响更新的文档。

### 投影

`projection`参数采用以下形式的文档：

```powershell
{ field1 : < boolean >, field2 : < boolean> ... }
```

`<boolean>` value 可以是以下任何一种：

*   `1`或`true`包括该字段。即使未在投影参数中明确说明，该方法也会返回`_id`字段。
*   `0`或`false`排除该字段。这可以在任何字段上使用，包括`_id`。

### 分片集合

要`db.collection.findOneAndUpdate()`在分片集合上使用，查询过滤器必须在分片键上包含相等条件。

#### 碎片键修改

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

要用于 `db.collection.findOneAndUpdate()`更新分片键：

- 您**必须**在运行`mongos`无论是在 事务或作为重试写。千万**不能**直接在碎片颁发运行。
- 您**必须**在查询过滤器的完整分片键上包含相等条件。例如，如果一个集合`messages` 使用的片键，更新为一个文件的碎片关键，你必须包括在查询过滤器。您可以根据需要在查询中包括其他字段。`{ country : 1, userid : 1 }``country: <value>, userid: <value>`

### 事务

`db.collection.findOneAndUpdate()`可以在多文档交易中使用。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

#### 现有的收藏和交易

在事务内部，您可以指定对现有集合的读/写操作。如果`db.collection.findOneAndUpdate()`导致upsert，则该集合必须已经存在。

如果该操作导致upsert，则该集合必须已经存在。

#### 写的担忧和事务

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

## <span id="examples">例子</span>

### 更新文档

`grades`集合包含类似于以下内容的文档：

```powershell
{ _id: 6305, name : "A. MacDyver", "assignment" : 5, "points" : 24 },
{ _id: 6308, name : "B. Batlock", "assignment" : 3, "points" : 22 },
{ _id: 6312, name : "M. Tagnum", "assignment" : 5, "points" : 30 },
{ _id: 6319, name : "R. Stiles", "assignment" : 2, "points" : 12 },
{ _id: 6322, name : "A. MacDyver", "assignment" : 2, "points" : 14 },
{ _id: 6234, name : "R. Stiles", "assignment" : 1, "points" : 10 }
```

以下操作查找`name : R. Stiles`的第一个文档，并按`5`递增得分：

```powershell
db.grades.findOneAndUpdate(
    { "name" : "R. Stiles" },
    { $inc: { "points" : 5 } }
)
```

该操作在更新之前返回原始文档：

```powershell
{ _id: 6319, name: "R. Stiles", "assignment" : 2, "points" : 12 }
```

如果`returnNewDocument`是 true，则操作将_return 更新文档。

### 排序和更新文档

`grades`集合包含类似于以下内容的文档：

```powershell
{ _id: 6305, name : "A. MacDyver", "assignment" : 5, "points" : 24 },
{ _id: 6308, name : "B. Batlock", "assignment" : 3, "points" : 22 },
{ _id: 6312, name : "M. Tagnum", "assignment" : 5, "points" : 30 },
{ _id: 6319, name : "R. Stiles", "assignment" : 2, "points" : 12 },
{ _id: 6322, name : "A. MacDyver", "assignment" : 2, "points" : 14 },
{ _id: 6234, name : "R. Stiles", "assignment" : 1, "points" : 10 }
```

以下操作更新`name : "A. MacDyver"`的文档。操作通过`points`升序对匹配文档进行排序，以更新具有最少点的匹配文档。

```powershell
db.grades.findOneAndUpdate(
    { "name" : "A. MacDyver" },
    { $inc : { "points" : 5 } },
    { sort : { "points" : 1 } }
)
```

该操作在更新之前返回原始文档：

```powershell
{ _id: 6322, name: "A. MacDyver", "assignment" : 2, "points" : 14 }
```

### 投射退回文件

以下操作使用 projection 仅显示返回文档中的`_id`，`points`和`assignment`字段：

```powershell
db.grades.findOneAndUpdate(
    { "name" : "A. MacDyver" },
    { $inc : { "points" : 5 } },
    { sort : { "points" : 1 }, projection: { "assignment" : 1, "points" : 1 } }
)
```

该操作仅返回原始文档，其中仅包含`projection`文档和`_id`字段中指定的字段，因为它未在投影文件中明确禁止(`_id: 0`)。

```powershell
{ "_id" : 6322, "assignment" : 2, "points" : 14 }
```

### 使用 Time 限制更新文档

以下操作设置 5ms time 限制以完成更新：

```powershell
try {
    db.grades.findOneAndUpdate(
        { "name" : "A. MacDyver" },
        { $inc : { "points" : 5 } },
        { sort: { "points" : 1 }, maxTimeMS : 5 };
    );
} catch(e){
    print(e);
}
```

如果操作超过 time 限制，则返回：

```powershell
Error: findAndModifyFailed failed: { "ok" : 0, "errmsg" : "operation exceeded time limit", "code" : 50 }
```

### 使用 Upsert 更新文档

如果没有匹配`filter`，则以下操作使用`upsert`字段来插入更新文档：

```powershell
try {
    db.grades.findOneAndUpdate(
        { "name" : "A.B. Abracus" },
        { $set: { "name" : "A.B. Abracus", "assignment" : 5}, $inc : { "points" : 5 } },
        { sort: { "points" : 1 }, upsert:true, returnNewDocument : true }
    );
} catch (e){
    print(e);
}
```

该操作返回以下内容：

```powershell
{
    "_id" : ObjectId("5789249f1c49e39a8adc479a"),
    "name" : "A.B. Abracus",
    "assignment" : 5,
    "points" : 5
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
db.myColl.findOneAndUpdate(
    { category: "cafe" },
    { $set: { status: "Updated" } },
    { collation: { locale: "fr", strength: 1 } }
);
```

该操作返回以下文档：

```powershell
{ "_id" : 1, "category" : "café", "status" : "A" }
```

### 为 Array Update Operations 指定 arrayFilters

version 3.6 中的新内容。

从 MongoDB 3.6 开始，在更新 array 字段时，您可以指定`arrayFilters`来确定要更新的 array 元素。

#### 更新元素 Match arrayFilters Criteria

使用以下文档创建集合`students`：

```powershell
db.students.insert([
    { "_id" : 1, "grades" : [ 95, 92, 90 ] },
    { "_id" : 2, "grades" : [ 98, 100, 102 ] },
    { "_id" : 3, "grades" : [ 95, 110, 100 ] }
])
```

要修改`grades` array 中大于或等于`100`的所有元素，请使用过滤后的位置 operator $ [&lt;identifier&gt;]和db.collection.findOneAndUpdate方法中的`arrayFilters`选项：

```powershell
db.students.findOneAndUpdate(
    { grades: { $gte: 100 } },
    { $set: { "grades.$[element]" : 100 } },
    { arrayFilters: [ { "element": { $gte: 100 } } ] }
)
```

该操作更新单个文档的`grades`字段，在操作之后，该集合具有以下文档：

```powershell
{ "_id" : 1, "grades" : [ 95, 92, 90 ] }
{ "_id" : 2, "grades" : [ 98, 100, 100 ] }
{ "_id" : 3, "grades" : [ 95, 110, 100 ] }
```

#### 更新 Array 文档的特定元素

使用以下文档创建集合`students2`：

```powershell
db.students2.insert([
    {
        "_id" : 1,
        "grades" : [
            { "grade" : 80, "mean" : 75, "std" : 6 },
            { "grade" : 85, "mean" : 90, "std" : 4 },
            { "grade" : 85, "mean" : 85, "std" : 6 }
        ]
    },
    {
        "_id" : 2,
        "grades" : [
            { "grade" : 90, "mean" : 75, "std" : 6 },
            { "grade" : 87, "mean" : 90, "std" : 3 },
            { "grade" : 85, "mean" : 85, "std" : 4 }
        ]
    }
])
```

要修改`grades` array 中等级大于或等于`85`的所有元素的`mean`字段的 value，请使用过滤后的位置 operator $ [&lt;identifier&gt;]和db.collection.findOneAndUpdate方法中的`arrayFilters`：

```powershell
db.students2.findOneAndUpdate(
    { },
    { $set: { "grades.$[elem].mean" : 100 } },
    { arrayFilters: [ { "elem.grade": { $gte: 85 } } ] }
)
```

该操作更新单个文档的`grades`字段，在操作之后，该集合具有以下文档：

```powershell
{
    "_id" : 1,
    "grades" : [
        { "grade" : 80, "mean" : 75, "std" : 6 },
        { "grade" : 85, "mean" : 100, "std" : 4 },
        { "grade" : 85, "mean" : 100, "std" : 6 }
    ]
}
{
    "_id" : 2,
    "grades" : [
        { "grade" : 90, "mean" : 75, "std" : 6 },
        { "grade" : 87, "mean" : 90, "std" : 3 },
        { "grade" : 85, "mean" : 85, "std" : 4 }
    ]
}
```

### 使用聚合管道进行更新

从MongoDB 4.2开始，`db.collection.findOneAndUpdate()`可以接受聚合管道进行更新。管道可以包括以下阶段：

- `$addFields`及其别名 `$set`
- `$project`及其别名 `$unset`
- `$replaceRoot`及其别名`$replaceWith`。

使用聚合管道可以实现更具表达力的更新语句，例如根据当前字段值表达条件更新，或使用另一个字段的值更新一个字段。

例如，`students2`使用以下文档创建一个集合：

```powershell
db.students2.insert([
   {
      "_id" : 1,
      "grades" : [
         { "grade" : 80, "mean" : 75, "std" : 6 },
         { "grade" : 85, "mean" : 90, "std" : 4 },
         { "grade" : 85, "mean" : 85, "std" : 6 }
      ]
   },
   {
      "_id" : 2,
      "grades" : [
         { "grade" : 90, "mean" : 75, "std" : 6 },
         { "grade" : 87, "mean" : 90, "std" : 3 },
         { "grade" : 85, "mean" : 85, "std" : 4 }
      ]
   }
])
```

以下操作将查找一个`_id`字段等于 的文档，`1`并使用聚合管道`total`从该`grades`字段中计算一个新 字段：

```powershell
db.students2.findOneAndUpdate(
   { _id : 1 },
   [ { $set: { "total" : { $sum: "$grades.grade" } } } ],  // The $set stage is an alias for ``$addFields`` stage
   { returnNewDocument: true }
)
```

> **注意**
>
> 该$set管道中的使用是指聚集阶段 $set，而不是更新操作$set。

该操作返回*更新的*文档：

```powershell
{
  "_id" : 1,
  "grades" : [ { "grade" : 80, "mean" : 75, "std" : 6 }, { "grade" : 85, "mean" : 90, "std" : 4 }, { "grade" : 85, "mean" :85, "std" : 6 } ],
  "total" : 250
}
```



译者：李冠飞

校对：