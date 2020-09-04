# [ ](#)$dateFromParts (aggregation)
[]()
在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$dateFromParts**

*3.6版的新功能。*

给定日期的组成属性，构造并返回Date对象。

`$dateFromParts`表达式具有以下语法：

```powershell
{
    $dateFromParts : {
        'year': <year>, 'month': <month>, 'day': <day>,
        'hour': <hour>, 'minute': <minute>, 'second': <second>,
        'millisecond': <ms>, 'timezone': <tzExpression>
    }
}
```

您还可以 使用以下语法以ISO周日期格式指定组成日期字段 ：

```powershell
{
    $dateFromParts : {
        'isoWeekYear': <year>, 'isoWeek': <week>, 'isoDayOfWeek': <day>,
        'hour': <hour>, 'minute': <minute>, 'second': <second>,
        'millisecond': <ms>, 'timezone': <tzExpression>
    }
}
```

在`$dateFromParts`需要具有以下字段的文档：

> **重要**
> 
> 构造`$dateFromParts`输入文档时，不能将日历日期字段和ISO周日期字段组合使用。

| 字段         | 必选/可选                           | 描述                                                         |
| ------------ | ----------------------------------- | ------------------------------------------------------------ |
| year         | 如果不使用`isoWeekYear`，则为必需的 | 公历年。可以是任何计算结果为数字的表达式。<br />值范围：`0`-`9999` |
| isoWeekYear  | 如果不使用`year`，是必需的          | ISO周日期年份。可以是任何计算结果为数字的表达式。<br />值范围： `0`-`9999` |
| month        | 可选。只能与`year`一起使用。        | `month`。可以是任何计算结果为数字的表达式。<br />默认为`1`。<br />值范围：`1`-`12`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| isoWeek      | 可选。只能与`isoWeekYear`一起使用。 | 一年中的一周。可以是任何计算结果为数字的表达式。<br />默认为`1`。<br />值范围：`1`-`53`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| day          | 可选的。只能与`year`一起使用。      | 一个月中的某天。可以是任何计算结果为数字的表达式。<br />默认为`1`。<br />值范围： `1`-`31`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| isoDayOfWeek | 可选。只能与`isoWeekYear`一起使用。 | 星期几（星期一`1`-星期日`7`）。可以是任何计算结果为数字的表达式。<br />默认为`1`。<br />值范围：`1`-`7`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| hour         | 可选                                | `<hour>`可以是任何计算结果为数字的表达式。<br />默认为`0`。<br />值范围： `0`-`23`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| minute       | 可选                                | `<minute>`可以是任何计算结果为数字的表达式。<br />默认为`0`。<br />值范围： `0`- `59`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，`$dateFromParts`则将日期计算中的差值纳入考虑范围。有关示例，请参见值范围。 |
| second       | 可选                                | `<second>`可以是任何计算结果为数字的表达式。<br />默认为`0`。<br />值范围：`0`-`59`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| millisecond  | 可选                                | `<millisecond>`。可以是任何计算结果为数字的表达式。<br />默认为`0`。<br />值范围： `0`-`999`<br />从MongoDB 4.0开始，如果指定的数字超出此范围，则会`$dateFromParts`在日期计算中纳入差异。有关示例，请参见值范围。 |
| timezone     | 可选                                | `<timezone>`可以是任何表达式，其值是字符串，其值可以是：<br />一个奥尔森时区标识符，例如`"Europe/London"`或`"America/New_York"`<br />UTC偏移量，格式为：<br />1. `+/-[hh]:[mm]`，例如`"+04:45"`<br />2. `+/-[hh][mm]`，例如`"-0530"`<br />3. `+/-[hh]`例如`"+03"`<br />有关表达式的更多信息，请参见 表达式。 |

## <span id="behavior">行为</span>

### 值范围

在MongoDB中4.0开始，如果比其它字段中指定的值 `year`，`isoYear`和`timezone`是在有效范围之外， `$dateFromParts`携带或减去从其它日期的差来计算的日期。

### 值大于范围

考虑以下`$dateFromParts`表达式，其中`month`字段值为`14`，比12个月（或1年）的最大值大2个月：

```powershell
{ $dateFromParts: { 'year' : 2017, 'month' : 14, 'day': 1, 'hour' : 12  } }
```

该表达式通过将`year`乘以1并将设置`month`为2来返回来计算日期：

```powershell
ISODate("2018-02-01T12:00:00Z")
```

### 值小于的范围

考虑以下`$dateFromParts`表达式，其中`month`字段值为`0`，比最小值1个月小1个月：

```powershell
{ $dateFromParts: { 'year' : 2017, 'month' : 0, 'day': 1, 'hour' : 12  } }
```

该表达式通过将减少`year`1并将设置`month`为12来返回，以计算日期：

```powershell
ISODate("2016-12-01T12:00:00Z")
```

### 时区

在`<timezone>` 字段中使用Olson时区标识符时，如果适用于指定的时区，MongoDB将应用DST偏移量。

例如，考虑`sales`包含以下文档的集合：

```powershell
{
   "_id" : 1,
   "item" : "abc",
   "price" : 20,
   "quantity" : 5,
   "date" : ISODate("2017-05-20T10:24:51.303Z")
}
```

以下汇总说明了MongoDB如何处理Olson时区标识符的DST偏移量。该示例使用 `$hour`and `$minute`运算符返回`date`字段的相应部分：

```powershell
db.sales.aggregate([
{
   $project: {
      "nycHour": {
         $hour: { date: "$date", timezone: "-05:00" }
       },
       "nycMinute": {
          $minute: { date: "$date", timezone: "-05:00" }
       },
       "gmtHour": {
          $hour: { date: "$date", timezone: "GMT" }
       },
       "gmtMinute": {
          $minute: { date: "$date", timezone: "GMT" } },
       "nycOlsonHour": {
          $hour: { date: "$date", timezone: "America/New_York" }
       },
       "nycOlsonMinute": {
          $minute: { date: "$date", timezone: "America/New_York" }
       }
   }
}])
```

该操作返回以下结果：

```powershell
{
   "_id": 1,
   "nycHour" : 5,
   "nycMinute" : 24,
   "gmtHour" : 10,
   "gmtMinute" : 24,
   "nycOlsonHour" : 6,
   "nycOlsonMinute" : 24
}
```

## <span id="example">例子</span>

以下聚合用于`$dateFromParts`从提供的输入字段构造三个日期对象：

```powershell
db.sales.aggregate([
{
   $project: {
      date: {
         $dateFromParts: {
            'year' : 2017, 'month' : 2, 'day': 8, 'hour' : 12
         }
      },
      date_iso: {
         $dateFromParts: {
            'isoWeekYear' : 2017, 'isoWeek' : 6, 'isoDayOfWeek' : 3, 'hour' : 12
         }
      },
      date_timezone: {
         $dateFromParts: {
            'year' : 2016, 'month' : 12, 'day' : 31, 'hour' : 23,
            'minute' : 46, 'second' : 12, 'timezone' : 'America/New_York'
         }
      }
   }
}])
```

该操作返回以下结果：

```powershell
{
  "_id" : 1,
  "date" : ISODate("2017-02-08T12:00:00Z"),
  "date_iso" : ISODate("2017-02-08T12:00:00Z"),
  "date_timezone" : ISODate("2017-01-01T04:46:12Z")
}
```



译者：李冠飞

校对：