# [ ](#)$atanh (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$asinh**

*4.2版中的新功能。*

返回值的反双曲正切（双曲反正切）。

`$atanh`具有以下语法：

```powershell
{ $atanh: <expression> }
```

`$atanh`接受任何有效的表达式，该表达式可解析为`-1` 和之间的数字`1`，例如。`-1 <= value <= 1`

`$atanh`返回以弧度为单位的值。使用 `$radiansToDegrees`运算符将输出值从弧度转换为度。

默认情况下以形式`$atanh`返回值`double`。 `$atanh`也可以返回值作为 128-bit小数 ，只要该`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

### `null`，`NaN`和`+/- Infinity`

如果参数解析为的值`null`或指向缺少的字段，则`$atanh`返回`null`。如果参数解析为`NaN`，则`$atanh`返回`NaN`。如果参数解析为负无穷大或正无穷大， `$atanh`则会引发错误。如果参数解析为 `+1`或`-1`，则分别`$atanh`返回`Infinity`和 `-Infinity`。

| 例子                                                     | 结果                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| { $atanh: NaN }                                          | NaN                                                          |
| { $atanh: null }                                         | null                                                         |
| { $atanh: 1 }                                            | Infinity                                                     |
| { $atanh: -1}                                            | -Infinity                                                    |
| { $atanh : Infinity}<br />or<br />{ $atanh : -Infinity } | 引发类似于以下格式化输出的错误消息：<br />"errmsg" :   "Failed to optimize pipeline :: caused by :: cannot   apply $atanh to -inf, value must in (-inf,inf)" |

## <span id="examples">例子</span>

**度数的反双曲正切值**

该`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("0.5")
}
```

以下聚合操作使用该 `$atanh`表达式计算的反双曲正切值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "y-coordinate" : {
        $radiansToDegrees : { $atanh : "$x-coordinate" }
      }
    }
  }
])
```

`$radiansToDegrees`表达式将返回的弧度值转换为`$atanh`以度为单位的等效值。

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("0.5"),
  "y-coordinate" : NumberDecimal("31.47292373094538001977241539068589")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$atanh`为128-bit十进制数。

**弧度的反双曲正切值**

`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("0.5")
}
```

以下聚合操作使用该 `$atanh`表达式计算的反双曲正切值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "y-coordinate" : {
        $atanh : "$x-coordinate"
      }
    }
  }
])
```

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("0.5"),
  "y-coordinate" : NumberDecimal("0.5493061443340548456976226184612628")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$asin`为128-bit十进制数。



译者：李冠飞

校对：