
# 读关注点"snapshot"

版本4.0中的新功能

读取关注**“snapshot”**只对多文档事务可用。

*   如果事务不是[因果一致会话](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions)的一部分，那么在事务提交时，写关注点为[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")，事务操作就保证已经从多数提交数据的快照中读取了数据。
*   如果事务是[因果一致会话](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions)的一部分，那么在事务提交时，write concern为[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")，事务操作保证已经从多数提交数据的快照中读取，该快照提供了与事务开始前的操作因果一致的数据。

## 操作

有关接受阅读关注的所有操作的列表，请参阅 [支持读关注的操作](https://docs.mongodb.com/manual/reference/read-concern/#read-concern-operations)。

## 阅读关注和事务

多文档事务支持阅读关注 [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern."snapshot")以及[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local")和 [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")。

> **[success] Note**
>
> 您可以在事务级别上而不是在单个操作级别上设置读取关注。要设置事务的已读关注点，请参见[事务和已读关注点](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern)。

对于分片群集上的事务，如果事务中的任何操作涉及已[禁用读关注度“多数”的分片](https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority)，则不能[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern."snapshot")对事务使用读关注度。您只能使用已读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local")或[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")用于事务。如果使用读取关注[`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern."snapshot")，则事务错误并中止。有关更多信息，请参见 [禁用阅读关注多数](https://docs.mongodb.com/manual/core/transactions/#transactions-disabled-rc-majority)。





译者：杨帅

校对：杨帅





  
