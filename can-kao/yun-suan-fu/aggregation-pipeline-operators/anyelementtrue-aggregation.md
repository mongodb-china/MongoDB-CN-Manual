# $anyElementTrue \(aggregation\)

在本页面

* [定义](anyelementtrue-aggregation.md#definition)
* [行为](anyelementtrue-aggregation.md#behavior)
* [例子](anyelementtrue-aggregation.md#examples)

## 定义

**$anyElementTrue**

将数组作为集合求值，`true`如果有则返回`true`，`false`否则返回。返回一个空数组`false`。

`$anyElementTrue`具有以下语法：

```text
{ $anyElementTrue: [ <expression> ] }
```

`<expression>`本身必须解析为一个阵列，分离从表示参数列表中的外部阵列。有关表达式的更多信息，请参见表达式。

## 行为

如果集合包含嵌套数组元素，`$anyElementTrue`则_不会_降级到嵌套数组中，而是在顶级对数组进行求值。

除了`false`布尔值，`$anyElementTrue`计算为`false`如下：`null`，`0`，和`undefined` 的值。在`$anyElementTrue`评估所有其它值`true`，包括非零数值和阵列。

| 例子 | 结果 |
| :--- | :--- |
| { $anyElementTrue: \[ \[ true, false \] \] } | true |
| { $anyElementTrue: \[ \[ \[ false \] \] \] } | true |
| { $anyElementTrue: \[ \[ null, false, 0 \] \] } | false |
| { $anyElementTrue: \[ \[ \] \] } | false |

## 例子

创建一个示例集合，其名称`survey`包含以下文档：

```text
db.survey.insertMany([
   { "_id" : 1, "responses" : [ true ] },
   { "_id" : 2, "responses" : [ true, false ] },
   { "_id" : 3, "responses" : [ ] },
   { "_id" : 4, "responses" : [ 1, true, "seven" ] },
   { "_id" : 5, "responses" : [ 0 ] },
   { "_id" : 6, "responses" : [ [ ] ] },
   { "_id" : 7, "responses" : [ [ 0 ] ] },
   { "_id" : 8, "responses" : [ [ false ] ] },
   { "_id" : 9, "responses" : [ null ] },
   { "_id" : 10, "responses" : [ undefined ] }
])
```

以下操作使用`$anyElementTrue`运算符来确定`responses`数组是否包含任何计算结果为`true`：

```text
db.survey.aggregate(
   [
     { $project: { responses: 1, isAnyTrue: { $anyElementTrue: [ "$responses" ] }, _id: 0 } }
   ]
)
```

该操作返回以下结果：

```text
{ "responses" : [ true ], "isAnyTrue" : true }
{ "responses" : [ true, false ], "isAnyTrue" : true }
{ "responses" : [ ], "isAnyTrue" : false }
{ "responses" : [ 1, true, "seven" ], "isAnyTrue" : true }
{ "responses" : [ 0 ], "isAnyTrue" : false }
{ "responses" : [ [ ] ], "isAnyTrue" : true }
{ "responses" : [ [ 0 ] ], "isAnyTrue" : true }
{ "responses" : [ [ false ] ], "isAnyTrue" : true }
{ "responses" : [ null ], "isAnyTrue" : false }
{ "responses" : [ undefined ], "isAnyTrue" : false }
```

译者：李冠飞

校对：

