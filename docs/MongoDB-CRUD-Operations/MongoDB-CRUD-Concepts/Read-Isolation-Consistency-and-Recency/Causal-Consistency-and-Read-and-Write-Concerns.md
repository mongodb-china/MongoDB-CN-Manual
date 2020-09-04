# 因果一致性和读写问题

通过MongoDB的[因果一致性客户端会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)，读写问题的不同组合可提供不同的 [因果一致性保证](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency-guarantees)。如果定义因果一致性以表示耐久性，则下表列出了各种组合提供的特定保证：

| 阅读关注                                                     | 写关注                                                       | 阅读自己的文章 | 单调读 | 单调写 | 写跟读 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------- | :----- | :----- | :----- |
| [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority") | [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") | ✅              | ✅      | ✅      | ✅      |
| [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority") | [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.) |                | ✅      |        | ✅      |
| [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") | [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.) |                |        |        |        |
| [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local") | [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") |                |        | ✅      |        |

如果因果一致性表示持久性，那么从表中可以看出，只有具有[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")读关注度的读取操作和具有[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")写关注度的写入操作才能保证所有四个因果一致性保证。也就是说， [因果一致的客户端会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)只能保证以下方面的因果一致性：

- [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")关注阅读操作；也就是说，读取操作将返回大多数复制集成员已确认且持久的数据。
- [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")关注写操作；也就是说，写操作要求确认该操作已应用于大多数复制集的有投票权的成员。

如果因果一致性并不意味着持久性(即，写操作可能会回滚)，则具有写顾虑的写操作也可以提供因果一致性。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

> **[success] 注意** 
>
> 在某些情况下(但不一定在所有情况下)，读和写关注点的其他组合也可以满足所有四个因果一致性保证。

读关注点[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")和写关注点 [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")确保即使在复制集中的两个成员*短暂地*认为它们是主要的[情况下(例如，使用网络分区)](https://docs.mongodb.com/manual/core/read-preference-use-cases/#edge-cases)，这四个因果一致性保证也成立 。尽管两个主数据库都可以完成写操作，但是只有一个主数据库能够完成写操作。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority").

例如，考虑网络分区划分五个成员复制集的情况：

![网络分区：一侧选择了新的主节点，而旧的主节点尚未卸任。](https://docs.mongodb.com/manual/_images/network-partition-two-primaries.svg)

<div class="section" id="scenarios">

## 场景

为了说明读写关注点要求，在以下情况下，客户端向客户端发出了一系列操作，并对复制集进行了读写关注点的各种组合：

*   [阅读关注“多数”并写关注“多数”](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-majority-wc-majority)
*   [阅读关注“多数”并发表关注{w：1}](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-majority-wc-1)
*   [阅读关注“本地”，写关注“多数”](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-local-wc-majority)
*   [阅读关注“本地”并写关注{w：1}](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-local-wc-1)

### 阅读关注`"majority"`和写关注`"majority"`


在因果一致的会话中使用读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")和写入关注 [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")可提供以下因果一致性保证：

✅自己读✅单调读read单调写✅写跟随读

##### 方案1（读关注`"majority"`和写关注`"majority"`）

在具有两个主操作的过渡期内，由于只有**P**<sub>new</sub>操作才能满足写关注的写操作，因此客户机会话可以成功发出以下操作序列：[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 到 新的关注**P**<sub>new</sub><br/>2.read<sub>1</sub>>与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>2</sub><br/>3.write<sub>2</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")到新的关注**P**<sub>new</sub><br/> 4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

![使用读关注多数和写关注多数的具有两个原语的数据状态](https://docs.mongodb.com/manual/_images/causal-rc-majority-wc-majority.svg)

| ✅ **自己写**   | read<sub>1</sub>从**S**<sub>2</sub>读取数据，该数据反映了write<sub>1</sub>之后的状态。read<sub>2</sub>从**S**<sub>1</sub>读取数据，该数据反映了write<sub>1</sub>之后是write<sub>2</sub>之后的状态。 |
| -------------- | ------------------------------------------------------------ |
| ✅ **单调读**   | read<sub>2</sub>从**S**<sub>3</sub>中读取反映read<sub>1</sub>之后状态的数据。 |
| ✅ **单调写**   | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映write<sub>1</sub>之后的状态。 |
| ✅ **写跟随读** | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的数据状态（即，较早的状态反映read<sub>1</sub>读取的数据）。 |

##### 方案2（读取关注“多数”和写入关注“多数”）

考虑一个替代序列，其中具有读关注的read<sub>1</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")路由到`S`1：

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 到 新的关注**P**<sub>new</sub><br/>2.read<sub>1</sub>>与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>2</sub><br/>3.write<sub>2</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")到新的关注**P**<sub>new</sub><br/> 4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

在这个序列中，read<sub>1</sub>在**P**<sub>old</sub>上的多数提交点提前之前不能返回。在**P**<sub>old</sub>和**S**<sub>1</sub>能够与复制集的其余部分通信之前，这是不可能发生的;此时，**P**<sub>old</sub>已经退出(如果还没有)，两个成员从副本集中的其他成员同步(包括write<sub>1</sub>)。

| ✅ **自己写                      ** | read<sub>1</sub>反映了write1<sub>1</sub>之后的数据状态，尽管在网络分区已修复并且该成员已与副本集的其他成员进行同步之后。read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映了write1<sub>1</sub>之后是write<sub>2</sub>之后的状态。 |
| ---------------------------------- | ------------------------------------------------------------ |
| ✅ **单调读**                       | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映read<sub>1</sub>之后的状态（即，较早的状态反映在read<sub>1</sub>读取的数据中）。 |
| ✅ **单调写**                       | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映write<sub>1</sub>之后的状态。 |
| ✅ **写跟随读**                     | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的数据状态（即，较早的状态反映read<sub>1</sub>读取的数据）。 |

#### 读关注`"majority"`和写关注`{w: 1}`

*如果因果一致性暗示持久性，则*在因果一致性会话中使用读关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")和写关注 可提供以下因果一致性保证：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

❌自己读   ✅单调读read单调写.   ✅写跟随读

*如果因果一致性并不意味着持久性*：

✅自己读.  ✅单调读read单调写.   ✅写跟随读

##### 方案3（“关注多数”和“关注关注” ）`{w: 1}`

在过渡期内有两个初选，因为无论**P**<sub>old</sub>与**P**<sub>new</sub>能满足与写入 的写入关注，一个客户端会话可以成功地发出以下的操作序列，但不是因果关系一致**，如果一致因果意味着耐久性**：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>与写入关注 到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>old</sub><br/>2.read<sub>1</sub>1与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>2</sub><br/>3.write<sub>2</sub>到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>new</sub> <br/>4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

![使用读关注多数和写关注1的具有两个原语的数据状态](https://docs.mongodb.com/manual/_images/causal-rc-majority-wc-1.svg)

按照这个顺序

- 直到**P**<sub>new</sub>上的大多数提交点超过了write<sub>1</sub>的时间，read<sub>1</sub>才会返回。
- 直到**P**<sub>new</sub>上的大多数提交点超过了write<sub>2</sub>的时间，read<sub>2</sub>才能返回。
- 当网络分区恢复时，write<sub>1</sub>将回滚。

➤ *如果因果一致性意味着持久性*

| ❌ **自己写**   | read<sub>1</sub>从**S**<sub>2</sub>读取的数据不反映write<sub>1</sub>之后的状态。 |
| -------------- | ------------------------------------------------------------ |
| ✅ **单调读**   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映read<sub>1</sub>之后的状态（即，较早的状态反映在read<sub>1</sub>读取的数据中）。 |
| ❌ **单调写**   | write<sub>2</sub>更新了**P**<sub>new</sub>数据，而不会反映write<sub>1</sub>之后的状态。 |
| ✅ **写跟随读** | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的状态（即，较早的状态反映read<sub>1</sub>读取的数据）。 |

➤ *如果因果一致性并不意味着持久性*

| ✅ **自己写**   | read<sub>1</sub>从**S**<sub>2</sub>读取数据，返回反映与write<sub>1</sub>等效的状态的数据，然后回退write<sub>1</sub>。 |
| -------------- | ------------------------------------------------------------ |
| ✅ **单调读**   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映read<sub>1</sub>之后的状态（即，较早的状态反映在read<sub>1</sub>读取的数据中）。 |
| ✅ **单调写**   | write<sub>2</sub>更新了**P**<sub>new</sub>的数据，这等效于write<sub>1</sub>之后回退写1的数据。 |
| ✅ **写跟随读** | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的状态（即，较早的状态反映read<sub>1</sub>读取的数据）。 |

##### 方案4（“关注多数”和“关注关注” ）`{w: 1}`

考虑一个替代序列，其中具有读关注的读1[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")路由到`S`1：

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>与写入关注 到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>old</sub><br/>2.read<sub>1</sub>1与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>1</sub><br/>3.write<sub>2</sub>到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>new</sub> <br/>4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

按此顺序：

* 直到**S**<sub>1</sub>上的大多数提交点提高，read<sub>1</sub>才能返回。在**P**<sub>old</sub>和**S**<sub>1</sub>能够与复制集的其他成员进行通信之前，这是不可能发生的。此时，**P**<sub>old</sub>已经退出(如果还没有)，write<sub>1</sub>将从**P**<sub>old</sub>和**S**<sub>1</sub>回滚，两个成员将与复制集的其他成员同步。

➤ *如果因果一致性意味着持久性*

| ❌ **自己写**   | read<sub>1</sub>读取的数据不反映已回退的write<sub>1</sub>的结果。 |
| -------------- | ------------------------------------------------------------ |
| ✅ **单调读**   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映read<sub>1</sub>之后的状态（即，其较早的状态反映read<sub>1</sub>读取的数据）。 |
| ❌ **单调写**   | write<sub>2</sub>更新关于**P**<sub>new</sub>的数据，该数据不反映write<sub>1</sub>之后的状态，该write<sub>1</sub>在write<sub>2</sub>之前但已回滚。 |
| ✅ **写跟随读** | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的状态（即，其较早的状态反映read<sub>1</sub>读取的数据）。 |

➤ *如果因果一致性并不意味着持久性*

| ✅ **自己写**   | read<sub>1</sub>返回反映write<sub>1</sub>最终结果的数据，因为write<sub>1</sub>最终会回滚。 |
| -------------- | ------------------------------------------------------------ |
| ✅ **单调读**   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据反映read<sub>1</sub>之后的状态（即，其较早的状态反映read<sub>1</sub>读取的数据）。 |
| ✅ **单调写**   | write<sub>2</sub>更新**P**<sub>new</sub>上的数据，这等效于write<sub>1</sub>之后回退write<sub>1</sub>的数据。 |
| ✅ **写跟随读** | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映read<sub>1</sub>之后的状态（即，其较早的状态反映read<sub>1</sub>读取的数据）。 |

#### 读关注`"local"`和写关注`{w: 1}`

在因果一致的会话中使用读关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local")和写关注 不能保证因果一致性。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

❌自己读.    ❌单调读read单调写.    ❌写跟随读

在某些情况下（但不一定在所有情况下），此组合可以满足所有四个因果一致性保证。

##### 方案5（“本地关注”和“关注关注” ）`{w: 1}`

在这个短暂的时期，因为无论**P**<sub>old</sub>与 **P**<sub>new</sub>能满足与写入的写入关注，一个客户端会话可以发出以下的操作序列成功，但不是因果关系是一致的：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>与写入关注到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>old</sub><br/>2.read<sub>1</sub>1与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>1</sub><br/>3.write<sub>2</sub>到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**<sub>new</sub> <br/>4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

![使用读关注本地和写关注1的具有两个主数据的数据状态](https://docs.mongodb.com/manual/_images/causal-rc-local-wc-1.svg)

| ❌自己写   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据仅反映write<sub>2</sub>之后的状态，而不反映write<sub>1</sub> 之后是write<sub>2</sub>的状态。 |
| --------- | ------------------------------------------------------------ |
| ❌单调读   | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据不反映read<sub>1</sub>之后的状态（即，较早的状态不反映read<sub>1</sub>读取的数据）。 |
| ❌单调写   | write<sub>2</sub>更新了**P**<sub>new</sub>数据，而不会反映write<sub>1</sub>之后的状态。 |
| ❌写跟随读 | write<sub>2</sub>更新**P**<sub>new</sub>的数据，该数据不反映read<sub>1</sub>之后的状态（即，较早的状态不反映read<sub>1</sub>读取的数据）。 |

#### 读关注`"local"`和写关注`"majority"`

在因果一致的会话中使用读取关注[`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local")和写入关注 [`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")可提供以下因果一致性保证：

❌自己读     ❌单调读read单调写     ❌写跟随读

在某些情况下（但不一定在所有情况下），此组合可以满足所有四个因果一致性保证。

##### 方案6（“关注本地”和“关注多数”）

在此过渡期间，因为只有`P`new才能完成与 写入有关的写入，所以客户机会话可以成功发出以下操作序列，但因果关系不一致：[`{ w: "majority" }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")

| 序列                                                         | 例                                                           |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1.write<sub>1</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority") 到 新的关注**P**<sub>new</sub><br/>2.read<sub>1</sub>>与读关心[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>1</sub><br/>3.write<sub>2</sub>[`"majority"`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority")到新的关注**P**<sub>new</sub><br/> 4.read<sub>2</sub>与读取关注[`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority")到**S**<sub>3</sub> | 对于项目`A`，更新`qty`为`50`。<br/>阅读项目`A`。对于`qty`小于或等于的项目`50`，<br/>更新`restock`到`true`。阅读项目`A`。 |

![使用读关注本地和写关注多数的两个主数据的状态](https://docs.mongodb.com/manual/_images/causal-rc-local-wc-majority.svg)

| ❌阅读自己的文章。 | read<sub>1</sub>从**S**<sub>1</sub>读取不反映write1<sub>1</sub>后状态的数据。 |
| ----------------- | ------------------------------------------------------------ |
| ❌单调读。         | read<sub>2</sub>从**S**<sub>3</sub>读取数据，该数据不反映read<sub>1</sub>之后的状态（即，较早的状态不反映read<sub>1</sub>读取的数据）。 |
| ✅单调写           | write<sub>2</sub>更新**P**<sub>new</sub>数据，以反映write<sub>1</sub>之后的状态。 |
| ❌写跟随阅读。     | write<sub>2</sub>更新**P**<sub>new</sub>的数据，该数据不反映read<sub>1</sub>之后的状态（即，较早的状态不反映read<sub>1</sub>读取的数据）。 |



译者：杨帅

校对：杨帅