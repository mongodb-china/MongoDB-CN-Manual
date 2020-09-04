# [ ](#)db.collection.copyTo（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `copyTo`(newCollection)

       *   自 version 3.0 以来已弃用。

使用 server-side JavaScript 将`collection`中的所有文档复制到`newCollection`。如果`newCollection`不存在，MongoDB 会创建它。

如果启用了授权，则必须能够访问 order run db.collection.copyTo()中所有资源的所有操作。建议不要提供此类访问权限，但如果您的组织要求用户 run db.collection.copyTo()，请创建一个在anyResource上授予anyAction的角色。不要将此角色分配给任何其他用户。

| 参数            | 类型   | 描述                        |
| --------------- | ------ | --------------------------- |
| `newCollection` | string | 要将数据写入的集合的 name。 |

> **警告**
>
> 使用 db.collection.copyTo()检查字段类型时，确保操作不会在从 BSON 转换为 JSON 期间从文档中删除类型信息。
> db.collection.copyTo()方法在内部使用EVAL命令。因此，db.collection.copyTo()操作采用 global 锁定，阻止所有其他读取和写入操作，直到db.collection.copyTo()完成。

copyTo()返回复制的文档数。如果复制失败，则抛出 exception。

## <span id="behavior">行为</span>

因为copyTo()在内部使用EVAL，所以复制操作将阻止mongod实例上的所有其他操作。

## <span id="examples">例子</span>

以下操作将`source`集合中的所有文档复制到`target`集合中。

```powershell
db.source.copyTo(target)
```



译者：李冠飞

校对：