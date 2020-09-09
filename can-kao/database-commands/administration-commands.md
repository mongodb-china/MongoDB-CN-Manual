# 管理命令

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

| 名称 | 描述 |
| :--- | :--- |
| [`clean`](administration-commands.md) | 内部名称空间管理命令。 |
| [`cloneCollection`](administration-commands.md) | 将集合从远程主机复制到当前主机。 |
| [`cloneCollectionAsCapped`](administration-commands.md) | 将未设置上限的集合复制为新的设置上限的集合。 |
| [`collMod`](administration-commands.md) | 向集合添加选项或修改视图定义。 |
| [`compact`](administration-commands.md) | 对集合进行分片整理并重建索引。 |
| [`connPoolSync`](administration-commands.md) | 用于刷新连接池的内部命令。 |
| [`convertToCapped`](administration-commands.md) | 将无上限的集合转换为有上限的集合。 |
| [`create`](administration-commands.md) | 创建一个集合或视图。 |
| [`createIndexes`](administration-commands.md) | 为一个集合构建一个或多个索引。 |
| [`currentOp`](administration-commands.md) | 返回一个文档，该文档包含有关数据库实例正在进行的操作的信息。 |
| [`drop`](administration-commands.md) | 从数据库中删除指定的集合。 |
| [`dropDatabase`](administration-commands.md) | 删除当前数据库。 |
| [`dropConnections`](administration-commands.md) | 将外向连接删除到指定的主机列表。 |
| [`dropIndexes`](administration-commands.md) | 从集合中删除索引。 |
| [`filemd5`](administration-commands.md) | 返回使用GridFS存储的文件的md5哈希值。 |
| [`fsync`](administration-commands.md) | 将挂起的写入刷新到存储层，并锁定数据库以允许备份。 |
| [`fsyncUnlock`](administration-commands.md) | 解锁一个fsync锁。 |
| [`getParameter`](administration-commands.md) | 检索配置选项。 |
| [`killCursors`](administration-commands.md) | 杀死集合的指定游标。 |
| [`killOp`](administration-commands.md) | 终止操作ID指定的操作。 |
| [`listCollections`](administration-commands.md) | 返回当前数据库中的集合列表。 |
| [`listDatabases`](administration-commands.md) | 返回列出所有数据库的文档，并返回基本数据库统计信息。 |
| [`listIndexes`](administration-commands.md) | 列出集合的所有索引。 |
| [`logRotate`](administration-commands.md) | 循环MongoDB日志，以防止单个文件占用过多空间。 |
| [`reIndex`](administration-commands.md) | 重建集合上的所有索引。 |
| [`renameCollection`](administration-commands.md) | 更改现有集合的名称。 |
| [`setFeatureCompatibilityVersion`](administration-commands.md) | 启用或禁用保留向后不兼容的数据的功能。 |
| [`setParameter`](administration-commands.md) | 修改配置选项。 |
| [`shutdown`](administration-commands.md) | 关闭`mongod`或`mongos`进程。 |

译者：李冠飞

校对：

