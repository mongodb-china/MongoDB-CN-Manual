## 日志消息

**在本页面**

- [概述](#概述)
- [结构化日志](#结构化)
- [日志消息字段类型](#类型)
- [详细程度](#详细)
- [解析结构化日志消息](#解析)
- [日志消息示例](#示例)

### <span id="概述">概述</span>

作为正常操作的一部分，MongoDB维护事件的运行日志，包括传入的连接、运行的命令和遇到的问题等条目。通常，日志消息对于诊断问题、监视部署和调优性能非常有用。

### <span id="结构化">结构化日志</span>

在MongoDB 4.4开始,[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod) /[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)实例中所有日志消息输出[结构化的JSON格式](https://docs.mongodb.com/master/reference/log-messages/ # log-message-json-output-format)。日志条目以一系列键值对的形式写入，其中每个键表示一个日志消息字段类型，比如“severity”，而每个对应的值记录该字段类型的相关日志信息，比如“information”。以前，日志条目是以明文输出的。

> 例子
>
> 以下是一个示例日志消息在JSON格式，因为它将出现在MongoDB日志文件:
>
> ```powershell
> {"t":{"$date":"2020-05-01T15:16:17.180+00:00"},"s":"I", "c":"NETWORK", "id":12345, "ctx":"listener", "msg":"Listening on","attr":{"address":"127.0.0.1"}}
> ```

> JSON日志条目可以[完美打印](https://docs.mongodb.com/master/reference/log-messages/# log-messageprettyprinting)以提高可读性。下面是相同的日志条目，打印得很整洁:

```powershell
{
  "t": {
    "$date": "2020-05-01T15:16:17.180+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 12345,
  "ctx": "listener",
  "msg": "Listening on",
  "attr": {
    "address": "127.0.0.1"
  }
}
```

> 例如，在这个日志条目中，代表严重性的键`s`具有相应的值`I`，表示[severity](https://docs.mongodb.com/master/reference/log-messages/ # log-severity-levels)，而表示[component](https://docs.mongodb.com/master/reference/log-messages/#log-message-components)的键`c`具有相应的值`NETWORK`，表示“网络”组件负责该特定消息。日志消息字段类型部分详细介绍了各种字段类型。

具有键值对的结构化日志记录允许通过自动化工具或日志摄入量服务进行高效解析，并使日志消息的编程搜索和分析更容易执行。分析结构化日志消息的示例可以在[解析结构化日志消息](https://docs.mongodb.com/master/reference/log-messages/#log-messageparsing)小节中找到。

#### JSON日志输出格式

在MongoDB 4.4中，所有日志输出现在都是JSON格式。这包括日志输出文件发送到 syslog 和 stdout (标准输出)[log destinations](https://docs.mongodb.com/master/reference/log-messages/#log-message-destinations),以及[`getLog`](https://docs.mongodb.com/master/reference/command/getLog/ # dbcmd.getLog)的输出命令。

每个日志条目作为一个自包含的JSON对象输出，遵循[放宽扩展JSON v2.0](https://docs.mongodb.com/master/reference/mongodb- extendedjson/)规范，并有以下布局和字段顺序:

```powershell
{
  "t": <Datetime>, // timestamp
  "s": <String>, // severity
  "c": <String>, // component
  "ctx": <String>, // context
  "id": <String>, // unique identifier
  "msg": <String>, // message body
  "attr": <Object> // additional attributes (optional)
  "tags": <Array of strings> // tags (optional)
  "truncated": <Object> // truncation info (if truncated)
  "size": <Integer> // original size of entry (if truncated)
}
```

- **时间戳** - 日志消息的时间戳，`ISO-8601`格式。参见[时间戳](https://docs.mongodb.com/master/reference/log-messages/ # log-message-timestamp)。
- **严重程度** - 表示日志消息的不足严重性代码的字符串。参见[严重性](https://docs.mongodb.com/master/reference/log-messages/ log-severity-levels)。
- **组件** - 表示日志消息的完整组件字符串的字符串。参见[组件](https://docs.mongodb.com/master/reference/log-messages/ log-message-components)。
- **上下文** - 表示发出日志语句的线程的名称的字符串。
- **id** - 表示日志语句的唯一标识符的字符串。参见[根据已知日志ID进行过滤](https://docs.mongodb.com/master/reference/log-messages/#log-messageparing-example-filterid)以获得一个示例。
- **消息** - 表示从服务器或驱动程序传递的原始日志输出消息的字符串。该消息根据JSON规范[根据需要进行转义](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson -escaping)。
- **属性** -*(可选)*对象，其中包含一个或多个键值对，用于提供任何附加属性。如果日志消息不包含任何附加属性，则省略此对象。属性值可以在*消息*正文中通过其键名引用，具体取决于消息。与**message**类似，属性根据JSON规范[根据需要进行转义](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson -escaping)。
- **标签** - *(可选)*字符串数组，表示适用于日志语句的任何标记，例如:**[startupWarnings]**。
- **截断** - *（如果被截断）*对象，包含有关[日志消息截断的信息](https://docs.mongodb.com/master/reference/log-messages/#log-message-truncation)（如果适用）。仅当日志条目包含至少一个被截断的**属性时**，此对象才会存在。
- **大小** - (如果被截断)整数，表示已被 [截断](https://docs.mongodb.com/master/reference/log-messages/#log-message-truncation)的日志条目的原始大小。只有当日志条目包含至少一个被截断的属性时，该字段才会出现。

##### 转译

根据[放宽扩展JSON v2.0](https://docs.mongodb.com/master/reference/mongodb- extendedjson/)规范，**message**和**属性**字段将根据需要转译控制字符:

| 字符表示            | 转义序列 |
| :------------------ | :------- |
| 引号(`"`)           | `\"`     |
| 反斜杠 (`\`)        | `\\`     |
| 回退 (`0x08`)       | `\b`     |
| 跳页 (`0x0C`)       | `\f`     |
| 换行符 (`0x0A`)     | `\n`     |
| 回车 (`0x0D`)       | `\r`     |
| 水平选项卡 (`0x09`) | `\t`     |

上面没有列出的控制字符用 **\uXXXX** 转义，其中 **XXXX** 是十六进制的unicode码点。UTF-8编码无效的字节被替换为 **\ufffd** 表示的unicode替换字符。

[examples](https://docs.mongodb.com/master/reference/log-messages/#log-messagejson -examples)提供了一个消息转译的示例。

#### 截断

任何超过[`maxLogSizeKB `](https://docs.mongodb.com/master/reference/parameters/#param.maxLogSizeKB)定义的最大大小的**属性**(默认为10kb)被截断。被截断的属性省略了超出配置限制的日志数据，但是保留了条目的JSON格式，以确保条目仍然可解析。

下面是一个带有截断属性的日志条目示例:

```powershell
{"t":{"$date":"2020-05-19T18:12:05.702+00:00"},"s":"I",  "c":"SHARDING", "id":22104,   "ctx":"conn33",
"msg":"Received splitChunk request","attr":{"request":{"splitChunk":"config.system.sessions",
"from":"production-shard1","keyPattern":{"_id":1},"epoch":{"$oid":"5ec42172996456771753a59e"},
"shardVersion":[{"$timestamp":{"t":1,"i":0}},{"$oid":"5ec42172996456771753a59e"}],"min":{"_id":{"$minKey":1}},
"max":{"_id":{"$maxKey":1}},"splitKeys":[{"_id":{"id":{"$uuid":"00400000-0000-0000-0000-000000000000"}}},
{"_id":{"id":{"$uuid":"00800000-0000-0000-0000-000000000000"}}},

...

{"_id":{"id":{"$uuid":"26c00000-0000-0000-0000-000000000000"}}},{"_id":{}}]}},
"truncated":{"request":{"splitKeys":{"155":{"_id":{"id":{"type":"binData","size":21}}}}}},
"size":{"request":46328}}
```

In this case, the `request` attribute has been truncated and the specific instance of its subfield `_id` that triggered truncation (i.e. caused the attribute to overrun [`maxLogSizeKB`](https://docs.mongodb.com/master/reference/parameters/#param.maxLogSizeKB)) is printed without data as `{"_id":{}}`. The remainder of the `request` attribute is then omitted.

在本例中，`request`属性被截断，其子字段`_id `的特定实例触发了截断(即导致属性溢出[`maxLogSizeKB`](https://docs.mongodb.com/master/reference/parameters/#param.maxLogSizeKB))被打印为`{"_id":{}}`，没有数据。然后省略`request`属性的其余部分。

包含一个或多个截断属性的日志实体包括一个`truncated`对象，该对象为日志条目中的每个截断属性提供以下信息:

- 被截断的属性
- 触发截断的属性的特定子对象(如果适用)
- 截断字段的数据**类型**
- 截断字段的**大小**

Log entries with truncated attributes may also include an additional `size` field at the end of the entry which indicates the original size of the attribute before truncation, in this case `46328` or about 46KB. This final `size` field is only shown if it is different from the `size` field in the `truncated` object, i.e. if the total object size of the attribute is different from the size of the truncated subobject, as is the case in the example above.

属性被截断的日志条目还可能在条目的末尾包含一个额外的`size`字段，该字段表示截断之前属性的原始大小，在本例中为`46328`或大约46KB。最后的`size`字段只在与`truncated`对象中的`size`字段不同的情况下显示，也就是说，如果属性的对象总大小与被截断的子对象的大小不同，就像上面的例子一样。

#### 填充

当输出到*file*或*syslog* log目的地时，在**severity**、**context**和**id**字段之后添加填充，以增加固定宽度字体时的可读性。

下面的MongoDB日志文件摘录演示了这种填充:

```powershell
{"t":{"$date":"2020-05-18T20:18:12.724+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2020-05-18T20:18:12.734+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
{"t":{"$date":"2020-05-18T20:18:12.734+00:00"},"s":"I",  "c":"NETWORK",  "id":4648601, "ctx":"main","msg":"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set tcpFastOpenServer, tcpFastOpenClient, and tcpFastOpenQueueSize."}
{"t":{"$date":"2020-05-18T20:18:12.814+00:00"},"s":"I",  "c":"STORAGE",  "id":4615611, "ctx":"initandlisten","msg":"MongoDB starting","attr":{"pid":10111,"port":27001,"dbPath":"/var/lib/mongo","architecture":"64-bit","host":"centos8"}}
{"t":{"$date":"2020-05-18T20:18:12.814+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"4.4.0","gitVersion":"328c35e4b883540675fb4b626c53a08f74e43cf0","openSSLVersion":"OpenSSL 1.1.1c FIPS  28 May 2019","modules":[],"allocator":"tcmalloc","environment":{"distmod":"rhel80","distarch":"x86_64","target_arch":"x86_64"}}}}
{"t":{"$date":"2020-05-18T20:18:12.814+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"CentOS Linux release 8.0.1905 (Core) ","version":"Kernel 4.18.0-80.11.2.el8_0.x86_64"}}}
```

#### 完美的印刷

当使用MongoDB结构化日志时，[jq命令行实用程序](https://stedolan.github.io/jq/)是一个非常有用的工具，它允许轻松地打印日志条目，以及强大的基于密钥的匹配和过滤。

`jq`是一个开源的JSON解析器，可用于Linux、Windows和macOS。

您可以使用` jq `来编辑打印日志内容，如下所示:

- 完整打印日志文件:

  ```powershell
  cat mongod.log | jq
  ```

- 完美地打印最近的日志条目:

  ```powershell
  cat mongod.log | tail -1 | jq
  ```

更多使用MongoDB结构化日志的例子可以在[解析结构化日志消息](https://docs.mongodb.com/master/reference/log-messages/# log-messageparsing)小节中找到。

#### 配置日志消息目的地

MongoDB日志消息可以输出到*file*、*syslog*或*stdout*(标准输出)。

要配置日志输出目的地，使用以下设置之一，要么在[配置文件](https://docs.mongodb.com/master/reference/configuring-options/)或在命令行:

**配置文件:**

* file*或*syslog*的[systemLog.destination](https://docs.mongodb.com/master/reference/configuring-options/#systemlog.destination)选项

**命令行:**

* 文件的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 的[`--logpath`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-logpath)选项
* 对于syslog, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 的[`--syslog`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-syslog)选项
* 文件[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的[`--logpath`](https://docs.mongodb.com/master/reference/program/mongos/#cmdoption-mongos-logpath) 选项
* 对于syslog, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的[`--syslog`](https://docs.mongodb.com/master/reference/program/mongos/#cmdoption-mongos-syslog)选项

不指定*file*或*syslog*将把所有日志输出发送到*stdout*。

有关日志设置和选项的完整列表，请参阅:

**配置文件:**

* [systemLog选项列表](https://docs.mongodb.com/master/reference/configuration-options/#systemlog-options)

**命令行:**

* 为 [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)的日志选项列表
* 为[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的日志选项列表

> 注意
>
> Error messages sent to `stderr` (standard error), such as fatal errors during startup when not using the *file* or *syslog* log destinations, or messages having to do with misconfigured logging settings, are not affected by the log output destination setting, and are printed to `stderr` in plaintext format.
>
> 发送到`stderr`的错误消息(标准错误)，比如在启动时没有使用*file*或*syslog*日志目的地时发生的致命错误，或者与日志设置配置错误有关的消息，不受日志输出目的地设置的影响，并以明文格式打印到`stderr`。

### <span id="类型">日志消息字段类型</span>

#### 时间戳

timestamp字段类型表示所记录事件发生的确切日期和时间。

```powershell
{
  "t": {
    "$date": "2020-05-01T15:16:17.180+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 12345,
  "ctx": "listener",
  "msg": "Listening on",
  "attr": {
    "address": "127.0.0.1"
  }
}
```

当登录到*file*或*syslog*时，时间戳的默认格式是`iso8601-local`。修改时间戳格式,使用[`--timeStampFormat `](https://docs.mongodb.com/master/reference/program/mongod/ cmdoption-mongod-timestampformat)运行时选项或[`systemLog.timeStampFormat`](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.timeStampFormat)设置。

请参见[按日期范围过滤](https://docs.mongodb.com/master/reference/log-messages/# log-messageparing-example-filt-timestamp)，了解在时间戳字段上过滤的日志解析示例。

> 注意
>
> 从MongoDB 4.4开始，不再支持“ctime”时间戳格式。

#### 严重程度

严重性字段类型指示与日志事件关联的严重性级别。

```powershell
{
  "t": {
    "$date": "2020-05-01T15:16:17.180+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 12345,
  "ctx": "listener",
  "msg": "Listening on",
  "attr": {
    "address": "127.0.0.1"
  }
}
```

严重级别从“致命”(最严重)到“调试”(最不严重):

| 标准        | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `F`         | Fatal                                                        |
| `E`         | Error                                                        |
| `W`         | Warning                                                      |
| `I`         | 信息性，用于[详细程度级别](https://docs.mongodb.com/master/reference/log-messages/# log-messages-configureverbosity)` 0 ` |
| `D1` - `D5` | 从4.2版本开始，MongoDB指出了特定的[调试详细级别](https://docs.mongodb.com/master/reference/log-messfigureverbosity)。例如，如果冗余级别为2,MongoDB表示“D2”。<br />在以前的版本中，MongoDB日志消息为所有调试冗长级别指定 **D**。 |

您可以指定各个组件的冗余级别，以确定MongoDB输出的**信息**和**调试**消息的数量。这些级别以上的严重性类别总是显示出来。设置详细级别，请参见[配置日志详细级别](https://docs.mongodb.com/master/reference/log-messages/# log-messages-configureverbosity)。

#### 组件

组件字段类型指示日志事件所属的类别，如**NETWORK**或**COMMAND**。

```powershell
{
  "t": {
    "$date": "2020-05-01T15:16:17.180+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 12345,
  "ctx": "listener",
  "msg": "Listening on",
  "attr": {
    "address": "127.0.0.1"
  }
}
```

每个组件都可以通过其自己的[verbosity过滤器](https://docs.mongodb.com/master/reference/log-messages/# log-messages-configureverbosity)进行单独配置。可提供的组件如下:

- **访问**

  与访问控制相关的消息，如身份验证。要指定[`ACCESS`](https://docs.mongodb.com/master/reference/log-messages/#access)组件的日志级别，使用[`systemLog.component.accessControl.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.accesscontrol.verbosity)设置。

- **命令**

  与[数据库命令](https://docs.mongodb.com/master/reference/command/)相关的消息，例如[`count`](https://docs.mongodb.com/master/reference/command/count/#dbcmd.count)。要指定[`COMMAND`](https://docs.mongodb.com/master/reference/log-messages/#命令)组件的日志级别，使用[`systemLog.component.command.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.command.verbosity)设置。

- **控制**

  与控制活动(如初始化)相关的消息。要指定[` CONTROL `](https://docs.mongodb.com/master/reference/log-messages/#control)组件的日志级别，使用[`systemLog.component.control.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.control.verbosity)设置。

- **选举**

  与复制集选举相关的消息。若要指定[`选举`](https://docs.mongodb.com/master/reference/log-messages/)组件的日志级别，请设置[`systemLog.component.replication.election.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/systemLog.component.replication.election.verbosity)参数。

  [`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL)是[`选举`](https://docs.mongodb.com/master/reference/log-messages/)的父组件。如果[`systemLog.component.replication.election.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.election.verbosity)未被设置，MongoDB对[`选举`](https://docs.mongodb.com/master/reference/log-messages/)组件使用[`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL) 详细级别。

- **FTDC**

  *新版本3.2。*与诊断数据收集机制相关的消息，如服务器统计信息和状态消息。要指定[`FTDC`](https://docs.mongodb.com/master/reference/log-messages/#ftdc)组件的日志级别，使用[`systemLog.component.ftdc.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.ftdc.verbosity)设置。

- **GEO**

  与地理空间形状解析相关的消息，例如验证GeoJSON形状。要指定[`GEO`](https://docs.mongodb.com/master/reference/log-messages/#GEO)组件的日志级别，请设置[`systemLog.component.geo.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.geo.verbosity)参数。

- **指数**

  与索引操作(如创建索引)相关的消息。要指定[`INDEX`](https://docs.mongodb.com/master/reference/log-messages/#INDEX)组件的日志级别，请设置[`systemLog.component.index.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.index.verbosity)参数。

- **INITSYNC**

  与初始同步操作相关的消息。指定的日志级别[`INITSYNC`](https://docs.mongodb.com/master/reference/log-messages/ # INITSYNC)组件,设置[`systemLog.component.replication.initialSync.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.component.replication.initialSync.verbosity)参数.

  [`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL)是[`INITSYNC`](https://docs.mongodb.com/master/reference/log-messages/#INITSYNC)的父组件。如果[`systemLog.component.replication.initialSync.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/systemLog.component.replication.initialSync.verbosity)未被设置,MongoDB使用[`REPL`](https://docs.mongodb.com/master/reference/log-messages/) REPL冗长水平[`INITSYNC`](https://docs.mongodb.com/master/reference/log-messages/ # INITSYNC)组件。

- **日志**

  与存储日志记录活动相关的消息。要指定[`JOURNAL`](https://docs.mongodb.com/master/reference/log-messages/#JOURNAL)组件的日志级别，使用[`systemLog.component.storage.journal.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.storage.journal.verbosity)设置。

  [`STORAGE`](https://docs.mongodb.com/master/reference/log-messages/#STORAGE)是[`JOURNAL`](https://docs.mongodb.com/master/reference/log-messages/#JOURNAL)的父组件。如果[`systemLog.component.storage.journal.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/systemLog.component.storage.journal.verbosity)未被设置,MongoDB使用[`存储`](https://docs.mongodb.com/master/reference/log-messages/存储)冗长水平[`JOURNAL`](https://docs.mongodb.com/master/reference/log-messages/#JOURNAL)组件。

- **网络**

  与网络活动相关的消息，例如接受连接。要指定[`NETWORK`](https://docs.mongodb.com/master/reference/log-messages/#NETWORK)组件的日志级别，请设置[`systemLog.component.network.verbosity `](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.network.verbosity)参数。

- **查询**

  与查询相关的消息，包括查询规划器活动。要指定[`QUERY`](https://docs.mongodb.com/master/reference/log-messages/#QUERY)组件的日志级别，请设置[`systemLog.component.query.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.query.verbosity)参数。

- **恢复**

  与存储恢复活动相关的消息。要指定[`RECOVERY`](https://docs.mongodb.com/master/reference/log-messages/#recovery)组件的日志级别，使用[`systemLog.component.storage.recovery.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.storage.recovery.verbosity)设置。

  [`STORAGE`](https://docs.mongodb.com/master/reference/log-messages/#STORAGE)是[`RECOVERY`](https://docs.mongodb.com/master/reference/log-messages/#RECOVERY)的父组件。如果[`systemLog.component.storage.recovery.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.component.storage.recovery.verbosity)未被设置,MongoDB使用[`存储`](https://docs.mongodb.com/master/reference/log-messages/存储)冗长水平[`恢复`](https://docs.mongodb.com/master/reference/log-messages/)组件。

- **REPL**

  与复制集相关的消息，如初始同步、心跳、稳定状态复制和回滚。为[`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL)组件指定日志级别，设置[`systemLog.component.replication.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.component.replication.verbosity)参数。

  [`REPL`](https://docs.mongodb.com/master/reference/log-messages/) 的父组件[`选举`](https://docs.mongodb.com/master/reference/log-messages/), [`INITSYNC`](https://docs.mongodb.com/master/reference/log-messages/ # INITSYNC),[`REPL_HB`](https://docs.mongodb.com/master/reference/log-messages/#REPL_HB)和[`ROLLBACK`](https://docs.mongodb.com/master/reference/log-messages/#ROLLBACK)组件。

- **REPL_HB**

  与复制集心跳相关的消息。指定的日志级别[`REPL_HB`](https://docs.mongodb.com/master/reference/log-messages/ # REPL_HB)组件,设置[`systemLog.component.replication.heartbeats.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.component.replication.heartbeats.verbosity)参数。

  [`REPL`](https://docs.mongodb.com/master/reference/log-messages/#repl)是[`REPL_HB`](https://docs.mongodb.com/master/reference/log-messages/#REPL_HB)的父组件。如果[`systemLog.component.replication.heartbeats.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/systemLog.component.replication.heartbeats.verbosity)未被设置,MongoDB使用[`REPL`](https://docs.mongodb.com/master/reference/log-messages/) REPL冗长水平[`REPL_HB`](https://docs.mongodb.com/master/reference/log-messages/ # REPL_HB)组件。

- **回滚**

  与[回滚](https://docs.mongodb.com/master/core/replica-set-rollbacks/#replica-set-rollbacks)操作相关的消息。指定的日志级别[`回滚`](https://docs.mongodb.com/master/reference/log-messages/回滚)组件,设置[`systemLog.component.replication.rollback.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.component.replication.rollback.verbosity)参数。

  [`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL)是[`ROLLBACK`](https://docs.mongodb.com/master/reference/log-messages/#ROLLBACK)的父组件。如果[`systemLog.component.replication.rollback.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/systemLog.component.replication.rollback.verbosity)未被设置,MongoDB使用[`REPL`](https://docs.mongodb.com/master/reference/log-messages/) 冗长水平[`回滚`](https://docs.mongodb.com/master/reference/log-messages/回滚)组件。

- **分片**

  与分片活动相关的消息，例如[` mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)的启动。要指定[`SHARDING`](https://docs.mongodb.com/master/reference/log-messages/#sharding)组件的日志级别，使用[`systemLog.component.sharding.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.sharding.verbosity)设置。

- **存储**

  与存储活动相关的消息，例如[`fsync`](https://docs.mongodb.com/master/reference/command/fsync/#dbcmd.fsync)命令所涉及的进程。要指定[`STORAGE`](https://docs.mongodb.com/master/reference/log-messages/#STORAGE)组件的日志级别，使用[`systemLog.component.storage.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.storage.verbosity)设置。

  [`STORAGE`](https://docs.mongodb.com/master/reference/log-messages/#STORAGE)是[`JOURNAL`](https://docs.mongodb.com/master/reference/log-messages/#RECOVERY)和 [`RECOVERY`](https://docs.mongodb.com/master/reference/log-messages/#RECOVERY)的父组件。

- **TXN**

  *新版本4.0.2.*

  与[多文档事务](https://docs.mongodb.com/master/core/transactions/)相关的消息。要指定[`TXN`](https://docs.mongodb.com/master/reference/log-messages/#TXN)组件的日志级别，使用[`systemLog.component.transaction.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.transaction.verbosity)设置。

- **写**

  与写操作相关的消息，例如[`update`](https://docs.mongodb.com/master/reference/command/update/#dbcmd.update)命令。要指定[`WRITE`](https://docs.mongodb.com/master/reference/log-messages/#WRITE)组件的日志级别，使用[`systemLog.component.write.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.write.verbosity)设置。

- `-`

  与命名组件不关联的消息。未命名组件在[`systemLog.verbosity`](https://docs.mongodb.com/master/reference/configuring-options/#systemlog.verbosity)设置中指定了默认的日志级别。[`systemLog.verbosity`](https://docs.mongodb.com/master/reference/configuring-options/#systemlog.verbosity)设置是已命名和未命名组件的默认设置。

#### 客户端数据

[MongoDB驱动程序](https://docs.mongodb.com/products/drivers/)和客户端应用程序(包括[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell)有能力在连接到服务器的时候发送识别信息。连接建立后，客户端不会再次发送标识信息，除非连接被删除并重新建立。

该标识信息包含在日志条目的**attributes**字段中。所包括的确切信息因客户端而异。

下面是一个示例日志消息，包含从一个[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell连接传输的客户端数据文档。客户端数据包含在 **doc** 对象的**属性**字段中:

```powershell
{"t":{"$date":"2020-05-20T16:21:31.561+00:00"},"s":"I",  "c":"NETWORK",  "id":51800,   "ctx":"conn202","msg":"client metadata","attr":{"remote":"127.0.0.1:37106","client":"conn202","doc":{"application":{"name":"MongoDB Shell"},"driver":{"name":"MongoDB Internal Client","version":"4.4.0"},"os":{"type":"Linux","name":"CentOS Linux release 8.0.1905 (Core) ","architecture":"x86_64","version":"Kernel 4.18.0-80.11.2.el8_0.x86_64"}}}}
```

当[replica set](https://docs.mongodb.com/master/core/replica-set-members/)的次要成员初始化到主节点的连接时，它们发送类似的数据。包含此启动连接的日志消息示例如下所示。客户端数据包含在 **doc** 对象的**属性**字段中:

```powershell
{"t":{"$date":"2020-05-20T16:33:40.595+00:00"},"s":"I",  "c":"NETWORK",  "id":51800,   "ctx":"conn214","msg":"client metadata","attr":{"remote":"127.0.0.1:37176","client":"conn214","doc":{"driver":{"name":"NetworkInterfaceTL","version":"4.4.0"},"os":{"type":"Linux","name":"CentOS Linux release 8.0.1905 (Core) ","architecture":"x86_64","version":"Kernel 4.18.0-80.11.2.el8_0.x86_64"}}}}
```

参见[示例部分](https://docs.mongodb.com/master/reference/log-messages/#log-messagejjson -examples)以获得一个显示客户端数据的[格式打印](https://docs.mongodb.com/master/reference/log-messages/#log-message-pretty-printing)示例。

有关客户端信息和必需字段的完整描述，请参见[MongoDB握手规范](https://github.com/mongodb/specifications/blob/master/source/mongodb-handshake/handshake.rst)。

### <span id="程度">详细程度</span>

您可以指定日志记录冗余级别，以增加或减少MongoDB输出的日志消息数量。可以针对所有组件一起调整详细级别，也可以针对特定的[已命名组件](https://docs.mongodb.com/master/reference/log-messages/#log-messagecomponents)单独调整详细级别。

详细程度只影响[severity](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)类别**information**和**Debug**中的日志名称。这些级别以上的严重性类别总是显示出来。

您可以将冗余级别设置为高值，以显示用于调试或开发的详细日志记录，或设置为低值，以尽量减少对经过审查的生产部署上的日志的写操作。

#### 查看当前日志的详细程度

要查看当前的详细级别，使用[`db.getLogComponents()`](https://docs.mongodb.com/master/reference/method/db.getLogComponents/#db.getLogComponents)方法:

```powershell
db.getLogComponents()
```

您的输出可能类似如下:

```powershell
{
 "verbosity" : 0,
 "accessControl" : {
    "verbosity" : -1
 },
 "command" : {
    "verbosity" : -1
 },
 ...
 "storage" : {
    "verbosity" : -1,
    "recovery" : {
       "verbosity" : -1
    },
    "journal" : {
        "verbosity" : -1
    }
 },
 ...
```

最初的`冗长`为所有组件条目父冗长水平,而个人[命名组件](https://docs.mongodb.com/master/reference/log-messages/ # log-message-components),如`accessControl`,指示组件的特定详细级别,覆盖全球冗长的水平如果设置特定的组件。

值`-1`表示，如果组件有父组件的冗余级别(与上面的`recovery`一样，从` storage `继承)，则组件继承父组件的冗余级别，如果没有，则继承全局冗余级别(与` command `一样)。详细级别的继承关系在[components](https://docs.mongodb.com/master/reference/log-messages/#log-messagecomponents)部分中指明。

#### 配置日志冗余级别

您可以配置使用冗长水平:[`systemLog.verbosity `](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.verbosity)和 `systemLog.component.<name>.verbosity` 设置,[`logComponentVerbosity`](https://docs.mongodb.com/master/reference/parameters/ # param.logComponentVerbosity)的参数,或[`db.setLogLevel()`](https://docs.mongodb.com/master/reference/method/db.setLogLevel/ # db.setLogLevel)方法。

#### systemLog冗长的设置

要为所有[组件](https://docs.mongodb.com/master/reference/log-messages/# log-messagecomponents)配置默认的日志级别，使用[`systemLog.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/ #systemLog.verbosity)设置。要配置特定组件的级别，请使用 `systemLog.component.<name>.verbosity` 设置。

例如,下面的配置设置 [`systemLog.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.verbosity) 冗长`1 `,在[`systemLog.component.query.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.component.query.verbosity)冗长`2`, [`systemLog.component.storage.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity)冗长 ` 2 `,和[`systemLog.component.storage.journal.verbosity `](https://docs.mongodb.com/master/reference/configurationoptions/ #systemLog.component.storage.journal.verbosity)到`1`:

```powershell
systemLog:
   verbosity: 1
   component:
      query:
         verbosity: 2
      storage:
         verbosity: 2
         journal:
            verbosity: 1
```

你会设置这些值[配置文件](https://docs.mongodb.com/master/reference/configuration-options/)或在命令行上为你的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)实例。

所有组件未指定明确的配置有一个冗长的`1`,表明他们继承冗长的父级,如果他们有一个,或全球冗长级别([` systemLog.verbosity `](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.verbosity))如果他们不这样做。

#### `logComponentVerbosity`参数

要设置[`logComponentVerbosity `](https://docs.mongodb.com/master/reference/parameters/#param.logComponentVerbosity)参数，需要传递一个文档，其中包含要更改的冗长设置。

For example, the following sets the [`default verbosity level`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.verbosity) to `1`, the [`query`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.query.verbosity) to `2`, the [`storage`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity) to `2`, and the [`storage.journal`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.journal.verbosity) to `1`.

例如,下面的设置[`默认的详细级别`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.verbosity) 冗长`1 `,在[‘查询’](https://docs.mongodb.com/master/reference/configuration-options/ systemLog.component.query.verbosity)冗长`2`,[`storage`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity)冗长`2` ,和[`storage.journal`](https://docs.mongodb.com/master/reference/configuring-options/#systemlog.component.storage.journal.verbosity)到` 1 `。

```powershell
db.adminCommand( {
   setParameter: 1,
   logComponentVerbosity: {
      verbosity: 1,
      query: {
         verbosity: 2
      },
      storage: {
         verbosity: 2,
         journal: {
            verbosity: 1
         }
      }
   }
} )
```

您可以从[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell中设置这些值。

#### `db.setLogLevel()`

使用[`db.setLogLevel()`](https://docs.mongodb.com/master/reference/method/db.setloglevel/#db . setloglevel)方法更新单个组件日志级别。对于组件，可以指定`0`到`5`的冗余级别，也可以指定`-1`来继承父组件的冗余。例如，下面将[`systemLog.component.query.verbosity`](https://docs.mongodb.com/master/reference/configurationoptions/#systemlog.component.query.verbosity)设置为其父级的冗长(即默认冗长):

```powershell
db.setLogLevel(-1, "query")
```

您可以从[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell中设置该值。

#### 日志记录操作缓慢

客户操作(例如查询)出现在日志如果他们的持续时间超过[缓慢操作阈值](https://docs.mongodb.com/master/reference/command/profile/ # slowms-threshold-option)或者[日志详细级别](https://docs.mongodb.com/master/reference/log-messages/ # log-message-verbosity-levels)是1或更高。这些日志条目包括与操作关联的完整命令对象。

从MongoDB 4.2开始，用于读/写操作的[profiler条目](https://docs.mongodb.com/master/tutorial/manage-the-databe-profiler/)和[诊断日志消息(即mongod/mongos日志消息)](https://docs.mongodb.com/master/reference/log-messages/# log-message慢ops)包括:

- `queryHash`帮助识别具有相同[查询形状](https://docs.mongodb.com/master/reference/glossary/#term-query-shape)的慢速查询。
- `planCacheKey`为慢速查询提供了更多关于[查询计划缓存](https://docs.mongodb.com/master/core/query-plans/)的信息。

下面的示例输出包含一个缓慢的[聚合](https://docs.mongodb.com/master/aggregation/)操作的信息:

```powershell
{"t":{"$date":"2020-05-20T20:10:08.731+00:00"},"s":"I",  "c":"COMMAND",  "id":51803,   "ctx":"conn281","msg":"Slow query","attr":{"type":"command","ns":"stocks.trades","appName":"MongoDB Shell","command":{"aggregate":"trades","pipeline":[{"$project":{"ticker":1.0,"price":1.0,"priceGTE110":{"$gte":["$price",110.0]},"_id":0.0}},{"$sort":{"price":-1.0}}],"allowDiskUse":true,"cursor":{},"lsid":{"id":{"$uuid":"fa658f9e-9cd6-42d4-b1c8-c9160fabf2a2"}},"$clusterTime":{"clusterTime":{"$timestamp":{"t":1590005405,"i":1}},"signature":{"hash":{"$binary":{"base64":"AAAAAAAAAAAAAAAAAAAAAAAAAAA=","subType":"0"}},"keyId":0}},"$db":"test"},"planSummary":"COLLSCAN","cursorid":1912190691485054730,"keysExamined":0,"docsExamined":1000001,"hasSortStage":true,"usedDisk":true,"numYields":1002,"nreturned":101,"reslen":17738,"locks":{"ReplicationStateTransition":{"acquireCount":{"w":1119}},"Global":{"acquireCount":{"r":1119}},"Database":{"acquireCount":{"r":1119}},"Collection":{"acquireCount":{"r":1119}},"Mutex":{"acquireCount":{"r":117}}},"storage":{"data":{"bytesRead":232899899,"timeReadingMicros":186017},"timeWaitingMicros":{"cache":849}},"protocol":"op_msg","durationMillis":22427}}
```

参见[examples部分](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson -examples)，以获得该日志条目的[pretty- printing](https://docs.mongodb.com/master/reference/log-messages/#log-message-pretty-printing)版本。

### <span id="解析">解析结构化日志消息</span>

日志解析是通过编程方式搜索和分析日志文件的行为，通常采用自动化的方式。随着MongoDB 4.4中结构化日志的引入，日志解析变得更加简单和强大。例如:

- 日志消息字段以键值对的形式显示。日志解析器可以根据感兴趣的特定键进行查询，从而有效地筛选结果。
- 日志消息总是包含相同的消息结构。日志解析器可以可靠地从任何日志消息中提取信息，而不需要为信息丢失或格式不同的情况编写代码。

下面的示例演示使用MongoDB JSON日志输出时常见的日志解析工作流。

#### 日志解析的例子

当使用MongoDB结构化日志时，[jq命令行实用程序](https://stedolan.github.io/jq/)是一个非常有用的工具，它允许轻松地打印日志条目，以及强大的基于密钥的匹配和过滤。

`jq`是一个开源的JSON解析器，可用于Linux、Windows和macOS。

这些示例使用`jq`来简化日志解析。

#### 计数独特的消息

下面的示例显示了给定日志文件中按频率排序的前10个唯一消息值:

```powershell
jq -r ".msg" /var/log/mongodb/mongod.log | sort | uniq -c | sort -rn | head -10
```

#### 监视连接

远程客户端连接显示在属性对象的“Remote”键下的日志中。下面将计算整个日志文件过程中的所有唯一连接，并按出现次数降序显示它们:

```powershell
jq -r '.attr.remote' /var/log/mongodb/mongod.log | grep -v 'null' | sort | uniq -c | sort -r
```

请注意，来自相同IP地址但通过不同端口连接的连接将被此命令视为不同的连接。您可以限制输出仅考虑IP地址，如下更改:

```powershell
jq -r '.attr.remote' /var/log/mongodb/mongod.log | grep -v 'null' | awk -F':' '{print $1}' | sort | uniq -c | sort -r
```

#### 分析客户类型

下面的例子分析报告[客户端数据](https://docs.mongodb.com/master/reference/log-messages/) log-messages-client-data远程[MongoDB驱动](https://docs.mongodb.com/ecosystem/drivers/)连接的客户机应用程序,包括 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell,并打印每个独特的操作系统类型,总联系,按频率:

```powershell
jq -r '.attr.doc.os.type' /var/log/mongodb/mongod.log | grep -v null | sort | uniq -c | sort -rn
```

这个日志字段中报告的字符串“Darwin”表示macOS客户机。

#### 慢速查询分析

启用了[慢操作日志](https://docs.mongodb.com/master/reference/log-messages/# log-messageslow -ops)，下面只返回耗时超过2000毫秒的慢操作:，供进一步分析:

```powershell
jq '. | select(.attr.durationMillis>=2000)' /var/log/mongodb/mongod.log
```

查阅[jq文档](https://stedolan.github.io/jq/manual/)以获得更多关于本例中显示的`jq`过滤器的信息。

#### 过滤已知的日志ID

日志id ([JSON日志输出格式](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson - outing-format)中的第5个字段)映射到特定的日志事件，可以依赖它在后续的MongoDB版本中保持稳定。

例如，您可能对以下两个日志事件感兴趣，它们显示了客户机连接后断开连接:

```powershell
{"t":{"$date":"2020-06-01T13:06:59.027-0500"},"s":"I", "c":"NETWORK", "id":22943,"ctx":"listener","msg":"connection accepted from {session_remote} #{session_id} ({connectionCount}{word} now open)","attr":{"session_remote":"127.0.0.1:61298","session_id":164,"connectionCount":11,"word":" connections"}}
{"t":{"$date":"2020-06-01T13:07:03.490-0500"},"s":"I", "c":"NETWORK", "id":22944,"ctx":"conn157","msg":"end connection {remote} ({connectionCount}{word} now open)","attr":{"remote":"127.0.0.1:61298","connectionCount":10,"word":" connections"}}
```

这两个实体的日志id分别是`22943`和` 22944 `。然后，您可以过滤日志输出，只显示这些日志id，有效地只显示客户端连接活动，使用以下`jq`语法:

```powershell
jq '. | select( .id as $id | [22943, 22944] | index($id) )' /var/log/mongodb/mongod.log
```

查阅[jq文档](https://stedolan.github.io/jq/manual/)以获得更多关于本例中显示的`jq`过滤器的信息。

#### 日期范围过滤

通过过滤timestamp字段，限制返回到特定日期范围的日志记录，可以进一步细化日志输出。例如，下面返回发生在2020年4月15日的所有日志条目:

```powershell
jq '. | select(.t["$date"] >= "2020-04-15T00:00:00.000" and .t["$date"] <= "2020-04-15T23:59:59.999")' /var/log/mongodb/mongod.log
```

注意，该语法包含完整的时间戳，包括毫秒，但不包括时区偏移量。

按日期范围进行筛选可以与上面的任何示例结合使用，例如创建周报告或年度摘要。下面的语法扩展了“监视连接”示例，将结果限制在2020年5月:

```powershell
jq '. | select(.t["$date"] >= "2020-05-01T00:00:00.000" and .t["$date"] <= "2020-05-31T23:59:59.999" and .attr.remote)' /var/log/mongodb/mongod.log
```

查阅[jq文档](https://stedolan.github.io/jq/manual/)以获得更多关于本例中显示的`jq`过滤器的信息。

#### 日志摄入服务

日志摄取服务是接收和聚合日志文件(通常来自分布式系统集群)的第三方产品，并在中心位置提供对该数据的持续分析。

[JSON日志格式](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson - outing-format)，在MongoDB 4.4中引入，在处理日志接收和分析服务时允许更大的灵活性。纯文本日志通常需要某种转换方式才能与这些产品一起使用，而JSON文件通常可以开箱即用，这取决于服务。此外，在对这些服务执行过滤时，json格式的日志提供了更多的控制，因为键-值结构提供了仅具体导入感兴趣的字段，而忽略其他字段的能力。

有关更多信息，请参阅您所选择的第三方日志摄取服务的文档。

### 日志消息事例

下面的示例展示了JSON输出格式的日志消息。

为了方便起见，这些日志消息以[完美打印格式](https://docs.mongodb.com/master/reference/log-messages/# log-messageprettyprinting)的形式呈现。

#### 启动预警

这个例子显示了一个启动警告:

```powershell
{
  "t": {
    "$date": "2020-05-20T19:17:06.188+00:00"
  },
  "s": "W",
  "c": "CONTROL",
  "id": 22120,
  "ctx": "initandlisten",
  "msg": "Access control is not enabled for the database. Read and write access to data and configuration is unrestricted",
  "tags": [
    "startupWarnings"
  ]
}
```

#### 客户端连接

这个例子显示了一个客户端连接，它包含了[客户端数据](https://docs.mongodb.com/master/reference/log-messages/#log-messages-client-data):

```powershell
{
  "t": {
    "$date": "2020-05-20T19:18:40.604+00:00"
  },
  "s": "I",
  "c": "NETWORK",
  "id": 51800,
  "ctx": "conn281",
  "msg": "client metadata",
  "attr": {
    "remote": "192.168.14.15:37666",
    "client": "conn281",
    "doc": {
      "application": {
        "name": "MongoDB Shell"
      },
      "driver": {
        "name": "MongoDB Internal Client",
        "version": "4.4.0"
      },
      "os": {
        "type": "Linux",
        "name": "CentOS Linux release 8.0.1905 (Core) ",
        "architecture": "x86_64",
        "version": "Kernel 4.18.0-80.11.2.el8_0.x86_64"
      }
    }
  }
}
```

#### 缓慢操作

这个例子显示了一个[慢速操作消息](https://docs.mongodb.com/master/reference/log-messages/# log-messageslow -ops):

```powershell
{
  "t": {
    "$date": "2020-05-20T20:10:08.731+00:00"
  },
  "s": "I",
  "c": "COMMAND",
  "id": 51803,
  "ctx": "conn281",
  "msg": "Slow query",
  "attr": {
    "type": "command",
    "ns": "stocks.trades",
    "appName": "MongoDB Shell",
    "command": {
      "aggregate": "trades",
      "pipeline": [
        {
          "$project": {
            "ticker": 1,
            "price": 1,
            "priceGTE110": {
              "$gte": [
                "$price",
                110
              ]
            },
            "_id": 0
          }
        },
        {
          "$sort": {
            "price": -1
          }
        }
      ],
      "allowDiskUse": true,
      "cursor": {},
      "lsid": {
        "id": {
          "$uuid": "fa658f9e-9cd6-42d4-b1c8-c9160fabf2a2"
        }
      },
      "$clusterTime": {
        "clusterTime": {
          "$timestamp": {
            "t": 1590005405,
            "i": 1
          }
        },
        "signature": {
          "hash": {
            "$binary": {
              "base64": "AAAAAAAAAAAAAAAAAAAAAAAAAAA=",
              "subType": "0"
            }
          },
          "keyId": 0
        }
      },
      "$db": "test"
    },
    "planSummary": "COLLSCAN",
    "cursorid": 1912190691485054700,
    "keysExamined": 0,
    "docsExamined": 1000001,
    "hasSortStage": true,
    "usedDisk": true,
    "numYields": 1002,
    "nreturned": 101,
    "reslen": 17738,
    "locks": {
      "ReplicationStateTransition": {
        "acquireCount": {
          "w": 1119
        }
      },
      "Global": {
        "acquireCount": {
          "r": 1119
        }
      },
      "Database": {
        "acquireCount": {
          "r": 1119
        }
      },
      "Collection": {
        "acquireCount": {
          "r": 1119
        }
      },
      "Mutex": {
        "acquireCount": {
          "r": 117
        }
      }
    },
    "storage": {
      "data": {
        "bytesRead": 232899899,
        "timeReadingMicros": 186017
      },
      "timeWaitingMicros": {
        "cache": 849
      }
    },
    "protocol": "op_msg",
    "durationMillis": 22427
  }
}
```

#### 转义

这个例子演示了[字符转义](https://docs.mongodb.com/master/reference/log-messages/# log-messagejson -escaping)，如属性对象的 **setName** 字段所示:

```powershell
{
  "t": {
    "$date": "2020-05-20T19:11:09.268+00:00"
  },
  "s": "I",
  "c": "REPL",
  "id": 21752,
  "ctx": "ReplCoord-0",
  "msg": "Scheduling remote command request",
  "attr": {
    "context": "vote request",
    "request": "RemoteCommand 229 -- target:localhost:27003 db:admin cmd:{ replSetRequestVotes: 1, setName: \"my-replica-name\", dryRun: true, term: 3, candidateIndex: 0, configVersion: 2, configTerm: 3, lastAppliedOpTime: { ts: Timestamp(1589915409, 1), t: 3 } }"
  }
}
```

