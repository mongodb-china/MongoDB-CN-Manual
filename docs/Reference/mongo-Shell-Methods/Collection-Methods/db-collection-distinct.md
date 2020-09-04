# [ ](#)db.collection.distinct（）

[]()

在本页面

*   [定义](#definition)

*   [选项](#options)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `distinct`(字段，查询，选项)

       *   在单个集合或视图中查找指定字段的不同值，并在 array 中返回结果。

| 参数      | 类型     | 描述                                 |
| --------- | -------- | ------------------------------------ |
| `field`   | string   | 要为其返回不同值的字段。             |
| `query`   | document | 一个查询，指定从中检索不同值的文档。 |
| `options` | document | 可选的。指定选项的文档。见选项。     |

db.collection.distinct()方法在不同命令周围提供 wrapper。

> **注意**
>
> 结果不得大于最大BSON 大小。如果结果超过最大 BSON 大小，请使用聚合管道使用$group operator 检索不同的值，如使用聚合管道检索不同的值中所述。

## <span id="options">选项</span>

```powershell
{ collation: <document> }
```

| 领域        | 类型     | 描述                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| `collation` | document | 可选的。指定要用于操作的整理。整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。<br />排序规则选项具有以下语法：<br />collation: {<br />    locale: &lt;string&gt;,<br />    caseLevel: &lt;boolean&gt;, <br />   caseFirst: &lt;string&gt;, <br />   strength: &lt;int&gt;,<br />    numericOrdering: &lt;boolean&gt;, <br />   alternate: &lt;string&gt;, <br />   maxVariable: &lt;string&gt;,<br />    backwards: &lt;boolean&gt;<br /> }<br />指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。<br /> 如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。<br />如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。<br /> 您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。<br /> version 3.4 中的新内容。 |

## <span id="behavior">行为</span>

在分片集群中，不同命令可能返回孤立文档。

### 数组字段

如果指定的`field`的 value 是 array，则db.collection.distinct()将 array 的每个元素视为单独的 value。

例如，如果某个字段的 value 为`[ 1, [1], 1 ]`，则db.collection.distinct()将`1`，`[1]`和`1`视为单独的值。

对于 example，请参阅返回 Array 字段的不同值。

### 索引使用

如果可能，db.collection.distinct()操作可以使用索引。

索引也可以覆盖 db.collection.distinct()操作。有关索引涵盖的查询的详细信息，请参阅涵盖查询。

### 事务

在事务中执行不同的操作：

- 对于未分片的集合，您可以在舞台上使用 `db.collection.distinct()`方法/ `distinct`命令以及聚合管道`$group`。

- 对于分片集合，不能使用 `db.collection.distinct()`方法或 `distinct`命令。

  要查找分片集合的不同值，请在`$group`阶段使用聚合管道。有关详细信息，请参见“ 区别操作 ”。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

### 客户端断开

从MongoDB 4.2开始，如果发出`db.collection.distinct()`断开连接的客户端在操作完成之前断开连接，则MongoDB将标记`db.collection.distinct()`为终止（即`killOp`在操作上）。

## <span id="examples">例子</span>

这些示例使用包含以下文档的`inventory`集合：

```powershell
{ "_id": 1, "dept": "A", "item": { "sku": "111", "color": "red" }, "sizes": [ "S", "M" ] }
{ "_id": 2, "dept": "A", "item": { "sku": "111", "color": "blue" }, "sizes": [ "M", "L" ] }
{ "_id": 3, "dept": "B", "item": { "sku": "222", "color": "blue" }, "sizes": "S" }
{ "_id": 4, "dept": "A", "item": { "sku": "333", "color": "black" }, "sizes": [ "S" ] }
```


### 返回字段的不同值

以下 example 从`inventory`集合中的所有文档返回字段`dept`的不同值：

```powershell
db.inventory.distinct( "dept" )
```


该方法返回以下 array 不同的`dept`值：

```powershell
[ "A", "B" ]
```

### 返回嵌入字段的不同值

以下 example 从`inventory`集合中的所有文档返回嵌入在`item`字段中的字段`sku`的不同值：

```powershell
db.inventory.distinct( "item.sku" )
```


该方法返回以下 array 不同的`sku`值：

```powershell
[ "111", "222", "333" ]
```

> **也可以看看**
>
> 点符号有关访问嵌入文档中的字段的信息

### 返回 Array 字段的不同值

以下 example 从`inventory`集合中的所有文档返回字段`sizes`的不同值：

```powershell
db.inventory.distinct( "sizes" )
```


该方法返回以下 array 不同的`sizes`值：

```powershell
[ "M", "S", "L" ]
```


有关distinct()和 array 字段的信息，请参阅行为部分。

### 使用 distinct 指定 Query

以下 example 从`dept`等于`"A"`的文档中返回嵌入在`item`字段中的字段`sku`的不同值：

```powershell
db.inventory.distinct( "item.sku", { dept: "A" } )
```


该方法返回以下 array 不同的`sku`值：

```powershell
[ "111", "333" ]
```


### 指定排序规则

version 3.4 中的新内容。

整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。

集合`myColl`具有以下文档：

```powershell
{ _id: 1, category: "café", status: "A" }
{ _id: 2, category: "cafe", status: "a" }
{ _id: 3, category: "cafE", status: "a" }
```


以下聚合操作包括整理选项：

```powershell
db.myColl.distinct( "category", {}, { collation: { locale: "fr", strength: 1 } } )
```


有关归类字段的说明，请参阅整理文件。



译者：李冠飞

校对：