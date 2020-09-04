# 读取隔离、一致性和近效性

**在本页面**


*   [隔离保证](#隔离)
*   [单调写](#单调)
*   [实时顺序](#订单)
*   [因果一致性](#因果)

## <span id="隔离">隔离保证</span>


### 读取未提交

根据读取的关注点，客户端可以在[持久](https://docs.mongodb.com/manual/reference/glossary/#term-durable)写入之前看到写入的结果：

- 不考虑一个写操作的[写关注](https://docs.mongodb.com/manual/reference/write-concern/)，其他客户端使用`local`或者`available`的读关注级别都可以看到写操作的结果。
- 使用`local`或`available`读取关注级别的客户端可以读取数据，这些数据随后可能会在副本集故障转移期间回滚。

对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)中的操作，当事务提交时，在事务中进行的所有数据更改都将保存并在事务外部可见。也就是说，一个事务在回滚其他事务时将不会提交其某些更改。

在提交事务之前，在事务外部看不到在事务中进行的数据更改。

但是，当事务写入多个分片时，并非所有外部读取操作都需要等待已提交事务的结果在所有分片上可见。例如，如果提交了一个事务，并且在分片A上可以看到写1，但是在分片B上还看不到写2，则外部一个读关注级别为`local`的读操作可以读取写1的结果而看不到写2。

读未提交是默认的隔离级别，适用于mongod独立实例以及复制集和分片群集。

### 读取未提交和单个文档原子性

对于单个文档，写操作是原子性的。 即，如果写操作正在更新文档中的多个字段，则读操作将永远不会看到仅更新了某些字段的文档。 但是，尽管客户端可能看不到部分更新的文档，但读未提交意味着并发读取操作仍可以在使更改持久之前看到更新的文档。

对于以独立模式部署的 [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例，对单个文档的一组读写操作是线性的。 使用复制集时，只有在没有回滚的情况下，对单个文档的一组读取和写入操作才是线性的。

### 读取未提交和多文档写入

当单个写入操作（例如 [`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)）修改多个文档时，每个文档的修改都是原子的，但整个操作不是原子的。

当单个写入操作（例如`db.collection.updateMany()`）修改多个文档时，每个文档的修改都是原子的，但整个操作不是原子的。

当执行多文档写操作时，无论是通过单个写操作还是通过多个写操作，其他操作都可能会交错。

对于需要原子性地读写多个文档（在单个或多个集合中）的情况，MongoDB支持多文档事务：

- 4.0版本中，MongoDB支持副本集内的多文档事务。
- 4.2版本中，MongoDB引入了分布式事务，从而增加了对分片群集上多文档事务的支持，并结合了对副本集上多文档事务的现有支持。

关于MongoDB事务的细节，请参考[事务](https://docs.mongodb.com/manual/core/transactions/)页。


> **[warning] 重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性并不能替代有效的架构设计。 在许多情况下，[非结构化化数据模型(嵌入式文档和数组)](https://docs.mongodb.com/manual/core/data-model-design/#data-modeling-embedding)将继续是您的数据和用例的最佳选择。 也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 关于其他事务使用方面的注意事项(比如运行时显示和oplog大小限制等)，请参考[生产注意事项](https://docs.mongodb.com/manual/core/transactions-production-consideration/).


在不隔离多文档写入操作的情况下，MongoDB表现出以下行为：

1. **非时间点(`Non-point-in-time`)读取操作**。假设读取操作在时间t1开始并开始读取文档。然后，写操作在稍后的某个时间t2提交对其中一个文档的更新。读操作可能会看到文档的更新后版本，因此看不到数据的point-in-time快照。
2. **非可序列化的操作**。假设读取操作在时间t1读取文档d1，而写入操作在稍后的时间t3更新d1。这引入了读写依赖性，因此，如果要序列化操作，则读取操作必须先于写入操作。但是还假设写操作在时间t2更新文档d2，而读取操作随后在稍后的时间t4读取d2。这就引入了写-读依赖关系，它将要求读操作在可序列化时间表中在写操作之后进行。有一个依赖循环，使可序列化成为不可能。
3. 读取操作可能会丢失在读取操作过程中更新的匹配文档。

### 游标快照

在某些情况下，MongoDB游标可以多次返回同一个文档。 当游标返回文档时，其他操作可能会与查询交错。 如果其中某些操作更改了查询使用的索引上的索引字段； 那么光标将多次返回同一文档。

如果您的集合中有一个或多个从未修改过的字段，则可以在此字段或这些字段上使用唯一索引，这样查询将最多返回每个文档一次。 使用`hint()`查询可显式强制查询使用该唯一索引。

## 单调写

**默认的，对于standalone和复制集，MongoDB提供单调写入保证。**

对于分片集群的单调写入，参考[因果一致性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)。

## <span id="订单">实时顺序</span>

*3.4后新引入*

对于主节点上的读取和写入操作，如果将读关注设置为`linearizable`，将写关注设置为`majority`，那么这种读写模型组合可以使多个线程可以在单个文档上执行读写操作，就好像单个线程实时执行了这些操作一样 ; 也就是说，这些读写的相应计划被认为是线性的。

亦可参考：

[因果一致性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)

## <span id="因果">因果一致性</span>

*3.6后新引入*

如果操作在逻辑上取决于先前的操作，则这些操作之间存在因果关系。 例如，基于指定条件删除所有文档的写入操作和验证删除操作的后续读取操作具有因果关系。

在因果一致的会话中，MongoDB按照尊重因果关系的顺序执行因果操作，并且客户观察到与因果关系一致的结果。

### 客户端会话与因果一致性保证

为了提供因果一致性，MongoDB 3.6在客户端会话中启用因果一致性。 因果一致的会话表示具有`majority`的读关注级别的读操作和具有`majority`的写关注级别的写操作的关联序列具有因果关系，这由它们的顺序反映出来。 **应用程序必须确保一次只有一个线程在客户端会话中执行这些操作**。

对于因果相关的操作：

1. 客户端开始一个客户端会话

   > **[warning] 重要**
   >
   > 客户端会话仅在以下情况下保证因果一致性：
   >
   > - 读取操作的读关注级别为`majority`；即返回数据已被大多数副本集成员确认并且是持久化的。
   >
   > - 写操作的写关注级别为`majority`；即要求确认该操作已应用于副本集中大多数可投票成员。
   >
   >   关于因果一致性和多种读关注级别/写关注级别，请参考[因果一致性和读/写关注级别](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/)。
   
2. 当客户端发出具有`majority`读关注和`majority`写关注的读取序列时，客户端将会话信息包含在每个操作中。

3. 对于与会话相关联的每个具有`majority`读关注的读取操作和具有`majority`写关注的写入操作，即使操作出错，MongoDB也会返回操作时间和集群时间。 客户端会话跟踪操作时间和群集时间。

   > **[success] 注意**
   >
   > 对于未确认的`（w：0）`写操作，MongoDB不返回操作时间和群集时间。未经确认的写入并不表示任何因果关系。
   >
   > 尽管MongoDB在客户端会话中返回读操作和已确认写操作的操作时间和集群时间，但是只有具有`majority`读关注的读取操作和具有`majority`写关注的写入操作才能保证因果一致性。 有关详细信息，请参见[因果一致性和读/写关注级别](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/)。

4. 相关的客户端会话会跟踪这两个时间字段。

   >  **[success] 注意**
   >
   > 不同会话之间的操作可以因果一致。 MongoDB驱动程序和mongo Shell提供了推进客户端会话的操作时间和集群时间的方法。 因此，客户端可以推进一个客户端会话的群集时间和操作时间，使其与另一客户端会话的操作保持一致。

#### 因果一致性保证

下表列出了因果一致会话为具有`majority`读关注的读取操作和具有`majority`写关注点的写入操作提供的因果一致性保证。

| 保证   | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| 写后读 | 读操作可以正确读到之前写的结果。                             |
| 单调读 | 多个读操作会返回一样的<br>比如在一个会话中“<br>- 写操作1在写操作2前，<br>- 读操作1在读操作2前，并且，<br>- 读操作1返回了写操作2的结果<br>那么读操作2并不会返回写操作1的结果。 |
| 单调写 | 写操作按顺序进行。<br>比如，如果会话中写操作1在写操作2前，数据在写操作2的状态必须是写操作1完成后的状态。其他写入操作可以在写操作1和写操作2之间进行交错，但写操作2不可能在写操作1之前进行。 |
| 读后写 | 写操作在读操作后执行。 即，写入时的数据状态必须包含之前的读取操作的数据状态。 |

#### 读偏好

这些保证适用于MongoDB部署的所有成员。 例如，如果在因果关系一致的会话中发出具有`majority`写关注级别的写操作，然后发出一个具有`majority`读关注级别的从节点（即，读偏好为`secondary`）读操作，则读取操作将反映写入操作后的数据库状态。

#### 隔离性

**因果一致的会话内的操作与会话外的操作不是隔离的**。 如果并发写操作在会话的写操作和读取操作之间交错，则会话的读操作可能返回反映会话写操作之后发生的写操作的结果。

### MongoDB驱动

> 提示：
>
> 应用程序必须确保一次只有一个线程在客户端会话中执行这些操作。

客户端需要使用MongoDB 3.6或更高版本的MongoDB驱动程序：

| Java 3.6+   | C# 2.5+   | Perl 2.0+  |
| ----------- | --------- | ---------- |
| Python 3.6+ | Node 3.0+ | PHPC 1.4+  |
| C 1.9+      | Ruby 2.5+ | Scala 2.2+ |

### 示例

> 重要**
>
> 因果一致性会话只能保证对于读关注级别为`majority`以及写关注级别为`majority`的读取操作的因果一致性。

考虑一个维护各种项目的当前和历史数据的`items`集合。 只有历史数据的`end`日期为非空。 如果项目的`sku`值更改，则具有旧`sku`值的文档需要使用`end`日期进行更新，此后，将使用当前`sku`值插入新文档。 客户端可以使用因果一致的会话来确保更新在插入之前发生。

(以python为例，其他实例查看原链接)

```
with client.start_session(causal_consistency=True) as s1:
    current_date = datetime.datetime.today()
    items = client.get_database(
        'test', read_concern=ReadConcern('majority'),
        write_concern=WriteConcern('majority', wtimeout=1000)).items
    items.update_one(
        {'sku': "111", 'end': None},
        {'$set': {'end': current_date}}, session=s1)
    items.insert_one(
        {'sku': "nuts-111", 'name': "Pecans",
         'start': current_date}, session=s1)
```

如果另一个客户端需要读取所有当前的sku值，则可以将集群时间和操作时间推进到另一个会话的集群时间和操作时间，以确保该客户端与另一个会话有因果关系，并在两次写入之后读取：

```
with client.start_session(causal_consistency=True) as s2:
    s2.advance_cluster_time(s1.cluster_time)
    s2.advance_operation_time(s1.operation_time)

    items = client.get_database(
        'test', read_preference=ReadPreference.SECONDARY,
        read_concern=ReadConcern('majority'),
        write_concern=WriteConcern('majority', wtimeout=1000)).items
    for item in items.find({'end': None}, session=s2):
        print(item)
```

### 限制

以下生成内存数据结构的操作并不是因果一致性的：

| 操作                                                         | 备注                                                     |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| [`collStats`](https://docs.mongodb.com/manual/reference/command/collStats/#dbcmd.collStats) |                                                          |
| [`$collStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/collStats/#pipe._S_collStats) with `latencyStats` option. |                                                          |
| [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#pipe._S_currentOp) | 如果操作和一个因果一致性的客户端会话相关，则会返回错误。 |
| [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#dbcmd.createIndexes) |                                                          |
| [`dbHash`](https://docs.mongodb.com/manual/reference/command/dbHash/#dbcmd.dbHash) | MongoDB4.2以后的版本才支持                               |
| [`dbStats`](https://docs.mongodb.com/manual/reference/command/dbStats/#dbcmd.dbStats) |                                                          |
| [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#dbcmd.getMore) | 如果操作和一个因果一致性的客户端会话相关，则会返回错误。 |
| [`$indexStats`](https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#pipe._S_indexStats) |                                                          |
| [`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce) | MongoDB4.2以后的版本才支持                               |
| [`ping`](https://docs.mongodb.com/manual/reference/command/ping/#dbcmd.ping) | 如果操作和一个因果一致性的客户端会话相关，则会返回错误。 |
| [`serverStatus`](https://docs.mongodb.com/manual/reference/command/serverStatus/#dbcmd.serverStatus) | 如果操作和一个因果一致性的客户端会话相关，则会返回错误。 |
| [`validate`](https://docs.mongodb.com/manual/reference/command/validate/#dbcmd.validate) | MongoDB4.2以后的版本才支持                               |



原文链接：https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#



译者：刘翔  杨帅

校对：徐雷

