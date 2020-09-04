# [ ](#)db.collection.totalIndexSize（）

[]()

* `db.collection.` `totalIndexSize` ()

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 集合的所有索引的总大小。<br/>如果索引使用前缀压缩(即WiredTiger 的默认值)，则返回的大小反映压缩大小。 |

此方法在collStats(即：db.collection.stats())操作的totalIndexSize输出周围提供包装。



译者：李冠飞

校对：