# [ ](#)db.collection.totalSize（）

[]()

* `db.collection.` `totalSize` ()

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 集合中数据的总大小(以字节为单位)加上集合中每个索引的大小。<br/>如果压缩了集合数据(即WiredTiger 的默认值)，则返回的大小反映了集合数据的压缩大小。<br/>如果索引使用前缀压缩(即WiredTiger 的默认值)，则返回的大小反映索引的压缩大小。 |

返回的 value 是db.collection.storageSize()和db.collection.totalIndexSize()的总和(以字节为单位)。



译者：李冠飞

校对：