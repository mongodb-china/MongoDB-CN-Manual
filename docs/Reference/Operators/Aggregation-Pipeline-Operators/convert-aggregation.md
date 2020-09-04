# [ ](#)$convert (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$convert**

*版本4.0中的新功能。*

将值转换为指定的类型。

`$convert`具有以下语法：

```powershell
{
   $convert:
      {
         input: <expression>,
         to: <type expression>,
         onError: <expression>,  // Optional.
         onNull: <expression>    // Optional.
      }
}
```

在`$convert`需要具有以下字段的文档：

| 字段      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| `input`   | 参数可以是任何有效的表达式。有关表达式的更多信息，请参见表达式。 |
| `to`      | 该参数可以是任何可解析为以下数字或字符串标识符之一的有效表达式，见下表 |
| `onError` | 可选的。在转换过程中遇到错误（包括不支持的类型转换）时返回的值。参数可以是任何有效的表达式。<br />如果未指定，则操作在遇到错误时将引发错误并停止。 |
| `onNull`  | 可选的。如果`input`为null或缺失，则返回的值。参数可以是任何有效的表达式。<br />如果未指定，`$convert`则如果`input`null为null或缺少，则返回null 。 |

| 字符串标识符 | 数值标识符 | 笔记                                                  |
| ------------ | ---------- | ----------------------------------------------------- |
| “double”     | 1          | 有关转换为double的更多信息，请参见 转换为Double。     |
| “string”     | 2          | 有关转换为字符串的更多信息，请参见 转换为字符串。     |
| “objectId”   | 7          | 有关转换为objectId的更多信息，请参见 转换为ObjectId。 |
| “bool”       | 8          | 有关转换为布尔值的更多信息，请参见 转换为布尔值。     |
| “date”       | 9          | 有关转换为日期的更多信息，请参见 转换为日期。         |
| “int”        | 16         | 有关转换为整数的更多信息，请参见 转换为整数。         |
| “long”       | 18         | 有关转换为long的更多信息，请参见 转换为long。         |
| “decimal”    | 19         | 有关转换为十进制的更多信息，请参见 转换为十进制。     |

除`$convert`之外，当默认的“ onError”和“ onNull”行为可以接受时，MongoDB还提供以下聚合运算符作为速记：

* $toBool
* $toDate
* $toDecimal
* $toDouble
* $toInt
* $toLong
* $toObjectId
* $toString

## <span id="behavior">行为</span>

### 转换为布尔值

下表列出了可以转换为布尔值的输入类型：

| 输入类型 | 行为                                                  |
| -------- | ----------------------------------------------------- |
| Boolean  | 无操作，返回布尔值。                                  |
| Double   | 如果不为零，则返回true。<br />如果为零，则返回false。 |
| Decimal  | 如果不为零，则返回true。<br />如果为零，则返回false。 |
| Integer  | 如果不为零，则返回true。<br />如果为零，则返回false。 |
| Long     | 如果不为零，则返回true。<br />如果为零，则返回false。 |
| ObjectId | 返回true。                                            |
| String   | 返回true。                                            |
| Date     | 返回true。                                            |

下表列出了一些转换为布尔值的示例：

| 例子                                                       | 结果  |
| ---------------------------------------------------------- | ----- |
| { input: true, to: "bool"}                                 | true  |
| { input: false, to: "bool" }                               | false |
| { input: 1.99999, to: "bool" }                             | true  |
| { input: NumberDecimal("5"), to: "bool"}                   | true  |
| { input: NumberDecimal("0"), to: "bool"}                   | false |
| { input: 100, to: "bool" }                                 | true  |
| { input: ISODate("2018-03-26T04:38:28.044Z"), to: "bool" } | true  |
| { input: "hello", to: "bool" }                             | true  |
| { input: "false", to: "bool" }                             | true  |
| { input: "", to: "bool" }                                  | true  |
| { input: null, to: "bool" }                                | Null  |

> **也可以看看**
> 
> `$toBool`

### 转换为整数

下表列出了可以转换为整数的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| Boolean  | 返回`0`的 `false`。<br />返回`1`的`true`。                   |
| Double   | 返回截断值。<br />截断后的double值必须在整数的最大值和最小值之内。<br />您不能转换其截断值小于最小整数值或大于最大整数值的double值。 |
| Decimal  | 返回截断值。<br />截断的十进制值必须在整数的最大值和最小值之内。<br />您不能转换截断值小于最小整数值或大于最大整数值的十进制值。 |
| Integer  | 无操作，返回整数值。                                         |
| Long     | 以整数形式返回long值。<br />long值必须落在整数的最小值和最大值之间。<br />您不能转换小于最小整数值或大于最大整数值的长值。 |
| String   | 以整数形式返回字符串的数值。<br />字符串值必须是base 10的整数（例如 `"-5"`，`"123456"`）并落在整数的最小值和最大值之内。<br />不能转换浮点数、十进制数或非base10数字的字符串值（例如`"-5.0"`，`"0x6400"`）或低于整数的最小和最大值的值。 |

