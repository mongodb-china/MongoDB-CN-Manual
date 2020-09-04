# [ ](#)db.collection.reIndex（）

[]()

在本页面

*   [行为](#behaviors)


`db.collection.`  `reIndex` ()

db.collection.reIndex()删除集合上的所有索引并重新创建它们。对于具有大量数据 and/or 大量索引的集合，此操作可能很费时。

> **警告**
> 
>*   对于大多数用户，不需要db.collection.reIndex()操作。
>*   避免对副本集中的集合 running db.collection.reIndex()。
>*   不要对分片 cluster 中的集合运行db.collection.reIndex()。
>
>*在版本4.2中进行了更改：* MongoDB不允许`db.collection.reIndex()`在上运行`mongos`，对分片`db.collection.reIndex()`群集中的集合实施了更严格的限制 。

## <span id="behaviors">行为</span>

> **注意**
>
> 对于副本_set，db.collection.reIndex()不会从主节点传播到从节点。 db.collection.reIndex()只会影响单个mongod实例。

<br />

> **重要**
>
> 由于多索引构建中描述的逻辑，db.collection.reIndex()始终在前台构建索引。

* 对于将featureCompatibilityVersion（fCV）设置为`"4.0"` 或更早版本的MongoDB 2.6至MongoDB版本， 如果现有文档的索引条目超过，则MongoDB **不会**在集合上创建索引。`Maximum Index Key Length`

### 资源锁定

*在版本4.2.2中更改。*

对于MongoDB 4.2.2及更高版本，请`db.collection.reIndex()`在集合上获得排他（W）锁，并阻止对该集合进行其他操作，直到完成。

对于MongoDB 4.0.0到4.2.1，`db.collection.reIndex()` 获得全局排他（W）锁并在上阻止其他操作， `mongod`直到完成。

对于MongoDB 3.6及更早版本，这些操作 `db.collection.reIndex()`在数据库上获得排他（W）锁，并阻塞数据库上的其他操作，直到完成。

有关锁定MongoDB的更多信息，请参阅FAQ：并发。

> **也可以看看**
>
> 索引



译者：李冠飞

校对：