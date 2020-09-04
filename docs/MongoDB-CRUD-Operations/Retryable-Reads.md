# 可重试读取


**在本页面**

*   [前提条件](#prerequisites)
*   [启用可重试读取](#enabling-retryable-reads)
*   [可重试的读取操作](#retryable-read-operations)
*   [行为](#behavior)

可重试读取允许MongoDB驱动程序在遇到某些网络或服务器错误时，可以一次自动重试某些读取操作。

## <span id="prerequisites">前提条件</span>

### 最小驱动程序版本

​		官方MongoDB驱动兼容MongoDB服务器4.2和以后支持重试读取。

​		有关官方MongoDB驱动程序的更多信息，请参阅 [MongoDB驱动程序](https://docs.mongodb.com/drivers/)。

### 最低服务器版本

​		如果连接到MongoDB Server 3.6或更高版本，驱动程序只能重试读取操作。


## <span id="enabling-retryable-reads">启用可重试读取</span>

官方MongoDB驱动程序兼容MongoDB服务器4.2和以后默认启用可重试读取。要显式禁用可重试读取，请在部署的[连接字符串中](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-uri)中指定[`retryReads=false`](https://docs.mongodb.com/manual/reference/connection-string/#urioption.retryReads)。

在[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)shell不支持重试读取。


## <span id="retryable-read-operations">可重试的读取操作</span>

MongoDB驱动程序支持重试以下读取操作。列表引用了每个方法的通用描述。对于特定的语法和用法，请遵循该方法的驱动程序文档。

| 方法                                                         | 内容描述          |
| ------------------------------------------------------------ | ----------------- |
| Collection.aggregate<br /> Collection.count <br />Collection.countDocuments<br /> Collection.distinct<br /> Collection.estimatedDocumentCount <br />Collection.find <br />Database.aggregate | CRUD API读取操作. |

对于`Collection.aggregate`和`Database.aggregate`，驱动程序只能重试不包括写阶段的聚合管道，如[$out](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)或[$merge](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge)。

|                                                              |                    |
| ------------------------------------------------------------ | ------------------ |
| Collection.watch<br /> Database.watch <br />MongoClient.watch | 更改流操作         |
| MongoClient.listDatabases<br /> Database.listCollections<br /> Collection.listIndexes | 枚举操作           |
| GridFS操作由`Collection.find` （<br />例如`GridFSBucket.openDownloadStream`）支持 | GridFS文件下载操作 |

MongoDB驱动程序可能包括对其他操作的可重试支持，比如帮助方法或包装可重试读操作的方法。根据[驱动程序文档](https://docs.mongodb.com/drivers/) 确定方法是否显式支持可重试读取。

也可以看看:

可重试读规范:[支持的读取操作](https://github.com/mongodb/specifications/blob/master/source/retryable-reads/retryable-reads.rst#supported-read-operations).

#### 不支持的读取操作

以下操作不支持可重试的读取：

*   [db.collection.mapReduce()](https://docs.mongodb.com/manual/reference/method/db.collection.mapReduce/#db.collection.mapReduce)
*   [getMore](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore)
*   传递给通用**Database.runCommand**帮助器的任何读命令，它与读或写命令无关。

## <span id="behavior">行为</span>

### 持久性网络错误

MongoDB可重试读取只做一次重试尝试。这有助于解决暂时的网络错误或[复制集选举](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)，但不能解决持久的网络错误。

### 故障转移期间

在重试读取操作之前，驱动程序使用read命令的原始[读取首选项](https://docs.mongodb.com/manual/core/read-preference/#read-preference)执行[服务器选择](https://docs.mongodb.com/manual/core/read-preference-mechanics/#replica-set-read-preference-behavior)。如果驱动程序不能选择使用原始读取首选项进行重试的服务器，则驱动程序返回原始错误。

驱动程序在执行服务器选择之前等待[`serverSelectionTimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.serverSelectionTimeoutMS)毫秒。可重试读取不会处理在等待[`serverSelectionTimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.serverSelectionTimeoutMS)后不存在合格服务器的实例。



译者：杨帅

校对：杨帅