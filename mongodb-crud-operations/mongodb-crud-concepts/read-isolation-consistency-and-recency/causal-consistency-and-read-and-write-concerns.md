# 因果一致性和读写关注

通过MongoDB的[因果一致性客户端会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)，读写问题的不同组合可提供不同的 [因果一致性保证](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency-guarantees)。如果定义因果一致性以表示耐久性，则下表列出了各种组合提供的特定保证：

| 阅读关注 | 写关注 | 阅读自己的文章 | 单调读 | 单调写 | 写跟读 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority)"\) | \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\) | ✅ | ✅ | ✅ | ✅ |
| \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority)"\) | [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.) |  | ✅ |  | ✅ |
| \[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local)"\) | [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.) |  |  |  |  |
| \[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local)"\) | \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\) |  |  | ✅ |  |

如果因果一致性表示持久性，那么从表中可以看出，只有具有\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)读关注度的读取操作和具有\[\`"majority"\`\]\(https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)写关注度的写入操作才能保证所有四个因果一致性保证。也就是说，](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29读关注度的读取操作和具有[`"majority"`]%28https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29写关注度的写入操作才能保证所有四个因果一致性保证。也就是说，) [因果一致的客户端会话](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions)只能保证以下方面的因果一致性：

* \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)关注阅读操作；也就是说，读取操作将返回大多数复制集成员已确认且持久的数据。](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29关注阅读操作；也就是说，读取操作将返回大多数复制集成员已确认且持久的数据。)
* \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)关注写操作；也就是说，写操作要求确认该操作已应用于大多数复制集的有投票权的成员。](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29关注写操作；也就是说，写操作要求确认该操作已应用于大多数复制集的有投票权的成员。)

如果因果一致性并不意味着持久性\(即，写操作可能会回滚\)，则具有写顾虑的写操作也可以提供因果一致性。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

> **\[success\] 注意**
>
> 在某些情况下\(但不一定在所有情况下\)，读和写关注点的其他组合也可以满足所有四个因果一致性保证。

读关注点\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)和写关注点](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29和写关注点) \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)确保即使在复制集中的两个成员\*短暂地\*认为它们是主要的\[情况下\(例如，使用网络分区\)\]\(https://docs.mongodb.com/manual/core/read-preference-use-cases/\#edge-cases\)，这四个因果一致性保证也成立](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29确保即使在复制集中的两个成员*短暂地*认为它们是主要的[情况下%28例如，使用网络分区%29]%28https://docs.mongodb.com/manual/core/read-preference-use-cases/#edge-cases%29，这四个因果一致性保证也成立) 。尽管两个主数据库都可以完成写操作，但是只有一个主数据库能够完成写操作。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\).

例如，考虑网络分区划分五个成员复制集的情况：

![&#x7F51;&#x7EDC;&#x5206;&#x533A;&#xFF1A;&#x4E00;&#x4FA7;&#x9009;&#x62E9;&#x4E86;&#x65B0;&#x7684;&#x4E3B;&#x8282;&#x70B9;&#xFF0C;&#x800C;&#x65E7;&#x7684;&#x4E3B;&#x8282;&#x70B9;&#x5C1A;&#x672A;&#x5378;&#x4EFB;&#x3002;](https://docs.mongodb.com/manual/_images/network-partition-two-primaries.svg)

## 场景

为了说明读写关注点要求，在以下情况下，客户端向客户端发出了一系列操作，并对复制集进行了读写关注点的各种组合：

* [阅读关注“多数”并写关注“多数”](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-majority-wc-majority)
* [阅读关注“多数”并发表关注{w：1}](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-majority-wc-1)
* [阅读关注“本地”，写关注“多数”](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-local-wc-majority)
* [阅读关注“本地”并写关注{w：1}](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/#causal-rc-local-wc-1)

### 阅读关注`"majority"`和写关注`"majority"`

在因果一致的会话中使用读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)和写入关注](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29和写入关注) \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)可提供以下因果一致性保证：](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29可提供以下因果一致性保证：)

✅自己读✅单调读read单调写✅写跟随读

**方案1（读关注"majority"和写关注"majority"）**

在具有两个主操作的过渡期内，由于只有**P**new操作才能满足写关注的写操作，因此客户机会话可以成功发出以下操作序列：\[`{ w: "majority" }`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\)

| 序列 | 例 |
| :--- | :--- |
| 1.write1\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\) 到 新的关注**P**new 2.read1&gt;与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)2 3.write2\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)到新的关注\*\*P\*\*](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29到新的关注**P**)new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

![&#x4F7F;&#x7528;&#x8BFB;&#x5173;&#x6CE8;&#x591A;&#x6570;&#x548C;&#x5199;&#x5173;&#x6CE8;&#x591A;&#x6570;&#x7684;&#x5177;&#x6709;&#x4E24;&#x4E2A;&#x539F;&#x8BED;&#x7684;&#x6570;&#x636E;&#x72B6;&#x6001;](https://docs.mongodb.com/manual/_images/causal-rc-majority-wc-majority.svg)

| ✅ **自己写** | read1从**S**2读取数据，该数据反映了write1之后的状态。read2从**S**1读取数据，该数据反映了write1之后是write2之后的状态。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3中读取反映read1之后状态的数据。 |
| ✅ **单调写** | write2更新**P**new数据，以反映write1之后的状态。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的数据状态（即，较早的状态反映read1读取的数据）。 |

**方案2（读取关注“多数”和写入关注“多数”）**

考虑一个替代序列，其中具有读关注的read1\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)路由到\`S\`1：](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29路由到`S`1：)

| 序列 | 例 |
| :--- | :--- |
| 1.write1\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\) 到 新的关注**P**new 2.read1&gt;与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)2 3.write2\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)到新的关注\*\*P\*\*](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29到新的关注**P**)new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

在这个序列中，read1在**P**old上的多数提交点提前之前不能返回。在**P**old和**S**1能够与复制集的其余部分通信之前，这是不可能发生的;此时，**P**old已经退出\(如果还没有\)，两个成员从副本集中的其他成员同步\(包括write1\)。

| ✅ **自己写**                       | read1反映了write11之后的数据状态，尽管在网络分区已修复并且该成员已与副本集的其他成员进行同步之后。read2从**S**3读取数据，该数据反映了write11之后是write2之后的状态。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3读取数据，该数据反映read1之后的状态（即，较早的状态反映在read1读取的数据中）。 |
| ✅ **单调写** | write2更新**P**new数据，以反映write1之后的状态。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的数据状态（即，较早的状态反映read1读取的数据）。 |

#### 读关注`"majority"`和写关注`{w: 1}`

_如果因果一致性暗示持久性，则_在因果一致性会话中使用读关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)和写关注](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29和写关注) 可提供以下因果一致性保证：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

❌自己读 ✅单调读read单调写. ✅写跟随读

_如果因果一致性并不意味着持久性_：

✅自己读. ✅单调读read单调写. ✅写跟随读

**方案3（“关注多数”和“关注关注” ）{w: 1}**

在过渡期内有两个初选，因为无论**P**old与**P**new能满足与写入 的写入关注，一个客户端会话可以成功地发出以下的操作序列，但不是因果关系一致**，如果一致因果意味着耐久性**：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

| 序列 | 例 |
| :--- | :--- |
| 1.write1与写入关注 到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**old 2.read11与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)2 3.write2到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

![&#x4F7F;&#x7528;&#x8BFB;&#x5173;&#x6CE8;&#x591A;&#x6570;&#x548C;&#x5199;&#x5173;&#x6CE8;1&#x7684;&#x5177;&#x6709;&#x4E24;&#x4E2A;&#x539F;&#x8BED;&#x7684;&#x6570;&#x636E;&#x72B6;&#x6001;](https://docs.mongodb.com/manual/_images/causal-rc-majority-wc-1.svg)

按照这个顺序

* 直到**P**new上的大多数提交点超过了write1的时间，read1才会返回。
* 直到**P**new上的大多数提交点超过了write2的时间，read2才能返回。
* 当网络分区恢复时，write1将回滚。

➤ _如果因果一致性意味着持久性_

| ❌ **自己写** | read1从**S**2读取的数据不反映write1之后的状态。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3读取数据，该数据反映read1之后的状态（即，较早的状态反映在read1读取的数据中）。 |
| ❌ **单调写** | write2更新了**P**new数据，而不会反映write1之后的状态。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的状态（即，较早的状态反映read1读取的数据）。 |

➤ _如果因果一致性并不意味着持久性_

| ✅ **自己写** | read1从**S**2读取数据，返回反映与write1等效的状态的数据，然后回退write1。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3读取数据，该数据反映read1之后的状态（即，较早的状态反映在read1读取的数据中）。 |
| ✅ **单调写** | write2更新了**P**new的数据，这等效于write1之后回退写1的数据。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的状态（即，较早的状态反映read1读取的数据）。 |

**方案4（“关注多数”和“关注关注” ）{w: 1}**

考虑一个替代序列，其中具有读关注的读1\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)路由到\`S\`1：](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29路由到`S`1：)

| 序列 | 例 |
| :--- | :--- |
| 1.write1与写入关注 到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**old 2.read11与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)1 3.write2到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

按此顺序：

* 直到**S**1上的大多数提交点提高，read1才能返回。在**P**old和**S**1能够与复制集的其他成员进行通信之前，这是不可能发生的。此时，**P**old已经退出\(如果还没有\)，write1将从**P**old和**S**1回滚，两个成员将与复制集的其他成员同步。

➤ _如果因果一致性意味着持久性_

| ❌ **自己写** | read1读取的数据不反映已回退的write1的结果。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3读取数据，该数据反映read1之后的状态（即，其较早的状态反映read1读取的数据）。 |
| ❌ **单调写** | write2更新关于**P**new的数据，该数据不反映write1之后的状态，该write1在write2之前但已回滚。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的状态（即，其较早的状态反映read1读取的数据）。 |

➤ _如果因果一致性并不意味着持久性_

| ✅ **自己写** | read1返回反映write1最终结果的数据，因为write1最终会回滚。 |
| :--- | :--- |
| ✅ **单调读** | read2从**S**3读取数据，该数据反映read1之后的状态（即，其较早的状态反映read1读取的数据）。 |
| ✅ **单调写** | write2更新**P**new上的数据，这等效于write1之后回退write1的数据。 |
| ✅ **写跟随读** | write2更新**P**new数据，以反映read1之后的状态（即，其较早的状态反映read1读取的数据）。 |

#### 读关注`"local"`和写关注`{w: 1}`

在因果一致的会话中使用读关注\[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local"\)和写关注](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local"%29和写关注) 不能保证因果一致性。[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

❌自己读. ❌单调读read单调写. ❌写跟随读

在某些情况下（但不一定在所有情况下），此组合可以满足所有四个因果一致性保证。

**方案5（“本地关注”和“关注关注” ）{w: 1}**

在这个短暂的时期，因为无论**P**old与 **P**new能满足与写入的写入关注，一个客户端会话可以发出以下的操作序列成功，但不是因果关系是一致的：[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)

| 序列 | 例 |
| :--- | :--- |
| 1.write1与写入关注到 [`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**old 2.read11与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)1 3.write2到新的关注[`{ w: 1 }`](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern.)**P**new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

