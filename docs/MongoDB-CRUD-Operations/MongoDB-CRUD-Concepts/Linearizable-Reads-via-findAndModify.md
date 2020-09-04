# 通过findAndModify可线性读取


## 概述

从复制集读取数据时，可能会读取过时(即可能并不能反映所有写道,发生前读操作)或不持久(即数据可能反映了写的状态还没有得到多数或复制集成员因此可以回滚)的数据，这取决于所使用的读取关注点。

从3.4版本开始，MongoDB引入了[可线性化](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")的读关注点，它返回的是持久的数据，不会过时。[可线性化](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")的读关注保证仅适用于读操作指定了唯一标识单个文档的查询筛选器。

本教程概述了一个替代过程，对于使用MongoDB 3.2的部署，该过程使用[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)来读取不过时且不能回滚的数据。对于MongoDB 3.4，尽管可以应用概述的过程，但请参阅[“线性化”]([https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern.%22linearizable%22](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern."linearizable"))阅读问题。


## 可线性通过findAndModify读取

此过程用于[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)读取不过期且无法回滚的数据。为此，该过程使用[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)具有[写关注](https://docs.mongodb.com/manual/reference/write-concern/#write-concern)的方法来修改文档中的伪字段。具体来说，该过程要求：

- [`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)使用**完全**匹配查询，并且**必须存在**[唯一索引](https://docs.mongodb.com/manual/core/index-unique/) **才能**满足该查询。
- [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)必须实际修改文档；即导致文档更改。
- [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)必须使用写关注 。[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")

> **[warning] 重要**
>
> “仲裁读取”过程比单纯使用读取问题要花费大量成本，[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")因为它会导致写入延迟而不是读取延迟。仅在绝对不过期的情况下才应使用此技术。


### 前提条件

本教程从名为**products**的集合中读取内容。使用以下操作初始化集合。

```shell
db.products.insert( [
   {
     _id: 1,
     sku: "xyz123",
     description: "hats",
     available: [ { quantity: 25, size: "S" }, { quantity: 50, size: "M" } ],
     _dummy_field: 0
   },
   {
     _id: 2,
     sku: "abc123",
     description: "socks",
     available: [ { quantity: 10, size: "L" } ],
     _dummy_field: 0
   },
   {
     _id: 3,
     sku: "ijk123",
     description: "t-shirts",
     available: [ { quantity: 30, size: "M" }, { quantity: 5, size: "L" } ],
     _dummy_field: 0
   }
] )
```

该集合中的文档包含一个虚拟字段`_dummy_field`，该字段 [`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)在本教程中将通过递增 。如果该字段不存在，则该[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)操作会将字段添加到文档中。该字段的目的是确保[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)对文档进行修改。
### 程序

#### 1.创建一个唯一索引。

在将用于指定[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)操作中完全匹配的字段上创建唯一索引。

本教程将在`sku`现场使用完全匹配。这样，在`sku`字段上创建唯一索引。

```shell
db.products.createIndex( { sku: 1 }, { unique: true } )
```

#### 2.使用`findAndModify`读取提交的数据。

使用该[`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)方法对要阅读的文档进行简单更新，然后返回修改后的文档。需要写关注。要指定要阅读的文档，必须使用唯一索引支持的完全匹配查询。[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")

下面的[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)操作在唯一索引的字段`sku`上指定精确匹配，并增加匹配文档中名为`_dummy_field`的字段。虽然不是必需的，但该命令的写操作还包括一个5000毫秒的[`wtimeout`](https://docs.mongodb.com/manual/reference/write-concern/#wc-wtimeout)值，以防止在写操作不能传播到大多数投票成员时永远阻塞操作。

```shell
var updatedDocument = db.products.findAndModify(
   {
     query: { sku: "abc123" },
     update: { $inc: { _dummy_field: 1 } },
     new: true,
     writeConcern: { w: "majority", wtimeout: 5000 }
   }
);
```

即使在副本集中的两个节点认为它们是主节点的情况下，也只有一个节点能够用[`w: "majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority").完成写操作。因此，只有当客户机连接到真正的主服务器来执行操作时，具有[“多数”](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority")写关注点的[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)方法才会成功。

由于仲裁读取过程只会增加文档中的虚拟字段，因此您可以安全地重复调用 [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify)，根据需要调整 [wtimeout](https://docs.mongodb.com/manual/reference/write-concern/#wc-wtimeout)。



译者：杨帅

校对：杨帅