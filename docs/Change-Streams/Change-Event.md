# Change Events

**在本页面**


Change Events  
insert Event  
update Event  
replace Event  
delete Event  
drop Event  
rename Event  
dropDatabase Event  
invalidate Event  


## Change Events  
以下文档表示变更流响应文档可以具有的所有可能的字段。
```
{
   _id : { <BSON Object> },
   "operationType" : "<operation>",
   "fullDocument" : { <document> },
   "ns" : {
      "db" : "<database>",
      "coll" : "<collection>"
   },
   "to" : {
      "db" : "<database>",
      "coll" : "<collection>"
   },
   "documentKey" : { "_id" : <value> },
   "updateDescription" : {
      "updatedFields" : { <document> },
      "removedFields" : [ "<field>", ... ]
   }
   "clusterTime" : <Timestamp>,
   "txnNumber" : <NumberLong>,
   "lsid" : {
      "id" : <UUID>,
      "uid" : <BinData>
   }
}
```
有些字段仅适用于某些操作，例如更新。下表描述了变更流响应文档中的每个字段：


| Field                                                        | Type       | Description                                                  |
| :----------------------------------------------------------- | :--------- | :----------------------------------------------------------- |
| [_id](https://docs.mongodb.com/master/reference/change-events/#change-stream-event-id) | document   | Metadata related to the operation. Acts as the `resumeToken` for the `resumeAfter` parameter when resuming a change stream.copycopied`{   "_data" : <BinData|hex string> } `The `_data` type depends on the MongoDB versions and, in some cases, the feature compatibility version (fcv) at the time of the change stream’s opening/resumption. For details, see [Resume Tokens](https://docs.mongodb.com/master/changeStreams/#change-stream-resume-token). |
| `operationType`                                              | string     | The type of operation that occurred. Can be any of the following values:`insert``delete``replace``update``drop``rename``dropDatabase``invalidate` |
| `fullDocument`                                               | document   | The document created or modified by the `insert`, `replace`, `delete`, `update` operations (i.e. CRUD operations).For `insert` and `replace` operations, this represents the new document created by the operation.For `delete` operations, this field is omitted as the document no longer exists.For `update` operations, this field only appears if you configured the change stream with `fullDocument` set to `updateLookup`. This field then represents the most current majority-committed version of the document modified by the update operation. This document may differ from the changes described in `updateDescription` if other majority-committed operations modified the document between the original update operation and the full document lookup. |
| `ns`                                                         | document   | The namespace (database and or collection) affected by the event. |
| `ns.db`                                                      | string     | The name of the database.                                    |
| `ns.coll`                                                    | string     | The name of the collection.For `dropDatabase` operations, this field is omitted. |
| `to`                                                         | document   | When `operationType : rename`, this document displays the new name for the `ns` collection. This document is omitted for all other values of `operationType`. |
| `to.db`                                                      | string     | The new name of the database.                                |
| `to.coll`                                                    | string     | The new name of the collection.                              |
| `documentKey`                                                | document   | A document that contains the `_id` of the document created or modified by the `insert`, `replace`, `delete`, `update` operations (i.e. CRUD operations). For sharded collections, also displays the full shard key for the document. The `_id` field is not repeated if it is already a part of the shard key. |
| `updateDescription`                                          | document   | A document describing the fields that were updated or removed by the update operation.This document and its fields only appears if the `operationType` is `update`. |
| `updateDescription.updatedFields`                            | document   | A document whose keys correspond to the fields that were modified by the update operation. The value of each field corresponds to the new value of those fields, rather than the operation that resulted in the new value. |
| `updateDescription.removedFields`                            | array      | An array of fields that were removed by the update operation. |
| `clusterTime`                                                | Timestamp  | The timestamp from the oplog entry associated with the event.For events that happened as part of a [multi-document transaction](https://docs.mongodb.com/master/core/transactions/), the associated change stream notifications will have the same `clusterTime` value, namely the time when the transaction was committed.On a sharded cluster, events that occur on different shards can have the same `clusterTime` but be associated with different transactions or even not be associcated with any transaction. To identify events for a single transaction, you can use the combination of `lsid` and `txnNumber` in the change stream event document.*New in version 4.0.* |
| `txnNumber`                                                  | NumberLong | The transaction number.Only present if the operation is part of a [multi-document transaction](https://docs.mongodb.com/master/core/transactions/).*New in version 4.0.* |
| `lsid`                                                       | Document   | The identifier for the session associated with the transaction.Only present if the operation is part of a [multi-document transaction](https://docs.mongodb.com/master/core/transactions/).*New in version 4.0.* |

## insert事件
以下示例说明了一个insert事件：
```
{
   _id: { < Resume Token > },
   operationType: 'insert',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   documentKey: {
      userName: 'alice123',
      _id: ObjectId("599af247bb69cd89961c986d")
   },
   fullDocument: {
      _id: ObjectId("599af247bb69cd89961c986d"),
      userName: 'alice123',
      name: 'Alice'
   }
}
```
该documentKey字段包括_id和userName 字段。这表示engineering.users集合已分片，并且在userName和上都有分片键_id。

该fullDocument文档表示插入时文档的版本。

## update事件
以下示例说明了一个update事件：
```
{
   _id: { < Resume Token > },
   operationType: 'update',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   documentKey: {
      _id: ObjectId("58a4eb4a30c75625e00d2820")
   },
   updateDescription: {
      updatedFields: {
         email: 'alice@10gen.com'
      },
      removedFields: ['phoneNumber']
   }
}
```
以下示例说明了update使用选项打开的变更流的事件：fullDocument : updateLookup
```
{
   _id: { < Resume Token > },
   operationType: 'update',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   documentKey: {
      _id: ObjectId("58a4eb4a30c75625e00d2820")
   },
   updateDescription: {
      updatedFields: {
         email: 'alice@10gen.com'
      },
      removedFields: ['phoneNumber']
   },
   fullDocument: {
      _id: ObjectId("58a4eb4a30c75625e00d2820"),
      name: 'Alice',
      userName: 'alice123',
      email: 'alice@10gen.com',
      team: 'replication'
   }
}
```
该fullDocument文档代表了更新文档的最新多数批准版本。该fullDocument文档可能与更新操作时的文档有所不同，具体取决于在更新操作和文档查找之间发生的交错多数授权操作的数量。

## replace事件
以下示例说明了一个replace事件：
```
{
   _id: { < Resume Token > },
   operationType: 'replace',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   documentKey: {
      _id: ObjectId("599af247bb69cd89961c986d")
   },
   fullDocument: {
      _id: ObjectId("599af247bb69cd89961c986d"),
      userName: 'alice123',
      name: 'Alice'
   }
}
```
一个replace操作使用update命令，并且包括两个阶段：

使用documentKey和删除原始文档
使用相同的插入新文档 documentkey
在fullDocument一个的replace事件表示替换文件的插入后的文件。

## delete事件
以下示例说明了一个delete事件：
```
{
   _id: { < Resume Token > },
   operationType: 'delete',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   documentKey: {
      _id: ObjectId("599af247bb69cd89961c986d")
   }
}
```
该fullDocument文档被省略，因为在更改流游标将delete事件发送到客户端时，该文档不再存在。

## drop事件
版本4.0.1中的新功能。

一个drop在集合从数据库中删除发生的事件。以下示例说明了一个drop事件：

```
{
   _id: { < Resume Token > },
   operationType: 'drop',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   }
}
```
一个drop事件导致一个无效事件 变革流张开攻击它的ns集合。

## rename事件
版本4.0.1中的新功能。

一个rename在集合重命名发生的事件。以下示例说明了一个rename事件：
```
{
   _id: { < Resume Token > },
   operationType: 'rename',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering',
      coll: 'users'
   },
   to: {
      db: 'engineering',
      coll: 'people'
   }
}
```
一个rename事件导致一个 无效事件的流变化对打开的ns集合或to集合。

## dropDatabase事件
版本4.0.1中的新功能。

一个dropDatabase当数据库被丢弃发生的事件。以下示例说明了一个dropDatabase事件：
```
{
   _id: { < Resume Token > },
   operationType: 'dropDatabase',
   clusterTime: <Timestamp>,
   ns: {
      db: 'engineering'
   }
}
```

一个dropDatabase事件导致一个 无效事件的流变化对打开的ns.db数据库。

## invalidate事件
以下示例说明了一个invalidate事件：

```
{
   _id: { < Resume Token > },
   operationType: 'invalidate',
   clusterTime: <Timestamp>
}
```
对于针对集合打开的变更流，影响监视的集合的 放置事件， 重命名事件或 dropDatabase事件导致 无效事件。

对于针对数据库打开的变更流，影响受监视数据库的 dropDatabase事件将导致 invalidate事件。

invalidate 事件关闭更改流游标。

resumeAfter在无效事件（例如，集合删除或重命名）关闭流之后，您不能用来恢复更改 流。从MongoDB 4.2开始，您可以使用 startAfter在invalidate事件之后启动新的更改流。

译者：wh
