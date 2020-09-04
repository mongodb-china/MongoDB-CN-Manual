# [ ](#)db.collection.update（）

[]()

在本页面

*   [定义](#definition)
*   [语法](#syntax)
*   [访问控制](#access-control)
*   [行为](#behaviors)
*   [例子](#examples)
*   [写结果](#writeresult)

## <span id="definition">定义</span>

*   `db.collection.` `update`(查询，更新，选项)

    修改集合中的现有文档。该方法可以修改现有文档的特定字段或完全替换现有文档，具体取决于更新参数。
    
    默认情况下，update()方法更新**单**文档。设置多参数`multi：true`以更新 match 查询条件的所有文档。

## <span id="syntax">语法</span>

`db.collection.update()`方法具有以下形式：

```powershell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

### 参数

`db.collection.update()`方法采用以下参数：

| 参数           | 类型                 | 描述                                                         |
| -------------- | -------------------- | ------------------------------------------------------------ |
| `query`        | document             | 更新的选择标准。提供与方法中相同的查询选择器`find()`。 <br/>当执行`update()`with 且查询不匹配任何现有文档时，如果查询使用点表示法在字段上指定条件，则MongoDB将拒绝插入新文档 。`upsert: true` `_id` |
| `update`       | document or pipeline | 要应用的修改。可以是以下之一：<br />更新文件：仅包含更新运算符表达式<br />更新文件：仅包含键值对`<field1>: <value1>`<br />聚合管道 （*从MongoDB 4.2开始*）：仅包含以下聚合阶段：<br />a. `$addFields`及其别名 `$set`<br />b. `$project`及其别名 `$unset`<br />c. `$replaceRoot`及其别名`$replaceWith`<br />有关详细信息和示例，请参见示例。 |
| `upsert`       | boolean              | 可选的。如果设置为`true`，则在没有文档与查询条件匹配时创建新文档。默认的 value 是`false`，当没有找到 match 时，它不会插入新文档。 |
| `multi`        | boolean              | 可选的。如果设置为`true`，则更新符合`query`条件的多个文档。如果设置为`false`，则更新一个文档。默认的 value 是`false`。有关其他信息，请参阅多参数。 |
| `writeConcern` | document             | 可选的。表示写关注的文件。省略使用默认写入问题。w: 1 <br/> 如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。<br />有关使用的示例`writeConcern`，请参见 覆盖默认写问题。 |
| `collation`    | document             | 可选的。 <br/>排序规则允许用户为字符串比较指定特定于语言的规则，例如字母大写和重音符号的规则。 <br/>有关使用的示例`collation`，请参见 指定排序规则。<br />version 3.4 中的新内容。 |
| `arrayFilters` | array                | 可选的。过滤器文档数组，用于确定要在数组 字段上为更新操作修改的数组元素。 <br/>在更新文档中，使用$ [&lt;identifier&gt;]来定义标识符，仅更新那些与arrayFilters中相应的filter文档相匹配的数组元素。 <br/>     **注意**<br/>     如果更新文档中未包含标识符，则不能具有数组过滤器文档作为标识符。 <br/>有关示例，请参阅为数组更新操作指定arrayFilters。<br />version 3.6 中的新内容。 |
| hint           | document or string   | 可选的。一个文档或字符串，它指定用于支持查询谓词的索引。<br />该选项可以采用索引规范文档或索引名称字符串。<br />如果指定的索引不存在，则操作错误。<br />有关示例，请参见为更新操作指定提示。<br />*4.2版中的新功能。* |

| <br /> |                                           |
| ------ | ----------------------------------------- |
| 返回： | 该方法返回包含操作状态的WriteResult文档。 |

## <span id="access-control">访问控制</span>

在运行时`authorization`，用户必须具有包括以下特权的访问权限：

- `update`对指定集合的操作。
- `find`对指定集合的操作。
- `insert`如果操作导致更新，则对指定的集合执行操作。

内置角色`readWrite`提供所需的特权。

## <span id="behaviors">行为</span>

### 分片集合

`db.collection.update()`要与分片集合一起使用，必须在 字段上包括完全匹配项或将目标设为单个分片（例如：通过包含分片键）。`multi: false_id`

当`db.collection.update()`执行更新操作（而不是文档替换操作）时， `db.collection.update()`可以针对多个分片。

> **也可以看看**
>
> `findAndModify()`

#### 替换分片集合上的文档操作

从MongoDB 4.2开始，替换文档操作首先尝试使用查询过滤器来针对单个分片。如果该操作无法通过查询过滤器定位到单个分片，则它将尝试以替换文档定位。

在早期版本中，该操作尝试使用替换文档作为目标。

#### `upsert`在分片集合上

对于`db.collection.update()`包含 `upsert：true`且位于分片集合上的操作，您必须在中包含完整的分片键`filter`：

- 用于更新操作。
- 用于替换文档操作（从MongoDB 4.2开始）。

#### 碎片键修改

从MongoDB 4.2开始，您可以更新文档的分片键值，除非分片键字段是不可变`_id`字段。有关更新分片键的详细信息，请参见更改文档的分片键值。

在MongoDB 4.2之前，文档的分片键字段值是不可变的。

要用于`db.collection.update()`更新分片键：

- 您必须指定。`multi: false`

- 您**必须**在运行`mongos`无论是在 事务或作为重试写。千万**不能**直接在碎片发布运行。
- 您**必须**在查询过滤器的完整分片键上包含相等条件。例如，如果一个集合`messages` 使用`{ country : 1, userid : 1 }`的片键，更新为一个文件的碎片关键，你必须包括`country: <value>, userid: <value>`在查询过滤器。您可以根据需要在查询中包括其他字段。 

### 事务

`db.collection.update()`可以在多文档交易中使用。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如：运行时限制和操作日志大小限制），另请参见 生产注意事项。

#### 现有的集合和事务

在事务内部，您可以指定对现有集合的读/写操作。如果`db.collection.update()`导致upsert，则该集合必须已经存在。

如果该操作导致upsert，则该集合必须已经存在。

#### 写关注和事务

如果在事务中运行，请不要为操作明确设置写关注点。要对事务使用写关注，请参见 事务和写关注。

## <span id="examples">例子</span>

#### 使用更新运算符表达式（$ inc，$ set）

在`mongo`shell中，创建一个`books`包含以下文档的集合。此命令首先从`books`集合中删除所有先前存在的文档：

```powershell
db.books.remove({});

db.books.insertMany([
  {
    "_id" : 1,
    "item" : "TBD",
    "stock" : 0,
    "info" : { "publisher" : "1111", "pages" : 430 },
    "tags" : [ "technology", "computer" ],
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
    "reorder" : false
   },
   {
    "_id" : 2,
    "item" : "XYZ123",
    "stock" : 15,
    "info" : { "publisher" : "5555", "pages" : 150 },
    "tags" : [ ],
    "ratings" : [ { "by" : "xyz", "rating" : 5 } ],
    "reorder" : false
   }
]);
```

如果`<update>`文档包含更新运算符修饰符（例如使用 `$set`修饰符的修饰符），则：

- 该`<update>`文档必须*仅* 包含更新运算符表达式。
- 该`db.collection.update()`方法仅更新文档中的相应字段。
  - 要整体更新嵌入式文档或数组，请为该字段指定替换值。
  - 要更新嵌入式文档或数组中的特定字段，请使用点表示法 指定该字段。

您可以使用下面的Web Shell插入示例文档并执行示例更新操作：

```powershell
db.books.update(
   { _id: 1 },
   {
     $inc: { stock: 5 },
     $set: {
       item: "ABC123",
       "info.publisher": "2222",
       tags: [ "software" ],
       "ratings.1": { by: "xyz", rating: 3 }
     }
   }
)
```

在此操作中：

- 该`<query>`参数指定更新哪个文档，`{ _id: 1 }`
- 在`$inc`操作递增`stock`字段，
- 在`$set`运算符替换值：
  - `item` 字段，
  - `publisher` `info`嵌入文档中的字段，
  - `tags` 字段
  - `ratings`数组中的第二个元素。

更新后的文档如下：

```powershell
{

  "_id" : 1,
  "item" : "ABC123",
  "stock" : 5,
  "info" : { "publisher" : "2222", "pages" : 430 },
  "tags" : [ "software" ],
  "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "xyz", "rating" : 3 } ],
  "reorder" : false
}
```

此操作对应于以下SQL语句：

```powershell
UPDATE books
SET    stock = stock + 5
       item = "ABC123"
       publisher = 2222
       pages = 430
       tags = "software"
       rating_authors = "ijk,xyz"
       rating_values = "4,3"
WHERE  _id = 1
```

> **注意**
>
> 如果`query`参数已匹配多个文档，则此操作将仅更新一个匹配的文档。要更新多个文档，必须将`multi`选项设置为`true`。

<br />

> **也可以看看**
>
> `$set`, `$inc`, `Update运算符`, `点符号`

#### 将元素添加到现有数组

在`mongo`shell中，创建一个`books`包含以下文档的集合。此命令首先从`books`集合中删除所有先前存在的文档：

```powershell
db.books.remove({});

db.books.insertMany([
  {
    "_id" : 1,
    "item" : "TBD",
    "stock" : 0,
    "info" : { "publisher" : "1111", "pages" : 430 },
    "tags" : [ "technology", "computer" ],
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
    "reorder" : false
   },
   {
    "_id" : 2,
    "item" : "XYZ123",
    "stock" : 15,
    "info" : { "publisher" : "5555", "pages" : 150 },
    "tags" : [ ],
    "ratings" : [ { "by" : "xyz", "rating" : 5 } ],
    "reorder" : false
   }
]);
```

以下操作使用`$push`update运算符将新对象附加到`ratings`数组。

您可以使用下面的Web Shell插入示例文档并执行示例更新操作：

```powershell
db.books.update(
   { _id: 2 },
   {
     $push: { ratings: { "by" : "jkl", "rating" : 2 } }
   }
)
```

更新后的文档如下：

```powershell
{
  "_id" : 2,
  "item" : "XYZ123",
  "stock" : 15,
  "info" : {
   "publisher" : "5555",
   "pages" : 150
  },
  "tags" : [ ],
  "ratings" : [
   { "by" : "xyz", "rating" : 5 },

   { "by" : "jkl", "rating" : 2 }

  ],
  "reorder" : false
 }
```

> **也可以看看**
>
> `$push`

#### 删除字段（$ unset）

在`mongo`shell中，创建一个`books`包含以下文档的集合。此命令首先从`books`集合中删除所有先前存在的文档：

```powershell
db.books.remove({});

db.books.insertMany([
  {
    "_id" : 1,
    "item" : "TBD",
    "stock" : 0,
    "info" : { "publisher" : "1111", "pages" : 430 },
    "tags" : [ "technology", "computer" ],
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
    "reorder" : false
   },
   {
    "_id" : 2,
    "item" : "XYZ123",
    "stock" : 15,
    "info" : { "publisher" : "5555", "pages" : 150 },
    "tags" : [ ],
    "ratings" : [ { "by" : "xyz", "rating" : 5 } ],
    "reorder" : false
   }
]);
```

以下操作使用`$unset`操作符通过删除`tags`文档中的字段。`{ _id: 1 }`

您可以使用下面的Web Shell插入示例文档并执行示例更新操作：

```
db.books.update( { _id: 1 }, { $unset: { tags: 1 } } )
```

更新后的文档如下：

```powershell
{
  "_id" : 1,
  "item" : "TBD",
  "stock" : 0,
  "info" : {
   "publisher" : "1111",
   "pages" : 430
  },
  "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
  "reorder" : false
 }
```

没有直接等效于`$unset`的SQL ，但是`$unset`类似于以下SQL命令，该命令`tags`从`books` 表中删除了该字段：

```sql
ALTER TABLE books
DROP COLUMN tags
```

> **也可以看看**
>
> `$unset`，`$rename`，update运算符

#### 替换整个文件

在`mongo`shell中，创建一个`books`包含以下文档的集合。此命令首先从`books`集合中删除所有先前存在的文档：

```powershell
db.books.remove({});

db.books.insertMany([
  {
    "_id" : 1,
    "item" : "TBD",
    "stock" : 0,
    "info" : { "publisher" : "1111", "pages" : 430 },
    "tags" : [ "technology", "computer" ],
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
    "reorder" : false
   },
   {
    "_id" : 2,
    "item" : "XYZ123",
    "stock" : 15,
    "info" : { "publisher" : "5555", "pages" : 150 },
    "tags" : [ ],
    "ratings" : [ { "by" : "xyz", "rating" : 5 } ],
    "reorder" : false
   }
]);
```

如果`<update>`文档*仅* 包含`field:value` 表达式，则：

- 该`db.collection.update()`方法*将*匹配的文档*替换*为`<update>`文档。该 `db.collection.update()`方法*不会*替换该 `_id`值。
- `db.collection.update()`*无法*更新多个文档。

以下操作将传递`<update>`仅包含字段和值对的文档。该`<update>`文档将完全替换原始文档（`_id`字段除外）。

您可以使用下面的Web Shell插入示例文档并执行示例更新操作：

```powershell
db.books.update(
   { _id: 2 },
   {
     item: "XYZ123",
     stock: 10,
     info: { publisher: "2255", pages: 150 },
     tags: [ "baking", "cooking" ]
   }
)
```

更新的文档*仅*包含替换文档中的`_id`字段和该字段。这样，这些字段 `ratings`和`reorder`不再存在于更新的文档中，因为这些字段不在替换文档中。

```powershell
{
   "_id" : 2,
   "item" : "XYZ123",
   "stock" : 10,
   "info" : { "publisher" : "2255", "pages" : 150 },
   "tags" : [ "baking", "cooking" ]
}
```

此操作对应于以下SQL语句：

```sql
DELETE from books WHERE _id = 2

INSERT INTO books
            (_id,
             item,
             stock,
             publisher,
             pages,
             tags)
VALUES     (2,
            "xyz123",
            10,
            "2255",
            150,
            "baking,cooking")
```

#### 更新多个文件

在`mongo`shell中，创建一个`books`包含以下文档的集合。此命令首先从`books`集合中删除所有先前存在的文档：

```powershell
db.books.remove({});

db.books.insertMany([
  {
    "_id" : 1,
    "item" : "TBD",
    "stock" : 0,
    "info" : { "publisher" : "1111", "pages" : 430 },
    "tags" : [ "technology", "computer" ],
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "lmn", "rating" : 5 } ],
    "reorder" : false
   },
   {
    "_id" : 2,
    "item" : "XYZ123",
    "stock" : 15,
    "info" : { "publisher" : "5555", "pages" : 150 },
    "tags" : [ ],
    "ratings" : [ { "by" : "xyz", "rating" : 5 } ],
    "reorder" : false
   }
]);
```

如果`multi`设置为`true`，则该 `db.collection.update()`方法将更新*所有*符合`<query>`条件的文档。该`multi`更新操作可以与其他的读/写操作交错。

以下操作将 所有小于或等于10的 文档的`reorder`字段设置`true`为。如果匹配的文档中不存在该字段，则运算符将使用指定的值添加该字段。

您可以使用下面的Web Shell插入示例文档并执行示例更新操作：

```powershell
db.books.update(
   { stock: { $lte: 10 } },
   { $set: { reorder: true } },
   { multi: true }
)
```

集合中的结果文档如下：

```powershell
[
  {
    "_id" : 1,
    "item" : "ABC123",
    "stock" : 5,
    "info" : {
     "publisher" : "2222",
     "pages" : 430
    },
    "ratings" : [ { "by" : "ijk", "rating" : 4 }, { "by" : "xyz", "rating" : 3 } ],

    "reorder" : true

   }
   {
     "_id" : 2,
     "item" : "XYZ123",
     "stock" : 10,
     "info" : { "publisher" : "2255", "pages" : 150 },
     "tags" : [ "baking", "cooking" ],

     "reorder" : true

   }
]
```

此操作对应于以下SQL语句：

```sql
UPDATE books
SET reorder=true
WHERE stock <= 10
```

> **注意**
>
> 您无法指定何时执行替换，即&lt;update&gt;文档何时*仅*包含表达式：`multi: true` `field:value`

<br />

> **也可以看看**
>
> $set`

### 如果不存在匹配项，则插入新文档（`Upsert`）

当您指定选项`upsert：true`时：

- 如果文档符合查询条件，请 `db.collection.update()`执行更新。
- 如果没有文件的查询条件匹配， `db.collection.update()`插入一个*单一的*文件。

如果在分片集合上指定`upsert: true`，则必须在`filter` 中包含完整的分片键。有关分片集合的其他 行为，请参见分片集合。 `db.collection.update()`

#### 替换文件更新

如果没有文档符合查询条件，并且`<update>` 参数是替换文档（即：仅包含字段和值的键值对），则更新将插入带有替换文档的字段和值的新文档。

- 如果您`_id`在查询参数或替换文档中指定字段，则MongoDB将`_id`在插入的文档中使用该字段。
- 如果您未`_id`在查询参数或替换文档中指定字段，则MongoDB生成的`_id`字段会添加带有随机生成的ObjectId值的 字段。

> **注意**
>
> 您不能`_id`在查询参数和替换文档中指定其他字段值。如果这样做，则操作错误。

例如，以下更新将upsert选项设置为`true`：

```powershell
db.books.update(
   { item: "ZZZ135" },   // Query parameter
   {                     // Replacement document
     item: "ZZZ135",
     stock: 5,
     tags: [ "database" ]
   },

   { upsert: true }      // Options

)
```

如果没有文档与该`<query>`参数匹配，则更新操作将插入*仅*包含替换文档的文档。由于`_id`在替换文档或查询文档中未指定任何字段，因此该操作`ObjectId`将为新文档的`_id`字段创建一个新的唯一性。您可以看到`upsert`反映在操作的WriteResult中：

```powershell
WriteResult({
  "nMatched" : 0,
  "nUpserted" : 1,
  "nModified" : 0,
  "_id" : ObjectId("5da78973835b2f1c75347a83")
 })
```

该操作将以下文档插入`books` 集合中（您的ObjectId值将有所不同）：

```powershell
{
  "_id" : ObjectId("5da78973835b2f1c75347a83"),
  "item" : "ZZZ135",
  "stock" : 5,
  "tags" : [ "database" ]
}
```

#### 带运算符表达式的Upsert

如果没有文档符合查询条件，并且`<update>` 参数是带有update运算符expression的文档，则该操作根据`<query>`参数中的equals子句创建基本文档，并应用参数中的表达式`<update>`。

来自的比较操作`<query>`将不会包含在新文档中。如果新文档不包含该`_id`字段，则MongoDB将`_id`使用ObjectId值添加该字段。

例如，以下更新将upsert选项设置为`true`：

```powershell
db.books.update(
   { item: "BLP921" },   // Query parameter
   {                     // Update document
      $set: { reorder: false },
      $setOnInsert: { stock: 10 }
   },
   { upsert: true }      // Options
)
```

如果没有文档符合查询条件，则该操作将插入以下文档（您的ObjectId值将有所不同）：

```powershell
{
  "_id" : ObjectId("5da79019835b2f1c75348a0a"),
  "item" : "BLP921",
  "reorder" : false,
  "stock" : 10
}
```

> **也可以看看**
>
> `$setOnInsert`

#### 使用Upsert的聚合管道

如果`<update>`参数是聚合管道，则更新将从`<query>` 参数中的equals子句创建基础文档，然后将管道应用于文档以创建要插入的文档。如果新文档不包含该`_id`字段，则MongoDB将`_id`使用ObjectId值添加该字段。

例如，以下`upsert：true`操作指定使用以下内容的聚合管道

- 该`$replaceRoot`阶段可以提供与`$setOnInsert`更新运算符表达式类似的行为，
- `$set`可以提供与`$set`更新操作符表达式相似的行为的阶段，
- 聚合变量`NOW`，它解析为当前日期时间，并且可以提供与`$currentDate`更新运算符表达式类似的行为 。

```powershell
db.books.update(
   { item: "MRQ014", ratings: [2, 5, 3] }, // Query parameter
   [                                       // Aggregation pipeline
      { $replaceRoot: { newRoot: { $mergeObjects: [ { stock: 0 }, "$$ROOT"  ] } } },
      { $set: { avgRating: { $avg: "$ratings" }, tags: [ "fiction", "murder" ], lastModified: "$$NOW" } }
   ],
   { upsert: true }   // Options
)
```

如果没有文档与该`<query>`参数匹配，则该操作会将以下文档插入到`books` 集合中（您的ObjectId值将有所不同）：

```powershell
{
   "_id" : ObjectId("5e2921e0b4c550aad59d1ba9"),
   "stock" : 0,
   "item" : "MRQ014",
   "ratings" : [ 2, 5, 3 ],
   "avgRating" : 3.3333333333333335,
   "tags" : [ "fiction", "murder" ],
   "lastModified" : ISODate("2020-01-23T04:32:32.951Z")
}
```

> **也可以看看**
>
> 有关使用聚合管道进行更新的其他示例，请参见使用聚合管道进行更新。

#### 结合Upsert和多选项

##### 结合使用Upsert和多选项（匹配）

在`mongo`shell中，将以下文档插入`books`集合中：

```powershell
db.books.insertMany([
  {
    _id: 5,
    item: "RQM909",
    stock: 18,
    info: { publisher: "0000", pages: 170 },
    reorder: true
  },
  {
    _id: 6,
    item: "EFG222",
    stock: 15,
    info: { publisher: "1111", pages: 72 },
    reorder: true
  }
])
```

以下操作同时指定了`multi`选项和`upsert`选项。如果存在匹配的文档，则该操作将更新所有匹配的文档。如果不存在匹配的文档，则该操作将插入一个新文档。

```powershell
db.books.update(
   { stock: { $gte: 10 } },        // Query parameter
   {                               // Update document
     $set: { reorder: false, tags: [ "literature", "translated" ] }
   },
   { upsert: true, multi: true }   // Options
)
```

该操作将更新所有匹配的文档，并产生以下结果：

```powershell
{
   "_id" : 5,
   "item" : "RQM909",
   "stock" : 18,
   "info" : { "publisher" : "0000", "pages" : 170 },
   "reorder" : false,
   "tags" : [ "literature", "translated" ]
}
{
   "_id" : 6,
   "item" : "EFG222",
   "stock" : 15,
   "info" : { "publisher" : "1111", "pages" : 72 },
   "reorder" : false,
   "tags" : [ "literature", "translated" ]
}
```

##### 结合使用Upsert和多选项（无匹配项）

如果集合中*没有*匹配的文档，则该操作将导致使用`<query>`和`<update>` 规范中的字段插入单个文档。例如，考虑以下操作：

```powershell
db.books.update(
  { "info.publisher": "Self-Published" },   // Query parameter
  {                                         // Update document
    $set: { reorder: false, tags: [ "literature", "hardcover" ], stock: 25 }
  },
  { upsert: true, multi: true }             // Options
)
```

该操作将以下文档插入`books` 集合中（您的ObjectId值将有所不同）：

```powershell
{
  "_id" : ObjectId("5db337934f670d584b6ca8e0"),
  "info" : { "publisher" : "Self-Published" },
  "reorder" : false,
  "stock" : 25,
  "tags" : [ "literature", "hardcover" ]
}
```

#### 带Dotted_id查询的Upsert

当执行`update()`with 且查询不匹配任何现有文档时，如果查询使用点表示法在字段上指定条件，则MongoDB将拒绝插入新文档 。`upsert: true` `_id`

此限制可确保`_id`文档中嵌入的字段的顺序 定义明确，并且不与查询中指定的顺序绑定。

如果您尝试以这种方式插入文档，MongoDB将引发错误。例如，考虑以下更新操作。由于更新操作指定`upsert:true`了`_id`字段并且查询使用点符号指定了字段上的条件，因此在构建要插入的文档时，更新将导致错误。

```powershell
db.collection.update( { "_id.name": "Robert Frost", "_id.uid": 0 },
   { "categories": ["poet", "playwright"] },
   { upsert: true } )
```

该`WriteResult`操作将返回以下错误：

```powershell
WriteResult({
  "nMatched" : 0,
  "nUpserted" : 0,
  "nModified" : 0,
  "writeError" : {
    "code" : 111,
    "errmsg" : "field at '_id' must be exactly specified, field at sub-path '_id.name'found"
  }
})
```

> **也可以看看**
>
> `WriteResult()`

#### 使用唯一索引

> **警告**
>
> 为避免多次插入同一文档，请仅在`query`字段是唯一索引时使用`upsert: true` 。

给定一个名为集合`people`，其中没有包含`Andy`值的`name`字段，考虑当多个客户端在同一时间使用`upsert: true`发出以下`db.collection.update()`时：

```powershell
db.people.update(
   { name: "Andy" },   // Query parameter
   {                   // Update document
      name: "Andy",
      rating: 1,
      score: 1
   },
   { upsert: true }    // Options
)
```

如果所有`db.collection.update()`操作`query`在任何客户端成功插入数据之前完成了该 部分，**并且** 该`name`字段上没有唯一索引，则每个更新操作都可能导致插入。

为防止MongoDB多次插入同一文档，请在字段上创建唯一索引`name`。使用唯一索引，如果多个应用程序使用`upsert: true`发出相同的更新，则*恰好一个*`db.collection.update()`将成功插入新文档。 

其余操作将是：

- 更新新插入的文档，

- 当他们尝试插入重复项时失败。

  如果操作由于重复的索引键错误而失败，则应用程序可以重试该操作，该操作将作为更新操作成功。

> **也可以看看**
>
> `$setOnInsert`

### 使用聚合管道更新

从MongoDB 4.2开始，`db.collection.update()`方法可以接受指定要执行的修改的聚合管道。管道可以包括以下阶段：`[ <stage1>, <stage2>, ... ]`

- `$addFields`及其别名 `$set`
- `$project`及其别名 `$unset`
- `$replaceRoot`及其别名`$replaceWith`。

使用聚合管道可以实现更具表达力的更新语句，例如根据当前字段值表达条件更新，或使用另一个字段的值更新一个字段。

#### 使用文档中其他字段的值修改字段

`members`使用以下文档创建一个集合：

```powershell
db.members.insertMany([
   { "_id" : 1, "member" : "abc123", "status" : "A", "points" : 2, "misc1" : "note to self: confirm status", "misc2" : "Need to activate", "lastUpdate" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 2, "member" : "xyz123", "status" : "A", "points" : 60, "misc1" : "reminder: ping me at 100pts", "misc2" : "Some random comment", "lastUpdate" : ISODate("2019-01-01T00:00:00Z") }
])
```

假设您希望将这些字段收集到一个新字段中，而不是使用单独的`misc1`和`misc2`字段`comments`。以下更新操作使用聚合管道执行以下操作：

- 添加新`comments`字段并设置该`lastUpdate`字段。
- 删除集合中所有文档的`misc1`和`misc2`字段。

```powershell
db.members.update(
   { },
   [
      { $set: { status: "Modified", comments: [ "$misc1", "$misc2" ], lastUpdate: "$$NOW" } },
      { $unset: [ "misc1", "misc2" ] }
   ],
   { multi: true }
)
```

> **注意**
>
> `$set`和`$unset`在管道中是指聚合阶段`$set`，并`$unset` 分别，而不是更新的运营商`$set`和 `$unset`。

**第一阶段**

[`$set`](https://docs.mongodb.com/manual/reference/operator/aggregation/set/#pipe._S_set)阶段：

* 创建一个新的数组字段，`comments`其元素是`misc1`和`misc2`字段的当前内容，
* 并且将字段设置为`lastUpdate`聚合变量的值`NOW`。聚合变量 `NOW`解析为当前日期时间值，并且在整个管道中保持不变。要访问聚合变量，请在变量前加双美元符号`$$` 并用引号引起来。

**第二阶段**

`$unset`阶段将删除`misc1`和`misc2`字段。

命令后，集合包含以下文档：

```powershell
{ "_id" : 1, "member" : "abc123", "status" : "Modified", "points" : 2, "lastUpdate" : ISODate("2020-01-23T05:11:45.784Z"), "comments" : [ "note to self: confirm status", "Need to activate" ] }
{ "_id" : 2, "member" : "xyz123", "status" : "Modified", "points" : 60, "lastUpdate" : ISODate("2020-01-23T05:11:45.784Z"), "comments" : [ "reminder: ping me at 100pts", "Some random comment" ] }
```

> **也可以看看**
>
> Updates with Aggregation Pipeline

#### 根据当前字段值执行条件更新

使用以下文档创建一个`students3`集合：

```powershell
db.students3.insert([
   { "_id" : 1, "tests" : [ 95, 92, 90 ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 2, "tests" : [ 94, 88, 90 ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 3, "tests" : [ 70, 75, 82 ], "lastUpdate" : ISODate("2019-01-01T00:00:00Z") }
]);
```

使用聚合管道，可以使用计算出的平均成绩和字母成绩更新文档。

```powershell
db.students3.update(
   { },
   [
     { $set: { average : { $trunc: [ { $avg: "$tests" }, 0 ] }, lastUpdate: "$$NOW" } },
     { $set: { grade: { $switch: {
                           branches: [
                               { case: { $gte: [ "$average", 90 ] }, then: "A" },
                               { case: { $gte: [ "$average", 80 ] }, then: "B" },
                               { case: { $gte: [ "$average", 70 ] }, then: "C" },
                               { case: { $gte: [ "$average", 60 ] }, then: "D" }
                           ],
                           default: "F"
     } } } }
   ],
   { multi: true }
)
```

> **注意**
>
> `$set`管道中的使用是指聚合阶段 `$set`，而不是更新运算符`$set`。

**第一阶段**

`$set`阶段：

* 根据字段`average`的平均值 计算一个新`tests`字段。请参阅`$avg`有关 `$avg`聚合运算符`$trunc`的更多信息和有关`$trunc`截取聚合运算符的更多信息 。
* 将字段设置为`lastUpdate`聚合变量的值`NOW`。聚合变量 `NOW`解析为当前日期时间值，并且在整个管道中保持不变。要访问聚合变量，请在变量前加双美元符号`$$` 并用引号引起来。

**第二阶段**

`$set`阶段根据前一阶段计算的平均`成绩`计算新的`成绩`等级。参见 `$switch`以获取有关`$switch` 聚合运算符的更多信息。

命令后，集合包含以下文档：

```powershell
{ "_id" : 1, "tests" : [ 95, 92, 90 ], "lastUpdate" : ISODate("2020-01-24T17:29:35.340Z"), "average" : 92, "grade" : "A" }
{ "_id" : 2, "tests" : [ 94, 88, 90 ], "lastUpdate" : ISODate("2020-01-24T17:29:35.340Z"), "average" : 90, "grade" : "A" }
{ "_id" : 3, "tests" : [ 70, 75, 82 ], "lastUpdate" : ISODate("2020-01-24T17:29:35.340Z"), "average" : 75, "grade" : "C" }
```

> **也可以看看**
>
> Updates with Aggregation Pipeline

### 指定`arrayFilters`数组更新操作

在更新文档中，使用`$[<identifier>]`过滤后的位置运算符定义一个标识符，然后在数组过滤器文档中引用该标识符。如果更新文档中未包含标识符，则不能具有数组过滤器文档作为标识符。

> **注意**
>
> 在`<identifier>`必须以小写字母开头，并且只包含字母数字字符。

您可以在更新文档中多次包含相同的标识符；但是，对于`$[identifier]`更新文档中的每个不同的标识符，必须**精确地**指定**一个** 对应的数组过滤器文档。即：您不能为同一标识符指定多个数组过滤器文档。例如：如果update语句包含标识符`x` （可能多次），则不能为以下内容指定以下内容 `arrayFilters`：包括2个单独的过滤器文档`x`：

```powershell
// INVALID

[
  { "x.a": { $gt: 85 } },
  { "x.b": { $gt: 80 } }
]
```

但是，可以在单个过滤器文档中的相同标识符上指定复合条件，例如以下示例：

```powershell
// Example 1
[
  { $or: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }
]
// Example 2
[
  { $and: [{"x.a": {$gt: 85}}, {"x.b": {$gt: 80}}] }
]
// Example 3
[
  { "x.a": { $gt: 85 }, "x.b": { $gt: 80 } }
]
```

`arrayFilters` 不适用于使用聚合管道的更新。

#### 更新元素匹配`arrayFilters`条件

要更新所有符合指定条件的数组元素，请使用 arrayFilters参数。

在`mongo`shell程序中，`students` 使用以下文档创建一个集合：

```powershell
db.students.insertMany([
   { "_id" : 1, "grades" : [ 95, 92, 90 ] },
   { "_id" : 2, "grades" : [ 98, 100, 102 ] },
   { "_id" : 3, "grades" : [ 95, 110, 100 ] }
])
```

要更新`grades`阵列中大于或等于`100`的所有元素 ，使用过滤的位置操作符 `$[<identifier>]`与所述`arrayFilters`选项：

```powershell
db.students.update(
   { grades: { $gte: 100 } },
   { $set: { "grades.$[element]" : 100 } },
   {
     multi: true,
     arrayFilters: [ { "element": { $gte: 100 } } ]
   }
)
```

操作后，集合包含以下文档：

```powershell
{ "_id" : 1, "grades" : [ 95, 92, 90 ] }
{ "_id" : 2, "grades" : [ 98, 100, 100 ] }
{ "_id" : 3, "grades" : [ 95, 100, 100 ] }
```

#### 更新文档数组的特定元素

您还可以使用arrayFilters参数更新文档数组中的特定文档字段。

在`mongo`shell程序中，`students2` 使用以下文档创建一个集合：

```powershell
db.students2.insertMany([
  {
    "_id" : 1,
    "grades" : [
       { "grade" : 80, "mean" : 75, "std" : 6 },
       { "grade" : 85, "mean" : 90, "std" : 4 },
       { "grade" : 85, "mean" : 85, "std" : 6 }
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
])
```

要修改`mean`的`grades`数组中`grade`大于或等于`85`的所有元素的字段 值，请使用`$[<identifier>]`带有过滤条件的位置运算符和`arrayFilters`：

```powershell
db.students2.update(
   { },
   { $set: { "grades.$[elem].mean" : 100 } },
   {
     multi: true,
     arrayFilters: [ { "elem.grade": { $gte: 85 } } ]
   }
)
```

操作后，集合具有以下文档：

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
      { "grade" : 90, "mean" : 100, "std" : 6 },
      { "grade" : 87, "mean" : 100, "std" : 3 },
      { "grade" : 85, "mean" : 100, "std" : 4 }
   ]
}
```

### 指定`hint`更新操作

*4.2版中的新功能。*

`mongo`shell程序中，`members` 使用以下文档创建一个集合：

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

以下更新操作`hints`明确指出要使用索引：`{status: 1 }`

> **注意**
>
> 如果指定的索引不存在，则操作错误。

```powershell
db.members.update(
   { points: { $lte: 20 }, status: "P" },     // Query parameter
   { $set: { misc1: "Need to activate" } },   // Update document
   { multi: true, hint: { status: 1 } }       // Options
)
```

update命令返回以下内容：

```powershell
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
```

要查看使用的索引，请运行`explain`以下操作：

```powershell
db.members.explain().update(
   { "points": { $lte: 20 }, "status": "P" },
   { $set: { "misc1": "Need to activate" } },
   { multi: true, hint: { status: 1 } }
)
```

`db.collection.explain().update()`不修改文件。

### 覆盖默认写问题

对副本集的以下操作指定5,000毫秒的写关注时间`"w: majority"`，以 使该方法在写传送到大多数有表决权的副本集成员之后返回，或者该方法在5秒钟后超时。

```powershell
db.books.update(
   { stock: { $lte: 10 } },
   { $set: { reorder: true } },
   {
     multi: true,
     writeConcern: { w: "majority", wtimeout: 5000 }
   }
)
```

### 指定排序规则

指定 用于操作的排序规则。

排序规则允许用户为字符串比较指定特定于语言的规则，例如字母大写和重音符号的规则。

排序规则选项具有以下语法：

```powershell
collation: {
   locale: <string>,
   caseLevel: <boolean>,
   caseFirst: <string>,
   strength: <int>,
   numericOrdering: <boolean>,
   alternate: <string>,
   maxVariable: <string>,
   backwards: <boolean>
}
```

指定排序规则时，该`locale`字段为必填字段；所有其他排序规则字段都是可选的。有关字段的说明，请参见整理文档。

如果未指定排序规则，但是集合具有默认排序规则（请参阅参考资料`db.createCollection()`），则该操作将使用为集合指定的排序规则。

如果没有为集合或操作指定排序规则，则MongoDB会将以前版本中使用的简单二进制比较用于字符串比较。

您不能为一个操作指定多个排序规则。例如，您不能为每个字段指定不同的排序规则，或者如果对排序执行查找，则不能对查找使用一种排序规则，而对排序使用另一种排序规则。

*3.4版的新功能。*

在`mongo`shell程序中，创建一个`myColl`包含以下文档的集合 ：

```powershell
db.myColl.insertMany(
  [
    { _id: 1, category: "café", status: "A" },
    { _id: 2, category: "cafe", status: "a" },
    { _id: 3, category: "cafE", status: "a" }
  ])
```

下面的操作包括排序选项和设置`multi`，以`true`更新所有匹配的文档：

```powershell
db.myColl.update(
   { category: "cafe" },
   { $set: { status: "Updated" } },
   {
     collation: { locale: "fr", strength: 1 },
     multi: true
   }
);
```

该操作的写入结果返回以下文档，指示集合中的所有三个文档均已更新：

```powershell
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
```

操作后，集合包含以下文档：

```powershell
{ "_id" : 1, "category" : "café", "status" : "Updated" }
{ "_id" : 2, "category" : "cafe", "status" : "Updated" }
{ "_id" : 3, "category" : "cafE", "status" : "Updated" }
```

## <span id="writeresult">写结果</span>

### 成功的结果

`db.collection.update()`方法返回一个 `WriteResult`包含操作状态的对象。成功后，`WriteResult`对象包含符合查询条件的文档数，更新插入的文档数以及修改的文档数：

```powershell
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

> **注意**
>
> WriteResult.nMatched，WriteResult.nUpserted，WriteResult.nModified

### 写关注错误

如果该`db.collection.update()`方法遇到写关注错误，则结果包括以下 `WriteResult.writeConcernError`字段：

```powershell
WriteResult({
    "nMatched" : 1,
    "nUpserted" : 0,
    "nModified" : 1,
    "writeConcernError" : {
        "code" : 64,
        "errmsg" : "waiting for replication timed out at shard-a"
    }
})
```

> **也可以看看**
>
> WriteResult.hasWriteConcernError()

### 与写关注无关的错误

如果`db.collection.update()`方法遇到非写关注错误，则结果包括以下`WriteResult.writeError`字段：

```powershell
WriteResult({
    "nMatched" : 0,
    "nUpserted" : 0,
    "nModified" : 0,
    "writeError" : {
        "code" : 7,
        "errmsg" : "could not contact primary for replica set shard-a"
    }
})
```

> **也可以看看**
>
> WriteResult.hasWriteError()



译者：李冠飞

校对：