下表列出了一些转换为整数的示例：

| 例子                                                         | 结果                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| { input: true, to: "int"}                                    | 1                                    |
| { input: false, to: "int" }                                  | 0                                    |
| { input: 1.99999, to: "int" }                                | 1                                    |
| { input: NumberDecimal("5.5000"), to: "int"}                 | 5                                    |
| { input: NumberDecimal("9223372036000.000"), to: "int"}      | Error                                |
| {<br />   input: NumberDecimal("9223372036000.000"),<br />   to: "int",<br />   onError: "Could not convert to type integer."<br /> } | “Could not convert to type integer.” |
| { input: NumberLong("5000"), to: "int"}                      | 5000                                 |
| { input: NumberLong("922337203600"), to: "int"}              | Error                                |
| { input: "-2", to: "int" }                                   | -2                                   |
| { input: "2.5", to: "int" }                                  | Error                                |
| { input: null, to: "int" }                                   | null                                 |

> **也可以看看**
> 
> `$toInt`操作符。

### 转换为十进制

下表列出了可以转换为十进制的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| Boolean  | false返回`NumberDecimal("0")`<br />true返回`NumberDecimal("1")` |
| Double   | 返回双精度值作为十进制数。                                   |
| Decimal  | 无操作，返回小数。                                           |
| Integer  | 以小数形式返回int值。                                        |
| Long     | 返回long值（十进制）。                                       |
| String   | 以十进制形式返回字符串的数值。<br />字符串值必须是base 10数字值（例如 `"-5.5"`，`"123456"`）。<br />您不能转换非base10数字的字符串值 （例如`"0x6400"`） |
| Date     | 返回自与日期值相对应的纪元以来的毫秒数。                     |

下表列出了一些转换为十进制的示例：

| 例子                                                         | 结果                              |
| ------------------------------------------------------------ | --------------------------------- |
| { input: true, to: "decimal"}                                | NumberDecimal(“1”)                |
| { input: false, to: "decimal" }                              | NumberDecimal(“0”)                |
| { input: 2.5, to: "decimal" }                                | NumberDecimal(“2.50000000000000”) |
| { input: NumberInt(5), to: "decimal"}                        | NumberDecimal(“5”)                |
| { input: NumberLong(10000), to: "decimal"}                   | NumberDecimal(“10000”)            |
| { input: "-5.5", to: "decimal" }                             | NumberDecimal(“-5.5”)             |
| { input: ISODate("2018-03-27T05:04:47.890Z"), to: "decimal" } | NumberDecimal(“1522127087890”)    |

> **也可以看看**
> 
> `$toDecimal`

### 转换为Double

下表列出了可以转换为双精度型的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| Boolean  | false返回的NumberLong（0）<br />true返回的NumberLong（1）    |
| Double   | 无操作，返回双精度型。                                       |
| Decimal  | 以双精度值返回十进制值。<br />小数值必须在双精度的最小值和最大值之内。<br />不能转换一个小于最小双精度值或大于最大双精度值的十进制值。 |
| Integer  | 以双精度值返回int值。                                        |
| Long     | 将long值作为双精度值返回。                                   |
| String   | 以双精度值形式返回字符串的数值。<br />字符串值必须是一个以10为基的数值(例如:"-5.5"， "123456")，并落在双精度的最小值和最大值之内。<br />不能转换非base10数字的字符串值。“0x6400”)或低于双精度的最小值和最大值的值。 |
| Date     | 返回自纪元以来对应于日期值的毫秒数。                         |

下表列出了一些转换为Double的示例：

| 例子                                                         | 结果                                |
| ------------------------------------------------------------ | ----------------------------------- |
| { input: true, to: "double"}                                 | 1                                   |
| { input: false, to: "double" }                               | 0                                   |
| { input: 2.5, to: "double" }                                 | 2.5                                 |
| { input: NumberInt(5), to: "double"}                         | 5                                   |
| { input: NumberLong(10000), to: "double"}                    | 10000                               |
| { input: "-5.5", to: "double" }                              | -5.5                                |
| { input: "5e10", to: "double" }                              | 50000000000                         |
| {<br />    input: "5e550",<br />    to: "double",<br />    onError: "Could not convert to type double."<br /> } | “Could not convert to type double.” |
| { input: ISODate("2018-03-27T05:04:47.890Z"), to: "double" } | 1522127087890                       |

