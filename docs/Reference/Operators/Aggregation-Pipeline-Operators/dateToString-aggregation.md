# [ ](#)$dateToString (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [格式说明符](#format-specifiers)

*   [例子](#example)

## <span id="definition">定义</span>

**$dateToString**

根据用户指定的格式将日期对象转换为字符串。

`$dateToString`表达式具有以下 运算符表达式语法：

```powershell
{ $dateToString: {
    date: <dateExpression>,
    format: <formatString>,
    timezone: <tzExpression>,
    onNull: <expression>
} }
```

在`$dateToString`需要具有以下字段的文档：

| 字段     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| date     | *在版本3.6中更改。*<br />转换为字符串的日期。`<dateExpression>`必须是可解析为Date， Timestamp或 ObjectID的有效表达式。 |
| format   | 可选的。日期格式规范。`<formatString>`可以是任何字符串文字，包含0或多个格式说明符。有关可用说明符的列表，请参阅“ 格式说明符” 。<br />如果未指定，则`$dateToString`使用 `"%Y-%m-%dT%H:%M:%S.%LZ"`默认格式。<br />*在版本4.0中更改：*`format`如果`featureCompatibilityVersion`（fCV）设置为`"4.0"`或更大，则此字段为可选 。有关fCV的更多信息，请参见 `setFeatureCompatibilityVersion`。 |
| timezone | `Optional.`运算结果的时区。 `<tzExpression>`必须是一个有效的表达式，可以解析为格式为Olson时区标识符或 UTC偏移量的字符串。如果未`timezone`提供，结果将显示在中`UTC`。<br /><br />1. 奥尔森时区标识符：<br />a. "America/New_York"<br />b. "Europe/London"<br />c. "GMT"<br />2. UTC偏移：<br />a. +/-[hh]:[mm], e.g. "+04:45"<br />b. +/-\[hh][mm], e.g. "-0530"<br />c. +/-[hh], e.g. "+03"<br /><br />*3.6版的新功能。* |
| onNull   | 可选的。如果`date`为null或缺少，则返回的值。参数可以是任何有效的表达式。<br />如果未指定，`$dateToString`则如果`date`null为null或缺少，则返回null 。<br />*版本4.0中的新功能：*要求`featureCompatibilityVersion`（fCV）设置为 `"4.0"`或更高。有关fCV的更多信息，请参见 `setFeatureCompatibilityVersion`。 |

> **也可以看看**
> 
> `$toString`和 `$convert`

## <span id="format-specifiers">格式说明符</span>

以下格式说明符可用于 `<formatString>`：

| 说明符 | 描述                                                         | 可能的值       |
| ------ | ------------------------------------------------------------ | -------------- |
| %d     | 每月的日期（2位数字，零填充）                                | `01`--`31`     |
| %G     | ISO 8601格式的年份<br />*3.4版的新功能。*                    | `0000`--`9999` |
| %H     | 小时（2位数字，零填充，24小时制）                            | `00`--`23`     |
| %j     | 一年中的某天（3位数字，零填充）                              | `001`--`366`   |
| %L     | 毫秒（3位数字，零填充）                                      | `000`--`999`   |
| %m     | 月（2位数字，零填充）                                        | `01`--`12`     |
| %M     | 分钟（2位数字，零填充）                                      | `00`--`59`     |
| %S     | 秒（2位数字，零填充）                                        | `00`--`60`     |
| %w     | 星期几（1-星期日，7-星期六）                                 | `1`--`7`       |
| %u     | ISO 8601格式的星期几编号（1-Monday，7-Sunday）<br />*3.4版的新功能。* | `1`--`7`       |
| %U     | 一年中的星期（2位数字，零填充）                              | `00`--`53`     |
| %V     | 一年中的星期，采用ISO 8601格式<br />*3.4版的新功能。*        | `01`--`53`     |
| %Y     | 年（4位数字，零填充）                                        | `0000`--`9999` |
| %z     | 与UTC的时区偏移量。<br />*3.6版的新功能。*                   | +/-\[hh][mm]   |
| %Z     | 分钟数从UTC偏移为数字。例如，如果时区偏移量（`+/-[hhmm]`）为`+0445`，则分钟偏移量为`+285`。<br />*3.6版的新功能。* | +/-mmm         |
| %%     | 文字字符百分比                                               | %              |

## <span id="example">例子</span>

考虑`sales`包含以下文档的集合：

```powershell
{
  "_id" : 1,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : ISODate("2014-01-01T08:15:39.736Z")
}
```

以下聚合用于`$dateToString`将`date`字段作为格式化的字符串返回：

```powershell
db.sales.aggregate(
   [
     {
       $project: {
          yearMonthDayUTC: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
          timewithOffsetNY: { $dateToString: { format: "%H:%M:%S:%L%z", date: "$date", timezone: "America/New_York"} },
          timewithOffset430: { $dateToString: { format: "%H:%M:%S:%L%z", date: "$date", timezone: "+04:30" } },
          minutesOffsetNY: { $dateToString: { format: "%Z", date: "$date", timezone: "America/New_York" } },
          minutesOffset430: { $dateToString: { format: "%Z", date: "$date", timezone: "+04:30" } }
       }
     }
   ]
)
```

该操作返回以下结果：

```powershell
{
   "_id" : 1,
   "yearMonthDayUTC" : "2014-01-01",
   "timewithOffsetNY" : "03:15:39:736-0500",
   "timewithOffset430" : "12:45:39:736+0430",
   "minutesOffsetNY" : "-300",
   "minutesOffset430" : "270"
}
```



译者：李冠飞

校对：