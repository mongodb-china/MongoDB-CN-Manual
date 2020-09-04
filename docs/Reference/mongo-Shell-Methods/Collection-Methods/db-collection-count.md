# [ ](#)db.collection.count（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection. count`(查询，选项)

       * 返回将_查询集合或视图的find()查询的文档计数。 db.collection.count()方法不执行find()操作，而是计算并返回匹配查询的结果数。

> **注意**
>
> 与4.0功能兼容的MongoDB驱动程序弃用各自的游标和收集`count()`的API，取而代之的是新的API `countDocuments()`和`estimatedDocumentCount()`。有关给定驱动程序的特定API名称，请参阅驱动程序文档。

<br />

> **重要**
>
> - 避免使用
>
>   `db.collection.count()`
>
>   没有查询谓词的方法，因为如果没有查询谓词，该方法将基于集合的元数据返回结果，这可能会导致近似计数。特别是，
>
>   - 在分片群集上，结果计数将无法正确过滤出孤立的文档。
>   - 不正常关机后，计数可能不正确。
>
> - 有关基于集合元数据的计数，另请参阅 带有count选项的collStats管道阶段。

| 参数      | 类型     | 描述                         |
| --------- | -------- | ---------------------------- |
| `query`   | document | 查询选择标准。               |
| `options` | document | 可选的。修改计数的额外选项。 |

`options`文档包含以下字段：

| 领域          | 类型               | 描述                                                         |
| ------------- | ------------------ | ------------------------------------------------------------ |
| `limit`       | integer            | 可选的。要计算的最大文档数。                                 |
| `skip`        | integer            | 可选的。计数前要跳过的文档数。                               |
| `hint`        | string or document | 可选的。查询的索引 name 提示或规范。  version 2.6 中的新内容。 |
| `maxTimeMS`   | integer            | 可选的。允许查询 run 的最大 time 时间。                      |
| `readConcern` | string             | 可选的。指定阅读关注。默认的 level 是“本地”。 要使用阅读关注的阅读关注 level，replica sets 必须使用WiredTiger 存储引擎并选举protocol version 1。 从 MongoDB 3.6 开始，默认情况下启用对读取关注“多数”的支持。对于 MongoDB 3.6.1 - 3.6.x，您可以禁用读取关注“多数”。有关更多信息，请参阅禁用阅读关注多数。 要确保单个线程可以读取自己的写入，请对副本集的主要使用“多数”读取关注和“多数”写入关注。 要使用“多数”的阅读关注 level，必须指定非空`query`条件。  version 中的新内容 3.2. |
| `collation`   | document           | 可选的。<br />指定 用于操作的排序规则。<br />归类允许用户为字符串比较指定特定于语言的规则，例如字母大写和重音符号的规则。<br />排序规则选项具有以下语法：<br />collation: {<br/>   locale: &lt;string&gt;,<br/>   caseLevel: &lt;boolean&gt;,<br/>   caseFirst: &lt;string&gt;,<br/>   strength: &lt;int&gt;,<br/>   numericOrdering: &lt;boolean&gt;,<br/>   alternate: &lt;string&gt;,<br/>   maxVariable: &lt;string&gt;,<br/>   backwards: &lt;boolean&gt;<br/>}<br />指定排序规则时，该`locale`字段为必填字段；所有其他排序规则字段都是可选的。有关字段的说明，请参见整理文档。<br />如果未指定排序规则，但是集合具有默认排序规则（请参阅参考资料`db.createCollection()`），则该操作将使用为集合指定的排序规则。<br />如果没有为集合或操作指定排序规则，则MongoDB会将以前版本中使用的简单二进制比较用于字符串比较。<br />您不能为一个操作指定多个排序规则。例如，您不能为每个字段指定不同的排序规则，或者如果对排序执行查找，则不能对查找使用一种排序规则，而对排序使用另一种排序规则。<br />*3.4版的新功能。* |

count()等同于`db.collection.find(query).count()`。

> **也可以看看**
>
> cursor.count()

## <span id="behavior">行为</span>

### Sharded Clusters

在分片 cluster 上，如果孤儿文件存在或块迁移正在进行中，db.collection.count()可能导致计数不准确。

要避免这些情况，请在分片 cluster 上使用db.collection.aggregate()方法：

* 您可以使用$count阶段来计算文档。对于 example，以下操作计算集合中的文档：

  ```powershell
  db.collection.aggregate([
     { $count: "myCount" }
  ])
  ```

* $count阶段等效于以下$group $project序列：

  ```powershell
  db.collection.aggregate( [
     { $group: { _id: null, myCount: { $sum: 1 } } },
     { $project: { _id: 0 } }
  ] )
  ```

*   要获取匹配查询条件的文档计数，还要包括$match阶段：
    ```powershell
    db.collection.aggregate( [
       { $match: <query condition> },
       { $count: "myCount" }
    ] )
    ```

或者，如果使用`$group + $project`等效：

```powershell
db.collection.aggregate( [
   { $match: <query condition> },
   { $count: "myCount" }
] )
```

> **也可以看看**
>
> $collStats返回基于集合的元数据的近似计数。

### 索引使用

考虑具有以下索引的集合：

```powershell
{ a: 1, b: 1 }
```

执行计数时，如果出现以下情况，MongoDB 可以仅使用索引返回计数：

* 查询可以使用索引，

* 查询只包含索引键的条件，和

*   查询谓词访问单个连续范围的索引键。

对于 example，以下操作可以仅使用索引_return 计数：

```powershell
db.collection.find( { a: 5, b: 5 } ).count()
db.collection.find( { a: { $gt: 5 } } ).count()
db.collection.find( { a: 5, b: { $gt: 10 } } ).count()
```

但是，如果查询可以使用索引但查询谓词不访问单个连续范围的索引键，或者查询还包含索引外部字段的条件，那么除了使用索引之外，MongoDB 还必须读取文档要_return 计数。

```powershell
db.collection.find( { a: 5, b: { $in: [ 1, 2, 3 ] } } ).count()
db.collection.find( { a: { $gt: 5 }, b: 5 } ).count()
db.collection.find( { a: 5, b: 5, c: 5 } ).count()
```

在这种情况下，在初始读取文档期间，MongoDB 将文档分页到 memory，以便后续 calls 相同的计数操作将具有更好的 performance。

### 意外关机后的准确性

使用有线老虎存储引擎不正常关闭mongod后，count()报告的计数统计信息可能不准确。

漂移量取决于在最后检查站和不干净关闭之间执行的 insert，update 或 delete 操作的数量。检查点通常每 60 秒发生一次。但是，使用 non-default --syncdelay设置运行mongod实例可能会有更多或更少的检查点。

在mongod上的每个集合上运行验证以在不正常关闭后恢复正确的统计信息。

> **注意**
>
> 这种精度损失仅适用于不包含查询谓词的count()操作。

## <span id="examples">例子</span>

### 计算集合中的所有文档

要计算`orders`集合中所有文档的数量，请使用以下操作：

```powershell
db.orders.count()
```

此操作等效于以下内容：

```powershell
db.orders.find().count()
```


### 计算匹配查询的所有文档

使用大于`new Date('01/01/2012')`的字段`ord_dt`计算`orders`集合中的文档数：

```powershell
db.orders.count( { ord_dt: { $gt: new Date('01/01/2012') } } )
```

该查询等效于以下内容：

```powershell
db.orders.find( { ord_dt: { $gt: new Date('01/01/2012') } } ).count()
```



译者：李冠飞

校对：