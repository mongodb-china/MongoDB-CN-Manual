
# GridFS


[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)是用于存储和检索超过16 MB[大小限制](https://docs.mongodb.com/manual/reference/limits/#std-label-limit-bson-document-size)的[BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)文档文件的规范。


> 注意
>
> GridFS 不支持[多文档事务](https://docs.mongodb.com/manual/core/transactions/)

相较于将一个文件存储在单条文档中，GridFS将文件分为多个部分或块[[1]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation)，并将每个块存储为单独的文档。默认情况下，GridFS使用的块默认大小为255kB；也就是说，除最后一个块，GridFS会将文件划分为255 kB的块。最后一个块只有必要的大小。同样，最后的那个块也不会大于默认的块大小，仅使用所需的空间以及一些其他元数据。


GridFS使用两个集合来存储文件。一个集合存储文件块，另一个集合存储文件元数据。 [GridFS集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-collections)一节详细介绍了每个集合。

当你从GridFS查询文件时，驱动程序将根据需要重新组装该文件所有的块。你可以对GridFS存储的文件进行范围查询。你还可以从文件的任意部分访问其信息，例如“跳到”视频或音频文件的中间


GridFS不仅可用于存储超过16 MB的文件，而且还可用于存储您要访问的任何文件而不必将整个文件加载到内存中。另请参阅何时使用[GridFS](https://docs.mongodb.com/manual/core/gridfs/#std-label-faq-developers-when-to-use-gridfs)



## 什么时候使用GridFS

在MongoDB中，使用[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)存储大于16 MB的文件。


在某些情况下，在MongoDB数据库中存储大型文件可能比在系统级文件系统上存储效率更高。

- 如果文件系统限制了目录中文件的数量，则可以使用GridFS来存储所需数量的文件。

- 当你要访问大文件部分的信息而不必将整个文件加载到内存中时，可以使用GridFS来调用文件的某些部分，而无需将整个文件读入内存。

- 当你希望保持文件和元数据在多个系统和设施之间自动同步和部署时，可以使用GridFS。使用[地理分布的复制集](https://docs.mongodb.com/manual/core/replica-set-architecture-geographically-distributed/#std-label-replica-set-geographical-distribution)时，MongoDB可以自动将文件及其元数据分发到多个[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)实例和设施。


如果您需要对整个文件的内容进行原子更新，请不要使用GridFS。或者，您可以存储每个文件的多个版本，并在元数据中指定文件的当前版本。上传文件的新版本后，您可以原子更新元数据中指示为“最新”状态的字段，然后在需要时删除以前的版本。


此外，如果文件均小于16 MB [BSON文档大小](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-BSON-Document-Size)限制，请考虑将每个文件存储在单个文档中，而不是使用GridFS。您可以使用BinData数据类型存储二进制数据。有关使用BinData的详细信息，请参见[驱动](https://docs.mongodb.com/drivers/)程序文档。



## 使用GridFS

要使用[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)存储和检索文件，请使用以下任一方法：

- MongoDB驱动程序。请参阅[驱动程序](https://docs.mongodb.com/drivers/)文档，以获取有关将GridFS与驱动程序一起使用的信息。
- [mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)命令行工具。有关文档，请参见[mongofiles](https://docs.mongodb.com/database-tools/mongofiles/#mongodb-binary-bin.mongofiles)参考。

## GridFS Collections

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)将文件存储在两个集合中:

- `块`存储二进制块。有关详细信息，请参见[chunks集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-chunks-collection)。
- `文件`存储文件的元数据。有关详细信息，请参见[文件集合](https://docs.mongodb.com/manual/core/gridfs/#std-label-gridfs-files-collection)。


GridFS通过使用存储桶名称为每个集合添加前缀，将集合放置在一个公共存储桶中。默认情况下，GridFS使用两个集合以及一个名为fs的存储桶：

- `fs.files`
- `fs.chunks`


您可以选择其他存储桶名称，也可以在一个数据库中创建多个存储桶。完整集合名称（包括存储桶名称）受[命名空间长度限制](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Namespace-Length)。



### `块`集合

`块`[[1]](https://docs.mongodb.com/manual/core/gridfs/#footnote-chunk-disambiguation)集合中的每个文档都代表了GridFS中表示的文件的不同的块。此集合中的文档具有以下格式：

```
{
  "_id" : <ObjectId>,
  "files_id" : <ObjectId>,
  "n" : <num>,
  "data" : <binary>
}
```


`chunks`集合中的文档包含以下字段：

- `chunks._id`


  块的唯一[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)。

- `chunks.files_id`

  在`files`集合中指定的“父”文档的`_id`。

- `chunks.n`


  块的序列号。 GridFS从0开始对所有块进行编号。

- `chunks.data`
  
  块[BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)二进制类型的荷载。




### `文件` 集合

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


`files`集合中的文档包含以下一些或全部字段：

- `files._id`


  该文档的唯一标识符。 `_id`是您为原始文档选择的数据类型。 MongoDB文档的默认类型是[BSON ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON)。

- `files.length`


  文档的大小（以字节为单位）。

- `files.chunkSize`


  每个块的大小（以**字节**为单位）。 GridFS将文档分为大小为`chunkSize`的块，最后一个除外，后者仅根据需要而变大。默认大小为255 KB。

- `files.uploadDate`

  GridFS首次存储这个文档的日期。此值为有`日期`类型。

- `files.md5`

  **过期**


  FIPS 140-2禁止使用MD5算法。 MongoDB驱动程序已弃用MD5支持，并将在未来版本中删除MD5的生成。需要文件摘要的应用程序应在GridFS外部实现它，并将其存储在[files.metadata]()中。

  [filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/)命令返回的完整文件的MD5哈希。此值为字符串类型。

- `files.filename`

  可选的。GridFS文件的可读名称。

- `files.contentType`

  **过期**

  可选的。GridFS文件的有效MIME类型。仅应用程序用。

  使用[files.metadata](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata)来存储与GridFS文件的MIME类型有关的信息。

- `files.aliases`


  **过期**

  可选的。别名字符串数组。仅用于应用程序

  使用[files.metadata](https://docs.mongodb.com/manual/core/gridfs/#mongodb-data-files.metadata)来存储与GridFS文件的MIME类型有关的信息。

- `files.metadata`
  
  可选的。元数据字段可以是任何数据类型，并且可以保存您要存储的任何其他信息。如果希望将其他任意字段添加到文件集合中的文档，请将其添加到元数据字段中的对象。


## GridFS索引


GridFS使用每个块和文件集合上的索引来提高效率。为了方便起见，符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)会自动创建这些索引。您还可以根据需要创建任何其他索引，以满足您的应用程序需求。



### `chunks`索引

[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)使用`files_id`和`n`字段在`chunks`集合上使用[唯一](https://docs.mongodb.com/manual/reference/glossary/#std-term-unique-index)的[复合](https://docs.mongodb.com/manual/reference/glossary/#std-term-compound-index)索引。可以有效地检索块，如以下示例所示：

```
db.fs.chunks.find( { files_id: myFileID } ).sort( { n: 1 } )
```


符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)将在读取和写入操作之前自动确保此索引存在。有关GridFS应用程序的特定行为，请参阅相关的驱动程序文档。


如果该索引不存在，则可以执行以下操作以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell创建它：

```
db.fs.chunks.createIndex( { files_id: 1, n: 1 }, { unique: true } );
```



### `files` 索引


[GridFS](https://docs.mongodb.com/manual/reference/glossary/#std-term-GridFS)在`files`集合上的`filename`和`uploadDate`字段上使用[索引](https://docs.mongodb.com/manual/reference/glossary/#std-term-index)。该索引允许高效地检索文件，如本示例所示：

```
db.fs.files.find( { filename: myFileName } ).sort( { uploadDate: 1 } )
```

符合[GridFS规范](https://github.com/mongodb/specifications/blob/master/source/gridfs/gridfs-spec.rst)的[驱动程序](https://docs.mongodb.com/drivers/)将在读取和写入操作之前自动确保此索引存在。有关GridFS应用程序的特定行为，请参阅相关的驱动程序文档。


如果该索引不存在，则可以执行以下操作以使用[mongo](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell创建它：

```
db.fs.files.createIndex( { filename: 1, uploadDate: 1 } );
```



| [1]  | *([1](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id1), [2](https://docs.mongodb.com/manual/core/gridfs/#ref-chunk-disambiguation-id3))* 在GridFS上下文中使用术语块与在分片上下文中使用术语块无关。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



## 分片GridFS

GridFS考虑两个集合- `files`和 `chunks`。


### `chunks`集合


要分片`chunks`集合，请使用`{ files_id : 1, n : 1 }` 或`{ files_id : 1 }` 作为分片键索引。 `files_id`是一个[ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId)，并且[单调](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-shard-key-monotonic)更改。


对于不运行[filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5)来验证成功上传的MongoDB驱动程序（例如，支持MongoDB 4.0或更高版本的MongoDB驱动程序），可以将[哈希分片](https://docs.mongodb.com/manual/core/hashed-sharding/)用于`chunks`集合。


如果MongoDB驱动程序运行[filemd5](https://docs.mongodb.com/manual/reference/command/filemd5/#mongodb-dbcommand-dbcmd.filemd5)，则不能使用[Hashed Sharding](https://docs.mongodb.com/manual/core/hashed-sharding/)。有关详细信息，请参阅[SERVER-9888](https://jira.mongodb.org/browse/SERVER-9888)。


### `files` 集合


`files` 集合很小，仅包含元数据。 GridFS所需的所有密钥都不适合在分片环境中进行平均分配。保留未分片的`files`允许所有文件元数据文档保留在主分片上。


如果必须分片 `files`集合，请使用`_id`字段，可能与应用程序字段结合使用。