> **也可以看看**
> 
> `$toDouble`

### 转换为Long

下表列出了可以转换为long的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| Boolean  | false返回0<br />true返回1                                    |
| Double   | 返回截断值。<br />截断后的double值必须长时间处于最小值和最大值之间。<br />不能转换其截断值小于最小长整型值或大于最大长整型值的双精度值。 |
| Decimal  | 返回截断值。<br />截断的十进制值必须在Long的最大值和最小值之间。<br />不能转换截断值小于最小长值或大于最大长值的十进制值。 |
| Integer  | 以long形式返回int值。                                        |
| Long     | 无操作。返回Long值。                                         |
| String   | 返回字符串的数值。<br />字符串值必须是base10长度的(例如。“-5”，“123456”)，并落在Long最大值和最小值之内。<br />不能转换浮点数、十进制数或非base10数字的字符串值。(例如 “-5.0”、“0x6400”)或处于Long最小和最大值之外的值。 |
| Date     | 将日期转换为纪元以来的毫秒数。                               |

下表列出了一些到长示例的转换：

| 例子                                                         | 结果                              |
| ------------------------------------------------------------ | --------------------------------- |
| { input: true, to: "long" }                                  | NumberLong(“1”)                   |
| { input: false, to: "long"  }                                | NumberLong(“0”)                   |
| { input: 1.99999, to: "long"  }                              | NumberLong(“1”)                   |
| { input: NumberDecimal("5.5000"), to: "long" }               | NumberLong(“5”)                   |
| { input: NumberDecimal("9223372036854775808.0"), to: "long" } | Error                             |
| {<br />    input: NumberDecimal("9223372036854775808.000"),<br />    to: "long",<br />    onError: "Could not convert to type long."<br /> } | “Could not convert to type long.” |
| { input: NumberInt(8), to: "long" }                          | NumberLong(8)                     |
| { input: ISODate("2018-03-26T04:38:28.044Z"), to: "long" }   | NumberLong(“1522039108044”)       |
| { input: "-2", to: "long" }                                  | NumberLong(“-2”)                  |
| { input: "2.5", to: "long" }                                 | Error                             |
| { input: null, to: "long" }                                  | null                              |

> **也可以看看**
> 
> `$toLong`

### 转换为日期

下表列出了可以转换为日期的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| Double   | 返回一个日期，该日期对应于被截断的双精度值所表示的毫秒数。<br />正值对应自1970年1月1日以来的毫秒数。<br />负数对应于1970年1月1日之前的毫秒数。 |
| Decimal  | 返回一个日期，该日期对应于被截断的十进制值所表示的毫秒数。<br />正值对应自1970年1月1日以来的毫秒数。<br />负数对应于1970年1月1日之前的毫秒数。 |
| Long     | 返回一个日期，该日期对应于Long值所表示的毫秒数。<br />正值对应自1970年1月1日以来的毫秒数。<br />负数对应于1970年1月1日之前的毫秒数。 |
| String   | 返回与日期字符串对应的日期。<br />字符串必须是一个有效的日期字符串，例如:<br />1. “2018-03-03”<br />2. “2018-03-03T12:00:00Z”<br />3. “2018-03-03T12:00:00+0500” |
| ObjectId | 返回与ObjectId的时间戳相对应的日期。                         |

下表列出了一些转换日期的示例：

| 例子                                                         | 结果                              |
| ------------------------------------------------------------ | --------------------------------- |
| { input: 120000000000.5, to: "date"}                         | ISODate(“1973-10-20T21:20:00Z”)   |
| { input: NumberDecimal("1253372036000.50"), to: "date"}      | ISODate(“2009-09-19T14:53:56Z”)   |
| { input: NumberLong("1100000000000"), to: "date"}            | ISODate(“2004-11-09T11:33:20Z”)   |
| { input:  NumberLong("-1100000000000"), to: "date"}          | ISODate(“1935-02-22T12:26:40Z”)   |
| { input: ObjectId("5ab9c3da31c2ab715d421285"), to: "date" }  | ISODate(“2018-03-27T04:08:58Z”)   |
| { input:  "2018-03-03", to: "date" }                         | ISODate(“2018-03-03T00:00:00Z”)   |
| { input: "2018-03-20 11:00:06 +0500", to: "date" }           | ISODate(“2018-03-20T06:00:06Z”)   |
| { input: "Friday", to: "date" }                              | Error                             |
| {<br />    input: "Friday",<br />    to: "date",<br />    onError: "Could not convert to type date."<br /> } | “Could not convert to type date.” |

> **也可以看看**
> 
> `$toDate`操作符， `$dateFromString`

