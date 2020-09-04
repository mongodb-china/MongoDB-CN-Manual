# [ ](#)db.collection.renameCollection（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `renameCollection`(target，dropTarget)


重命名集合。为renameCollection 数据库命令提供包装。

| 参数         | 类型    | 描述                                                         |
| ------------ | ------- | ------------------------------------------------------------ |
| `target`     | string  | 集合的新 name。将 string 括在引号中。                        |
| `dropTarget` | boolean | 可选的。如果`true`，mongod在重命名集合之前删除了renameCollection的目标。默认的 value 是`false`。 |

## <span id="behaviors">行为</span>

db.collection.renameCollection()方法通过更改与给定集合关联的元数据在集合中运行。

有关其他警告和消息，请参阅文档renameCollection。

> **警告**
>
> db.collection.renameCollection()方法和renameCollection命令将使打开的游标无效，这会中断当前返回数据的查询。
>
> 对于Change Streams，该 `db.collection.renameCollection()`方法和 `renameCollection`命令为在源或目标集合上打开的任何现有 Change Streams创建一个 无效事件。

*   该方法具有以下限制：

    *   db.collection.renameCollection()无法在数据库之间移动集合。使用renameCollection进行这些重命名操作。
    *   分片集合不支持db.collection.renameCollection()。
    *   您无法重命名意见。

### 资源锁定

*在版本4.2中进行了更改。*

`renameCollection()`在操作期间获得对源集合和目标集合的排他锁。集合上的所有后续操作都必须等到 `renameCollection()`完成。在MongoDB 4.2之前的版本中，`renameCollection`需要获得独占数据库锁才能重命名同一数据库内的集合 。

### 与`mongodump`交互

如果客户端在转储过程中发出`db.collection.renameCollection()`，则`mongodump`以`--oplog`失败开始。看到`mongodump.--oplog`获取更多信息。

## <span id="examples">例子</span>

在集合 object 上调用db.collection.renameCollection()方法。例如：

```powershell
db.rrecord.renameCollection("record")
```

此操作会将`rrecord`集合重命名为`record`。如果目标 name(i.e.`record`)是现有集合的 name，则操作将失败。



译者：李冠飞

校对：