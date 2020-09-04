# [ ](#)$asinh (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$asinh**

*4.2版中的新功能。*

返回值的反双曲正弦（双曲反正弦）。

`$asinh` 具有以下语法：

```powershell
{ $asinh: <expression> }
```

`$asinh`接受可解析为数字的任何有效表达式。

`$asinh`返回以弧度为单位的值。使用 `$radiansToDegrees`运算符将输出值从弧度转换为度。

默认情况下以形式`$asinh`返回值`double`。 `$asinh`也可以返回值作为 128-bit小数，只要该`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## <span id="behavior">行为</span>

### `null`，`NaN`和`+/- Infinity`

如果参数解析为的值`null`或指向缺少的字段，则`$asinh`返回`null`。如果参数解析为`NaN`，则`$asinh`返回`NaN`。如果参数解析为负无穷大或正无穷大，则`$asinh`分别返回负无穷大或正无穷大。

| 例子                   | 结果      |
| ---------------------- | --------- |
| { $asinh: NaN }        | NaN       |
| { $asinh: null }       | null      |
| { $asinh : Infinity}   | Infinity  |
| { $asinh : -Infinity } | -Infinity |

## <span id="examples">例子</span>

**度数的反双曲正弦值**

该`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("1")
}
```

以下聚合操作使用该 `$asinh`表达式计算的反双曲正弦值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "y-coordinate" : {
        $radiansToDegrees : { $asinh : "$x-coordinate" }
      }
    }
  }
])
```

该`$radiansToDegrees`表达式将返回的弧度值转换为`$asinh`以度为单位的等效值。

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("1"),
  "y-coordinate" : NumberDecimal("50.49898671052621144221476300417157")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$asinh`为128-bit十进制数。

**弧度的反双曲正弦值**

该`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("1")
}
```

以下聚合操作使用该 `$asinh`表达式计算的反双曲正弦值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "y-coordinate" : {
        $asinh : "$x-coordinate"
      }
    }
  }
])
```

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("1"),
  "y-coordinate" : NumberDecimal("1.818446459232066823483698963560709")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$asinh`为128-bit十进制数。



译者：李冠飞

校对：