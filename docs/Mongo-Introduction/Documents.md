# 文档

在本页面

- [文档结构](https://docs.mongodb.com/v4.2/core/document/#document-structure)
- [点符号](https://docs.mongodb.com/v4.2/core/document/#dot-notation)
- [文档限制](https://docs.mongodb.com/v4.2/core/document/#document-limitations)
- [文档结构的其他用途](https://docs.mongodb.com/v4.2/core/document/#other-uses-of-the-document-structure)
- [更多阅读](https://docs.mongodb.com/v4.2/core/document/#further-reading)

MongoDB将数据记录存储为BSON文档。BSON是[JSON](https://docs.mongodb.com/v4.2/reference/glossary/#term-json)文档的二进制表示[形式](https://docs.mongodb.com/v4.2/reference/glossary/#term-json)，尽管它包含比JSON更多的数据类型。有关BSON规范，请参见[bsonspec.org](http://bsonspec.org/)。另请参阅[BSON类型](https://docs.mongodb.com/v4.2/reference/bson-types/)。

![A MongoDB document.](https://docs.mongodb.com/v4.2/_images/crud-annotated-document.bakedsvg.svg)



## 文档结构

MongoDB文档由字段和值对组成，并具有以下结构：

复制

```
{
   field1: value1,
   field2: value2,
   field3: value3,
   ...
   fieldN: valueN
}
```

字段的值可以是任何BSON [数据类型](https://docs.mongodb.com/v4.2/reference/bson-types/)，包括其他文档，数组和文档数组。例如，以下文档包含各种类型的值：

复制

```
var mydoc = {
               _id: ObjectId("5099803df3f4948bd2f98391"),
               name: { first: "Alan", last: "Turing" },
               birth: new Date('Jun 23, 1912'),
               death: new Date('Jun 07, 1954'),
               contribs: [ "Turing machine", "Turing test", "Turingery" ],
               views : NumberLong(1250000)
            }
```

上面的字段具有以下数据类型：

- `_id`拥有一个[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)。

- `name`包含一个包含字段`first`和`last`的*嵌入式文档*。

- `birth`和`death`保留*Date*类型的值。

- `contribs`拥有*字符串数组*。

- `views`拥有*NumberLong*类型的值。




### 字段名称

字段名称是字符串。

[文档](https://docs.mongodb.com/v4.2/core/document/#)对字段名称有以下限制：

- 字段名称`_id`保留用作主键；它的值在集合中必须是唯一的，不可变的，并且可以是数组以外的任何类型。

- 字段名称**不能**包含`null`字符。

- 顶级字段名称**不能**以美元符号（`$`）字符开头。

  否则，从MongoDB 3.6开始，服务器允许存储包含点（即`.`）和美元符号（即 `$`）的字段名称。



重要

MongoDB查询语言不能总是有效地表达对字段名称包含这些字符的文档的查询（请参阅[SERVER-30575](https://jira.mongodb.org/browse/SERVER-30575)）。

在查询语句中添加支持之前，不推荐在字段名称中使用`$`和 `.`，官方MongoDB的驱动程序不支持。

BSON文档可能有多个具有相同名称的字段。但是，大多数[MongoDB接口](https://docs.mongodb.com/ecosystem/drivers)都使用不支持重复字段名称的结构（例如，哈希表）来表示MongoDB。如果需要处理具有多个同名字段的[文档](https://docs.mongodb.com/ecosystem/drivers)，请参见[驱动程序文档](https://docs.mongodb.com/ecosystem/drivers)。

内部MongoDB流程创建的某些文档可能具有重复的字段，但是*任何* MongoDB流程都*不会*向现有的用户文档添加重复的字段。



### 字段值限制

- MongoDB 2.6至MongoDB版本，[并将featureCompatibilityVersion](https://docs.mongodb.com/v4.2/reference/command/setFeatureCompatibilityVersion/#view-fcv)（[fCV](https://docs.mongodb.com/v4.2/reference/command/setFeatureCompatibilityVersion/#view-fcv)）设置为`"4.0"`或更早版本

  对于[索引集合](https://docs.mongodb.com/v4.2/indexes/)，索引字段的值有一个最大索引键长度限制。有关详细信息，请参见[`Maximum Index Key Length`](https://docs.mongodb.com/v4.2/reference/limits/#Index-Key-Limit)。

  

## 点符号

MongoDB使用*点符号*访问数组的元素并访问嵌入式文档的字段。

### 数组

要通过从零开始的索引位置指定或访问数组的元素，请将数组名称与点（`.`）和从零开始的索引位置连接起来，并用引号引起来：

复制

```
"<array>.<index>"
```

例如，给定文档中的以下字段：

复制

```
{
   ...
   contribs: [ "Turing machine", "Turing test", "Turingery" ],
   ...
}
```

要指定`contribs`数组中的第三个元素，请使用点符号`"contribs.2"`。

有关查询数组的示例，请参见：

- [查询数组](https://docs.mongodb.com/v4.2/tutorial/query-arrays/)
- [查询嵌入式文档数组](https://docs.mongodb.com/v4.2/tutorial/query-array-of-documents/)



也可以看看

- `$[\]`用于更新操作的所有位置运算符，

- `$[/<identifier/>]` 过滤后的位置运算符，用于更新操作，

- [`$`](https://docs.mongodb.com/v4.2/reference/operator/update/positional/#up._S_) 用于更新操作的位置运算符，

- [`$`](https://docs.mongodb.com/v4.2/reference/operator/projection/positional/#proj._S_) 数组索引位置未知时的投影运算符

- [在数组中查询带数组](https://docs.mongodb.com/v4.2/tutorial/query-arrays/#read-operations-arrays)的点符号示例。

  

### 嵌入式文档

要使用点符号指定或访问嵌入式文档的字段，请将嵌入式文档名称与点（`.`）和字段名称连接在一起，并用引号引起来：

复制

```
"<embedded document>.<field>"
```

例如，给定文档中的以下字段：

复制

```
{
   ...
   name: { first: "Alan", last: "Turing" },
   contact: { phone: { type: "cell", number: "111-222-3333" } },
   ...
}
```

- 要指定在字段中命名`last`的`name`字段，请使用点符号`"name.last"`。
- 要在字段`number`中的`phone`文档中 指定`contact`，请使用点号`"contact.phone.number"`。

有关查询嵌入式文档的示例，请参见：

- [查询嵌入/嵌套文档](https://docs.mongodb.com/v4.2/tutorial/query-embedded-documents/)
- [查询嵌入式文档数组](https://docs.mongodb.com/v4.2/tutorial/query-array-of-documents/)





## 文件限制[¶](https://docs.mongodb.com/v4.2/core/document/#document-limitations)

文档具有以下属性：



### 文档大小限制

BSON文档的最大大小为16 MB。

最大文档大小有助于确保单个文档不会使用过多的RAM或在传输过程中占用过多的带宽。要存储大于最大大小的文档，MongoDB提供了GridFS API。有关GridFS的更多信息，请参见[`mongofiles`](https://docs.mongodb.com/v4.2/reference/program/mongofiles/#bin.mongofiles)和[驱动程序](https://docs.mongodb.com/ecosystem/drivers)的文档。



### 文档字段顺序

*除*以下情况*外*，MongoDB会在执行写操作后保留文档字段的顺序：

- 该`_id`字段始终是文档中的第一个字段。

- 包含[`renaming`](https://docs.mongodb.com/v4.2/reference/operator/update/rename/#up._S_rename)字段名称的更新可能会导致文档中字段的重新排序。

  

### `_id`字段

在MongoDB中，存储在集合中的每个文档都需要一个唯一的 [_id](https://docs.mongodb.com/v4.2/reference/glossary/#term-id)字段作为[主键](https://docs.mongodb.com/v4.2/reference/glossary/#term-primary-key)。如果插入的文档省略了该`_id`字段，则MongoDB驱动程序会自动为该`_id`字段生成一个[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)。

这也适用于通过使用[upsert：true](https://docs.mongodb.com/v4.2/reference/method/db.collection.update/#upsert-parameter)更新操作插入的文档。

该`_id`字段具有以下行为和约束：

- 默认情况下，MongoDB 在创建集合期间会在`_id`字段上创建唯一索引。
- 该`_id`字段始终是文档中的第一个字段。如果服务器首先接收到没有该`_id`字段的文档，则服务器会将字段移到开头。
- 该`_id`字段可以包含除数组之外的任何[BSON数据类型的](https://docs.mongodb.com/v4.2/reference/bson-types/)值。

警告

为确保复制正常进行，请勿在`_id` 字段中存储BSON正则表达式类型的值。



以下是用于存储值的常用选项`_id`：

- 使用一个[ObjectId](https://docs.mongodb.com/v4.2/reference/bson-types/#objectid)。

- 使用自然的唯一标识符（如果有）。这样可以节省空间并避免附加索引。

- 生成一个自动递增的数字。

- 在您的应用程序代码中生成一个UUID。为了在集合和`_id` 索引中更有效地存储UUID值，请将UUID存储为BSON `BinData`类型的值。

  在以下情况下，`BinData`更有效地将类型为索引的键存储在索引中：

  - 二进制子类型的值在0-7或128-135的范围内，并且
  - 字节数组的长度为：0、1、2、3、4、5、6、7、8、10、12、14、16、20、24或32。

- 使用驱动程序的BSON UUID工具生成UUID。请注意，驱动程序实现可能会以不同的方式实现UUID序列化和反序列化逻辑，这可能与其他驱动程序不完全兼容。有关UUID互操作性的信息，请参阅[驱动程序文档](https://docs.mongodb.com/drivers/)。

  

注意

大多数MongoDB驱动程序客户端将包括该`_id`字段，并`ObjectId`在将插入操作发送到MongoDB之前生成一个；但是，如果客户发送的文档中没有`_id` 字段，则[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)会添加该`_id`字段并生成`ObjectId`。



## 文档结构的其他用途

除了定义数据记录外，MongoDB还在整个文档结构中使用，包括但不限于：[查询过滤器](https://docs.mongodb.com/v4.2/core/document/#document-query-filter)，[更新规范文档](https://docs.mongodb.com/v4.2/core/document/#document-update-specification)和[索引规范文档](https://docs.mongodb.com/v4.2/core/document/#document-index-specification)。



### 查询过滤器文档

查询过滤器文档指定确定用于选择哪些记录以进行读取，更新和删除操作的条件。

您可以使用 `<field>:<value>` 表达式指定相等条件和[查询运算符](https://docs.mongodb.com/v4.2/reference/operator/query/) 表达式。

复制

```
{
  <field1>: <value1>,
  <field2>: { <operator>: <value> },
  ...
}
```

有关示例，请参见：

- [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)
- [查询嵌入/嵌套文档](https://docs.mongodb.com/v4.2/tutorial/query-embedded-documents/)
- [查询数组](https://docs.mongodb.com/v4.2/tutorial/query-arrays/)
- [查询嵌入式文档数组](https://docs.mongodb.com/v4.2/tutorial/query-array-of-documents/)



### 更新规范文档

更新规范文档使用[更新运算符](https://docs.mongodb.com/v4.2/reference/operator/update/#id1)来指定要在[`db.collection.update()`](https://docs.mongodb.com/v4.2/reference/method/db.collection.update/#db.collection.update)操作期间在特定字段上执行的数据修改。

复制

```
{
  <operator1>: { <field1>: <value1>, ... },
  <operator2>: { <field2>: <value2>, ... },
  ...
}
```

有关示例，请参阅[更新规范](https://docs.mongodb.com/v4.2/tutorial/update-documents/#update-documents-modifiers)。



### 索引规范文档

索引规范文档定义了要索引的字段和索引类型：

复制

```
{ <field1>: <type1>, <field2>: <type2>, ...  }
```





## 进一步阅读

有关MongoDB文档模型的更多信息，请下载 [MongoDB应用程序现代化指南](https://www.mongodb.com/modernize?tck=docs_server)。

下载内容包括以下资源：

- 演示使用MongoDB进行数据建模的方法
- 白皮书涵盖了从[RDBMS](https://docs.mongodb.com/v4.2/reference/glossary/#term-rdbms)数据模型迁移到MongoDB的最佳实践和注意事项
- 参考MongoDB模式及其等效RDBMS
- 应用程序现代化记分卡



原文链接：https://docs.mongodb.com/v4.2/core/document/

译者：小芒果