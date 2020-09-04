

# BSON类型

在本页面

- [对象Id ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)
- [字符串 String](https://docs.mongodb.com/v4.2/reference/bson-types/#string)
- [时间戳 Timestamps](https://docs.mongodb.com/v4.2/reference/bson-types/#timestamps)
- [日期 Date](https://docs.mongodb.com/v4.2/reference/bson-types/#date)



[BSON](https://docs.mongodb.com/v4.2/reference/glossary/#term-bson)是一种二进制序列化格式，用于在MongoDB中存储文档和进行远程过程调用。BSON规范位于[ bsonspec.org](http://bsonspec.org/)。

每种BSON类型都具有整数和字符串标识符，如下表所示：

| 类型 Type                    | 对应数字 Number | 别名 Alias            | 备注 Notes      |
| :--------------------------- | :-------------- | :-------------------- | :-------------- |
| 双精度浮点型Double           | 1               | “double”              |                 |
| 字符串String                 | 2               | “string”              |                 |
| 对象Object                   | 3               | “object”              |                 |
| 数组Array                    | 4               | “array”               |                 |
| 二进制数据Binary data        | 5               | “binData”             |                 |
| 未定义Undefined              | 6               | “undefined”           | 不推荐使用。    |
| 对象编号ObjectId             | 7               | “objectId”            |                 |
| 布尔型Boolean                | 8               | “bool”                |                 |
| 日期Date                     | 9               | “date”                |                 |
| 空值Null                     | 10              | “null”                |                 |
| 正则表达式Regular Expression | 11              | “regex”               |                 |
| DBPointer                    | 12              | “dbPointer”           | 不推荐使用。    |
| JavaScript                   | 13              | “javascript”          |                 |
| Symbol                       | 14              | “symbol”              | 不推荐使用。    |
| JavaScript (带范围)          | 15              | “javascriptWithScope” |                 |
| 32位整数 32-bit integer      | 16              | “int”                 |                 |
| 时间戳 Timestamp             | 17              | “timestamp”           |                 |
| 64位整数 64-bit integer      | 18              | “long”                |                 |
| 小数128 Decimal128           | 19              | “decimal”             | 3.4版的新功能。 |
| 最小键 Min key               | -1              | “minKey”              |                 |
| 最大键 Max key               | 127             | “maxKey”              |                 |

您可以将这些值与[`$type`](https://docs.mongodb.com/v4.2/reference/operator/query/type/#op._S_type)运算符一起使用，以按其BSON类型查询文档。所述[`$type`](https://docs.mongodb.com/v4.2/reference/operator/aggregation/type/#exp._S_type)聚合操作者返回的类型[操作者表达](https://docs.mongodb.com/v4.2/meta/aggregation-quick-reference/#agg-quick-ref-operator-expressions)使用列出的BSON类型字符串之一。

要确定字段的类型，请参阅[mongo Shell中的Check Types](https://docs.mongodb.com/v4.2/core/shell-types/#check-types-in-shell)。

如果将BSON转换为JSON，请参阅[扩展JSON](https://docs.mongodb.com/v4.2/reference/mongodb-extended-json/)参考。

以下各节描述了特定BSON类型的特殊注意事项。





## ObjectId 

ObjectId很小，可能唯一，可以快速生成并排序。ObjectId值的长度为12个字节，包括：

- 一个4字节的*时间戳记值*，代表自Unix时代以来以秒为单位的ObjectId的创建
- 5字节*随机值*
- 3字节*递增计数器*，初始化为随机值

虽然BSON格式本身是低位优先的，但*时间戳*和 *计数器*值却是高位优先的，最高有效字节在字节序列中排在最前面。

在MongoDB中，存储在集合中的每个文档都需要一个唯一的 [_id](https://docs.mongodb.com/v4.2/reference/glossary/#term-id)字段作为[主键](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary-key)。如果插入的文档省略了该`_id`字段，则MongoDB驱动程序会自动为该字段生成一个[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)`_id`。

这也适用于通过[upsert：true](https://docs.mongodb.com/v4.2/reference/method/db.collection.update/#upsert-parameter)通过更新操作插入的文档。



MongoDB客户端应添加一个`_id`具有唯一ObjectId 的字段。在该`_id`字段中使用ObjectIds 还可以带来以下好处：

- 在[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell中，您可以使用[`ObjectId.getTimestamp()`](https://docs.mongodb.com/v4.2/reference/method/ObjectId.getTimestamp/#ObjectId.getTimestamp)方法访问`ObjectId`的创建时间。

- 在存储`ObjectId`值的`_id`字段上按大致相当于创建时间进行排序。

  重要

  尽管[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)值应随时间增加，但不一定是单调的。这是因为他们：

  - 仅包含一秒的时间分辨率，因此 在同一秒内创建的[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)值没有保证的顺序，并且
  - 由客户端生成，客户端可能具有不同的系统时钟。



也可以看看

[`ObjectId()`](https://docs.mongodb.com/v4.2/reference/method/ObjectId/#ObjectId)



## 字符串

BSON字符串为UTF-8。通常，在对BSON进行序列化和反序列化时，每种编程语言的驱动程序都会从该语言的字符串格式转换为UTF-8。这样就可以轻松地将大多数国际字符存储在BSON字符串中。 [[1\]](https://docs.mongodb.com/v4.2/reference/bson-types/#sort-string-internationalization)此外，MongoDB [`$regex`](https://docs.mongodb.com/v4.2/reference/operator/query/regex/#op._S_regex)查询在正则表达式字符串中支持UTF-8。

| [[1\]](https://docs.mongodb.com/v4.2/reference/bson-types/#id3) | 给定使用UTF-8字符集的[`sort()`](https://docs.mongodb.com/v4.2/reference/method/cursor.sort/#cursor.sort)字符串，在字符串上使用将是合理正确的。但是，由于内部 [`sort()`](https://docs.mongodb.com/v4.2/reference/method/cursor.sort/#cursor.sort)使用C ++ `strcmp`API，因此排序顺序可能会错误地处理某些字符。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |



## 时间戳

BSON有一个特殊的时间戳类型给MongoDB*内部* 使用，而非常规相关的[日期](https://docs.mongodb.com/v4.2/reference/bson-types/#document-bson-type-date) 类型。此内部时间戳记类型是64位值，其中：

- 最重要的32位是一个`time_t`值（自Unix时代以来的秒数）
- 最低有效32位是`ordinal`给定秒内的操作增量。

虽然BSON格式是低位优先的，因此首先存储了最低有效位，但是无论字节序如何，在所有平台上[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)实例始终将`time_t`值与`ordinal`值比较。

在单个[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)实例中，时间戳记值始终是唯一的。

在复制中，操作[日志](https://docs.mongodb.com/v4.2/reference/glossary/#term-oplog)具有一个`ts`字段。该字段中的值反映了使用BSON时间戳值的操作时间。



注意

BSON时间戳类型供MongoDB*内部* 使用。在大多数情况下，在应用程序开发中，您将需要使用BSON日期类型。有关更多信息，请参见[日期](https://docs.mongodb.com/v4.2/reference/bson-types/#document-bson-type-date)。



当插入包含带有空时间戳值的顶级字段的文档时，MongoDB会将空时间戳值替换为当前时间戳值，但以下情况除外。如果`_id` 字段本身包含空的时间戳记值，则将始终按原样插入而不替换它。



示例

插入带有空时间戳值的文档：

复制

```
db.test.insertOne( { ts: new Timestamp() } );
```

运行[`db.test.find()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.find/#db.collection.find) 然后将返回类似于以下内容的文档：

```
{ "_id" : ObjectId("542c2b97bac0595474108b48"), "ts" : Timestamp(1412180887, 1) }
```

服务器已使用插入时的时间戳值替换了`ts`的空时间戳值。



## 日期 Date

BSON Date是一个64位整数，代表自Unix纪元（1970年1月1日）以来的毫秒数。这导致可以追溯到过去和未来约2.9亿年的日期范围。

该[官方BSON规范](http://bsonspec.org/#/specification) 指的是BSON Date类型为*UTC日期时间*。

BSON日期类型是有符号整数。[[2\]](https://docs.mongodb.com/v4.2/reference/bson-types/#unsigned-date)负值表示1970年之前的日期。



示例

在 [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell中使用构造函数 `new Date()` 构造一个Date ：

复制

```
var mydate1 = new Date()
```



示例

在 [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell中使用构造函数`ISODate()`构造一个Date ：

复制

```
var mydate2 = ISODate()
```



示例

以字符串形式返回`Date`值：

复制

```
mydate1.toString()
```



示例

返回日期值的月份部分；月是零索引，因此一月是`0`月：

复制

```
mydate1.getMonth()
```

| [[2\]](https://docs.mongodb.com/v4.2/reference/bson-types/#id4) | 在2.0版之前，`Date`值被错误地解释为*无符号*整数，这会影响排序，范围查询和`Date`字段索引。由于升级时不会重新创建索引，因此，如果您早期版本使用`Date`值创建了索引，请对与应用相关的、1970年前的日期进行重新索引。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |



原文链接：https://docs.mongodb.com/v4.2/reference/bson-types/

译者：小芒果