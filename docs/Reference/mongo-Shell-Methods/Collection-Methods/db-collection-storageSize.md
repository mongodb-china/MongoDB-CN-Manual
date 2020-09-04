# [ ](#)db.collection.storageSize（）

[]()

* `db.collection.` `storageSize` ()

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 分配给此集合以进行文档存储的总存储量。<br/>如果压缩了集合数据(即WiredTiger 的默认值)，则存储大小反映压缩大小，可能小于db.collection.dataSize()返回的 value。 |

在collStats(即：db.collection.stats())输出的storageSize字段周围提供 wrapper。



译者：李冠飞

校对：