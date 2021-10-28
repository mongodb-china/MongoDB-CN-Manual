# Transactions (Version 5.0)

# 事务

In MongoDB, an operation on a single document is atomic. Because you can use embedded documents and arrays to capture relationships between data in a single document structure instead of normalizing across multiple documents and collections, this single-document atomicity obviates the need for multi-document transactions for many practical use cases.

For situations that require atomicity of reads and writes to multiple documents (in a single or multiple collections), MongoDB supports multi-document transactions. With distributed transactions, transactions can be used across multiple operations, collections, databases, documents, and shards.



在 MongoDB 中，对单个文档的操作是原子的。 因为你可以使用内嵌的文档和数组来获取单个文档结构中数据之间的关系，而不是跨多个文档和集合进行规范化，所以这种单文档原子性避免了许多实际用例对多文档事务的需求。

对于需要对多个文档（在单个或多个集合中）进行原子性读写的情况，MongoDB 支持多文档事务。 通过分布式事务，事务可以跨多个操作、集合、数据库、文档和分片使用。



## Transactions API

# 事务 API

------

➤ Use the **Select your language** drop-down menu in the upper-right to set the language of the following example.

------

➤ 使用右上角的**Select your language**下拉菜单来设置以下示例的语言。

------

This example highlights the key components of the transactions API.

此示例突出显示了事务 API 的关键组件。

The example uses the new callback API for working with transactions, which starts a transaction, executes the specified operations, and commits (or aborts on error). The new callback API also incorporates retry logic for `TransientTransactionError` or `UnknownTransactionCommitResult` commit errors.

该示例使用新的回调 API 来处理事务，它启动事务、执行指定的操作并提交（或在出错时中止）。 新的回调 API 还包含针对 `TransientTransactionError` 或 `UnknownTransactionCommitResult` 提交错误的重试逻辑。

IMPORTANT

- *Recommended*. Use the MongoDB driver updated for the version of your MongoDB deployment. For transactions on MongoDB 4.2 deployments (replica sets and sharded clusters), clients **must** use MongoDB drivers updated for MongoDB 4.2.
- When using the drivers, each operation in the transaction **must** be associated with the session (i.e. pass in the session to each operation).
- Operations in a transaction use [transaction-level read concern](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern), [transaction-level write concern](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern), and [transaction-level read preference](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference).
- In MongoDB 4.2 and earlier, you cannot create collections in transactions. Write operations that result in document inserts (e.g. `insert` or update operations with `upsert: true`) must be on **existing** collections if run inside transactions.
- Starting in MongoDB 4.4, you can create collections in transactions implicitly or explicitly. You must use MongoDB drivers updated for 4.4, however. See [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) for details.

重要

