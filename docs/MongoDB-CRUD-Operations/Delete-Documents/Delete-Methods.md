
# 删除方法
MongoDB提供以下删除集合文档的方法：

|                                                              |                                                              |
| ------------------------------------------------------------ | :----------------------------------------------------------- |
| [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne) | 即使多个文档可能与指定过滤器匹配，也最多删除一个与指定过滤器匹配的文档。<br/>*3.2版中的新功能*<br /> |
| [db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany) | 删除所有与指定过滤器匹配的文档。<br />*3.2版中的新功能*<br /> |
| [db.collection.remove()](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#db.collection.remove) | 删除单个文档或与指定过滤器匹配的所有文档。                   |

## 附加方法

以下方法也可以从集合中删除文档:

* [`db.collection.findOneAndDelete()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndDelete/#db.collection.findOneAndDelete).<br /> [`findOneAndDelete()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#findandmodify-wrapper-sorted-remove)提供排序选项。该选项允许删除按指定 order 排序的第一个文档。

* [`db.collection.findAndModify()`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify).

  [`db.collection.findAndModify()`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 提供了一个排序选项。 该选项允许删除按指定顺序排序的第一个文档.

* [`db.collection.bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite).

有关更多方法和示例，请参见各个方法的参考页。



译者：杨帅

校对：杨帅