# MongoDB Limits and Thresholds

On this page

- [BSON Documents](https://docs.mongodb.com/master/reference/limits/#bson-documents)
- [Naming Restrictions](https://docs.mongodb.com/master/reference/limits/#naming-restrictions)
- [Namespaces](https://docs.mongodb.com/master/reference/limits/#namespaces)
- [Indexes](https://docs.mongodb.com/master/reference/limits/#indexes)
- [Data](https://docs.mongodb.com/master/reference/limits/#data)
- [Replica Sets](https://docs.mongodb.com/master/reference/limits/#replica-sets)
- [Sharded Clusters](https://docs.mongodb.com/master/reference/limits/#sharded-clusters)
- [Operations](https://docs.mongodb.com/master/reference/limits/#operations)
- [Sessions](https://docs.mongodb.com/master/reference/limits/#sessions)
- [Shell](https://docs.mongodb.com/master/reference/limits/#shell)

This document provides a collection of hard and soft limitations of the MongoDB system.

## BSON Documents



- `BSON Document Size`

  The maximum BSON document size is 16 megabytes.The maximum document size helps ensure that a single document cannot use excessive amount of RAM or, during transmission, excessive amount of bandwidth. To store documents larger than the maximum size, MongoDB provides the GridFS API. See [`mongofiles`](https://docs.mongodb.com/database-tools/mongofiles/#bin.mongofiles) and the documentation for your [driver](https://docs.mongodb.com/ecosystem/drivers) for more information about GridFS.



- `Nested Depth for BSON Documents`

  MongoDB supports no more than 100 levels of nesting for [BSON documents](https://docs.mongodb.com/master/reference/glossary/#term-document).



## Naming Restrictions

- `Database Name Case Sensitivity`

  Since database names are case *insensitive* in MongoDB, database names cannot differ only by the case of the characters.

- `Restrictions on Database Names for Windows`

  For MongoDB deployments running on Windows, database names cannot contain any of the following characters:copycopied`/\. "$*<>:|? `Also database names cannot contain the null character.

- `Restrictions on Database Names for Unix and Linux Systems`

  For MongoDB deployments running on Unix and Linux systems, database names cannot contain any of the following characters:copycopied`/\. "$ `Also database names cannot contain the null character.

- `Length of Database Names`

  Database names cannot be empty and must have fewer than 64 characters.

- `Restriction on Collection Names`

  Collection names should begin with an underscore or a letter character, and *cannot*:contain the `$`.be an empty string (e.g. `""`).contain the null character.begin with the `system.` prefix. (Reserved for internal use.)If your collection name includes special characters, such as the underscore character, or begins with numbers, then to access the collection use the [`db.getCollection()`](https://docs.mongodb.com/master/reference/method/db.getCollection/#db.getCollection) method in the [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell or a [similar method for your driver](https://docs.mongodb.com/drivers/).Namespace Length:For [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) set to `"4.4"` or greater, MongoDB raises the limit on collection/view namespace to 255 bytes. For a collection or a view, the namespace includes the database name, the dot (`.`) separator, and the collection/view name (e.g. `<database>.<collection>`),For [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) set to `"4.2"` or earlier, the maximum length of the collection/view namespace remains 120 bytes.



- `Restrictions on Field Names`

  Field names **cannot** contain the `null` character.Top-level field names **cannot** start with the dollar sign (`$`) character.Otherwise, starting in MongoDB 3.6, the server permits storage of field names that contain dots (i.e. `.`) and dollar signs (i.e. `$`).IMPORTANTThe MongoDB Query Language cannot always meaningfully express queries over documents whose field names contain these characters (see [SERVER-30575](https://jira.mongodb.org/browse/SERVER-30575)).Until support is added in the query language, the use of `$` and `.` in field names is not recommended and is not supported by the official MongoDB drivers.MONGODB DOES NOT SUPPORT DUPLICATE FIELD NAMESThe MongoDB Query Language is undefined over documents with duplicate field names. BSON builders may support creating a BSON document with duplicate field names. While the BSON builder may not throw an error, inserting these documents into MongoDB is not supported *even if* the insert succeeds. For example, inserting a BSON document with duplicate field names through a MongoDB driver may result in the driver silently dropping the duplicate values prior to insertion.



## Namespaces



- `Namespace Length`

  For [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) set to `"4.4"` or greater, MongoDB raises the limit on collection/view namespace to 255 bytes. For a collection or a view, the namespace includes the database name, the dot (`.`) separator, and the collection/view name (e.g. `<database>.<collection>`),For [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) set to `"4.2"` or earlier, the maximum length of the collection/view namespace remains 120 bytes.SEE ALSO[Naming Restrictions](https://docs.mongodb.com/master/reference/limits/#faq-restrictions-on-collection-names)



## Indexes



- `Index Key Limit`

  CHANGED IN VERSION 4.2Starting in version 4.2, MongoDB removes the `Index Key Limit` for [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) (fCV) set to `"4.2"` or greater.For MongoDB 2.6 through MongoDB versions with fCV set to `"4.0"` or earlier, the *total size* of an index entry, which can include structural overhead depending on the BSON type, must be *less than* 1024 bytes.When the [`Index Key Limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit) applies:MongoDB will **not** create an index on a collection if the index entry for an existing document exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit).Reindexing operations will error if the index entry for an indexed field exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit). Reindexing operations occur as part of the [`compact`](https://docs.mongodb.com/master/reference/command/compact/#dbcmd.compact) command as well as the [`db.collection.reIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.reIndex/#db.collection.reIndex) method.Because these operations drop *all* the indexes from a collection and then recreate them sequentially, the error from the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit) prevents these operations from rebuilding any remaining indexes for the collection.MongoDB will not insert into an indexed collection any document with an indexed field whose corresponding index entry would exceed the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit), and instead, will return an error. Previous versions of MongoDB would insert but not index such documents.Updates to the indexed field will error if the updated value causes the index entry to exceed the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit).If an existing document contains an indexed field whose index entry exceeds the limit, *any* update that results in the relocation of that document on disk will error.[`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore) and [`mongoimport`](https://docs.mongodb.com/database-tools/mongoimport/#bin.mongoimport) will not insert documents that contain an indexed field whose corresponding index entry would exceed the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit).In MongoDB 2.6, secondary members of replica sets will continue to replicate documents with an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit) on initial sync but will print warnings in the logs.Secondary members also allow index build and rebuild operations on a collection that contains an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit) but with warnings in the logs.With *mixed version* replica sets where the secondaries are version 2.6 and the primary is version 2.4, secondaries will replicate documents inserted or updated on the 2.4 primary, but will print error messages in the log if the documents contain an indexed field whose corresponding index entry exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit).For existing sharded collections, [chunk migration](https://docs.mongodb.com/master/core/sharding-balancer-administration/) will fail if the chunk has a document that contains an indexed field whose index entry exceeds the [`index key limit`](https://docs.mongodb.com/master/reference/limits/#Index-Key-Limit).



- `Number of Indexes per Collection`

  A single collection can have *no more* than 64 indexes.



- `Index Name Length`

  CHANGED IN VERSION 4.2Starting in version 4.2, MongoDB removes the `Index Name Length Limit` for MongoDB versions with [featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) (fCV) set to `"4.2"` or greater.In previous versions of MongoDB or MongoDB versions with fCV set to `"4.0"` or earlier, fully qualified index names, which include the namespace and the dot separators (i.e. `<database name>.<collection name>.$<index name>`), cannot be longer than 127 bytes.By default, `<index name>` is the concatenation of the field names and index type. You can explicitly specify the `<index name>` to the [`createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex) method to ensure that the fully qualified index name does not exceed the limit.

- `Number of Indexed Fields in a Compound Index`

  There can be no more than 32 fields in a compound index.

- `Queries cannot use both text and Geospatial Indexes`

  You cannot combine the [`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) query, which requires a special [text index](https://docs.mongodb.com/master/core/index-text/#create-text-index), with a query operator that requires a different type of special index. For example you cannot combine [`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) query with the [`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near) operator.

- `Fields with 2dsphere Indexes can only hold Geometries`

  Fields with [2dsphere](https://docs.mongodb.com/master/core/2dsphere/) indexes must hold geometry data in the form of [coordinate pairs](https://docs.mongodb.com/master/reference/glossary/#term-legacy-coordinate-pairs) or [GeoJSON](https://docs.mongodb.com/master/reference/glossary/#term-geojson) data. If you attempt to insert a document with non-geometry data in a `2dsphere` indexed field, or build a `2dsphere` index on a collection where the indexed field has non-geometry data, the operation will fail.

SEE ALSO

The unique indexes limit in [Sharding Operational Restrictions](https://docs.mongodb.com/master/reference/limits/#limits-sharding-operations).

- `NaN values returned from Covered Queries by the WiredTiger Storage Engine are always of type double`

  If the value of a field returned from a query that is [covered by an index](https://docs.mongodb.com/master/core/query-optimization/#covered-queries) is `NaN`, the type of that `NaN` value is *always* `double`.

- `Multikey Index`

  [Multikey indexes](https://docs.mongodb.com/master/core/index-multikey/#index-type-multikey) cannot cover queries over array field(s).

- `Geospatial Index`

  [Geospatial indexes](https://docs.mongodb.com/master/geospatial-queries/#index-feature-geospatial) cannot [cover a query](https://docs.mongodb.com/master/core/query-optimization/#covered-queries).

- `Memory Usage in Index Builds`

  [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) supports building one or more indexes on a collection. [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) uses a combination of memory and temporary files on disk to complete index builds. The default limit on memory usage for [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) is 200 megabytes (for versions 4.2.3 and later) and 500 (for versions 4.2.2 and earlier), shared between all indexes built using a single [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) command. Once the memory limit is reached, [`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes) uses temporary disk files in a subdirectory named `_tmp` within the [`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-dbpath) directory to complete the build.You can override the memory limit by setting the [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/master/reference/parameters/#param.maxIndexBuildMemoryUsageMegabytes) server parameter. Setting a higher memory limit may result in faster completion of index builds. However, setting this limit too high relative to the unused RAM on your system can result in memory exhaustion and server shutdown.*Changed in version 4.2.*For [feature compatibility version (fcv)](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.2"`, the index build memory limit applies to all index builds.For [feature compatibility version (fcv)](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.0"`, the index build memory limit only applies to foreground index builds.Index builds may be initiated either by a user command such as [Create Index](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/) or by an administrative process such as an [initial sync](https://docs.mongodb.com/master/core/replica-set-sync/). Both are subject to the limit set by [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/master/reference/parameters/#param.maxIndexBuildMemoryUsageMegabytes).An [initial sync operation](https://docs.mongodb.com/master/core/replica-set-sync/) populates only one collection at a time and has no risk of exceeding the memory limit. However, it is possible for a user to start index builds on multiple collections in multiple databases simultaneously and potentially consume an amount of memory greater than the limit set in [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/master/reference/parameters/#param.maxIndexBuildMemoryUsageMegabytes).TIPTo minimize the impact of building an index on replica sets and sharded clusters with replica set shards, use a rolling index build procedure as described on [Rolling Index Builds on Replica Sets](https://docs.mongodb.com/master/tutorial/build-indexes-on-replica-sets/).

- `Collation and Index Types`

  The following index types only support simple binary comparison and do not support [collation](https://docs.mongodb.com/master/reference/bson-type-comparison-order/#collation):[text](https://docs.mongodb.com/master/core/index-text/) indexes,[2d](https://docs.mongodb.com/master/core/2d/) indexes, and[geoHaystack](https://docs.mongodb.com/master/core/geohaystack/) indexes.TIPTo create a `text`, a `2d`, or a `geoHaystack` index on a collection that has a non-simple collation, you must explicitly specify `{collation: {locale: "simple"} }` when creating the index.

- `Hidden Indexes`

  You cannot [hide](https://docs.mongodb.com/master/core/index-hidden/) the `_id` index.You cannot use [`hint()`](https://docs.mongodb.com/master/reference/method/cursor.hint/#cursor.hint) on a [hidden index](https://docs.mongodb.com/master/core/index-hidden/).

## Data

- `Maximum Number of Documents in a Capped Collection`

  If you specify a maximum number of documents for a capped collection using the `max` parameter to [`create`](https://docs.mongodb.com/master/reference/command/create/#dbcmd.create), the limit must be less than 232 documents. If you do not specify a maximum number of documents when creating a capped collection, there is no limit on the number of documents.

## Replica Sets

- `Number of Members of a Replica Set`

  Replica sets can have up to 50 members.

- `Number of Voting Members of a Replica Set`

  Replica sets can have up to 7 voting members. For replica sets with more than 7 total members, see [Non-Voting Members](https://docs.mongodb.com/master/core/replica-set-elections/#replica-set-non-voting-members).

- `Maximum Size of Auto-Created Oplog`

  If you do not explicitly specify an oplog size (i.e. with [`oplogSizeMB`](https://docs.mongodb.com/master/reference/configuration-options/#replication.oplogSizeMB) or [`--oplogSize`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-oplogsize)) MongoDB will create an oplog that is no larger than 50 gigabytes. [[1\]](https://docs.mongodb.com/master/reference/limits/#oplog)[[1\]](https://docs.mongodb.com/master/reference/limits/#id1)Starting in MongoDB 4.0, the oplog can grow past its configured size limit to avoid deleting the [`majority commit point`](https://docs.mongodb.com/master/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime).



## Sharded Clusters

Sharded clusters have the restrictions and thresholds described here.



### Sharding Operational Restrictions

- `Operations Unavailable in Sharded Environments`

  [`$where`](https://docs.mongodb.com/master/reference/operator/query/where/#op._S_where) does not permit references to the `db` object from the [`$where`](https://docs.mongodb.com/master/reference/operator/query/where/#op._S_where) function. This is uncommon in un-sharded collections.The [`geoSearch`](https://docs.mongodb.com/master/reference/command/geoSearch/#dbcmd.geoSearch) command is not supported in sharded environments.

- `Covered Queries in Sharded Clusters`

  Starting in MongoDB 3.0, an index cannot [cover](https://docs.mongodb.com/master/core/query-optimization/#covered-queries) a query on a [sharded](https://docs.mongodb.com/master/reference/glossary/#term-shard) collection when run against a [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) if the index does not contain the shard key, with the following exception for the `_id` index: If a query on a sharded collection only specifies a condition on the `_id` field and returns only the `_id` field, the `_id` index can cover the query when run against a [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) even if the `_id` field is not the shard key.In previous versions, an index cannot [cover](https://docs.mongodb.com/master/core/query-optimization/#covered-queries) a query on a [sharded](https://docs.mongodb.com/master/reference/glossary/#term-shard) collection when run against a [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos).

- `Sharding Existing Collection Data Size`

  An existing collection can only be sharded if its size does not exceed specific limits. These limits can be estimated based on the average size of all [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) values, and the configured [chunk](https://docs.mongodb.com/master/reference/glossary/#term-chunk) size.IMPORTANTThese limits only apply for the initial sharding operation. Sharded collections can grow to *any* size after successfully enabling sharding.Use the following formulas to calculate the *theoretical* maximum collection size.copycopied`maxSplits = 16777216 (bytes) / <average size of shard key values in bytes> maxCollectionSize (MB) = maxSplits * (chunkSize / 2) `NOTEThe maximum [BSON](https://docs.mongodb.com/master/reference/glossary/#term-bson) document size is 16MB or `16777216` bytes.All conversions should use base-2 scale, e.g. 1024 kilobytes = 1 megabyte.If `maxCollectionSize` is less than or nearly equal to the target collection, increase the chunk size to ensure successful initial sharding. If there is doubt as to whether the result of the calculation is too ‘close’ to the target collection size, it is likely better to increase the chunk size.After successful initial sharding, you can reduce the chunk size as needed. If you later reduce the chunk size, it may take time for all chunks to split to the new size. See [Modify Chunk Size in a Sharded Cluster](https://docs.mongodb.com/master/tutorial/modify-chunk-size-in-sharded-cluster/) for instructions on modifying chunk size.This table illustrates the approximate maximum collection sizes using the formulas described above:Average Size of Shard Key Values512 bytes256 bytes128 bytes64 bytesMaximum Number of Splits32,76865,536131,072262,144Max Collection Size (64 MB Chunk Size)1 TB2 TB4 TB8 TBMax Collection Size (128 MB Chunk Size)2 TB4 TB8 TB16 TBMax Collection Size (256 MB Chunk Size)4 TB8 TB16 TB32 TB

- `Single Document Modification Operations in Sharded Collections`

  All [`update()`](https://docs.mongodb.com/master/reference/method/db.collection.update/#db.collection.update) and [`remove()`](https://docs.mongodb.com/master/reference/method/db.collection.remove/#db.collection.remove) operations for a sharded collection that specify the `justOne` or `multi: false` option must include the [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) *or* the `_id` field in the query specification. [`update()`](https://docs.mongodb.com/master/reference/method/db.collection.update/#db.collection.update) and [`remove()`](https://docs.mongodb.com/master/reference/method/db.collection.remove/#db.collection.remove) operations specifying `justOne` or `multi: false` in a sharded collection which do not contain either the [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) or the `_id` field return an error.



- `Unique Indexes in Sharded Collections`

  MongoDB does not support unique indexes across shards, except when the unique index contains the full shard key as a prefix of the index. In these situations MongoDB will enforce uniqueness across the full key, not a single field.SEE[Unique Constraints on Arbitrary Fields](https://docs.mongodb.com/master/tutorial/unique-constraints-on-arbitrary-fields/#shard-key-arbitrary-uniqueness) for an alternate approach.



- `Maximum Number of Documents Per Chunk to Migrate`

  By default, MongoDB cannot move a chunk if the number of documents in the chunk is greater than 1.3 times the result of dividing the configured [chunk size](https://docs.mongodb.com/master/core/sharding-data-partitioning/#sharding-chunk-size) by the average document size. [`db.collection.stats()`](https://docs.mongodb.com/master/reference/method/db.collection.stats/#db.collection.stats) includes the `avgObjSize` field, which represents the average document size in the collection.For chunks that are [too large to migrate](https://docs.mongodb.com/master/core/sharding-balancer-administration/#migration-chunk-size-limit), starting in MongoDB 4.4:A new balancer setting `attemptToBalanceJumboChunks` allows the balancer to migrate chunks too large to move as long as the chunks are not labeled [jumbo](https://docs.mongodb.com/master/core/sharding-data-partitioning/#jumbo-chunk). See [Balance Chunks that Exceed Size Limit](https://docs.mongodb.com/master/tutorial/manage-sharded-cluster-balancer/#balance-chunks-that-exceed-size-limit) for details.The [`moveChunk`](https://docs.mongodb.com/master/reference/command/moveChunk/#dbcmd.moveChunk) command can specify a new option [forceJumbo](https://docs.mongodb.com/master/reference/command/moveChunk/#movechunk-forcejumbo) to allow for the migration of chunks that are too large to move. The chunks may or may not be labeled [jumbo](https://docs.mongodb.com/master/core/sharding-data-partitioning/#jumbo-chunk).



### Shard Key Limitations

- `Shard Key Size`

  Starting in version 4.4, MongoDB removes the limit on the shard key size.For MongoDB 4.2 and earlier, a shard key cannot exceed 512 bytes.

- `Shard Key Index Type`

  A [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) index can be an ascending index on the shard key, a compound index that start with the shard key and specify ascending order for the shard key, or a [hashed index](https://docs.mongodb.com/master/core/index-hashed/).A [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) index cannot be an index that specifies a [multikey index](https://docs.mongodb.com/master/core/index-multikey/), a [text index](https://docs.mongodb.com/master/core/index-text/) or a [geospatial index](https://docs.mongodb.com/master/geospatial-queries/#index-feature-geospatial) on the [shard key](https://docs.mongodb.com/master/reference/glossary/#term-shard-key) fields.

- `Shard Key Selection is Immutable in MongoDB 4.``2 and Earlier`

  CHANGED IN VERSION 4.4Starting in MongoDB 4.4, you can refine a collection’s shard key by adding a suffix field or fields to the existing key. See [`refineCollectionShardKey`](https://docs.mongodb.com/master/reference/command/refineCollectionShardKey/#dbcmd.refineCollectionShardKey).In MongoDB 4.2 and earlier, once you shard a collection, the selection of the shard key is immutable; i.e. you cannot select a different shard key for that collection.If you must change a shard key:Dump all data from MongoDB into an external format.Drop the original sharded collection.Configure sharding using the new shard key.[Pre-split](https://docs.mongodb.com/master/tutorial/create-chunks-in-sharded-cluster/) the shard key range to ensure initial even distribution.Restore the dumped data into MongoDB.

- `Monotonically Increasing Shard Keys Can Limit Insert Throughput`

  For clusters with high insert volumes, a shard keys with monotonically increasing and decreasing keys can affect insert throughput. If your shard key is the `_id` field, be aware that the default values of the `_id` fields are [ObjectIds](https://docs.mongodb.com/master/reference/glossary/#term-objectid) which have generally increasing values.When inserting documents with monotonically increasing shard keys, all inserts belong to the same [chunk](https://docs.mongodb.com/master/reference/glossary/#term-chunk) on a single [shard](https://docs.mongodb.com/master/reference/glossary/#term-shard). The system eventually divides the chunk range that receives all write operations and migrates its contents to distribute data more evenly. However, at any moment the cluster directs insert operations only to a single shard, which creates an insert throughput bottleneck.If the operations on the cluster are predominately read operations and updates, this limitation may not affect the cluster.To avoid this constraint, use a [hashed shard key](https://docs.mongodb.com/master/core/hashed-sharding/#sharding-hashed-sharding) or select a field that does not increase or decrease monotonically.[Hashed shard keys](https://docs.mongodb.com/master/core/hashed-sharding/#sharding-hashed-sharding) and [hashed indexes](https://docs.mongodb.com/master/core/index-hashed/#index-type-hashed) store hashes of keys with ascending values.

## Operations



- `Sort Operations`

  If MongoDB cannot use an index or indexes to obtain the sort order, MongoDB must perform a blocking sort operation on the data. The name refers to the requirement that the `SORT` stage reads all input documents before returning any output documents, blocking the flow of data for that specific query.If MongoDB requires using more than 100 megabytes of system memory for the blocking sort operation, MongoDB returns an error *unless* the query specifies [`cursor.allowDiskUse()`](https://docs.mongodb.com/master/reference/method/cursor.allowDiskUse/#cursor.allowDiskUse) (*New in MongoDB 4.4*). [`allowDiskUse()`](https://docs.mongodb.com/master/reference/method/cursor.allowDiskUse/#cursor.allowDiskUse) allows MongoDB to use temporary files on disk to store data exceeding the 100 megabyte system memory limit while processing a blocking sort operation.*Changed in version 4.4:* For MongoDB 4.2 and prior, blocking sort operations could not exceed 32 megabytes of system memory.For more information on sorts and index use, see [Sort and Index Use](https://docs.mongodb.com/master/reference/method/cursor.sort/#sort-index-use).



- `Aggregation Pipeline Operation`

  Pipeline stages have a limit of 100 megabytes of RAM. If a stage exceeds this limit, MongoDB will produce an error. To allow for the handling of large datasets, use the `allowDiskUse` option to enable aggregation pipeline stages to write data to temporary files.*Changed in version 3.4.*The [`$graphLookup`](https://docs.mongodb.com/master/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup) stage must stay within the 100 megabyte memory limit. If `allowDiskUse: true` is specified for the [`aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate) operation, the [`$graphLookup`](https://docs.mongodb.com/master/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup) stage ignores the option. If there are other stages in the [`aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate) operation, `allowDiskUse: true` option is in effect for these other stages.Starting in MongoDB 4.2, the [profiler log messages](https://docs.mongodb.com/master/tutorial/manage-the-database-profiler/) and [diagnostic log messages](https://docs.mongodb.com/master/reference/log-messages/) includes a `usedDisk` indicator if any aggregation stage wrote data to temporary files due to [memory restrictions](https://docs.mongodb.com/master/core/aggregation-pipeline-limits/#agg-memory-restrictions).SEE ALSO[$sort and Memory Restrictions](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#sort-memory-limit) and [$group Operator and Memory](https://docs.mongodb.com/master/reference/operator/aggregation/group/#group-memory-limit).

- `Aggregation and Read Concern`

  Starting in MongoDB 4.2, the [`$out`](https://docs.mongodb.com/master/reference/operator/aggregation/out/#pipe._S_out) stage cannot be used in conjunction with read concern [`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable"). That is, if you specify [`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable") read concern for [`db.collection.aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate), you cannot include the [`$out`](https://docs.mongodb.com/master/reference/operator/aggregation/out/#pipe._S_out) stage in the pipeline.The [`$merge`](https://docs.mongodb.com/master/reference/operator/aggregation/merge/#pipe._S_merge) stage cannot be used in conjunction with read concern [`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable"). That is, if you specify [`"linearizable"`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable") read concern for [`db.collection.aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate), you cannot include the [`$merge`](https://docs.mongodb.com/master/reference/operator/aggregation/merge/#pipe._S_merge) stage in the pipeline.

- `2d Geospatial queries cannot use the $or operator`

  SEE[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or) and [2d Index Internals](https://docs.mongodb.com/master/core/geospatial-indexes/).

- `Geospatial Queries`

  For spherical queries, use the `2dsphere` index result.The use of `2d` index for spherical queries may lead to incorrect results, such as the use of the `2d` index for spherical queries that wrap around the poles.

- `Geospatial Coordinates`

  Valid longitude values are between `-180` and `180`, both inclusive.Valid latitude values are between `-90` and `90`, both inclusive.

- `Area of GeoJSON Polygons`

  For [`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin), if you specify a single-ringed polygon that has an area greater than a single hemisphere, include [`the custom MongoDB coordinate reference system in the $geometry`](https://docs.mongodb.com/master/reference/operator/query/geometry/#op._S_geometry) expression; otherwise, [`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin) queries for the complementary geometry. For all other GeoJSON polygons with areas greater than a hemisphere, [`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects) or [`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin) queries for the complementary geometry.

- `Multi-document Transactions`

  For [multi-document transactions](https://docs.mongodb.com/master/core/transactions/):You can specify read/write (CRUD) operations on **existing** collections. For a list of CRUD operations, see [CRUD Operations](https://docs.mongodb.com/master/core/transactions-operations/#transactions-operations-crud).When using [feature compatibility version (fcv)](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.4"` or greater, you can create collections and indexes in transactions. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/master/core/transactions/#transactions-create-collections-indexes)The collections used in a transaction can be in different databases.NOTEYou cannot create new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.You cannot write to [capped](https://docs.mongodb.com/master/core/capped-collections/) collections. (Starting in MongoDB 4.2)You cannot read/write to collections in the `config`, `admin`, or `local` databases.You cannot write to `system.*` collections.You cannot return the supported operation’s query plan (i.e. `explain`).For cursors created outside of a transaction, you cannot call [`getMore`](https://docs.mongodb.com/master/reference/command/getMore/#dbcmd.getMore) inside the transaction.For cursors created in a transaction, you cannot call [`getMore`](https://docs.mongodb.com/master/reference/command/getMore/#dbcmd.getMore) outside the transaction.Starting in MongoDB 4.2, you cannot specify [`killCursors`](https://docs.mongodb.com/master/reference/command/killCursors/#dbcmd.killCursors) as the first operation in a [transaction](https://docs.mongodb.com/master/core/transactions/).*Changed in version 4.4.*The following operations are not allowed in transactions:Operations that affect the database catalog, such as creating or dropping a collection or an index when using [feature compatibility version (fcv)](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) `"4.2"` or lower. With fcv `"4.4"` or greater, you can create collections and indexes in transactions unless the transaction is a cross-shard write transaction. For details, see [Create Collections and Indexes In a Transaction](https://docs.mongodb.com/master/core/transactions/#transactions-create-collections-indexes).Creating new collections in cross-shard write transactions. For example, if you write to an existing collection in one shard and implicitly create a collection in a different shard, MongoDB cannot perform both operations in the same transaction.[Explicit creation of collections](https://docs.mongodb.com/master/core/transactions-operations/#transactions-operations-ddl-explicit), e.g. [`db.createCollection()`](https://docs.mongodb.com/master/reference/method/db.createCollection/#db.createCollection) method, and indexes, e.g. [`db.collection.createIndexes()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndexes/#db.collection.createIndexes) and [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex) methods, when using a read concern level other than [`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local").The [`listCollections`](https://docs.mongodb.com/master/reference/command/listCollections/#dbcmd.listCollections) and [`listIndexes`](https://docs.mongodb.com/master/reference/command/listIndexes/#dbcmd.listIndexes) commands and their helper methods.Other non-CRUD and non-informational operations, such as [`createUser`](https://docs.mongodb.com/master/reference/command/createUser/#dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/master/reference/command/getParameter/#dbcmd.getParameter), [`count`](https://docs.mongodb.com/master/reference/command/count/#dbcmd.count), etc. and their helpers.Transactions have a lifetime limit as specified by [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/master/reference/parameters/#param.transactionLifetimeLimitSeconds). The default is 60 seconds.

- `Write Command Batch Limit Size`

  `100,000` [writes](https://docs.mongodb.com/master/reference/command/nav-crud/) are allowed in a single batch operation, defined by a single request to the server.*Changed in version 3.6:* The limit raises from `1,000` to `100,000` writes. This limit also applies to legacy `OP_INSERT` messages.The [`Bulk()`](https://docs.mongodb.com/master/reference/method/Bulk/#Bulk) operations in the [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and comparable methods in the drivers do not have this limit.

- `Views`

  The view definition `pipeline` cannot include the [`$out`](https://docs.mongodb.com/master/reference/operator/aggregation/out/#pipe._S_out) or the [`$merge`](https://docs.mongodb.com/master/reference/operator/aggregation/merge/#pipe._S_merge) stage. If the view definition includes nested pipeline (e.g. the view definition includes [`$lookup`](https://docs.mongodb.com/master/reference/operator/aggregation/lookup/#pipe._S_lookup) or [`$facet`](https://docs.mongodb.com/master/reference/operator/aggregation/facet/#pipe._S_facet) stage), this restriction applies to the nested pipelines as well.Views have the following operation restrictions:Views are read-only.You cannot rename [views](https://docs.mongodb.com/master/core/views/).[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) operations on views do not support the following [projection](https://docs.mongodb.com/master/reference/operator/projection/) operators:[`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_)[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/projection/elemMatch/#proj._S_elemMatch)[`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice)[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta)[Views](https://docs.mongodb.com/master/core/views/) do not support text search.[Views](https://docs.mongodb.com/master/core/views/) do not support map-reduce operations.[Views](https://docs.mongodb.com/master/core/views/) do not support geoNear operations (i.e. [`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear) pipeline stage).

- `Projection Restrictions`

  *New in version 4.4:*`$`-Prefixed Field Path RestrictionStarting in MongoDB 4.4, the [`find`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) and [`findAndModify`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify) projection cannot project a field that starts with `$` with the exception of the [DBRef fields](https://docs.mongodb.com/master/reference/database-references/#dbref-explanation).For example, starting in MongoDB 4.4, the following operation is invalid:`db.inventory.find( {}, { "$instock.warehouse": 0, "$item": 0, "detail.$price": 1 } ) // Invalid starting in 4.4 `MongoDB already has a [`restriction`](https://docs.mongodb.com/master/reference/limits/#Restrictions-on-Field-Names) where top-level field names cannot start with the dollar sign (`$`).In earlier version, MongoDB ignores the `$`-prefixed field projections.`$` Positional Operator Placement RestrictionStarting in MongoDB 4.4, the [`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_) projection operator can only appear at the end of the field path; e.g. `"field.$"` or `"fieldA.fieldB.$"`.For example, starting in MongoDB 4.4, the following operation is invalid:`db.inventory.find( { }, { "instock.$.qty": 1 } ) // Invalid starting in 4.4 `To resolve, remove the component of the field path that follows the [`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_) projection operator.In previous versions, MongoDB ignores the part of the path that follows the `$`; i.e. the projection is treated as `"instock.$"`.Empty Field Name Projection RestrictionStarting in MongoDB 4.4, [`find`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) and [`findAndModify`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify) projection cannot include a projection of an empty field name.For example, starting in MongoDB 4.4, the following operation is invalid:`db.inventory.find( { }, { "": 0 } ) // Invalid starting in 4.4 `In previous versions, MongoDB treats the inclusion/exclusion of the empty field as it would the projection of non-existing fields.Path Collision: Embedded Documents and Its FieldsStarting in MongoDB 4.4, it is illegal to project an embedded document with any of the embedded document’s fields.For example, consider a collection `inventory` with documents that contain a `size` field:`{ ..., size: { h: 10, w: 15.25, uom: "cm" }, ... } `Starting in MongoDB 4.4, the following operation fails with a `Path collision` error because it attempts to project both `size` document and the `size.uom` field:`db.inventory.find( {}, { size: 1, "size.uom": 1 } )  // Invalid starting in 4.4 `In previous versions, lattermost projection between the embedded documents and its fields determines the projection:If the projection of the embedded document comes after any and all projections of its fields, MongoDB projects the embedded document. For example, the projection document `{ "size.uom": 1, size: 1 }` produces the same result as the projection document `{ size: 1 }`.If the projection of the embedded document comes before the projection any of its fields, MongoDB projects the specified field or fields. For example, the projection document `{ "size.uom": 1, size: 1, "size.h": 1 }` produces the same result as the projection document `{ "size.uom": 1, "size.h": 1 }`.Path Collision: `$slice` of an Array and Embedded FieldsStarting in MongoDB 4.4, [`find`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) and [`findAndModify`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify) projection cannot contain both a [`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice) of an array and a field embedded in the array.For example, consider a collection `inventory` that contains an array field `instock`:`{ ..., instock: [ { warehouse: "A", qty: 35 }, { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ], ... } `Starting in MongoDB 4.4, the following operation fails with a `Path collision` error:`db.inventory.find( {}, { "instock": { $slice: 1 }, "instock.warehouse": 0 } ) // Invalid starting in 4.4 `In previous versions, the projection applies both projections and returns the first element (`$slice: 1`) in the `instock` array but suppresses the `warehouse` field in the projected element. Starting in MongoDB 4.4, to achieve the same result, use the [`db.collection.aggregate()`](https://docs.mongodb.com/master/reference/method/db.collection.aggregate/#db.collection.aggregate) method with two separate [`$project`](https://docs.mongodb.com/master/reference/operator/aggregation/project/#pipe._S_project) stages.`$` Positional Operator and `$slice` RestrictionStarting in MongoDB 4.4, [`find`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) and [`findAndModify`](https://docs.mongodb.com/master/reference/method/db.collection.findAndModify/#db.collection.findAndModify) projection cannot include [`$slice`](https://docs.mongodb.com/master/reference/operator/projection/slice/#proj._S_slice) projection expression as part of a [`$`](https://docs.mongodb.com/master/reference/operator/projection/positional/#proj._S_) projection expression.For example, starting in MongoDB 4.4, the following operation is invalid:`db.inventory.find( { "instock.qty": { $gt: 25 } }, { "instock.$": { $slice: 1 } } ) // Invalid starting in 4.4 `MongoDB already has a `restriction` where top-level field names cannot start with the dollar sign (`$`).In previous versions, MongoDB returns the first element (`instock.$`) in the `instock` array that matches the query condition; i.e. the positional projection `"instock.$"` takes precedence and the `$slice:1` is a no-op. The `"instock.$": { $slice: 1 }` does not exclude any other document field.

## Sessions

- `Sessions and $external Username Limit`

  *Changed in version 3.6.3:* To use sessions with `$external` authentication users (i.e. Kerberos, LDAP, x.509 users), the usernames cannot be greater than 10k bytes.

- `Session Idle Timeout`

  Sessions that receive no read or write operations for 30 minutes *or* that are not refreshed using [`refreshSessions`](https://docs.mongodb.com/master/reference/command/refreshSessions/#dbcmd.refreshSessions) within this threshold are marked as expired and can be closed by the MongoDB server at any time. Closing a session kills any in-progress operations and open cursors associated with the session. This includes cursors configured with [`noCursorTimeout`](https://docs.mongodb.com/master/reference/method/cursor.noCursorTimeout/#cursor.noCursorTimeout) or a [`maxTimeMS`](https://docs.mongodb.com/master/reference/method/cursor.maxTimeMS/#cursor.maxTimeMS) greater than 30 minutes.Consider an application that issues a [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find). The server returns a cursor along with a batch of documents defined by the [`cursor.batchSize()`](https://docs.mongodb.com/master/reference/method/cursor.batchSize/#cursor.batchSize) of the [`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find). The session refreshes each time the application requests a new batch of documents from the server. However, if the application takes longer than 30 minutes to process the current batch of documents, the session is marked as expired and closed. When the application requests the next batch of documents, the server returns an error as the cursor was killed when the session was closed.For operations that return a cursor, if the cursor may be idle for longer than 30 minutes, issue the operation within an explicit session using `Session.startSession()` and periodically refresh the session using the [`refreshSessions`](https://docs.mongodb.com/master/reference/command/refreshSessions/#dbcmd.refreshSessions) command. For example:copycopied`var session = db.getMongo().startSession() var sessionId = session.getSessionId().id var cursor = session.getDatabase("examples").getCollection("data").find().noCursorTimeout() var refreshTimestamp = new Date() // take note of time at operation start while (cursor.hasNext()) {   // Check if more than 5 minutes have passed since the last refresh  if ( (new Date()-refreshTimestamp)/1000 > 300 ) {    print("refreshing session")    db.adminCommand({"refreshSessions" : [sessionId]})    refreshTimestamp = new Date()  }   // process cursor normally } `In the example operation, the [`db.collection.find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) method is associated with an explicit session. The cursor is configured with [`noCursorTimeout()`](https://docs.mongodb.com/master/reference/method/cursor.noCursorTimeout/#cursor.noCursorTimeout) to prevent the server from closing the cursor if idle. The `while` loop includes a block that uses [`refreshSessions`](https://docs.mongodb.com/master/reference/command/refreshSessions/#dbcmd.refreshSessions) to refresh the session every 5 minutes. Since the session will never exceed the 30 minute idle timeout, the cursor can remain open indefinitely.For MongoDB drivers, defer to the [driver documentation](https://docs.mongodb.com/ecosystem/drivers) for instructions and syntax for creating sessions.

## Shell

The [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell prompt has a limit of 4095 codepoints for each line. If you enter a line with more than 4095 codepoints, the shell will truncate it.