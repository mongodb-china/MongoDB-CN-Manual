# 批量写入操作

>  **在本页面**
> - [ 总览](https://docs.mongodb.com/manual/core/bulk-write-operations/#overview)
> - [ 有序 VS 无序操作](https://docs.mongodb.com/manual/core/bulk-write-operations/#ordered-vs-unordered-operations)
> - [bulkWrite()方法](https://docs.mongodb.com/manual/core/bulk-write-operations/#bulkwrite-methods)
> - [批量插入分片集合的策略](https://docs.mongodb.com/manual/core/bulk-write-operations/#strategies-for-bulk-inserts-to-a-sharded-collection)

## 总览
MongoDB使客户端能够批量执行写操作。 批量写入操作会影响单个集合。 MongoDB允许应用程序确定批量写入操作所需的可接受的确认级别。

*3.2版本新增*

[`db.collection.bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)方法提供了执行批量插入，更新和删除操作的能力。对于批量插入而言，MongoDB也支持[`db.collection.insertMany()`](https://docs.mongodb.com/master/reference/method/db.collection.insertMany/#db.collection.insertMany).

## 有序 VS 无序操作
批量写操作可以是有序的，也可以无序的。

使用操作的有序列表，MongoDB串行地执行操作。 如果在某个单独的写操作的处理过程中发生错误，MongoDB将直接返回而不再继续处理列表中任何剩余的写操作。 请参考[有序的批量写入](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-example-bulk-write-operation).

使用无序的操作列表，MongoDB可以并行地执行操作，但是不能保证此行为。 如果某个单独的写操作的处理过程中发生错误，MongoDB将继续处理列表中剩余的写操作。 请参考[无序的批量写入](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-example-unordered-bulk-write)。

在分片集合上执行有序的批量写操作通常比执行无序批量写操作要慢。这是因为对于有序列表而言，每个操作都必须等待上一个操作完成后才能执行。

默认情况下，[`bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite) 执行**有序的**写入。 要指定**无序的**写入，请在选项文档中设置**ordered：false**。

请参考[操作的执行](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-executionofoperations).

## bulkWrite()方法
[`bulkWrite()`](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)支持如下操作：

- [insertOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-insertone)
- [updateOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-updateonemany)
- [updateMany](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-updateonemany)
- [replaceOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-replaceone)
- [deleteOne](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-deleteonemany)
- [deleteMany](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-write-operations-deleteonemany)

每个写操作都以数组中的文档形式被传递给[[`bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite) 

例如，下面执行多个写操作:

**characters**集合包含以下文档:

```powershell
{ "_id" : 1, "char" : "Brisbane", "class" : "monk", "lvl" : 4 },
{ "_id" : 2, "char" : "Eldon", "class" : "alchemist", "lvl" : 3 },
{ "_id" : 3, "char" : "Meldane", "class" : "ranger", "lvl" : 3 }
```
接下来的[`bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)将在此集合上执行批量写入的操作。
```powershell
try {
   db.characters.bulkWrite(
      [
         { insertOne :
            {
               "document" :
               {
                  "_id" : 4, "char" : "Dithras", "class" : "barbarian", "lvl" : 4
               }
            }
         },
         { insertOne :
            {
               "document" :
               {
                  "_id" : 5, "char" : "Taeln", "class" : "fighter", "lvl" : 3
               }
            }
         },
         { updateOne :
            {
               "filter" : { "char" : "Eldon" },
               "update" : { $set : { "status" : "Critical Injury" } }
            }
         },
         { deleteOne :
            { "filter" : { "char" : "Brisbane"} }
         },
         { replaceOne :
            {
               "filter" : { "char" : "Meldane" },
               "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl" : 4 }
            }
         }
      ]
   );
}
catch (e) {
   print(e);
}
```
该操作将返回如下的结果：
```powershell
{
   "acknowledged" : true,
   "deletedCount" : 1,
   "insertedCount" : 2,
   "matchedCount" : 2,
   "upsertedCount" : 0,
   "insertedIds" : {
      "0" : 4,
      "1" : 5
   },
   "upsertedIds" : {

   }
}
```
想了解更多例子，请参考[`bulkWrite() 示例`](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#bulkwrite-example-bulk-write-operation).

## 批量插入分片集合的策略
大容量插入操作(包括初始数据插入或例程数据导入)可能会影响分片集群的性能。对于批量插入，考虑以下策略:

###  对分片集合进行预拆分
如果分片集合为空，则该集合只有一个存储在单个分片上的初始数据块，MongoDB必须花一些时间来接收数据，创建拆分并将拆分的块分发到其他分片上。为了避免这种性能开销，您可以对分片集合进行预拆分，请参考 [分片集群中的数据块拆分](https://docs.mongodb.com/master/tutorial/split-chunks-in-sharded-cluster/)中的描述。

###  对mongos的无序写入
要提高对分片集群的写入性能，请使用带有可选参数`ordered:false`的[`bulkWrite()`](https://docs.mongodb.com/master/reference/method/db.collection.bulkWrite/#db.collection.bulkWrite)方法。[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 会尝试同时将写入发送到多个分片。对于空集合，请首先按照[分片集群中的数据块拆分](https://docs.mongodb.com/master/tutorial/split-chunks-in-sharded-cluster/)中描述的进行集合的预拆分。

###  避免单调插入带来的瓶颈
如果您的分片键再插入过程中时单调增加的，那么所有插入的数据都会插入到该分片集合的最后一个数据块中，也就是说会落到某单个分片上。因此，集群的插入能力将永远不会超过该单个跟片的插入性能（木桶的短板原理）。

如果插入量大于单个分片可以处理的数据量，并且无法避免单调增加的分片键，那么可以考虑对应用程序进行如下修改：

- 翻转分片键的二进制位。这样可以保留信息的同时避免插入顺序与递增插入值之间的关联性。
- 交换第一个和最后16比特来实现“随机”插入。

**示例**

下面的C++例子中，交换生成的[`BSON`](https://docs.mongodb.com/master/reference/glossary/#term-bson) [`ObjectIds`](https://docs.mongodb.com/master/reference/glossary/#term-objectid)的前导和后16位字，使它们不再单调递增。

```powershell
using namespace mongo;
OID make_an_id() {
  OID x = OID::gen();
  const unsigned char *p = x.getData();
  swap( (unsigned short&) p[0], (unsigned short&) p[10] );
  return x;
}

void foo() {
  // create an object
  BSONObj o = BSON( "_id" << make_an_id() << "x" << 3 << "name" << "jane" );
  // now we may insert o into a sharded collection
}
```
另请参考

[分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-shard-keys)来获得如何选择分片键的相关信息。另请参考【分片键】（尤其是其中【[选择分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-operations-and-reliability)】的相关章节）

关于选择[分片键](https://docs.mongodb.com/manual/core/sharding-shard-key/#sharding-internals-shard-keys)的信息。还请参阅[分片键内部](https://docs.mongodb.com/master/core/sharding-shard-key/#sharding-internals-shard-keys)(特别是，[选择一个切分键](https://docs.mongodb.com/master/core/sharding-shard-key/#sharding-internals-operations-and-reliability))。



译者：杨帅 刘翔

校验：杨帅
