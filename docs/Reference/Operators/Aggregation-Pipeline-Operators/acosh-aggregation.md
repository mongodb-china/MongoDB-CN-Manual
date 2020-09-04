# [ ](#)$acosh (aggregation)

[]()

在本页面

*   [行为](#behavior)

*   [例子](#examples)

**$acosh**

4.2版中的新功能。

返回值的反双曲余弦（双曲反余弦）。

`$acosh` 具有以下语法：

```powershell
{ $acosh: <expression> }
```

`$acosh`接受任何有效的表达式，该表达式可解析为`1` 和之间的数字`+Infinity`，例如：`1 <= value <= +Infinity`

`$acosh`返回以弧度为单位的值。使用 `$radiansToDegrees`运算符将输出值从弧度转换为度。

默认情况下以形式`$acosh`返回值`double`。 `$acosh`也可以返回值作为 128-bit小数，只要该`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## 行为

### `null`，`NaN`和`+/- Infinity`

如果参数解析为的值`null`或指向缺少的字段，则`$acosh`返回`null`。如果参数解析为`NaN`，则`$acosh`返回`NaN`。如果参数解析为负无穷大， `$acosh`则会引发错误。如果参数解析为`Infinity`，则`$acosh`返回`Infinity`。如果参数解析为包含`[-1, Infinity]`范围之外的值 ，则`$acosh`会引发错误。 

| 例子                 | 结果                                                         |
| -------------------- | ------------------------------------------------------------ |
| { $acosh: NaN }      | `NaN`                                                        |
| { $acosh: null }     | `null`                                                       |
| { $acosh : Infinity} | `Infinity`                                                   |
| { $acosh : 0 }       | 引发类似于以下格式化输出的错误消息：<br />"errmsg" :   "Failed to optimize pipeline :: caused by :: cannot   apply $acosh to -inf, value must in (1,inf)" |

## 例子

**度数的反双曲余弦值**

`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
    "_id" : ObjectId("5c50782193f833234ba90d85"),
    "x-coordinate" : NumberDecimal("3")
}
```

以下聚合操作使用该 `$acosh`表达式计算的反双曲余弦值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
    {
        $addFields : {
            "y-coordinate" : {
                $radiansToDegrees : { $acosh : "$x-coordinate" }
            }
        }
    }
])
```

`$radiansToDegrees`表达式将返回的弧度值转换为`$acosh`以度为单位的等效值。

该命令返回以下输出：

```powershell
{
    "_id" : ObjectId("5c50782193f833234ba90d85"),
    "x-coordinate" : NumberDecimal("3"),
    "y-coordinate" : NumberDecimal("100.9979734210524228844295260083432")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$acosh`为128-bit十进制数。

**弧度的反双曲余弦值**

`trigonometry`集合包含一个文档，该文档沿`x`二维图形的轴存储值：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("3")
}
```

以下聚合操作使用该 `$acosh`表达式计算的反双曲余弦值，`x-coordinate`并使用`$addFields`管道阶段将其添加到输入文档中。

```powershell
db.trigonometry.aggregate([
  {
    $addFields : {
      "y-coordinate" : {
        $acosh : "$x-coordinate"
      }
    }
  }
])
```

该命令返回以下输出：

```powershell
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "x-coordinate" : NumberDecimal("3"),
  "y-coordinate" : NumberDecimal("1.762747174039086050465218649959585")
}
```

由于`x-coordinate`存储为 128-bit十进制数，因此输出 `$acosh`为128-bit十进制数。



译者：李冠飞

校对：