# MongoDB Limits and Thresholds MongoDB中的限制与阈值

This document provides a collection of hard and soft limitations of the MongoDB system.

本文档提供了MongoDB系统的各种硬性和软性限制。

## BSON Documents BSON文档

- `BSON Document Size` **BSON文档大小**

  The maximum BSON document size is 16 megabytes.

  The maximum document size helps ensure that a single document cannot use excessive amount of RAM or, during transmission, excessive amount of bandwidth. To store documents larger than the maximum size, MongoDB provides the GridFS API. See [`mongofiles`](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles) and the documentation for your [driver](https://docs.mongodb.com/drivers/) for more information about GridFS.

  BSON的最大文档大小为16MB。

  最大文档大小有助于确保单个文档不会使用过多的RAM或在传输过程中占用过多的带宽。要存储大于该限制的文档，MongoDB提供了GridFS API。有关GridFS的更多信息，请参阅[mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)和[驱动程序](https://docs.mongodb.com/drivers/)的文档。

- `Nested Depth for BSON Documents` **BSON文档的嵌套深度**

  MongoDB supports no more than 100 levels of nesting for [BSON documents](https://docs.mongodb.com/manual/reference/glossary/#std-term-document).

  MongoDB支持不超过100层嵌套深度的[BSON文档](https://docs.mongodb.com/manual/reference/glossary/#std-term-document)。

## Naming Restrictions 命名限制

- `Database Name Case Sensitivity` **数据库名称的大小写敏感性**

  Since database names are case *insensitive* in MongoDB, database names cannot differ only by the case of the characters.

  由于数据库名称在MongoDB中*不区分大小写*，因此数据库名称不能仅因字符的大小写而不同。

- `Restrictions on Database Names for Windows` **Windows环境下的数据库名称限制**

  For MongoDB deployments running on Windows, database names cannot contain any of the following characters:

  对于在Windows上运行的MongoDB环境，数据库名不能包含以下任意一个字符：

  ````bash
  /\. "$*<>:|?
  ````

  Also database names cannot contain the null character.

  另外，数据库名不能包含空字符。

- `Restrictions on Database Names for Unix and Linux Systems` **Unix/Linux系统中的数据库名称限制**

  For MongoDB deployments running on Unix and Linux systems, database names cannot contain any of the following characters:

  对于在Unix和Linux系统上运行的MongoDB环境，数据库名不能包含以下任意一个字符：

  ```bash
  `/\. "$`
  ```

  Also database names cannot contain the null character.

  同样的，数据库名不能包含空字符。

- `Length of Database Names` **数据库名称的长度**

  Database names cannot be empty and must have fewer than 64 characters.

  数据库名不能为空并且必须小于64个字符。

- `Restriction on Collection Names` **集合名称的限制**

  Collection names should begin with an underscore or a letter character, and *cannot*:

  - contain the `$`.

  - be an empty string (e.g. `""`).

  - contain the null character.

  - begin with the `system.` prefix. (Reserved for internal use.)
  
  If your collection name includes special characters, such as the underscore character, or begins with numbers, then to access the collection use the [`db.getCollection()`](https://docs.mongodb.com/manual/reference/method/db.getCollection/#mongodb-method-db.getCollection) method in the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell or a [similar method for your driver](https://api.mongodb.com/).
  
  集合名必须以下划线或者字母符号开始，并且*不能*：
  
  - 包含`$`；
  - 为空字符串（比如`""`）；
  - 包含空字符；
  - 以`system.`为前缀（这部分表保留给内部使用）；
  
  如果您的集合名称包含特殊字符（例如下划线字符）或以数字开头，则可以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell中的[db.getCollection()](https://docs.mongodb.com/manual/reference/method/db.getCollection/#mongodb-method-db.getCollection)方法或[驱动程序的类似方法](https://api.mongodb.com/)来访问集合。
  
  Namespace Length:
  
  - For [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) set to `"4.4"` or greater, MongoDB raises the limit on collection/view namespace to 255 bytes. For a collection or a view, the namespace includes the database name, the dot (`.`) separator, and the collection/view name (e.g. `<database>.<collection>`),
  - For [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) set to `"4.2"` or earlier, the maximum length of the collection/view namespace remains 120 bytes.
  
  命名空间长度：
  
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.4"`**及以上的集群，MongoDB会将对集合/视图名称空间的限制提高到255个字节。对于集合或视图，命名空间包括数据库名称、点号（**`.`**）分隔符和集合/视图名称（例如**`<database>.<collection>`**）;
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为`"4.2"`及以下的集群，集合/视图名称空间的最大长度仍然为120个字节。
  
- `Restrictions on Field Names` **字段名称的限制**

  - Field names **cannot** contain the `null` character.

  - Top-level field names **cannot** start with the dollar sign (`$`) character.Otherwise, starting in MongoDB 3.6, the server permits storage of field names that contain dots (i.e. `.`) and dollar signs (i.e. `$`).

  - 字段名称**不能**包含空字符。
  
  - 顶级字段名称**不能**以美元符号（`$`）字符开头。
  
  此外，从MongoDB 3.6开始，服务器允许存储包含点（即`.`）和美元符号（即`$`）的字段名称。
  
    > **IMPORTANT  重要**
    >
    > The MongoDB Query Language cannot always meaningfully express queries over documents whose field names contain these characters (see [SERVER-30575](https://jira.mongodb.org/browse/SERVER-30575)).Until support is added in the query language, the use of `$` and `.` in field names is not recommended and is not supported by the official MongoDB drivers.
    >
    > MongoDB查询语言无法始终对字段名称包含这些字符的文档查询进行有效地表达（请参阅[SERVER-30575](https://jira.mongodb.org/browse/SERVER-30575)）。
    > 在查询语言添加相关支持之前，建议不要在字段名称中包含`.`和`$`，并且不受MongoDB官方驱动程序支持。
  
  > **WARNING 警告**
  >
  > **MongoDB does not support duplicate field names**
  >
  > The MongoDB Query Language is undefined over documents with duplicate field names. BSON builders may support creating a BSON document with duplicate field names. While the BSON builder may not throw an error, inserting these documents into MongoDB is not supported *even if* the insert succeeds. For example, inserting a BSON document with duplicate field names through a MongoDB driver may result in the driver silently dropping the duplicate values prior to insertion.
  >
  > **MongoDB不支持重复的字段名称**
  >
  > MongoDB查询语言对于具有重复字段名称的文档是未定义的。BSON构建器可能支持使用重复的字段名称创建BSON文档。尽管BSON构建器可能不会抛出错误，但是*即使*插入操作返回成功，也不支持将这些文档插入MongoDB。例如，通过MongoDB驱动程序插入具有重复字段名称的BSON文档可能会导致驱动程序在插入之前静默删除重复值。

## Namespaces 命名空间

- `Namespace Length` **命名空间长度**

  - For [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) set to `"4.4"` or greater, MongoDB raises the limit on collection/view namespace to 255 bytes. For a collection or a view, the namespace includes the database name, the dot (`.`) separator, and the collection/view name (e.g. `<database>.<collection>`)
  - For [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) set to `"4.2"` or earlier, the maximum length of the collection/view namespace remains 120 bytes.
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.4"`**及以上的环境，MongoDB会将对集合/视图名称空间的限制提高到255个字节。对于集合或视图，命名空间包括数据库名称、点号（**`.`**）分隔符和集合/视图名称（例如**`<database>.<collection>`**）;
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.2"`**及以下的环境，集合/视图名称空间的最大长度仍然为120个字节。

  > **TIP 提示**
  >
  > See also:[Naming Restrictions](https://docs.mongodb.com/manual/reference/limits/#std-label-faq-restrictions-on-collection-names)
  >
  > 另请参考：[命名限制](https://docs.mongodb.com/manual/reference/limits/#std-label-faq-restrictions-on-collection-names)


## Indexes 索引

- `Index Key Limit` **索引键的限制**

  > **NOTE 注意**
  >
  > **Changed in version 4.2 4.2版本有变更**
  >
  > Starting in version 4.2, MongoDB removes the [`Index Key Limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit) for [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) (fCV) set to `"4.2"` or greater.
  >
  > 从4.2版本开始，MongoDB对于将[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置成**`"4.2"`**及以上的环境去除了此[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)。
  
  For MongoDB 2.6 through MongoDB versions with fCV set to `"4.0"` or earlier, the *total size* of an index entry, which can include structural overhead depending on the BSON type, must be *less than* 1024 bytes.
  
  When the [`Index Key Limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit) applies:
  
  - MongoDB will **not** create an index on a collection if the index entry for an existing document exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit).
  - Reindexing operations will error if the index entry for an indexed field exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit). Reindexing operations occur as part of the [`compact`](https://docs.mongodb.com/manual/reference/command/compact/#mongodb-dbcommand-dbcmd.compact) command as well as the [`db.collection.reIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.reIndex/#mongodb-method-db.collection.reIndex) method.Because these operations drop *all* the indexes from a collection and then recreate them sequentially, the error from the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit) prevents these operations from rebuilding any remaining indexes for the collection.
  - MongoDB will not insert into an indexed collection any document with an indexed field whose corresponding index entry would exceed the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit), and instead, will return an error. Previous versions of MongoDB would insert but not index such documents.
  - Updates to the indexed field will error if the updated value causes the index entry to exceed the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit).If an existing document contains an indexed field whose index entry exceeds the limit, *any*update that results in the relocation of that document on disk will error.
  -  and [`mongoimport`](https://docs.mongodb.com/database-tools/mongoimport/#mongodb-binary-bin.mongoimport) will not insert documents that contain an indexed field whose corresponding index entry would exceed the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit).
  - In MongoDB 2.6, secondary members of replica sets will continue to replicate documents with an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit) on initial sync but will print warnings in the logs.Secondary members also allow index build and rebuild operations on a collection that contains an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit) but with warnings in the logs.With *mixed version* replica sets where the secondaries are version 2.6 and the primary is version 2.4, secondaries will replicate documents inserted or updated on the 2.4 primary, but will print error messages in the log if the documents contain an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit).
  - For existing sharded collections, [chunk migration](https://docs.mongodb.com/manual/core/sharding-balancer-administration/) will fail if the chunk has a document that contains an indexed field whose index entry exceeds the [`index key limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit).
  
  对于从MongoDB 2.6到将fCV设置为**`"4.2"`**或更早的MongoDB版本，索引条目的*总大小*必须*小于*1024字节，该总大小可能包括结构体开销，具体取决于BSON类型。
  
  当[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)存在时：
  
  - 如果现有文档的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则MongoDB**不会**在集合上创建索引。
  - 如果索引字段的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则重新索引操作将出错。重新索引操作是[compact]()命令以及[db.collection.reIndex()]()方法的一部分，因为这些操作会删除集合中的*所有*索引，然后按顺序重新创建它们，所以[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)中的错误阻止了这些操作的重建集合的所有剩余索引。
  - MongoDB不会将任何具有索引字段的文档插入到索引集合中，该文档的索引字段的对应索引条目将超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，而是将返回错误。MongoDB的早期版本将插入此类文档，但不会为其创建索引。
  - 如果更新的值导致索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，则对索引字段的更新将出错。如果现有文档包含索引条目超过该限制的索引字段，则导致该文档在磁盘上重新定位的*任何*更新都将返回错误。
  - [mongorestore](https://docs.mongodb.com/database-tools/mongorestore/#mongodb-binary-bin.mongorestore)和[mongoimport](https://docs.mongodb.com/database-tools/mongoimport/#mongodb-binary-bin.mongoimport)将不会插入包含索引字段的文档，该字段的相应索引条目将超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)。
  - 在MongoDB 2.6中，如果该索引字段的对应索引条目在初始同步时超出了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)，副本集的从节点将继续复制带有索引字段的文档，但会在日志中显示警告信息。从节点还允许对包含了对应的索引条目超过了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)的索引字段的集合进行索引构建和重建操作，但在日志中显示警告信息。使用混合版本副本集（其中次要版本为2.6和主版本为版本2.4），从节点将复制在2.4主版本上插入或更新的文档，但是如果文档包含一个索引字段（其对应的索引条目超过了[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)），则会在日志中显示错误消息。
  - 对于现有分片集合，如果块中包含文档的索引条目超过[索引键限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)的索引字段，则块迁移将失败。
  
- `Number of Indexes per Collection` **每个集合中的索引个数**

  A single collection can have *no more* than 64 indexes.

  单个集合内*不能超过*64个索引。

- `Index Name Length` **索引名称长度**

  > **NOTE 注意**
  >
  > **Changed in version 4.2 4.2版本有变更**
  >
  > Starting in version 4.2, MongoDB removes the [`Index Name Length`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length) limit for MongoDB versions with[featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) (fCV) set to `"4.2"` or greater.
  >
  > 从4.2版本开始，MongoDB对于将[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置成**`"4.2"`**及以上的环境去除了此[索引名称长度限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)。

  In previous versions of MongoDB or MongoDB versions with fCV set to `"4.0"` or earlier, fully qualified index names, which include the namespace and the dot separators (i.e. `<database name>.<collection name>.$<index name>`), cannot be longer than 127 bytes.

  By default, `<index name>` is the concatenation of the field names and index type. You can explicitly specify the `<index name>` to the [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) method to ensure that the fully qualified index name does not exceed the limit.

  在将fCV设置为**`"4.0"`**及以下的MongoDB或MongoDB的早期版本中，标准的索引名称，包括名称空间和点分隔符（即`<database name>.<collection name>.$<index name>`），不能超过127个字节。

  默认情况下，`<index name>`是字段名称和索引类型的串联。您可以为`createIndex()`方法显式指定`<index name>`，以确保标准索引名称不超过限制。

- `Number of Indexed Fields in a Compound Index` **复合索引的字段数量**

  There can be no more than 32 fields in a compound index.

  复合索引中所包含的字段不能超过32个。

- `Queries cannot use both text and Geospatial Indexes` **查询不能同时使用文本索引和地理空间索引**

  You cannot combine the [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text) query, which requires a special [text index](https://docs.mongodb.com/manual/core/index-text/#std-label-create-text-index), with a query operator that requires a different type of special index. For example you cannot combine [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text) query with the [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near) operator.

  您不能将需要特殊[文本索引](https://docs.mongodb.com/manual/core/index-text/#std-label-create-text-index)的[$text](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)查询与需要不同类型特殊索引的查询运算符组合在一起。例如，您不能将[$text](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)查询与[$near](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)运算符结合使用。

- `Fields with 2dsphere Indexes can only hold Geometries` **具有2dsphere索引的字段只能保存几何数据**

  Fields with [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) indexes must hold geometry data in the form of [coordinate pairs](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs) or [GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON) data. If you attempt to insert a document with non-geometry data in a `2dsphere` indexed field, or build a `2dsphere`index on a collection where the indexed field has non-geometry data, the operation will fail.

  具有[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)索引的字段必须以[坐标对](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs)或[GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON)数据的形式保存几何数据。如果您尝试在**2dsphere**索引字段中插入包含非几何数据的文档，或者在索引字段包含非几何数据的集合上构建**2dsphere**索引，则该操作将失败。

  > **TIP 提示**
  >
  > **See also: 另请参考：**
  >
  > The unique indexes limit in [Sharding Operational Restrictions](https://docs.mongodb.com/manual/reference/limits/#std-label-limits-sharding-operations).
  >
  > [分片操作限制](https://docs.mongodb.com/manual/reference/limits/#std-label-limits-sharding-operations)中的唯一索引限制

- `NaN values returned from Covered Queries by the WiredTiger Storage Engine are always of type double` **WiredTiger存储引擎从覆盖查询返回的NaN值始终为double类型**

  If the value of a field returned from a query that is [covered by an index](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries) is `NaN`, the type of that `NaN` value is *always* `double`.

  如果从索引覆盖的查询返回的字段的值为**NaN**，则该**NaN**值的类型*始终*为**double**。

- `Multikey Index` **多键索引**

  [Multikey indexes](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) cannot cover queries over array field(s).

  多键索引不能覆盖对数组字段的查询。

- `Geospatial Index` **地理位置索引**

  [Geospatial indexes](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial) cannot [cover a query](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries).

  地理位置索引无法覆盖查询。

- `Memory Usage in Index Builds` **索引构建中的内存使用情况** 

  supports building one or more indexes on a collection. [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) uses a combination of memory and temporary files on disk to complete index builds. The default limit on memory usage for [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) is 200 megabytes (for versions 4.2.3 and later) and 500 (for versions 4.2.2 and earlier), shared between all indexes built using a single [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) command. Once the memory limit is reached, [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) uses temporary disk files in a subdirectory named `_tmp` within the [`--dbpath`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--dbpath)directory to complete the build.

  You can override the memory limit by setting the [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes) server parameter. Setting a higher memory limit may result in faster completion of index builds. However, setting this limit too high relative to the unused RAM on your system can result in memory exhaustion and server shutdown.

  [createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)支持在集合上构建一个或多个索引。[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)使用内存和磁盘上的临时文件的组合来完成索引构建。[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)的内存使用量的默认限制是200MB（对于4.2.3和更高版本）和500MB（对于4.2.2和更早版本），这是使用单个[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)命令构建的所有索引之间共享的。一旦达到内存限制，[createIndexes](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)将使用[--dbpath](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--dbpath)指定的目录中名为`_tmp`子目录中的临时磁盘文件来完成构建。

  您可以通过设置[maxIndexBuildMemoryUsageMegabytes](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes)这一服务器参数来覆盖该内存限制。设置更高的内存限制可能会导致索引构建更快地完成。但是，相对于系统上未使用的RAM设置此限制过高会导致内存耗尽和MongoDB服务停止。

  *Changed in version 4.2*. 

  - For [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.2"`, the index build memory limit applies to all index builds.
  - For [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.0"`, the index build memory limit only applies to foreground index builds.

  *4.2版本有更新*

  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.2"`**的环境，索引创建的内存限制对所有索引创建生效；
  - 对于[fCV](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)设置为**`"4.0"`**的环境，索引创建的内存限制仅对前台建索引生效；

  Index builds may be initiated either by a user command such as [Create Index](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/) or by an administrative process such as an [initial sync](https://docs.mongodb.com/manual/core/replica-set-sync/). Both are subject to the limit set by [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes).

  An [initial sync operation](https://docs.mongodb.com/manual/core/replica-set-sync/) populates only one collection at a time and has no risk of exceeding the memory limit. However, it is possible for a user to start index builds on multiple collections in multiple databases simultaneously and potentially consume an amount of memory greater than the limit set in [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes).

  可以通过诸如[创建索引](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/)之类的用户命令或诸如[初始化同步](https://docs.mongodb.com/manual/core/replica-set-sync/)之类的管理过程来启动索引构建。两者均受[maxIndexBuildMemoryUsageMegabytes](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxIndexBuildMemoryUsageMegabytes)设置的限制。

  [初始化同步操作](https://docs.mongodb.com/manual/core/replica-set-sync/)一次仅填充一个集合，并且没有超过内存限制的风险。但是，用户可能会同时在多个数据库中的多个集合上启动索引构建，并且可能消耗的内存量大于[maxIndexBuildMemoryUsageMegabytes]()中设置的限制。

  >**TIP 提示**
  >To minimize the impact of building an index on replica sets and sharded clusters with replica set shards, use a rolling index build procedure as described on [Rolling Index Builds on Replica Sets](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/).
  >
  >为了最大程度地减少在副本集和具有副本集分片的分片集群上建立索引的影响，请使用滚动索引生成过程，如[在副本集上滚动索引构建](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/)所述。

- `Collation and Index Types` **字节序和索引类型**

  The following index types only support simple binary comparison and do not support [collation](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation):

  -  [text](https://docs.mongodb.com/manual/core/index-text/) indexes,
  -  [2d](https://docs.mongodb.com/manual/core/2d/) indexes, and
  -  [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/) indexes.

  以下索引类型仅支持简单的二进制比较规则而不支持[字节序](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation)：

  - [文本](https://docs.mongodb.com/manual/core/index-text/)索引;
  - [2d](https://docs.mongodb.com/manual/core/2d/)索引；
  - [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/)索引。

  > **TIP 提示**
  >
  > To create a `text`, a `2d`, or a `geoHaystack` index on a collection that has a non-simple collation, you must explicitly specify `{collation: {locale: "simple"} }` when creating the index.
  >
  > 为了在一个包含非简单字节序的集合上创建一个`text`，`2d`或`geoHaystack`索引，您必须在创建索引时显示指定`collation: {locale: "simple"}`。

- `Hidden Indexes` **隐藏索引**

  - You cannot [hide](https://docs.mongodb.com/manual/core/index-hidden/) the `_id` index.
  
  - You cannot use [`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#mongodb-method-cursor.hint) on a [hidden index](https://docs.mongodb.com/manual/core/index-hidden/).
  - 你无法[隐藏](https://docs.mongodb.com/manual/core/index-hidden/)`_id`索引。
  - 在[隐藏索引](https://docs.mongodb.com/manual/core/index-hidden/)上无法使用[hint()](https://docs.mongodb.com/manual/reference/method/cursor.hint/#mongodb-method-cursor.hint)

## Data 数据

- `Maximum Number of Documents in a Capped Collection` **限制集合中的最大文档数量**

  If you specify a maximum number of documents for a capped collection using the `max` parameter to [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create), the limit must be less than 2^32 documents. If you do not specify a maximum number of documents when creating a capped collection, there is no limit on the number of documents.
  
  如果使用`max`参数为限制集合指定最大文档数，则该限制必须少于`2^32`个文档。如果在创建上限集合时未指定最大文档数，则对文档数没有限制。

## Replica Sets 副本集

- `Number of Members of a Replica Set` **副本集成员个数**

  Replica sets can have up to 50 members. 副本集能拥有不超过50个成员。

- `Number of Voting Members of a Replica Set` **副本集中可投票成员个数**

  Replica sets can have up to 7 voting members. For replica sets with more than 7 total members, see [Non-Voting Members](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-non-voting-members).

  副本集最多可以有7个投票成员。有关成员总数超过7个的副本集，请参阅[非投票成员](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-non-voting-members)。

- `Maximum Size of Auto-Created Oplog` **自动创建的oplog表的最大大小**

  If you do not explicitly specify an oplog size (i.e. with [`oplogSizeMB`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.oplogSizeMB) or [`--oplogSize`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--oplogSize)) MongoDB will create an oplog that is no larger than 50 gigabytes. [[1\]](https://docs.mongodb.com/manual/reference/limits/#footnote-oplog)[[1](https://docs.mongodb.com/manual/reference/limits/#ref-oplog-id1)]Starting in MongoDB 4.0, the oplog can grow past its configured size limit to avoid deleting the [`majority commit point`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#mongodb-data-replSetGetStatus.optimes.lastCommittedOpTime).
  
  如果您未明确指定oplog表的大小（即使用[oplogSizeMB](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.oplogSizeMB)或[--oplogSize](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--oplogSize)），则MongoDB将创建一个不超过50GB的oplog表。[1]
  
  > [1]从MongoDB 4.0开始，操作日志可以超过其配置的大小限制，以避免删除[大多数提交点](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#mongodb-data-replSetGetStatus.optimes.lastCommittedOpTime)。

## Sharded Clusters 分片集群

Sharded clusters have the restrictions and thresholds described here.

分片群集具有此处描述的限制和阈值。

### Sharding Operational Restrictions 分片操作限制

- `Operations Unavailable in Sharded Environments` **分片环境中无法执行的操作**

   [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) does not permit references to the `db` object from the [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) function. This is uncommon in un-sharded collections.

  The [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch) command is not supported in sharded environments.
  [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) 不允许从[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where) 函数引用db对象。这在未分片的集合中并不常见。

  分片环境不支持[geoSearch](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)命令。

- `Covered Queries in Sharded Clusters` **分片集群中的覆盖索引**

  Starting in MongoDB 3.0, an index cannot [cover](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries) a query on a [sharded](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard) collection when run against a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)if the index does not contain the shard key, with the following exception for the `_id` index: If a query on a sharded collection only specifies a condition on the `_id` field and returns only the `_id` field, the `_id` index can cover the query when run against a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos) even if the `_id` field is not the shard key.

  In previous versions, an index cannot [cover](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries) a query on a [sharded](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard) collection when run against a [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos).

  从MongoDB 3.0开始，如果索引不包含分片键，则对于运行在[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)上的查询而言，索引不能[覆盖](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries)[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)集合上的查询，但`_id`索引除外：如果分片集合上的查询仅指定条件在`_id`字段上并仅返回`_id`字段，即使`_id`字段不是分片键，`_id`索引也可以覆盖查询。

  在以前的版本中，对于运行在[mongos](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)上的查询而言，索引无法[覆盖](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries)[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)集合上的查询。

- `Sharding Existing Collection Data Size` 对已存在的集合进行分片的数据大小限制

  An existing collection can only be sharded if its size does not exceed specific limits. These limits can be estimated based on the average size of all [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) values, and the configured [chunk](https://docs.mongodb.com/manual/reference/glossary/#std-term-chunk) size.

  如果现有集合的大小未超过特定限制，则只能对其进行分片。可以基于所有分片键值的平均大小以及配置的块大小来估计这些限制。
  
  >  **IMPORTANT 重要**
  >
  >  These limits only apply for the initial sharding operation. Sharded collections can grow to *any* size after successfully enabling sharding.
>
  >  这些限制仅适用于初始化分片操作。成功启用分片后，分片集合可以增长到任何大小。

  Use the following formulas to calculate the *theoretical* maximum collection size.

  如果如下的公式来计算*理论*最大集合大小。

  ```bash
  maxSplits = 16777216 (bytes) / <average size of shard key values in bytes> maxCollectionSize (MB) = maxSplits * (chunkSize / 2)
  ```

  > **NOTE 注意**
  >
  > The maximum [BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON) document size is 16MB or `16777216` bytes.
  >
  > All conversions should use base-2 scale, e.g. 1024 kilobytes = 1 megabyte.
  >
  > BSON文档的最大大小为`16MB`或者`16777216B`。
  >
  > 所有的转换都是基于二进制的，比如1024KB = 1MB。

If `maxCollectionSize` is less than or nearly equal to the target collection, increase the chunk size to ensure successful initial sharding. If there is doubt as to whether the result of the calculation is too 'close' to the target collection size, it is likely better to increase the chunk size.

After successful initial sharding, you can reduce the chunk size as needed. If you later reduce the chunk size, it may take time for all chunks to split to the new size. See [Modify Chunk Size in a Sharded Cluster](https://docs.mongodb.com/manual/tutorial/modify-chunk-size-in-sharded-cluster/) for instructions on modifying chunk size.

This table illustrates the approximate maximum collection sizes using the formulas described above:

如果**maxCollectionSize**小于或几乎等于目标集合，则增加块大小以确保成功进行初始分片。如果对计算结果是否过于“接近”目标集合大小有疑问，最好增加块大小。

成功完成初始化分片后，您可以根据需要减小块大小。如果以后减小块大小，则所有块可能都需要花费一些时间才能拆分为新的大小。有关修改块大小的说明，请参阅[修改分片群集中的块大小](https://docs.mongodb.com/manual/tutorial/modify-chunk-size-in-sharded-cluster/)。

下表使用上述公式说明了最大的集合规模： 

| 分片键值的平均大小              | 512字节 | 256字节 | 128字节 | 64字节 |
| :------------------------------ | ------- | ------- | ------- | ------ |
| **拆分的最大次数**              | 32768   | 65536   | 131072  | 262144 |
| **最大集合大小（块大小64MB）**  | 1TB     | 2TB     | 4TB     | 8TB    |
| **最大集合大小（块大小128MB）** | 2TB     | 4TB     | 8TB     | 16TB   |
| **最大集合大小（块大小256MB）** | 4TB     | 8TB     | 16TB    | 32TB   |

- `Single Document Modification Operations in Sharded Collections` **分片集合中的单文档修改操作**

  All [`update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update) and [`remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#mongodb-method-db.collection.remove) operations for a sharded collection that specify the `justOne` or `multi: false` option must include the [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) *or* the `_id` field in the query specification. [`update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update) and [`remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#mongodb-method-db.collection.remove) operations specifying `justOne` or `multi: false` in a sharded collection which do not contain either the [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) or the `_id` field return an error.
  
  指定了`justOne`或`multi：false`选项的分片集合的所有[`update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update)和[`remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/#mongodb-method-db.collection.remove)操作必须在查询条件中包括[分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)或`_id`字段。否则将返回错误。

- `Unique Indexes in Sharded Collections` **分片集合中的唯一索引**

  MongoDB does not support unique indexes across shards, except when the unique index contains the full shard key as a prefix of the index. In these situations MongoDB will enforce uniqueness across the full key, not a single field.
  
  MongoDB不支持跨分片的唯一索引，除非唯一索引包含完整的分片键作为索引前缀。在这些情况下，MongoDB将在整个索引键上而不是单个字段上进行唯一性约束。
  
  > **TIP 提示**
  >
  > See:[Unique Constraints on Arbitrary Fields](https://docs.mongodb.com/manual/tutorial/unique-constraints-on-arbitrary-fields/#std-label-shard-key-arbitrary-uniqueness) for an alternate approach.
  >
  > 替代方法请参考[任意字段的唯一性约束](https://docs.mongodb.com/manual/tutorial/unique-constraints-on-arbitrary-fields/#std-label-shard-key-arbitrary-uniqueness)。

- `Maximum Number of Documents Per Chunk to Migrate` **迁移时每个块的最大文档数量**

  By default, MongoDB cannot move a chunk if the number of documents in the chunk is greater than 1.3 times the result of dividing the configured [chunk size](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-sharding-chunk-size) by the average document size. [`db.collection.stats()`](https://docs.mongodb.com/manual/reference/method/db.collection.stats/#mongodb-method-db.collection.stats)includes the `avgObjSize` field, which represents the average document size in the collection.
  
  For chunks that are [too large to migrate](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#std-label-migration-chunk-size-limit), starting in MongoDB 4.4:
  
  - A new balancer setting `attemptToBalanceJumboChunks` allows the balancer to migrate chunks too large to move as long as the chunks are not labeled [jumbo](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk). See [Balance Chunks that Exceed Size Limit](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-balance-chunks-that-exceed-size-limit) for details.
  - The [`moveChunk`](https://docs.mongodb.com/manual/reference/command/moveChunk/#mongodb-dbcommand-dbcmd.moveChunk) command can specify a new option [forceJumbo](https://docs.mongodb.com/manual/reference/command/moveChunk/#std-label-movechunk-forceJumbo) to allow for the migration of chunks that are too large to move. The chunks may or may not be labeled [jumbo](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk).
  
  默认情况下，如果块中的文档数大于配置的[块大小](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-sharding-chunk-size)除以平均文档大小所得结果的1.3倍，则MongoDB无法移动该块。[`db.collection.stats()`](https://docs.mongodb.com/manual/reference/method/db.collection.stats/#mongodb-method-db.collection.stats)的返回结果包含了`avgObjSize`字段，该字段表示集合中的平均文档大小。
  
  对于[太大而无法迁移](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#std-label-migration-chunk-size-limit)的块，从MongoDB 4.4开始：
  
  - 新的平衡器设置——`tryToBalanceJumboChunks`允许平衡器迁移过大而无法移动的块，只要这些块未标记为[巨型(Jubmo)](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk)即可。有关详细信息请参见[均衡超出大小限制的块](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#std-label-balance-chunks-that-exceed-size-limit)。
  - [`moveChunk`](https://docs.mongodb.com/manual/reference/command/moveChunk/#mongodb-dbcommand-dbcmd.moveChunk)命令可以指定一个新选项[forceJumbo](https://docs.mongodb.com/manual/reference/command/moveChunk/#std-label-movechunk-forceJumbo)，以允许迁移过大而无法移动的块，无论该块有没有被标记为[巨型(Jubmo)](https://docs.mongodb.com/manual/core/sharding-data-partitioning/#std-label-jumbo-chunk)。

### Shard Key Limitations 分片键限制

- `Shard Key Size` **分片键大小**

  Starting in version 4.4, MongoDB removes the limit on the shard key size.

  For MongoDB 4.2 and earlier, a shard key cannot exceed 512 bytes.

  从4.4版本开始，MongoDB去除了关于分片键大小的限制。

  在4.2及之前的版本，一个分片键大小不能超过512B。

- `Shard Key Index Type` **分片键索引类型**

  A [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) index can be an ascending index on the shard key, a compound index that start with the shard key and specify ascending order for the shard key, or a [hashed index](https://docs.mongodb.com/manual/core/index-hashed/).

  A [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) index cannot be an index that specifies a [multikey index](https://docs.mongodb.com/manual/core/index-multikey/), a [text index](https://docs.mongodb.com/manual/core/index-text/) or a [geospatial index](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial) on the [shard key](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) fields.

  [分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)索引可以是分片键上的升序索引，也可以是以分片键开头并为分片键指定升序的复合索引，也可以是[哈希索引](https://docs.mongodb.com/manual/core/index-hashed/)。

  [分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)索引不能是在[分片键](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key)字段上指定的[多键索引](https://docs.mongodb.com/manual/core/index-multikey/)，[文本索引](https://docs.mongodb.com/manual/core/index-text/)或[地理空间索引](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial)。

- `Shard Key Selection is Immutable in MongoDB 4.2 and Earlier` **分片键在MongoDB4.2及以前的版本中是不可改变的**

  > **NOTE 注意**
  >
  > **Changed in Version 4.4**
  >
  > Starting in MongoDB 4.4, you can refine a collection's shard key by adding a suffix field or fields to the existing key. See [`refineCollectionShardKey`](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#mongodb-dbcommand-dbcmd.refineCollectionShardKey).
  >
  > **4.4版本中更新**
  >
  > 从MongoDB 4.4开始，您可以通过向现有键添加一个或多个后缀字段来优化集合的分片键。请参阅[fineCollectionShardKey](https://docs.mongodb.com/manual/reference/command/refineCollectionShardKey/#mongodb-dbcommand-dbcmd.refineCollectionShardKey)。

  In MongoDB 4.2 and earlier, once you shard a collection, the selection of the shard key is immutable; i.e. you cannot select a different shard key for that collection.

  If you must change a shard key:

  - Dump all data from MongoDB into an external format.
  - Drop the original sharded collection.
  - Configure sharding using the new shard key.
  - [Pre-split](https://docs.mongodb.com/manual/tutorial/create-chunks-in-sharded-cluster/) the shard key range to ensure initial even distribution.
  - Restore the dumped data into MongoDB.

  在MongoDB 4.2和更早版本中，一旦对集合进行分片，则分片键是不可改变的。也就是说，您不能为该集合选择其他分片键。

  如果必须更改分片键（则需要进行以下的重建步骤）：

  - 将MongoDB中的所有数据转储为外部格式。
  - 删除原始分片集合。
  - 使用新的分片密钥配置分片。
  - 对分片建范围进行[预分片](https://docs.mongodb.com/manual/tutorial/create-chunks-in-sharded-cluster/)以确保初始均匀分配。
  - 将转储的数据还原到MongoDB中。

- `Monotonically Increasing Shard Keys Can Limit Insert Throughput` **单调递增的分片键会限制插入性能**

  For clusters with high insert volumes, a shard keys with monotonically increasing and decreasing keys can affect insert throughput. If your shard key is the `_id` field, be aware that the default values of the `_id` fields are [ObjectIds](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId) which have generally increasing values.
  
  When inserting documents with monotonically increasing shard keys, all inserts belong to the same [chunk](https://docs.mongodb.com/manual/reference/glossary/#std-term-chunk) on a single [shard](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard). The system eventually divides the chunk range that receives all write operations and migrates its contents to distribute data more evenly. However, at any moment the cluster directs insert operations only to a single shard, which creates an insert throughput bottleneck.
  
  If the operations on the cluster are predominately read operations and updates, this limitation may not affect the cluster.
  
  To avoid this constraint, use a [hashed shard key](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) or select a field that does not increase or decrease monotonically.
  
  [Hashed shard keys](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) and [hashed indexes](https://docs.mongodb.com/manual/core/index-hashed/#std-label-index-type-hashed) store hashes of keys with ascending values.
  
  对于具有高插入量的集群，具有单调递增和递减性质的分片键可能会影响插入的吞吐量。如果您的分片键是`_id`字段，请注意`_id`字段的默认值是通常具有递增值的[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)。
  
  当使用单调递增的分片键进行插入文档操作时，所有的插入都落在单个[分片](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard)上的同一[块](https://docs.mongodb.com/manual/reference/glossary/#std-term-chunk)。系统最终划分接收所有写操作的块范围，并迁移其内容以更均匀地分配数据。但是，群集在任何时候都只将插入操作定向到单个分片，这会造成插入吞吐量的瓶颈。
  
  如果集群上的操作主要是读取操作和更新，则此限制可能不会影响集群。
  
  为避免此约束，请使用[哈希分片键](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding)或选择一个不会单调增加或减少的字段。
  
  [哈希分片键](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding)和[哈希索引](https://docs.mongodb.com/manual/core/index-hashed/#std-label-index-type-hashed)存储具有升序值的键的哈希值。

## Operations 操作

- `Sort Operations` **排序操作**

  If MongoDB cannot use an index or indexes to obtain the sort order, MongoDB must perform a blocking sort operation on the data. The name refers to the requirement that the `SORT` stage reads all input documents before returning any output documents, blocking the flow of data for that specific query.
  
  If MongoDB requires using more than 100 megabytes of system memory for the blocking sort operation, MongoDB returns an error *unless* the query specifies [`cursor.allowDiskUse()`](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse) (*New in MongoDB 4.4*). [`allowDiskUse()`](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse) allows MongoDB to use temporary files on disk to store data exceeding the 100 megabyte system memory limit while processing a blocking sort operation.
  
  *Changed in version 4.4*: For MongoDB 4.2 and prior, blocking sort operations could not exceed 32 megabytes of system memory.
  
  For more information on sorts and index use, see [Sort and Index Use](https://docs.mongodb.com/manual/reference/method/cursor.sort/#std-label-sort-index-use).
  
  如果MongoDB无法使用一个或多个索引来获取排序顺序，则MongoDB必须对数据执行阻塞式排序操作。该名称指的是`SORT`阶段在返回任何输出文档之前读取所有输入文档的要求，从而阻止了该特定查询的数据流。
  
  如果MongoDB要求使用100MB以上的系统内存进行阻塞排序操作，则除非查询指定[`cursor.allowDiskUse()`](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse)（*MongoDB 4.4中的新增功能*），否则MongoDB将返回错误。[allowDiskUse](https://docs.mongodb.com/manual/reference/method/cursor.allowDiskUse/#mongodb-method-cursor.allowDiskUse)允许MongoDB在处理阻塞排序操作时使用磁盘上的临时文件来存储超过100MB系统内存限制的数据。
  
   *在版本4.4中进行了更改*：对于MongoDB 4.2和更低版本，阻塞排序操作不能超过32MB系统内存。
  
  有关排序和索引使用的更多信息，请参见[排序和索引使用](https://docs.mongodb.com/manual/reference/method/cursor.sort/#std-label-sort-index-use)。
  
- `Aggregation Pipeline Operation` **聚合管道操作**

  Pipeline stages have a limit of 100 megabytes of RAM. If a stage exceeds this limit, MongoDB will produce an error. To allow for the handling of large datasets, use the `allowDiskUse` option to enable aggregation pipeline stages to write data to temporary files.*Changed in version 3.4*.The [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup) stage must stay within the 100 megabyte memory limit. If `allowDiskUse: true` is specified for the [`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate) operation, the [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup) stage ignores the option. If there are other stages in the [`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate) operation, `allowDiskUse: true` option is in effect for these other stages.Starting in MongoDB 4.2, the [profiler log messages](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) and [diagnostic log messages](https://docs.mongodb.com/manual/reference/log-messages/) includes a `usedDisk`indicator if any aggregation stage wrote data to temporary files due to [memory restrictions](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#std-label-agg-memory-restrictions).

  流水线级的RAM限制为100MB。如果阶段超出此限制，则MongoDB将产生错误。要允许处理大型数据集，请使用`allowDiskUse`选项启用聚合管道阶段以将数据写入临时文件。

  *在版本3.4中进行了更改*。

  [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)阶段必须保持在100 MB内存限制内。如果为[`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)操作指定了`allowDiskUse:true`，则[`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup)阶段将忽略该选项。如果[`aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)操作中还有其他阶段，则`allowDiskUse:true`选项对这些其他阶段有效。

  从MongoDB 4.2开始，[事件探查器日志消息]()和[诊断日志消息]()均包含`usedDisk`字段，其指示了是有否有聚合阶段由于[内存限制](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#std-label-agg-memory-restrictions)而将数据写入磁盘上临时文件。

  > **TIP 提示**
  >
  > **See also:**
  >
  > - [`$sort` and Memory Restrictions](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#std-label-sort-memory-limit) 
  >
  > - [`$group` Operator and Memory](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#std-label-group-memory-limit)
  >
  > **另请参考：**
  >
  > - [`$sort`与内存限制](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#std-label-sort-memory-limit)
  > - [`$group`操作符与内存](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#std-label-group-memory-limit)

- `Aggregation and Read Concern` **聚合以及读关注**

  - Starting in MongoDB 4.2, the [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) stage cannot be used in conjunction with read concern [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-). That is, if you specify [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-) read concern for[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate), you cannot include the [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) stage in the pipeline.
  - The [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) stage cannot be used in conjunction with read concern [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-). That is, if you specify [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-) read concern for [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate), you cannot include the [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) stage in the pipeline.
  - 从MongoDB 4.2开始，[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out)阶段不能与[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)指定[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注，则不能在管道中包括[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out)阶段。
  -  [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段不能与[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)级别的读关注结合使用。也就是说，如果为[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)指定[`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-)读取关注点，则不能在管道中包括[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段。

- `2d Geospatial queries cannot use the $or operator` **2d地理位置查询无法使用`$or`操作符**

  > **TIP 提示**
  >
  > **查看: 参考：**
  >
  > - [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#mongodb-query-op.-or)
  > - [`2d` Index Internals](https://docs.mongodb.com/manual/core/geospatial-indexes/)

- `Geospatial Queries` **地理位置查询**

  For spherical queries, use the `2dsphere` index result.

  The use of `2d` index for spherical queries may lead to incorrect results, such as the use of the `2d` index for spherical queries that wrap around the poles.

  对于地理位置查询，使用`dusphere`索引的结果。

  将`2d`索引用于球形查询可能会导致错误的结果，例如将`2d`索引用于环绕两极的球形查询。

- `Geospatial Coordinates` **地理空间坐标**

  - Valid longitude values are between `-180` and `180`, both inclusive.
  - Valid latitude values are between `-90` and `90`, both inclusive.
  - 有效的经度值在-180到180之间（包括两者）。
  - 有效的纬度值在-90到90之间（包括两者）。

- `Area of GeoJSON Polygons` **GeoJSON多边形的面积**

  For [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin), if you specify a single-ringed polygon that has an area greater than a single hemisphere, include [`the custom MongoDB coordinate reference system in the $geometry`](https://docs.mongodb.com/manual/reference/operator/query/geometry/#mongodb-query-op.-geometry) expression; otherwise, [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) queries for the complementary geometry. For all other GeoJSON polygons with areas greater than a hemisphere, [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) queries for the complementary geometry.

  对于[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects)或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)，如果您指定面积大于单个半球的单环多边形，则[在$ geometry表达式中包括自定义MongoDB坐标参考系统](https://docs.mongodb.com/manual/reference/operator/query/geometry/#mongodb-query-op.-geometry)； 否则，[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) 或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) 将查询互补几何。对于面积大于半球的所有其他GeoJSON多边形，[`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) 或 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) 查询互补几何。

- `Multi-document Transactions` **多文档事务**

  For [multi-document transactions](https://docs.mongodb.com/manual/core/transactions/):

  - You can specify read/write (CRUD) operations on **existing** collections. For a list of CRUD operations, see [CRUD Operations](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud).

  - When using  `"4.4"` or greater, you can create collections and indexes in transactions. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)

  - The collections used in a transaction can be in different databases.

    >  NOTEYou cannot create new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.

  - You cannot write to [capped](https://docs.mongodb.com/manual/core/capped-collections/) collections. (Starting in MongoDB 4.2)

  - You cannot read/write to collections in the `config`, `admin`, or `local` databases.

  - You cannot write to `system.*` collections.

  - You cannot return the supported operation's query plan (i.e. `explain`).

  - For cursors created outside of a transaction, you cannot call [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore) inside the transaction.

  - For cursors created in a transaction, you cannot call [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore) outside the transaction.

  - Starting in MongoDB 4.2, you cannot specify [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors) as the first operation in a [transaction](https://docs.mongodb.com/manual/core/transactions/).

  对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)而言：

  - 您可以在**现有**集合上指定读/写（CRUD）操作。有关CRUD操作的列表，请参阅[CRUD操作](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud)。

  - 使用[fcv](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)**`“4.4”`**或更高版本时，可以在事务中创建集合和索引。有关详细信息，请参见[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。

  - 事务中使用的集合可以位于不同的数据库中。

    > **注意** 
    >
    > 您无法在跨分片写入事务中创建新集合。例如，如果您在一个分片中写入现有集合，而在另一个分片中隐式创建一个集合，则MongoDB无法在同一事务中执行这两项操作。

  - 您无法写[限制（capped）](https://docs.mongodb.com/manual/core/capped-collections/)集合。（从MongoDB 4.2开始）

  - 您无法在`config`，`admin`或`local`数据库中读取/写入集合。

  -  您无法写入`system.*`集合。

  - 您无法返回受支持操作的查询计划（即`explain`）。

  - 对于在事务外部创建的游标，不能在事务内部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。对于在事务中创建的游标，不能在事务外部调用[`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)。从MongoDB 4.2开始，您不能将 [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)指定为[事务](https://docs.mongodb.com/manual/core/transactions/)中的第一个操作。

  *Changed in version 4.4*.

  The following operations are not allowed in transactions:

  - Operations that affect the database catalog, such as creating or dropping a collection or an index when using [feature compatibility version (fcv)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.2"` or lower. With fcv `"4.4"` or greater, you can create collections and indexes in transactions unless the transaction is a cross-shard write transaction. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes).
  - Creating new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.
  - , e.g. [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) method, and indexes, e.g.[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) and [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) methods, when using a read concern level other than [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-).
  - The [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections) and [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes) commands and their helper methods.
  - Other non-CRUD and non-informational operations, such as [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count), etc. and their helpers.

  Transactions have a lifetime limit as specified by [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds). The default is 60 seconds.

  *4.4版本中有更新*

  以下操作在事务中不被允许：

  - 影响数据库目录的操作，例如在使用[fcv](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)**`"4.2"`**或更低版本时创建/删除集合或索引。使用fcv**`"4.4"`**或更高版本时，您可以在事务中创建集合和索引，除非该事务是跨分片写入事务。有关详细信息，请参考[在事务中创建集合和索引](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)。
  - 在跨分片写入事务中创建新集合。例如，如果您在一个分片中写入现有集合，而在另一个分片中隐式创建一个集合，则MongoDB无法在同一事务中执行这两项操作。
  - 当使用除[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)以外的其他读关注级别时[显示创建集合](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)，如 [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)方法；以及显示创建索引，如[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) 和 [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) 方法。
  - [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections) 和 [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes)命令及其辅助方法。
  - 其他非CRUD和非信息性操作，例如[`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count)等及其辅助方法。

  事务存在一个生命周期限制，由[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)指定，默认值为60s。

- `Write Command Batch Limit Size` **批量写大小限制**

  `100,000` [writes](https://docs.mongodb.com/manual/reference/command/nav-crud/) are allowed in a single batch operation, defined by a single request to the server.

  *Changed in version 3.6*: The limit raises from `1,000` to `100,000` writes. This limit also applies to legacy `OP_INSERT` messages.

  The [`Bulk()`](https://docs.mongodb.com/manual/reference/method/Bulk/#mongodb-method-Bulk) operations in the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell and comparable methods in the drivers do not have this limit.

  在单个批处理操作中允许`100,000`次[写入](https://docs.mongodb.com/manual/reference/command/nav-crud/)，这由对服务器的单个请求定义。

  *在3.6版中进行了更改*：写入限制从`1,000`增加到`100,000`。此限制也适用于旧式`OP_INSERT`消息。

  [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell中的[`Bulk()`](https://docs.mongodb.com/manual/reference/method/Bulk/#mongodb-method-Bulk) 操作和驱动程序中的类似方法没有此限制。

- `Views` **视图**

  The view definition `pipeline` cannot include the [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) or the [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) stage. If the view definition includes nested pipeline (e.g. the view definition includes [`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup) or [`$facet`](https://docs.mongodb.com/manual/reference/operator/aggregation/facet/#mongodb-pipeline-pipe.-facet) stage), this restriction applies to the nested pipelines as well.

  Views have the following operation restrictions:

  - Views are read-only.
  - You cannot rename [views](https://docs.mongodb.com/manual/core/views/).
  -  operations on views do not support the following [projection](https://docs.mongodb.com/manual/reference/operator/projection/) operators:
    - [`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)
    - [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch)
    - [`$slice`](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)
    - [`$meta`](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta)
  - [Views](https://docs.mongodb.com/manual/core/views/) do not support text search.
  - [Views](https://docs.mongodb.com/manual/core/views/) do not support map-reduce operations.
  - [Views](https://docs.mongodb.com/manual/core/views/) do not support geoNear operations (i.e. [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear) pipeline stage).

  视图定义`管道`不能包含 [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) 或者 [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) 阶段。如果视图定义包括嵌套管道（例如，视图定义包括[`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup) 或者[`$facet`](https://docs.mongodb.com/manual/reference/operator/aggregation/facet/#mongodb-pipeline-pipe.-facet) 阶段），则此限制也适用于嵌套管道。

  视图具有以下操作限制：

  - 只读
  - 不能重命名
  - 不支持以下[投射](https://docs.mongodb.com/manual/reference/operator/projection/)操作符：[$](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)、[$elemMatch](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch)、[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)、[$meta](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta)
  - 不支持文本索引
  - 不支持map-reduce操作
  - 不支持geoNear操作（即[$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)管道阶段）

- `Projection Restrictions` **投射限制**

  *New in version 4.4* 4.4版的新功能:

  **`$`-Prefixed Field Path Restriction** **$前缀的字段路径限制**

  Starting in MongoDB 4.4, the [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) and [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) projection cannot project a field that starts with `$` with the exception of the [DBRef fields](https://docs.mongodb.com/manual/reference/database-references/#std-label-dbref-explanation).For example, starting in MongoDB 4.4, the following operation is invalid:

  从MongoDB 4.4开始， [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) 无法投射以`$`开头的字段，但[DBRef字段](https://docs.mongodb.com/manual/reference/database-references/#std-label-dbref-explanation)除外。例如，从MongoDB 4.4开始，以下操作无效：

  ```js
  db.inventory.find( {}, { "$instock.warehouse": 0, "$item": 0, "detail.$price": 1 } ) // Invalid starting in 4.4
  ```

  MongoDB already has a [`restriction`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names) where top-level field names cannot start with the dollar sign (`$`).In earlier version, MongoDB ignores the `$`-prefixed field projections.

  MongoDB已经有一个[限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names)，即顶级字段名称不能以美元符号（`$`）开头。在早期版本中，MongoDB忽略`$`前缀的字段投射。

  **`$` Positional Operator Placement Restriction** **$位置运算符的放置限制**

  Starting in MongoDB 4.4, the [`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-) projection operator can only appear at the end of the field path; e.g. `"field.$"` or `"fieldA.fieldB.$"`.For example, starting in MongoDB 4.4, the following operation is invalid:

  从MongoDB 4.4开始，[`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)投射运算符只能出现在字段路径的末尾。例如`"field.$"`或`"fieldA.fieldB.$"`。例如，从MongoDB 4.4开始，以下操作无效： 

  ```js
  db.inventory.find( { }, { "instock.$.qty": 1 } ) // Invalid starting in 4.4
  ```

  To resolve, remove the component of the field path that follows the [`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-) projection operator.In previous versions, MongoDB ignores the part of the path that follows the `$`; i.e. the projection is treated as `"instock.$"`.

  要解决此问题，请删除`$`投射运算符后面的字段路径部分。在以前的版本中，MongoDB会忽略`$`后面的路径部分； 即，该投射被视为`"instock.$"`。

  **Empty Field Name Projection Restriction** **空字段名称投射限制**

  Starting in MongoDB 4.4, [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) and [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) projection cannot include a projection of an empty field name.For example, starting in MongoDB 4.4, the following operation is invalid:

  从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify)不能包含空字段名称的投射。例如，从MongoDB 4.4开始，以下操作无效： 

  ```js
  db.inventory.find( { }, { "": 0 } ) // Invalid starting in 4.4
  ```

  In previous versions, MongoDB treats the inclusion/exclusion of the empty field as it would the projection of non-existing fields.

  在以前的版本中，MongoDB将空字段的包含/排除视为不存在字段的投射。

  **Path Collision: Embedded Documents and Its Fields** **路径冲突：嵌入式文档及其字段**

  Starting in MongoDB 4.4, it is illegal to project an embedded document with any of the embedded document's fields.For example, consider a collection `inventory` with documents that contain a `size`field:

  从MongoDB 4.4开始，使用嵌入文档的任何字段来投射嵌入文档都是非法的，例如，考虑包含文档的集合`inventory`，其中包含`size`字段： 

  ```js
  { ..., size: { h: 10, w: 15.25, uom: "cm" }, ... }
  ```

  Starting in MongoDB 4.4, the following operation fails with a `Path collision` error because it attempts to project both `size` document and the `size.uom` field:

  从MongoDB 4.4开始，以下操作因`路径冲突`错误而失败，因为它尝试同时投射`size`文档和`size.uom`字段： 

  ```js
  db.inventory.find( {}, { size: 1, "size.uom": 1 } )  // Invalid starting in 4.4
  ```

  In previous versions, lattermost projection between the embedded documents and its fields determines the projection:

  - If the projection of the embedded document comes after any and all projections of its fields, MongoDB projects the embedded document. For example, the projection document `{ "size.uom": 1, size: 1 }` produces the same result as the projection document `{ size: 1 }`.
  - If the projection of the embedded document comes before the projection any of its fields, MongoDB projects the specified field or fields. For example, the projection document `{ "size.uom": 1, size: 1, "size.h": 1 }` produces the same result as the projection document `{"size.uom": 1, "size.h": 1 }`.

  在以前的版本中，嵌入文档及其字段之间的最后一个投射决定了整个投射：

  - 如果嵌入式文档的投射紧随其字段的所有投射之后，则MongoDB会投射嵌入式文档。例如，投射文档`{"size.uom"：1, size：1}`产生与投射文档`{size：1}`相同的结果。
  - 如果嵌入式文档的投射先于其任何字段的投射，则MongoDB会投射指定的一个或多个字段。例如，投射文档`{"size.uom"：1, size：1,"size.h"：1}`产生与投射文档`{"size.uom"：1, "size.h":1}`相同的结果。

  **Path Collision: `$slice` of an Array and Embedded Fields** **路径冲突：数组和嵌入式字段的`$slice`**

  Starting in MongoDB 4.4, [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) and [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) projection cannot contain both a [`$slice`](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)of an array and a field embedded in the array.For example, consider a collection `inventory` that contains an array field `instock`:

  从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify)投射不能同时包含数组的[$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)和数组中嵌入的字段，例如，考虑包含数组字段`instock`的集合`inventory`： 

  ```js
  { ..., instock: [ { warehouse: "A", qty: 35 }, { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ], ... }
  ```

  Starting in MongoDB 4.4, the following operation fails with a `Path collision` error:

  从MongoDB 4.4开始，以下操作会因`路径冲突`而失败：

  ```js
  db.inventory.find( {}, { "instock": { $slice: 1 }, "instock.warehouse": 0 } ) // Invalid starting in 4.4
  ```

  In previous versions, the projection applies both projections and returns the first element (`$slice: 1`) in the `instock` array but suppresses the `warehouse` field in the projected element. Starting in MongoDB 4.4, to achieve the same result, use the [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate) method with two separate[`$project`](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project) stages.

  在以前的版本中，投射会同时应用这两个投射并返回`instock`数组中的第一个元素（`$slice: 1`），但会抑制投射元素中的`warehouse`字段。从MongoDB 4.4开始，要获得相同的结果，请使用带两个独立[`$project`](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project)阶段的[`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)方法。

  **`$` Positional Operator and `$slice` Restriction** **`$`位置运算符和`$slice`限制**

  Starting in MongoDB 4.4, [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) and [`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) projection cannot include [`$slice`](https://docs.mongodb.com/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)projection expression as part of a [`$`](https://docs.mongodb.com/manual/reference/operator/projection/positional/#mongodb-projection-proj.-) projection expression.For example, starting in MongoDB 4.4, the following operation is invalid:

  从MongoDB 4.4开始，[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)和[`findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) 投射不能包含`$slice`投射表达式作为`$`投射表达式的一部分。例如，从MongoDB 4.4开始，以下操作无效：

  ```js
  db.inventory.find( { "instock.qty": { $gt: 25 } }, { "instock.$": { $slice: 1 } } ) // Invalid starting in 4.4
  ```

  MongoDB already has a [`restriction`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names) where top-level field names cannot start with the dollar sign (`$`).In previous versions, MongoDB returns the first element (`instock.$`) in the `instock` array that matches the query condition; i.e. the positional projection `"instock.$"` takes precedence and the `$slice:1` is a no-op. The `"instock.$": { $slice: 1 }` does not exclude any other document field.

  MongoDB已经有一个[限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Restrictions-on-Field-Names)，即顶级字段名称不能以美元符号（`$`）开头。在以前的版本中，MongoDB返回`instock`数组中与查询条件匹配的第一个元素（`instock.$`）； 即位置投射`"instock.$"`优先，而`$slice：1`是空操作。`"instock.$"：{$slice：1}`不排除任何其他文档字段。

## Sessions 会话

- `Sessions and $external Username Limit`**会话和`$external`用户名限制**

  *Changed in version 3.6.3*: To use sessions with `$external` authentication users (i.e. Kerberos, LDAP, x.509 users), the usernames cannot be greater than 10k bytes.

  *在版本3.6.3中更改*：要与`$external`身份验证用户（即Kerberos，LDAP，x.509用户）一起使用会话，用户名不能大于10KB。

- `Session Idle Timeout` **会话空闲超时**

  Sessions that receive no read or write operations for 30 minutes *or* that are not refreshed using [`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) within this threshold are marked as expired and can be closed by the MongoDB server at any time. Closing a session kills any in-progress operations and open cursors associated with the session. This includes cursors configured with [`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout) or a [`maxTimeMS()`](https://docs.mongodb.com/manual/reference/method/cursor.maxTimeMS/#mongodb-method-cursor.maxTimeMS) greater than 30 minutes.
  
  Consider an application that issues a [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find). The server returns a cursor along with a batch of documents defined by the [`cursor.batchSize()`](https://docs.mongodb.com/manual/reference/method/cursor.batchSize/#mongodb-method-cursor.batchSize) of the [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find). The session refreshes each time the application requests a new batch of documents from the server. However, if the application takes longer than 30 minutes to process the current batch of documents, the session is marked as expired and closed. When the application requests the next batch of documents, the server returns an error as the cursor was killed when the session was closed.
  
  在30分钟内未执行任何读或写操作或未使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) 刷新的会话在此阈值之内被标记为已过期，并且MongoDB服务器可以随时将其关闭。关闭会话将终止所有正在进行的操作以及与该会话关联的已打开游标。这包括使用[`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout) 或 [`maxTimeMS()`](https://docs.mongodb.com/manual/reference/method/cursor.maxTimeMS/#mongodb-method-cursor.maxTimeMS) 大于30分钟配置的游标。
  
   考虑一个发出[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)命令的应用程序。服务器返回一个游标以及由[`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)的 [`cursor.batchSize()`](https://docs.mongodb.com/manual/reference/method/cursor.batchSize/#mongodb-method-cursor.batchSize)定义的一批文档。每次应用程序从服务器请求新一批文档时，会话都会刷新。但是，如果应用程序花费超过30分钟的时间来处理当前批次的文档，则该会话将被标记为已过期并关闭。当应用程序请求下一批文档时，服务器将返回错误，因为在关闭会话时游标已被杀死。
  
  For operations that return a cursor, if the cursor may be idle for longer than 30 minutes, issue the operation within an explicit session using [`Mongo.startSession()`](https://docs.mongodb.com/manual/reference/method/Mongo.startSession/#mongodb-method-Mongo.startSession) and periodically refresh the session using the [`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) command. For example:
  
  ```js
  var session = db.getMongo().startSession()
  var sessionId = session.getSessionId().id
  
  var cursor = session.getDatabase("examples").getCollection("data").find().noCursorTimeout()
  var refreshTimestamp = new Date() // take note of time at operation start
  
  while (cursor.hasNext()) {
  
    // Check if more than 5 minutes have passed since the last refresh
    if ( (new Date()-refreshTimestamp)/1000 > 300 ) {
      print("refreshing session")
      db.adminCommand({"refreshSessions" : [sessionId]})
      refreshTimestamp = new Date()
    }
  
    // process cursor normally
  
  }
  ```
  
  In the example operation, the [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) method is associated with an explicit session. The cursor is configured with [`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout) to prevent the server from closing the cursor if idle. The `while` loop includes a block that uses [`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) to refresh the session every 5 minutes. Since the session will never exceed the 30 minute idle timeout, the cursor can remain open indefinitely.
  
  For MongoDB drivers, defer to the [driver documentation](https://docs.mongodb.com/drivers/) for instructions and syntax for creating sessions.
  
  对于返回游标的操作，如果游标可能闲置30分钟以上，请使用[`Mongo.startSession()`](https://docs.mongodb.com/manual/reference/method/Mongo.startSession/#mongodb-method-Mongo.startSession) 在显式会话中发出该操作，并使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions) 命令定期刷新该会话。例如： 
  
  ```js
  var session = db.getMongo().startSession()
  var sessionId = session.getSessionId().id
  
  var cursor = session.getDatabase("examples").getCollection("data").find().noCursorTimeout()
  var refreshTimestamp = new Date() // 记录操作开始的时间
  
  while (cursor.hasNext()) {
  
    // 检查距离上一次刷新是否已经过去了5分钟
    if ( (new Date()-refreshTimestamp)/1000 > 300 ) {
      print("refreshing session")
      db.adminCommand({"refreshSessions" : [sessionId]})
      refreshTimestamp = new Date()
    }
  
    // 正常地处理游标
  
  }
  ```
  
  在示例操作中，[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)方法与显式会话相关联。游标使用[`noCursorTimeout()`](https://docs.mongodb.com/manual/reference/method/cursor.noCursorTimeout/#mongodb-method-cursor.noCursorTimeout)配置，以防止服务器在空闲时关闭游标。`while`循环包含一个代码块，使用[`refreshSessions`](https://docs.mongodb.com/manual/reference/command/refreshSessions/#mongodb-dbcommand-dbcmd.refreshSessions)每5分钟刷新一次会话。由于会话将永远不会超过30分钟的空闲超时，因此游标可以无限期保持打开状态。
  
  对于MongoDB驱动程序，请参考[驱动程序文档](https://docs.mongodb.com/drivers/)中有关创建会话的说明和语法。

## Shell 终端

The [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell prompt has a limit of 4095 codepoints for each line. If you enter a line with more than 4095 codepoints, the shell will truncate it.

[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)终端提示符每行的限制为4095个代码点。如果您输入的行中包含4095个以上的代码点，则将被截断。



------

译者：phoenix

时间： 2021.04.26

原文： https://docs.mongodb.com/manual/reference/limits/

版本： 4.4
