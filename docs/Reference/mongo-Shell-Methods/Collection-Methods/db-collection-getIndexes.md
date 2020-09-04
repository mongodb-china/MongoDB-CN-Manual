# [ ](#)db.collection.getIndexes（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [必需的访问权](#required-access)

*   [输出](#output)

## <span id="definition">定义</span>

*   `db.collection.`  `getIndexes` ()

返回一个 array，其中包含用于标识和描述集合上现有索引的文档列表。您必须在集合上调用db.collection.getIndexes()。例如：

```powershell
db.collection.getIndexes()
```

将`collection`更改为要为其返回索引信息的集合的 name。

## <span id="behavior">行为</span>

从MongoDB 4.2开始，如果发出`db.collection.getIndexes()`断开连接的客户端在操作完成之前断开连接，则MongoDB将标记`db.collection.getIndexes()`为终止（即`killOp`在操作上）。

## <span id="required-access">必需的访问权</span>

要`db.collection.getIndexes()`在强制执行访问控制时运行，使用者必须`listIndexes`对该集合具有访问权限。

内置角色`read`提供了`db.collection.getIndexes()`为数据库中的集合运行所需的特权。

## <span id="output">输出</span>

db.collection.getIndexes()返回包含集合索引信息的 array 文档。索引信息包括用于创建索引的键和选项。有关键和索引选项的信息，请参阅db.collection.createIndex()。



译者：李冠飞

校对：