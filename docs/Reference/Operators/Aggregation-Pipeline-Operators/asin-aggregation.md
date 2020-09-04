# [ ](#)$asin (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$asin**

*4.2版中的新功能。*

返回值的反正弦（弧正弦）。

`$asin`具有以下语法：

```powershell
{ $asin: <expression> }
```

`$asin`接受任何有效的表达式，该表达式可解析为`-1` 和之间的数字`1`，例如。`-1 <= value <= 1`

`$asin`返回以弧度为单位的值。使用 `$radiansToDegrees`运算符将输出值从弧度转换为度。

默认情况下以形式`$asin`返回值`double`。 `$asin`也可以返回值作为 128-bit小数 ，只要该`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

如果参数解析为的值`null`或指向缺少的字段，则`$asin`返回`null`。如果参数解析为`NaN`，则`$asin`返回`NaN`。如果参数解析为包含`[-1, 1]`范围之外的值 ，则`$asin`会引发错误。 

| 例子                                                   | 结果                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| { $asin: NaN }                                         | NaN                                                          |
| { $asin: null }                                        | null                                                         |
| { $asin : Infinity}<br />OR<br />{ $asin : -Infinity } | 引发类似于以下格式化输出的错误消息：<br />"errmsg" :   "Failed to optimize pipeline :: caused by :: cannot   apply $asin to -inf, value must in [-1,1]" |

## <span id="examples">例子</span>

**度数的反正弦值**

该`trigonometry`集合包含一个文档，该文档存储直角三角形的三个边：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$asin`表达式计算与之成反角，`side_a`并使用`$addFields`管道阶段将其添加到输入文档中 。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "angle_a" : {
        $radiansToDegrees : {
          $asin : {
            $divide : [ "$side_a", "$hypotenuse" ]
          }
        }
      }
    }
  }
])
```

该`$radiansToDegrees`表达式将返回的弧度值转换为`$asin`以度为单位的等效值。

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5"),
  "angle_a" : NumberDecimal("36.86989764584402129685561255909341")
}
```

由于`side_a`和`hypotenuse`被存储为 128-bit小数，因此输出 `$asin`为128-bit小数。

**弧度的反正弦**

该`trigonometry`集合包含一个文档，该文档存储直角三角形的三个边：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$asin`表达式来计算`side_a`与`$addFields`管道之间相邻的角度并将其添加到输入文档中 。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "angle_a" : {
        $asin : {
          $divide : [ "$side_a", "$hypotenuse" ]
        }
      }
    }
  }
])
```

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5"),
  "angle_a" : NumberDecimal("0.6435011087932843868028092287173226")
}
```

由于`side_a`和`hypotenuse`被存储为 128-bit小数，因此输出 `$asin`为128-bit小数。



译者：李冠飞

校对：