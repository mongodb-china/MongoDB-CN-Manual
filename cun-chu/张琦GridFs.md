# GridFS

# GridFS

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) is a specification for storing and retrieving files that exceed the [BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)-document [size limit](https://docs.mongodb.com/manual/reference/limits/#std-label-limit-bson-document-size) of 16 MB.

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)是用于存储和检索超过16 MB[大小限制](https://docs.mongodb.com/manual/reference/limits/#std-label-limit-bson-document-size)的[BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)文档文件的规范。

> NOTE
>
> GridFS does not support [multi-document transactions](https://docs.mongodb.com/manual/core/transactions/).

> 注意
>
> GridFS 不支持[多文档事务](https://docs.mongodb.com/manual/core/transactions/)

Instead of storing a file in a single document, GridFS divides the file into parts, or chunks [[1\]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation), and stores each chunk as a separate document. By default, GridFS uses a default chunk size of 255 kB; that is, GridFS divides a file into chunks of 255 kB with the exception of the last chunk. The last chunk is only as large as necessary. Similarly, files that are no larger than the chunk size only have a final chunk, using only as much space as needed plus some additional metadata.

相较于将一个文件存储在单条文档中，GridFS将文件分为多个部分或块[[1]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation)，并将每个块存储为单独的文档。默认情况下，GridFS使用的块默认大小为255kB；也就是说，除最后一个块，GridFS会将文件划分为255 kB的块。最后一个块只有必要的大小。同样，最后的那个块也不会大于默认的块大小，仅使用所需的空间以及一些其他元数据。

GridFS uses two collections to store files. One collection stores the file chunks, and the other stores file metadata. The section [GridFS Collections](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-collections) describes each collection in detail.

GridFS使用两个集合来存储文件。一个集合存储文件块，另一个集合存储文件元数据。 [GridFS集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-collections)一节详细介绍了每个集合。

When you query GridFS for a file, the driver will reassemble the chunks as needed. You can perform range queries on files stored through GridFS. You can also access information from arbitrary sections of files, such as to "skip" to the middle of a video or audio file.

当你从GridFS查询文件时，驱动程序将根据需要重新组装该文件所有的块。你可以对GridFS存储的文件进行范围查询。你还可以从文件的任意部分访问其信息，例如“跳到”视频或音频文件的中间

GridFS is useful not only for storing files that exceed 16 MB but also for storing any files for which you want access without having to load the entire file into memory. See also [When to Use GridFS](https://docs.mongodb.com/manual/core/gridfs/#std-label-faq-developers-when-to-use-gridfs).

GridFS不仅可用于存储超过16 MB的文件，而且还可用于存储您要访问的任何文件而不必将整个文件加载到内存中。另请参阅何时使用[GridFS](https://docs.mongodb.com/manual/core/gridfs/#std-label-faq-developers-when-to-use-gridfs)



## When to Use GridFS

## 什么时候使用GridFS

In MongoDB, use [GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) for storing files larger than 16 MB.

在MongoDB中，使用[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)存储大于16 MB的文件。

In some situations, storing large files may be more efficient in a MongoDB database than on a system-level filesystem.

在某些情况下，在MongoDB数据库中存储大型文件可能比在系统级文件系统上存储效率更高。

- If your filesystem limits the number of files in a directory, you can use GridFS to store as many files as needed.
- 如果文件系统限制了目录中文件的数量，则可以使用GridFS来存储所需数量的文件。
- When you want to access information from portions of large files without having to load whole files into memory, you can use GridFS to recall sections of files without reading the entire file into memory.
- 当你要访问大文件部分的信息而不必将整个文件加载到内存中时，可以使用GridFS来调用文件的某些部分，而无需将整个文件读入内存。
- When you want to keep your files and metadata automatically synced and deployed across a number of systems and facilities, you can use GridFS. When using [geographically distributed replica sets](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/#std-label-replica-set-geographical-distribution), MongoDB can distribute files and their metadata automatically to a number of [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) instances and facilities.
- 当你希望保持文件和元数据在多个系统和设施之间自动同步和部署时，可以使用GridFS。使用[地理分布的复制集](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/#std-label-replica-set-geographical-distribution)时，MongoDB可以自动将文件及其元数据分发到多个[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)实例和设施。

Do not use GridFS if you need to update the content of the entire file atomically. As an alternative you can store multiple versions of each file and specify the current version of the file in the metadata. You can update the metadata field that indicates "latest" status in an atomic update after uploading the new version of the file, and later remove previous versions if needed.

如果您需要对整个文件的内容进行原子更新，请不要使用GridFS。或者，您可以存储每个文件的多个版本，并在元数据中指定文件的当前版本。上传文件的新版本后，您可以原子更新元数据中指示为“最新”状态的字段，然后在需要时删除以前的版本。

Furthermore, if your files are all smaller than the 16 MB [`BSON Document Size`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-BSON-Document-Size) limit, consider storing each file in a single document instead of using GridFS. You may use the BinData data type to store the binary data. See your [drivers](https://docs.mongodb.com/drivers/) documentation for details on using BinData.

此外，如果文件均小于16 MB [BSON文档大小](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-BSON-Document-Size)限制，请考虑将每个文件存储在单个文档中，而不是使用GridFS。您可以使用BinData数据类型存储二进制数据。有关使用BinData的详细信息，请参见[驱动](https://docs.mongodb.com/drivers/)程序文档。



## Use GridFS

## 使用GridFS

To store and retrieve files using [GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS), use either of the following:

- A MongoDB driver. See the [drivers](https://docs.mongodb.com/drivers/) documentation for information on using GridFS with your driver.
- The [`mongofiles`](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles) command-line tool. See the [`mongofiles`](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles) reference for documentation.

要使用[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)存储和检索文件，请使用以下任一方法：

- MongoDB驱动程序。请参阅[驱动程序](https://docs.mongodb.com/drivers/)文档，以获取有关将GridFS与驱动程序一起使用的信息。
- [mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)命令行工具。有关文档，请参见[mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)参考。

## GridFS Collections

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) stores files in two collections:

- `chunks` stores the binary chunks. For details, see [The `chunks` Collection](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-chunks-collection).
- `files` stores the file's metadata. For details, see [The `files` Collection](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-files-collection).

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)将文件存储在两个集合中:

- `块`存储二进制块。有关详细信息，请参见[chunks集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-chunks-collection)。
- `文件`存储文件的元数据。有关详细信息，请参见[文件集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-files-collection)。



GridFS places the collections in a common bucket by prefixing each with the bucket name. By default, GridFS uses two collections with a bucket named `fs`:

- `fs.files`
- `fs.chunks`

GridFS通过使用存储桶名称为每个集合添加前缀，将集合放置在一个公共存储桶中。默认情况下，GridFS使用两个集合以及一个名为fs的存储桶：

- `fs.files`
- `fs.chunks`



You can choose a different bucket name, as well as create multiple buckets in a single database. The full collection name, which includes the bucket name, is subject to the [`namespace length limit`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Namespace-Length).

您可以选择其他存储桶名称，也可以在一个数据库中创建多个存储桶。完整集合名称（包括存储桶名称）受[命名空间长度限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Namespace-Length)。



### The `chunks` Collection

### `块`集合

Each document in the `chunks` [[1\]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation) collection represents a distinct chunk of a file as represented in [GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS). Documents in this collection have the following form:

`块`[[1]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation)集合中的每个文档都代表了GridFS中表示的文件的不同的块。此集合中的文档具有以下格式：

```
{
  "_id" : <ObjectId>,
  "files_id" : <ObjectId>,
  "n" : <num>,
  "data" : <binary>
}
```

A document from the `chunks` collection contains the following fields:

`chunks`集合中的文档包含以下字段：

- `chunks._id`

  The unique [ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId) of the chunk.

  块的唯一[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)。

- `chunks.files_id`

  The `_id` of the "parent" document, as specified in the `files` collection.

  在`files`集合中指定的“父”文档的`_id`。

- `chunks.n`

  The sequence number of the chunk. GridFS numbers all chunks, starting with 0.

  块的序列号。 GridFS从0开始对所有块进行编号。

- `chunks.data`

  The chunk's payload as a [BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON) `Binary` type.
  
  块[BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)二进制类型的荷载。



### The `files` Collection

### `文件` 集合

Each document in the `files` collection represents a file in [GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS).

文件集合中的每个文档代表GridFS中的一个[文件](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)。

```
{
  "_id" : <ObjectId>,
  "length" : <num>,
  "chunkSize" : <num>,
  "uploadDate" : <timestamp>,
  "md5" : <hash>,
  "filename" : <string>,
  "contentType" : <string>,
  "aliases" : <string array>,
  "metadata" : <any>,
}
```

Documents in the `files` collection contain some or all of the following fields:

`files`集合中的文档包含以下一些或全部字段：

- `files._id`

  The unique identifier for this document. The `_id` is of the data type you chose for the original document. The default type for MongoDB documents is [BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON) [ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId).

  该文档的唯一标识符。 `_id`是您为原始文档选择的数据类型。 MongoDB文档的默认类型是[BSON ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)。

- `files.length`

  The size of the document in bytes.

  文档的大小（以字节为单位）。

- `files.chunkSize`

  The size of each chunk in **bytes**. GridFS divides the document into chunks of size `chunkSize`, except for the last, which is only as large as needed. The default size is 255 kilobytes (kB).

  每个块的大小（以**字节**为单位）。 GridFS将文档分为大小为`chunkSize`的块，最后一个除外，后者仅根据需要而变大。默认大小为255 KB。

- `files.uploadDate`

  The date the document was first stored by GridFS. This value has the `Date` type.

  GridFS首次存储这个文档的日期。此值为有`日期`类型。

- `files.md5`

  **Deprecated**

  **过期**

  The MD5 algorithm is prohibited by FIPS 140-2. MongoDB drivers deprecate MD5 support and will remove MD5 generation in future releases. Applications that require a file digest should implement it outside of GridFS and store in [`files.metadata`](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata).

  An MD5 hash of the complete file returned by the [filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/) command. This value has the `String` type.

  FIPS 140-2禁止使用MD5算法。 MongoDB驱动程序已弃用MD5支持，并将在未来版本中删除MD5的生成。需要文件摘要的应用程序应在GridFS外部实现它，并将其存储在[files.metadata]()中。

  [filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/)命令返回的完整文件的MD5哈希。此值为字符串类型。

- `files.filename`

  Optional. A human-readable name for the GridFS file.

  可选的。GridFS文件的可读名称。

- `files.contentType`

  **Deprecated**

  **过期**

  Optional. A valid MIME type for the GridFS file. For application use only.

  Use [`files.metadata`](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata) for storing information related to the MIME type of the GridFS file.

  可选的。GridFS文件的有效MIME类型。仅应用程序用。

  使用[files.metadata](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata)来存储与GridFS文件的MIME类型有关的信息。

- `files.aliases`

  **Deprecated**

  **过期**

  Optional. An array of alias strings. For application use only.

  Use [`files.metadata`](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata) for storing information related to the MIME type of the GridFS file.

  可选的。别名字符串数组。仅用于应用程序

  使用[files.metadata](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata)来存储与GridFS文件的MIME类型有关的信息。

- `files.metadata`

  Optional. The metadata field may be of any data type and can hold any additional information you want to store. If you wish to add additional arbitrary fields to documents in the `files` collection, add them to an object in the metadata field.
  
  可选的。元数据字段可以是任何数据类型，并且可以保存您要存储的任何其他信息。如果希望将其他任意字段添加到文件集合中的文档，请将其添加到元数据字段中的对象。



## GridFS Indexes

## GridFS索引

GridFS uses indexes on each of the `chunks` and `files` collections for efficiency. [Drivers](https://docs.mongodb.com/drivers/) that conform to the [GridFS specification](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst) automatically create these indexes for convenience. You can also create any additional indexes as desired to suit your application's needs.

GridFS使用每个块和文件集合上的索引来提高效率。为了方便起见，符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)会自动创建这些索引。您还可以根据需要创建任何其他索引，以满足您的应用程序需求。



### The `chunks` Index

### `chunks`索引

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) uses a [unique](https://docs.mongodb.com/manual/reference/glossary/#std-term-unique-index), [compound](https://docs.mongodb.com/manual/reference/glossary/#std-term-compound-index) index on the `chunks` collection using the `files_id` and `n` fields. This allows for efficient retrieval of chunks, as demonstrated in the following example:

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)使用`files_id`和`n`字段在`chunks`集合上使用[唯一](https://docs.mongodb.com/manual/reference/glossary/#std-term-unique-index)的[复合](https://docs.mongodb.com/manual/reference/glossary/#std-term-compound-index)索引。可以有效地检索块，如以下示例所示：

```
db.fs.chunks.find( { files_id: myFileID } ).sort( { n: 1 } )
```



[Drivers](https://docs.mongodb.com/drivers/) that conform to the [GridFS specification](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst) will automatically ensure that this index exists before read and write operations. See the relevant driver documentation for the specific behavior of your GridFS application.

符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)将在读取和写入操作之前自动确保此索引存在。有关GridFS应用程序的特定行为，请参阅相关的驱动程序文档。

If this index does not exist, you can issue the following operation to create it using the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell:

如果该索引不存在，则可以执行以下操作以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell创建它：

```
db.fs.chunks.createIndex( { files_id: 1, n: 1 }, { unique: true } );
```



### The `files` Index

### `files` 索引

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) uses an [index](https://docs.mongodb.com/manual/reference/glossary/#std-term-index) on the `files` collection using the `filename` and `uploadDate` fields. This index allows for efficient retrieval of files, as shown in this example:

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)在`files`集合上的`filename`和`uploadDate`字段上使用[索引](https://docs.mongodb.com/manual/reference/glossary/#std-term-index)。该索引允许高效地检索文件，如本示例所示：

```
db.fs.files.find( { filename: myFileName } ).sort( { uploadDate: 1 } )
```

[Drivers](https://docs.mongodb.com/drivers/) that conform to the [GridFS specification](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst) will automatically ensure that this index exists before read and write operations. See the relevant driver documentation for the specific behavior of your GridFS application.

符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)将在读取和写入操作之前自动确保此索引存在。有关GridFS应用程序的特定行为，请参阅相关的驱动程序文档。

If this index does not exist, you can issue the following operation to create it using the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell:

如果该索引不存在，则可以执行以下操作以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell创建它：

```
db.fs.files.createIndex( { filename: 1, uploadDate: 1 } );
```



| [1]  | *([1](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id1), [2](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id3))* The use of the term *chunks* in the context of GridFS is not related to the use of the term *chunks* in the context of sharding. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| [1]  | *([1](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id1), [2](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id3))* 在GridFS上下文中使用术语块与在分片上下文中使用术语块无关。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



## Sharding GridFS

## 分片GridFS

There are two collections to consider with [GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS) - `files` and `chunks`.

GridFS考虑两个集合- `files`和 `chunks`。

### `chunks` Collection

### `chunks`集合

To shard the `chunks` collection, use either `{ files_id : 1, n : 1 }` or `{ files_id : 1 }` as the shard key index. `files_id` is an [ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId) and changes [monotonically](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-shard-key-monotonic).

要分片`chunks`集合，请使用`{ files_id : 1, n : 1 }` 或`{ files_id : 1 }` 作为分片键索引。 `files_id`是一个[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)，并且[单调](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-shard-key-monotonic)更改。

For MongoDB drivers that do not run [`filemd5`](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5) to verify successful upload (for example, MongoDB drivers that support MongoDB 4.0 or greater), you can use [Hashed Sharding](https://docs.mongodb.com/manual/core/hashed-sharding/) for the `chunks` collection.

对于不运行[filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5)来验证成功上传的MongoDB驱动程序（例如，支持MongoDB 4.0或更高版本的MongoDB驱动程序），可以将[哈希分片](https://docs.mongodb.com/manual/core/hashed-sharding/)用于`chunks`集合。

If the MongoDB driver runs [`filemd5`](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5), you cannot use [Hashed Sharding](https://docs.mongodb.com/manual/core/hashed-sharding/). For details, see [SERVER-9888](https://jira.mongodb.org/browse/SERVER-9888).

如果MongoDB驱动程序运行[filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5)，则不能使用[Hashed Sharding](https://docs.mongodb.com/manual/core/hashed-sharding/)。有关详细信息，请参阅[SERVER-9888](https://jira.mongodb.org/browse/SERVER-9888)。

### `files` Collection

### `files` 集合

The `files` collection is small and only contains metadata. None of the required keys for GridFS lend themselves to an even distribution in a sharded environment. Leaving `files` unsharded allows all the file metadata documents to live on the [primary shard](https://docs.mongodb.com/manual/reference/glossary/#std-term-primary-shard).

`files` 集合很小，仅包含元数据。 GridFS所需的所有密钥都不适合在分片环境中进行平均分配。保留未分片的`files`允许所有文件元数据文档保留在主分片上。

If you *must* shard the `files` collection, use the `_id` field, possibly in combination with an application field.

如果必须分片 `files`集合，请使用`_id`字段，可能与应用程序字段结合使用。

