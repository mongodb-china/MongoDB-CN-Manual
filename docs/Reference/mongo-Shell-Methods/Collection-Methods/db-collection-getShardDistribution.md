# [ ](#)db.collection.getShardDistribution（）

[]()

在本页面

*   [定义](#definition)

*   [输出](#output)

## <span id="definition">定义</span>

*   `db.collection.`  `getShardDistribution` ()

       *   打印分片集合的数据分布统计信息。

> **建议**
>
> 在运行方法之前，使用flushRouterConfig命令刷新缓存的路由 table，以避免返回集合的陈旧分发信息。刷新后，run db.collection.getShardDistribution()为您希望 build 索引的集合。

例如：

```powershell
db.adminCommand( { flushRouterConfig: "test.myShardedCollection" } );
db.getSiblingDB("test").myShardedCollection.getShardDistribution();
```

> **也可以看看**
>
> 分片

## <span id="output">输出</span>

### Sample 输出

以下是分片集合分布的 sample 输出：

```powershell
Shard shard-a at shard-a/MyMachine.local:30000,MyMachine.local:30001,MyMachine.local:30002
data : 38.14Mb docs : 1000003 chunks : 2
estimated data per chunk : 19.07Mb
estimated docs per chunk : 500001

Shard shard-b at shard-b/MyMachine.local:30100,MyMachine.local:30101,MyMachine.local:30102
data : 38.14Mb docs : 999999 chunks : 3
estimated data per chunk : 12.71Mb
estimated docs per chunk : 333333

Totals
data : 76.29Mb docs : 2000002 chunks : 5
Shard shard-a contains 50% data, 50% docs in cluster, avg obj size on shard : 40b
Shard shard-b contains 49.99% data, 49.99% docs in cluster, avg obj size on shard : 40b
```

### 输出字段

```powershell
Shard <shard-a> at <host-a>
 data : <size-a> docs : <count-a> chunks : <number of chunks-a>
 estimated data per chunk : <size-a>/<number of chunks-a>
 estimated docs per chunk : <count-a>/<number of chunks-a>

Shard <shard-b> at <host-b>
 data : <size-b> docs : <count-b> chunks : <number of chunks-b>
 estimated data per chunk : <size-b>/<number of chunks-b>
 estimated docs per chunk : <count-b>/<number of chunks-b>

Totals
 data : <stats.size> docs : <stats.count> chunks : <calc total chunks>
 Shard <shard-a> contains  <estDataPercent-a>% data, <estDocPercent-a>% docs in cluster, avg obj size on shard : stats.shards[ <shard-a> ].avgObjSize
 Shard <shard-b> contains  <estDataPercent-b>% data, <estDocPercent-b>% docs in cluster, avg obj size on shard : stats.shards[ <shard-b> ].avgObjSize
```

输出信息显示：

* `<shard-x>`是一个包含分片 name 的 string。

* `<host-x>`是一个包含 host name(s 的 string。

* `<size-x>`是一个包含数据大小的数字，包括度量单位(如： `b`，`Mb`)。

* `<count-x>`是一个报告分片中文档数量的数字。

* `<number of chunks-x>`是一个报告分片中块数的数字。

* `<size-x>/<number of chunks-x>`是计算的 value，它反映了分片的每个块的估计数据大小，包括度量单位(如： `b`，`Mb`)。

* `<count-x>/<number of chunks-x>`是计算出的 value，它反映了碎片每个块的估计文档数。

* `<stats.size>`是一个 value，用于报告分片集合中数据的总大小，包括度量单位。

* `<stats.count>`是一个 value，用于报告分片集合中的文档总数。

*   `<calc total chunks>`是一个计算出的数字，用于报告所有分片的块数，例如：
    ```powershell
    <calc total chunks> = <number of chunks-a> + <number of chunks-b>
    ```

*   `<estDataPercent-x>`是一个计算的 value，对于每个分片，数据大小反映为集合总数据大小的百分比，对于 example：
    
	```powershell
	<estDataPercent-x> = <size-x>/<stats.size>
	```

*   `<estDocPercent-x>`是一个计算的 value，对于每个分片，它反映了文档的数量，作为集合的文档总数的百分比，对于 example：
    
	```powershell
   <estDocPercent-x> = <count-x>/<stats.count>
	```

*   `stats.shards[ <shard-x> ].avgObjSize`是反映分片的平均 object 大小(包括度量单位)的数字。



译者：李冠飞

校对：