![&#x4F7F;&#x7528;&#x8BFB;&#x5173;&#x6CE8;&#x672C;&#x5730;&#x548C;&#x5199;&#x5173;&#x6CE8;1&#x7684;&#x5177;&#x6709;&#x4E24;&#x4E2A;&#x4E3B;&#x6570;&#x636E;&#x7684;&#x6570;&#x636E;&#x72B6;&#x6001;](https://docs.mongodb.com/manual/_images/causal-rc-local-wc-1.svg)

| ❌自己写 | read2从**S**3读取数据，该数据仅反映write2之后的状态，而不反映write1 之后是write2的状态。 |
| :--- | :--- |
| ❌单调读 | read2从**S**3读取数据，该数据不反映read1之后的状态（即，较早的状态不反映read1读取的数据）。 |
| ❌单调写 | write2更新了**P**new数据，而不会反映write1之后的状态。 |
| ❌写跟随读 | write2更新**P**new的数据，该数据不反映read1之后的状态（即，较早的状态不反映read1读取的数据）。 |

#### 读关注`"local"`和写关注`"majority"`

在因果一致的会话中使用读取关注\[`"local"`\]\([https://docs.mongodb.com/manual/reference/read-concern-local/\#readconcern."local"\)和写入关注](https://docs.mongodb.com/manual/reference/read-concern-local/#readconcern."local"%29和写入关注) \[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)可提供以下因果一致性保证：](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29可提供以下因果一致性保证：)

❌自己读 ❌单调读read单调写 ❌写跟随读

在某些情况下（但不一定在所有情况下），此组合可以满足所有四个因果一致性保证。

**方案6（“关注本地”和“关注多数”）**

在此过渡期间，因为只有`P`new才能完成与 写入有关的写入，所以客户机会话可以成功发出以下操作序列，但因果关系不一致：\[`{ w: "majority" }`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\)

| 序列 | 例 |
| :--- | :--- |
| 1.write1\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority)"\) 到 新的关注**P**new 2.read1&gt;与读关心\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)1 3.write2\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/write-concern/\#writeconcern."majority"\)到新的关注\*\*P\*\*](https://docs.mongodb.com/manual/reference/write-concern/#writeconcern."majority"%29到新的关注**P**)new  4.read2与读取关注\[`"majority"`\]\([https://docs.mongodb.com/manual/reference/read-concern-majority/\#readconcern."majority"\)到\*\*S\*\*](https://docs.mongodb.com/manual/reference/read-concern-majority/#readconcern."majority"%29到**S**)3 | 对于项目`A`，更新`qty`为`50`。 阅读项目`A`。对于`qty`小于或等于的项目`50`， 更新`restock`到`true`。阅读项目`A`。 |

![&#x4F7F;&#x7528;&#x8BFB;&#x5173;&#x6CE8;&#x672C;&#x5730;&#x548C;&#x5199;&#x5173;&#x6CE8;&#x591A;&#x6570;&#x7684;&#x4E24;&#x4E2A;&#x4E3B;&#x6570;&#x636E;&#x7684;&#x72B6;&#x6001;](https://docs.mongodb.com/manual/_images/causal-rc-local-wc-majority.svg)

| ❌阅读自己的文章。 | read1从**S**1读取不反映write11后状态的数据。 |
| :--- | :--- |
| ❌单调读。 | read2从**S**3读取数据，该数据不反映read1之后的状态（即，较早的状态不反映read1读取的数据）。 |
| ✅单调写 | write2更新**P**new数据，以反映write1之后的状态。 |
| ❌写跟随阅读。 | write2更新**P**new的数据，该数据不反映read1之后的状态（即，较早的状态不反映read1读取的数据）。 |

译者：杨帅

校对：杨帅

