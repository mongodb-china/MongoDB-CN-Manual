# 读关注 "majority"

**在本页面**

* [性能](read-concern-majority.md#性能)
* [可用性](read-concern-majority.md#可用性)
* [例子](read-concern-majority.md#例子)
* [存储引擎支持](read-concern-majority.md#支持)
* [读关注`"majority"`和事务](read-concern-majority.md#事务)
* [读关注`"majority"`和汇总](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/MongoDB-CRUD-Operations/Read-Isolation-Read-Concern/总/README.md)
* [读取自己的写入](read-concern-majority.md#写入)
* [禁用读关注多数](read-concern-majority.md#禁用)

对于[多文档事务](https://docs.mongodb.com/master/core/transactions/)中无关的读操作，阅读问题**“majority”**保证所读的数据得到了大多数复制集成员的认可\(即，所读的文档是持久的，并且保证不会回滚\)。

对于[多文档事务](https://docs.mongodb.com/master/core/transactions/)中的操作，只有当事务以写关注点“多数”提交时，读关注点\[`多数`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority"\)才提供保证。否则，“多数”读取关注不能保证在事务中读取的数据。](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"%29才提供保证。否则，“多数”读取关注不能保证在事务中读取的数据。)

不管读关注级别是什么，节点上的最新数据都可能不能反映系统中数据的最新版本。

## 性能

每个复制集成员在内存中维护多数提交点处的数据视图。多数提交点是由初级计算的。为了满足读取关注**"majority"**，该节点从该视图返回数据，并且性能成本与其他读取关注相当。

## 可用性

无论会话和事务是否一致，都可以使用读关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29。)

对于使用三成员`主-副-仲裁(PSA)`体系结构的部署，可以禁用读关注 \[`"majority"`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority"\)”,然而，这对更改流\(MongoDB](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"%29”,然而，这对更改流%28MongoDB) 4.0和更早版本中只使用\)和分片集群上的事务有影响。有关更多信息，请参见[禁用读关注多数](https://docs.mongodb.com/master/reference/read-concern-majority/#disable-read-concern-majority).。

## 例子

考虑写入操作 Write0 到三个成员复制集的以下时间轴：

> **注意**
>
> * Write0 之前的所有写操作都已成功复制到所有成员。
> * Writeprev 是 Write0之前的写入。
> * 在 Write0之后没有发生其他写操作。

![Timeline of a write operation to a three member replica set.](https://www.mongodb.com/docs/manual/images/read-concern-write-timeline.svg)

| 时间 | 事件 | 最新写 | 最新的多数写 |
| :--- | :--- | :--- | :--- |
| t0 | 主要适用于Write0 | 主要：Write0 次要1：Writeprev 次要2：Writeprev | 主要：Writeprev 次要1：Writeprev 次要2：Writeprev |
| t1 | Secondary1适用于Write0 | 主要：Write0 次要1：Write0 次要2：Writeprev | 主要：Writeprev 次要1：Writeprev 次要2：Writeprev |
| t2 | Secondary2适用于Write0 | 主要：Write0 次要1：Write0 次要2：Write0 | 主要：Writeprev 次要1：Writeprev 次要2：Writeprev |
| t3 | Primary知道到Secondary1的复制成功，并向客户端发送确认 | 主要：Write0 次要1：Write0 次要2：Write0 | 主要：Write0 次要1：Writeprev 次要2：Writeprev |
| t4 | Primary 知道成功复制到 Secondary2 | 主要：Write0 次要1：Write0 次要2：Write0 | 主要：Write0 次要1：Writeprev 次要2：Writeprev |
| t5 | Secondary1接收通知\(通过常规复制机制\)以更新其最近 w：“多数”写入的快照 | 主要：Write0 次要1：Write0 次要2：Write0 | 主要：Write0 次要1：Write0 次要2：Writeprev |
| t6 | Secondary2接收通知\(通过常规复制机制\)以更新其最近 w：“多数”写入的快照 | 主要：Write0 次要1：Write0 次要2：Write0 | 主要：Write0 次要1：Write0 次要2：Write0 |

然后，下表总结了具有\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)读关注的读取操作在时间将看到的数据状态\`T\`。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29读关注的读取操作在时间将看到的数据状态`T`。)

![Timeline of a write operation to a three member replica set.](https://www.mongodb.com/docs/manual/images/read-concern-write-timeline.svg)

| 阅读目标 | Time `T` | 数据状态 |
| :--- | :--- | :--- |
| Primary | 在t3之前 | 数据反映了 Writeprev |
| Primary | 在t3之后 | 数据反映了 Write0 |
| Secondary1 | 在t5之前 | 数据反映了 Writeprev |
| Secondary1 | 在t5之后 | 数据反映了 Write0 |
| Secondary2 | 在t6之前 | 数据反映了 Writeprev |
| Secondary2 | 在t6之后 | 数据反映了 Write0 |

## 存储引擎支持

阅读关注“多数”是可用的WiredTiger存储引擎。

> **提示**
>
> [serverStatus](read-concern-majority.md)命令返回[storageEngine.supportsCommittedReads](read-concern-majority.md)字段，该字段指示存储引擎是否支持**”majority“**读取问题。

## 读关注`"majority"`和事务

> **\[success\] Note**
>
> 您可以在事务级别上而不是在单个操作级别上设置读关注。要设置事务的已读关注点，请参见[事务和已读关注点](https://docs.mongodb.com/manual/core/transactions/#transactions-read-concern)。

对于[多文档事务中的操作](https://docs.mongodb.com/manual/core/transactions/)，`"majority"`仅当事务以[写关注“多数”](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern)提交时，读关注才提供其保证。否则， \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)读取关注点不能保证事务中读取的数据。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29读取关注点不能保证事务中读取的数据。)

## 读关注`"majority"`和汇总

从MongoDB 4.2开始，您可以为包含[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)阶段的聚合指定[读取关注](https://docs.mongodb.com/manual/reference/read-concern/) level \[`"majority"`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority"\)。](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"%29。)

在MongoDB 4.0和更早版本中，您不能包括将读取关注用于聚合的[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out) 阶段\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29。)

## 读取自己的写入

更改了 version 3.6.

从 MongoDB 3.6 开始，如果写请求确认，则可以使用因果关系一致来读取您自己的写入。

在MongoDB 3.6之前，您必须发出具有写入关注点的写入操作， 然后 对读取操作使用或关注读取，以确保单个线程可以读取自己的写入。\[`{ w: "majority" }`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)\[\`"majority"\`\]\(https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)\[\`"linearizable"\`\]\(https://docs.mongodb.com/manual/reference/read-concern-linearizable/\#readconcern."linearizable](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29[`"majority"`]%28https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29[`"linearizable"`]%28https://docs.mongodb.com/manual/reference/read-concern-linearizable/#readconcern."linearizable)"\).

在MongoDB 3.6之前，你必须使用\[`{ w: "majority" }`\]\([https://docs.mongodb.com/master/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority)"\) 写关注点来发布写操作，然后使用\[`"majority"`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority"\)或\[\`"linearizable"\`\]\(https://docs.mongodb.com/master/reference/read-concern-linearizable/\#readconcern."linearizable](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"%29或[`"linearizable"`]%28https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable)"\) 的读关注点来执行读操作，以确保单个线程可以读取自己的写操作。

## 禁用读关注多数

适用于3成员`主-副-仲裁器`体系结构

\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)如果您具有具有主要-次要仲裁器（PSA）体系结构的三成员复制集或具有三成员PSA分片的分片群集，则可以禁用读关注。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29如果您具有具有主要-次要仲裁器（PSA）体系结构的三成员复制集或具有三成员PSA分片的分片群集，则可以禁用读关注。)

> **\[success\] Note**
>
> 如果您使用的是 3-member PSA 以外的部署，则无需禁用多数读关注。

对于三成员PSA架构，缓存压力将增加，如果任何承载数据的节点是关闭的。为了防止存储缓存压力使PSA架构的部署无法被锁定，您可以通过设置以下任一项来禁用read concern:

* [`--enableMajorityReadConcern`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-enablemajorityreadconcern)的命令行选项`false`。
* [`replication.enableMajorityReadConcern`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern)配置文件设置为`false`。

要检查是否已禁用“大多数”的读关注，您可以[`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#db.serverStatus)在[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例上运行 并检查该[`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/manual/reference/command/serverStatus/#serverstatus.storageEngine.supportsCommittedReads)字段。如果为`false`，则禁用“大多数”关注。

> **\[warning\] 重要**
>
> 通常，除非必要，否则请避免禁用\[`"majority"`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority)"\) 读取问题。但是，如果您的 three-member 复制集具有 `主-副-仲裁(PSA)`体系结构 或带有 three-member PSA 分片的分片 cluster，请禁用以防止存储缓存压力导致部署无法运行。 禁用“多数”读取问题会禁用对改变流的支持。

**变更流**

禁用\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)读取关注会禁用对MongoDB](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29读取关注会禁用对MongoDB) 4.0及更早版本的[变更流的](https://docs.mongodb.com/manual/changeStreams/)支持。对于MongoDB 4.2+，禁用读取关注**"majority"**不会影响变更流的可用性。

**事务次数**

禁用\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)读取关注会影响对分片群集上\[事务的\]\(https://docs.mongodb.com/manual/core/transactions/\)支持](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29读取关注会影响对分片群集上[事务的]%28https://docs.mongodb.com/manual/core/transactions/%29支持) 。特别：

* \[`"snapshot"`\]\([https://docs.mongodb.com/manual/reference/read-concern-snapshot/\#readconcern."snapshot"\)如果事务涉及已\[禁用读取关注“多数”的分片\]\(https://docs.mongodb.com/manual/reference/read-concern-majority/\#disable-read-concern-majority\)，则该事务不能使用读取关注。](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern."snapshot"%29如果事务涉及已[禁用读取关注“多数”的分片]%28https://docs.mongodb.com/manual/reference/read-concern-majority/#disable-read-concern-majority%29，则该事务不能使用读取关注。)
* 如果事务的任何读或写操作写入多个分片错误，则该事务涉及已禁用读取关注的分片\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29。)

但是，它不影响复制集上的[事务](https://docs.mongodb.com/manual/core/transactions/)。对于复制集上的事务，即使禁用了读关注，也可以为多文档事务指定读关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)（或\[\`"snapshot"\`\]\(https://docs.mongodb.com/manual/reference/read-concern-snapshot/\#readconcern."snapshot](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29（或[`"snapshot"`]%28https://docs.mongodb.com/manual/reference/read-concern-snapshot/#readconcern."snapshot)"\) 或\[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local"\)）\[\`"majority"\`\]\(https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)。](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local"%29）[`"majority"`]%28https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29。)

**回滚的注意事项**

禁用\[`"majority"`\]\([https://docs.mongodb.com/master/reference/read-concern-majority/\#readconcern."majority"\)读关注可以防止修改索引的\[\`collMod\`\]\(https://docs.mongodb.com/master/reference/command/collMod/\#dbcmd.collMod](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"%29读关注可以防止修改索引的[`collMod`]%28https://docs.mongodb.com/master/reference/command/collMod/#dbcmd.collMod)\) 命令回滚。如果需要[回滚](https://docs.mongodb.com/master/core/replica-set-rollbacks/#replica-set-rollbacks)此类操作，则必须将受影响的节点与主节点重新同步。

