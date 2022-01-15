# Drivers API

# 驱动程序API

## Callback API vs Core API

## 回调API和核心API

The [Callback API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-callback-api):

- Starts a transaction, executes the specified operations, and commits (or aborts on error).
- Automatically incorporates error handling logic for [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) and [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result).

The [Core API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-core-api):

- Requires explicit call to start the transaction and commit the transaction.
- Does not incorporate error handling logic for [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) and [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), and instead provides the flexibility to incorporate custom error handling for these errors.

 [回调API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-callback-api)：

- 启动一个事务，执行指定的操作，并提交（或出错时中止）。
- 自动包含[`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)和[` "UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)的错误处理逻辑。

[核心API](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-txn-core-api)：

- 需要显式调用来启动事务并提交事务。
- 不包含[`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)和[`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)的错误处理逻辑，而是为这些错误提供了包含自定义错误处理的灵活性。



## Callback API

## 回调API

The callback API incorporates logic:

- To retry the transaction as a whole if the transaction encounters a [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error).
- To retry the commit operation if the commit encounters an [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result).

回调API包含以下逻辑：

- 如果事务遇到[`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)，则作为一个整体重试事务。
- 如果提交遇到[`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)，则重新这个提交操作。

### Example

### 示例

------

➤ Use the **Select your language** drop-down menu in the upper-right to set the language of the examples on this page.

------

The example uses the new callback API for working with transactions, which starts a transaction, executes the specified operations, and commits (or aborts on error). The new callback API incorporates retry logic for [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error) or [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result) commit errors.

IMPORTANT

- *Recommended*. Use the MongoDB driver updated for the version of your MongoDB deployment. For transactions on MongoDB 4.2 deployments (replica sets and sharded clusters), clients **must** use MongoDB drivers updated for MongoDB 4.2.
- When using the drivers, each operation in the transaction **must** be associated with the session (i.e. pass in the session to each operation).
- Operations in a transaction use [transaction-level read concern](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-concern), [transaction-level write concern](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-write-concern), and [transaction-level read preference](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-preference).
- In MongoDB 4.2 and earlier, you cannot create collections in transactions. Write operations that result in document inserts (e.g. `insert` or update operations with `upsert: true`) must be on **existing** collections if run inside transactions.
- Starting in MongoDB 4.4, you can create collections in transactions implicitly or explicitly. You must use MongoDB drivers updated for 4.4, however. See [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-create-collections-indexes) for details.

### 

------

➤ 使用右上角的**选择您的语言**下拉菜单来设置此页面上示例的语言。

------

该示例使用新的回调API来处理事务，它启动事务、执行指定的操作并提交（或在出错时中止）。新的回调API包含[`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)或[`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)提交错误的重试逻辑。

重要

- *推荐*.。使用针对MongoDB部署版本更新的MongoDB驱动程序。对于MongoDB 4.2部署（副本集和分片集群）上的事务，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序。
- 使用驱动程序时，事务中的每个操作**必须**与会话相关联（即将会话传递给每个操作）。
- 事务中的操作使用[事务级别的读关注](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-concern)，[事务级别的写关注](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-write-concern)，和[事务级别的读偏好](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-read-preference)。
- 在MongoDB 4.2及更早版本中，你无法在事务中创建集合。如果在事务内部运行，导致文档插入的写操作（例如`insert`或带有`upsert: true`的更新操作）必须在**已有的**集合上执行。
- 从MongoDB 4.4开始，你可以隐式或显式地在事务中创建集合。但是，你比须使用针对4.4更新的MongoDB驱动程序。有关详细信息，请参阅[在事务中创建集合和索引](https://docs.mongodb.com/v4.4/core/transactions/#std-label-transactions-create-collections-indexes)。

```
// WithTransactionExample是一个使用Session.WithTransaction函数的例子。
func WithTransactionExample() {
	ctx := context.Background()
	// 对于副本集，在URI字符串中包含副本集名称和成员的种子列表；例如
	// uri := "mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017/?replicaSet=myRepl"
	// 对于分片集群，连接到mongos实例；例如
	// uri := "mongodb://mongos0.example.com:27017,mongos1.example.com:27017/"
	var uri string

	clientOpts := options.Client().ApplyURI(uri)
	client, err := mongo.Connect(ctx, clientOpts)
	if err != nil {
		panic(err)
	}
	defer func() { _ = client.Disconnect(ctx) }()

	// Prereq: 创建集合。
	wcMajority := writeconcern.New(writeconcern.WMajority(), writeconcern.WTimeout(1*time.Second))
	wcMajorityCollectionOpts := options.Collection().SetWriteConcern(wcMajority)
	fooColl := client.Database("mydb1").Collection("foo", wcMajorityCollectionOpts)
	barColl := client.Database("mydb1").Collection("bar", wcMajorityCollectionOpts)

	// 第一步：定义回调函数，该回调函数指定在事务内部执行的操作序列。
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

	// 第二步：启动一个会话并使用WithTransaction运行回调。
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





## Core API

## 核心API

The core transaction API does not incorporate retry logic for errors labeled:

- [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error). If an operation in a transaction returns an error labeled [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error), the transaction as a whole can be retried.

  To handle [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error), applications should explicitly incorporate retry logic for the error.

- [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result). If the commit returns an error labeled [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), the commit can be retried.

  To handle [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), applications should explicitly incorporate retry logic for the error.

核心事务API不包含标记错误的重试逻辑：

- [`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error)。如果事务中的操作返回标记为[`"TransientTransactionError"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transient-transaction-error )的错误，则事务会被作为一个整体进行重试。

- [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)。如果提交返回标记为[`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)的错误，提交会被重试。

  为了处理[`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)，应用程序应该明确地包含错误的重试逻辑。

### Example

### 示例

------

➤ Use the **Select your language** drop-down menu in the upper-right to set the language of the examples on this page.

------

The following example incorporates logic to retry the transaction for transient errors and retry the commit for unknown commit error:

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

			// 如果出现暂时性错误，重试整个事务
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
				// 可以重试提交
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

	// 更新事务中的两个集合。
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



## Driver Versions

## 驱动程序版本

*For transactions on MongoDB 4.2 deployments (replica sets and sharded clusters)*, clients **must** use MongoDB drivers updated for MongoDB 4.2:

*对于MongoDB 4.2部署（副本集和分片集群）上的事务*，客户端**必须**使用为MongoDB 4.2更新的MongoDB驱动程序：

| [C 1.15.0](http://mongoc.org/libmongoc/)[C# 2.9.0](https://mongodb.github.io/mongo-csharp-driver/)[Go 1.1](https://godoc.org/go.mongodb.org/mongo-driver/mongo) | [Java 3.11.0](https://mongodb.github.io/mongo-java-driver/)[Node 3.3.0](https://mongodb.github.io/node-mongodb-native/)[Perl 2.2.0](https://metacpan.org/author/MONGODB) | [Python 3.9.0](https://api.mongodb.com/pymongo)[Ruby 2.10.0](https://docs.mongodb.com/ruby-driver/current/)[Scala 2.7.0](https://mongodb.github.io/mongo-scala-driver/) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |                                                              |

*For transactions on MongoDB 4.0 replica sets*, clients require MongoDB drivers updated for MongoDB 4.0 or later.

*对于MongoDB 4.0副本集上的事务*，客户端需要为MongoDB 4.0或更高版本更新MongoDB驱动程序。

| Java 3.8.0Python 3.7.0C 1.11.0 | C# 2.7Node 3.1.0Ruby 2.6.0 | Perl 2.0.0PHP (PHPC) 1.5.0Scala 2.4.0 |
| ------------------------------ | -------------------------- | ------------------------------------- |
|                                |                            |                                       |



## Transaction Error Handling

## 事务错误处理

Regardless of the database system, whether MongoDB or relational databases, applications should take measures to handle errors during transaction commits and incorporate retry logic for transactions.

无论是哪种数据库系统，无论是MongoDB还是关系型数据库，应用程序都应该采取措施处理事务提交过程中的错误，并包含事务的重试逻辑。

### `"TransientTransactionError"`

The *individual* write operations inside the transaction are not retryable, regardless of the value of [`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites). If an operation encounters an error [associated with the label](https://github.com/mongodb/specifications/blob/master/source/transactions/transactions.rst#error-labels) `"TransientTransactionError"`, such as when the primary steps down, the transaction as a whole can be retried.

- The callback API incorporates retry logic for `"TransientTransactionError"`.
- The core transaction API does not incorporate retry logic for `"TransientTransactionError"`. To handle `"TransientTransactionError"`, applications should explicitly incorporate retry logic for the error.

无论[`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites)的值是多少，事务内部的*单个*写操作都不可重试。如果操作遇到一个错误[与标签相关](https://github.com/mongodb/specifications/blob/master/source/transactions/transactions.rst#error-labels)`"TransientTransactionError"`，比如当主节点降级，事务会作为一个整体被重试。

- 回调API包含了`"TransientTransactionError"`的重试逻辑。
- 核心事务API不包含`"TransientTransactionError"`的重试逻辑。为了处理`"TransientTransactionError"`，应用程序应该明确地包含错误的重试逻辑。

### `"UnknownTransactionCommitResult"`

The commit operations are [retryable write operations](https://docs.mongodb.com/v4.4/core/retryable-writes/). If the commit operation encounters an error, MongoDB drivers retry the commit regardless of the value of [`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites).

If the commit operation encounters an error labeled `"UnknownTransactionCommitResult"`, the commit can be retried.

- The callback API incorporates retry logic for `"UnknownTransactionCommitResult"`.
- The core transaction API does not incorporate retry logic for `"UnknownTransactionCommitResult"`. To handle `"UnknownTransactionCommitResult"`, applications should explicitly incorporate retry logic for the error.

提交操作是[可重试的写操作](https://docs.mongodb.com/v4.4/core/retryable-writes/)。如果提交操作遇到错误，无论[`retryWrites`](https://docs.mongodb.com/v4.4/reference/connection-string/#mongodb-urioption-urioption.retryWrites)的值是多少，MongoDB驱动程序都会重试提交。

如果提交操作遇到标记为`"UnknownTransactionCommitResult"`的错误，提交可以被重试。

- 回调API包含了`"UnknownTransactionCommitResult"`的重试逻辑。
- 核心事务API不包含`"UnknownTransactionCommitResult"`的重试逻辑。为了处理`"UnknownTransactionCommitResult"`，应用程序应该明确地包含错误的重试逻辑。



### Driver Version Errors

### 驱动程序版本错误

On sharded clusters with multiple [`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos) instances, performing transactions with drivers updated for MongoDB 4.0 (instead of MongoDB 4.2) will fail and can result in errors, including:

NOTE

Your driver may return a different error. Refer to your driver's documentation for details.

在具有多个[`mongos`](https://docs.mongodb.com/v4.4/reference/program/mongos/#mongodb-binary-bin.mongos)实例的分片集群上，使用为MongoDB 4.0更新的驱动程序执行事务（而不是 MongoDB 4.2）将失败并可能导致错误，包括：

注意

你的驱动程序可能会返回不同的错误。有关详细信息，请参阅驱动程序的文档。

| Error Code | Error Message                                           |
| :--------- | :------------------------------------------------------ |
| 251        | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940      | `cannot commit with no participants`                    |

*For transactions on MongoDB 4.2 deployments (replica sets and sharded clusters)*, use the MongoDB drivers updated for MongoDB 4.2

*对于MongoDB 4.2部署（副本集和分片集群）上的事务*，使用为MongoDB 4.2更新的MongoDB驱动程序

## Additional Information

## 附加信息

### `mongo` Shell Example

### `mongo` Shell示例

The following [`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell methods are available for transactions:

- [`Session.startTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.startTransaction/#mongodb-method-Session.startTransaction)
- [`Session.commitTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.commitTransaction/#mongodb-method-Session.commitTransaction)
- [`Session.abortTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.abortTransaction/#mongodb-method-Session.abortTransaction)

NOTE

The [`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell example omits retry logic and robust error handling for simplicity's sake. For a more practical example of incorporating transactions in applications, see [Transaction Error Handling](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transactions-retry) instead.

下面列出的[`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell方法可用于事务：

- [`Session.startTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.startTransaction/#mongodb-method-Session.startTransaction)
- [`Session.commitTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.commitTransaction/#mongodb-method-Session.commitTransaction)
- [`Session.abortTransaction()`](https://docs.mongodb.com/v4.4/reference/method/Session.abortTransaction/#mongodb-method-Session.abortTransaction)

注意

[`mongo`](https://docs.mongodb.com/v4.4/reference/program/mongo/#mongodb-binary-bin.mongo) shell示例为了简单起见省略了重试逻辑和强大的错误处理。有关在应用程序中包含事务的更实际示例，请参阅[事务错误处理](https://docs.mongodb.com/v4.4/core/transactions-in-applications/#std-label-transactions-retry) .

```
// 创建集合：
db.getSiblingDB("mydb1").foo.insert( {abc: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );
db.getSiblingDB("mydb2").bar.insert( {xyz: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );

// 开启一个会话。
session = db.getMongo().startSession( { readPreference: { mode: "primary" } } );

coll1 = session.getDatabase("mydb1").foo;
coll2 = session.getDatabase("mydb2").bar;

// 开启一个事务
session.startTransaction( { readConcern: { level: "local" }, writeConcern: { w: "majority" } } );

// 事务内的操作
try {
   coll1.insertOne( { abc: 1 } );
   coll2.insertOne( { xyz: 999 } );
} catch (error) {
   // 遇到错误时中止事务
   session.abortTransaction();
   throw error;
}

// 在事务开始时使用写关注提交事务
session.commitTransaction();

session.endSession();
```



原文链接：https://docs.mongodb.com/manual/core/transactions-in-applications/

译者：李正洋
