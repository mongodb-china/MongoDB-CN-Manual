# 读关注“available”

version 3.6 中的新内容。

与read有关的“available”查询从实例返回数据，但不保证数据已经被写入大多数复制集成员(即可能被回滚)。

如果读操作不与因果一致的会话相关联，那么读关注“available”是对次要操作的默认读操作。

**对于分片 cluster**，[`"available"`](https://docs.mongodb.com/master/reference/read-concern-available/#readconcern."available") 读取问题为分区提供了更大的容忍度，因为它不会等待以确保一致性保证。但是，如果分片正在进行大块迁移，那么带有 [`"available"`](https://docs.mongodb.com/master/reference/read-concern-available/#readconcern."available")读取问题的查询可能会return孤立文档，因为“本地”读取问题与“本地”读取问题不同，它不会联系分片的主服务器或配置服务器以更新元数据。

**对于unsharded集合**(包括独立部署或复制集部署中的集合)，[`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local") 和 [`"available"`](https://docs.mongodb.com/master/reference/read-concern-available/#readconcern."available") 读取问题的行为相同。

不管[read concern](https://docs.mongodb.com/master/reference/glossary/#term-read-concern)级别，节点上的最新数据可能不能反映系统中数据的最新版本。

> **也可以看看**
>
> [`orphanCleanupDelaySecs`](https://docs.mongodb.com/master/reference/parameters/#param.orphanCleanupDelaySecs)

## 可用行

读关注 **available**对于因果一致的会话和事务不可用。

## 例子

考虑写入操作 Write<sub>0</sub> 到三个成员复制集的以下时间轴：

> **[success] Note**
>
> 为了简化，本例假设:
>
> * Write<sub>0</sub> 之前的所有写操作都已成功复制到所有成员。
>
> * Write<sub>prev</sub> 是 Write<sub>0</sub>之前的写入。
>
> * 在 Write<sub>0</sub>之后没有发生其他写操作。

![Timeline of a write operation to a three member replica set.](https://docs.mongodb.com/manual/_images/read-concern-write-timeline.svg)

| 时间          | 事件                                                         | 最新写                                                       | 最新的多数写                                                 |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| t<sub>0</sub> | 主要适用于Write<sub>0</sub>                                  | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>1</sub> | Secondary<sup>1</sup>适用于Write<sub>0</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>2</sub> | Secondary<sup>2</sup>适用于Write<sub>0</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>prev</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>3</sub> | Primary知道到Secondary<sub>1</sub>的复制成功，并向客户端发送确认 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>4</sub> | Primary 知道成功复制到 Secondary<sub>2</sub>                 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>prev</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>5</sub> | Secondary<sub>1</sub>接收通知(通过常规复制机制)以更新其最近 w：“多数”写入的快照 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>prev</sub> |
| t<sub>6</sub> | Secondary<sub>2</sub>接收通知(通过常规复制机制)以更新其最近 w：“多数”写入的快照 | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> | 主要：Write<sub>0</sub><br/>次要<sub>1</sub>：Write<sub>0</sub><br />次要<sub>2</sub>：Write<sub>0</sub> |


然后，下表总结了 time读取关注的读操作在 time `T`处将看到的数据的 state。

| 阅读目标              | Time `T`             | 状态的数据                         |
| :-------------------- | :------------------- | :--------------------------------- |
| Primary               | After t<sub>0</sub>  | Data reflects Write<sub>0</sub>    |
| Secondary<sub>1</sub> | Before t<sub>1</sub> | Data reflects Write<sub>prev</sub> |
| Secondary<sub>1</sub> | After t<sub>1</sub>  | Data reflects Write<sub>0</sub>    |
| Secondary<sub>2</sub> | Before t<sub>2</sub> | Data reflects Write<sub>prev</sub> |
| Secondary<sub>2</sub> | After t<sub>2</sub>  | Data reflects Write<sub>0</sub>    |



译者：杨帅

校对：杨帅
