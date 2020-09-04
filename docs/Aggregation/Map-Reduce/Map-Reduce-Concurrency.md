# [ ](#)Map-Reduce 并发

[]()

map-reduce 操作由许多任务组成，包括从输入集合中读取，执行`map` function，执行`reduce` function，在处理期间写入临时集合以及写入输出集合。

在操作期间，map-reduce 采用以下锁定：

*   读取阶段采用读锁定。它产生每 100 个文件。

*   insert 进入临时集合会为单次写入执行写锁定。

*   如果输出集合不存在，则输出集合的创建将采用写入锁定。

*   如果输出集合存在，则输出操作(即：`merge`，`replace`，`reduce`)将执行写入锁定。此写锁定是 global，并阻止[mongod]()实例上的所有操作。
> **注意**
>
> 后处理期间的最终写锁定使结果自动显示。然而，输出操作`merge`和`reduce`可能需要时间来处理。对于`merge`和`reduce`，该 `nonAtomic`标志可用，从而释放写入每个输出文档之间的锁定。从MongoDB 4.2开始，不推荐使用`nonAtomic: false`显式设置。有关[`db.collection.mapReduce()`]()更多信息，请参见参考。



译者：李冠飞

校对：