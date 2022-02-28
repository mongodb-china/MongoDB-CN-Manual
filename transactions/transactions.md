
# 事务


在MongoDB中，对单个文档的操作是原子的。由于可以在单个文档结构中使用内嵌文档和数组来获得数据之间的关系，而不必跨多个文档和集合进行范式化，所以这种单文档原子性避免了许多实际场景中对多文档事务的需求。

对于那些需要对多个文档（在单个或多个集合中）进行原子性读写的场景，MongoDB支持多文档事务。而使用分布式事务，事务可以跨多个操作、集合、数据库、文档和分片使用。


# 事务API


------

➤ 使用右上角的**Select your language**下拉菜单来设置以下示例的语言。

------


此示例突出显示了事务API的关键组件。


该示例使用新的回调API来进行事务处理，其中涉及启动事务、执行指定的操作并提交（或在出错时中止）。新的回调API还包含针对`TransientTransactionError`或`UnknownTransactionCommitResult`提交错误的重试逻辑。


重要

- *推荐*。使用针对MongoDB部署版本更新的MongoDB驱动程序。对于MongoDB 4.2部署（副本集和分片集群上的事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。
- 使用驱动程序时，事务中的每个操作**必须**与会话相关联（即将会话传递给每个操作）。
- 事务中的操作使用[事务级读关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern)，[事务级写关注](https ://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)和[事务级读偏好](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)。
- 在MongoDB 4.2及更早版本中，你无法在事务中创建集合。如果在事务内部运行会导致文档插入的写操作（例如`insert`或带有`upsert: true`的更新操作），必须在**已存在**的集合上才能执行。
- 从MongoDB 4.4开始，你可以隐式或显式地在事务中创建集合。但是，必须使用针对4.4更新的MongoDB驱动程序。有关详细信息，请参阅[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

```
static bool
with_transaction_example (bson_error_t *error)
{
   mongoc_client_t *client = NULL;
   mongoc_write_concern_t *wc = NULL;
   mongoc_read_concern_t *rc = NULL;
   mongoc_read_prefs_t *rp = NULL;
   mongoc_collection_t *coll = NULL;
   bool success = false;
   bool ret = false;
   bson_t *doc = NULL;
   bson_t *insert_opts = NULL;
   mongoc_client_session_t *session = NULL;
   mongoc_transaction_opt_t *txn_opts = NULL;

   /* For a replica set, include the replica set name and a seedlist of the
    * members in the URI string; e.g.
    * uri_repl = "mongodb://mongodb0.example.com:27017,mongodb1.example.com:" \
    *    "27017/?replicaSet=myRepl";
    * client = test_framework_client_new (uri_repl);
    * For a sharded cluster, connect to the mongos instances; e.g.
    * uri_sharded =
    * "mongodb://mongos0.example.com:27017,mongos1.example.com:27017/";
    * client = test_framework_client_new (uri_sharded);
    */

   client = get_client ();

   /* Prereq: Create collections. */
   wc = mongoc_write_concern_new ();
   mongoc_write_concern_set_wmajority (wc, 1000);
   insert_opts = bson_new ();
   mongoc_write_concern_append (wc, insert_opts);
   coll = mongoc_client_get_collection (client, "mydb1", "foo");
   doc = BCON_NEW ("abc", BCON_INT32 (0));
   ret = mongoc_collection_insert_one (
      coll, doc, insert_opts, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   coll = mongoc_client_get_collection (client, "mydb2", "bar");
   doc = BCON_NEW ("xyz", BCON_INT32 (0));
   ret = mongoc_collection_insert_one (
      coll, doc, insert_opts, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }

   /* Step 1: Start a client session. */
   session = mongoc_client_start_session (client, NULL /* opts */, error);
   if (!session) {
      goto fail;
   }

   /* Step 2: Optional. Define options to use for the transaction. */
   txn_opts = mongoc_transaction_opts_new ();
   rp = mongoc_read_prefs_new (MONGOC_READ_PRIMARY);
   rc = mongoc_read_concern_new ();
   mongoc_read_concern_set_level (rc, MONGOC_READ_CONCERN_LEVEL_LOCAL);
   mongoc_transaction_opts_set_read_prefs (txn_opts, rp);
   mongoc_transaction_opts_set_read_concern (txn_opts, rc);
   mongoc_transaction_opts_set_write_concern (txn_opts, wc);

   /* Step 3: Use mongoc_client_session_with_transaction to start a transaction,
    * execute the callback, and commit (or abort on error). */
   ret = mongoc_client_session_with_transaction (
      session, callback, txn_opts, NULL /* ctx */, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }

   success = true;
fail:
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   bson_destroy (insert_opts);
   mongoc_read_concern_destroy (rc);
   mongoc_read_prefs_destroy (rp);
   mongoc_write_concern_destroy (wc);
   mongoc_transaction_opts_destroy (txn_opts);
   mongoc_client_session_destroy (session);
   mongoc_client_destroy (client);
   return success;
}

/* Define the callback that specifies the sequence of operations to perform
 * inside the transactions. */
static bool
callback (mongoc_client_session_t *session,
          void *ctx,
          bson_t **reply,
          bson_error_t *error)
{
   mongoc_client_t *client = NULL;
   mongoc_collection_t *coll = NULL;
   bson_t *doc = NULL;
   bool success = false;
   bool ret = false;

   client = mongoc_client_session_get_client (session);
   coll = mongoc_client_get_collection (client, "mydb1", "foo");
   doc = BCON_NEW ("abc", BCON_INT32 (1));
   ret =
      mongoc_collection_insert_one (coll, doc, NULL /* opts */, *reply, error);
   if (!ret) {
      goto fail;
   }
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   coll = mongoc_client_get_collection (client, "mydb2", "bar");
   doc = BCON_NEW ("xyz", BCON_INT32 (999));
   ret =
      mongoc_collection_insert_one (coll, doc, NULL /* opts */, *reply, error);
   if (!ret) {
      goto fail;
   }

   success = true;
fail:
   mongoc_collection_destroy (coll);
   bson_destroy (doc);
   return success;
}
```



同样请参阅：

有关[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell中的示例，请参阅[`mongo`Shell示例](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-mongo-shell-example)。



## 事务和原子性


说明

分布式事务和多文档事务

从MongoDB 4.2开始，这两个术语是同义词。分布式事务是指分片集群和副本集上的多文档事务。从MongoDB 4.2开始，多文档事务（无论是在分片集群上还是副本集上）也称为分布式事务。

对于多文档（在单个或多个集合中）读写上有原子性要求的场景，MongoDB提供了多文档事务支持：

- **在4.0版本**中，MongoDB支持副本集上的多文档事务。

- **在4.2版本**中，MongoDB引入了分布式事务，增加了对分片集群上多文档事务的支持，并合并了对副本集上多文档事务的现有支持。

  为了在MongoDB 4.2部署（副本集和分片集群）上使用事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。

Multi-document transactions are atomic (i.e. provide an "all-or-nothing" proposition):

多文档事务是原子的（即提供“全有或全无”的语义）：

- 当事务提交时，事务中所做的所有数据更改都将保存并在事务外部可见。也就是说，事务不会在回滚其他更改时提交其某些更改。

  在事务提交之前，事务中所做的数据更改在事务之外是不可见的。

  然而，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在分片中可见。例如，如果事务已提交并且写入操作1在分片A上可见，但写入操作2在分片B上尚不可见，则外部读关注为[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)的读操作可以读取写入操作1的结果，看不到写入操作2。

- 当事务中止时，事务中所做的所有数据更改都将被丢弃，而不会变得可见。例如，如果事务中的任何操作失败，事务就会中止，并且事务中所做的所有数据更改都将被丢弃，而不会变得可见。

重要

在大多数情况下，多文档事务比单文档写入会产生更大的性能成本，并且多文档事务的可用性不应替代有效的模型设计。对于许多场景，[反范式化数据模型（嵌入文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) 依然会是最适合你的数据和用例。也就是说，对于许多场景，适当地对数据进行建模可以最大限度地减少对多文档事务的需求。

有关其他事务使用注意事项（例如runtime限制和oplog大小限制），另请参阅[生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-Consideration/)。

TIP

同样请参阅:

[Commit期间的外部读操作](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-prod-consideration-outside-reads)


## 事务和操作



分布式事务可用于跨多个操作、集合、数据库、文档以及从MongoDB 4.2开始可以跨分片。


对于事务：

- 可以在**现有**集合上指定读/写（CRUD）操作。 有关CRUD操作的列表，请参阅[CRUD操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud)。

- 当使用[功能兼容版本(fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)`"4.4"`或更高版本时，可以在事务中创建集合和索引。详情请参考[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

- 事务中使用的集合可以位于不同的数据库中。

  提示

  你不能在跨分片的写事务中创建新集合。例如，如果你想对一个分片中已存在的集合进行写入且在另外一个不同的分片中隐式地创建集合，那么MongoDB无法在同一事务中执行这两种操作。

- 你不能写入[capped](https://docs.mongodb.com/manual/core/capped-collections/)集合。（从 MongoDB 4.2 开始）

- 你不能读/写`config`、`admin`或`local`数据库中的集合。

- 你不能写`system.*`集合。

- 你不能返回这类支持操作的查询计划（即`explain`）。

- 对于在事务外部创建的游标，你不能在事务内部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。

- 对于在事务内创建的游标，你不能在事务外调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。


- 从MongoDB 4.2开始，你不能将[`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)定义为[事务](https://docs.mongodb.com/manual/core/transactions/)中的第一个操作。

有关事务中不支持的操作列表，请参阅[受限操作](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)。

提示

在开始事务之前立即创建或删除集合时，如果在事务内访问该集合，注意使用写关注[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)来执行这些创建或删除操作，从而确保事务可以获取到所需要的锁。

提示

同样请参阅：

[事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



### 在事务中创建集合和索引


从MongoDB 4.4开始，使用[功能兼容性版本(fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)`"4.4"`，可以在[多文档事务](https://docs.mongodb.com/manual/core/transactions/)中创建集合和索引，除非事务是跨分片写入事务。如果使用`"4.2"`或更低版本，事务中**不允许**使用影响数据库目录的操作，例如创建或删除集合和索引。

当在事务内部创建一个集合时：

- 可以[隐式创建一个集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)，例如：
  - 针对一个不存在的集合上执行[插入操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)，或者
  - 针对一个不存在集合上执行带有`upsert: true`选项的[update/findAndModify操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)。
- 可以使用[`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create)命令或其帮助函数[`db.createCollection()`](https://docs.mongodb.com/manual /reference/method/db.createCollection/#mongodb-method-db.createCollection)[显式地创建一个集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)。

[在事务中创建索引](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)[[1\]](https: //docs.mongodb.com/manual/core/transactions/#footnote-create-existing-index)，要创建的索引需满足下面两者之一情况：

- 一个不存在的集合。集合的创建是作为操作的一部分。
- 先前在同一事务中创建的新空集合。

| [[1](https://docs.mongodb.com/manual/core/transactions/#ref-create-existing-index-id2)] | 还可以对现有索引运行[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)和[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes)来检查是否存在。这些操作会成功地返回且不会创建索引。|
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |


#### 限制


- 你不能在跨分片的写事务中创建新集合。例如，如果要对一个分片中已存在的集合执行写入操作且在另外一个不同的分片中隐式地创建集合，那么MongoDB无法在同一事务中执行这两种操作。

- 要在事务内显式创建集合或索引，事务读关注级别必须为[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)。显式创建是通过：

  | 命令                                                         | Method                                                       |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) |
  | [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) |

- 参数[`shouldMultiDocTxnCreateCollectionAndIndexes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes)必须为`true`（默认值）。在对分片集群设置参数时，请在所有分片上设置该参数。

提示

同样请参阅:

[受限制的操作](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)



### 计数操作

要在事务中执行计数操作，请使用[`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count)聚合阶段或带有[`$sum`](https ://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)表达式的[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)聚合阶段。

与4.0特性兼容的MongoDB驱动程序提供了一个集合级别的API`countDocuments(filter, options)`作为带有[`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)表达式的[`$group`](https://docs.mongodb.com/manual/reference/operator /aggregation/group/#mongodb-pipeline-pipe.-group)的帮助函数来执行计数。4.0驱动程序已弃用`count()` API。

从MongoDB4.0.3开始，[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell提供了在[`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments)中使用带有[`$sum`](https://docs.mongodb.com/ manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)表达式的[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)来执行计数的帮助函数。



### Distinct操作

为了在事务中执行一个distinct操作：

- 对于未分片的集合，可以使用[`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection .distinct)方法/[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)命令以及带有[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)阶段的聚合管道。

- 对于分片的集合，不能使用[`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)  方法或者[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)命令。

  要查找分片集合的不同值，请使用带[`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)阶段的聚合管道作为替代。例如：

  - 替代`db.coll.distinct("x")`，请使用：

    ```
    db.coll.aggregate([
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  - 替代`db.coll.distinct("x", { status: "A" })`，请使用：

    ```
    db.coll.aggregate([
       { $match: { status: "A" } },
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```


  管道将游标返回到文档：

  ```
  { "distinctValues" : [ 2, 3, 1 ] }
  ```

  迭代游标来访问结果集文档。



### 信息类操作

信息类操作命令，比如[`hello`](https://docs.mongodb.com/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#mongodb-dbcommand-dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#mongodb-dbcommand-dbcmd.connectionStatus)（以及它们的辅助函数）是被允许在事务中使用的。然而，它们不能作为事务中的第一个操作。



### 受限制的操作

*在4.4版本中变更。*

下列这些操作在事务中是不被允许的：

- 影响数据库catalog的操作，例如在创建或删除集合和索引时使用`"4.2"`或更低的[功能兼容版本（fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)。使用fcv`"4.4"`或更高版本，可以在事务中创建集合和索引，除非事务是跨分片写入事务。有关详细信息，请参阅[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。
- 在跨分片写入事务中创建新的集合。例如，如果在一个分片中对现有集合进行写入并在不同分片中隐式创建一个集合，则MongoDB无法在同一事务中执行这两种操作。
- [显式创建集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)，例如[`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)方法和索引，例如[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes)和[`db. collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)方法，当使用[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)以外的读取关注级别时。
- [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections)和[`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes)命令及其辅助函数。
- 其他非CRUD和非信息类操作，比如[`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser)，[`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter)，[`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count)等以及它们的辅助函数。

提示

同样请参阅：

- [待处理的DDL操作和事务](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
- [事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



## 事务和会话

- 事务是与某个会话相关联的；即你为一个会话启动一个事务。
- 在任何给定时间，一个会话最多可以有一个打开的事务。
- 使用驱动程序时，事务中的每个操作都必须与会话相关联。有关详细信息，请参阅你使用的驱动程序文档。
- 如果一个会话结束了并且它有一个打开的事务，则事务会中止。



## 读关注/写关注/读偏好


### 事务和读偏好

在事务中使用事务级[读偏好](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)的操作。

在使用驱动时，你可以在事务开始时设置事务级别的[读偏好](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)：

- 如果事务级别的读偏好没有设置，事务会使用会话级别的读偏好。
- 如果事务级别和会话级别的读偏好没有设置，事务使用客户端级别的读偏好。默认情况下，客户端级别的读偏好是[`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary)。

包含读操作的[多文档事务](https://docs.mongodb.com/manual/core/transactions/)必须使用[`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary)读偏好。在一个给定事务中的所有操作都必须路由到同一个成员。



### 事务和读关注

在事务中的操作会使用事务级[读关注](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)。也就是说，在事务内部忽略在集合和数据库级别设置的任何读关注。

可以在事务开始时设置事务级别的[读关注](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)。

- 如果事务级别的读关注没有设置，事务级的读关注默认为会话级的读关注。
- 如果事务级和会话级的读关注没有设置，事务级的读关注默认为客户端级的读关注。默认情况下，客户端级的读取关注是[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)用于针对主节点的读取。同样请参阅：
  - [事务和读偏好](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)
  - [默认MongoDB读关注/写关注](https://docs.mongodb.com/manual/reference/mongodb-defaults/)

事务支持下列的读关注级别：

#### `"local"`

- 读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)会返回节点最新可用的数据，但可能被回滚。
- 对于分片集群上的事务，[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)读关注不能保证数据来自同一个跨分片的快照视图。如果需要快照隔离，请使用 [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot)读关注。
- 从MongoDB 4.4开始，使用[功能兼容版本（fcv）](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)`"4.4"`或更高，可以在事务内[创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。如果[显式](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)地创建集合或索引，事务必须使用读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)。[隐式](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)地创建集合可使用任何适用于事务的读关注。

#### `"majority"`

- **如果**事务以[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，则读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)会返回已被副本集中大多数成员确认的数据（即数据不会被回滚）。
- 如果事务不用[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读关注**不能**保证读操作读取到大多数已提交的数据。
- 对于分片集群上的事务，[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)读关注不能保证数据来自同一个跨分片的快照视图。如果需要快照隔离，请使用[`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot)读关注。

#### `"snapshot"`

- **如果**事务以[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，则读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)会从一个大多数已提交数据的快照中返回数据。

- 如果事务不用[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)读关注**不能**保证读操作读取到大多数已提交的数据。

- 对于分片集群上的事务，数据的[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)视图**是**跨分片同步的。

  

### 事务和写关注

事务使用事务级[写关注](https://docs.mongodb.com/manual/reference/write-concern/)来提交写操作。事务内的写操作必须没有显式定义写关注，并使用默认的写关注。在提交时，然后使用事务级写关注提交写入。

提示

不要为事务内的单个写操作显式设置写关注。为事务内的单个写操作设置写关注会导致错误。 

可以在事务开始时设置事务级别的[写关注](https://docs.mongodb.com/manual/reference/write-concern/)：

- 如果事务级别的写关注没有设置，事务级写关注默认为提交的会话级写关注。
- 如果事务级写关注和会话级写关注没有设置，事务级写关注默认为客户端级写关注。默认情况下，客户端写关注为[`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)。也可以参考[默认MongoDB读关注/写关注](https://docs.mongodb.com/manual/reference/mongodb-defaults/)。

事务支持所有写关注[w](https://docs.mongodb.com/manual/reference/write-concern/#std-label-wc-w)的值，包括：

#### `w: 1`

- 写关注[`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)会在提交已经被应用到主节点后反馈确认结果。

  重要

  当使用[`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)提交，事务在发生故障时可能会回滚。

- 当使用[`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)写关注提交，事务级[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)的读关注**无法**保证事务中的读操作能读取大多数已提交的数据。

- 当使用 [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)写关注提交，事务级[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)的读关注**无法**保证事务中的读操作能使用大多数已提交数据的快照。

#### `w: "majority"`

- 写关注[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)会在提交的数据被应用到大多数（M）有投票权的成员后返回确认；即提交数据已被应用到主节点和（M-1）有投票权的从节点。
- 当使用[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)写关注提交时，事务级[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)读关注可以确保操作能读取到大多数已提交的数据。对于分片集群上的事务，这种大多数已提交数据的视图在分片之间不会同步。
- 当使用[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)写关注提交时，事务级[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)读关注可以确保证操作能获取来自大多数已提交数据的同步快照。

说明

不管[为事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)，分片集群事务的提交操作包括一部分使用了`{w: "majority", j: true}`写关注的操作。



## 通用信息


### 生产注意事项


关于使用事务的各种生产注意事项，请参阅[生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-sumption/)。另外，如果是分片集群，同样请参阅[生产注意事项（分片集群）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)。



### 仲裁节点


如果任何事务操作从包含仲裁节点的分片读取或写入，其写操作跨越多个分片的事务将出错并中止。

另请参阅[Disabled Read Concern Majority](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-disabled-rc-majority)，了解在分片上已禁用读关注 majority的事务限制。



### 禁用读关注majority



三成员PSA（主-从-仲裁）副本集或者拥有三成员PSA分片的分片集群可能已经禁用了读关注majority（[`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)或[`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)）

- 在分片集群上，

  如果事务涉及到具有[禁用读关注“majority”](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority)的分片，则不能对事务使用读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。你只能对事务使用读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)或者[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)。如果使用读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)，则事务会报错并中止。`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false`。如果事务的任何读取或写入操作涉及已禁用读关注`"majority"`的分片，则其跨越多个分片进行写入操作的事务会出错并中止。

- 在副本集上，

  可以定义读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)、[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)或者甚至在已[禁用读关注"majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority)的副本集上使用[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。但是，如果你计划迁移到有分片禁用读关注majority的分片集群上，可能希望避免使用读关注`"snapshot"`。

提示

要检查读关注"majority"是否被禁用，可以在[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)实例上运行[`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)并检查[`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads)字段。如果值为`false`，则表示读关注"majority"已禁用。

更多信息请参考[三成员PSA架构](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-psa)和[三成员PSA分片](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-psa)。



### 分片配置限制


不能在包含[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)设置为`false` 分片的分片集群上运行事务（例如包含使用了[内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)作为投票成员的分片）。

说明

不管[为事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)，分片集群事务的提交操作都会包含一部分使用了`{w: "majority", j: true}`写关注的操作。



### 诊断


MongoDB提供了多种事务相关指标：

| Source                                                       | Returns                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)方法中的[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-dbcommand-dbcmd.serverStatus)命令 | 返回[transactions](https://docs.mongodb.com/manual/reference/command/serverStatus/#std-label-server-status-transactions) 相关指标。 |
| [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-pipeline-pipe.-currentOp)聚合管道 | 如果操作作为事务的一部分则返回：[`$currentOp.transaction`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.transaction)。持有锁的[非活动会话](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#std-label-currentOp-stage-idleSessions)的信息会作为事务的一部分。[`$currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator) 是写入多个分片的分片事务的指标。 |
| [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#mongodb-method-db.currentOp)方法中的[`currentOp`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-dbcommand-dbcmd.currentOp)命令 | 如果操作作为事务的一部分则返回[`currentOp.transaction`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.transaction)。[`$currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator)是写入多个分片的分片事务的指标。 |
| [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)和[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)日志信息 | 在[`TXN`](https://docs.mongodb.com/manual/reference/log-messages/#mongodb-data-TXN)日志组件下包含慢事务的信息（即事务超过了[`operationProfiling.slowOpThresholdMs`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs) 阈值） |



### 功能兼容版本（FCV）


为了使用事务，部署架构中所有成员的[featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)至少为：

| 部署架构 | 最小`featureCompatibilityVersion` |
| :--------- | :------------------------------------ |
| 副本集     | `4.0`                                 |
| 分片集群   | `4.2`                                 |

为了检查成员的FCV，连接到成员并运行下面的命令：

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

更多信息详见[`setFeatureCompatibilityVersion`](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)参考页。




### 存储引擎


从MongoDB 4.2开始，[多文档事务](https://docs.mongodb.com/manual/core/transactions/)支持副本集和分片集群，其中：

- 主节点使用WiredTiger存储引擎，同时
- 从节点使用WiredTiger存储引擎或[in-memory](https://docs.mongodb.com/manual/core/inmemory/)存储引擎。

在MongoDB 4.0中，只有使用WiredTiger存储引擎的副本集支持事务。

说明

你不能在包含[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)设置为 `false` 分片的分片集群上运行事务，例如包含使用了[内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)作为投票成员的分片。





## 附加事务主题

- [驱动程序API](https://docs.mongodb.com/manual/core/transactions-in-applications/)
- [生产考虑因素](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
- [生产考虑因素（分片集群）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)
- [事务和操作](https://docs.mongodb.com/manual/core/transactions-operations/)
- 要了解有关何时使用事务以及它们是否支持你的用例的更多信息，请参阅[Are Transactions Right for You?](https://www.mongodb.com/presentations/are-transactions-right-for-you-)来自**MongoDB.live 2020**的演示。



原文链接：https://docs.mongodb.com/manual/core/transactions/

译者：李正洋
