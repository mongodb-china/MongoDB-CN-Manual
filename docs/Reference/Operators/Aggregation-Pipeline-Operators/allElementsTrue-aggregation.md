# [ ](#)$allElementsTrue (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

**$allElementsTrue**

将数组评估为集合，如果 array 中没有元素为`false`，则返回`true`。否则，返回`false`。空 array 返回`true`。

$allElementsTrue具有以下语法：

```powershell
{ $allElementsTrue: [ <expression> ] }
```

`<expression>`本身必须解析为 array，与表示参数列表的外部 array 分开。有关表达式的更多信息，请参阅表达式。

## <span id="behavior">行为</span>

如果集合包含嵌套的 array 元素，则$allElementsTrue不会降级到嵌套的 array 中，而是在 top-level 处计算 array。

除了`false` 布尔值之外，$allElementsTrue还将`false`计算为以下值：`null`，`0`和`undefined`值。 $allElementsTrue将所有其他值计算为`true`，包括非零数值和数组。

| 例子                                                  | 结果   |
| ----------------------------------------------------- | ------ |
| `{ $allElementsTrue: [ [ true, 1, "someString" ] ] }` | `true` |
| `{ $allElementsTrue: [ [ [ false ] ] ] }`             | `true` |
| `{ $allElementsTrue: [ [ ] ] }`                       | `true` |
| `{ $allElementsTrue: [ [ null, false, 0 ] ] }`        | `true` |


## <span id="examples">例子</span>

考虑带有以下文档的`survey`集合：

```powershell
{ "_id" : 1, "responses" : [ true ] }
{ "_id" : 2, "responses" : [ true, false ] }
{ "_id" : 3, "responses" : [ ] }
{ "_id" : 4, "responses" : [ 1, true, "seven" ] }
{ "_id" : 5, "responses" : [ 0 ] }
{ "_id" : 6, "responses" : [ [ ] ] }
{ "_id" : 7, "responses" : [ [ 0 ] ] }
{ "_id" : 8, "responses" : [ [ false ] ] }
{ "_id" : 9, "responses" : [ null ] }
{ "_id" : 10, "responses" : [ undefined ] }
```

以下操作使用$allElementsTrue operator 来确定`responses` array 是否仅包含求值为`true`的值：

```powershell
db.survey.aggregate(
    [
        { $project: { responses: 1, isAllTrue: { $allElementsTrue: [ "$responses" ] }, _id: 0 } }
    ]
)
```

该操作返回以下结果：

```powershell
{ "responses" : [ true ], "isAllTrue" : true }
{ "responses" : [ true, false ], "isAllTrue" : false }
{ "responses" : [ ], "isAllTrue" : true }
{ "responses" : [ 1, true, "seven" ], "isAllTrue" : true }
{ "responses" : [ 0 ], "isAllTrue" : false }
{ "responses" : [ [ ] ], "isAllTrue" : true }
{ "responses" : [ [ 0 ] ], "isAllTrue" : true }
{ "responses" : [ [ false ] ], "isAllTrue" : true }
{ "responses" : [ null ], "isAllTrue" : false }
{ "responses" : [ null ], "isAllTrue" : false }
```



译者：李冠飞

校对：