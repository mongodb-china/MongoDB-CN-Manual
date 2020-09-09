# $cos \(aggregation\)

 在本页面

* [定义](cos-aggregation.md#definition)
* [行为](cos-aggregation.md#behavior)
* [例子](cos-aggregation.md#example)

## 定义

**$cos**

_4.2版中的新功能。_

返回以弧度为单位的值的余弦值。

`$cos` 具有以下语法：

```text
{ $cos: <expression> }
```

`$cos`接受可解析为数字的任何有效表达式。如果表达式返回以度为单位的值，请使用`$degreesToRadians`运算符将结果转换为弧度。

默认情况下以形式`$cos`返回值是`double`。 `$cos$cos`还可以以128-bit小数的形式返回值，只要`<expression>`解析为一个128-bit的十进制值。

有关表达式的更多信息，请参见 表达式。

## 行为

### `null`，`NaN`和`+/- Infinity`

如果参数解析的值为`null`或指向缺少的字段，则`$cos`返回`null`。如果参数解析为`NaN`，则`$cos`返回`NaN`。如果参数解析为负无穷大或正无穷大， `$cos`则会引发错误。

| 例子 | 结果 |
| :--- | :--- |
| `{ $cos: NaN }` | NaN |
| `{ $cos: null }` | null |
| `{ $cos : Infinity}` or `{ $cos : -Infinity }` | 引发类似于以下格式化输出的错误消息： "errmsg" :   "Failed to optimize pipeline :: caused by :: cannot   apply $cos to -inf, value must in \(-inf,inf\)" |

## 例子

**度数的余弦值**

该`trigonometry`集合包含一个文档，该文档存储斜边和直角三角形中的一个角度：

```text
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "angle_a" : NumberDecimal("53.13010235415597870314438744090659"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$cos`表达式来计算相邻的边，`angle_a`并使用`$addFields`管道阶段将其添加到输入文档中 。

```text
db.trigonometry.aggregate([
  {
    $addFields : {
      "side_a" : {
        $multiply : [
          { $cos : {$degreesToRadians : "$angle_a"} },
          "$hypotenuse"
        ]
      }
    }
  }
])
```

`$degreesToRadians`表达式将的度数值转换为`angle_a`以弧度为单位的等效值。

该操作返回以下结果：

```text
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "angle_a" : NumberDecimal("53.13010235415597870314438744090659"),
  "side_a" : NumberDecimal("2.999999999999999999999999999999999"),
  "hypotenuse" : NumberDecimal("5"),
}
```

由于`angle_a`和`hypotenuse`被存储为 128-bit小数，因此输出 `$cos`为128-bit小数。

**弧度中的正弦值**

`trigonometry`集合包含一个文档，该文档存储斜边和直角三角形中的一个角度：

```text
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "angle_a" : NumberDecimal("0.9272952180016122324285124629224288"),
  "hypotenuse" : NumberDecimal("5")
}
```

以下聚合操作使用该 `$cos`表达式来计算相邻的边，`angle_a`并使用`$addFields`管道阶段将其添加到输入文档中 。

```text
db.trigonometry.aggregate([
  {
    $addFields : {
      "side_b" : {
        $multiply : [
          { $cos : "$angle_a" },
          "$hypotenuse"
        ]
      }
    }
  }
])
```

该命令返回以下输出：

```text
{
  "_id" : ObjectId("5c50782193f833234ba90d85"),
  "angle_a" : NumberDecimal("0.9272952180016122324285124629224288"),
  "side_b" : NumberDecimal("3.000000000000000000000000000000000"),
  "hypotenuse" : NumberDecimal("5"),
}
```

由于`angle_a`和`hypotenuse`被存储为 128-bit小数，因此输出 `$cos`为128-bit小数。

译者：李冠飞

校对：

