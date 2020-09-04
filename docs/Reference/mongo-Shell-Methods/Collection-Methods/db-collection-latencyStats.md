# [ ](#)db.collection.latencyStats（）

[]()

在本页面

*   [定义](#definition)

*   [输出](#output)

*   [例子](#examples)


## <span id="definition">定义</span>

*   `db.collection.` `latencyStats`(选项)

       *   db.collection.latencyStats()返回给定集合的延迟统计信息。它是一个包装 `$collStats`。

此方法具有以下形式：

```powershell
db.collection.latencyStats( { histograms: <boolean> } )
```

`histograms`参数是可选的 boolean。如果`histograms: true`则latencyStats()将延迟直方图添加到 return 文档。

> **也可以看看**
>
> $collStats

## <span id="output">输出</span>

latencyStats()返回包含字段`latencyStats`的文档，其中包含以下字段：

| 字段       | 描述                       |
| ---------- | -------------------------- |
| `reads`    | 读取请求的延迟统计信息。   |
| `writes`   | 写请求的延迟统计信息。     |
| `commands` | 数据库命令的延迟统计信息。 |

每个字段都包含一个包含以下字段的嵌入式文档：

| 字段        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| `latency`   | 一个64位整数，以毫秒为单位给出总的组合延迟。                 |
| `ops`       | 一个64位整数，给出自启动以来对集合执行的操作总数。           |
| `histogram` | 嵌入式文档的 array，每个都代表一个延迟范围。每个文档涵盖以前文档范围的两倍。对于介于 2048 微秒和大约 1 秒之间的上限值，直方图包括 half-steps。 <br/>此字段仅在`latencyStats: { histograms: true }`选项的情况下存在。输出中省略了具有零`count`的空范围。 <br/>每个文档都包含以下字段：<br/>字段 :描述<br/> `micros` :一个64位整数，以毫秒为单位给出当前等待时间范围的上限时间。该文档的范围介于上一个文档的 `micros`值（不包括此值）和该文档的 值（包括不包括在内）之间。 <br/> `count` :一个64位整数，给出延迟小于或等于的操作数`micros`。 <br/>例如，如果`collStats`返回以下直方图：<br/>histogram: [<br/>   { micros: NumberLong(1), count: NumberLong(10) },<br/>   { micros: NumberLong(2), count: NumberLong(1) },<br/>   { micros: NumberLong(4096), count: NumberLong(1) },<br/>   { micros: NumberLong(16384), count: NumberLong(1000) },<br/>   { micros: NumberLong(49152), count: NumberLong(100) }<br/> ] <br/>这表示：<br/> 10 次操作占用 1 微秒或更少，<br/> 1 操作范围(1,2)微秒，<br/> 1 操作范围内的范围(3072,4096)微秒，<br/> 1000 次操作(12288,16384)和范围内的<br/> 100 次操作(32768,49152)。 |

## <span id="examples">例子</span>

您可以在mongo shell 中运行latencyStats()，如下所示：

```powershell
db.data.latencyStats( { histograms: true } ).pretty()
```

latencyStats()返回如下文档：

```powershell
{
    "ns" : "test.data",
    "localTime" : ISODate("2016-11-01T21:56:28.962Z"),
    "latencyStats" : {
        "reads" : {
            "histogram" : [
                {
                    "micros" : NumberLong(16),
                    "count" : NumberLong(6)
                },
                {
                    "micros" : NumberLong(512),
                    "count" : NumberLong(1)
                }
            ],
            "latency" : NumberLong(747),
            "ops" : NumberLong(7)
        },
        "writes" : {
            "histogram" : [
                {
                    "micros" : NumberLong(64),
                    "count" : NumberLong(1)
                },
                {
                    "micros" : NumberLong(24576),
                    "count" : NumberLong(1)
                }
            ],
            "latency" : NumberLong(26845),
            "ops" : NumberLong(2)
        },
        "commands" : {
            "histogram" : [ ],
            "latency" : NumberLong(0),
            "ops" : NumberLong(0)
        }
    }
}
```



译者：李冠飞

校对：