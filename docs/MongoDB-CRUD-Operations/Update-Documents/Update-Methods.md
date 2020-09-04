
# 更新方法
MongoDB提供了以下方法来更新集合中的文档：

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne) | 即使多个文档可能与指定的过滤器匹配，最多更新与指定的过滤器匹配的单个文档。<br />*3.2版中的新功能* |
| [db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany) | 更新所有与指定过滤器匹配的文档。<br />*3.2版中的新功能*      |
| [db.collection.replaceOne()](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne) | 即使多个文档可能与指定过滤器匹配，也最多替换一个与指定过滤器匹配的文档。<br />*3.2版中的新功能* |
| [db.collection.update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update) | 更新或替换与指定过滤器匹配的单个文档，或更新与指定过滤器匹配的所有文档。<br />默认情况下，[db.collection.update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update)方法更新单个文档。 要更新多个文档，请使用**multi**选项。 |

## 附加方法

以下方法还可以更新集合中的文档：

- [db.collection.findOneAndReplace()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace).
- [db.collection.findOneAndUpdate()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate).
- [db.collection.findAndModify()](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify).
- [db.collection.save()](https://docs.mongodb.com/manual/reference/method/db.collection.save/#db.collection.save).
- [db.collection.bulkWrite()](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite).

有关更多方法和示例，请参见各个方法的参考页。



译者：杨帅

校对：杨帅