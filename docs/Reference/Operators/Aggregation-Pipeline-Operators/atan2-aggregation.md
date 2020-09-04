# [ ](#)$atan2 (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$atan2**

*4.2版中的新功能。*

返回`y / x`的反正切（弧形切线），其中`y`和`x`是分别传递给表达式的第一个和第二个值。

`$atan2`具有以下语法：

```powershell
{ $atan2: [ <expression 1>, <expression 2> ] }
```

`$atan2`接受可解析为数字的任何有效表达式。

`$atan2`返回以弧度为单位的值。使用 `$radiansToDegrees`运算符将输出值从弧度转换为度。

默认情况下以形式`$atan2`返回值`double`。 `$atan2`也可以返回值作为 128-bit小数，只要该`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

### `null`和`NaN`

如果的第一个参数`$atan2`是`null`，则 `$atan2`返回`null`。如果的第一个参数 `$atan2`是`NaN`，则`$atan2`返回`NaN`。如果第一个参数解析为数字*，*第二个参数解析为`NaN`或`null`， `$atan2`则分别返回`NaN`或`null`。

| 例子                                                         | 结果 |
| ------------------------------------------------------------ | ---- |
| { $atan2: [ NaN, &lt;value&gt; ] }<br />or<br />{ $atan2: [ &lt;value&gt;, NaN ] } | NaN  |
| { $atan2: [ null, &lt;value&gt; ] }<br />or<br />{ $atan2: [ &lt;value&gt;, null ] } | null |

## <span id="examples">例子</span>

**度数的反正切值**

该`trigonometry`集合包含一个文档，该文档存储直角三角形的三个边：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$atan2`表达式来计算`side_a`与`$addFields`管道之间相邻的角度并将其添加到输入文档中 。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "angle_a" : {
        $radiansToDegrees : {
          $atan2 : [ "$side_b", "$side_a" ]
        }
      }
    }
  }
])
```

`$radiansToDegrees`表达式将返回的弧度值转换为`$atan2`以度为单位的等效值。

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5"),
  "angle_a" : NumberDecimal("53.13010235415597870314438744090658")
}
```

由于`side_b`和`side_a`被存储为 128-bit小数，因此输出 `$atan2`为128-bit小数。

**弧度的反正切值**

该`trigonometry`集合包含一个文档，该文档存储直角三角形的三个边：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "side_a" : NumberDecimal("3"),
  "side_b" : NumberDecimal("4"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$atan2`表达式来计算`side_a`与`$addFields`管道之间相邻的角度并将其添加到输入文档中 。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "angle_a" : {
        $atan2 : [ "$side_b", "$side_a" ]
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
  "angle_a" : NumberDecimal("0.9272952180016122324285124629224287")
}
```

由于`side_b`和`side_a`被存储为 128-bit小数，因此输出 `$atan2`为128-bit小数。



译者：李冠飞

校对：