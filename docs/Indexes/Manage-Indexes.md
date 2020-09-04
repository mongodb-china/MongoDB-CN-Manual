## 管理索引

- **Mongo Shell**

**在本页面**

- [查看现有索引](#查看)
- [删除索引](#删除)
- [修改索引](#修改)
- [在分片中查找不一致的索引](#不一致)

此页显示如何管理现有索引。有关创建索引的说明，请参阅特定索引类型页。

### <span id="查看">查看现有索引</span>

以下部分提供了查看集合或整个数据库上现有索引的方法。

#### 列出集合上的所有索引

要返回一个集合上所有索引的列表，使用[`db. collections . getindexes ()`](https://docs.mongodb.com/master/reference/method/db.collection.getindexes)方法或类似的[驱动程序的方法](https://docs.mongodb.com/drivers/)。

例如，要查看`people`集合上的所有索引，运行以下命令:

```powershell
db.people.getIndexes()
```

#### 列出数据库的所有索引

在[` mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中,可以使用以下操作列出数据库中所有的集合索引:

```powershell
db.getCollectionNames().forEach(function(collection) {
   indexes = db[collection].getIndexes();
   print("Indexes for " + collection + ":");
   printjson(indexes);
});
```

从3.0版本开始，MongoDB不再支持对系统的直接访问。索引集合，以前用于列出数据库中的所有索引。

#### 列出特定类型的索引

列出所有索引的类型(例如[散列](https://docs.mongodb.com/master/core/index-hashed/),[文本](https://docs.mongodb.com/master/core/index-text/))集合在所有数据库,您可以使用以下操作在[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)shell:

```powershell
// The following finds all hashed indexes

db.adminCommand("listDatabases").databases.forEach(function(d){
   let mdb = db.getSiblingDB(d.name);
   mdb.getCollectionInfos({ type: "collection" }).forEach(function(c){
      let currentCollection = mdb.getCollection(c.name);
      currentCollection.getIndexes().forEach(function(idx){
        let idxValues = Object.values(Object.assign({}, idx.key));

        if (idxValues.includes("hashed")) {
          print("Hashed index: " + idx.name + " on " + idx.ns);
          printjson(idx);
        };
      });
   });
});
```

### <span id="删除">删除索引</span>

MongoDB提供了两种方法从集合中删除索引:

- [`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)
- [`db.collection.dropIndexes()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndexes/#db.collection.dropIndexes)

#### 删除特定的指数

要删除一个索引，使用[`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)方法。

例如，下面的操作删除了 **accounts** 集合中的 **tax-id** 字段的升序索引:

```powershell
db.accounts.dropIndex( { "tax-id": 1 } )
```

该操作返回一个文档，其中显示了该操作的状态:

```powershell
{ "nIndexesWas" : 3, "ok" : 1 }
```

其中`nIndexesWas`的值反映了*在删除这个索引之前*的索引数量。

对于[文本](https://docs.mongodb.com/manual/core/index-text/)索引，将索引名称传递给 [`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#db.collection.dropIndex)方法。有关详细信息，请参见[使用索引名称删除文本索引](https://docs.mongodb.com/manual/tutorial/avoid-text-index-name-limit/#drop-text-index)。

> 注意
>
> 从MongoDB 4.2开始，[' db.collection.dropIndexes() '](https://docs.mongodb.com/master/reference/method/db.collection.dropIndexes/#db.collection.dropIndexes)可以接受一个索引名称数组。

#### 删除所有索引

你也可以使用[`db. collections . dropindexes () `](https://docs.mongodb.com/master/reference/method/db.collection.dropIndexes/#db.collection.dropIndexes)从一个集合中删除[_id索引](https://docs.mongodb.com/master/indexes/#index-type-id)之外的所有索引。

例如，下面的命令从 **accounts** 集合中删除所有索引:

```powershell
db.accounts.dropIndexes()
```

这些shell助手提供了[`dropIndexes 数据库命令`](https://docs.mongodb.com/master/reference/glossary/# term-databs-command)的包装器。您的[客户端库](https://docs.mongodb.com/ecostem/drivers)可能有一个不同的或额外的接口用于这些操作。

### <span id="修改">修改索引</span>

要修改现有索引，您需要删除并重新创建索引。[TTL索引](https://docs.mongodb.com/manual/core/index-ttl/)是该规则的例外 ，可以通过[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)命令与[`index`](https://docs.mongodb.com/manual/reference/command/collMod/#index)收集标志一起 对其进行修改。

### <span id="不一致">在分片中查找不一致的索引</span>

如果分片集合在每个包含该分片块的分片上没有完全相同的索引（包括索引选项），则该集合具有不一致的索引。虽然在正常操作中不应该出现索引不一致的情况，但也会出现索引不一致的情况，例如:

- 当用户创建具有`unique`键约束的索引并且一个分片包含具有重复文档的块时。在这种情况下，创建索引操作可能会在没有重复的分片上成功，但在没有重复的分片上不会成功。
- 当用户创建一个索引碎片在对面(滚动的方式[(即手动构建跨多个碎片索引一个接一个地)](https://docs.mongodb.com/master/tutorial/build-indexes-on-sharded-clusters/)但是无论未能构建相关碎片或是不正确的索引构建索引与不同的规范。

从MongoDB 4.2.6,[配置服务器](https://docs.mongodb.com/master/core/sharded-cluster-config-servers/)主,默认情况下,检查索引不一致在分片的碎片集合,和命令[("serverStatus")](https://docs.mongodb.com/master/reference/command/serverStatus/ # dbcmd.serverStatus),主要配置服务器上运行时,返回字段[`shardedIndexConsistency`](https://docs.mongodb.com/master/reference/command/serverStatus/#serverstatus.shardedIndexConsistency)来报告索引不一致的分片集合的数量。

如果[`shardedIndexConsistency`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.shardedIndexConsistency)报告任何索引不一致，则可以对分片集合运行以下管道，直到找到不一致为止。

> 注意
>
> 下面的管道用于MongoDB 4.2.4及以上版本。

1. 定义以下[聚合管道](https://docs.mongodb.com/manual/core/aggregation-pipeline/):

   ```powershell
const pipeline = [
       // Get indexes and the shards that they belong to.
       {$indexStats: {}},
       // Attach a list of all shards which reported indexes to each document from $indexStats.
       {$group: {_id: null, indexDoc: {$push: "$$ROOT"}, allShards: {$addToSet: "$shard"}}},
       // Unwind the generated array back into an array of index documents.
       {$unwind: "$indexDoc"},
       // Group by index name.
       {
           $group: {
               "_id": "$indexDoc.name",
               "shards": {$push: "$indexDoc.shard"},
               // Convert each index specification into an array of its properties
               // that can be compared using set operators.
               "specs": {$push: {$objectToArray: {$ifNull: ["$indexDoc.spec", {}]}}},
               "allShards": {$first: "$allShards"}
           }
       },
       // Compute which indexes are not present on all targeted shards and
       // which index specification properties aren't the same across all shards.
       {
           $project: {
               missingFromShards: {$setDifference: ["$allShards", "$shards"]},
               inconsistentProperties: {
                    $setDifference: [
                        {$reduce: {
                            input: "$specs",
                            initialValue: {$arrayElemAt: ["$specs", 0]},
                            in: {$setUnion: ["$$value", "$$this"]}}},
                        {$reduce: {
                            input: "$specs",
                            initialValue: {$arrayElemAt: ["$specs", 0]},
                            in: {$setIntersection: ["$$value", "$$this"]}}}
                    ]
                }
           }
       },
       // Only return output that indicates an index was inconsistent, i.e. either a shard was missing
       // an index or a property on at least one shard was not the same on all others.
       {
           $match: {
               $expr:
                   {$or: [
                       {$gt: [{$size: "$missingFromShards"}, 0]},
                       {$gt: [{$size: "$inconsistentProperties"}, 0]},
                   ]
               }
           }
       },
       // Output relevant fields.
       {$project: {_id: 0, indexName: "$$ROOT._id", inconsistentProperties: 1, missingFromShards: 1}}
   ];
   ```
   
2. 运行要测试的分片集合的聚合管道。例如，要测试分片集合是否测试。在相关的碎片上有不一致的索引:

   ```powershell
db.getSiblingDB("test").reviews.aggregate(pipeline)
   ```
   
   如果集合的索引不一致，则该集合的聚合将返回关于不一致索引的详细信息:

   ```powershell
{ "missingFromShards" : [ "shardB" ], "inconsistentProperties" : [ ], "indexName" : "page_1_score_1" }
   { "missingFromShards" : [ ], "inconsistentProperties" : [ { "k" : "expireAfterSeconds", "v" : 60 }, { "k" : "expireAfterSeconds", "v" : 600 } ], "indexName" : "reviewDt_1" }
   ```


   返回的文档指出了分片集合 **test.reviews** 的两个不一致之处:

   1. **shardB**上的集合中缺少一个名为`page_1_score_1`的索引。
2. 一个名为`reviewDt_1`的索引在集合的各个分片上具有不一致的属性，特别是**expireAfterSeconds**属性不同。

**要解决特定分片集合中缺少索引的不一致问题**

​	从受影响的分片上的集合中删除不正确的索引，然后重建索引。要重建索引，您可以：

- 在受影响的分片上为集合执行[滚动索引构建](https://docs.mongodb.com/manual/tutorial/build-indexes-on-sharded-clusters/)。

  或者

- 从一个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例发出一个索引构建 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)。该操作仅在没有索引的分片上构建集合的索引。

**要解决索引属性在各个分片之间的差异**

​	从受影响的分片上的集合中删除不正确的索引，并重新构建索引。重建索引，你可以:

* 在受影响的碎片上为集合执行[滚动索引构建](https://docs.mongodb.com/manual/tutorial/build-indexes-on-sharded-clusters/)。

  或者

* 从一个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos) 实例发出一个索引构建 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)。该操作仅在没有索引的碎片上构建集合的索引。

或者，如果不一致是该`expireAfterSeconds`属性，则可以运行[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)命令以更新秒数，而不是删除并重建索引。