## 排序

**在本页面**

- [排序文件](#文件)
- [支持排序的操作](#操作)
- [行为](#行为)

*新版本3.4*

排序允许用户为字符串比较指定特定于语言的规则，例如字母大小写和重音符号的规则。<br/ >

可以为集合或视图、索引或支持排序的特定操作指定排序。

### <span id="文件">排序文件</span>

一个排序文档有以下字段:

```powershell
{
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

在指定排序时，`locale`字段是强制性的;所有其他排序字段都是可选的。有关字段的描述，请参见[Collation Document](https://docs.mongodb.com/master/reference/collation/#coll-document-fields)。

默认的排序参数值因指定的语言环境而异。有关默认排序参数和它们关联的地区的完整列表，请参见[排序默认参数](https://docs.mongodb.com/master/reference/coll-locales-defaults/ # coll-default-params)。

| 字段              | Type    | Description                                                  |
| :---------------- | :------ | :----------------------------------------------------------- |
| `locale`          | string  | ICU的语言环境。参见[支持的语言和地区](https://docs.mongodb.com/master/reference/coll-locales-defaults/ # coll-languages-locales)获取支持的地区列表。<br />若要指定简单二进制比较，请将`locale`值指定为`simple`。 |
| `strength`        | integer | 可选的。要执行的比较级别。对应[ICU比较水平](http://userguide.icu-project.org/collation/concepts#TOC-Comparison-Levels)。值可能是:<br />**值**                  **描述**<br />1.               初级水平的比较。排序只对基本字符进行比较，忽略其他差异，如变音符号和大小写.<br />2.              二级比较。排序会执行到次要差异的比较，比如变音符号。也就是说，排序执行基本字符(主要差异)和变音符号(次要差异)的比较。基本字符之间的差异优先于次要字符之间的差异.<br />3.               三级比较。排序执行到第三级差异的比较，例如大小写和字母变体。也就是说，排序执    行基本字符(主要差异)、变音符号(次要差异)以及大小写和变体(第三差异)的比较。基本字符之间的差异优先于次要差异，后者优先于第三级差异.<br />                 这是默认级别.<br />4.              四级比较。当级别1-3忽略标点符号或用于处理日语文本时，限制了特定用例考虑标点符号。<br />5.                相同的水平。限制了连接断路器的特定用例.<br />                  参见[ICU排序:比较级别](http://userguide.icu-project.org/collation/concepts#TOC-Comparison-Levels)了解详细信息.<br /> |
| `caseLevel`       | boolean | 可选。决定是否在`<强度>`级别`1`或`2`包含大小写比较的标志。<br />如果为`true`，包括大小写比较;即:<br />      当与`strength:1`一起使用时，排序比较基本字符和大小写。<br />      当与`strength:2`一起使用时，排序比较基本字符、变音符号(以及可能的其他次要差异)和大小写。<br />如果为`false`，不要在`1`或`2`级别包括大小写比较。默认值是`false`。<br />更多信息，请参见[ICU Collation: Case Level](http://userguide.icu-project.org/collation/concepts#TOC-CaseLevel). |
| `caseFirst`       | string  | 可选的。一个字段，用于确定第三级比较时大小写差异的排序顺序。<br />值可能是:<br />值                     描述<br />`upper`        大写排序在小写之前。<br />`lower`        小写排序在大写排序之前。<br />`off`            默认值。与`lower`相似，但略有不同。有关差异的详细信息，请参阅http://userguide.icu-project.org/collation/customization。 |
| `numericOrdering` | boolean | 可选的。决定将数字字符串作为数字或字符串比较的标志。<br />如果为`true`, 比较数字;即.“10”大于“2”。<br />如果为 `false`, 比较字符串;即.“10”比“2”小。<br />默认设置是`false`。 |
| `alternate`       | string  | 可选的。确定排序规则是否应将空格和标点符号视为基本字符以便进行比较的字段。<br />值可能是：<br />值                                        描述<br />`"non-ignorable"`        空格和标点符号被认为是基本字符。<br />`"shifted"`                    空格和标点符号不被认为是基本字符，只在强度级别大于3时区分。<br />更多信息请参见[ICU Collation: Comparison Levels](http://userguide.icu-project.org/collation/concepts#TOC-Comparison-Levels)。<br />默认是`non-ignorable`的. |
| `maxVariable`     | string  | 可能的. 当`alternate:”shift`时，决定哪些字符被认为是可忽略的字段。如果`alternate: non-ignorable`没有任何影响.<br />值可能是:<br />值                              描述<br />`"punct"`            空格和标点符号都是“可忽略的”，即:不考虑基本字符。<br />`"space" `            空格是“可忽略的”，即:不考虑基本字符。 |
| `backwards`       | boolean | 可选的。确定带有变音符号的字符串是否从字符串后面排序的标志，例如使用法语字典排序。<br />如果为 `true`,   从后面到前面比较。<br />如果为 `false`, 从前面到后面进行比较。<br />默认值是`false`。 |
| `normalization`   | boolean | 可选的。决定是否检查文本是否需要标准化以及是否执行标准化的标志。通常，大多数文本不需要这种规范化处理。<br />如果为 `true`,          检查是否完全标准化，并执行标准化来比较文本。<br />如果为 `false`,        不检查。默认值是`false`。<br />有关详细信息，请参阅http://userguide.icu-project.org/collation/concepts#TOC-Normalization。 |

### <span id="操作">支持排序的操作</span>

您可以为以下操作指定排序规则:

> **注意**
>
> 不能为操作指定多个排序规则。例如，不能为每个字段指定不同的排序规则，如果使用排序执行查找，则不能对查找使用一个排序规则，对排序使用另一个排序规则。

| 命令                                                         | `mongo` Shell 方法                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`create`](https://docs.mongodb.com/master/reference/command/create/#dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/master/reference/method/db.createCollection/#db.createCollection)[`db.createView()`](https://docs.mongodb.com/master/reference/method/db.createView/#db.createView) |
| [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex) |
| [`aggregate`](https://docs.mongodb.com/master/reference/command/aggregate/#dbcmd.aggregate) | [`db.collection.aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate) |
| [`distinct`](https://docs.mongodb.com/master/reference/command/distinct/#dbcmd.distinct) | [`db.collection.distinct()`](https://docs.mongodb.com/master/reference/method/db.collection.distinct/#db.collection.distinct) |
| [`findAndModify`](https://docs.mongodb.com/master/reference/command/findAndModify/#dbcmd.findAndModify) | [`db.collection.findAndModify()`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify)[`db.collection.findOneAndDelete()`](https://docs.mongodb.com/master/reference/method/db.collection.findOneAndDelete/#db.collection.findOneAndDelete)[`db.collection.findOneAndReplace()`](https://docs.mongodb.com/master/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace)[`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/master/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate) |
| [`find`](https://docs.mongodb.com/master/reference/command/find/#dbcmd.find) | [`cursor.collation()`](https://docs.mongodb.com/master/reference/method/cursor.collation/#cursor.collation) 指定排序 [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) |
| [`mapReduce`](https://docs.mongodb.com/master/reference/command/mapReduce/#dbcmd.mapReduce) | [`db.collection.mapReduce()`](https://docs.mongodb.com/master/reference/method/db.collection.mapReduce/#db.collection.mapReduce) |
| [`delete`](https://docs.mongodb.com/master/reference/command/delete/#dbcmd.delete) | [`db.collection.deleteOne()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteOne/#db.collection.deleteOne)[`db.collection.deleteMany()`](https://docs.mongodb.com/master/reference/method/db.collection.deleteMany/#db.collection.deleteMany)[`db.collection.remove()`](https://docs.mongodb.com/master/reference/method/db.collection.remove/#db.collection.remove) |
| [`update`](https://docs.mongodb.com/master/reference/command/update/#dbcmd.update) | [`db.collection.update()`](https://docs.mongodb.com/master/reference/method/db.collection.update/#db.collection.update)[`db.collection.updateOne()`](https://docs.mongodb.com/master/reference/method/db.collection.updateOne/#db.collection.updateOne),[`db.collection.updateMany()`](https://docs.mongodb.com/master/reference/method/db.collection.updateMany/#db.collection.updateMany),[`db.collection.replaceOne()`](https://docs.mongodb.com/master/reference/method/db.collection.replaceOne/#db.collection.replaceOne) |
| [`shardCollection`](https://docs.mongodb.com/master/reference/command/shardCollection/#dbcmd.shardCollection) | [`sh.shardCollection()`](https://docs.mongodb.com/master/reference/method/sh.shardCollection/#sh.shardCollection) |
| [`count`](https://docs.mongodb.com/master/reference/command/count/#dbcmd.count) | [`db.collection.count()`](https://docs.mongodb.com/master/reference/method/db.collection.count/#db.collection.count) |
|                                                              | [`db.collection.bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)中的个别更新、替换和删除操作 |

### <span id="行为">行为</span>

#### 局部变量

一些排序区域有变量，它们使用特定于语言的规则。要指定语言环境变量，请使用以下语法:

```powershell
{ "locale" : "<locale code>@collation=<variant>" }
```

例如，使用中文排序的`unihan`变体:

```powershell
{ "locale" : "zh@collation=unihan" }
```

有关所有排序locale及其变量的完整列表，请参见[排序locale](https://docs.mongodb.com/master/reference/coll-locales-defaults/ # coll-languages-locales)。

#### 排序和视图

- 您可以在创建时为视图指定一个默认的[collation](https://docs.mongodb.com/master/reference/collation/#)。如果没有指定排序，视图的默认排序规则是“简单”二进制比较排序。也就是说，视图不继承集合的默认排序。
- 视图上的字符串比较使用视图的默认排序。试图更改或覆盖视图默认排序规则的操作将失败并出现错误。
- 如果从另一个视图创建视图，则不能指定与源视图的排序不同的排序。
- 如果执行涉及多个视图的聚合，例如使用[`$lookup`](https://docs.mongodb.com/master/reference/operator/aggregation/lookup/#pipe._S_lookup)或[`$graphLookup`](https://docs.mongodb.com/master/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup)，视图必须具有相同的[collation](https://docs.mongodb.com/master/reference/collation/#)。

#### 排序和索引的使用

若要使用索引进行字符串比较，操作还必须指定相同的排序。也就是说，如果索引指定了不同的排序，则具有排序的索引不能支持对索引字段执行字符串比较的操作。

例如，集合`myColl` 在字符串字段`category` 上有一个索引，其排序区域设置为`"fr"` 。

```powershell
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

下面的查询操作指定了与索引相同的排序，可以使用索引:

```powershell
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

但是，以下查询操作，默认使用“simple”二进制排序器，不能使用索引:

```powershell
db.myColl.find( { category: "cafe" } )
```

对于索引前缀键不是字符串、数组和嵌入文档的复合索引，指定不同排序规则的操作仍然可以使用索引来支持对索引前缀键的比较。

例如，集合`myColl`在数值字段`score`和`price`以及字符串字段`category`上有一个复合索引;索引是用collation locale `"fr"`创建的，用于字符串比较:

```powershell
db.myColl.createIndex(
   { score: 1, price: 1, category: 1 },
   { collation: { locale: "fr" } } )
```

以下使用`"simple" `二进制排序来进行字符串比较的操作可以使用索引:

```powershell
db.myColl.find( { score: 5 } ).sort( { price: 1 } )
db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

下面的操作使用`"simple"`二进制排序来对索引的`category`字段进行字符串比较，可以使用索引来完成查询的`score: 5`部分:

```powershell
db.myColl.find( { score: 5, category: "cafe" } )
```

#### 排序和不支持的索引类型

以下索引只支持简单的二进制比较，不支持[collation](https://docs.mongodb.com/master/reference/bson-type-comparison-order/#collation):

- [文字](https://docs.mongodb.com/master/core/index-text/)索引
- [2d](https://docs.mongodb.com/master/core/2d/)索引
- [geoHaystack](https://docs.mongodb.com/master/core/geohaystack/)索引。

> 提示
>
> 要在具有非简单排序规则的集合上创建**text**、**2d**或**geoHaystack**索引，必须在创建索引时显式指定`{collation: {locale: "simple"}}`。



译者：李冠飞

校对：