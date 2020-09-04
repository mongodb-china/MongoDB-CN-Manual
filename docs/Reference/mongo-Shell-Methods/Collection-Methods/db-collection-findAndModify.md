# [ ](#)db.collection.findAndModify（）

[]()

在本页面

*   [定义](#definition)

*   [Return 数据](#return-data)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `findAndModify`(文件)

       *   修改并返回单个文档。默认情况下，返回的文档不包括对更新所做的修改。要通过对更新进行的修改来回显文档，请使用`new`选项。 findAndModify()方法是findAndModify命令周围的 shell 助手。

findAndModify()方法具有以下形式：

更改了 version 3.6.

```powershell
db.collection.findAndModify({
    query: <document>,
    sort: <document>,
    remove: <boolean>,
    update: <document>,
    new: <boolean>,
    fields: <document>,
    upsert: <boolean>,
    bypassDocumentValidation: <boolean>,
    writeConcern: <document>,
    collation: <document>,
    arrayFilters: [ <filterdocument1>, ... ]
});
```

db.collection.findAndModify()方法采用带有以下嵌入文档字段的文档参数：

| 参数                       | 类型     | 描述                                                         |
| -------------------------- | -------- | ------------------------------------------------------------ |
| `query`                    | document | 可选的。修改的选择标准。 `query`字段使用与db.collection.find()方法中使用的query selectors相同的query selectors。虽然查询可能匹配多个文档，但findAndModify() **只会选择一个文档来修改**。 <br/>如果未指定，则默认为空文档。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果查询参数不是文档，则操作错误。 |
| `sort`                     | document | 可选的。如果查询选择多个文档，则确定操作修改的文档。 findAndModify()修改此参数指定的 sort order 中的第一个文档。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果 sort 参数不是文档，则操作错误。 |
| `remove`                   | boolean  | 必须指定`remove`或`update`字段。删除`query`字段中指定的文档。将其设置为`true`以删除所选文档。默认值为`false`。 |
| `update`                   | document | 必须指定`remove`或`update`字段。执行所选文档的更新。 `update`字段使用相同的更新 operators或`field: value`规范来修改所选文档。 |
| `new`                      | boolean  | 可选的。当`true`时，返回修改后的文档而不是原始文档。 findAndModify()方法忽略`remove`操作的`new`选项。默认值为`false`。 |
| `fields`                   | document | 可选的。 return 的字段子集。 `fields`文档指定包含`1`的字段，如：`fields: { <field1>: 1, <field2>: 1, ... }`。见投影。 <br/>从 MongoDB 3.6.14(和 3.4.23)开始，如果 fields 参数不是文档，则操作错误。 |
| `upsert`                   | boolean  | 可选的。与`update`字段结合使用。 <br/>当`true`，findAndModify()时：<br/>如果没有文件匹配`query`，则创建一个新文档。有关详细信息，请参阅upsert 行为。 <br/>更新与`query`匹配的单个文档。 <br/>要避免多次 upsert，请确保`query`字段为唯一索引。 <br/>默认为`false`。 |
| `bypassDocumentValidation` | boolean  | 可选的。允许db.collection.findAndModify在操作期间绕过文档验证。这使您可以更新不符合验证要求的文档。 <br/> version 3.2 中的新内容。 |
| `writeConcern`             | document | 可选的。表示写关注的文件。省略使用默认写入问题。 <br/> version 3.2 中的新内容。 |
| `maxTimeMS`                | integer  | 可选的。指定处理操作的 time 限制(以毫秒为单位)。             |
| `collation`                | document | 可选的。 <br/>指定要用于操作的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |
| `arrayFilters`             | array    | 可选的。过滤器文档的 array，用于确定要在 array 字段上为更新操作修改哪些 array 元素。 <br/>在更新文档中，使用$ [&lt;identifier&gt;]过滤后的位置 operator 来定义标识符，然后在 array 过滤器文档中进行 reference。如果标识符未包含在更新文档中，则不能为标识符提供 array 过滤器文档。 <br/> **注意**<br/> `<identifier>`必须以小写字母开头，并且只包含字母数字字符。 <br/>您可以在更新文档中多次包含相同的标识符;但是，对于更新文档中的每个不同标识符(`$[identifier]`)，您必须指定**恰好一个**对应的 array 过滤器文档。也就是说，您不能为同一标识符指定多个 array 过滤器文档。对于 example，如果 update 语句包含标识符`x`(可能多次)，则不能为`arrayFilters`指定以下内容，其中包含 2 个单独的`x`过滤器文档：<br/>[<br/>   { "x.a": { $gt: 85 } },<br/>   { "x.b": { $gt: 80 } }<br/> ] <br/>但是，您可以在同一标识符上指定复合条件单个过滤器文档，例如以下示例：<br/>// Example 1<br/> [<br/>   { $or: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }<br/> ]<br/> // Example 2<br/> [<br/>   { $and: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }<br/> ]<br/> // Example 3<br/> [<br/>   { "x.a": { $gt: 85 }, "x.b": { $gt: 80 } }<br/> ]<br/>例如，请参阅为 Array Update Operations 指定 arrayFilters。 <br/> version 3.6 中的新内容。 |

## <span id="return-data">Return 数据</span>

对于删除操作，如果查询与文档匹配，findAndModify()将返回已删除的文档。如果查询未匹配要删除的文档，findAndModify()将返回`null`。

对于更新操作，findAndModify()返回以下之一：

*   如果未设置`new`参数或`false`：
    *   如果查询与文档匹配，则为 pre-modification 文档;
    *   否则，`null`。
*   如果`new`是`true`：
    *   修改后的文档，如果查询返回 match;
    *   插入的文档，如果`upsert: true`，没有文档与查询匹配;
    *   否则，`null`。

更改 version 3.0：在以前的版本中，如果更新，`sort`已指定，`upsert: true`，`new`选项未设置或`new: false`，db.collection.findAndModify()将返回空文档`{}`而不是`null`。

## <span id="behavior">行为</span>

### Upsert 和 Unique Index

当findAndModify()包含`upsert: true`选项**并且**查询 field(s)没有唯一索引时，该方法可以在某些情况下多次插入文档。

在下面的示例中，不存在 name `Andy`的文档，并且多个 clients 发出以下命令：

```powershell
db.people.findAndModify({
    query: { name: "Andy" },
    sort: { rating: 1 },
    update: { $inc: { score: 1 } },
    upsert: true
})
```

然后，如果这些 clients 的findAndModify()方法在任何命令启动`modify`阶段之前完成`query`阶段，**和**在`name`字段上没有唯一索引，则命令可以全部执行 upsert，创建多个重复文档。

要防止使用相同的 name 创建多个重复文档，请在`name`字段上创建独特的指数。有了这个唯一索引，多个方法将表现出以下行为之一：

*   正好一个findAndModify()成功插入一个新文档。
*   零个或多个findAndModify()方法更新新插入的文档。
*   零个或多个findAndModify()方法在尝试 Insert 具有相同 name 的文档时失败。如果由于`name`字段上的唯一索引约束违规而导致方法失败，则可以重试该方法。如果没有删除文档，则重试不应失败。

### Sharded Collections

在分片环境中使用findAndModify时，`query` **必须**包含针对分片集合的分片 cluster 的所有操作的碎片 key。

`findAndModify`针对*非分片*集合的`mongos`实例发出的操作正常运行。

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

### 文件验证

db.collection.findAndModify()方法添加了对`bypassDocumentValidation`选项的支持，该选项允许您在使用验证规则插入或更新集合中的文档时绕过文件验证。

### 与update方法的比较

更新文档时，findAndModify()和update()方法的操作方式不同：

*   默认情况下，两个操作都会修改单个文档。但是，带有`multi`选项的update()方法可以修改多个文档。
*   如果多个文档匹配更新条件，对于findAndModify()，您可以指定`sort`以提供对要更新的文档的某种控制措施。

使用update()方法的默认行为，您无法在多个文档 match 时指定要更新的单个文档。

* 默认情况下，findAndModify()返回文档的 pre-modified version。要获取更新的文档，请使用`new`选项。

  update()方法返回包含操作状态的写结果 object。要 return 更新的文档，请使用find()方法。但是，其他更新可能已在您的更新和文档检索之间修改了文档。此外，如果更新仅修改了单个文档但匹配了多个文档，则需要使用其他逻辑来标识更新的文档。

修改单个文档时，findAndModify()和update()方法都会自动更新文档。有关这些方法的操作的交互和顺序的更多详细信息，请参阅原子性和 Transactions。

### 事务

`db.collection.findAndModify()`可以在多文档交易中使用。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

#### 现有的收藏和交易

在事务内部，您可以指定对现有集合的读/写操作。如果`db.collection.findAndModify()`导致upsert，则该集合必须已经存在。

#### 写的担忧和事务

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

## <span id="examples">例子</span>

### 更新和 Return

以下方法更新并返回文档与查询条件匹配的人员集合中的现有文档：

```powershell
db.people.findAndModify({
    query: { name: "Tom", state: "active", rating: { $gt: 10 } },
    sort: { rating: 1 },
    update: { $inc: { score: 1 } }
})
```

此方法执行以下操作：

* `query`在`people`集合中查找`name`字段具有 value `Tom`，`state`字段具有 value `active`且`rating`字段具有 value `greater than` 10 的文档。

* `sort`以升序 order 命令查询结果。如果多个文档符合`query`条件，则该方法将选择修改此`sort`所订购的第一个文档。

* 更新`increments` `score`字段的 value 为 1。

*   该方法返回为此更新选择的原始(i.e.pre-modification)文档：

    ```powershell
    {
        "_id" : ObjectId("50f1e2c99beb36a0f45c6453"),
        "name" : "Tom",
        "state" : "active",
        "rating" : 100,
        "score" : 5
    }
    ```

要 return 修改的文档，请将`new:true`选项添加到方法中。

如果没有文档与`query`条件匹配，则该方法返回`null`。

### UPSERT

以下方法包括`update`选项的`upsert: true`选项，用于更新匹配的文档;如果不存在匹配的文档，则创建新文档：

```powershell
db.people.findAndModify({
    query: { name: "Gus", state: "active", rating: 100 },
    sort: { rating: 1 },
    update: { $inc: { score: 1 } },
    upsert: true
})
```

如果方法找到匹配的文档，则该方法执行更新。

如果方法**不**找到匹配的文档，则该方法创建一个新文档。因为该方法包含`sort`选项，所以它返回一个空文档`{ }`作为原始(pre-modification)文档：

```powershell
{ }
```

如果方法确实**不包含`sort`选项，则该方法返回`null`。

```powershell
null
```

### 返回新文档

以下方法包括`upsert: true`选项和`new:true`选项。该方法更新匹配的文档并返回更新的文档，或者，如果不存在匹配的文档，则插入文档并在`value`字段中返回新插入的文档。

在以下 example 中，`people`集合中的任何文档都不匹配`query`条件：

```powershell
db.people.findAndModify({
    query: { name: "Pascal", state: "active", rating: 25 },
    sort: { rating: 1 },
    update: { $inc: { score: 1 } },
    upsert: true,
    new: true
})
```

该方法返回新插入的文档：

```powershell
{
    "_id" : ObjectId("50f49ad6444c11ac2448a5d6"),
    "name" : "Pascal",
    "rating" : 25,
    "score" : 1,
    "state" : "active"
}
```

### 排序和删除

通过在`rating`字段上包含`sort`规范，以下 example 将从`people`集合中删除`state` value 为`active`且匹配文档中最低`rating`的单个文档：

```powershell
db.people.findAndModify(
    {
        query: { state: "active" },
        sort: { rating: 1 },
        remove: true
    }
)
```

该方法返回已删除的文档：

```powershell
{
    "_id" : ObjectId("52fba867ab5fdca1299674ad"),
    "name" : "XYZ123",
    "score" : 1,
    "state" : "active",
    "rating" : 3
}
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
db.myColl.findAndModify({
    query: { category: "cafe", status: "a" },
    sort: { category: 1 },
    update: { $set: { status: "Updated" } },
    collation: { locale: "fr", strength: 1 }
});
```

该操作返回以下文档：

```powershell
{ "_id" : 1, "category" : "café", "status" : "A" }
```

### 为 Array Update Operations 指定 arrayFilters

> **注意**
>
> `arrayFilters` 不适用于使用聚合管道的更新。

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

要修改`grades` array 中大于或等于`100`的所有元素，请使用过滤后的位置 operator $ [&lt;identifier&gt; ]和db.collection.findAndModify方法中的`arrayFilters`选项：

```powershell
db.students.findAndModify({
    query: { grades: { $gte: 100 } },
    update: { $set: { "grades.$[element]" : 100 } },
    arrayFilters: [ { "element": { $gte: 100 } } ]
})
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

要修改`grades` array 中等级大于或等于`85`的所有元素的`mean`字段的 value，请使用过滤后的位置 operator $ [&lt;identifier&gt; ]和db.collection.findAndModify方法中的`arrayFilters`：

```powershell
db.students2.findAndModify({
    query: { },
    update: { $set: { "grades.$[elem].mean" : 100 } },
    arrayFilters: [ { "elem.grade": { $gte: 85 } } ]
})
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

从MongoDB 4.2开始，`db.collection.findAndModify()`可以接受聚合管道进行更新。管道可以包括以下阶段：

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
db.students2.findAndModify( {
   query: {  "_id" : 1 },
   update: [ { $set: { "total" : { $sum: "$grades.grade" } } } ],  // The $set stage is an alias for ``$addFields`` stage
   new: true
} )
```

> **注意**
>
> $set管道中的使用是指聚集阶段 $set，而不是更新操作$set。

该操作返回*更新的*文档：

```powershell
{
   "_id" : 1,
   "grades" : [ { "grade" : 80, "mean" : 75, "std" : 6 }, { "grade" : 85, "mean" : 90, "std" : 4 }, { "grade" : 85, "mean" : 85, "std" : 6 } ],
   "total" : 250
}
```

> **也可以看看**
>
> 可线性化通过 findAndModify 读取



译者：李冠飞

校对：