## MongoDB的Wire协议

**在本页面**

- [介绍](#介绍)
- [TCP / IP套接字](#套接字)
- [消息类型和格式](#消息)
- [标准消息头](#标准)
- [客户端请求消息](#客户端)
- [数据库响应消息](#响应)

### <span id="介绍">介绍</span>

MongoDB有线协议是一个简单的基于套接字的请求-响应风格的协议。客户端通过一个常规的TCP/IP套接字与数据库服务器通信。

### <span id="套接字">TCP / IP套接字</span>

客户机应该使用常规的TCP/IP套接字连接到数据库。没有连接握手。

#### Port

实例[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的默认端口号是27017。[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的xw端口号是可配置的，可能会有所不同。

#### 字节次序

MongoDB线路协议中的所有整数都使用`little-end`字节顺序:也就是说，最低有效字节优先。

### <span id="消息">消息类型和格式</span>

有两种类型的消息:客户机请求和数据库响应。

> 注意
>
> 该页面使用类似于C的`struct`形式描述消息结构。
>
> 本文档中使用的类型(` cstring `、` int32 `等)与[BSON规范](http://bsonspec.org/#/specification)中定义的类型相同。
>
> 为了表示重复，文档使用了来自[BSON规范](http://bsonspec.org/#/specification)的星号符号。例如，` int64* `表示可以将一个或多个指定类型依次写入套接字。
>
> 标准的消息标题类型为**MsgHeader**。整型常量用大写字母表示(例如:`0`表示整数值0)。

### <span id="标准">标准消息头</span>

通常，每个消息由一个标准消息头和特定于请求的数据组成。标准消息头的结构如下:

```powershell
struct MsgHeader {
    int32   messageLength; // total message size, including this
    int32   requestID;     // identifier for this message
    int32   responseTo;    // requestID from the original request
                           //   (used in responses from db)
    int32   opCode;        // request type - see table below for details
}
```

| 字段            | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `messageLength` | 消息的总大小，以字节为单位。这个总数包括保存消息长度的4个字节。 |
| `requestID`     | 客户端或数据库生成的唯一标识符，用于标识此消息。客户端生成的消息(如[OP_QUERY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ wire-op-query)和[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ # wire-op-get-more)),它将返回的“幻想”字段[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ # wire-op-reply)消息。客户端可以使用 **requestID** 和 **responseTo** 字段将查询响应与原始查询关联起来。 |
| `responseTo`    | 从数据库的消息这将是从客户机的[OP_QUERY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ wire-op-query)或[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ # wire-op-get-more)消息中提取的 **requestID**。客户端可以使用 **requestID** 和 **responseTo** 字段将查询响应与原始查询关联起来。 |
| `opCode`        | 类型的消息。详细信息请参见[请求操作码](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ #wp-request-opcodes)。 |

#### 请求操作码

> 注意
>
> 启动MongoDB 2.6和[`maxWireVersion`](https://docs.mongodb.com/master/reference/command/ismaster/# isMaster.maxWireVersion)`3 `,MongoDB使用[数据库命令](https://docs.mongodb.com/master/reference/command/ # collection-commands)[`插入`](https://docs.mongodb.com/master/reference/command/insert/ # dbcmd.insert),[`更新`](https://docs.mongodb.com/master/reference/command/update/ # dbcmd.update),和[`删除`](https://docs.mongodb.com/master/reference/command/delete/ dbcmd.delete)而不是`OP_INSERT`,`OP_UPDATE`和`OP_DELETE`公认写。大多数驱动程序继续对未确认的写操作使用操作码。
>
> 在4.2版本中，MongoDB删除了内部不赞成的**OP_COMMAND**和**OP_COMMANDREPLY**协议。

以下是支持的`操作码`:

| 操作码的名字      | 值   | 评论                                  |
| :---------------- | :--- | :------------------------------------ |
| `OP_REPLY`        | 1    | 响应客户端请求。`responseTo`被设置    |
| `OP_UPDATE`       | 2001 | 更新文档。                            |
| `OP_INSERT`       | 2002 | 插入新文档。                          |
| `RESERVED`        | 2003 | 以前用于OP_GET_BY_OID。               |
| `OP_QUERY`        | 2004 | 查询集合。                            |
| `OP_GET_MORE`     | 2005 | 从查询中获取更多数据。看到游标。      |
| `OP_DELETE`       | 2006 | 删除文件。                            |
| `OP_KILL_CURSORS` | 2007 | 通知数据库客户机已经使用完游标。      |
| `OP_MSG`          | 2013 | 使用MongoDB 3.6中引入的格式发送消息。 |

### <span id="客户端">客户端请求消息</span>

客户端可以发送除[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -reply)之外的所有操作码的请求消息。[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -reply)为数据库保留使用。

只有[OP_QUERY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/# wireop -query)和[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/# wireop -get-more)消息会导致数据库的响应。将不会发送任何其他消息的响应。

您可以使用getLastError命令确定消息是否成功。

#### OP_UPDATE

OP_UPDATE消息用于更新集合中的文档。OP_UPDATE消息的格式如下:

```powershell
struct OP_UPDATE {
    MsgHeader header;             // standard message header
    int32     ZERO;               // 0 - reserved for future use
    cstring   fullCollectionName; // "dbname.collectionname"
    int32     flags;              // bit vector. see below
    document  selector;           // the query to select the document
    document  update;             // specification of the update to perform
}
```

| 字段                 | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `header`             | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `ZERO`               | 整数值为0。留作将来使用。                                    |
| `fullCollectionName` | 完整的集合名称;即名称空间。完整的集合名称是数据库名称与集合名称的连接，使用a`.`为了连接。例如，对于数据库`foo`和集合` bar `，完整的集合名称是` foo.bar `。 |
| `flags`              | 位向量，用于指定操作的标志。位值对应如下:<br>`0`对应于插入。如果设置，没有找到匹配的文档，数据库将把提供的对象插入到集合中.<br>`1`对应于多数插入。如果设置，数据库将更新集合中所有匹配的对象。否则只更新第一个匹配的文档。<br>**2--31** 保留。必须设置为0。 |
| `selector`           | BSON文档，为选择要更新的文档指定查询。                       |
| `update`             | 指定要执行的更新的BSON文档。有关指定更新的信息，请参阅MongoDB手册中的[Update Operations](https://docs.mongodb.com/manual/applications/update)文档。 |

没有对OP_UPDATE消息的响应。

#### OP_INSERT

OP_INSERT消息用于将一个或多个文档插入到集合中。OP_INSERT消息的格式为

```powershell
struct {
    MsgHeader header;             // standard message header
    int32     flags;              // bit vector - see below
    cstring   fullCollectionName; // "dbname.collectionname"
    document* documents;          // one or more documents to insert into the collection
}
```

| 字段                 | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `header`             | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `flags`              | 位向量，用于指定操作的标志。位值对应如下:<br/>`0`对应于ContinueOnError。如果设置了该设置，那么如果大容量插入失败(例如由于id重复)，数据库将不会停止处理。 这使得大容量插入的行为类似于一系列单个插入，只是如果任何插入失败(而不仅仅是最后一个插入失败)将设置lastError。如果出现多个错误，getLastError只报告最近的错误。(新1.9.1)<br/>**1--31** 保留。必须设置为`0`. |
| `fullCollectionName` | 完整的集合名称;即名称空间。 完整的集合名称是数据库名称与集合名称的连接，使用a`.`为了连接. 例如, 对于数据库` foo `和集合` bar `，完整的集合名称是` foo.bar `。 |
| `documents`          | 要插入到集合中的一个或多个文档。如果有多个，则依次将它们写入套接字。 |

没有对OP_INSERT消息的响应。

#### OP_QUERY

OP_QUERY消息用于在数据库中查询集合中的文档。OP_QUERY消息的格式为:

```powershell
struct OP_QUERY {
    MsgHeader header;                 // standard message header
    int32     flags;                  // bit vector of query options.  See below for details.
    cstring   fullCollectionName ;    // "dbname.collectionname"
    int32     numberToSkip;           // number of documents to skip
    int32     numberToReturn;         // number of documents to return
                                      //  in the first OP_REPLY batch
    document  query;                  // query object.  See below for details.
  [ document  returnFieldsSelector; ] // Optional. Selector indicating the fields
                                      //  to return.  See below for details.
}
```

| 字段                   | Description                                                  |
| :--------------------- | :----------------------------------------------------------- |
| `header`               | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `flags`                | 位向量，用于指定操作标志。这些位值对应于以下内容：<br>`0`被预定了。必须设置为0。<br> `1`对应于TailableCursor。可拖尾表示检索到最后一个数据时光标未关闭。而是，光标标记最终对象的位置。如果收到更多数据，您可以稍后从其所在位置继续使用光标。像任何“潜在游标”一样，游标可能会在某个时候失效（CursorNotFound）–例如，如果删除了它所引用的最终对象。<br> `2`对应于SlaveOk。允许查询副本从属。通常，这些返回错误，但名称空间“ local”除外。<br> `3`对应于OplogReplay。从MongoDB 4.4开始，您无需指定此标志，因为优化是针对oplog上的合格查询自动进行的。有关更多信息，请参见 [oplogReplay](https://docs.mongodb.com/master/reference/command/find/#find-cmd-fields)。<br> `4`对应于NoCursorTimeout。服务器通常在闲置时间（10分钟）后使空闲游标超时，以防止过多使用内存。设置此选项可以防止这种情况。<br> `5`对应于AwaitData。与TailableCursor一起使用。如果我们在数据的末尾，请阻塞一会而不是不返回任何数据。超时后，我们照常返回。<br> `6`对应于排气。假设客户端将完全读取所有查询的数据，则将数据以多个“更多”包的形式完整传输。当您提取大量数据并知道要全部提取时，速度更快。注意：除非客户端关闭连接，否则不允许客户端不读取所有数据。<br> `7`对应于部分。如果某些分片发生故障，则从mongos获得部分结果（而不是引发错误）<br> `8`- `31`保留。必须设置为0。 |
| `fullCollectionName`   | 完整的集合名称;即名称空间。完整的集合名称是数据库名称与集合名称的连接，使用a`.`为了连接。例如，对于数据库`foo`和集合` bar `，完整的集合名称是` foo.bar `。 |
| `numberToSkip`         | 在返回查询结果时，设置要省略的文档数量—从结果数据集中的第一个文档开始。 |
| `numberToReturn`       | 限制第一个[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -reply)消息中查询的文档数量。但是，如果结果多于**numberToReturn**，数据库仍然会建立一个游标并返回**cursorID**给客户端。如果客户端驱动程序提供了` limit `功能(如SQL limit关键字)，那么由客户端驱动程序来确保返回给调用应用程序的文档数量不超过指定的数量。如果**numberToReturn**是**0**，db将使用默认的返回大小。如果数字是负数，那么数据库将返回该数字并关闭游标。无法获取该查询的进一步结果。如果**numberToReturn**是**1**，服务器将把它视为**-1**(自动关闭光标)。 |
| `query`                | 表示查询的BSON文档。查询将包含一个或多个元素，所有元素都必须匹配才能包含在结果集中的文档。可能的元素包括`$query `、` $orderby `、` $hint `和` $explain `。 |
| `returnFieldsSelector` | 可选的。限制返回文档中的字段的BSON文档。**returnFieldsSelector**包含一个或多个元素，每个元素是应该返回的字段的名称，以及整数值**1**。在JSON表示法中，限制为**a**、**b**和**c**的**returnFieldsSelector**将是:<br>`{ a : 1, b : 1, c : 1} ` |

数据库将用一个[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/# wireop -reply)消息响应一个OP_QUERY消息。

#### OP_GET_MORE

OP_GET_MORE消息用于在数据库中查询集合中的文档。OP_GET_MORE消息的格式为:

```powershell
struct {
    MsgHeader header;             // standard message header
    int32     ZERO;               // 0 - reserved for future use
    cstring   fullCollectionName; // "dbname.collectionname"
    int32     numberToReturn;     // number of documents to return
    int64     cursorID;           // cursorID from the OP_REPLY
}
```

| Field                | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `header`             | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `ZERO`               | 整数值为0。留作将来使用。                                    |
| `fullCollectionName` | 完整的集合名称;即名称空间。完整的集合名称是数据库名称与集合名称的连接，使用`a.`为了连接。例如，对于数据库`foo`和集合` bar `，完整的集合名称是` foo.bar `。 |
| `numberToReturn`     | 限制第一个[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -reply)消息中查询的文档数量。但是，如果结果多于**numberToReturn**，数据库仍然会建立一个游标并返回**cursorID**给客户端。如果客户端驱动程序提供了` limit `功能(如SQL limit关键字)，那么由客户端驱动程序来确保返回给调用应用程序的文档数量不超过指定的数量。如果**numberToReturn**是**0**，db将使用默认的返回大小。 |
| `cursorID`           | 在[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -reply)中的光标标识符。这必须是来自数据库的值。 |

数据库将用一个[OP_REPLY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/# wireop -reply)消息响应一个OP_GET_MORE消息。

#### OP_DELETE

OP_DELETE消息用于从集合中删除一个或多个文档。OP_DELETE消息的格式为:

```powershell
struct {
    MsgHeader header;             // standard message header
    int32     ZERO;               // 0 - reserved for future use
    cstring   fullCollectionName; // "dbname.collectionname"
    int32     flags;              // bit vector - see below for details.
    document  selector;           // query object.  See below for details.
}
```

| 字段                 | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `header`             | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `ZERO`               | 整数值为0。留作将来使用。                                    |
| `fullCollectionName` | 完整的集合名称;即名称空间。完整的集合名称是数据库名称与集合名称的连接，使用a`.`为了连接。例如，对于数据库`foo`和集合` bar `，完整的集合名称是`foo.bar `。 |
| `flags`              | 位向量，用于指定操作的标志。位值对应如下:<br />`0`对应于SingleRemove。如果设置，数据库将只删除集合中第一个匹配的文档。否则，所有匹配的文件将被删除。<br/>`1--31`保留。必须设置为0。 |
| `selector`           | 表示用于选择要删除的文档的查询的BSON文档。选择器将包含一个或多个元素，所有元素必须匹配才能从集合中删除文档。 |

没有对OP_DELETE消息的响应。

#### OP_KILL_CURSORS

OP_KILL_CURSORS消息用于关闭数据库中的活动游标。这对于确保在查询结束时回收数据库资源是必要的。OP_KILL_CURSORS消息的格式为:

```powershell
struct {
    MsgHeader header;            // standard message header
    int32     ZERO;              // 0 - reserved for future use
    int32     numberOfCursorIDs; // number of cursorIDs in message
    int64*    cursorIDs;         // sequence of cursorIDs to close
}
```

| 字段                | Description                                                  |
| :------------------ | :----------------------------------------------------------- |
| `header`            | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `ZERO`              | 整数值为0。留作将来使用。                                    |
| `numberOfCursorIDs` | 消息中游标id的数量。                                         |
| `cursorIDs`         | 要关闭的游标id的“数组”。如果有多个，则依次将它们写入套接字。 |

如果游标读取,直到耗尽(读,直到[OP_QUERY](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ wire-op-query)或[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ wire-op-get-more)返回0光标id),不需要杀死光标。

#### OP_MSG

*新版本MongoDB:* 3.6

`OP_MSG`是一种可扩展的消息格式，旨在包含其他操作码的功能。此操作码的格式如下:

```powershell
OP_MSG {
    MsgHeader header;          // standard message header
    uint32 flagBits;           // message flags
    Sections[] sections;       // data sections
    optional<uint32> checksum; // optional CRC-32C checksum
}
```

| 字段       | Description                                                  |
| :--------- | :----------------------------------------------------------- |
| `header`   | 标准消息头，如[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中所述。 |
| `flagBits` | 一个包含消息标志的整数位掩码，如[Flag Bits](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wiremsg -flags)中所述。 |
| `sections` | 消息主体部分，如[部分](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wiremsg -sections)所述。 |
| `checksum` | 一个可选的CRC-32C校验和，如[checksum](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wiremsg -checksum)中所述。 |

#### 标志位

整数`flagBits`是位掩码编码标志，用于修改 **OP_MSG** 的格式和行为。

前16位(0-15)是*必须的*，如果设置了未知位，解析器**一定会**出错。

最后16位(16-31)是*可选的*，解析器**一定会**忽略任何未知的设置位。代理和其他消息转代必须在转发消息之前清除任何未知的可选位。

| 位   | Name              | 请求 | Response | Description                                                  |
| :--- | :---------------- | :--- | :------- | :----------------------------------------------------------- |
| 0    | `checksumPresent` | ✓    | ✓        | 消息以包含CRC-32C校验和的4个字节结束。详细信息请参见[Checksum](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wiremsg - Checksum)。 |
| 1    | `moreToCome`      | ✓    | ✓        | 另一条消息将跟随此消息而不需要接收者采取进一步的操作。接收方在接收到**moreToCome**设置为0的消息之前，不能再发送另一条消息，因为发送可能会阻塞，导致死锁。带有**moreToCome**位集的请求将不会收到回复。对于设置了“exhaustAllowed”位的请求，应答将只有此设置。 |
| 16   | `exhaustAllowed`  | ✓    |          | 客户端已经准备好使用**moreToCome**位来多次响应此请求。除非请求设置了这个位，否则服务器将永远不会产生设置了**moreToCome**位的响应。<br>这确保了只有当请求者的网络层为它们准备好时，多个响应才会被发送。<br />重要:MongoDB 3.6会忽略这个标志，并以一条消息响应。 |

#### 部分

一个`OP_MSG`消息包含一个或多个部分。每个节以表示其类型的**kind**字节开始。`kind`字节之后的所有内容构成了节的有效负载。

下面是可用的部分类型。

##### Kind 0: 身体

主体部分被编码为一个**单个** [BSON对象](https://docs.mongodb.com/master/reference/bson-types/# BSON -types)。BSON对象中的大小也用作部分的大小。本节类是标准命令请求和应答体。

所有顶级字段**必须**有一个唯一的名称。

##### Kind 1: 文档顺序

| Type                      | Description                                                  |
| :------------------------ | :----------------------------------------------------------- |
| int32                     | 节的大小，以字节为单位。                                     |
| C String                  | 文档序列标识符。在当前的所有命令中，这个字段是它从主体部分替换的(可能是嵌套的)字段.<br />这个字段**不能**也存在于body部分中. |
| Zero or more BSON objects | 不使用分隔符对对象进行反向排序。<br/>每个对象被限制为服务器的**maxBSONObjectSize**。所有对象的组合并不局限于**maxBSONObjSize**。<br />文档序列在消耗完**size**字节后结束。<br />在转换为语言级对象时，解析器可以选择将这些对象作为数组合并到主体中，并将其作为序列标识符指定的路径。 |

#### Checksum

每个消息**可能**以`CRC-32C`Checksum结束，它涵盖了消息中除了Checksum本身之外的所有字节。

从MongoDB 4.2开始:

- 如果不使用TLS/SSL连接，[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例、[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例和[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell实例将使用Checksum交换消息。
- 如果使用TLS/SSL连接，[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 实例、 [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例和 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell实例将跳过Checksum。

如果显示带有Checksum的消息，驱动程序和较老的二进制文件将忽略Checksum。

Checksum的存在是由**checksumPresent**标志位来表示的。

### <span id="响应">数据库响应消息</span>

#### OP_REPLY

`OP_REPLY`消息由数据库发送，以响应一个[OP_QUERY](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -query)或[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb- wireprotocol/ # wireop -get-more)消息。OP_REPLY消息的格式为:

```powershell
struct {
    MsgHeader header;         // standard message header
    int32     responseFlags;  // bit vector - see details below
    int64     cursorID;       // cursor id if client needs to do get more's
    int32     startingFrom;   // where in the cursor this reply is starting
    int32     numberReturned; // number of documents in the reply
    document* documents;      // documents
}
```

| Field            | Description                                                  |
| :--------------- | :----------------------------------------------------------- |
| `header`         | 消息头，正如在[标准消息头](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/#wp- messageheader)中描述的那样。 |
| `responseFlags`  | 指定标志的位向量。位值对应如下:<br />`0`对应于CursorNotFound。在调用 **getMore** 时设置，但游标id在服务器上无效。返回的结果为零。<br />`1`对应于QueryFailure。查询失败时设置。结果包含一个文档，其中包含描述失败的“$err”字段。<br />`2`对应于ShardConfigStale。驱动程序应该忽略这一点。只有[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)会看到这个设置，在这种情况下，它需要从服务器更新配置。<br />`3`对应于awaitable。当服务器支持AwaitData查询选项时设置。如果没有，客户端应该在可定制游标的getMore之间稍作休息。Mongod 1.6版本支持AwaitData，因此总是设置AwaitCapable。<br />`4--31`保留。忽视。 |
| `cursorID`       | `OP_REPLY`所属的 **cursorID** 。如果查询的结果集能放入一个OP_REPLY消息中，' cursorID '将为0。这种**cursorID**必须使用在任何[OP_GET_MORE](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ # wire-op-get-more)消息用于获取更多的数据,也不再需要时,必须关闭客户端通过一个[OP_KILL_CURSORS](https://docs.mongodb.com/master/reference/mongodb-wire-protocol/ # wire-op-kill-cursors)消息。 |
| `startingFrom`   | 光标的起始位置。                                             |
| `numberReturned` | 回复文件的数量。                                             |
| `documents`      | 返回的文档。                                                 |