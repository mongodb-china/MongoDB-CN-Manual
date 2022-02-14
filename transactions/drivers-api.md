
# 驱动程序 API

## 回调 API 和 核心 API

 [回调 API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-callback-api)：

- 启动一个事务，执行指定的操作，并提交（或出错时中止）。
- 自动包含 [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) 和 [` "UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result) 的错误处理逻辑。

[核心 API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-core-api)：

- 需要显式调用来启动事务并提交事务。
- 不包含 [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) 和 [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result) 的错误处理逻辑，而是为这些错误提供了包含自定义错误处理的灵活性。




## 回调 API


回调 API 包含以下逻辑：

- 如果事务遇到 [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)，则作为一个整体重试事务。
- 如果提交遇到 [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)，则重新这个提交操作。


### 示例

------

➤ 使用右上角的**选择您的语言**下拉菜单来设置此页面上示例的语言。

------

该示例使用新的回调 API 来处理事务，它启动事务、执行指定的操作并提交（或在出错时中止）。新的回调 API 包含 [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) 或 [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result) 提交错误的重试逻辑。

重要

- *推荐*.。使用针对 MongoDB 部署版本更新的 MongoDB 驱动程序。 对于 MongoDB 4.2 部署（副本集和分片集群）上的事务，客户端**必须**使用为 MongoDB 4.2 更新的 MongoDB 驱动程序。
- 使用驱动程序时，事务中的每个操作**必须**与会话相关联（即将会话传递给每个操作）。
- 事务中的操作使用 [事务级别的读关注](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-concern)，[事务级别的写关注](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-write-concern)，和 [事务级别的读偏好](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-preference)。
- 在 MongoDB 4.2 及更早版本中，你无法在事务中创建集合。 如果在事务内部运行，导致文档插入的写操作（例如 `insert` 或带有 `upsert: true` 的更新操作）必须在 **已有的** 集合上执行。
- 从 MongoDB 4.4 开始，你可以隐式或显式地在事务中创建集合。 但是，你比须使用针对 4.4 更新的 MongoDB 驱动程序。 有关详细信息，请参阅 [在事务中创建集合和索引](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-create-collections-indexes)。

```
// WithTransactionExample is an example of using the Session.WithTransaction function.
func WithTransactionExample() {
	ctx := context.Background()
	// For a replica set, include the replica set name and a seedlist of the members in the URI string; e.g.
	// uri := "mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017/?replicaSet=myRepl"
	// For a sharded cluster, connect to the mongos instances; e.g.
	// uri := "mongodb://mongos0.example.com:27017,mongos1.example.com:27017/"
	var uri string

	clientOpts := options.Client().ApplyURI(uri)
	client, err := mongo.Connect(ctx, clientOpts)
	if err != nil {
		panic(err)
	}
	defer func() { _ = client.Disconnect(ctx) }()

	// Prereq: Create collections.
	wcMajority := writeconcern.New(writeconcern.WMajority(), writeconcern.WTimeout(1*time.Second))
	wcMajorityCollectionOpts := options.Collection().SetWriteConcern(wcMajority)
	fooColl := client.Database("mydb1").Collection("foo", wcMajorityCollectionOpts)
	barColl := client.Database("mydb1").Collection("bar", wcMajorityCollectionOpts)

	// Step 1: Define the callback that specifies the sequence of operations to perform inside the transaction.
	callback := func(sessCtx mongo.SessionContext) (interface{}, error) {
		// Important: You must pass sessCtx as the Context parameter to the operations for them to be executed in the
		// transaction.
		if _, err := fooColl.InsertOne(sessCtx, bson.D{{"abc", 1}}); err != nil {
			return nil, err
		}
		if _, err := barColl.InsertOne(sessCtx, bson.D{{"xyz", 999}}); err != nil {
			return nil, err
		}

		return nil, nil
	}

	// Step 2: Start a session and run the callback using WithTransaction.
	session, err := client.StartSession()
	if err != nil {
		panic(err)
	}
	defer session.EndSession(ctx)

	result, err := session.WithTransaction(ctx, callback)
	if err != nil {
		panic(err)
	}
	fmt.Printf("result: %v\n", result)
}
```





## 核心 API


核心事务 API 不包含标记错误的重试逻辑：

- [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)。如果事务中的操作返回标记为 [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error )的错误，则事务会被作为一个整体进行重试。

- [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)。如果提交返回标记为 [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)的错误，提交会被重试。

  为了处理 [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)，应用程序应该明确地包含错误的重试逻辑。


### 示例


### 

------

➤ 使用右上角的**选择您的语言**下拉菜单来设置此页面上示例的语言。

------

以下的示例包含了针对暂时性错误重试事务和针对未知提交错误重试提交的逻辑：

```
	runTransactionWithRetry := func(sctx mongo.SessionContext, txnFn func(mongo.SessionContext) error) error {
		for {
			err := txnFn(sctx) // Performs transaction.
			if err == nil {
				return nil
			}

			log.Println("Transaction aborted. Caught exception during transaction.")

			// If transient error, retry the whole transaction
			if cmdErr, ok := err.(mongo.CommandError); ok && cmdErr.HasErrorLabel("TransientTransactionError") {
				log.Println("TransientTransactionError, retrying transaction...")
				continue
			}
			return err
		}
	}

	commitWithRetry := func(sctx mongo.SessionContext) error {
		for {
			err := sctx.CommitTransaction(sctx)
			switch e := err.(type) {
			case nil:
				log.Println("Transaction committed.")
				return nil
			case mongo.CommandError:
				// Can retry commit
				if e.HasErrorLabel("UnknownTransactionCommitResult") {
					log.Println("UnknownTransactionCommitResult, retrying commit operation...")
					continue
				}
				log.Println("Error during commit...")
				return e
			default:
				log.Println("Error during commit...")
				return e
			}
		}
	}

	// Updates two collections in a transaction.
	updateEmployeeInfo := func(sctx mongo.SessionContext) error {
		employees := client.Database("hr").Collection("employees")
		events := client.Database("reporting").Collection("events")

		err := sctx.StartTransaction(options.Transaction().
			SetReadConcern(readconcern.Snapshot()).
			SetWriteConcern(writeconcern.New(writeconcern.WMajority())),
		)
		if err != nil {
			return err
		}

		_, err = employees.UpdateOne(sctx, bson.D{{"employee", 3}}, bson.D{{"$set", bson.D{{"status", "Inactive"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Println("caught exception during transaction, aborting.")
			return err
		}
		_, err = events.InsertOne(sctx, bson.D{{"employee", 3}, {"status", bson.D{{"new", "Inactive"}, {"old", "Active"}}}})
		if err != nil {
			sctx.AbortTransaction(sctx)
			log.Println("caught exception during transaction, aborting.")
			return err
		}

		return commitWithRetry(sctx)
	}

	return client.UseSessionWithOptions(
		ctx, options.Session().SetDefaultReadPreference(readpref.Primary()),
		func(sctx mongo.SessionContext) error {
			return runTransactionWithRetry(sctx, updateEmployeeInfo)
		},
	)
}
```



## 驱动程序版本

*对于 MongoDB 4.2 部署（副本集和分片集群）上的事务*，客户端**必须**使用为 MongoDB 4.2 更新的 MongoDB 驱动程序：

| [C 1.15.0](http://mongoc.org/libmongoc/)[C# 2.9.0](https://mongodb.github.io/mongo-csharp-driver/)[Go 1.1](https://godoc.org/go.mongodb.org/mongo-driver/mongo) | [Java 3.11.0](https://mongodb.github.io/mongo-java-driver/)[Node 3.3.0](https://mongodb.github.io/node-mongodb-native/)[Perl 2.2.0](https://metacpan.org/author/MONGODB) | [Python 3.9.0](https://api.mongodb.com/pymongo)[Ruby 2.10.0](https://docs.mongodb.com/ruby-driver/current/)[Scala 2.7.0](https://mongodb.github.io/mongo-scala-driver/) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |                                                              |

*对于 MongoDB 4.0 副本集上的事务*，客户端需要为 MongoDB 4.0 或更高版本更新 MongoDB 驱动程序。

| Java 3.8.0Python 3.7.0C 1.11.0 | C# 2.7Node 3.1.0Ruby 2.6.0 | Perl 2.0.0PHP (PHPC) 1.5.0Scala 2.4.0 |
| ------------------------------ | -------------------------- | ------------------------------------- |
|                                |                            |                                       |



## 事务错误处理

无论是哪种数据库系统，无论是MongoDB还是关系型数据库，应用程序都应该采取措施处理事务提交过程中的错误，并包含事务的重试逻辑。

### `"TransientTransactionError"`


无论 [`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites)的值是多少，事务内部的*单个*写操作都不可重试。如果操作遇到一个错误[与标签相关](https://github.com/mongodb/specifications/blob/master/source/transactions/transactions.rst#error-labels) `"TransientTransactionError"`，比如当主节点降级，事务会作为一个整体被重试。

- 回调 API 包含了 `"TransientTransactionError"` 的重试逻辑。
- 核心事务 API 不包含 `"TransientTransactionError"` 的重试逻辑。为了处理 `"TransientTransactionError"`，应用程序应该明确地包含错误的重试逻辑。

### `"UnknownTransactionCommitResult"`


提交操作是[可重试的写操作](https://docs.mongodb.com/v4.4/core/retryable-writes/)。如果提交操作遇到错误，无论 [`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites)的值是多少，MongoDB 驱动程序都会重试提交。

如果提交操作遇到标记为 `"UnknownTransactionCommitResult"`的错误，提交可以被重试。

- 回调 API 包含了 `"UnknownTransactionCommitResult"`的重试逻辑。
- 核心事务 API 不包含 `"UnknownTransactionCommitResult"`的重试逻辑。为了处理 `"UnknownTransactionCommitResult"`，应用程序应该明确地包含错误的重试逻辑。



### 驱动程序版本错误


在具有多个 [`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos) 实例的分片集群上，使用为 MongoDB 4.0 更新的驱动程序执行事务 （而不是 MongoDB 4.2）将失败并可能导致错误，包括：

注释

你的驱动程序可能会返回不同的错误。 有关详细信息，请参阅驱动程序的文档。

| Error Code | Error Message                                           |
| :--------- | :------------------------------------------------------ |
| 251        | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940      | `cannot commit with no participants`                    |


*对于 MongoDB 4.2 部署（副本集和分片集群）上的事务*，使用为 MongoDB 4.2 更新的 MongoDB 驱动程序


## 附加信息


### `mongo` Shell 示例


下面列出的 [`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell 方法可用于事务：

- [`Session.startTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.startTransaction/#mongodb-method-Session.startTransaction)
- [`Session.commitTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.commitTransaction/#mongodb-method-Session.commitTransaction)
- [`Session.abortTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.abortTransaction/#mongodb-method-Session.abortTransaction)

注释

[`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell 示例为了简单起见省略了重试逻辑和强大的错误处理。 有关在应用程序中包含事务的更实际示例，请参阅 [事务错误处理](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transactions-retry) .

```
// Create collections:
db.getSiblingDB("mydb1").foo.insert( {abc: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );
db.getSiblingDB("mydb2").bar.insert( {xyz: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );

// Start a session.
session = db.getMongo().startSession( { readPreference: { mode: "primary" } } );

coll1 = session.getDatabase("mydb1").foo;
coll2 = session.getDatabase("mydb2").bar;

// Start a transaction
session.startTransaction( { readConcern: { level: "local" }, writeConcern: { w: "majority" } } );

// Operations inside the transaction
try {
   coll1.insertOne( { abc: 1 } );
   coll2.insertOne( { xyz: 999 } );
} catch (error) {
   // Abort transaction on error
   session.abortTransaction();
   throw error;
}

// Commit the transaction using write concern set at transaction start
session.commitTransaction();

session.endSession();
```



原文链接：https://docs.mongodb.com/manual/core/transactions-in-applications/

译者：李正洋
