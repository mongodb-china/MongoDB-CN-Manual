# [ ](#)聚合管道操作符
> **注意：**
>
> 有关特定运算符的详细信息，包括语法和示例，请单击特定运算符以转到其参考页。

[]()

[]()

## 表达式运算符

[]()

在这个部分

*   [算术表达式运算符](#arithmetic-expression-operators)
*   [列表表达式运算符](#array-expression-operators)
*   [布尔表达式运算符](#boolean-expression-operators)
*   [比较表达式运算符](#comparison-expression-operators)
*   [条件表达式运算符](#conditional-expression-operators)
*   [日期表达式运算符](#date-expression-operators)
*   [文字表达式运算符](#literal-expression-operator)
*   [对象表达式运算符](#object-expression-operators)
*   [集合表达式运算符](#set-expression-operators)
*   [字符串表达式运算符](#string-expression-operators)
*   [文本表达式运算符](#text-expression-operator)
*   [三角表达式运算符](#trigonometry-expression-operators)
*   [类型表达式运算符](#type-expression-operators)
*   [累加器($group)](#accumulators-group)
*   [累加器(处于其他阶段)](#accumulators-in-other-stages)
*   [变量表达式运算符](#variable-expression-operators)

这些表达式运算符可用于构造[表达式](../../Aggregation/Aggregation-Reference/Aggregation-Pipeline-Quick-Reference.md#表达式)以在[聚合管道阶段](Aggregation-Pipeline-Stages.md)中使用。

运算符表达式类似于带有参数的函数。通常，这些表达式采用参数数组并具有以下形式：

```powershell
{ <operator>: [ <argument1>, <argument2> ... ] }
```


如果 operator 接受单个参数，可以省略指定参数列表的外部数组：

```powershell
{ <operator>: <argument> }
```

为了避免在参数是文字数组的情况下解析歧义，必须将文字数组包装在[`$literal`](Aggregation-Pipeline-Operators/literal-aggregation.md)表达式中，或者保留指定参数列表的外部数组。

[]()

### <span id="arithmetic-expression-operators">算术表达式运算符</span>

算术表达式对 numbers 执行数学运算。一些算术表达式也可以支持 date 算术。

| 名称                                                        | 描述                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [$abs](Aggregation-Pipeline-Operators/abs-aggregation.md)   | 返回数字的绝对 value。                                       |
| [$add](Aggregation-Pipeline-Operators/add-aggregation.md)   | 添加 numbers 以 return 总和，或添加 numbers 和 date 以 return 新的 date。如果添加 numbers 和 date，则将 numbers 视为毫秒。接受任意数量的参数表达式，但最多只能有一个表达式解析为 date。 |
| [$ceil](Aggregation-Pipeline-Operators/ceil-aggregation.md) | 返回大于或等于指定数字的最小 integer。                       |
| [$divide]()                                                 | 返回将第一个数除以第二个数的结果。接受两个参数表达式。       |
| [$exp]()                                                    | 将 e 提高到指定的指数。                                      |
| [$floor]()                                                  | 返回小于或等于指定数字的最大 integer。                       |
| [$ln]()                                                     | 计算数字的自然 log。                                         |
| [$log]()                                                    | 计算指定基数中的数字的 log。                                 |
| [$log10]()                                                  | 计算数字的 log 基数 10。                                     |
| [$mod]()                                                    | 返回第一个数字的余数除以第二个数字。接受两个参数表达式。     |
| [$multiply]()                                               | 将 numbers 乘以_return 产品。接受任意数量的参数表达式。      |
| [$pow]()                                                    | 将数字提高到指定的指数。                                     |
| [$round]()                                                  | 将数字四舍五入为整数*或*指定的小数位。                       |
| [$sqrt]()                                                   | 计算平方根。                                                 |
| [$subtract]()                                               | 返回从第一个中减去第二个 value 的结果。如果这两个值是 numbers，返回差异。如果这两个值是日期，则返回差异(以毫秒为单位)。如果这两个值是 date 和一个以毫秒为单位的数字，_return 结果 date。接受两个参数表达式。如果这两个值是 date 和数字，请首先指定 date 参数，因为从数字中减去 date 没有意义。 |
| [$trunc]()                                                  | 截断其整数的数字。                                           |

[]()

### <span id="array-expression-operators">列表表达式运算符</span>

| 名称               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| [$arrayElemAt]()   | 返回指定的数组索引处的元素。                                 |
| [$arrayToObject]() | 将键值对的数组转换为文档。                                   |
| [$concatArrays]()  | 连接数组以返回连接的数组。                                   |
| [$filter]()        | 选择数组的子集以返回仅包含与过滤条件匹配的元素的数组。       |
| [$in]()            | 返回一个布尔值，指示指定的值是否在数组中。                   |
| [$indexOfArray]()  | 在数组中搜索指定值的出现，并返回第一个出现的数组索引。如果未找到子字符串，则返回`-1`。 |
| [$isArray]()       | 确定操作数是否为数组。返回一个布尔值。                       |
| [$map]()           | 对数组的每个元素应用子表达式，并按顺序返回结果值的数组。接受命名参数。 |
| [$objectToArray]() | 将文档转换为代表键值对的文档数组。                           |
| [$range]()         | 根据用户定义的输入输出包含整数序列的数组。                   |
| [$reduce]()        | 将表达式应用于数组中的每个元素，并将它们组合为单个值。       |
| [$reverseArray]()  | 返回具有相反顺序元素的数组。                                 |
| [$size]()          | 返回数组中元素的数量。接受单个表达式作为参数。               |
| [$slice]()         | 返回数组的子集。                                             |
| [$zip]()           | 合并两个数组。                                               |

[]()

### <span id="boolean-expression-operators">布尔表达式运算符</span>

布尔表达式将其参数表达式计算为布尔值，并返回布尔值作为结果。

除了`false`布尔值，布尔表达式为`false`如下：`null`，`0`，和`undefined` 的值。布尔表达式将所有其他值评估为`true`，包括非零数字值和数组。

| 名称     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| [$and]() | 仅当其所有表达式求值为`true`时才返回`true`。接受任意数量的参数表达式。 |
| [$not]() | 返回与其参数表达式相反的 boolean value。接受单个参数表达式。 |
| [$or]()  | 当任何表达式求值为`true`时返回`true`。接受任意数量的参数表达式。 |


[]()

### <span id="comparison-expression-operators">比较表达式运算符</span>

比较表达式返回一个布尔值，但[$cmp](reference-operator-aggregation-cmp.html#exp._S_cmp)返回一个数字。

比较表达式采用两个参数表达式并比较 value 和 type，使用[指定的 BSON 比较顺序](reference-bson-type-comparison-order.html#bson-types-comparison-order)表示不同类型的值。

| 名称     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| [$cmp]() | 如果两个值相等则返回`0`，如果第一个 value 大于第二个值则返回`1`，如果第一个 value 小于第二个值，则返回`-1`。 |
| [$eq]()  | 如果值相等，则返回`true`。                                   |
| [$gt]()  | 如果第一个 value 大于第二个，则返回`true`。                  |
| [$gte]() | 如果第一个 value 大于或等于第二个，则返回`true`。            |
| [$lt]()  | 如果第一个 value 小于第二个，则返回`true`。                  |
| [$lte]() | 如果第一个 value 小于或等于第二个值，则返回`true`。          |
| [$ne]()  | 如果值不相等，则返回`true`。                                 |


[]()

### <span id="conditional-expression-operators">条件表达式运算符</span>

| 名称        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| [$cond]()   | 一个三元运算符，它计算一个表达式，并根据结果返回另外两个表达式之一的值。接受有序列表中的三个表达式或三个命名参数。 |
| [$ifNull]() | 如果第一个表达式导致结果为null ，则返回第一个表达式的非空结果或第二个表达式的结果。空结果包含未定义值或缺少字段的实例。接受两个表达式作为参数。第二个表达式的结果可以为 null。 |
| [$switch]() | 计算一系列案例表达式。当它找到一个计算结果为`true`的表达式时，`$switch`执行一个指定的表达式并退出控制流。 |

[]()

### <span id="date-expression-operators">日期表达式运算符</span>

以下运算符返回日期对象或日期对象的组成部分：

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [$dateFromParts]()  | 给出日期的组成部分，构造一个 BSON Date对象。                 |
| [$dateFromString]() | 将 date/time 字符串转换为 date 对象。                        |
| [$dateToParts]()    | 返回包含 date 组成部分的文档。                               |
| [$dateToString]()   | 将 date 作为格式化的 string 返回。                           |
| [$dayOfMonth]()     | 将 date 的月中某天返回为 1 到 31 之间的数字。                |
| [$dayOfWeek]()      | 将 date 的星期几返回为 1(星期日)和 7(星期六)之间的数字。     |
| [$dayOfYear]()      | 将 date 的年中日期作为 1 到 366(闰年)之间的数字返回。        |
| [$hour]()           | 将 date 的小时数作为 0 到 23 之间的数字返回。                |
| [$isoDayOfWeek]()   | 返回 ISO 8601 格式的工作日编号，范围从`1`(星期一)到`7`(星期日)。 |
| [$isoWeek]()        | 返回 ISO 8601 格式的周数，范围从`1`到`53`。 星期数从`1`开始，周(星期一到星期日)包含年份的第一个星期四。 |
| [$isoWeekYear]()    | 以 ISO 8601 格式返回年份编号。年份从第 1 周的星期一(ISO 8601)开始，结束于上周的星期日(ISO 8601)。 |
| [$millisecond]()    | 返回 date 的毫秒数，作为 0 到 999 之间的数字。               |
| [$minute]()         | 将 date 的分钟作为 0 到 59 之间的数字返回。                  |
| [$month]()          | 将 date 的月份返回为 1(1 月)和 12(12 月)之间的数字。         |
| [$second]()         | 返回 date 的秒数，作为 0 到 60 之间的数字(闰秒)。            |
| [$toDate]()         | 将值转换为日期。<br />*版本4.0中的新功能。*                  |
| [$week]()           | 返回 date 的周数，作为 0(在一年的第一个星期日之前的部分周)和 53(闰年)之间的数字。 |
| [$year]()           | 将 date 的年份作为数字返回(例：2014)。                       |

以下算术运算符可以使用 date 操作数：

| 名称          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| [$add]()      | 添加 numbers 和 date 以返回新的 date。如果添加 numbers 和 date，则将 numbers 视为毫秒。接受任意数量的参数表达式，但最多只能有一个表达式解析为 date。 |
| [$subtract]() | 返回从第一个中减去第二个值的结果。如果这两个值是日期，则返回差异(以毫秒为单位)。如果这两个值是 date 和一个以毫秒为单位的数字，返回结果 date。接受两个参数表达式。如果这两个值是 date 和数字，请首先指定 date 参数，因为从数字中减去 date 没有意义。 |


[]()

### <span id="literal-expression-operator">文字表达式运算符</span>

| 名称         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| [$literal]() | 无需解析即可返回 value。用于聚合管道可以将其解释为表达式的值。例如，对以$开头的字符串使用[]$literal]()表达式，以避免解析为字段路径。 |


[]()

### <span id="object-expression-operators">对象表达式运算符</span>

| 名称               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| [$mergeObjects]()  | 将多个文档合并为一个文档。 <br/> version 3.6 中的新内容。    |
| [$objectToArray]() | 将文档转换为表示 key-value 对的文档的 array。 <br/> version 3.6 中的新内容。 |


[]()

### <span id="set-expression-operators">集合表达式运算符</span>

Set 表达式对数组执行 set 操作，将数组视为 sets。 Set 表达式忽略每个输入数组中的重复条目和元素的顺序。

如果 set 操作返回一个 set，则该操作会过滤掉结果中的重复项，以输出仅包含唯一条目的 array。输出 array 中元素的顺序未指定。

如果集合包含嵌套的 array 元素，则 set 表达式不会下降到嵌套的 array 中，而是在顶层level 处计算 array。

| 名称                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| [$allElementsTrue]() | 如果没有集合的元素计算为`false`，则返回`true`，否则返回`false`。接受单个参数表达式。 |
| [$anyElementTrue]()  | 如果集合中的任何元素求值为`true`，则返回`true`;否则，返回`false`。接受单个参数表达式。 |
| [$setDifference]()   | 返回一个集合，其中的元素出现在第一个集合中但不出现在第二个集合中; 即：相对于第一组执行第二组的相对补充。接受两个参数表达式。 |
| [$setEquals]()       | 如果输入 sets 具有相同的不同元素，则返回`true`。接受两个或多个参数表达式。 |
| [$setIntersection]() | 返回一个包含所有输入 sets 中出现的元素的集合。接受任意数量的参数表达式。 |
| [$setIsSubset]()     | 如果第一组的所有元素出现在第二组中，则返回`true`，包括第一组的等于第二组的时间; 即：不是严格的子集。接受两个参数表达式。 |
| [$setUnion]()        | 返回一个包含任何输入 sets 中出现的元素的集合。               |


[]()

### <span id="string-expression-operators">字符串表达式运算符</span>

字符串表达式（除外 [`$concat`](reference-operator-aggregation-concat.html#exp._S_concat)）仅对ASCII字符字符串具有明确定义的行为。

[`$concat`](reference-operator-aggregation-concat.html#exp._S_concat) 行为是明确定义的，与所使用的字符无关。

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [$concat]()         | 连接任意数量的 strings。                                     |
| [$dateFromString]() | 将 date/time string 转换为 date object。                     |
| [$dateToString]()   | 将 date 作为格式化的 string 返回。                           |
| [$indexOfBytes]()   | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 字节索引。如果未找到子字符串，则返回`-1`。 |
| [$indexOfCP]()      | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 code 点索引。如果找不到子字符串，则返回`-1` |
| [$ltrim]()          | 从字符串开头删除空格或指定的字符。<br />*版本4.0中的新功能。* |
| [$regexFind]()      | 将正则表达式（regex）应用于字符串，并返回*第一个*匹配的子字符串的信息。<br />*4.2版中的新功能。* |
| [$regexFindAll]()   | 将正则表达式（regex）应用于字符串，并返回所有匹配的子字符串的信息。<br />*4.2版中的新功能。* |
| [$regexMatch]()     | 将正则表达式（regex）应用于字符串，并返回一个布尔值，该布尔值指示是否找到匹配项。<br />*4.2版中的新功能。* |
| [$rtrim]()          | 从字符串末尾删除空格或指定的字符。<br />*版本4.0中的新功能。* |
| [$split]()          | 根据分隔符将 string 拆分为子字符串。返回子字符串的 array。如果在 string 中找不到分隔符，则返回包含原始 string 的 array。 |
| [$strLenBytes]()    | 返回 string 中 UTF-8 编码字节的数量。                        |
| [$strLenCP]()       | 返回 string 中 UTF-8 [code 点](http://www.unicode.org/glossary/#code_point)的数量。 |
| [$strcasecmp]()     | 执行 case-insensitive string 比较并返回：如果两个 strings 相等则返回`0`，如果第一个 string 大于第二个，则返回`1`，如果第一个 string 小于第二个，则返回`-1`。 |
| [$substr]()         | 已过时。使用[$substrBytes](reference-operator-aggregation-substrBytes.html#exp._S_substrBytes)或[$substrCP](reference-operator-aggregation-substrCP.html#exp._S_substrCP)。 |
| [$substrBytes]()    | 返回 string 的子字符串。从 string 中指定的 UTF-8 字节索引(zero-based)处的字符开始，并继续指定的字节数。 |
| [$substrCP]()       | 返回 string 的子字符串。从 string 中指定的 UTF-8 [code point(CP)](http://www.unicode.org/glossary/#code_point)索引(zero-based)处的字符开始，并继续指定的 code 点数。 |
| [$toLower]()        | 将 string 转换为小写。接受单个参数表达式。                   |
| [$toString]()       | 将值转换为字符串。<br />*版本4.0中的新功能。*                |
| [$trim]()           | 从字符串的开头和结尾删除空格或指定的字符。<br />*版本4.0中的新功能。* |
| [$toUpper]()        | 将 string 转换为大写。接受单个参数表达式。                   |

[]()

### <span id="text-expression-operator">文本表达式运算符</span>

| 名称      | 描述                 |
| --------- | -------------------- |
| [$meta]() | 访问文本搜索元数据。 |

[]()

### 三角表达式运算符

三角表达式对数字执行三角运算。表示角度的值始终以弧度为单位输入或输出。使用 `$degreesToRadians`和`$radiansToDegrees`在度和弧度测量之间转换。

| 名称                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| [$sin]()              | 返回以弧度为单位的值的正弦值。                               |
| [$cos]()              | 返回以弧度为单位的值的余弦值。                               |
| [$tan]()              | 返回以弧度为单位的值的切线。                                 |
| [$asin]()             | 返回弧度值的反正弦（弧正弦）。                               |
| [$acos]()             | 返回弧度值的反余弦（弧余弦）。                               |
| [$atan]()             | 返回弧度值的反正切（弧切）。                                 |
| [$atan2]()            | 返回弧度表示的`y / x`的反正切（弧切线），其中`y`和`x`是分别传递给表达式的第一个和第二个值。 |
| [$asinh]()            | 返回弧度值的反双曲正弦（双曲反正弦）。                       |
| [$acosh]()            | 返回弧度值的反双曲余弦（双曲反余弦）。                       |
| [$atanh]()            | 返回弧度值的反双曲正切（双曲反正切）。                       |
| [$degreesToRadians]() | 将值从度转换为弧度。                                         |
| [$radiansToDegrees]() | 将值从弧度转换为度。                                         |

[]()

### 类型表达式运算符

| 名称            | 描述                                              |
| --------------- | ------------------------------------------------- |
| [$convert]()    | 将值转换为指定的类型。<br />*版本4.0中的新功能。* |
| [$toBool]()     | 将值转换为布尔值。<br />*版本4.0中的新功能。*     |
| [$toDate]()     | 将值转换为日期。<br />*版本4.0中的新功能。*       |
| [$toDecimal]()  | 将值转换为Decimal128。<br />*版本4.0中的新功能。* |
| [$toDouble]()   | 将值转换为双精度。<br />*版本4.0中的新功能。*     |
| [$toInt]()      | 将值转换为整数。<br />*版本4.0中的新功能。*       |
| [$toLong]()     | 将值转换为long。<br />*版本4.0中的新功能。*       |
| [$toObjectId]() | 将值转换为ObjectId。<br />*版本4.0中的新功能。*   |
| [$toString]()   | 将值转换为字符串。<br />*版本4.0中的新功能。*     |
| [$type]()       | 返回该字段的BSON数据类型。                        |

[]()

### <span id="accumulators-group">累加器($group)</span>

累加器是可以在[$group]()阶段使用的运算符，它们在文档通过管道时保持其状态(例如： 总计，最大值，最小值和相关数据)。

当在[$group]()阶段用作累加器时，这些运算符将单个表达式作为输入，为每个输入文档计算一次表达式，并为共享相同 group key 的 group 文档保持其阶段。

| 名称              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| [$addToSet]()     | 返回每个 group 的唯一表达式值的 array。 数组元素的顺序是未定义的。 |
| [$avg]()          | 返回数值的平均值。忽略非数字值。                             |
| [$first]()        | 从每个 group 的第一个文档返回一个值。仅当文档按定义的顺序定义顺序。 |
| [$last]()         | 从每个 group 的最后一个文档返回一个值。仅当文档按定义的顺序定义顺序。 |
| [$max]()          | 返回每个 group 的最高表达式值。                              |
| [$mergeObjects]() | 返回通过组合每个 group 的输入文档创建的文档。                |
| [$min]()          | 返回每个 group 的最低表达式值。                              |
| [$push]()         | 返回每个 group 的表达式值的 array。                          |
| [$stdDevPop]()    | 返回输入值的总体标准偏差。                                   |
| [$stdDevSamp]()   | 返回输入值的 sample 标准偏差。                               |
| [$sum]()          | 返回数值的总和。忽略非数字值。                               |

[]()

### <span id="accumulators-in-other-stages">累加器(处于其他阶段)</span>

一些可用作[$group]()阶段累加器的运算符也可用于[$project]()阶段，但不能用作累加器。在[$project]()阶段使用时，这些 operator 不会维护它们的 state，并且可以将单个参数或多个 arguments 作为输入。

*更改了 version 3.2.*

以下累加器 operators 也可用于[$project]()、[$addFields]()和[$set]()阶段。

| 名称            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| [$avg]()        | 返回每个文档的指定表达式或表达式列表的平均值。忽略非数字值。 |
| [$max]()        | 返回每个文档的指定表达式或表达式列表的最大值。               |
| [$min]()        | 返回每个文档的指定表达式或表达式列表的最小值。               |
| [$stdDevPop]()  | 返回输入值的总体标准偏差。                                   |
| [$stdDevSamp]() | 返回输入值的样本标准偏差。                                   |
| [$sum]()        | 返回数值的总和。忽略非数字值。                               |


[]()

### <span id="variable-expression-operators">变量表达式运算符</span>

| 名称     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| [$let]() | 定义在子表达式范围内使用的变量，并返回子表达式的结果。接受命名参数。 <br/>接受任意数量的参数表达式。 |


[]()

## 表达式运算符的字母顺序列表

| 名称                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| [$abs]()              | 返回数字的绝对值。                                           |
| [$acos]()             | 返回弧度值的反余弦（弧余弦）。                               |
| [$acosh]()            | 返回弧度值的反双曲余弦（双曲反余弦）。                       |
| [$add]()              | 添加数字以返回总和，或者添加数字和日期以返回新日期。如果添加数字和日期，则将数字视为毫秒。接受任意数量的参数表达式，但最多只能一个表达式解析为日期。 |
| [$addToSet]()         | 返回每个 group 的唯一表达式值的 array。 数组元素的顺序是未定义的。 <br/>仅适用于[$group]()阶段 |
| [$allElementsTrue]()  | 如果没有集合的元素计算为`false`，则返回`true`，否则返回`false`。接受单个参数表达式。 |
| [$and]()              | 仅当其所有表达式求值为`true`时才返回`true`。接受任意数量的参数表达式。 |
| [$anyElementTrue]()   | 如果集合中的任何元素求值为`true`，则返回`true`;否则，返回`false`。接受单个参数表达式。 |
| [$arrayElemAt]()      | 返回指定的 array 索引处的元素。                              |
| [$arrayToObject]()    | 将键值对的 array 转换为文档。                                |
| [$asin]()             | 返回弧度值的反正弦（弧正弦）。                               |
| [$asinh]()            | 返回弧度值的反双曲正弦（双曲反正弦）。                       |
| [$atan]()             | 返回弧度值的反正切（弧切）。                                 |
| [$atan2]()            | 返回弧度表示的`y / x`的反正切（弧切线），其中`y`和`x`是分别传递给表达式的第一个和第二个值。 |
| [$atanh]()            | 返回弧度值的反双曲正切（双曲反正切）。                       |
| [$avg]()              | 返回数值的平均值。忽略非数字值。 <br/>在 version 3.2 中更改：在[$group]()和[$project]()阶段均可用。 |
| [$ceil]()             | 返回大于或等于指定数字的最小整数。                           |
| [$cmp]()              | 如果两个值相等则返回`0`，如果第一个 value 大于第二个值则返回`1`，如果第一个 value 小于第二个值，则返回`-1`。 |
| [$concat]()           | 连接任意数量的字符串。                                       |
| [$concatArrays]()     | 连接数组以返回连接的数组。                                   |
| [$cond]()             | 一个三元运算符，它计算一个表达式，并根据结果返回另外两个表达式之一的值。接受有序列表中的三个表达式或三个命名参数。 |
| [$convert]()          | 将值转换为指定的类型。                                       |
| [$cos]()              | 返回以弧度为单位的值的余弦值。                               |
| [$dateFromParts]()    | 给出 date 的组成部分，构造一个 BSON Date对象。               |
| [$dateToParts]()      | 返回包含 date 组成部分的文档。                               |
| [$dateFromString]()   | 返回 date/time 作为 date 对象。                              |
| [$dateToString]()     | 将 date 作为格式化的 string 返回。                           |
| [$dayOfMonth]()       | 将 date 的月中某天返回为 1 到 31 之间的数字。                |
| [$dayOfWeek]()        | 将 date 的星期几返回为 1(星期日)和 7(星期六)之间的数字。     |
| [$dayOfYear]()        | 将 date 的年中日期作为 1 到 366(闰年)之间的数字返回。        |
| [$degreesToRadians]() | 将值从度转换为弧度。                                         |
| [$divide]()           | 返回将第一个数除以第二个数的结果。接受两个参数表达式。       |
| [$eq]()               | 如果值相等，则返回`true`。                                   |
| [$exp]()              | 将 e 提高到指定的指数。                                      |
| [$filter]()           | 选择 array 的子集以返回 array 仅包含 match 过滤条件的元素。  |
| [$first]()            | 从每个 group 的第一个文档返回一个 value。仅当文档位于已定义的 order 中时才定义 Order。 <br/>仅适用于[$group]()阶段。 |
| [$floor]()            | 返回小于或等于指定数字的最大 integer。                       |
| [$gt]()               | 如果第一个 value 大于第二个，则返回`true`。                  |
| [$gte]()              | 如果第一个 value 大于或等于第二个，则返回`true`。            |
| [$hour]()             | 将 date 的小时数作为 0 到 23 之间的数字返回。                |
| [$ifNull]()           | 如果第一个表达式导致 null 结果，则返回第一个表达式的 non-null 结果或第二个表达式的结果。空结果包含未定义值或缺少字段的实例。接受两个表达式作为参数。第二个表达式的结果可以为 null。 |
| [$in]()               | 返回一个 boolean，指示指定的 value 是否在 array 中。         |
| [$indexOfArray]()     | 搜索 array 以查找指定 value 的出现并返回第一次出现的 array 索引。如果未找到子字符串，则返回`-1`。 |
| [$indexOfBytes]()     | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 字节索引。如果未找到子字符串，则返回`-1`。 |
| [$indexOfCP]()        | 搜索 string 以查找子字符串的出现并返回第一次出现的 UTF-8 code 点索引。如果未找到子字符串，则返回`-1`。 |
| [$isArray]()          | 确定操作数是否为 array。返回 boolean。                       |
| [$isoDayOfWeek]()     | 返回 ISO 8601 格式的工作日编号，范围从`1`(星期一)到`7`(星期日)。 |
| [$isoWeek]()          | 返回 ISO 8601 格式的周数，范围从`1`到`53`。 Week numbers 从`1`开始，周(星期一到星期日)包含年份的第一个星期四。 |
| [$isoWeekYear]()      | 以 ISO 8601 格式返回年份编号。年份从第 1 周的星期一(ISO 8601)开始，结束于上周的星期日(ISO 8601)。 |
| [$last]()             | 从每个 group 的最后一个文档返回一个 value。仅当文档位于已定义的 order 中时才定义 Order。 <br/>仅适用于[$group]()阶段。 |
| [$let]()              | 定义在子表达式范围内使用的变量，并返回子表达式的结果。接受命名参数。 <br/>接受任意数量的参数表达式。 |
| [$literal]()          | 无需解析即可返回 value。用于聚合管道可以将其解释为表达式的值。例如，对以$开头的字符串使用[$literal]()表达式，以避免解析为字段路径。 |
| [$ln]()               | 计算数字的自然对数。                                         |
| [$log]()              | 计算指定基数中的数字的对数。                                 |
| [$log10]()            | 计算数字的以10为底的对数。                                   |
| [$lt]()               | 如果第一个 value 小于第二个，则返回`true`。                  |
| [$lte]()              | 如果第一个 value 小于或等于第二个值，则返回`true`。          |
| [$ltrim]()            | 从字符串开头删除空格或指定的字符。                           |
| [$map]()              | 将子表达式应用于 array 的每个元素，并按顺序返回结果值的 array。接受命名参数。 |
| [$max]()              | 返回每个 group 的最高表达式 value。 <br/>在 version 3.2 中更改：在[$group]()和[$project]()阶段均可用。 |
| [$mergeObjects]()     | 将多个文档合并为一个文档。                                   |
| [$meta]()             | 访问文本搜索元数据。                                         |
| [$min]()              | 返回每个 group 的最低表达式 value。 <br/>在 version 3.2 中更改：在[$group]()和[$project]()阶段均可用。 |
| [$millisecond]()      | 返回 date 的毫秒数，作为 0 到 999 之间的数字。               |
| [$minute]()           | 将 date 的分钟作为 0 到 59 之间的数字返回。                  |
| [$mod]()              | 返回第一个数字的余数除以第二个数字。接受两个参数表达式。     |
| [$month]()            | 将 date 的月份返回为 1(1 月)和 12(12 月)之间的数字。         |
| [$multiply]()         | 乘以数字可返回乘积。接受任意数量的参数表达式。               |
| [$ne]()               | 如果值不相等，则返回`true`。                                 |
| [$not]()              | 返回与其参数表达式相反的布尔值。接受单个参数表达式。         |
| [$objectToArray]()    | 将文档转换为表示键值对的文档的 array。                       |
| [$or]()               | 当任何表达式求值为`true`时返回`true`。接受任意数量的参数表达式。 |
| [$pow]()              | 将数字提高到指定的指数。                                     |
| [$push]()             | 返回每个 group 的表达式值的 array。 <br/>仅适用于[$group]()阶段。 |
| [$range]()            | 根据用户定义输入输出包含整数序列的 array。                   |
| [$reduce]()           | 将表达式应用于 array 中的每个元素，并将它们组合为单个 value。 |
| [$regexFind]()        | 将正则表达式（regex）应用于字符串，并返回*第一个*匹配的子字符串的信息。 |
| [$regexFindAll]()     | 将正则表达式（regex）应用于字符串，并返回所有匹配的子字符串的信息。 |
| [$regexMatch]()       | 将正则表达式（regex）应用于字符串，并返回一个布尔值，该布尔值指示是否找到匹配项。 |
| [$reverseArray]()     | 返回具有相反顺序元素的 array。                               |
| [$round]()            | 将数字四舍五入为整数*或*指定的小数位。                       |
| [$rtrim]()            | 从字符串末尾删除空格或指定的字符。                           |
| [$second]()           | 返回 date 的秒数，作为 0 到 60 之间的数字(闰秒)。            |
| [$setDifference]()    | 返回一个集合，其中的元素出现在第一个集合中但不出现在第二个集合中; 即：相对于第一组执行第二组的[相对补充](http://en.wikipedia.org/wiki/Complement_(set_theory))。接受两个参数表达式。 |
| [$setEquals]()        | 如果输入集合具有相同的不同元素，则返回`true`。接受两个或多个参数表达式。 |
| [$setIntersection]()  | 返回一个包含所有输入集合中出现的元素的集合。接受任意数量的参数表达式。 |
| [$setIsSubset]()      | 如果第一组的所有元素出现在第二组中，则返回`true`，包括第一组的等于第二组的时间; 即：不是严格的子集。接受两个参数表达式。 |
| [$setUnion]()         | 返回一个包含任何输入集合中出现的元素的集合。                 |
| [$size]()             | 返回 array 中的元素数。接受单个表达式作为参数。              |
| [$sin]()              | 返回以弧度为单位的值的正弦值。                               |
| [$slice]()            | 返回 array 的子集。                                          |
| [$split]()            | 根据分隔符将 string 拆分为子字符串。返回子字符串的 array。如果在 string 中找不到分隔符，则返回包含原始 string 的 array。 |
| [$sqrt]()             | 计算平方根。                                                 |
| [$stdDevPop]()        | 返回输入值的总体标准偏差。 <br/>在 version 3.2 中更改：在[$group](reference-operator-aggregation-group.html#pipe._S_group)和[$project](reference-operator-aggregation-project.html#pipe._S_project)阶段均可用。 |
| [$stdDevSamp]()       | 返回输入值的样本标准偏差。 <br/>在 version 3.2 中更改：在[$group](reference-operator-aggregation-group.html#pipe._S_group)和[$project](reference-operator-aggregation-project.html#pipe._S_project)阶段均可用。 |
| [$strcasecmp]()       | 执行不区分大小写的字符串比较并返回：如果两个字符串相等则返回`0`，如果第一个字符串大于第二个，则返回`1`，如果第一个字符串小于第二个，则返回`-1`。 |
| [$strLenBytes]()      | 返回 string 中 UTF-8 编码字节的数量。                        |
| [$strLenCP]()         | 返回 string 中 UTF-8 [code 点](http://www.unicode.org/glossary/#code_point)的数量。 |
| [$substr]()           | 已过时。使用[$substrBytes](reference-operator-aggregation-substrBytes.html#exp._S_substrBytes)或[$substrCP](reference-operator-aggregation-substrCP.html#exp._S_substrCP)。 |
| [$substrBytes]()      | 返回 string 的子字符串。从 string 中指定的 UTF-8 字节索引(从零开始)处的字符开始，并继续指定的字节数。 |
| [$substrCP]()         | 返回 string 的子字符串。从 string 中指定的 UTF-8 [code point(CP)](http://www.unicode.org/glossary/#code_point)索引(从零开始)处的字符开始，并继续指定的 code 点数 |
| [$subtract]()         | 返回从第一个中减去第二个 value 的结果。如果这两个值是数字，返回差值。如果这两个值是日期，则返回差值(以毫秒为单位)。如果这两个值是 date 和一个以毫秒为单位的数字，则返回结果 date。接受两个参数表达式。如果这两个值是 date 和数字，请首先指定 date 参数，因为从数字中减去 date 没有意义。 |
| [$sum]()              | 返回数值的总和。忽略非数字值。 <br/>在 version 3.2 中更改：在[$group](reference-operator-aggregation-group.html#pipe._S_group)和[$project](reference-operator-aggregation-project.html#pipe._S_project)阶段均可用。 |
| [$switch]()           | 计算一系列案例表达。当它找到一个计算结果为`true`的表达式时，`$switch`执行一个指定的表达式并退出控制流。 |
| [$tan]()              | 返回以弧度为单位的值的切线。                                 |
| [$toBool]()           | 将值转换为布尔值。                                           |
| [$toDate]()           | 将值转换为日期。                                             |
| [$toDecimal]()        | 将值转换为Decimal128。                                       |
| [$toDouble]()         | 将值转换为双精度。                                           |
| [$toInt]()            | 将值转换为整数。                                             |
| [$toLong]()           | 将值转换为long。                                             |
| [$toObjectId]()       | 将值转换为ObjectId。                                         |
| [$toString]()         | 将值转换为字符串。                                           |
| [$toLower]()          | 将 string 转换为小写。接受单个参数表达式。                   |
| [$toUpper]()          | 将 string 转换为大写。接受单个参数表达式。                   |
| [$trim]()             | 从字符串的开头和结尾删除空格或指定的字符。                   |
| [$trunc]()            | 截断其整数的数字。                                           |
| [$type]()             | 返回该字段的 BSON 数据类型。                                 |
| [$week]()             | 返回 date 的周数，作为 0(在一年的第一个星期日之前的部分周)和 53(闰年)之间的数字。 |
| [$year]()             | 将 date 的年份作为数字返回(例：2014)。                       |
| [$zip]()              | 将两个数组合并在一起。                                       |


对于管道阶段，请参见[聚合管道阶段](Aggregation-Pipeline-Stages.md)。



译者：李冠飞

校对：