# [ ](#)$dateFromString (aggregation)
[]()

在本页面

*   [定义](#definition)
*   [行为](#behavior)
*   [格式说明](#format-specifiers)
*   [例子](#example)

## <span id="definition">定义</span>

**$dateFromString**

*3.6版的新功能。*

将日期/时间字符串转换为日期对象。

该`$dateFromString`表达式具有以下语法：

```powershell
{ $dateFromString: {
     dateString: <dateStringExpression>,
     format: <formatStringExpression>,
     timezone: <tzExpression>,
     onError: <onErrorExpression>,
     onNull: <onNullExpression>
} }
```

在`$dateFromString`需要具有以下字段的文档：

| 字段       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| dateString | 要转换为日期对象的日期/时间字符串。有关日期/时间格式的更多信息，请参见日期。<br />**注意**:<br />如果`timezone`为操作符指定选项，请不要在`dateString`中包含时区信息。 |
| format     | 可选的。`dateString`的日期格式规范 。`format`可以是计算结果为字符串文字的任何表达式，其中包含0个或多个格式说明符。有关可用的说明符列表，请参见格式说明符。<br />如果未指定，则`$dateFromString`使用 `"%Y-%m-%dT%H:%M:%S.%LZ"`默认格式。<br />*版本4.0中的新功能。* |
| timezone   | 可选的。用于格式化日期的时区。<br />**注意**:<br />如果`dateString`参数的格式类似于“ 2017-02-08T12：10：40.787Z”，其中末尾的“ Z”表示祖鲁时间（UTC时区），则无法指定`timezone`参数。<br />`<timezone>` 允许使用以下选项和对其求值的表达式：<br />1. 一个奥尔森时区标识符，例如`"Europe/London"`或`"America/New_York"`，<br />2. UTC偏移量，格式为：<br />a. `+/-[hh]:[mm]`，例如`"+04:45"`<br />b. `+/-[hh][mm]`，例如`"-0530"`<br />c. `+/-[hh]`，例如`"+03"`<br />3. 字符串“ Z”，“ UTC”或“ GMT”<br />有关表达式的更多信息，请参见表达式。 |
| onError    | 可选的。如果`$dateFromString`在解析给定`dateString`时遇到错误，则输出所提供`onError` 表达式的结果值。此结果值可以是任何类型。<br />如果未指定`onError`，`$dateFromString` 则无法解析`dateString`时将引发错误。 |
| onNull     | 可选的。如果提供给`$dateFromString`的`dateString`为空或缺失，则输出提供的`onNull`表达式的结果值。这个结果值可以是任何类型。<br />如果不指定`onNull`，并且`dateString`为`null` 或丢失，然后`$dateFromString`输出`null`。 |

> **也可以看看**
> 
> `$toDate`和 `$convert`

## <span id="behavior">行为</span>

| 例子                                                         | 结果                                |
| ------------------------------------------------------------ | ----------------------------------- |
| { $dateFromString: {     dateString: "2017-02-08T12:10:40.787" } } | ISODate("2017-02-08T12:10:40.787Z") |
| { $dateFromString: {      dateString: "2017-02-08T12:10:40.787",      timezone: "America/New_York" } } | ISODate("2017-02-08T17:10:40.787Z") |
| { $dateFromString: {      dateString: "2017-02-08" } }       | ISODate("2017-02-08T00:00:00Z")     |
| { $dateFromString: {      dateString: "06-15-2018",      format: "%m-%d-%Y" } } | ISODate("2018-06-15T00:00:00Z")     |
| { $dateFromString: {      dateString: "15-06-2018",      format: "%d-%m-%Y" } } | ISODate("2018-06-15T00:00:00Z")     |

## <span id="format-specifiers">格式说明</span>

以下格式说明符可用于 `<formatString>`：

| 说明符 | 描述                                                         | 可能的值       |
| ------ | ------------------------------------------------------------ | -------------- |
| %d     | 每月的日期（2位数字，零填充）                                | `01`--`31`     |
| %G     | ISO 8601格式的年份                                           | `0000`--`9999` |
| %H     | 小时（2位数字，零填充，24小时制）                            | `00`--`23`     |
| %L     | 毫秒（3位数字，零填充）                                      | `000`--`999`   |
| %m     | 月（2位数字，零填充）                                        | `01`--`12`     |
| %M     | 分钟（2位数字，零填充）                                      | `00`--`59`     |
| %S     | 秒（2位数字，零填充）                                        | `00`--`60`     |
| %u     | ISO 8601格式的星期几编号（1-Monday，7-Sunday）               | `1`--`7`       |
| %V     | 一年中的星期，采用ISO 8601格式                               | `1`--`53`      |
| %Y     | 年（4位数字，零填充）                                        | `0000`--`9999` |
| %z     | 与UTC的时区偏移量。                                          | +/-\[hh][mm]   |
| %Z     | 分钟数从UTC偏移为数字。例如，如果时区偏移量（`+/-[hhmm]`）为`+0445`，则分钟偏移量为`+285`。 | +/-mmm         |
| %%     | 文字字符百分比                                               | %              |

## <span id="example">例子</span>

### 转换日期

考虑一个`logmessages`包含以下带有日期的文档的集合。

```powershell
{ _id: 1, date: "2017-02-08T12:10:40.787", timezone: "America/New_York", message:  "Step 1: Started" },
{ _id: 2, date: "2017-02-08", timezone: "-05:00", message:  "Step 1: Ended" },
{ _id: 3, message:  " Step 1: Ended " },
{ _id: 4, date: "2017-02-09", timezone: "Europe/London", message: "Step 2: Started"}
{ _id: 5, date: "2017-02-09T03:35:02.055", timezone: "+0530", message: "Step 2: In Progress"}
```

以下聚合使用`$dateFromString`将`date`值转换为日期对象：

```powershell
db.logmessages.aggregate( [ {
   $project: {
      date: {
         $dateFromString: {
            dateString: '$date',
            timezone: 'America/New_York'
         }
      }
   }
} ] )
```

上述汇总返回以下文档，并将每个`date`字段转换为东部时区：

```powershell
{ "_id" : 1, "date" : ISODate("2017-02-08T17:10:40.787Z") }
{ "_id" : 2, "date" : ISODate("2017-02-08T05:00:00Z") }
{ "_id" : 3, "date" : null }
{ "_id" : 4, "date" : ISODate("2017-02-09T05:00:00Z") }
{ "_id" : 5, "date" : ISODate("2017-02-09T08:35:02.055Z") }
```

`timezone`参数也可以通过一个文档字段，而不是硬编码参数提供的。例如：

```powershell
db.logmessages.aggregate( [ {
   $project: {
      date: {
         $dateFromString: {
            dateString: '$date',
            timezone: '$timezone'
         }
      }
   }
} ] )
```

上面的汇总返回以下文档，并将每个`date`字段转换为其各自的UTC表示形式。

```powershell
{ "_id" : 1, "date" : ISODate("2017-02-08T17:10:40.787Z") }
{ "_id" : 2, "date" : ISODate("2017-02-08T05:00:00Z") }
{ "_id" : 3, "date" : null }
{ "_id" : 4, "date" : ISODate("2017-02-09T00:00:00Z") }
{ "_id" : 5, "date" : ISODate("2017-02-08T22:05:02.055Z") }
```

### `onError`

如果您的集合包含带有`$dateFromString`无法解析的日期字符串的文档， 除非您向可选参数提供聚合表达式， 否则将引发错误 `onError`。

例如，给定一个`dates`具有以下文档的集合：

```powershell
{ "_id" : 1, "date" : "2017-02-08T12:10:40.787", timezone: "America/New_York" },
{ "_id" : 2, "date" : "20177-02-09T03:35:02.055", timezone: "America/New_York" }
```

您可以使用`onError`参数以其原始字符串形式返回无效日期：

```powershell
db.dates.aggregate( [ {
   $project: {
      date: {
         $dateFromString: {
            dateString: '$date',
            timezone: '$timezone',
            onError: '$date'
         }
      }
   }
} ] )
```

这将返回以下文档：

```powershell
{ "_id" : 1, "date" : ISODate("2017-02-08T17:10:40.787Z") }
{ "_id" : 2, "date" : "20177-02-09T03:35:02.055" }
```

### `onNull`

如果您的集合包含带有`null`日期字符串的文档，则 `$dateFromString`返回`null`，除非您为可选的`onNull`参数的聚合表达式。

例如，给定一个`dates`具有以下文档的集合：

```powershell
{ "_id" : 1, "date" : "2017-02-08T12:10:40.787", timezone: "America/New_York" },
{ "_id" : 2, "date" : null, timezone: "America/New_York" }
```

您可以使用`onNull`参数让`$dateFromString`返回代表Unix纪元的日期，而不是`null`：

```powershell
db.dates.aggregate( [ {
   $project: {
      date: {
         $dateFromString: {
            dateString: '$date',
            timezone: '$timezone',
            onNull: new Date(0)
         }
      }
   }
} ] )
```

这将返回以下文档：

```powershell
{ "_id" : 1, "date" : ISODate("2017-02-08T17:10:40.787Z") }
{ "_id" : 2, "date" : ISODate("1970-01-01T00:00:00Z") }
```



译者：李冠飞

校对：