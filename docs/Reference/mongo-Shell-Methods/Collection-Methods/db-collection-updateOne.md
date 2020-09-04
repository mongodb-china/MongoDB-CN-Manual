# [ ](#)db.collection.updateOne（）

[]()

在本页面

*   [定义](#definition)
*   [语法](#syntax)
*   [访问控制](#access-control)
*   [行为](#behaviors)
*   [例子](#examples)

## <span id="definition">定义</span>

* `db.collection.` `updateOne`(过滤，更新，选项)

  version 3.2 中的新内容。

  根据过滤器更新集合中的单个文档。

## 语法

updateOne()方法具有以下形式：

```powershell
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

### 参数

`db.collection.updateOne()`方法采用以下参数：

| 参数           | 类型               | 描述                                                         |
| -------------- | ------------------ | ------------------------------------------------------------ |
| `filter`       | document           | 更新的选择标准。可以使用与find()方法相同的query selectors。 <br/>指定一个空文档`{ }`以更新集合中的所有文档。 |
| `update`       | document           | 要应用的修改。可以是以下之一： <br/>1. 更新文件：仅包含更新运算符表达式。有关更多信息，请参见 使用更新运算符表达式文档进行更新。<br/>2. 聚合管道（*从MongoDB 4.2开始*）：仅包含以下聚合阶段：<br />a. `$addFields`及其别名 `$set`<br />b. `$project`及其别名 `$unset`<br />c. `replaceRoot`及其别名`$replaceWith`。<br />有关更多信息，请参见 使用聚合管道更新。<br />要使用替换文档进行更新，请参阅 `db.collection.replaceOne()`。 |
| `upsert`       | boolean            | 可选的。当`true`，updateOne()时：<br/>1. 如果没有文档匹配`filter`，则创建一个新文档。有关详细信息，请参阅upsert 行为。 <br/>2. 更新匹配`filter`的文档。 <br/>要避免多次 upsert，请确保`filter`字段为唯一索引。 <br/>默认为`false`。 |
| `writeConcern` | document           | 可选的。表示写关注的文件。省略使用默认写入问题。<br />如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。 |
| `collation`    | document           | 可选的。 <br/>指定要用于操作的排序规则。 <br/>排序规则允许用户为字符串比较指定特定于语言的规则，例如字母大写和重音符号的规则。<br />排序规则选项具有以下语法：<br/>collation：{<br/>     locale：&lt;string&gt;，<br/>     caseLevel：&lt;boolean&gt;，<br/>     caseFirst：&lt;string&gt;，<br/>     strength：&lt;int&gt;，<br/>     numericOrdering：&lt;boolean&gt;，<br/>     alternate：&lt;string&gt;，<br/>     maxVariable：&lt;string&gt;，<br/>     backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他排序规则字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，则MongoDB会将以前版本中使用的简单二进制比较用于字符串比较。 <br/>您不能为一个操作指定多个排序规则。例如，您不能为每个字段指定不同的排序规则，或者如果对排序执行查找，则不能对查找使用一种排序规则，而对排序使用另一种排序规则。 <br/>*3.4版的新功能。* |
| `arrayFilters` | array              | 可选的。过滤器文档的 array，用于确定要在 array 字段上为更新操作修改哪些 array 元素。 <br/>在更新文档中，使用`$[<identifier>]`过滤后的位置运算符定义一个标识符，然后在数组过滤器文档中引用该标识符。如果更新文档中未包含标识符，则不能具有数组过滤器文档作为标识符。 <br/><br/>**注意**<br/>`<identifier>`必须以小写字母开头，并且只包含字母数字字符。 <br/><br/>您可以在更新文档中多次包含相同的标识符;但是，对于更新文档中的每个不同标识符(`$[identifier]`)，您必须指定**恰好一个**对应的 array 过滤器文档。也就是说，您不能为同一标识符指定多个 array 过滤器文档。对于 example，如果 update 语句包含标识符`x`(可能多次)，则不能为`arrayFilters`指定以下内容，其中包含 2 个单独的`x`过滤器文档：<br/>[<br/>   { "x.a": { $gt: 85 } },<br/>   { "x.b": { $gt: 80 } }<br/> ]<br/>但是，您可以在同一标识符上指定复合条件单个过滤器文档，例如以下示例：<br/>[<br/>     {$or：[{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}]} <br/>] <br/>[<br/>     {$and：[{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}]} <br/>] <br/>[<br/>   { "x.a": { $gt: 85 }, "x.b": { $gt: 80 } }<br/>] <br/>有关示例，请参阅为数组更新操作指定arrayFilters。 <br/>version 3.6 中的新内容。 |
| `hint`         | Document or string | 可选的。一个文档或字符串，它指定用于支持查询谓词的索引。<br />该选项可以采用索引规范文档或索引名称字符串。<br />如果指定的索引不存在，则操作错误。<br />有关示例，请参见为更新操作指定提示。<br />*4.2.1版中的新功能。* |

| <br/>  |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 包含以下内容的文档：<br/>一个布尔值`acknowledged`，就好像该操作在运行时带有 写关注关系`true`或是否禁用了写关注关系`false`<br/> `matchedCount`包含匹配文档数<br/> `modifiedCount`包含已修改文档数<br/> `upsertedId`包含`_id` 要提交的文档 |

## <span id="access-control">访问控制</span>

在运行`authorization`时，用户必须具有包括以下特权的访问权限：

- `update` 对指定集合的操作。
- `find`对指定集合的操作。
- `insert`如果操作导致更新，则对指定的集合执行操作。

内置角色`readWrite`提供所需的特权。

## <span id="behaviors">行为</span>

### 更新单个文件

`db.collection.updateOne()`查找与过滤器匹配的第一个文档，并应用指定的 更新修改。

### 使用更新运算符表达式文档进行更新

对于更新规范，`db.collection.updateOne()`方法可以接受仅包含更新运算符表达式的文档。

例如：

```powershell
db.collection.updateOne(
   <query>,
   { $set: { status: "D" }, $inc: { quantity: 2 } },
   ...
)
```

### 使用聚合管道进行更新

从MongoDB 4.2开始，`db.collection.updateOne()`方法可以接受指定要执行的修改的聚合管道 。管道可以包括以下阶段：`[ <stage1>, <stage2>, ... ]`

- `$addFields`及其别名 `$set`
- `$project`及其别名 `$unset`
- `$replaceRoot`及其别名`$replaceWith`。

使用聚合管道可以实现更具表达力的更新语句，例如根据当前字段值表达条件更新，或使用另一个字段的值更新一个字段。

例如：

```powershell
db.collection.updateOne(
   <query>,
   [
      { $set: { status: "Modified", comments: [ "$misc1", "$misc2" ] } },
      { $unset: [ "misc1", "misc2" ] }
   ]
   ...
)
```

> **注意**
>
> `$set`和`$unset`在管道中是指聚集阶段`$set`，并`$unset`分别，而不是更新的运算符`$set`和`$unset`。

有关示例，请参见使用聚合管道更新。

### UPSERT 

如果`upsert: true`和`filter`没有文档匹配，则 `db.collection.updateOne()`根据条件`filter`和`update`修改创建一个新文档。请参阅 使用Upsert更新。

如果在分片集合上指定`upsert: true`，则必须在filter中包括完整的分片键。有关分片集合`db.collection.updateOne()`的其他行为，请参见分片集合。

### 固定集合

如果更新操作更改了文档大小，则该操作将失败。

### 分片集合

要`db.collection.updateOne()`在分片集合上使用：

- 如果未指定`upsert: true`，则必须在字段`_id`上包含完全匹配项或将目标指定为单个分片（例如，通过在过滤器中包含分片键）。
- 如果指定`upsert: true`，则过滤器 必须包含分片键。

#### 碎片键修改

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

要用于`db.collection.updateOne()`更新分片键：

- 您**必须**在运行`mongos`无论是在事务或作为重试写。千万**不能**直接在碎片颁发运行。
- 您**必须**在查询过滤器的完整分片键上包含相等条件。例如，如果一个集合`messages` 使用`{ country : 1, userid : 1 }`的片键，更新为一个文件的碎片关键，你必须包括在`country: <value>, userid: <value>`查询过滤器。您可以根据需要在查询中包括其他字段。

### 可解释性

`updateOne()`与不兼容 `db.collection.explain()`。

使用`update()`代替。

### 事务

`db.collection.updateOne()`可以在多文档事务中使用。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

#### 现有的集合和事务

在事务内部，您可以指定对现有集合的读/写操作。如果`db.collection.updateOne()`导致upsert，则该集合必须已经存在。

#### 写关注和事务

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

## <span id="examples">例子</span>

### 更新

`restaurant`集合包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan" },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 0 }
```

以下操作使用`violations`字段更新`name: "Central Perk Cafe"`的单个文档：

```powershell
try {
    db.restaurant.updateOne(
        { "name" : "Central Perk Cafe" },
        { $set: { "violations" : 3 } }
    );
} catch (e) {
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

如果未找到 match，则设置`upsert: true`将插入文档。见使用 Upsert 更新

### 使用聚合管道更新

从MongoDB 4.2开始，`db.collection.updateOne()`可以使用聚合管道进行更新。管道可以包括以下阶段：

- `$addFields`及其别名 `$set`
- `$project`及其别名 `$unset`
- `$replaceRoot`及其别名`$replaceWith`。

使用聚合管道可以实现更具表达力的更新语句，例如根据当前字段值表达条件更新，或使用另一个字段的值更新一个字段。

#### 实施例1 

以下示例使用聚合管道文档中其他字段的值来修改字段。

`members`使用以下文档创建一个集合：

```powershell
db.members.insertMany([
    { "_id" : 1, "member" : "abc123", "status" : "A", "points" : 2, "misc1" : "note to self: confirm status", "misc2" : "Need to activate", "lastUpdate" : ISODate("2019-01-01T00:00:00Z") },
    { "_id" : 2, "member" : "xyz123", "status" : "A", "points" : 60, comments: [ "reminder: ping me at 100pts", "Some random comment" ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") }
])
```

假设您希望将这些字段收集到一个新字段中，而不是使用单独的`misc1`和`misc2`字段`comments`。以下更新操作使用聚合管道执行以下操作：

- 添加新`comments`字段并设置该`lastUpdate`字段。
- 删除集合中所有文档的`misc1`和`misc2`字段。

```powershell
db.members.updateOne(
   { _id: 1 },
   [
      { $set: { status: "Modified", comments: [ "$misc1", "$misc2" ], lastUpdate: "$$NOW" } },
      { $unset: [ "misc1", "misc2" ] }
   ]
)
```

> **注意**
>
> `$set`和`$unset`在管道中是指聚合阶段`$set`，并`$unset`分别，而不是更新的运营商`$set`和`$unset`。

**第一阶段**

`$set`阶段：

* 创建一个新的数组字段，`comments`其元素是`misc1`和`misc2`字段的当前内容
* 将字段设置为`lastUpdate`聚合变量的值`NOW`。聚合变量 `NOW`解析为当前日期时间值，并且在整个管道中保持不变。要访问聚合变量，请在变量前加双美元符号`$$` 并用引号引起来。

第二阶段

该`$unset`阶段将删除`misc1`和`misc2`字段。

命令后，集合包含以下文档：

```powershell
{ "_id" : 1, "member" : "abc123", "status" : "Modified", "points" : 2, "lastUpdate" : ISODate("2020-01-23T05:21:59.321Z"), "comments" : [ "note to self: confirm status", "Need to activate" ] }
{ "_id" : 2, "member" : "xyz123", "status" : "A", "points" : 60, "comments" : [ "reminder: ping me at 100pts", "Some random comment" ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") }
```

#### 示例

聚合管道允许基于当前字段值执行条件更新，以及使用当前字段值来计算单独的字段值。

例如，`students3`使用以下文档创建一个集合：

```powershell
db.students3.insert([
   { "_id" : 1, "tests" : [ 95, 92, 90 ], "average" : 92, "grade" : "A", "lastUpdate" : ISODate("2020-01-23T05:18:40.013Z") },
   { "_id" : 2, "tests" : [ 94, 88, 90 ], "average" : 91, "grade" : "A", "lastUpdate" : ISODate("2020-01-23T05:18:40.013Z") },
   { "_id" : 3, "tests" : [ 70, 75, 82 ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") }
]);
```

第三个文档`_id: 3`缺少`average`和 `grade` 字段。使用聚合管道，可以使用计算出的平均成绩和字母成绩更新文档。

```powershell
db.students3.updateOne(
   { _id: 3 },
   [
     { $set: { average: { $trunc: [  { $avg: "$tests" }, 0 ] }, lastUpdate: "$$NOW" } },
     { $set: { grade: { $switch: {
                           branches: [
                               { case: { $gte: [ "$average", 90 ] }, then: "A" },
                               { case: { $gte: [ "$average", 80 ] }, then: "B" },
                               { case: { $gte: [ "$average", 70 ] }, then: "C" },
                               { case: { $gte: [ "$average", 60 ] }, then: "D" }
                           ],
                           default: "F"
     } } } }
   ]
)
```

> **注意**
>
> `$set`管道中的使用是指聚合阶段 `$set`，而不是更新运算符`$set`。

第一阶段

`$set`阶段：

* 根据字段`average`的平均值 计算一个新`tests`字段。请参阅`$avg`有关 `$avg`聚合运算符`$trunc`的更多信息和有关`$trunc`截断聚合运算符的更多信息。
* 将字段设置为`lastUpdate`聚合变量的值`NOW`。聚合变量 `NOW`解析为当前日期时间值，并且在整个管道中保持不变。要访问聚合变量，请在变量前加双美元符号`$$` 并用引号引起来。

第二阶段

`$set`阶段计算新字段`grade`基础上，`average`在前一阶段计算。参见 `$switch`以获取有关`$switch` 聚合运算符的更多信息。

命令后，集合包含以下文档：

```powershell
{ "_id" : 1, "tests" : [ 95, 92, 90 ], "average" : 92, "grade" : "A", "lastUpdate" : ISODate("2020-01-23T05:18:40.013Z") }
{ "_id" : 2, "tests" : [ 94, 88, 90 ], "average" : 91, "grade" : "A", "lastUpdate" : ISODate("2020-01-23T05:18:40.013Z") }
{ "_id" : 3, "tests" : [ 70, 75, 82 ], "lastUpdate" : ISODate("2020-01-24T17:33:30.674Z"), "average" : 75, "grade" : "C" }
```

> **也可以看看**
>
> 聚合管道更新

### 使用 Upsert 更新

`restaurant`集合包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan", "violations" : 3 },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : "0" }
```

以下操作尝试使用`name : "Pizza Rat's Pizzaria"`更新文档，而`upsert: true`：

```powershell
try {
    db.restaurant.updateOne(
        { "name" : "Pizza Rat's Pizzaria" },
        { $set: {"_id" : 4, "violations" : 7, "borough" : "Manhattan" } },
        { upsert: true }
    );
} catch (e) {
    print(e);
}
```

从`upsert:true`开始，文档基于`filter`和`update`标准`inserted`。操作返回：

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
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 4 },
{ "_id" : 4, "name" : "Pizza Rat's Pizzaria", "Borough" : "Manhattan", "violations" : 7 }
```

`name`字段使用`filter`条件填充，而`update` operators 用于创建文档的 rest。

以下操作使用`violations`更新大于`10`的第一个文档：

```powershell
try {
    db.restaurant.updateOne(
        { "violations" : { $gt: 10} },
        { $set: { "Closed" : true } },
        { upsert: true }
    );
} catch (e) {
    print(e);
}
```

操作返回：

```powershell
{
    "acknowledged" : true,
    "matchedCount" : 0,
    "modifiedCount" : 0,
    "upsertedId" : ObjectId("56310c3c0c5cbb6031cafaea")
}
```

该集合现在包含以下文档：

```powershell
{ "_id" : 1, "name" : "Central Perk Cafe", "Borough" : "Manhattan", "violations" : 3 },
{ "_id" : 2, "name" : "Rock A Feller Bar and Grill", "Borough" : "Queens", "violations" : 2 },
{ "_id" : 3, "name" : "Empire State Pub", "Borough" : "Brooklyn", "violations" : 4 },
{ "_id" : 4, "name" : "Pizza Rat's Pizzaria", "Borough" : "Manhattan", "grade" : 7 }
{ "_id" : ObjectId("56310c3c0c5cbb6031cafaea"), "Closed" : true }
```

由于没有文档与过滤器匹配，并且`upsert`是`true`，updateOne仅使用生成的`_id`和`update`条件插入文档。

### 写关注更新

给定三个成员副本集，以下操作指定`majority` `wtimeout`，`wtimeout` `100`：

```powershell
try {
    db.restaurant.updateOne(
        { "name" : "Pizza Rat's Pizzaria" },
        { $inc: { "violations" : 3}, $set: { "Closed" : true } },
        { w: "majority", wtimeout: 100 }
    );
} catch (e) {
    print(e);
}
```

如果主要和至少一个辅助设备在 100 毫秒内确认每个写入操作，则返回：

```powershell
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

如果确认时间超过`wtimeout`限制，则抛出以下 exception：

```powershell
WriteConcernError({
    "code" : 64,
    "errInfo" : {
        "wtimeout" : true
    },
    "errmsg" : "waiting for replication timed out"
}):
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
db.myColl.updateOne(
    { category: "cafe" },
    { $set: { status: "Updated" } },
    { collation: { locale: "fr", strength: 1 } }
);
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

要修改`grades` array 中大于或等于`100`的所有元素，请使用过滤后的位置 operator $ [&lt;identifier&gt;]和db.collection.updateOne方法中的`arrayFilters`选项：

```powershell
db.students.updateOne(
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

要修改是大于或等于所有元素`100`中 `grades`阵列中，使用过滤的位置操作者 `$[<identifier>]`与`arrayFilters`在选项 `db.collection.updateOne`方法：

```powershell
db.students.updateOne(
   { grades: { $gte: 100 } },
   { $set: { "grades.$[element]" : 100 } },
   { arrayFilters: [ { "element": { $gte: 100 } } ] }
)
```

该操作将更新`grades`单个文档的字段，并且在操作之后，集合具有以下文档：

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

要修改`grades` array 中等级大于或等于`85`的所有元素的`mean`字段的 value，请使用过滤后的位置 operator $ [&lt;identifier&gt;]和db.collection.updateOne方法中的`arrayFilters`：

```powershell
db.students2.updateOne(
    { },
    { $set: { "grades.$[elem].mean" : 100 } },
    { arrayFilters: [ { "elem.grade": { $gte: 85 } } ] }
)
```

该操作更新单个文档的 array，并且在操作之后，该集合具有以下文档：

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

> **也可以看看**
>
> 要更新多个文档，请参阅db.collection.updateMany()。

### 指定`hint`更新操作

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
db.members.updateOne(
   { "points": { $lte: 20 }, "status": "P" },
   { $set: { "misc1": "Need to activate" } },
   { hint: { status: 1 } }
)
```

update命令返回以下内容：

```powershell
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

要查看使用的索引，可以使用`$indexStats`管道：

```powershell
db.members.aggregate( [ { $indexStats: { } }, { $sort: { name: 1 } } ] )
```



译者：李冠飞

校对：