### 转换成的ObjectId 

下表列出了可以转换为ObjectId的输入类型：

| 输入类型 | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| String   | 返回长度为24的十六进制字符串的ObjectId。<br />不能转换长度为24的十六进制字符串以外的字符串值。 |

下表列出了一些转换日期的示例：

| 例子                                                         | 结果                                  |
| ------------------------------------------------------------ | ------------------------------------- |
| { input: "5ab9cbfa31c2ab715d42129e", to: "objectId"}         | ObjectId(“5ab9cbfa31c2ab715d42129e”)  |
| { input: "5ab9cbfa31c2ab715d42129", to: "objectId"}          | Error                                 |
| {<br />    input: "5ab9cbfa31c2ab715d42129",<br />    to: "objectId",<br />    onError: "Could not convert to type ObjectId."<br /> } | “Could not convert to type ObjectId.” |

> **也可以看看**
> 
> `$toObjectId`操作符。

### 转换为字符串

下表列出了可以转换为字符串的输入类型：

| 输入类型 | 行为                                   |
| -------- | -------------------------------------- |
| Boolean  | 以字符串形式返回布尔值。               |
| Double   | 以字符串形式返回双精度值。             |
| Decimal  | 以字符串形式返回十进制值。             |
| Integer  | 以字符串的形式返回整数值。             |
| Long     | 以字符串形式返回长值。                 |
| ObjectId | 以十六进制字符串的形式返回ObjectId值。 |
| String   | 无操作。返回字符串值。                 |
| Date     | 以字符串形式返回日期。                 |

下表列出了一些转换为字符串的示例：

| 例子                                                         | 结果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| { input: true, to: "string" }                                | “true”                                                       |
| { input: false, to: "string"  }                              | “false”                                                      |
| { input: 2.5, to: "string"}                                  | “2.5”                                                        |
| { input: NumberInt(2), to: "string"}                         | “2”                                                          |
| { input:  NumberLong(1000), to: "string"}                    | “1000”                                                       |
| { input: ObjectId("5ab9c3da31c2ab715d421285"), to: "string" } | “5ab9c3da31c2ab715d421285”                                   |
| { input:  ISODate("2018-03-27T16:58:51.538Z"), to: "string" } | { input:  ISODate("2018-03-27T16:58:51.538Z"), to: "string" }“2018-03-27T16:58:51.538Z” |

> **也可以看看**
> 
> `$toString`操作符。 `$dateToString`

## <span id="example">例子</span>

`orders`使用以下文档创建一个集合：

```powershell
db.orders.insert( [
   { _id: 1, item: "apple", qty: 5, price: 10 },
   { _id: 2, item: "pie", qty: 10, price: NumberDecimal("20.0") },
   { _id: 3, item: "ice cream", qty: 2, price: "4.99" },
   { _id: 4, item: "almonds" },
   { _id: 5, item: "bananas", qty: 5000000000, price: NumberDecimal("1.25") }
] )
```

集合上的以下汇总操作`orders`会将转换`price`为小数：

```powershell
// 定义使用转换后的价格和数量值添加convertedPrice和convertedQty字段的阶段
// 如果没有price或qty值，则返回十进制值或整数值
// 如果不能转换价格或数量值，将返回一个字符串

priceQtyConversionStage = {
   $addFields: {
      convertedPrice: { $convert: { input: "$price", to: "decimal", onError: "Error", onNull: NumberDecimal("0") } },
      convertedQty: { $convert: {
         input: "$qty", to: "int",
         onError:{$concat:["Could not convert ", {$toString:"$qty"}, " to type integer."]},
         onNull: NumberInt("0")
      } },
   }
};

totalPriceCalculationStage = {
   $project: { totalPrice: {
     $switch: {
        branches: [
          { case: { $eq: [ { $type: "$convertedPrice" }, "string" ] }, then: "NaN" },
          { case: { $eq: [ { $type: "$convertedQty" }, "string" ] }, then: "NaN" },
        ],
        default: { $multiply: [ "$convertedPrice", "$convertedQty" ] }
     }
} } };

db.orders.aggregate( [
   priceQtyConversionStage,
   totalPriceCalculationStage
])
```

该操作返回以下文档：

```powershell
{ "_id" : 1, "totalPrice" : NumberDecimal("50.0000000000000") }
{ "_id" : 2, "totalPrice" : NumberDecimal("200.0") }
{ "_id" : 3, "totalPrice" : NumberDecimal("9.98") }
{ "_id" : 4, "totalPrice" : NumberDecimal("0") }
{ "_id" : 5, "totalPrice" : "NaN" }
```



译者：李冠飞

校对：