# 读关注“local”

具有读取关注点的查询`local`从实例返回数据，但不保证数据已写入大多数复制集成员(即：可能会回滚)。

读取关注`local`是默认值：

*   读取针对主要的操作

*   如果读取与因果关系一致关联，则读取针对辅助节点的操作。

不管[读关注](https://docs.mongodb.com/manual/reference/glossary/#term-read-concern)级别如何，节点上的最新数据都可能无法反映系统中数据的最新版本。

## 可用性

读关注`local`可用于有或没有因果关系一致的会话和事务。

## 读关注”local“和事务

您可以在事务级别上而不是在单个操作级别上设置读取关注。要设置事务的已读关注点，请参见[事务和已读关注点](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern)。

从MongoDB 4.4开始，[功能兼容版本(fcv)](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv) “4.4”或更高版本，您可以在事务中创建集合和索引。如果显式地创建集合或索引，则事务必须使用read concern“[`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local")。[隐式](https://docs.mongodb.com/master/core/transactions-operations/#transactions-operations-ddl-implicit)创建集合可以使用事务可用的任何读取关注点。

## 例子

考虑写入操作 Write<sub>0</sub> 到三个成员副本集的以下时间轴：

> **注意**
>
> * Write<sub>0</sub> 之前的所有写操作都已成功复制到所有成员。
>
> * Write<sub>prev</sub> 是 Write<sub>0</sub>之前的写入。
>
> * 在 Write<sub>0</sub>之后没有发生其他写操作。

![对三个成员复制集的写操作的时间轴。](https://docs.mongodb.com/manual/_images/read-concern-write-timeline.svg)

| 时间          | 事件                                                         | 最新写                                                       | 最新的多数写                                                 |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| t<sub>0</sub> | 主要适用于Write<sub>0</sub>                                  | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>1</sub> | Secondary<sup>1</sup>适用于Write<sub>0</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>2</sub> | Secondary<sup>2</sup>适用于Write<sub>0</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>3</sub> | Primary知道到Secondary<sub>1</sub>的复制成功，并向客户端发送确认 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>4</sub> | Primary 知道成功复制到 Secondary<sub>2</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>5</sub> | Secondary<sub>1</sub>接收通知(通过常规复制机制)以更新其最近 w：“多数”写入的快照 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>6</sub> | Secondary<sub>2</sub>接收通知(通过常规复制机制)以更新其最近 w：“多数”写入的快照 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> |

然后，下表总结了具有[“local”](#)读关注的读操作在T时刻看到的数据状态。

![Timeline of a write operation to a three member replica set.](https://docs.mongodb.com/manual/_images/read-concern-write-timeline.svg)

| 阅读目标              | Time `T`            | 数据状态                        |
| --------------------- | ------------------- | ------------------------------- |
| Primary               | 在t<sub>0</sub>之后 | 数据反映了 Write<sub>0</sub>    |
| Secondary<sub>1</sub> | 在t<sub>1</sub>之前 | 数据反映了 Write<sub>prev</sub> |
| Secondary<sub>1</sub> | 在t<sub>1</sub>之后 | 数据反映了 Write<sub>0</sub>    |
| Secondary<sub>2</sub> | 在t<sub>2</sub>之前 | 数据反映了 Write<sub>prev</sub> |
| Secondary<sub>2</sub> | 在t<sub>2</sub>之后 | 数据反映了 Write<sub>0</sub>    |



译者：杨帅

校对：杨帅
