# 插入方法

**MongoDB 提供了以下方法将文件插入集合**：

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) | 将单个文档插入到集合中。                                     |
| [db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | [db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)将多个文件插入集合中。 |
| [db.collection.insert()](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert) | [db.collection.insert()](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert)将单个文档或多个文档插入到集合中。 |

## 插入的其他方法

以下方法还可以向集合中添加新文档：

*   与`upsert: true`选项一起使用时[db.collection.update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/#db.collection.update)。

*   与`upsert: true`选项一起使用时[db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)。

*   与`upsert: true`选项一起使用时[db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)。

*   与`upsert: true`选项一起使用时[db.collection.findAndModify()](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)。

*   与`upsert: true`选项一起使用时[db.collection.findOneAndUpdate()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#db.collection.findOneAndUpdate)。

*   与`upsert: true`选项一起使用时[db.collection.findOneAndReplace()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#db.collection.findOneAndReplace)。

*   [db.collection.save()](https://docs.mongodb.com/manual/reference/method/db.collection.save/#db.collection.save).

*   [db.collection.bulkWrite()](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite).

有关更多信息和示例，请参阅方法的各个 reference 页面。



译者：杨帅

校对：杨帅