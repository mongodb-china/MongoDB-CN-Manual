## 默认的MongoDB读/写关注

**在本页面**

- [阅读关注](#读)
- [写关注](#写)
- [因果一致性保证](#因果)

### 阅读关注

![Read/Write Concern Inheritance](https://docs.mongodb.com/master/_images/read-write-concern-inheritance.bakedsvg.svg)

#### 默认读取问题

**默认的** [`read concern`](https://docs.mongodb.com/master/reference/read-concern/)如下:

| 操作                                                         | 默认读取问题                                                 |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 针对主要的读取                                               | [` "local" `](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local")<br />注意，这个读取的关注点可以返回可能被回滚的数据。此读取关注点**不**保证[因果一致性](https://docs.mongodb.com/master/core/read-isol-consist-recency/ #sessions)。 |
| 如果读操作与[因果一致的会话](https://docs.mongodb.com/master/core/read-isol-consistic-recency/#sessions)相关联，则读取二级会话。 | [`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local")<br />注意<br />这个读关注可以返回可能回滚的数据。<br />此读取关注点**不**保证[因果一致性](https://docs.mongodb.com/master/core/read-isol-consist-recency/ #sessions)。 |
| 如果读操作与[因果一致的会话](https://docs.mongodb.com/master/core/read-isol-consistency -recency/#sessions)没有关联，则对辅助会话进行读取。 | [`"available"`](https://docs.mongodb.com/master/reference/read-concern-available/#readconcern."available")<br />注意<br />这个读关注可以返回可能回滚的数据<br />这种已读关注点**不能**保证[因果关系的一致性](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#sessions)。<br />对于分片集合，这个读关注也可以返回**孤立的文档**。 |

#### 指定读取关注:MongoDB驱动程序

- **外部事务操作**

> 注意
>
> 以下内容适用于在事务外部发出的操作。
>
> 要阅读与事务内部发出的操作相关的关注信息，请单击事务中的操作选项卡。

使用[MongoDB 驱动](https://docs.mongodb.com/drivers/)，您可以覆盖默认的[read concern](https://docs.mongodb.com/master/reference/read-concern/)，并设置以下级别的操作的read concern:

| 水平       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| 客户级别   | 应用于操作，除非在数据库/集合/操作级别设置了更细致的读取关注。否则将应用于操作。 |
| 数据库级别 | 应用于数据库集合上的操作(即覆盖客户端读关注)，除非已在集合级别或操作级别设置了读关注。<br />注意：<br />不适用于事务内部的操作。 |
| 集合级别   | 应用对集合的读操作(即覆盖数据库/客户端读关注)，除非已在操作级别设置了读关注。<br />注意：<br />不适用于事务内部的操作。 |
| 操作级别   | 应用特定的读操作(例如，覆盖数据库/客户端/集合读关注)。在操作中设置read concern的能力取决于驱动程序。请参考您的[驱动程序文档](https://docs.mongodb.com/drivers/)。<br/>注意：<br />不适用于事务内部的操作。 |

* **事务中的操作**

> 注意
>
> 以下内容适用于在事务内部发出的操作。
>
> 要阅读与发出外部事务的操作相关的关注信息，请单击“外部事务的操作”选项卡。

使用[MongoDB 驱动](https://docs.mongodb.com/drivers/)，您可以覆盖默认的[read concern](https://docs.mongodb.com/master/reference/read-concern/)，并设置以下级别的操作的read concern:

| 水平     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| 客户级别 | 应用于事务，除非在会话/事务级别设置了更细致的读取关注。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。 |
| 会话级别 | 应用于在会话中启动的事务(即覆盖客户端读取关注)，除非在特定事务级别上设置了更细致的读取关注级别。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。<br />有关更多信息，请参阅[事务的阅和读关注](https://docs.mongodb.com/master/core/transactions/#transactions-read-concern)。 |
| 事务级别 | 应用于特定的事务。<br />事务写关注应用于提交操作和事务内部的操作。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。 |

#### 额外的信息

有关可用的读取关注点的更多信息，请参见[read Concern](https://docs.mongodb.com/master/reference/read-concern/)。

### <span id="写">写关注</span>

![Read/Write Concern Inheritance](https://docs.mongodb.com/master/_images/read-write-concern-inheritance.bakedsvg.svg)

#### 默认写问题

**默认的** [write concern](https://docs.mongodb.com/master/reference/writconcern/)是 **w: 1** 。

> 请注意
>
> * 使用默认的写关注，数据可以回滚。
> * 此写关注点**不**保证[因果一致性](https://docs.mongodb.com/master/core/read-isol-consist-recency/ #sessions)。

#### 指定写关注:MongoDB驱动程序

- **外部事务操作**

> 注意
>
> 以下内容适用于在[transactions](https://docs.mongodb.com/master/core/transactions/)外部发出的操作。
>
> 要阅读与事务内部发布的操作相关的关注信息，请单击“事务中的操作”选项卡。

使用[MongoDB drivers](https://docs.mongodb.com/drivers/)，您可以覆盖默认的[write concern](https://docs.mongodb.com/master/reference/writconcern/)，并在以下级别设置操作的write concern:

| Level      | Description                                                  |
| :--------- | :----------------------------------------------------------- |
| 客户级别   | 除非在操作/数据库/集合中为操作设置了更细的写关注点，否则将应用于操作。 |
| 数据库级别 | 应用于数据库集合上的写操作(即覆盖客户端写关注点)，除非在集合级别或操作级别上设置了写关注点<br />注意<br />不适用于事务内部的操作。 |
| 集合级别   | 应用于集合上的写操作(即覆盖数据库和客户端写关注点)，除非在操作级别上设置了写关注点。<br />注意<br/>不适用于事务内部的操作。 |
| 操作级别   | 应用于特定的写操作。在操作中设置写关注点的能力取决于驱动程序。请参考您的[驱动程序文档](https://docs.mongodb.com/drivers/)。<br/>注意<br />不适用于事务内部的操作。 |

* **事务中的操作**

> 注意
>
> 以下内容适用于在事务内部发出的操作。
>
> 要阅读与发出外部事务的操作相关的关注信息，请单击“外部事务的操作”选项卡。

| 水平     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| 客户级别 | 应用于事务，除非在会话/事务级别设置了更细致的读取关注。<br />事务写关注点适用于提交操作和事务内部的操作。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。 |
| 会话级别 | 应用于在会话中启动的事务(即覆盖客户端读取关注)，除非在特定事务级别上设置了更细致的读取关注级别。<br />事务写关注点适用于提交操作和事务内部的操作。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。<br />有关更多信息，请参阅[事务的阅和读关注](https://docs.mongodb.com/master/core/transactions/#transactions-read-concern)。 |
| 事务级别 | 应用于特定的事务。<br />事务写关注应用于提交操作和事务内部的操作。<br />注意<br/>事务中的所有操作都使用事务读关注;即,在事务内部忽略在操作/集合/数据库级别设置的任何读关注。 |

使用MongoDB驱动程序，你可以覆盖默认的写关注和设置写关注为以下级别的事务:

#### 额外的信息

有关可用的写关注点的更多信息，请参见[写关注点](https://docs.mongodb.com/master/reference/writconcern/)。

### <span id="因果">因果一致性的保证</span>

使用[因果一致的客户端会话](https://docs.mongodb.com/master/core/read-isol-consist-recency/ #sessions)，客户端会话仅在以下情况下保证因果一致:

- 相关的读取操作使用[ "majority" ](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority")读取concern，
- 相关的写操作使用[ "majority" ](https://docs.mongodb.com/master/reference/writeconcern /#writeconcern."majority")写关注。