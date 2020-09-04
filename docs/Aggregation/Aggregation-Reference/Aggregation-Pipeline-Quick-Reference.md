# [ ](#)聚合管道快速参考

[]()

在本页面

*   [阶段](#stages)

*   [表达式](#expressions)

*   [运算符表达式](#operator-expressions)

*   [表达式运算符的索引](#index-of-expression-operators)
> 有关特定运算符的详细信息，包括语法和示例，请单击特定的运算符以转到其参考页面。

[]()

## <span id="stages">阶段</span>

[]()

### 阶段(db.collection.aggregate)

在[db.collection.aggregate](../../Reference/mongo-Shell-Methods/Collection-Methods/db-collection-aggregate.md)方法中，管道阶段出现在数组中。文档按顺序通过各个阶段。除[$out](), [$merge]()和[$geoNear]()阶段之外的所有阶段都可以在管道中多次出现。

```powershell
db.collection.aggregate( [ { <stage> }, ... ] )
```

| 阶段                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [$addFields]()      | 向文档添加新字段。类似于[$project]()，[$addFields]()重塑了流中的每个文档;具体而言，通过向输出文档添加新字段，该文档包含输入文档和新添加字段中的现有字段。<br />[`$set`]()是[`$addFields`]()的别名。 |
| [$bucket]()         | 根据指定的表达式和存储区边界，将传入的文档分组，称为bucket。 |
| [$bucketAuto]()     | 根据指定的表达式将传入的文档分类为特定数量的组(称为bucket)。自动确定bucket边界，以便将文档均匀地分配到指定数量的bucket中。 |
| [$collStats]()      | 返回有关集合或视图的统计信息。                               |
| [$count]()          | 返回聚合管道此阶段的文档数量计数。                           |
| [$facet]()          | 在同一组输入文档的单个阶段内处理多个[聚合管道]()。允许创建能够在单个阶段中跨多个维度或方面描述数据的多面聚合。 |
| [$geoNear]()        | 根据与地理空间点的接近程度返回一个有序的文档流。将[$match]()，[$sort]()和[$limit]()的功能合并到地理空间数据中。输出文档包括附加距离字段，并且可以包括位置标识符字段。 |
| [$graphLookup]()    | 对集合执行递归搜索。对于每个输出文档，添加一个新的数组字段，该字段包含该文档的递归搜索的遍历结果。 |
| [$group]()          | 按指定的标识符表达式对文档进行分组，并将累加器表达式(如果指定)应用于每个组。使用所有输入文档并为每个不同的组输出一个文档。输出文档只包含标识符字段和累积字段(如果指定的话)。 |
| [$indexStats]()     | 返回有关集合的每个索引的使用情况的统计信息。                 |
| [$limit]()          | 将未修改的前 n 个文档传递给管道，其中 n 是指定的限制。对于每个输入文档，输出一个文档(对于前 n 个文档)或零文档(在前 n 个文档之后)。 |
| [$listSessions]()   | 列出足以传播到`system.sessions`集合的所有会话。              |
| [$lookup]()         | 对同一数据库中的另一个集合执行左外连接，从“已连接”集合中过滤文档以进行处理。 |
| [$match]()          | 过滤文档流以仅允许匹配的文档未经修改地传递到下一个管道阶段。 [$match]()使用标准的 MongoDB 查询。对于每个输入文档，输出一个文档(匹配)或零文档(不匹配)。 |
| [$merge]()          | 将聚合管道的结果文档写入集合。这个阶段可以将结果合并到一个输出集合中(插入新文档、合并文档、替换文档、保留现有文档、操作失败、使用自定义更新管道处理文档)。要使用[`$merge`]()阶段，它必须是管道中的最后一个阶段。<br />version 4.2 中的新功能 |
| [$out]()            | 将聚合管道的结果文档写入集合。要使用[$out]()阶段，它必须是管道中的最后一个阶段。 |
| [$planCacheStats]() | 返回集合的计划缓存信息。                                     |
| [$project]()        | 重新整形流中的每个文档，例如添加新字段或删除现有字段。对于每个输入文档，输出一个文档。<br />有关删除现有字段，请参见[`$unset`]()。 |
| [$redact]()         | 通过基于文档本身中存储的信息限制每个文档的内容来重塑流中的每个文档。合并[$project]()和[$match]()的功能。可用于实现字段级修订。对于每个输入文档，输出一个或零个文档。 |
| [$replaceRoot]()    | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶层。<br />[`$replaceWith`]()是[`$replaceRoot`]()阶段的别名。 |
| [$replaceWith]()    | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括`_id`字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶层。<br />[`$replaceWith`]()是[`$replaceRoot`]()阶段的别名。 |
| [$sample]()         | 从输入中随机选择指定数量的文档。                             |
| [$set]()            | 向文档添加新字段。与[`$project`]()类似，[`$set`]()会重新塑造流中的每个文档；具体来说，通过向包含输入文档中的现有字段和新添加字段的输出文档添加新字段。<br />[`$set`]()是[`$addFields`]()阶段的别名。 |
| [$skip]()           | 跳过前 n 个文档，其中 n 是指定的跳过编号，并将其余未修改的文档传递给管道。对于每个输入文档，输出零文档(对于前 n 个文档)或一个文档(如果在前 n 个文档之后)。 |
| [$sort]()           | 按指定的排序键重新排序文档。只有顺序改变;文件保持不变。对于每个输入文档，输出一个文档。 |
| [$sortByCount]()    | 根据指定表达式的值对传入文档进行分组，然后计算每个不同组中的文档计数。 |
| [$unionWith]()      | 执行两个集合的并集;例如，将来自两个集合的管道结果组合成一个结果集。<br />version 4.4 中的新功能 |
| [$unset]()          | 从文档中移除/排除字段。<br />[`$unset`]()是移除字段阶段的[`$project stage`]()的别名。 |
| [$unwind]()         | 解析输入文档中的数组字段，为每个元素输出一个文档。每个输出文档用一个元素值替换数组。对于每个输入文档，输出n个文档，其中n是数组元素的数量，对于空数组可以为零。 |

[]()

### 阶段(db.aggregate)

从 version 3.6 开始，MongoDB 也提供了[db.aggregate]()方法：

```powershell
db.aggregate( [ { <stage> }, ... ] )
```

以下阶段使用[db.aggregate()]()方法而不是[db.collection.aggregate()]()方法。

| 阶段                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| [$currentOp]()         | 返回有关 MongoDB 部署的活动 and/or 休眠操作的信息。          |
| [$listLocalSessions]() | 列出当前连接的[mongos]()或[mongod]()实例上正在使用的所有活动会话。这些会话可能尚未传播到`system.sessions`集合。 |

### 阶段可用更新

从MongoDB 4.2开始，你可以使用聚合管道更新:

| 命令              | mongo Shell 方法                                             |
| ----------------- | ------------------------------------------------------------ |
| [findAndModify]() | [db.collection.findOneAndUpdate()]()<br />[db.collection.findAndModify()]() |
| [update]()        | [db.collection.updateOne()]()<br />[db.collection.updateMany()]()<br />[db.collection.update()]()<br /><br />[Bulk.find.update()]()<br />[Bulk.find.updateOne()]()<br />[Bulk.find.upsert()]() |

对于更新，管道可以包括以下阶段:

* [`$addFields`]()及其别名[`$set`]()
* [`$project`]()及其别名[`$unset`]()
* [`$replaceRoot`]()及其别名[`$replaceWith`]()

> **[success] 也可以看看**
>
> [聚合管道更新]()

[]()
[]()

## <span id="expressions">表达式</span>

表达式可以包括[字段路径]()，[Literals]()，[系统变量]()，[表达对象]()和[表达式操作符]()。表达式可以嵌套。
[]()
[]()

### 字段路径

聚合表达式使用[字段路径]()来访问输入文档中的字段。要指定字段路径，请在字段名或虚线字段名(如果字段在嵌入的文档中)前加上美元符号$。例如，“`$user`”指定用户字段的字段路径，“`$user.name`”指定“`user.name`”字段的字段路径。

`"$<field>"`等效于`"$$CURRENT.<field>"`，其中[CURRENT]()是系统变量，默认为当前对象的根，除非在特定阶段另有说明。

[]()
[]()

### 聚合变量

MongoDB提供了在表达式中使用的各种聚合[系统变量]()。要访问变量，请在变量名前加上`$$`。例如:

| 变量             | 通过$$访问         | 简介/描述                                                    |
| ---------------- | ------------------ | ------------------------------------------------------------ |
| [NOW]()          | **$$NOW**          | 返回当前的日期时间值，该值在部署的所有成员之间是相同的，并在整个聚合管道中保持不变。(4.2 + 版本中可用) |
| [CLUSTER_TIME]() | **$$CLUSTER_TIME** | 返回当前时间戳值，该值在部署的所有成员之间是相同的，并在整个聚合管道中保持不变。仅用于复制集和分片集群。(4.2 + 版本中可用) |
| [ROOT]()         | **$$ROOT**         | 引用根文档，即：顶级文档。                                   |
| [CURRENT]()      | **$$CURRENT**      | 引用字段路径的开始，默认情况下该路径是[ROOT]()，但可以更改。 |
| [REMOVE]()       | **$$REMOVE**       | 允许有条件地排除字段。(3.6 + 版本中可用)                     |
| [DESCEND]()      | **$$DESCEND**      | [`$redact`]()表达式允许的结果之一。                          |
| [PRUNE]()        | **$$PRUNE**        | [`$redact`]()表达式允许的结果之一。                          |
| [KEEP]()         | **$$KEEP**         | [`$redact`]()表达式允许的结果之一。                          |

有关这些变量的更详细描述，请参阅[系统变量]()。

[]()
[]()

### Literals

Literals 可以是任何类型。但是，MongoDB将以美元符号`$`开头的字符串字面值作为字段的路径，并将表达式对象中的数值/布尔字面值作为投影标志。为了避免解析文字，可以使用[$literal]()表达式。
[]()
[]()

### 表达式对象

表达式对象具有以下形式：

```powershell
{ <field1>: <expression1>, ... }
```

如果表达式是数值型或 boolean 型文字，MongoDB 将 literals 视为投影标志(例如： `1`或`true`包括该字段)，仅在[$project]()阶段有效。要避免将数值或 boolean 型文字视为投影标志，请使用[$literal]()表达式来包装数值型或 boolean 文字型。
[]()
[]()

## <span id="operator-expressions">运算符表达式</span>

[]()
在这个部分

*   [算数表达式运算符](#arithmetic-expression-operators)
*   [数组表达式运算符](#array-expression-operators)
*   [布尔表达式运算符](#boolean-expression-operators)
*   [比较表达式运算符](#comparison-expression-operators)
*   [条件表达式运算符](#conditional-expression-operators)
*   [自定义聚合表达式运算符](#custom-aggregation-expression-operators)
*   [数据大小表达式运算符](#data-size-expression-operators)
*   [日期表达式运算符](#date-expression-operators)
*   [文字表达式运算符](#literal-expression-operator)
*   [对象表达式运算符](#object-expression-operators)
*   [集合表达式运算符](#set-expression-operators)
*   [字符串表达式运算符](#string-expression-operators)
*   [文本表达式运算符](#text-expression-operator)
*   [角度表达式运算符](#trigonometry-expression-operators)
*   [类型表达式运算符](#type-expression-operators)
*   [累加器($group)](#accumulators-group)
*   [累加器($project 和$addFields)](#accumulators-project-addfields)
*   [变量表达式运算符](#variable-expression-operators)

运算符表达式与采用带参数的函数类似。通常，这些表达式有一个数组参数 并具有以下形式：

```powershell
{ <operator>: [ <argument1>, <argument2> ... ] }
```

如果操作符接受单个参数，则可以省略指定参数列表的外部数组：

```powershell
{ <operator>: <argument> }
```

如果参数是文字数组，为了避免解析歧义，必须将文字数组包装在[$literal]()表达式中，或者保留指定参数列表的外部数组。

[]()

### <span id="arithmetic-expression-operators">算数表达式运算符</span>

算术表达式对数字执行数学运算。一些算术表达式也可以支持日期算术。

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$abs](../../Reference/Operators/Aggregation-Pipeline-Operators/abs-aggregation.md) | 返回数字的绝对值。                                           |
| [$add](../../Reference/Operators/Aggregation-Pipeline-Operators/add-aggregation.md) | 添加 numbers 以返回总和，或添加 numbers 和 date 以返回新的 date。如果添加 numbers 和 date，则将 numbers 视为毫秒。接受任意数量的参数表达式，但最多只能有一个表达式解析为 date。 |
| [$ceil](../../Reference/Operators/Aggregation-Pipeline-Operators/ceil-aggregation.md) | 返回大于或等于指定数字的最小整数。                           |
| [$divide](../../Reference/Operators/Aggregation-Pipeline-Operators/divide-aggregation.md) | 返回将第一个数除以第二个数的结果。接受两个参数表达式。       |
| [$exp](../../Reference/Operators/Aggregation-Pipeline-Operators/exp-aggregation.md) | 将 e 提高到指定的指数。                                      |
| [$floor](../../Reference/Operators/Aggregation-Pipeline-Operators/floor-aggregation.md) | 返回小于或等于指定数字的最大整数。                           |
| [$ln](../../Reference/Operators/Aggregation-Pipeline-Operators/ln-aggregation.md) | 计算数字的自然对数。                                         |
| [$log](../../Reference/Operators/Aggregation-Pipeline-Operators/log-aggregation.md) | 计算指定基数中的数字的对数。                                 |
| [$log10](../../Reference/Operators/Aggregation-Pipeline-Operators/log10-aggregation.md) | 计算以10为底的对数。                                         |
| [$mod](../../Reference/Operators/Aggregation-Pipeline-Operators/mod-aggregation.md) | 返回第一个数字除以第二个数字的余数。接受两个参数表达式。     |
| [$multiply](../../Reference/Operators/Aggregation-Pipeline-Operators/multiply-aggregation.md) | 将数字相乘返回乘积。接受任意数量的参数表达式。               |
| [$pow](../../Reference/Operators/Aggregation-Pipeline-Operators/pow-aggregation.md) | 将数字提高到指定的指数。                                     |
| [$round](../../Reference/Operators/Aggregation-Pipeline-Operators/round-aggregation.md) | 将数字四舍五入为整数或指定的小数位。                         |
| [$sqrt](../../Reference/Operators/Aggregation-Pipeline-Operators/sqrt-aggregation.md) | 计算平方根。                                                 |
| [$subtract](../../Reference/Operators/Aggregation-Pipeline-Operators/subtract-aggregation.md) | 返回从第一个值中减去第二个值的结果。如果这两个值是数字，返回差值。如果这两个值是日期，则返回差值(以毫秒为单位)。如果这两个值是日期和一个以毫秒为单位的数字，返回结果日期。接受两个参数表达式。如果这两个值是日期和数字，请首先指定 date 参数，因为从数字中减去 date 没有意义。 |
| [$trunc](../../Reference/Operators/Aggregation-Pipeline-Operators/trunc-aggregation.md) | 将数字截断为整数或指定的小数位。                             |

[]()

### <span id="array-expression-operators">数组表达式运算符</span>

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$arrayElemAt](../../Reference/Operators/Aggregation-Pipeline-Operators/arrayElemAt-aggregation.md) | 返回指定的数组索引处的元素。                                 |
| [$arrayToObject](../../Reference/Operators/Aggregation-Pipeline-Operators/arrayToObject-aggregation.md) | 将键值对的数组转换为文档。                                   |
| [$concatArrays](../../Reference/Operators/Aggregation-Pipeline-Operators/concatArrays-aggregation.md) | 连接数组以返回连接的数组。                                   |
| [$filter](../../Reference/Operators/Aggregation-Pipeline-Operators/filter-aggregation.md) | 选择 array 的子集以 return array 仅包含 match 过滤条件的元素。 |
| [$first](../../Reference/Operators/Aggregation-Pipeline-Operators/first-aggregation.md) | 返回第一个数组元素，不同于[$first]()累加器                   |
| [$in](../../Reference/Operators/Aggregation-Pipeline-Operators/in-aggregation.md) | 返回一个 boolean 值，指示指定的值是否在列表中。              |
| [$indexOfArray](../../Reference/Operators/Aggregation-Pipeline-Operators/indexOfArray-aggregation.md) | 搜索列表以查找指定值的出现并返回第一个匹配项的数组索引。如果未找到子字符串，则返回`-1`。 |
| [$isArray](../../Reference/Operators/Aggregation-Pipeline-Operators/isArray-aggregation.md) | 确定操作数是否为数组。返回 boolean 值。                      |
| [$last](../../Reference/Operators/Aggregation-Pipeline-Operators/last-aggregation.md) | 返回最后一个数组元素，不同于[$last]()累加器。                |
| [$map](../../Reference/Operators/Aggregation-Pipeline-Operators/map-aggregation.md) | 将子表达式应用于数组的每个元素，并按顺序返回结果值的数组。接受命名参数。 |
| [$objectToArray](../../Reference/Operators/Aggregation-Pipeline-Operators/objectToArray-aggregation.md) | 将文档转换为表示键值对的文档的数组。                         |
| [$range](../../Reference/Operators/Aggregation-Pipeline-Operators/range-aggregation.md) | 根据用户定义的输入输出包含整数序列的数组。                   |
| [$reduce](../../Reference/Operators/Aggregation-Pipeline-Operators/reduce-aggregation.md) | 将表达式应用于数组中的每个元素，并将它们组合为单个值。       |
| [$reverseArray](../../Reference/Operators/Aggregation-Pipeline-Operators/reverseArray-aggregation.md) | 返回元素顺序相反的数组。                                     |
| [$size](../../Reference/Operators/Aggregation-Pipeline-Operators/size-aggregation.md) | 返回数组中的元素数。接受单个表达式作为参数。                 |
| [$slice](../../Reference/Operators/Aggregation-Pipeline-Operators/slice-aggregation.md) | 返回数组的子集。                                             |
| [$zip](../../Reference/Operators/Aggregation-Pipeline-Operators/zip-aggregation.md) | 将两个数组合并在一起。                                       |

[]()

### <span id="boolean-expression-operators">布尔表达式运算符</span>

Boolean 表达式将其参数表达式计算为布尔值，并返回一个boolean值作为结果。

除了`false` 布尔值之外，Boolean 表达式的计算结果如下：`null`，`0`和`undefined`值。 Boolean 表达式将所有其他值计算为`true`，包括非零数值和数组。

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$and](../../Reference/Operators/Aggregation-Pipeline-Operators/and-aggregation.md) | 仅当其所有表达式求值为`true`时才返回`true`。接受任意数量的参数表达式。 |
| [$not](../../Reference/Operators/Aggregation-Pipeline-Operators/not-aggregation.md) | 返回与其参数表达式相反的 boolean 值。接受单个参数表达式。    |
| [$or](../../Reference/Operators/Aggregation-Pipeline-Operators/or-aggregation.md) | 当其表达式的值为`true`时返回`true`。接受任意数量的参数表达式。 |

[]()

### <span id="comparison-expression-operators">比较表达式运算符</span>

比较表达式返回一个布尔值，除了[$cmp]()，它返回一个数字。

比较表达式采用两个参数表达式并对值和类型进行比较，使用[指定的 BSON 比较顺序]()表示不同类型的值。

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$cmp](../../Reference/Operators/Aggregation-Pipeline-Operators/cmp-aggregation.md) | 如果两个值相等则返回`0`，如果第一个 value 大于第二个值则返回`1`，如果第一个值小于第二个值，则返回`-1`。 |
| [$eq](../../Reference/Operators/Aggregation-Pipeline-Operators/eq-aggregation.md) | 如果值相等，则返回`true`。                                   |
| [$gt](../../Reference/Operators/Aggregation-Pipeline-Operators/gt-aggregation.md) | 如果第一个值大于第二个，则返回`true`。                       |
| [$gte](../../Reference/Operators/Aggregation-Pipeline-Operators/gte-aggregation.md) | 如果第一个值大于或等于第二个，则返回`true`。                 |
| [$lt](../../Reference/Operators/Aggregation-Pipeline-Operators/lt-aggregation.md) | 如果第一个值小于第二个，则返回`true`。                       |
| [$lte](../../Reference/Operators/Aggregation-Pipeline-Operators/lte-aggregation.md) | 如果第一个值小于或等于第二个值，则返回`true`。               |
| [$ne](../../Reference/Operators/Aggregation-Pipeline-Operators/ne-aggregation.md) | 如果值不相等，则返回`true`。                                 |

[]()

### <span id="conditional-expression-operators">条件表达式运算符</span>

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$cond](../../Reference/Operators/Aggregation-Pipeline-Operators/cond-aggregation.md) | 对一个表达式求值的三元运算符，并根据结果返回另外两个表达式之一的值。接受有序列表中的三个表达式或三个命名参数。 |
| [$ifNull](../../Reference/Operators/Aggregation-Pipeline-Operators/ifNull-aggregation.md) | 返回第一个表达式的非空结果，如果第一个表达式的结果为空，则返回第二个表达式的结果。Null结果包含未定义值或缺少字段的实例。接受两个表达式作为参数。第二个表达式的结果可以为null。 |
| [$switch](../../Reference/Operators/Aggregation-Pipeline-Operators/switch-aggregation.md) | 计算一系列用例表达。当它找到一个计算结果为`true`的表达式时，`$switch`执行一个指定的表达式并跳出控制流。 |

[]()

### <span id="custom-aggregation-expression-operators">自定义聚合表达式运算符</span>

| 名称                                                         | 描述                                             |
| ------------------------------------------------------------ | ------------------------------------------------ |
| [$accumulator](../../Reference/Operators/Aggregation-Pipeline-Operators/accumulator-aggregation.md) | 定义一个自定义累加器函数<br />version 4.4 新功能 |
| [$function](../../Reference/Operators/Aggregation-Pipeline-Operators/function-aggregation.md) | 定义一个自定义函数<br />version 4.4 新功能       |

[]()

### <span id="data-size-expression-operators">数据大小表达式运算符</span>

以下运算符返回数据元素的大小:

| 名称                                                         | 描述                                                     |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| [$binarySize](../../Reference/Operators/Aggregation-Pipeline-Operators/binarySize-aggregation.md) | 返回给定字符串或二进制数据值内容的字节大小。             |
| [$bsonSize](../../Reference/Operators/Aggregation-Pipeline-Operators/bsonSize-aggregation.md) | 返回编码为BSON的给定文档(例如：bsontype对象)的字节大小。 |

[]()

### <span id="date-expression-operators">日期表达式运算符</span>

以下操作符返回 date 对象或 date 对象的组件：

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$dateFromParts](../../Reference/Operators/Aggregation-Pipeline-Operators/dateFromParts-aggregation.md) | 给出日期的组成部分，构造一个 BSON Date 对象。                |
| [$dateFromString](../../Reference/Operators/Aggregation-Pipeline-Operators/dateFromString-aggregation.md) | 将 date/time 字符串转换为 date 对象。                        |
| [$dateToParts](../../Reference/Operators/Aggregation-Pipeline-Operators/dateToParts-aggregation.md) | 返回包含日期组成部分的文档。                                 |
| [$dateToString](../../Reference/Operators/Aggregation-Pipeline-Operators/dateToString-aggregation.md) | 将 date 作为格式化的字符串返回。                             |
| [$dayOfMonth](../../Reference/Operators/Aggregation-Pipeline-Operators/dayOfMonth-aggregation.md) | 将 date 的月中某天返回为 1 到 31 之间的数字。                |
| [$dayOfWeek](../../Reference/Operators/Aggregation-Pipeline-Operators/dayOfWeek-aggregation.md) | 将 date 的星期几返回为 1(星期日)和 7(星期六)之间的数字。     |
| [$dayOfYear](../../Reference/Operators/Aggregation-Pipeline-Operators/dayOfYear-aggregation.md) | 将 date 的年中日期作为 1 到 366(闰年)之间的数字返回。        |
| [$hour](../../Reference/Operators/Aggregation-Pipeline-Operators/hour-aggregation.md) | 将 date 的小时数作为 0 到 23 之间的数字返回。                |
| [$isoDayOfWeek](../../Reference/Operators/Aggregation-Pipeline-Operators/hour-aggregation.md) | 返回 ISO 8601 格式的工作日编号，范围从`1`(星期一)到`7`(星期日)。 |
| [$isoWeek](../../Reference/Operators/Aggregation-Pipeline-Operators/isoWeek-aggregation.md) | 返回 ISO 8601 格式的周数，范围从`1`到`53`。 Week numbers 从`1`开始，周(星期一到星期日)包含年份的第一个星期四。 |
| [$isoWeekYear](../../Reference/Operators/Aggregation-Pipeline-Operators/isoWeekYear-aggregation.md) | 以 ISO 8601 格式返回年份编号。年份从第 1 周的星期一(ISO 8601)开始，结束于上周的星期日(ISO 8601)。 |
| [$millisecond](../../Reference/Operators/Aggregation-Pipeline-Operators/millisecond-aggregation.md) | 返回 date 的毫秒数，作为 0 到 999 之间的数字。               |
| [$minute](../../Reference/Operators/Aggregation-Pipeline-Operators/minute-aggregation.md) | 将 date 的分钟作为 0 到 59 之间的数字返回。                  |
| [$month](../../Reference/Operators/Aggregation-Pipeline-Operators/month-aggregation.md) | 将 date 的月份返回为 1(1 月)和 12(12 月)之间的数字。         |
| [$second](../../Reference/Operators/Aggregation-Pipeline-Operators/second-aggregation.md) | 返回 date 的秒数，作为 0 到 60 之间的数字(闰秒)。            |
| [$toDate](../../Reference/Operators/Aggregation-Pipeline-Operators/toDate-aggregation.md) | 将值转换为日期。<br />version 4.0 中的新功能。               |
| [$week](../../Reference/Operators/Aggregation-Pipeline-Operators/week-aggregation.md) | 返回日期的周数，该数字介于0(一年的第一个星期日之前的部分周)和53(闰年)之间。 |
| [$year](../../Reference/Operators/Aggregation-Pipeline-Operators/year-aggregation.md) | 将日期的年份作为数字返回(例： 2014)。                        |

以下算术运算符可以使用日期操作数：

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$add](../../Reference/Operators/Aggregation-Pipeline-Operators/add-aggregation.md) | 添加数字和日期以返回新的日期。如果添加数字和日期，则将这些数字视为毫秒。接受任意数量的参数表达式，但一个表达式最多只能解析一个日期。 |
| [$subtract](../../Reference/Operators/Aggregation-Pipeline-Operators/subtract-aggregation.md) | 返回从第一个值减去第二个值的结果。如果这两个值是日期，则返回差值(以毫秒为单位)。如果这两个值是日期和一个以毫秒为单位的数字，则返回结果日期。接受两个参数表达式。如果这两个值是日期和数字，请首先指定日期参数，因为从数字中减去日期没有意义。 |

[]()

### <span id="literal-expression-operator">文字表达式运算符</span>

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$literal](../../Reference/Operators/Aggregation-Pipeline-Operators/literal-aggregation.md) | 返回一个不需要解析的值。用于聚合管道可解释为表达式的值。例如，将[$literal]()表达式用于以`$`开头的 string，以避免解析为字段路径。 |

[]()

### <span id="object-expression-operators">对象表达式运算符</span>

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$mergeObjects](../../Reference/Operators/Aggregation-Pipeline-Operators/mergeObjects-aggregation.md) | 将多个文档合并为一个文档。 <br/> version 3.6 中的新内容。    |
| [$objectToArray](../../Reference/Operators/Aggregation-Pipeline-Operators/objectToArray-aggregation.md) | 将文档转换为表示键值对的文档的数组。 <br/> version 3.6 中的新内容。 |

[]()

### <span id="set-expression-operators">集合表达式运算符</span>

Set 表达式对数组执行 set 操作，将数组视为集合。 Set 表达式忽略每个输入数组中的重复条目和元素的顺序。

如果 set 操作返回一个集合，则该操作会过滤掉结果中的重复项，以输出仅包含唯一条目的数组。输出数组中元素的顺序未指定。

如果集合包含嵌套的数组元素，则 set 表达式不会深入到嵌套的数组中，而是在最外层处计算数组。

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$allElementsTrue](../../Reference/Operators/Aggregation-Pipeline-Operators/allElementsTrue-aggregation.md) | 如果没有集合的元素计算为`false`，则返回`true`，否则返回`false`。接受单个参数表达式。 |
| [$anyElementTrue](../../Reference/Operators/Aggregation-Pipeline-Operators/anyElementTrue-aggregation.md) | 如果集合中的任意一个元素求值为`true`，则返回`true`；否则，返回`false`。接受单个参数表达式。 |
| [$setDifference](../../Reference/Operators/Aggregation-Pipeline-Operators/setDifference-aggregation.md) | 返回一个集合，其中的元素出现在第一个集合中但不出现在第二个集合中；即：执行第二个集合相对于第一个集合的[相对补充]()。接受两个参数表达式。 |
| [$setEquals](../../Reference/Operators/Aggregation-Pipeline-Operators/setEquals-aggregation.md) | 如果输入 sets 具有相同的不同元素，则返回`true`。接受两个或多个参数表达式。 |
| [$setIntersection](../../Reference/Operators/Aggregation-Pipeline-Operators/setIntersection-aggregation.md) | 返回一个包含所有输入 sets 中出现的元素的集合。接受任意数量的参数表达式。 |
| [$setIsSubset](../../Reference/Operators/Aggregation-Pipeline-Operators/setIsSubset-aggregation.md) | 如果第一组的所有元素出现在第二组中，包括第一个集合和第二个集合相等时，则返回`true`；即：不是[严格的子集](http://en.wikipedia.org/wiki/Subset)。接受两个参数表达式。 |
| [$setUnion](../../Reference/Operators/Aggregation-Pipeline-Operators/setUnion-aggregation.md) | 返回包含出现在任何输入集合中的元素的集合。                   |


[]()

### <span id="string-expression-operators">字符串表达式运算符</span>

除了[$concat]()之外，字符串表达式只对ASCII字符的字符串具有定义良好的行为。

无论使用哪个字符，[$concat]()行为都是定义良好的。

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [$concat]()         | 连接任意数量的 strings。                                     |
| [$dateFromString]() | 将 date/time string 转换为 date object。                     |
| [$dateToString]()   | 将 date 作为格式化的 string 返回。                           |
| [$indexOfBytes]()   | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 字节索引。如果未找到子字符串，则返回`-1`。 |
| [$indexOfCP]()      | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 code 点索引。如果找不到子字符串，则返回`-1` |
| [$split]()          | 根据分隔符将 string 拆分为子字符串。返回子字符串的 array。如果在 string 中找不到分隔符，则返回包含原始 string 的 array。 |
| [$strLenBytes]()    | 返回 string 中 UTF-8 编码字节的数量。                        |
| [$strLenCP]()       | 返回 string 中 UTF-8 [code 点](http://www.unicode.org/glossary/#exp._S_strLenBytes)的数量。 |
| [$strcasecmp]()     | 执行 case-insensitive string 比较并返回：如果两个 strings 相等则返回`0`，如果第一个 string 大于第二个，则返回`1`，如果第一个 string 小于第二个，则返回`-1`。 |
| [$substr]()         | 已过时。使用[$substrBytes]()或[$substrCP]()。                |
| [$substrBytes]()    | 返回 string 的子字符串。从 string 中指定的 UTF-8 字节索引(zero-based)处的字符开始，并继续指定的字节数。 |
| [$substrCP]()       | 返回 string 的子字符串。从 string 中指定的 UTF-8 [code point(CP)](http://www.unicode.org/glossary/#exp._S_substrBytes)索引(zero-based)处的字符开始，并继续指定的 code 点数。 |
| [$toLower]()        | 将 string 转换为小写。接受单个参数表达式。                   |
| [$toUpper]()        | 将 string 转换为大写。接受单个参数表达式。                   |

[]()

### <span id="text-expression-operator">文本表达式运算符</span>

| 名称      | 描述                 |
| --------- | -------------------- |
| [$meta]() | 访问文本搜索元数据。 |

[]()

### <span id="trigonometry-expression-operators">角度表达式运算符</span>

| 名称      | 描述                         |
| --------- | ---------------------------- |
| [$type]() | 返回该字段的 BSON 数据类型。 |


[](s

### <span id="accumulators-group">累加器($group)</span>

可以在[$group]()阶段使用，累加器是 operators，它们在文档通过管道时保持其 state(例： 总计，最大值，最小值和相关数据)。

当在[$group]()阶段用作累加器时，这些 operators 将单个表达式作为输入，为每个输入文档计算一次表达式，并为共享相同 group key 的 group 文档保持其阶段。

| 名称              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| [$addToSet]()     | 返回每个 group 的唯一表达式值的 array。 _Oray 元素的 Order 是未定义的。 |
| [$avg]()          | 返回数值的平均值。忽略 non-numeric 值。                      |
| [$first]()        | 从每个 group 的第一个文档返回一个 value。仅当文档位于已定义的 order 中时才定义 Order。 |
| [$last]()         | 从每个 group 的最后一个文档返回一个 value。仅当文档位于已定义的 order 中时才定义 Order。 |
| [$max]()          | 返回每个 group 的最高表达式 value。                          |
| [$mergeObjects]() | 返回通过组合每个 group 的输入文档创建的文档。                |
| [$min]()          | 返回每个 group 的最低表达式 value。                          |
| [$push]()         | 返回每个 group 的表达式值的 array。                          |
| [$stdDevPop]()    | 返回输入值的总体标准偏差。                                   |
| [$stdDevSamp]()   | 返回输入值的 sample 标准偏差。                               |
| [$sum]()          | 返回数值的总和。忽略 non-numeric 值。                        |

[]()

### <span id="accumulators-project-addfields">累加器($project 和$addFields)</span>

一些可用作[$group]()阶段累加器的运算符也可用于[$project]()和[$addFields]()阶段，但不能用作累加器。在[$project]()和[$addFields]()阶段使用时，这些 operators 不会维护它们的 state，并且可以将单个参数或多个 arguments 作为输入。

更改了 version 3.2.

以下累加器 operators 也可用于[$project]()和[$addFields]()阶段。

| 名称            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| [$avg]()        | 返回每个文档的指定表达式或表达式列表的平均值。忽略 non-numeric 值。 |
| [$max]()        | 返回每个文档的指定表达式或表达式列表的最大值                 |
| [$min]()        | 返回每个文档的指定表达式或表达式列表的最小值                 |
| [$stdDevPop]()  | 返回输入值的总体标准偏差。                                   |
| [$stdDevSamp]() | 返回输入值的 sample 标准偏差。                               |
| [$sum]()        | 返回数值的总和。忽略 non-numeric 值。                        |


[]()

### <span id="variable-expression-operators">变量表达式运算符</span>

| 名称     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| [$let]() | 定义在子表达式范围内使用的变量，并返回子表达式的结果。接受命名参数。 <br/>接受任意数量的参数表达式。 |


[]()

## <span id="index-of-expression-operators">表达式运算符的索引</span>

| <br />                                                       |                                                              |                                                              |                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [$abs]()<br/>[$add]()<br/>[$addToSet]()<br/>[$allElementsTrue]()<br/>[$and]()<br/>[$anyElementTrue]()<br/>[$arrayElemAt]()<br/>[$arrayToObject]()<br/>[$avg]()<br/>[$cmp]()<br/>[$concat]()<br/>[$concatArrays]()<br/>[$cond]()<br/>[$dateFromParts]()<br/>[$dateToParts]()<br/>[$dateFromString]()<br/>[$dateToString]() | [$dayOfMonth]()<br/>[$dayOfWeek]()<br/>[$dayOfYear]()<br/>[$divide]()<br/>[$eq]()<br/>[$exp]()<br/>[$filter]()<br/>[$first]()<br/>[$floor]()<br/>[$gt]()<br/>[$gte]()<br/>[$hour]()<br/>[$ifNull]()<br/>[$in]()<br/>[$indexOfArray]()<br/>[$indexOfBytes]()<br/>[$indexOfCP]()<br/>[$isArray]() | [$isoDayOfWeek]()<br/>[$isoWeek]()<br/>[$isoWeekYear]()<br/>[$last]()<br/>[$let]()<br/>[$literal]()<br/>[$ln]()<br/>[$log]()<br/>[$log10]()<br/>[$lt]()<br/>[$lte]()<br/>[$map]()<br/>[$max]()<br/>[$mergeObjects]()<br/>[$meta]()<br/>[$min]()<br/>[$millisecond]() | [$minute]()<br/>[$mod]()<br/>[$month]()<br/>[$multiply]()<br/>[$ne]()<br/>[$not]()<br/>[$objectToArray]()<br/>[$or]()<br/>[$pow]()<br/>[$push]()<br/>[$range]()<br/>[$reduce]()<br/>[$reverseArray]()<br/>[$second]()<br/>[$setDifference]()<br/>[$setEquals]()<br/>[$setIntersection]()<br/>[$setIsSubset]()<br/>[$setUnion]()<br/>[$size]() | [$slice]()<br/>[$split]()<br/>[$sqrt]()<br/>[$stdDevPop]()<br/>[$stdDevSamp]()<br/>[$strcasecmp]()<br/>[$strLenBytes]()<br/>[$strLenCP]()<br/>[$substr]()<br/>[$substrBytes]()<br/>[$substrCP]()<br/>[$subtract]()<br/>[$sum]()<br/>[$switch]()<br/>[$toLower]()<br/>[$toUpper]()<br/>[$trunc]()<br/>[$type]()<br/>[$week]()<br/>[$year]()<br/>[$zip]() |



译者：李冠飞

校对：