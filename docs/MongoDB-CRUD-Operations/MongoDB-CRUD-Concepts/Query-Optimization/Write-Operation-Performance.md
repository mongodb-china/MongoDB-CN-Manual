# 写操作性能

**在本页面**

- [索引](#索引)
- [储存性能](#储存)

## <span id="索引">索引</span>

集合上的每个索引都会给写操作的性能增加一些负担。

对于集合上的每个操作[`insert`](https://docs.mongodb.com/manual/reference/command/insert/#dbcmd.insert)或[`delete`](https://docs.mongodb.com/manual/reference/command/delete/#dbcmd.delete)写入操作，MongoDB从目标集合的每个索引中插入或删除相应的文档键。根据受[`update`](https://docs.mongodb.com/manual/reference/command/update/#dbcmd.update)影响的键，更新操作可能导致对集合上的索引子集进行更新。

> **[success] 注意**
>
> 如果写操作中涉及的文档包含在索引中，则MongoDB仅更新[稀疏](https://docs.mongodb.com/manual/core/index-sparse/#index-type-sparse)索引或 [部分](https://docs.mongodb.com/manual/core/index-partial/#index-type-partial)索引。

一般来说，索引为读操作提供的性能收益抵得上插入损失。但是，为了尽可能优化写性能，在创建新索引和评估现有索引时要小心，以确保您的查询实际使用这些索引。

有关索引和查询，请参见[查询优化](https://docs.mongodb.com/manual/core/query-optimization/)。有关索引的更多信息，请参见[索引](https://docs.mongodb.com/manual/indexes/)和 [索引策略](https://docs.mongodb.com/manual/applications/indexes/)。

## <span id="储存">储存性能</span>

### 硬件

存储系统的功能为MongoDB的写操作性能创建了一些重要的物理限制。与驱动器的存储系统相关的许多独特因素都会影响写入性能，包括随机访问模式，磁盘缓存，磁盘预读和RAID配置。

对于随机工作负载，固态驱动器（SSD）的性能可比旋转硬盘（HDD）高100倍或更多。

​	请看:

​	[生产说明](https://docs.mongodb.com/manual/administration/production-notes/)中有关其他硬件和配置选项的建议。

### 日记

为了在崩溃时提供持久性，MongoDB使用预*写日志记录*到磁盘[日志上](https://docs.mongodb.com/manual/reference/glossary/#term-journal)。MongoDB首先将内存中的更改写入磁盘上的日志文件。如果MongoDB在对数据文件进行更改之前终止或遇到错误，MongoDB可以使用日志文件对数据文件应用写操作。

虽然日志提供的持久性保证通常超过了额外写操作的性能成本，但考虑一下日志和性能之间的以下交互:

- 如果日志和数据文件位于同一块设备上，则数据文件和日志可能必须竞争有限数量的可用I / O资源。将日志移动到单独的设备可能会增加写操作的容量。
- 如果应用程序指定了包括**J**选项的[写关注点](https://docs.mongodb.com/manual/reference/write-concern/)，mongod将减少日志写之间的持续时间，这会增加总体写负载。
- 日志写入之间的持续时间可以使用[`commitIntervalMs`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.journal.commitIntervalMs)运行时选项进行配置 。减少日志提交之间的时间间隔将增加写入操作的数量，这可能会限制MongoDB的写入操作能力。增加日志提交之间的时间量可能会减少写操作的总数，但也会增加在发生故障的情况下日志不会记录写操作的机会。

有关日志记录的其他信息，请参见[日志记录](https://docs.mongodb.com/manual/core/journaling/)。



译者：杨帅

校对：杨帅