# [ ](#)$dateToParts (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$dateToParts**

*3.6版的新功能。*

返回包含给定BSON日期值的组成部分作为单个属性的文档。返回的属性`year`，`month`，`day`，`hour`，`minute`，`second` 和`millisecond`。

您可以将`iso8601`属性设置为`true`，以返回代表ISO周日期的部分 。这将返回一个文档，其中的属性是 `isoWeekYear`，`isoWeek`，`isoDayOfWeek`，`hour`， `minute`，`second`和`millisecond`。

`$dateToParts`表达式具有以下语法：

```powershell
{
    $dateToParts: {
        'date' : <dateExpression>,
        'timezone' : <timezone>,
        'iso8601' : <boolean>
    }
}
```

在`$dateToParts`需要具有以下字段的文档：

| 字段     | 必选/可选 | 描述                                                         |
| -------- | --------- | ------------------------------------------------------------ |
| year     | 必选      | *在版本3.6中更改。*<br />返回部分的输入日期。`<dateExpression>`可以是解析为日期、时间戳或ObjectID的任何表达式。有关表达式的更多信息，请参见表达式。 |
| timezone | 可选      | 用于格式化日期的时区。默认情况下， `$dateToParts`使用UTC。<br />`<timezone>`可以是任何表达式，该表达式的值可以是:<br />1. 一个奥尔森时区标识符，例如`"Europe/London"`或`"America/New_York"`，<br />2. UTC偏移量，格式为：<br />a. `+/-[hh]:[mm]`，例如`"+04:45"`<br />b. `+/-[hh][mm]`，例如`"-0530"`<br />c. `+/-[hh]`例如`"+03"`<br />有关表达式的更多信息，请参见 表达式。 |
| iso8601  | 可选      | 如果设置为`true`，则修改输出文档以使用ISO周日期字段。默认为`false`。 |

## <span id="behavior">行为</span>

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

以下汇总说明了MongoDB如何处理Olson时区标识符的DST偏移量。该示例使用 `$hour`和 `$minute`运算符返回`date`字段的相应部分：

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

考虑`sales`包含以下文档的集合：

```powershell
{
  "_id" : 2,
  "item" : "abc",
  "price" : 10,
  "quantity" : 2,
  "date" : ISODate("2017-01-01T01:29:09.123Z")
}
```

以下聚合用于`$dateToParts`返回包含`date`字段组成部分的文档。

```powershell
db.sales.aggregate([
 {
    $project: {
       date: {
          $dateToParts: { date: "$date" }
       },
       date_iso: {
          $dateToParts: { date: "$date", iso8601: true }
       },
       date_timezone: {
          $dateToParts: { date: "$date", timezone: "America/New_York" }
       }
    }
}])
```

该操作返回以下结果：

```powershell
{
   "_id" : 2,
   "date" : {
      "year" : 2017,
      "month" : 1,
      "day" : 1,
      "hour" : 1,
      "minute" : 29,
      "second" : 9,
      "millisecond" : 123
   },
   "date_iso" : {
      "isoWeekYear" : 2016,
      "isoWeek" : 52,
      "isoDayOfWeek" : 7,
      "hour" : 1,
      "minute" : 29,
      "second" : 9,
      "millisecond" : 123
   },
   "date_timezone" : {
      "year" : 2016,
      "month" : 12,
      "day" : 31,
      "hour" : 20,
      "minute" : 29,
      "second" : 9,
      "millisecond" : 123
   }
}
```



译者：李冠飞

校对：