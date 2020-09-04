
# 读关注“linearizable”


 3.4版本中的新功能。

该查询返回的数据反映了在开始读操作之前完成的所有成功的经过多数确认的写操作。在返回结果之前，查询可以等待并发执行的写传播到大多数复制集成员。

如果大多数复制集成员在读取操作后崩溃并重新启动，则如果[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`true`的默认 state，则读取操作返回的文档是持久的。

当[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`时，MongoDB 不会等待 [`w: "majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority")写入在确认写入之前写入磁盘上日志。因此，`majority`写操作可能会在给定复制集中的大多数节点的瞬时丢失(即. 崩溃和重启)的事件中回滚。

您可以仅为主节点上的读操作指定可线性化的读关注。

可线性化读取关注保证仅在读取操作指定唯一标识单个文档的查询过滤器时才适用。

> **提示**
>
> 如果大多数数据承载成员不可用，请始终使用带有线性化读取问题的`maxTimeMS`。 `maxTimeMS`确保操作不会无限期地阻塞，而是确保在无法满足读取关注时操作返回错误。

## 因果一致的会话

对于因果一致会话，读关注**linearizable**不可用。

## 聚集限制

不能将[`$out`](https://docs.mongodb.com/master/reference/operator/aggregation/out/#pipe._S_out) 或 [`$merge`](https://docs.mongodb.com/master/reference/operator/aggregation/merge/#pipe._S_merge) 阶段与read关注点[`线性化`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")结合使用。也就是说，如果您为[`db.collection.aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate)指定了[`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")读关注，则不能在管道中包含这两个阶段。

## 实时订单

结合[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写关注， [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern."linearizable")读关注使多个线程可以在单个文档上执行读写操作，就好像单个线程实时执行了这些操作一样。也就是说，这些读写的相应计划被认为是线性的。

## 读取自己的写入

更改了3.6版本.

从 MongoDB 3.6 开始，如果写请求确认，则可以使用 [因果关系一致](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions)来读取您自己的写入。

在MongoDB 3.6之前，你必须使用 [`{ w: "majority" }`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority") 写关注点来发布写操作，然后使用[`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") 或[`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable") 的读关注点来执行读操作，以确保单个线程可以读取自己的写操作。

## 性能比较

与“多数”不同，“可线性化”的读关注点向辅助成员确认读操作是从能够用 [`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 写关注点确认写操作的主成员读取的。这样，线性化的读取可能比“多数”或“局部”读取要慢得多。

与[`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority")不同，[`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")的读关注点向辅助成员确认读操作是从能够用 [`{ w: "majority" }`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority")写关注点确认写操作的主成员读取的。[[1\]](https://docs.mongodb.com/master/reference/read-concern-linearizable/#edge-cases-2-primaries) 这样，线性化的读取可能比 [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") 或 [`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local")读取要慢得多。

总是使用可线性化读取关注的maxTimeMS，以防大多数数据承载成员不可用。maxTimeMS确保了操作不会无限期地阻塞，相反，它确保了如果读问题不能实现，操作会返回一个错误。

例如：

```shell
db.restaurants.find( { _id: 5 } ).readConcern("linearizable").maxTimeMS(10000)

db.runCommand( {
     find: "restaurants",
     filter: { _id: 5 },
     readConcern: { level: "linearizable" },
     maxTimeMS: 10000
} )
```

[[1\]](https://docs.mongodb.com/master/reference/read-concern-linearizable/#id1) 在某些情况下，一个复制集中的两个节点可能暂时认为它们是主节点，但它们中的一个最多能够完成[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写关注点的写操作。能够完成[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写操作的节点是当前主节点，而另一个节点是前主节点，它还没有意识到降级，通常是由于网络分区。当发生这种情况时，连接到前主服务器的客户机可能会观察到陈旧的数据，尽管已经请求了读首选项主服务器，并且对前主服务器的新写操作最终将回滚。



译者：杨帅

校对：杨帅