- *推荐*。 使用针对 MongoDB 部署版本更新的 MongoDB 驱动程序。 对于 MongoDB 4.2 部署（副本集和分片集群）上的事务，客户端**必须**使用为 MongoDB 4.2 更新的 MongoDB 驱动程序。
- 使用驱动程序时，事务中的每个操作**必须**与会话相关联（即将会话传递给每个操作）。
- 事务中的操作使用[事务级读关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern)，[事务级写关注](https ://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) 和 [事务级读偏好](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)。
- 在 MongoDB 4.2 及更早版本中，你无法在事务中创建集合。 如果在事务内部运行会导致文档插入的写操作（例如 `insert` 或带有 `upsert: true` 的更新操作），必须在**已存在**的集合上才能执行。
- 从 MongoDB 4.4 开始，你可以隐式或显式地在事务中创建集合。 但是，您必须使用针对 4.4 更新的 MongoDB 驱动程序。 有关详细信息，请参阅 [在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

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



TIP

See also:

For an example in [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell, see [`mongo` Shell Example](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-mongo-shell-example).

也可以参考：

有关 [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell 中的示例，请参阅 [`mongo` Shell 示例](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-mongo-shell-example)。



## Transactions and Atomicity

## 事务和原子性

NOTE

Distributed Transactions and Multi-Document Transactions

Starting in MongoDB 4.2, the two terms are synonymous. Distributed transactions refer to multi-document transactions on sharded clusters and replica sets. Multi-document transactions (whether on sharded clusters or replica sets) are also known as distributed transactions starting in MongoDB 4.2.

For situations that require atomicity of reads and writes to multiple documents (in a single or multiple collections), MongoDB supports multi-document transactions:

- **In version 4.0**, MongoDB supports multi-document transactions on replica sets.

- **In version 4.2**, MongoDB introduces distributed transactions, which adds support for multi-document transactions on sharded clusters and incorporates the existing support for multi-document transactions on replica sets.

  To use transactions on MongoDB 4.2 deployments (replica sets and sharded clusters), clients **must** use MongoDB drivers updated for MongoDB 4.2.

Multi-document transactions are atomic (i.e. provide an "all-or-nothing" proposition):

- When a transaction commits, all data changes made in the transaction are saved and visible outside the transaction. That is, a transaction will not commit some of its changes while rolling back others.

  Until a transaction commits, the data changes made in the transaction are not visible outside the transaction.

  However, when a transaction writes to multiple shards, not all outside read operations need to wait for the result of the committed transaction to be visible across the shards. For example, if a transaction is committed and write 1 is visible on shard A but write 2 is not yet visible on shard B, an outside read at read concern [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) can read the results of write 1 without seeing write 2.

- When a transaction aborts, all data changes made in the transaction are discarded without ever becoming visible. For example, if any operation in the transaction fails, the transaction aborts and all data changes made in the transaction are discarded without ever becoming visible.

IMPORTANT

In most cases, multi-document transaction incurs a greater performance cost over single document writes, and the availability of multi-document transactions should not be a replacement for effective schema design. For many scenarios, the [denormalized data model (embedded documents and arrays)](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) will continue to be optimal for your data and use cases. That is, for many scenarios, modeling your data appropriately will minimize the need for multi-document transactions.

For additional transactions usage considerations (such as runtime limit and oplog size limit), see also [Production Considerations](https://docs.mongodb.com/manual/core/transactions-production-consideration/).

TIP

See also:

[Outside Reads During Commit](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-prod-consideration-outside-reads)

说明

分布式事务和多文档事务

从 MongoDB 4.2 开始，这两个术语是同义词。 分布式事务是指分片集群和副本集上的多文档事务。 从 MongoDB 4.2 开始，多文档事务（无论是在分片集群上还是副本集上）也称为分布式事务。

在多文档（在单个或多个集合中）读写上有原子性要求的情况，MongoDB 支持多文档事务：

- **在 4.0 版本**中，MongoDB 支持副本集上的多文档事务。

- **在 4.2 版本**中，MongoDB 引入了分布式事务，增加了对分片集群上多文档事务的支持，并合并了对副本集上多文档事务的现有支持。

  为了在 MongoDB 4.2 部署（副本集和分片集群）上使用事务，客户端**必须**使用为 MongoDB 4.2 更新的 MongoDB 驱动程序。

Multi-document transactions are atomic (i.e. provide an "all-or-nothing" proposition):

多文档事务是原子的（即提供“全有或全无”的主张）：

- 当事务提交时，事务中所做的所有数据更改都将保存并在事务外部可见。 也就是说，事务不会在回滚其他更改时提交其某些更改。

  在事务提交之前，事务中所做的数据更改在事务之外是不可见的。

  然而，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在分片中可见。 例如，如果事务已提交并且写入 1 在分片 A 上可见，但写入 2 在分片 B 上尚不可见，则外部读关注为 [`"local"`](https://docs.mongodb.com /manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 的读操作可以读取 write 1的结果，看不到 write 2。

- 当事务中止时，事务中所做的所有数据更改都将被丢弃，而不会变得可见。 例如，如果事务中的任何操作失败，事务就会中止，并且事务中所做的所有数据更改都将被丢弃，而不会变得可见。

重要

在大多数情况下，多文档事务比单文档写入会产生更大的性能成本，并且多文档事务的可用性不应替代有效的模型设计。 对于许多场景，[非规范化数据模型（嵌入文档和数组）](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) 将继续成为最适合你的数据和用例。 也就是说，对于许多场景，适当地对数据建模将最大限度地减少对多文档事务的需求。

有关其他事务使用注意事项（例如 runtime 限制和 oplog 大小限制），另请参阅 [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production- Consideration/)。

TIP

也就可以参考:

[ Commit 期间的外部读操作](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-prod-consideration-outside-reads)

## Transactions and Operations

## 事务和操作

Distributed transactions can be used across multiple operations, collections, databases, documents, and, starting in MongoDB 4.2, shards.

For transactions:

- You can specify read/write (CRUD) operations on **existing** collections. For a list of CRUD operations, see [CRUD Operations](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud).

- When using [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` or greater, you can create collections and indexes in transactions. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)

- The collections used in a transaction can be in different databases.

  NOTE

  You cannot create new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.

- You cannot write to [capped](https://docs.mongodb.com/manual/core/capped-collections/) collections. (Starting in MongoDB 4.2)

- You cannot read/write to collections in the `config`, `admin`, or `local` databases.

- You cannot write to `system.*` collections.

- You cannot return the supported operation's query plan (i.e. `explain`).

- For cursors created outside of a transaction, you cannot call [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore) inside the transaction.
- For cursors created in a transaction, you cannot call [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore) outside the transaction.

- Starting in MongoDB 4.2, you cannot specify [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors) as the first operation in a [transaction](https://docs.mongodb.com/manual/core/transactions/).

For a list of operations not supported in transactions, see [Restricted Operations](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted).

TIP

When creating or dropping a collection immediately before starting a transaction, if the collection is accessed within the transaction, issue the create or drop operation with write concern [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) to ensure that the transaction can acquire the required locks.

TIP

See also:

[Transactions and Operations Reference](https://docs.mongodb.com/manual/core/transactions-operations/)

分布式事务可用于跨多个操作、集合、数据库、文档以及从 MongoDB 4.2 开始的分片。

For transactions:

对于事务：

- 你可以在 **现有** 集合上指定读/写 (CRUD) 操作。 有关 CRUD 操作的列表，请参阅 [CRUD 操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud)。

- 当使用[功能兼容版本(fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)`"4.4"`或更高版本时，你可以在事务中创建集合和索引。 详情请参考[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

- 事务中使用的集合可以位于不同的数据库中。

  提示

  你不能在跨分片的写事务中创建新集合。 例如，如果你想对一个分片中已存在的集合进行写入且在另外一个不同的分片中隐式地创建集合，那么 MongoDB 无法在同一事务中执行这两种操作。

- 你不能写入 [capped](https://docs.mongodb.com/manual/core/ipped-collections/) 集合。 （从 MongoDB 4.2 开始）

- 你不能读/写 `config`、`admin` 或 `local` 数据库中的集合。

- 你不能写 `system.*` 集合。

- 你不能返回这类支持操作的查询计划（即`explain`）。

- 对于在事务外部创建的游标，你不能在事务内部调用 [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。

- 对于在事务内创建的游标，你不能在事务外调用 [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。

- Starting in MongoDB 4.2, you cannot specify [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors) as the first operation in a [transaction](https://docs.mongodb.com/manual/core/transactions/).

- 从 MongoDB 4.2 开始，你不能将 [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors) 定义为[事务](https://docs.mongodb.com/manual/core/transactions/)中的第一个操作 。

有关事务中不支持的操作列表，请参阅 [受限操作](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)。

TIP

在开始事务之前立即创建或删除集合时，如果在事务内访问该集合，注意使用写关注 [`"majority"`](https://docs.mongodb.com/manual/ reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) 来执行这些创建或删除操作，从而确保事务可以获取到所需要的锁。

TIP

也可以参考：

[事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



### Create Collections and Indexes In a Transaction

### 在事务中创建集合和索引

Starting in MongoDB 4.4 with [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"`, you can create collections and indexes inside a [multi-document transaction](https://docs.mongodb.com/manual/core/transactions/) unless the transaction is a cross-shard write transaction. With `"4.2"` or less, operations that affect the database catalog, such as creating or dropping a collection or an index, are **disallowed** in transactions.

When creating a collection inside a transaction:

- You can [implicitly create a collection](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit), such as with:
  - an [insert operation](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) against a non-existing collection, or
  - an [update/findAndModify operation](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) with `upsert: true` against a non-existing collection.
- You can [explicitly create a collection](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) using the [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) command or its helper [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection).

When [creating an index inside a transaction](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) [[1\]](https://docs.mongodb.com/manual/core/transactions/#footnote-create-existing-index), the index to create must be on either:

- a non-existing collection. The collection is created as part of the operation.
- a new empty collection created earlier in the same transaction.

| [[1](https://docs.mongodb.com/manual/core/transactions/#ref-create-existing-index-id2)] | You can also run [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) and [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) on existing indexes to check for existence. These operations return successfully without creating the index. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

从 MongoDB 4.4 开始，使用[功能兼容性版本 (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"`，你可以在 [多文档事务](https://docs.mongodb.com/manual/core/transactions/) 中创建集合和索引，除非事务是跨分片写入事务。 使用 `"4.2"` 或更低版本，事务中**不允许**使用影响数据库目录的操作，例如创建或删除集合和索引。

当在事务内部上传一个集合时：

- 您可以[隐式创建一个集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)，例如：
  - 针对一个不存在的集合上执行 [插入操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit)，或者
  - 针对一个不存在集合上执行带有 `upsert: true` 选项的 [update/findAndModify 操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) 。
- 你可以使用 [`create`](https: //docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) 命令或其帮助命令 [`db.createCollection()`](https://docs.mongodb.com/manual /reference/method/db.createCollection/#mongodb-method-db.createCollection) [显式地创建一个集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) 。

[在事务中创建索引](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) [[1\]](https: //docs.mongodb.com/manual/core/transactions/#footnote-create-existing-index)，要创建的索引需满足下面两者之一情况：

- 一个不存在的集合。 集合的创建是作为操作的一部分。
- 先前在同一事务中创建的新空集合。

| [[1](https://docs.mongodb.com/manual/core/transactions/#ref-create-existing-index-id2)] | 你还可以对现有索引运行 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) 和 [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) 来检查是否存在。 这些操作会成功地返回且不会创建索引。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

#### Restrictions

#### 限制

- You cannot create new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.

- For explicit creation of a collection or an index inside a transaction, the transaction read concern level must be [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-). Explicit creation is through:

  | Command                                                      | Method                                                       |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) |
  | [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) |

- The parameter [`shouldMultiDocTxnCreateCollectionAndIndexes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes) must be `true` (the default). When setting the parameter for a sharded cluster, set the parameter on all shards.

TIP

See also:

[Restricted Operations](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)

- 你不能在跨分片的写事务中创建新集合。 例如，如果你想对一个分片中已存在的集合进行写入且在另外一个不同的分片中隐式地创建集合，那么 MongoDB 无法在同一事务中执行这两种操作。

- 要在事务内显式创建集合或索引，事务读关注级别必须为 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb -阅读关注-阅读关注。-本地-)。 显式创建是通过：

  | 命令                                                         | Method                                                       |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) |
  | [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) |

- 参数 [`shouldMultiDocTxnCreateCollectionAndIndexes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes) 必须为 `true`（默认值）。 在对分片集群设置参数时，请在所有分片上设置该参数。

TIP

也可以参考:

[受限制的操作](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)



### Count Operation

To perform a count operation within a transaction, use the [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count) aggregation stage or the [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) (with a [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) expression) aggregation stage.

MongoDB drivers compatible with the 4.0 features provide a collection-level API `countDocuments(filter, options)` as a helper method that uses the [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) with a [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) expression to perform a count. The 4.0 drivers have deprecated the `count()` API.

Starting in MongoDB 4.0.3, the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell provides the [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments) helper method that uses the [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) with a [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) expression to perform a count.

### 计数操作

要在事务中执行计数操作，请使用 [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count) 聚合阶段 或 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)（带有 [`$sum`](https ://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) 表达式）聚合阶段。

与 4.0 特性兼容的 MongoDB 驱动程序提供了一个集合级别的 API `countDocuments(filter, options)` 作为使用 [`$group`](https://docs.mongodb.com/manual/reference/operator /aggregation/group/#mongodb-pipeline-pipe.-group) 带有 [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp .-sum) 表达式来执行计数。 4.0 驱动程序已弃用 `count()` API。

从 MongoDB 4.0.3 开始，[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell 提供了在 [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments) 中使用 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) 的带有 [`$sum`](https://docs.mongodb.com/ manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) 表达式来执行计数的帮助命令。



### Distinct Operation

To perform a distinct operation within a transaction:

- For unsharded collections, you can use the [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct) method/the [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct) command as well as the aggregation pipeline with the [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) stage.

- For sharded collections, you cannot use the [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct) method or the [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct) command.

  To find the distinct values for a sharded collection, use the aggregation pipeline with the [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) stage instead. For example:

  - Instead of `db.coll.distinct("x")`, use

    ```
    db.coll.aggregate([
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

    

  - Instead of `db.coll.distinct("x", { status: "A" })`, use:

    ```
    db.coll.aggregate([
       { $match: { status: "A" } },
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

    

  The pipeline returns a cursor to a document:

  ```
  { "distinctValues" : [ 2, 3, 1 ] }
  ```

  Iterate the cursor to access the results document.

### Distinct 操作

为了在事务中执行一个 distinct 操作：

- 对于未分片的集合，你可以使用 [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection .distinct) 方法/[`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct) 命令以及带有 [` $group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) 阶段的聚合管道。

- 对于分片的集合，你不能使用  [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)  方法或者  [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)  命令。

  要查找分片集合的不同值，请使用带 [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe. -group) 阶段的聚合管道来替代。 例如：

  - 替代 `db.coll.distinct("x")`，请使用：

    ```
    db.coll.aggregate([
       { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
       { $project: { _id: 0 } }
    ])
    ```

  - 替代  `db.coll.distinct("x", { status: "A" })`，请使用：

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



### Informational Operations

Informational commands, such as [`hello`](https://docs.mongodb.com/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#mongodb-dbcommand-dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#mongodb-dbcommand-dbcmd.connectionStatus) (and their helper methods) are allowed in transactions; however, they cannot be the first operation in the transaction.

### 信息操作

信息操作命令，比如  [`hello`](https://docs.mongodb.com/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#mongodb-dbcommand-dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#mongodb-dbcommand-dbcmd.connectionStatus) (以及它们的辅助方法）是被允许在使用中使用。然而，它们不能作为事务中的第一个操作



### Restricted Operations

*Changed in version 4.4*.

The following operations are not allowed in transactions:

- Operations that affect the database catalog, such as creating or dropping a collection or an index when using [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.2"` or lower. With fcv `"4.4"` or greater, you can create collections and indexes in transactions unless the transaction is a cross-shard write transaction. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes).
- Creating new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.
- [Explicit creation of collections](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit), e.g. [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) method, and indexes, e.g. [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) and [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) methods, when using a read concern level other than [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-).
- The [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections) and [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes) commands and their helper methods.
- Other non-CRUD and non-informational operations, such as [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count), etc. and their helpers.

TIP

See also:

- [Pending DDL Operations and Transactions](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
- [Transactions and Operations Reference](https://docs.mongodb.com/manual/core/transactions-operations/)

### 受限制的操作

*4.4 版本中的改变。*

下列这些操作在事务中是不被允许的：

- 影响数据库 catalog 的操作，例如在创建或删除集合和索引时使用`"4.2"` 或更低的[功能兼容版本（fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label -view-fcv) 。 使用 fcv `"4.4"` 或更高版本，你可以在事务中创建集合和索引，除非事务是跨分片写入事务。 有关详细信息，请参阅[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。
- 在跨分片写入事务中创建新的集合。例如，如果你在一个分片中对现有集合进行写入并在不同分片中隐式创建一个集合，则 MongoDB 无法在同一事务中执行这两种操作。
- [显式创建集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)，例如 [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) 方法和索引，例如 [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) 和 [`db. collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) 方法，当使用 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)以外的读取关注级别时。
- [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections) 和 [`listIndexes`](https://docs.mongodb.com /manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes) 命令及其辅助方法。
- 其他非 CRUD 和非信息操作，比如  [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count) 等以及它们的辅助命令。

TIP

也可以参考：

- [待处理的 DDL 操作和事务](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)
- [事务和操作参考](https://docs.mongodb.com/manual/core/transactions-operations/)



## Transactions and Sessions

- Transactions are associated with a session; i.e. you start a transaction for a session.
- At any given time, you can have at most one open transaction for a session.
- When using the drivers, each operation in the transaction must be associated with the session. Refer to your driver specific documentation for details.
- If a session ends and it has an open transaction, the transaction aborts.

## 事务和会话

- 事务与会话相关联； 即你为一个会话启动一个事务。
- 在任何给定时间，一个会话最多可以有一个打开的事务。
- 使用驱动程序时，事务中的每个操作都必须与会话相关联。 有关详细信息，请参阅你使用的驱动程序文档。
- 如果一个会话结束了并且它有一个打开的事务，则事务会中止。



## Read Concern/Write Concern/Read Preference

## 读关注/写关注/读偏好

### Transactions and Read Preference

Operations in a transaction use the transaction-level [read preference](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference).

Using the drivers, you can set the transaction-level [read preference](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference) at the transaction start:

- If the transaction-level read preference is unset, the transaction uses the session-level read preference.
- If transaction-level and the session-level read preference are unset, the transaction uses the client-level read preference. By default, the client-level read preference is [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary).

[Multi-document transactions](https://docs.mongodb.com/manual/core/transactions/) that contain read operations must use read preference [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary). All operations in a given transaction must route to the same member.

### 事务和读偏好

在事务中使用事务级[读偏好](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)的操作。

在使用驱动时，你可以在事务开始时设置事务级别的[读偏好](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference) ：

- 如果事务级别的读偏好没有设置，事务会使用会话级别的读偏好。
- 如果事务级别和会话级别的读偏好没有设置，事务使用客户端级别的读偏好。默认情况下，客户端级别的读偏好是 [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary)。

包含读操作的[多文档事务](https://docs.mongodb.com/manual/core/transactions/) 必须使用 [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary)读偏好。在一个给定事务中的所有操作都必须路由到同一个成员。



### Transactions and Read Concern

Operations in a transaction use the transaction-level [read concern](https://docs.mongodb.com/manual/reference/read-concern/). That is, any read concern set at the collection and database level is ignored inside the transaction.

You can set the transaction-level [read concern](https://docs.mongodb.com/manual/reference/read-concern/) at the transaction start.

- If the transaction-level read concern is unset, the transaction-level read concern defaults to the session-level read concern.
- If transaction-level and the session-level read concern are unset, the transaction-level read concern defaults to the client-level read concern. By default, client-level read concern is [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) for reads against the primary. See also:
  - [Transactions and Read Preference](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)
  - [Default MongoDB Read Concerns/Write Concerns](https://docs.mongodb.com/manual/reference/mongodb-defaults/)

Transactions support the following read concern levels:

#### `"local"`

- Read concern [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) returns the most recent data available from the node but can be rolled back.
- For transactions on sharded cluster, [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) read concern cannot guarantee that the data is from the same snapshot view across the shards. If snapshot isolation is required, use [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot) read concern.
- Starting in MongoDB 4.4, with [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` or greater, you can [create collections and indexes](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) inside a transaction. If [explicitly](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) creating a collection or an index, the transaction must use read concern [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-). [Implicit](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) creation of a collection can use any of the read concerns available for transactions.

#### `"majority"`

- Read concern [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) returns data that has been acknowledged by a majority of the replica set members (i.e. data cannot be rolled back) **if** the transaction commits with [write concern "majority"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern).
- If the transaction does not use [write concern "majority"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) for the commit, the [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) read concern provides **no** guarantees that read operations read majority-committed data.
- For transactions on sharded cluster, [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) read concern cannot guarantee that the data is from the same snapshot view across the shards. If snapshot isolation is required, use [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot) read concern.

#### `"snapshot"`

- Read concern [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) returns data from a snapshot of majority committed data **if** the transaction commits with [write concern "majority"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern).
- If the transaction does not use [write concern "majority"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) for the commit, the [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) read concern provides **no** guarantee that read operations used a snapshot of majority-committed data.
- For transactions on sharded clusters, the [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) view of the data **is** synchronized across shards.

### 事务和读关注

在事务中的操作会使用事务级[读关注](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)。也就是说，在事务内部忽略在集合和数据库级别设置的任何读关注。

你可以在事务开始时设置事务级别的[读关注](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference)。

- 如果事务级别的读关注没有设置，事务级的读关注默认为会话级的读关注
- 如果事务级和会话级的读关注没有设置，事务级的读关注默认为客户端级的读关注。默认情况下，客户端级的读取关注是 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 用于针对主节点的读取 . 也可以参考：
  - [事务和读偏好](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)
  - [默认 MongoDB 读关注/写关注](https://docs.mongodb.com/manual/reference/mongodb-defaults/)

事务支持下列的读关注级别：

#### `"local"`

- 读关注 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 会返回节点最新可用的数据，但可以被回滚。
- 对于分片集群上的事务，[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 读关注不能保证数据来自同一个跨分片的快照视图。 如果需要快照隔离，请使用 [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot) 读关注。
- 从 MongoDB 4.4 开始，使用[功能兼容版本（fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label -view-fcv)  `"4.4"` 或更高值，你可以在事务内[创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) 。如果[显式](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) 地创建集合或索引，事务必须使用读关注 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 。[隐式](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) 地创建集合可使用任何适用于事务的读关注。

#### `"majority"`

- **如果**事务以[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，则读关注 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 会返回已被副本集中大多数成员确认的数据（即数据不能回滚）。
- 如果事务不用[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 读关注**不能**保证读操作读取到大多数-已提交的数据。
- 对于分片集群上的事务， [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 读关注不能保证数据来自同一个跨分片的快照视图。 如果需要快照隔离，请使用 [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot) 读关注。

#### `"snapshot"`

- **如果**事务以[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，则读关注 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) 会从一个大多数已提交数据的快照中返回数据。

- 如果事务不用[写关注“majority”](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)的方式提交，[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) 读关注**不能**保证读操作读取到大多数-已提交的数据。

- 对于分片集群上的事务，数据的 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) 视图**是**跨分片同步的。

  

### Transactions and Write Concern

Transactions use the transaction-level [write concern](https://docs.mongodb.com/manual/reference/write-concern/) to commit the write operations. Write operations inside transactions must be issued without explicit write concern specification and use the default write concern. At commit time, the writes are then commited using the transaction-level write concern.

TIP

Do not explicitly set the write concern for the individual write operations inside a transaction. Setting write concerns for the individual write operations inside a transaction results in an error.

You can set the transaction-level [write concern](https://docs.mongodb.com/manual/reference/write-concern/) at the transaction start:

- If the transaction-level write concern is unset, the transaction-level write concern defaults to the session-level write concern for the commit.
- If the transaction-level write concern and the session-level write concern are unset, transaction-level write concern defaults to the client-level write concern. By default, client-level write concern is [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-). See also [Default MongoDB Read Concerns/Write Concerns](https://docs.mongodb.com/manual/reference/mongodb-defaults/).

Transactions support all write concern [w](https://docs.mongodb.com/manual/reference/write-concern/#std-label-wc-w) values, including:

#### `w: 1`

- Write concern [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) returns acknowledgement after the commit has been applied to the primary.

  IMPORTANT

  When you commit with [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-), your transaction can be [rolled back if there is a failover](https://docs.mongodb.com/manual/core/replica-set-rollbacks/).

- When you commit with [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) write concern, transaction-level [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) read concern provides **no** guarantees that read operations in the transaction read majority-committed data.

- When you commit with [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) write concern, transaction-level [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) read concern provides **no** guarantee that read operations in the transaction used a snapshot of majority-committed data.

#### `w: "majority"`

- Write concern [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) returns acknowledgement after the commit has been applied to a majority (M) of voting members; i.e. the commit has been applied to the primary and (M-1) voting secondaries.
- When you commit with [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) write concern, transaction-level [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) read concern guarantees that operations have read majority-committed data. For transactions on sharded clusters, this view of the majority-committed data is not synchronized across shards.
- When you commit with [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) write concern, transaction-level [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) read concern guarantees that operations have from a synchronized snapshot of majority-committed data.

NOTE

Regardless of the [write concern specified for the transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern), the commit operation for a sharded cluster transaction includes some parts that use `{w: "majority", j: true}` write concern.

### 事务和写关注

事务使用事务级[写关注](https://docs.mongodb.com/manual/reference/write-concern/) 来提交写操作。 事务内的写操作必须没有显式定义写关注，并使用默认的写关注。 在提交时，然后使用事务级写关注提交写入。

TIP

不要为事务内的单个写操作显式设置写关注。 为事务内的单个写操作设置写关注会导致错误。 

你可以在事务开始时设置事务级别的[写关注](https://docs.mongodb.com/manual/reference/write-concern/) ：

- 如果事务级别的写关注没有设置，事务级写关注默认为提交的会话级写关注。
- 如果事务级写关注和会话级写关注没有设置，事务级写关注默认为客户端级写关注。默认情况下，客户端写关注为 [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)。也可以参考[默认 MongoDB 读关注/写关注](https://docs.mongodb.com/manual/reference/mongodb-defaults/)。

事务支持所有写关注 [w](https://docs.mongodb.com/manual/reference/write-concern/#std-label-wc-w) 的值，包括：

#### `w: 1`

- 写关注  [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) 会在提交已经被应用到主节点后反馈确认结果。

  重要

  当你使用 [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)提交，你的事务在发生故障时可以回滚。

- 当你使用 [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) 写关注提交，事务级 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 的读关注**无法**保证事务中的读操作能读取大多数-已提交的数据。

- 当你使用 [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-) 写关注提交，事务级 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) 的读关注**无法**保证事务中的读操作能使用大多数-已提交数据的快照。

#### `w: "majority"`

- 写关注 [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) 会在提交的数据被应用到大多数 (M) 有投票权的成员后返回确认 ； 即提交数据已被应用到主节点和（M-1）有投票圈的从节点。
- 当你使用 [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) 写关注提交时，事务级 [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) 读关注可以确保操作能读取到大多数-已提交的数据。对于分片集群上的事务，这种大多数-已提交数据的视图在分片之间不会同步。
- 当你使用 [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) 写关注提交时，事务级 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) 读关注可以确保证操作能获取来自大多数-已提交数据的同步快照。

说明

不管[为事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)，分片集群事务的提交操作包括一部分使用了 `{w: "majority", j: true}` 写关注的操作。



## General Information

## 通用信息

### Production Considerations

### 生产注意事项

For various production considerations with using transactions, see [Production Considerations](https://docs.mongodb.com/manual/core/transactions-production-consideration/). In addition, or sharded clusters, see also [Production Considerations (Sharded Clusters)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/).

关于使用事务的各种生产注意事项，请参阅 [生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-sumption/)。 另外，或者是分片集群，也可以参考[生产注意事项（分片集群）](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)。



### Arbiters

### 仲裁节点

Transactions whose write operations span multiple shards will error and abort if any transaction operation reads from or writes to a shard that contains an arbiter.

See also [Disabled Read Concern Majority](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-disabled-rc-majority) for transaction restrictions on shards that have disabled read concern majority.

如果任何事务操作从包含仲裁节点的分片读取或写入，其写操作跨越多个分片的事务将出错并中止。

另请参阅 [Disabled Read Concern Majority](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-disabled-rc-majority)，了解在分片上已禁用读关注 majority 的事务限制。



### Disabled Read Concern Majority

### 禁止读关注 majority

A 3-member PSA (Primary-Secondary-Arbiter) replica set or a sharded cluster with 3-member PSA shards may have disabled read concern majority ([`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern) or [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern))

- On sharded clusters,

  If a transaction involves a shard that has [disabled read concern "majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority), you cannot use read concern [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) for the transaction. You can only use read concern [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) or [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) for the transaction. If you use read concern [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-), the transaction errors and aborts.`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false.`Transactions whose write operations span multiple shards will error and abort if any of the transaction's read or write operations involves a shard that has disabled read concern `"majority"`.

- On replica set,

  You can specify read concern [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) or [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) or [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) even in the replica set has [disabled read concern "majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority).However, if you are planning to transition to a sharded cluster with disabled read concern majority shards, you may wish to avoid using read concern `"snapshot"`.

TIP

To check if read concern "majority" is disabled, You can run [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus) on the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) instances and check the [`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads) field. If `false`, read concern "majority" is disabled.

For more information, see [3-Member Primary-Secondary-Arbiter Architecture](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-psa) and [Three Member Primary-Secondary-Arbiter Shards](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-psa).

三成员 PSA（主-从-仲裁）副本集或者拥有三成员 PSA 分片的分片集群也许已经禁用了读关注 majority ([`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern) 或者 [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern))

- 在分片集群上，

  如果事务涉及到具有[禁用读关注“majority”](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority)的分片，你不能对事务使用读关注 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。你只能对事务使用读关注 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 或者 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 。如果你使用读关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)，则事务会报错并中止。`readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false`。如果事务的任何读取或写入操作涉及已禁用读关注 `"majority"`的分片，则其写入操作会跨越多个分片的事务将出错并中止。

- 在副本集上，

  你可以定义读关注 [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) 、 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) 或者 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) ，甚至副本集已[禁用读关注 "majority"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority)。但是，如果你计划迁移到在分片上禁用读关注 majority 的分片集群上，你可能希望避免使用读关注 `"snapshot"`。

TIP

要检查读关注 "majority" 是否被禁用，你可以在 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 实例上运行 [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db .serverStatus) 并检查 [`storageEngine.supportsCommittedReads`]( https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads) 字段。 如果值为`false`，则表示读关注 "majority" 已禁用。

更多信息请参考 [3-Member Primary-Secondary-Arbiter Architecture](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-psa) 和 [Three Member Primary-Secondary-Arbiter Shards](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-psa)。



### Shard Configuration Restriction

### 分片配置限制

You cannot run transactions on a sharded cluster that has a shard with [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) set to `false` (such as a shard with a voting member that uses the [in-memory storage engine](https://docs.mongodb.com/manual/core/inmemory/)).

NOTE

Regardless of the [write concern specified for the transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern), the commit operation for a sharded cluster transaction includes some parts that use `{w: "majority", j: true}` write concern.

你不能在包含 [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) 设置为 `false` 的分片的分片集群上运行事务 （例如包含使用了 [内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)的投票成员的分片）。

说明

不管[为事务指定的写关注](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern)，分片集群事务的提交操作包括一部分使用了 `{w: "majority", j: true}` 写关注的操作。



### Diagnostics

### 诊断

MongoDB provides various transactions metrics:

| Source                                                       | Returns                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus) method[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-dbcommand-dbcmd.serverStatus) command | Returns [transactions](https://docs.mongodb.com/manual/reference/command/serverStatus/#std-label-server-status-transactions) metrics. |
| [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-pipeline-pipe.-currentOp) aggregation pipeline | Returns:[`$currentOp.transaction`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.transaction) if an operation is part of a transaction.Information on [inactive sessions](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#std-label-currentOp-stage-idleSessions) that are holding locks as part of a transaction.[`$currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator) metrics for sharded transactions that involes writes to multiple shards. |
| [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#mongodb-method-db.currentOp) method[`currentOp`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-dbcommand-dbcmd.currentOp) command | Returns:[`currentOp.transaction`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.transaction) if an operation is part of a transaction.[`currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.twoPhaseCommitCoordinator) metrics for sharded transactions that involes writes to multiple shards. |
| [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) and [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) log messages | Includes information on slow transactions (i.e. transactions that exceed the [`operationProfiling.slowOpThresholdMs`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs) threshhold) under the [`TXN`](https://docs.mongodb.com/manual/reference/log-messages/#mongodb-data-TXN) log component. |

MongoDB 提供了多种事务相关指标：

| Source                                                       | Returns                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus) 方法中的[`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-dbcommand-dbcmd.serverStatus) 命令 | 返回 [transactions](https://docs.mongodb.com/manual/reference/command/serverStatus/#std-label-server-status-transactions) 相关指标。 |
| [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-pipeline-pipe.-currentOp) 聚合管道 | 如果操作作为事务的一部分则返回：[`$currentOp.transaction`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.transaction) 。 持有锁的 [非活动会话](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#std-label-currentOp-stage-idleSessions) 的信息会作为事务的一部分。[` $currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator) 是写入多个分片的分片事务的指标。 |
| [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#mongodb-method-db.currentOp)方法中的[`currentOp`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-dbcommand-dbcmd.currentOp) 命令 | 如果操作作为事务的一部分则返回[`currentOp.transaction`](https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.transaction)。[` $currentOp.twoPhaseCommitCoordinator`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator) 是写入多个分片的分片事务的指标。 |
| [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) 和 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) 日志信息 | 在[`TXN`](https://docs.mongodb.com/manual/reference/log-messages/#mongodb-data-TXN)日志组件下包含慢事务的信息（即事务超过了[`operationProfiling.slowOpThresholdMs`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs) 阈值 ） |



### Feature Compatibility Version (FCV)

### 功能兼容版本（FCV）

To use transactions, the [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) for all members of the deployment must be at least:

| Deployment      | Minimum `featureCompatibilityVersion` |
| :-------------- | :------------------------------------ |
| Replica Set     | `4.0`                                 |
| Sharded Cluster | `4.2`                                 |

To check the fCV for a member, connect to the member and run the following command:

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

For more information, see the [`setFeatureCompatibilityVersion`](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion) reference page.

为了使用事务，部署架构中所有成员的 [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) 至少为：

| Deployment | Minimum `featureCompatibilityVersion` |
| :--------- | :------------------------------------ |
| 副本集     | `4.0`                                 |
| 分片集群   | `4.2`                                 |

为了检查成员的 FCV，连接到成员并运行下面的命令：

```
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

更多信息详见 [`setFeatureCompatibilityVersion`](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion) 参考页。



### Storage Engines

### 存储引擎

Starting in MongoDB 4.2, [multi-document transactions](https://docs.mongodb.com/manual/core/transactions/) are supported on replica sets and sharded clusters where:

- the primary uses the WiredTiger storage engine, and
- the secondary members use either the WiredTiger storage engine or the [in-memory](https://docs.mongodb.com/manual/core/inmemory/) storage engines.

In MongoDB 4.0, only replica sets using the WiredTiger storage engine supported transactions.

NOTE

You cannot run transactions on a sharded cluster that has a shard with [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) set to `false`, such as a shard with a voting member that uses the [in-memory storage engine](https://docs.mongodb.com/manual/core/inmemory/).

从 MongoDB 4.2 开始，[多文档事务](https://docs.mongodb.com/manual/core/transactions/) 支持副本集和分片集群，其中：

- 主节点使用 WiredTiger 存储引擎，同时
- 从节点使用 WiredTiger 存储引擎或 [in-memory](https://docs.mongodb.com/manual/core/inmemory/) 存储引擎。

在 MongoDB 4.0，只有使用 WiredTiger 存储引擎的副本集支持事务。

说明

你不能在包含 [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault) 设置为 `false` 的分片的分片集群上运行事务，例如包含使用了 [内存存储引擎](https://docs.mongodb.com/manual/core/inmemory/)的投票成员的分片。



## Additional Transactions Topics

- [Drivers API](https://docs.mongodb.com/manual/core/transactions-in-applications/)
- [Production Considerations](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
- [Production Considerations (Sharded Clusters)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)
- [Transactions and Operations](https://docs.mongodb.com/manual/core/transactions-operations/)
- To learn more about when to use transactions and if they support your use case, see the [Are Transactions Right For You?](https://www.mongodb.com/presentations/are-transactions-right-for-you-) presentation from **MongoDB.live 2020**.

## 附加事务主题

- [驱动 API](https://docs.mongodb.com/manual/core/transactions-in-applications/)
- [生产考虑因素](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
- [生产考虑因素 (分片集群)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)
- [事务和操作](https://docs.mongodb.com/manual/core/transactions-operations/)
- 要了解有关何时使用事务以及它们是否支持你的用例的更多信息，请参阅 [Are Transactions Right for You?](https://www.mongodb.com/presentations/are-transactions-right-for-you-) 来自 **MongoDB.live 2020** 的演示。



原文链接：https://docs.mongodb.com/manual/core/transactions/

译者：李正洋
