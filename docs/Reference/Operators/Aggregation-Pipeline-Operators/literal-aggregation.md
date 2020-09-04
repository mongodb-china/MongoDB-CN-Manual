

# [ ](#)$literal (aggregation)

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

[]()

## <span id="definition">定义</span>

**$literal**

返回 value 而不解析。用于聚合管道可以将其解释为表达式的值。

$literal表达式具有以下语法：

    { $literal: <value> }


## <span id="behavior">行为</span>

如果`<value>`是表达，$literal不会计算表达式，而是返回未解析的表达式。

| 例                                 | 结果                    |
| ---------------------------------- | ----------------------- |
| `{ $literal: { $add: [ 2, 3 ] } }` | `{ “$add“ : [ 2, 3 ] }` |
| `{ $literal: { $literal: 1 } }`    | `{ “$literal“ : 1 }`    |

## <span id="example">例子</span>

### 将$视为文字

在表达中，美元符号`$`评估为字段路径; 即：提供对该字段的访问。对于 example，`$eq` expression `$eq: [ “$price“, “$1“ ]`在名为`price`的字段中的 value 与文档中名为`1`的字段中的 value 之间执行相等性检查。

以下 example 使用$literal表达式将包含美元符号`“$1“`的 string 视为常量 value。

集合`records`具有以下文档：

```
{ “_id“ : 1, “item“ : “abc123“, price: “$2.50“ }
{ “_id“ : 2, “item“ : “xyz123“, price: “1“ }
{ “_id“ : 3, “item“ : “ijk123“, price: “$1“ }
```

```
db.records.aggregate( [
    { $project: { costsOneDollar: { $eq: [ “$price“, { $literal: “$1“ } ] } } }
] )
```

此操作投影名为`costsOneDollar`的字段，该字段包含 boolean value，指示`price`字段的 value 是否等于 string `“$1“`：

```
{ “_id“ : 1, “costsOneDollar“ : false }
{ “_id“ : 2, “costsOneDollar“ : false }
{ “_id“ : 3, “costsOneDollar“ : true }
```

### 使用 Value 1 投影新字段

$project阶段使用表达式`<field>: 1`在输出中包含`<field>`。以下 example 使用$literal来_return 将新字段设置为`1`的 value。

集合`bids`具有以下文档：

```
{ “_id“ : 1, “item“ : “abc123“, condition: “new“ }
{ “_id“ : 2, “item“ : “xyz123“, condition: “new“ }
```

以下聚合计算表达式`item: 1`以表示 return 输出中的现有字段`item`，但使用{$literal：1 }表达式 return 新字段`startAt`设置为 value `1`：

```
db.bids.aggregate( [
    { $project: { item: 1, startAt: { $literal: 1 } } }
] )
```

该操作产生以下文件：

```
{ “_id“ : 1, “item“ : “abc123“, “startAt“ : 1 }
{ “_id“ : 2, “item“ : “xyz123“, “startAt“ : 1 }
```

