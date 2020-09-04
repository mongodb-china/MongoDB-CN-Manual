
# 写关注

**在本页面**

*   [编写关注规范](#规范)
*   [确认行为](#行为)
*   [因果一致的会话和写问题](#问题)
*   [计算关注的多数](#多数)

写关注描述了MongoDB请求对独立[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)或[复制集](https://docs.mongodb.com/manual/replication/)或分片[群集](https://docs.mongodb.com/manual/sharding/)进行写操作的确认级别。在分片群集中，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)实例会将写关注事项传递给分片。

> **[success] Note**
>
> 对于[多文档事务](https://docs.mongodb.com/manual/core/transactions/)，可以在事务级别**而不是在单个操作级别**设置写关注。不要为事务中的各个写操作显式设置写关注点。

## <span id="规范">编写关注规范</span>

写关注可包括以下字段：

```powershell
{  w ： < value > ， j ： < boolean > ， wtimeout ： < number >  }
```

*   使用[w](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)选项来请求确认写入操作已传播到指定数量的[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例或[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)具有指定标签的实例。
*   该[j](https://docs.mongodb.com/manual/reference/write-concern/#wc-j)选项要求确认写操作已被写入到磁盘上的杂志，和
*   该[wtimeout](https://docs.mongodb.com/manual/reference/write-concern/#wc-wtimeout)选项来指定一个时间限制，以防止无限期阻塞写操作。

#### w 选项

`w`选项请求确认写操作已传播到指定数量的mongod实例或具有指定标记的mongod实例。

使用`w`选项，可以使用以下**w:<`value`>**写入问题：

| 值                            | 描述                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| `<number>`                    | 请求确认写操作已传播到指定数量的[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例。例如：`w: 1`请求确认写入操作已传播到[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)复制集中的独立副本或主副本。是MongoDB的默认写关注点。 如果在写操作已复制到任何辅助数据库之前主数据库已降级，则可以[回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#rollback-avoid)数据。`w: 1``w: 0`不要求确认写入操作。但是，可能会将有关套接字异常和网络错误的信息返回给应用程序。如果在写操作已复制到任何辅助数据库之前主数据库已降级，则可以[回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#rollback-avoid)数据 。`w: 0`如果指定但包括[j：true](https://docs.mongodb.com/manual/reference/write-concern/#wc-j)，则优先使用 [j：true](https://docs.mongodb.com/manual/reference/write-concern/#wc-j)来请求独立或复制集主副本的确认。`w: 0`[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)`w`大于1则需要来自主数据库的确认以及满足指定写入问题所需的尽可能多的数据承载辅助数据库的确认。例如，考虑一个具有一个主要成员和两个次要成员的3成员复制集。指定将需要主数据库和辅助数据库之一的确认。指定将需要主要和次要确认。`w: 2``w: 3`注意[Hidden](https://docs.mongodb.com/manual/core/replica-set-hidden-member/#replica-set-hidden-members)， [delayed](https://docs.mongodb.com/manual/core/replica-set-delayed-member/#replica-set-delayed-members)和[ priority 0](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members) 成员可以确认写操作。[`w: `](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)延迟的辅助副本可以在不早于configure的情况下返回写确认[`slaveDelay`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].slaveDelay)。有关实例何时确认写入的信息，请参见[确认行为](https://docs.mongodb.com/manual/reference/write-concern/#wc-ack-behavior)[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)。 |
| `"majority"`                  | 请求确认写入操作已传播到所[计算的大多数](https://docs.mongodb.com/manual/reference/write-concern/#calculating-majority-count)含数据投票成员（即具有的[`members[n\].votes`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].votes)大于的主要和次要成员 `0`）。例如，考虑一个具有3个投票成员的复制集，即Primary-Secondary-Secondary（PSS）。对于此复制集， [计算出的多数](https://docs.mongodb.com/manual/reference/write-concern/#calculating-majority-count)为2，并且写入必须传播到主要对象和一个辅助对象，以向客户端确认写入问题。注意[隐藏](https://docs.mongodb.com/manual/core/replica-set-hidden-member/#replica-set-hidden-members)， [延迟](https://docs.mongodb.com/manual/core/replica-set-delayed-member/#replica-set-delayed-members)和[优先级为0的](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members) 成员（[`members[n\].votes`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].votes)大于`0` 可以确认[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写入操作）。延迟的辅助副本可以在不早于configure的情况下返回写确认[`slaveDelay`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].slaveDelay)。在写操作返回确认给客户端之后，客户端可以使用readConcern 读取该写操作的结果 。[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")有关实例何时确认写入的信息，请参见[确认行为](https://docs.mongodb.com/manual/reference/write-concern/#wc-ack-behavior)[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)。 |
| `<custom write concern name>` | 请求确认写操作已传播到[`tagged`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.members[n].tags)满足中定义的自定义写关注点的成员 [`settings.getLastErrorModes`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.settings.getLastErrorModes)。有关示例，请参阅“ [自定义多数据中心写入问题”](https://docs.mongodb.com/manual/tutorial/configure-replica-set-tag-sets/#configure-custom-write-concern)。如果自定义写入问题仅需要在写入操作复制到任何辅助数据库之前先要求主要数据库和主要数据库降级的确认，则可以[回滚](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#rollback-avoid)数据。有关 实例何时确认写入的信息，请参见[确认行为](https://docs.mongodb.com/manual/reference/write-concern/#wc-ack-behavior)[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)。 |

**也可以看看** 

* [默认的MongoDB读问题/写问题](https://docs.mongodb.com/manual/reference/mongodb-defaults/)
* [复制集协议版本](https://docs.mongodb.com/manual/reference/replica-set-protocol-versions/)

#### j 选项

该`j`选项要求MongoDB确认写入操作已写入[磁盘日志中](https://docs.mongodb.com/manual/core/journaling/)。

| `j`  | 如果为，则请求确认[w：中](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)指定的 实例已写入磁盘日志中。本身并不能保证不会因副本集主故障转移而回滚写操作。`j: true`[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)`j: true`<br/>*在版本3.2中进行了更改：*使用时，MongoDB仅在请求数量的成员（包括主要成员）写入日志后才返回。不管[w：](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)写入关注点如何，副本集中以前的写入关注点只要求[主](https://docs.mongodb.com/manual/reference/glossary/#term-primary)记录写到日志。[`j: true`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.j)[`j: true`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.j) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

> **[success] Note**
>
> - 指定一个写入关注点，该关注点包含到正在运行且没有日志记录的 实例中会产生错误。`j: true`[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)
> - 如果启用日记功能，则可能暗示。的 副本集配置设置确定的行为。有关详细信息，请参见 [确认行为](https://docs.mongodb.com/manual/reference/write-concern/#wc-ack-behavior)。[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")`j: true`[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault).
> - 如果日志是启用的，[`w: "majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority") 可能意味着**j：true**。[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault) 复制集配置设置决定了行为。有关详细信息，请参阅[确认行为](https://docs.mongodb.com/master/reference/write-concern/#wc-ack-behavior) 。

#### wtimeout

此选项指定写入问题的 time 限制(以毫秒为单位)。 `wtimeout`仅适用于大于`1`的`w`值。

`wtimeout`导致写入操作返回到指定限制后的错误，即使所需的写入关注最终会成功。当这些写操作 return 时，MongoDB 不会撤消在写入关注超过`wtimeout` time 限制之前执行的成功数据修改。

如果未指定`wtimeout`选项且写入关注的 level 无法实现，则写入操作将无限期阻止。指定`0`的`wtimeout` value 等同于没有`wtimeout`选项的写入问题。

## <span id="行为">确认行为</span>

 当[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例确认写操作时 [w](https://docs.mongodb.com/master/reference/write-concern/#wc-w)选项和 [j](https://docs.mongodb.com/master/reference/write-concern/#wc-j) 选项决定。

#### 独立

独立[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)应用程序在应用了内存中的写入之后或写入磁盘日志后会确认写入操作。下表列出了独立服务器的确认行为以及相关的写入问题：

|                 | `j` 未指定                       | `j:true` | `j:false` |
| :-------------- | :------------------------------- | :------- | --------- |
| `w: 1`          | 在记忆中                         | 磁盘日志 | 在内存中  |
| `w: "majority"` | 磁盘日志（*如果与日志一起运行）* | 磁盘日志 | 在内存中  |

> **[success] Note**
>
> 随着[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为`false`，MongoDB的不等待 写入承认写之前被写入到磁盘上的日志。这样，在给定副本集中的大多数节点出现瞬时丢失（例如崩溃和重新启动）的情况下，写操作可能会回滚。[`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")`majority`

#### 复制集

指定给[w](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)的值确定返回成功之前必须确认写入的复制集成员的数量。对于每个合格的复制集成员，[j](https://docs.mongodb.com/manual/reference/write-concern/#wc-j) 选项确定成员是在内存中应用写操作之后还是在写到磁盘日志上之后是否确认写。

**w: "majority"**

复制集的任何带有数据的投票成员都可以对[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写操作进行写确认。

下表列出了成员何时可以基于[j](https://docs.mongodb.com/manual/reference/write-concern/#wc-j)值确认写入：

| `j`是不确定的 | 确认取决于[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)的值:<br/>如果为true，确认需要对磁盘上的日志进行写入操作(j: true)<br />[`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)默认为true<br />如果为false，确认需要在内存中进行写入操作(j: false)。 |
| :------------ | ------------------------------------------------------------ |
| `j: true`     | 确认需要对磁盘日志进行写入操作。                             |
| `j: false`    | 确认需要在内存中进行写入操作。                               |

> **[success] Note**
> 
> 行为细节,参见 [w: "majority" Behavior](https://docs.mongodb.com/master/reference/write-concern/#wc-majority-behavior).

**w: <`number`>**

复制集的任何承载数据的成员都可以参与[`w：<number>`](https://docs.mongodb.com/manual/reference/write-concern/#wc-w)写操作的写确认。

下表列出了成员何时可以基于[j](https://docs.mongodb.com/manual/reference/write-concern/#wc-j)值确认写入：

| `j` 未指定 | 确认需要在内存中进行写操作(j: false)。 |
| :--------- | -------------------------------------- |
| `j: true`  | 确认需要将操作写入磁盘日志。           |
| `j: false` | 确认需要在内存中进行写操作。           |

> 注意
>
> [隐藏](https://docs.mongodb.com/manual/core/replica-set-hidden-member/#replica-set-hidden-members)， [延迟](https://docs.mongodb.com/manual/core/replica-set-delayed-member/#replica-set-delayed-members)和[优先级为0](https://docs.mongodb.com/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members) 的成员可以确认[`w:<number> `](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)写操作。
>
> 延迟的辅助程序可以不早于配置的slaveDelay返回写确认。

## <span id="问题">因果一致的会话和写问题</span>

使用[因果一致的客户机会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)，客户机会话仅在以下情况下保证因果一致性：

- 相关的读取操作使用[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")读取关注，并且

- 相关的写操作使用[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 写关注。

有关详细信息，请参见[因果一致性](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency)。

#### `w: "majority"` 的行为

* [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault)设置为**false**，MongoDB不等待[`w: "majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority") 写入被写入到磁盘日志之前，确认写入，因此，在给定复制集中的大多数节点出现短暂损失(例如崩溃和重新启动)时，大多数写操作可能会回滚。
* [隐藏的](https://docs.mongodb.com/master/core/replica-set-hidden-member/#replica-set-hidden-members), [延迟的](https://docs.mongodb.com/master/core/replica-set-delayed-member/#replica-set-delayed-members), and [priority 0](https://docs.mongodb.com/master/core/replica-set-priority-0-member/#replica-set-secondary-only-members)的成员，成员数为[n]。投票大于0可以确认[`"majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority") ”写操作。

## <span id="多数">计算关注的多数</span>

> 提示
>
> 从版本4.2.1开始，[`rs.status()`](https://docs.mongodb.com/manual/reference/method/rs.status/#rs.status)返回[`writeMajorityCount`](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/#replSetGetStatus.writeMajorityCount)包含计算出的多数数的 字段。

写入关注的多数[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")由以下值中的较小者计算得出：

- 所有投票成员（包括仲裁员）中的大多数

- 所有**带有数据的**投票成员的数量。

> **[warning] 警告**
>
> 如果计算出的多数数等于所有**带有数据的**投票成员的人数（例如，由3个成员组成的主要-次要仲裁员部署），则写关注 [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")可能会超时，或者如果有数据的投票成员则永远不会得到承认掉线或无法到达。如果可能，请使用带有数据的投票成员而不是仲裁者。

例如，考虑：

- 具有3个投票成员的复制集，主要-次要（PSS）：

  - 所有投票成员中大多数为2。
  - 所有有数据投票的成员人数为3。

计算得出的多数为2，最小值为2和3。写入必须传播到主要对象和辅助对象之一，[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")以向客户端确认写入问题。

- 复制集包含3个投票成员，主要-次要仲裁员（PSA）

  - 所有投票成员中大多数为2。
  - 所有有数据投票的成员人数为2。

计算得出的多数为2，为2和2的最小值。由于该写操作只能应用于数据承载成员，因此该写操作必须传播到主要对象和辅助对象，[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")以向客户端确认写问题。

> 提示
>
> 避免在a (p - a)或其他拓扑结构中使用“多数”写关注点，这些拓扑结构要求所有支持数据的投票成员都可用来确认写操作。想要使用**“majority”**写关注点的持久性保证的客户应该部署不需要所有数据承载投票成员可用的拓扑(例如P-S-S)。



#### 写问题出处

从4.4版本开始，MongoDB跟踪写关注点的来源，它表示一个特定的写关注点的来源。您可能会在[`getLastError`](https://docs.mongodb.com/master/reference/command/serverStatus/#serverstatus.metrics.getLastError) 指标、write concern error对象和MongoDB日志中看到显示出处的信息。

下表显示了可能的写问题**provenance**值及其重要性:

| Provenance             | Description                                                  |
| :--------------------- | :----------------------------------------------------------- |
| `clientSupplied`       | 写关注点是在应用程序中指定的。                               |
| `customDefault`        | 写关注点源自自定义的默认值。参见[`setDefaultRWConcern`](https://docs.mongodb.com/master/reference/command/setDefaultRWConcern/#dbcmd.setDefaultRWConcern). |
| `getLastErrorDefaults` | 写关系起源于复制集的设置[`getLastErrorDefaults`](https://docs.mongodb.com/master/reference/replica-configuration/#rsconf.settings.getLastErrorDefaults)字段。 |
| `implicitDefault`      | 在没有其他所有写关注规范的情况下，写关注源自服务器。         |



译者：杨帅 张琦

校对：杨帅
