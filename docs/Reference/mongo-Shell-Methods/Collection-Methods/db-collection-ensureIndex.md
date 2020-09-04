# [ ](#)db.collection.ensureIndex（）

[]()

在本页面

*   [定义](#definition)

*   [附加信息](#additional-information)

## 定义

*   `db.collection.` `ensureIndex`(键，选项)

       *   *从3.0.0版开始不推荐使用：*`db.collection.ensureIndex()`现在是的别名 `db.collection.createIndex()`。

如果索引尚不存在，则在指定字段上创建索引。

## 附加信息

*   使用db.collection.createIndex()而不是db.collection.ensureIndex()来创建新索引。

*   本手册的索引部分用于 MongoDB 中索引和索引的完整文档。

*   db.collection.getIndexes()查看集合的现有索引的规范。



译者：李冠飞

校对：